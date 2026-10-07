# Overview

QC-Studio is a web-based quality control (QC) application for neuroimaging data. It gives raters
one place to look at raw BIDS data, processed pipeline derivatives, and image quality metrics
(IQMs), assign a structured QC decision with optional notes, and export the result as a
tab-separated table.

![QC-Studio design overview](https://raw.githubusercontent.com/nipoppy/qc-studio/main/assets/nipoppy-qc-studio_overview.jpg)

## Why it exists

Neuroimaging pipelines such as fMRIPrep, FreeSurfer, QSIPrep, XCP-D, and MRIQC produce
volumes, figures, and summary metrics for every subject and session. Reviewing those outputs
usually means opening files by hand, in one tool per output type, and keeping ratings in a
spreadsheet. That is slow, hard to reproduce, and it loses the rater metadata (experience,
fatigue, screen size) that QC studies need in order to be meaningful.

QC-Studio collapses that loop into a single Streamlit application:

1. A **configuration file** (`qc.json`) says which files belong to a QC task.
2. The app loads the cohort you ask for and walks you through it one **page** at a time.
3. You assign **PASS / FAIL / UNCERTAIN**, optionally add notes, and move on.
4. Results are written to a **TSV** with the rater metadata attached.

## Who uses it

| Role | What they do in QC-Studio |
|------|---------------------------|
| **Rater** | Reviews cohort pages, assigns ratings and notes, checkpoints progress |
| **Pipeline / QC supervisor** | Writes `qc.json` for the pipeline being reviewed, picks the cohort, exports and aggregates results |
| **Analyst** | Consumes the exported TSV downstream (agreement analysis, pipeline QC reports) |

## How a session works

```text
CLI flags  →  Landing page  →  QC viewer  →  Congratulations page
              rater form        rate/notes      export final results
              panel + montage   next / auto
              resume upload     checkpoint
```

1. **Launch.** You start the app with command-line flags that point at a dataset, a participant
   list, a pipeline name, a QC task, and a `qc.json`. See [Configuration](configuration.md).
2. **Landing page.** You identify yourself as a rater, choose which panels to display, set the
   montage grid, and optionally upload a previously saved results file to resume.
3. **QC viewer.** Each page covers one **(participant, session)** pair. You rate the task
   displayed for that page, add notes if needed, and advance with **Next**, or let
   [Autoplay](autoplay.md) advance for you.
4. **Progress is saved** to `<output_dir>/<rater>_<task>_status.tsv` as you go, with
   timestamped checkpoints under `<output_dir>/checkpoints/`.
5. **Congratulations page.** When the cohort is complete you can export the final results to a
   path of your choice.

## Vocabulary

These terms are used throughout the documentation.

| Term | Definition |
|------|------------|
| **Cohort page** | One review unit: a single (participant, session) pair. The app builds the page list from your participant list crossed with `--session_list`. |
| **QC task** | One key in `qc.json` (for example `anat_wf_qc` or `sdc_wf_qc`) mapping to the files displayed for that task. `--qc_task all` shows every task from `qc.json` on one page. |
| **Pipeline** | The neuroimaging pipeline whose outputs are being reviewed; it names the `qc.json` family and is stamped into the exported `pipeline` column. |
| **Panel** | One of the three optional views on the QC viewer: the Niivue 3D MRI viewer, the Montage, or QC Metrics (IQM distributions). |
| **Montage** | A 2D grid of images (SVG, PNG, JPG/JPEG) such as fMRIPrep figures, laid out automatically or to a fixed maximum number of rows and columns. |
| **IQM** | Image quality metric, typically an MRIQC group-level table or a per-subject JSON sidecar, shown as a distribution plot. |
| **Rating** | One of **PASS**, **FAIL**, or **UNCERTAIN**, recorded per QC task per cohort page. |
| **Checkpoint** | A timestamped snapshot of the current QC records under `<output_dir>/checkpoints/`. |
| **Autoplay** | Timed auto-advance through the cohort. See [Autoplay](autoplay.md). |

## Panels

Panels are chosen on the landing page, and at least one must be selected.

| Panel | Default | Contents |
|-------|---------|----------|
| 🧠 **3D MRI Viewer (Niivue)** | on | Interactive 3D rendering of a base NIfTI (and optional overlay), with view mode, colormap, and opacity controls |
| 📊 **Montage** | on | 2D image montage for the task, either full-width or alongside the viewer |
| 📈 **QC Metrics** | off | IQM distribution plots and a per-subject metrics table |

Constraints worth knowing up front:

- The Niivue panel supports **one base image and one overlay** at a time.
- The Montage panel supports **2D image files only**: SVG, PNG, JPG/JPEG. HTML figures are not
  supported.
- Only **PASS | FAIL | UNCERTAIN** ratings are supported; there is no numeric scale.
- 4D volumes are too large for the browser in many cases, so QC-Studio may show only the first
  BOLD volume.

## Supported pipelines

Ready-made `qc.json` files ship in the repository under [`pipelines/`](https://github.com/nipoppy/qc-studio/tree/main/pipelines):

| Pipeline | Directory | Notes |
|----------|-----------|-------|
| fMRIPrep | `pipelines/fmriprep/` | Anatomical, SDC, and coregistration tasks; `qc_demo.json` for the legacy demo |
| FreeSurfer | `pipelines/freesurfer/` | Single anatomical task reading fMRIPrep derivative paths |
| QSIPrep | `pipelines/qsiprep/` | Four tasks (`seg_brainmask_qc`, `t1_2_mni_qc`, `sdc_wf_qc`, `coreg_wf_qc`) |
| XCP-D | `pipelines/xcpd/` | Three tasks (`atlas_coverage_qc`, `coreg_wf_qc`, `denoised_bold_qc`) |
| NODDIreg | `pipelines/noddireg/` | Density and OD/ICVF/ISOVF tasks |
| MRIQC | `pipelines/mriqc/` | Test configuration for the DWI IQM viewer |

Each pipeline also has its own README describing its `qc.json` and its links to upstream
pass/fail criteria. QC-Studio itself does not define the scientific criteria — those come from
the pipeline's QC guidelines.

## Where the data comes from

QC-Studio reads data you already have access to, locally or over SSH. It does not run pipelines.

- **`--dataset_dir`** — a BIDS dataset root (or a flat directory root for project-specific
  layouts). All relative paths inside `qc.json` are resolved against this.
- **`--participant_list`** — a TSV listing the participants to review, with an optional
  `session_id` column.
- **`--qc_json`** — the task configuration file described above.
- **`--output_dir`** — where session state, checkpoints, and results are written.

The bundled [`sample_data/`](https://github.com/nipoppy/qc-studio/tree/main/sample_data)
directory is a small BIDS dataset plus fMRIPrep / FreeSurfer / QSIPrep / XCP-D / NODDIreg
derivatives, so every demo can run without any external data.

## Related projects

- [Nipoppy](https://github.com/nipoppy/nipoppy) — standardized organization and processing of
  neuroimaging-clinical datasets.
- [NiiVue](https://github.com/niivue/niivue) — the 3D medical image viewer behind the Niivue
  panel.
- [Streamlit](https://streamlit.io/) — the Python web app framework QC-Studio is built on.
- [MRIQC](https://github.com/nipreps/mriqc) — the usual source of image quality metrics.

## Next steps

Continue with [Installation](installation.md), or jump straight to the
[Quickstart](quickstart.md) if you already have an environment.
