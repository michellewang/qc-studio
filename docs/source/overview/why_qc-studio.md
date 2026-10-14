# Why QC-Studio

Neuroimaging pipelines produce volumes, figures, and summary metrics for every participant and session.
Reviewing those outputs usually means opening files by hand, in one tool per output type, and keeping ratings in a spreadsheet.
That is slow and hard to reproduce.

QC-Studio aims to simplify the mechanics of this tedious process. It acknowledges that no single QC protocol fits all, and therefore offers the flexibility to define study-, datatype-, and pipeline-specific "QC tasks" (see qc.json).

It integrates "visual evidence" from the 3D volumes, 2D montages, and calculated image quality metrics for human auditing.

***QC Anything, Anywhere, All at Once!***

## Vocabulary

These terms are used throughout the documentation.

| Term | Definition |
|------|------------|
| **QC record** | One unit of review: a single participant and session. |
| **QC page** | The page that shows one QC record, with one or more QC tasks. |
| **QC task** | A named set of files to review, defined as one top-level key in `qc.json`. |
| **Pipeline** | The neuroimaging pipeline whose outputs are being reviewed. |
| **Panel** | One of the three optional views on a QC page: 3D MRI (NiiVue viewer), Montage, or QC Metrics (IQM distributions). See [Display panels](../guides/configuration.md#display-panels). |
| **Montage** | A 2D grid of images (SVG, PNG, JPG/JPEG). |
| **IQM** | Image quality metric, typically an MRIQC group-level table or a per-participant JSON sidecar, shown as a distribution plot. |
| **Rating** | A QC decision (by default **PASS**, **FAIL**, or **UNCERTAIN**), recorded per QC task per QC record. See also [Rating schemes](../guides/ratings.md). |
| **Facet** | A named aspect of the image (such as motion) or a brain region, rated separately in a multi-facet task. |
| **Checkpoint** | A timestamped snapshot of the current QC records, which can be loaded later to resume. |
| **Autoplay** | Timed auto-advance from one QC page to the next. See [Autoplay](../guides/autoplay.md). |

## Supported pipelines

Ready-made `qc.json` files are available in the [`pipelines/`](https://github.com/nipoppy/qc-studio/tree/main/pipelines)
directory for these pipelines:

- fMRIPrep
- FreeSurfer
- FreeSurfer QC (fsqc)
- QSIPrep
- XCP-D
- NODDIreg

You can also write your own `qc.json` for any other pipeline. See [Configuration](../guides/configuration.md#qc-json).
