# QC-Studio documentation

QC-Studio is a web-based quality control (QC) application for neuroimaging data. It gives raters
one place to look at raw BIDS data, processed pipeline derivatives, and image quality metrics
(IQMs), assign a structured QC decision with optional notes, and export the result as a
tab-separated table.

It is part of the [Nipoppy](https://github.com/nipoppy/nipoppy) ecosystem: it reads the BIDS
datasets and pipeline derivatives that Nipoppy helps standardize, and scores them against
per-pipeline QC criteria.

## Start here

- **[Overview](overview.md)** — what QC-Studio does, the ideas behind it, and the vocabulary you
  need to read the rest of the documentation.
- **[Installation](installation.md)** — set up a working environment from a clean machine.
- **[Quickstart](quickstart.md)** — run a bundled demo against the sample dataset in a couple of
  commands and take your first QC ratings.

## Reference

- **[Configuration](configuration.md)** — every command-line flag, the `qc.json` schema, path
  substitutions, environment variables, in-app settings, and the shape of the exported results.
- **[Autoplay](autoplay.md)** — how timed auto-advance works, how to control it, and what it does
  with unrated pages and notes.

```{toctree}
---
caption: Overview
maxdepth: 2
titlesonly:
hidden:
includehidden:
---
overview
installation
quickstart
```

```{toctree}
---
caption: Guides
maxdepth: 2
titlesonly:
hidden:
includehidden:
---
configuration
autoplay
```

```{toctree}
---
caption: Development
maxdepth: 2
titlesonly:
hidden:
includehidden:
---
architecture
dev_plan
```
