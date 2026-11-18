# Changelog

All notable changes to this project will be documented in this file.

## 0.7.0

### Changed

- Switched from `openalexR` to the openalexPro ecosystem (`openalexPro`, `openalexSnowball`). The snowball search is now done with `openalexSnowball::pro_snowball()` and stored as Parquet in `data/snowball/` (replaces `data/snowball.rds` and `data/snowball_supplemented.rds`).
- DOIs are passed directly to the snowball search; the conversion to OpenAlex ids with `oa_fetch()` is not needed anymore.
- The supplemented citation network uses the `extended` edges returned by the snowball search instead of being computed from `referenced_works`.
- Plotting code is in two functions used for both networks.
- Static PNG figures are saved at 300 instead of 600 dpi.
- Re-ran the snowball search (2026-11-18); results and figures updated.

### Added

- Sections on the OpenAlex API key, the structure of the results (node roles, edge types, provenance) and the offline snowball search against an OpenAlex snapshot.
- Reproducibility as point 7 in "Things to keep in mind".
- Table with the provenance of the snowball search (date, key papers, package versions) and a note on citing the packages.
- The interactive graphs are published to GitHub Pages, so the "Open in new window" links work.
- Download links for the PDF and PNG versions of the static network graphs (also published to GitHub Pages). Saving as SVG is shown but commented out, as the files are very large.
- The OpenAlex API key can be provided to CI as the repository secret `OPENALEX_API_KEY`.

### Fixed

- Node size in the interactive graphs used a hard-coded year (2024), giving negative or infinite sizes for works published later.
- Point 2 in "Things to keep in mind": works neither cited by nor citing the key papers are missing.

## 0.5.1

### Added

Definition of **Snowball search**

### Changed

### Fixed
