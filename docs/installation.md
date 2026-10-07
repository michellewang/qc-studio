# Installation

QC-Studio is a Python application run from a source checkout. It is not published on PyPI, so
you clone the repository and install its dependencies into a virtual environment.

## Requirements

| | |
|---|---|
| **Python** | 3.10 or newer (3.10 is what CI runs; 3.12 is known to work) |
| **Git** | any recent version |
| **Browser** | required to interact with the app once it is running |
| **OS** | Linux or macOS; Windows works for the app itself, but the bundled demo scripts are Bash |

No system packages, compilers, or display servers are needed. QC-Studio reads data from
directories you already have access to, locally or over SSH — it does not run any pipeline.

## Get the code

```bash
git clone https://github.com/nipoppy/qc-studio.git
cd qc-studio
```

## Create an environment

Pick one of the two options below. Both end with an activated virtual environment at
`.venv/` in the repository root.

### Option A — uv (recommended)

```bash
uv venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
```

### Option B — pip and venv

```bash
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```

`requirements.txt` covers Streamlit, Pydantic, pandas, NumPy, Plotly, nibabel, Pillow, CairoSVG,
pyarrow, requests, and python-dotenv.

## Install the Niivue 3D viewer component

The interactive 3D MRI panel comes from a separate package that is not on PyPI yet:

```bash
pip install --index-url https://test.pypi.org/simple/ --no-deps niivue-streamlit
```

(with `uv`: `uv pip install --index-url https://test.pypi.org/simple/ --no-deps niivue-streamlit`)

`--no-deps` matters: TestPyPI would otherwise pull that package's dependency versions from
TestPyPI, which frequently fails.

```{admonition} What happens if you skip this
:class: important

QC-Studio still runs. The Niivue panel is replaced by a placeholder that tells you the
interactive viewer is unavailable, and everything else — montage, QC metrics, ratings,
export — works normally. The placeholder is defined in
[`ui/_niivue_viewer_fallback.py`](https://github.com/nipoppy/qc-studio/blob/main/ui/_niivue_viewer_fallback.py),
and the app picks it up automatically whenever `niivue_component` cannot be imported.
```

## Optional extras

### Session discovery (`pybids`)

If you omit `--session_list`, QC-Studio scans `--dataset_dir` with
[PyBIDS](https://github.com/bids-standard/pybids) to discover which sessions to review (see
[`ui/main.py`](https://github.com/nipoppy/qc-studio/blob/main/ui/main.py)).

````{admonition} Current gap
:class: warning

`pybids` is **not** listed in `requirements.txt`. If you rely on session auto-discovery you
must install it yourself:

```bash
pip install pybids
```

Passing `--session_list` explicitly avoids the import entirely and is the recommended path
today — all bundled demo scripts do this. This gap is tracked for a fix.
````

### Reference IQM data

The **Dataset + reference** mode of the IQM viewer downloads a group-level reference table
(the MRIQC-derived Parquet distributed with QC-Studio's reference host) the first time it is
used. Point QC-Studio at that host with an environment variable:

```bash
cp .env.example .env
# then edit .env and fill in REFERENCE_DATA_URL=
```

`REFERENCE_DATA_URL` is read from the process environment or from a `.env` file in the working
directory. It is not needed if you only use the local IQM sources configured in `qc.json`, or
if `.streamlit/reference_cache/` already holds the modalities you want (cached for seven days).
See [Configuration](configuration.md#environment-variables) for details.

### Development and test tooling

```bash
pip install -r requirements-test.txt
pre-commit install
```

This installs pytest, pytest-mock, pytest-cov, black, flake8, codespell, and pre-commit.
Before pushing, run:

```bash
pre-commit run --all-files
```

That runs the same formatting, linting, spelling, and UI test commands CI uses.

## Verify the installation

```bash
python ui/main.py --help
```

You should see the full list of flags (`--dataset_dir`, `--participant_list`, `--session_list`,
`--qc_pipeline`, `--qc_task`, `--output_dir`, `--qc_json`). If that works, the environment is
functional — you are ready for the [Quickstart](quickstart.md).

## Build this documentation locally

```bash
pip install -r docs/requirements.txt
make -C docs html
```

The rendered site lands in `docs/_build/html/index.html`. On Windows use
`docs\make.bat html` instead of `make`. `make -C docs clean` removes the build output.

## Upgrading

```bash
git pull
pip install -r requirements.txt
pip install --index-url https://test.pypi.org/simple/ --no-deps niivue-streamlit
```

QC-Studio writes session state into `--output_dir`, never into the repository, so pulling new
code never overwrites your results.

## Uninstall

Delete the repository checkout and the virtual environment:

```bash
rm -rf qc-studio       # includes .venv, docs/_build, and any output/ you created
rm -rf ~/.streamlit    # optional: Streamlit's own configuration directory
```

## Next steps

Proceed to the [Quickstart](quickstart.md) to run a demo, or read
[Configuration](configuration.md) first if you want to understand the flags.
