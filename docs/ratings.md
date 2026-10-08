# Rating schemes

Each QC task in `qc.json` is rated using one of two schemes:

| Scheme | What you rate | Typical use |
|--------|---------------|-------------|
| **Single** (default) | One overall rating per QC task | Quick pass/fail screening |
| **Multi-facet** | One rating per *facet*, i.e. a named aspect of the image | Detailed QC, e.g. rating motion, ringing and cropping separately |

## Configuration schema

| Field | Description | Default |
|-------|-------------|---------|
| `type` | `"single"` or `"multi"` | `"single"` |
| `scale` | The rating options offered (shared by all facets) | `["PASS", "FAIL", "UNCERTAIN"]` |
| `facets` | The facet names to rate (used when `type` is `"multi"`) | none |

If `type` is `"multi"` but no facets are listed, the task falls back to a single rating.

### Example multi-facet configurations

The bundled `pipelines/freesurfer/qc.json` and `pipelines/fsqc/qc.json` include multi-facet tasks you can use as starting points.


```json
"FS_preproc_workflow": {
    "base_mri_image_path": "derivatives/fmriprep/[[NIPOPPY_BIDS_PARTICIPANT_ID]]/...",
    "montage_path": ["derivatives/fmriprep/[[NIPOPPY_BIDS_PARTICIPANT_ID]]/figures/..."],
    "rating": {
        "type": "multi",
        "scale": ["PASS", "FAIL", "UNCERTAIN"],
        "facets": ["Cropped", "Aliasing", "Motion", "Susceptibility", "Ringing", "Inhomogeneity"]
    }
}
```

## Multi-facet rating interface

For a multi-facet task, the **📊 Rate** section of the QC viewer has:

- one rating choice for each facet
- an **Apply same rating to all facets** option that sets every facet at once. You can then change individual facets as needed.

Each subject-session pair will also get an overall rating based on the set of facet ratings:

| Overall rating | Meaning |
|----------------|---------|
| **All-Pass** | Every facet is PASS |
| **All-Fail** | Every facet is FAIL |
| **All-Uncertain** | Every facet is UNCERTAIN |
| **Mixed** | Every facet is rated, but not all the same |

In the exported results, each facet is written as a separate row. See [Exported results](configuration.md#exported-results-tsv).

## Default rating

To speed up rating, QC-Studio allows preselecting a rating on pages you haven't rated yet.
For multi-facet tasks, every facet is preselected.

This can be configured in two places:

- **At launch**, with `--default_qc_rating` (`PASS`, `FAIL`, `UNCERTAIN` or `None`). The default is **PASS**.
- **On the landing page**, under **Default QC rating** in the sidebar. Choose `None` to leave everything unselected until you rate it.
