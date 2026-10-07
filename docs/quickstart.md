# Quickstart

The fastest way to see QC-Studio end to end is the bundled fMRIPrep demo. It runs against
`sample_data/` — a small BIDS dataset with fMRIPrep derivatives already in place — so you need
no external data and no pipeline run.

## 1. Install

Follow [Installation](installation.md) if you have not already. The Niivue component is optional
for the demo.

## 2. Run the demo

From the repository root:

```bash
./fmriprep_demo.sh
```

Streamlit starts on port 8501 and prints the launch configuration:

```text
Launching QC-Studio with:
  qc_json=./pipelines/fmriprep/qc_demo.json
  qc_json_for_ui=../pipelines/fmriprep/qc_demo.json
  qc_task=anat_wf_qc
  dataset_dir=sample_data
  participant_list=sample_data/qc_participants_demo.tsv
  session_list=ses-01
  port=8501
```

Open <http://localhost:8501>. The demo reviews **`sub-ED01`**, session **`ses-01`**, task
**`anat_wf_qc`**.

Everything the script does can be overridden with environment variables:

```bash
PORT=8502 QC_TASK=sdc_wf_qc SESSION_LIST=ses-01 ./fmriprep_demo.sh
```

## 3. Take your first ratings

**Landing page** — three columns:

1. **👤 Rater Information** — enter your rater name or ID, pick your QC experience level, how
   tired you feel, and your monitor's screen size. The autoplay duration slider is here too.
   Your rater ID is lowercased and stripped of spaces, because it is written into the exported
   filenames.
2. **🖼️ Display Panels** — choose which panels to show (3D MRI viewer, Montage, QC Metrics), and
   set the montage grid's maximum rows and columns. At least one panel is required.
3. **📤 Upload Existing QC File** — optional. Leave it empty for a fresh run.

Press **✅ Continue to QC**.

**QC viewer** — one page per (participant, session):

- Left sidebar: **▶️ Play** / **⏸️ Pause** for [Autoplay](autoplay.md), **◀️ Previous** /
  **Next ▶️**, the page counter, and the QC subject list with a search box.
- Main area: the selected panels for the task, then **📊 Rate** with **PASS / FAIL / UNCERTAIN**
  and a notes box.

Click a rating. It is saved the moment you click, so navigating away never loses work.
**Next** saves the page and advances; **🏁 Create checkpoint** writes a timestamped snapshot.

**Congratulations page** — when every page has a decided rating for every task in scope:

- **💾 Export Final Results** writes the final TSV. The default path is derived from
  `--output_dir`; a second explicit click is required before an existing file is overwritten.

## 4. Find your results

For the demo, `--output_dir` is `./output` at the repository root.

```text
output/
├── <rater_id>_anat_wf_qc_status.tsv   # live results, one file per rater + task
└── checkpoints/
    └── ...                            # timestamped snapshots
```

The filename is `<rater_id>_<qc_task>_status.tsv`, with the rater ID lowercased and any spaces
removed (so a rater ID of `Jane Doe` and task `anat_wf_qc` gives `janedoe_anat_wf_qc_status.tsv`).
`--qc_task all` becomes `all_tasks`.

The status file is TSV and is appended to across saves, with duplicate
`(participant_id, session_id, pipeline, qc_task)` rows replaced by the most recent one. The
columns are described in [Configuration → Output](configuration.md#exported-results-tsv).

Reload or share that file from the landing page to resume a session or review someone else's
ratings.

## The other bundled demos

Each pipeline ships its own `qc.json` and a launcher script under `ui/`. Note the working
directory — the scripts in `ui/` must be run **from `ui/`**, while `fmriprep_demo.sh` runs
**from the repository root**.

| Script | Run from | Pipeline | Default `qc.json` | Default task | Sessions |
|--------|----------|----------|-------------------|--------------|----------|
| `./fmriprep_demo.sh` | repo root | `fmriprep` | `pipelines/fmriprep/qc_demo.json` | `anat_wf_qc` | `ses-01` |
| `cd ui && ./fmriprep_test.sh` | `ui/` | `fmriprep` | `pipelines/fmriprep/qc.json` | `sdc_wf_qc` | `ses-01,ses-02` |
| `cd ui && ./freesurfer_test.sh` | `ui/` | `freesurfer` | `pipelines/freesurfer/qc.json` | `anat_wf_qc` | `ses-01,ses-02` |
| `cd ui && ./qsiprep_test.sh` | `ui/` | `qsiprep` | `pipelines/qsiprep/qc.json` | `seg_brainmask_qc` | `ses-01,ses-02` |
| `cd ui && ./xcpd_test.sh` | `ui/` | `xcpd` | `pipelines/xcpd/qc.json` | `atlas_coverage_qc` | `ses-01` |
| `cd ui && ./noddireg_test.sh` | `ui/` | `noddireg` | `pipelines/noddireg/qc.json` | `noddireg_density` | `ses-01,ses-02` |
| `./ui/dwi_iqm_test.sh` | anywhere | `mriqc` | `pipelines/mriqc/dwi_iqm_test_qc.json` | `dwi_iqm_qc` | `ses-01,ses-02` |

Every script accepts two positional arguments — a path to a `qc.json` and a QC task name — and
honours `QC_TASK`, `QC_JSON`, `SESSION_LIST`, `PARTICIPANT_LIST`, and `PORT`:

```bash
# From ui/: every task in qsiprep's qc.json on one scrollable page
./qsiprep_test.sh ../pipelines/qsiprep/qc.json all

# Equivalent, via the environment
QC_TASK=all ./qsiprep_test.sh

# Free port 8501 already in use?
PORT=8502 ./fmriprep_test.sh
```

`dwi_iqm_test.sh` is the one script you can invoke from any directory — it resolves its own
location. It exercises the DWI IQM viewer with both a group TSV and a per-subject JSON sidecar
in the same run, and it requests `ses-01,ses-02` even though `sub-CMH0003` only has DWI data for
`ses-02`, so you can see how the app handles a participant missing one of the requested
sessions.

## What the demos load

`sample_data/` is a miniature BIDS dataset:

```text
sample_data/
├── bids/sub-CMH0001/ses-*/func/           # reference BOLD volumes
├── bids/sub-ED01/ses-01/                  # legacy demo subject (T1w + dwi + func)
├── derivatives/fmriprep/...                # figures + session-level anat
├── derivatives/freesurfer/...             # FreeSurfer outputs
├── derivatives/qsiprep/...                # QSIPrep figures
├── derivatives/xcpd/...                   # XCP-D figures
├── derivatives/noddireg/...               # NODDIreg outputs
├── derivatives/mriqc/...                  # MRIQC group tables + per-subject outputs
├── derivatives/fsqc/...                   # FreeSurfer QC outputs
├── qc_participants.tsv                    # sub-CMH0001  (most demos)
├── qc_participants_demo.tsv               # sub-ED01     (fmriprep_demo.sh)
└── dwi_iqm_test_participants.tsv          # sub-CMH0001/2/3 (DWI IQM demo)
```

Paths inside `qc.json` are always relative to `--dataset_dir`, and the participants to review
come from the participant list TSV. See [Configuration](configuration.md) for both formats.

## Running QC-Studio on your own data

The demos are just wrappers around this command. From the repository root:

```bash
streamlit run ui/main.py --server.port=8501 -- \
  --qc_json ../pipelines/fmriprep/qc.json \
  --qc_task sdc_wf_qc \
  --qc_pipeline fmriprep \
  --dataset_dir /path/to/your_bids_dataset \
  --participant_list /path/to/qc_participants.tsv \
  --session_list ses-01,ses-02 \
  --output_dir ./output
```

```{admonition} The qc_json flag is resolved relative to ui/, not your current directory
:class: warning

`ui/main.py` joins the value you pass onto its own directory. From the repository root, pass a
path that starts with `../`, as above. The demo scripts translate the path for you — that is
why `fmriprep_demo.sh` prints both `qc_json` and `qc_json_for_ui`. An absolute path works from
anywhere.
```

You can also use `python ui/main.py --help` to read the flag documentation, or start the
multipage app directly with `streamlit run ui/main.py` and reach `ui/pages/1_Landing_Page.py`
from the sidebar.

## Next steps

- [Configuration](configuration.md) — every flag, the `qc.json` schema, substitutions, and
  output columns.
- [Autoplay](autoplay.md) — timed auto-advance for fast-pass review of large cohorts.
- [Architecture](architecture.md) — how the code is organized, if you plan to contribute.
