# Plan: user accounts & authentication for qc-studio server deployments

## Context

An admin runs qc-studio on a server and raters connect from a browser. Right now there is no authentication.
- The rater identity is free text typed into the landing-page form ([landing_page.py:478](ui/views/landing_page.py#L478)) or passed with `--rater_id`.
- Importing a TSV takes on the identity of whoever wrote it ([landing_page.py:299](ui/views/landing_page.py#L299)).
- The rater ID is only used to prefix output files in the shared `--output_dir`.

This means anyone who can reach the port can rate as anyone.

Goals:
- Only users the admin has listed can get in.
- Each user's rater ID is fixed to their account.
- The admin can assign each user a subset of participants/sessions (and tasks).

Constraints (from your answers):
- Deployment differs by site: sometimes on an institutional network/VPN, sometimes public.
- Imaging data must not go through a third-party proxy.
- Login uses tokens issued by the admin.

### Options considered

| Option | Login identity | Pros | Cons |
|---|---|---|---|
| **A. Built-in tokens (recommended)** | Admin-issued token / magic link | No external dependencies; works offline, on a VPN or in public; data stays on the server; fits the per-user assignment model | We store the tokens ourselves (only hashes, and the tokens are random and high-entropy); needs TLS when public |
| B. Streamlit `st.login` (OIDC) | Email via Google, Microsoft Entra, Keycloak… | Real SSO with no passwords to manage; already supported by the pinned `streamlit>=1.56` | Needs an IdP app registration and a public HTTPS redirect URI, which doesn't fit air-gapped or VPN-only sites |
| C. Cloudflare Access / Zero Trust | Email OTP or SSO at the edge | No auth code in the app; the app reads a header | All data goes through Cloudflare (ruled out by your data policy); needs a domain on Cloudflare |
| D. Self-hosted proxy (oauth2-proxy, Authelia, Caddy basic-auth) | Varies | Data stays on-site; also protects `/media` URLs | More ops work per site; still needs app work to map the user to a rater and their assignments |

**Recommendation:** build A into the app. Write the identity check behind a small interface so B or D can be added later as `--auth_mode oidc|header` without touching the rest of the app. Don't use Cloudflare.

## Design

### 1. Admin-managed files (opt-in, backward compatible)
New optional CLI flags in `parse_args` ([ui/main.py:30](ui/main.py#L30)):
- `--users_file users.tsv`. If this is absent, the app runs exactly as it does today (local single-user mode).
- `--assignments_file assignments.tsv`. Optional. If absent, everyone sees the full cohort.

`users.tsv` columns:
- `user_id`: becomes the rater ID (lowercase, filename-safe).
- `display_name`
- `email`: optional, for reference only.
- `token_sha256`
- `qc_tasks`: optional and comma-separated; blank means all tasks.
- `active`: `true`/`false`.

`assignments.tsv` columns: `user_id`, `participant_id`, `session_id` (blank means all sessions).

Both use TSV to match the project's existing convention.

### 2. Admin CLI: new `ui/auth_admin.py`
- `python ui/auth_admin.py add-user --users_file users.tsv --user_id alice [--email ...] [--base_url https://host:8501]`
  - Makes a token with `secrets.token_urlsafe(32)` and stores only its SHA-256 hash.
  - Prints the token **once**, plus a magic link `<base_url>/?token=...`.
- `rotate-token`, `deactivate` and `list` subcommands.
- Plain SHA-256 is enough because the tokens are random and high-entropy, so a slow hash isn't needed.

### 3. Auth gate: new `ui/auth.py`
- `load_users(path)` and `load_assignments(path)` are cached by file mtime. That way the admin can add, deactivate or rotate users without restarting the server.
- `require_login(users_file)`:
  - Runs at the very top of `main()` and of each multipage entry (`ui/pages/1_Landing_Page.py`, `ui/pages/2_Congratulations_Page.py`), before any data is read.
  - Takes the token from `st.query_params["token"]` (magic link) or from a password-type `st.text_input` form.
  - Compares hashes with `hmac.compare_digest`. On success it stores the user in session_state (new key in `SESSION_KEYS`, [ui/constants.py:74](ui/constants.py#L74)) and then calls `st.query_params.clear()`.
  - Re-checks `active` on every rerun, so deactivating someone ends their access. Otherwise it calls `st.stop()`.
- Adds a sidebar "Signed in as X · Log out" control.
- Does nothing when `--users_file` isn't given.

### 4. Lock rater identity
- When authenticated, `SessionManager.set_rater_id(user.user_id)` is called in `main()` ([main.py:176](ui/main.py#L176)), and `--rater_id` is ignored.
- `_display_rater_form` ([landing_page.py:473](ui/views/landing_page.py#L473)) shows the rater ID as a disabled field. Experience, fatigue and screen size stay editable.
- In the TSV import confirm ([landing_page.py:295-302](ui/views/landing_page.py#L295-L302)), if the file's `rater_id` doesn't match the logged-in user, the import is refused with an error. It no longer takes on that identity.
- Output and checkpoint filenames already depend on `SessionManager.get_rater_id()` ([main.py:195-201](ui/main.py#L195-L201), [qc_viewer.py:999-1091](ui/components/qc_viewer.py#L999)). They need no changes, and raters can no longer overwrite each other's files.

### 5. Per-user assignment
- In `get_cli_run_context` ([main.py:122-130](ui/main.py#L122-L130)), after `build_qc_cohort`, apply a new `filter_cohort_for_user(qc_cohort, assignments, user_id)` in `ui/utils/cohort.py`, next to `build_qc_cohort`.
  - Everything that comes after it (total_participants, participant_ids, sidebar nav) then uses only the assigned subset.
- Narrow `qc_tasks` from `resolve_qc_tasks` to the user's allowed `qc_tasks`.
- Use the same allow-list in `_landing_qc_task_options` on the landing page.
- A user with an empty assignment gets a "No participants assigned — contact the admin" message.

### 6. Deployment docs: new `docs/source/guides/server_deployment.md`, plus updates to `configuration.md`
- **VPN/institutional:** bind Streamlit to `127.0.0.1` and either have raters use an SSH tunnel or put nginx/Caddy with an institutional certificate in front.
- **Public:** put Caddy in front (automatic Let's Encrypt) or use Streamlit's `server.sslCertFile`/`sslKeyFile`. TLS is required because tokens are bearer credentials.
- Add `users.tsv`/`assignments.tsv` to `.gitignore` and warn against committing them.
- **Known limitations:**
  - Streamlit `/media/<hash>` URLs aren't auth-checked. They are unguessable and short-lived, but sites that need strict protection can add proxy-level auth (option D) on top.
  - Magic links stay in browser history, so tell users to rotate tokens if a link is shared.
  - One user in two tabs writes to the same status file.

## Files
- **New:** `ui/auth.py`, `ui/auth_admin.py`, `ui/tests/test_auth.py`, `docs/source/guides/server_deployment.md`
- **Modified:**
  - `ui/main.py` (flags, gate, lock rater, filter cohort)
  - `ui/pages/1_Landing_Page.py`, `ui/pages/2_Congratulations_Page.py` (gate)
  - `ui/views/landing_page.py` (disabled rater field, import mismatch check, task options)
  - `ui/utils/cohort.py` (`filter_cohort_for_user`)
  - `ui/constants.py` (session key, messages)
  - `docs/source/guides/configuration.md`
  - `.gitignore`

## Verification
1. **Unit tests** (`pytest ui/tests/test_auth.py`):
   - token hash/verify
   - users/assignments parsing, including blank and inactive rows
   - `filter_cohort_for_user`
   - task allow-list
   - import mismatch rejection logic
2. **Back-compat:** run `fmriprep_demo.sh` without `--users_file`. The app should behave as before, and existing tests should pass.
3. **End-to-end:**
   - Create two users with `auth_admin.py add-user`, give them different assignments, and launch with `--users_file/--assignments_file`.
   - Open both magic links in separate browser profiles. Check that each sees only their own participants and tasks, that the rater field is locked, and that output files are `alice_*.tsv` / `bob_*.tsv`.
   - Check that a bad token is rejected, that importing Bob's TSV as Alice is refused, and that running `deactivate alice` logs Alice out on her next interaction.
