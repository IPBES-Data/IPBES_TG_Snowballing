[![DOI](https://zenodo.org/badge/706159588.svg)](https://zenodo.org/doi/10.5281/zenodo.10257156)
[![GitHub release](https://img.shields.io/github/release/IPBES-Data/IPBES_TG_Snowballing.svg)](https://github.com/IPBES-Data/IPBES_TG_Snowballing/releases/latest)
[![GitHub commits since latest release](https://img.shields.io/github/commits-since/IPBES-Data/IPBES_TG_Snowballing/latest)](https://github.com/IPBES-Data/IPBES_TG_Snowballing/commits/main)
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

TODO: Badge for link to ICT guide!

# Snowballing for Literature Search and Analysis

This repository contains the source code for a Technical Guideline (TG).

The final version is available via the DOI above, which points to a versioned DOI, i.e. always the last version of the Technical Guideline.

## **Github Repo**: [IPBES_TG_Snowballing](https://github.com/IPBES-Data/IPBES_TG_Snowballing)

## Folders

- **`data`**: data created when running the code. `data/snowball/` contains the cached results of the snowball search (Parquet, as written by `openalexSnowball::pro_snowball()`). Delete it to re-run the search.
- **`figures`**: figures created when running the code, in low-res (png) as well as high-res (pdf), and the interactive network graphs (html).

## Abstract

In addition to the typical literature search using search terms, one can conduct a snowball search. A snowball search starts from a set of identified key-papers, and the snowball search will identify the publications cited in the key papers as well as the publications citing the key-paper. This search strategy identifies related articles using citation relationships (not keywords), building a citation network based on the key-papers. Here we will demonstrate how a snowball search can be conducted using [OpenAlex](https://OpenAlex.org) and R with the [openalexPro](https://github.com/openalexPro) ecosystem of packages, in particular [openalexSnowball](https://github.com/openalexPro/openalexSnowball), with code examples, and discuss some shortcomings and advantages of this approach. Furthermore, we will outline some analysis approaches and possibilities of a citation network without going into too much detail.

## Requirements

Rendering `TG.qmd` runs the R code. The packages of the openalexPro ecosystem are installed from r-universe:

```r
install.packages(
    "openalexSnowball",
    repos = c("https://openalexpro.r-universe.dev", "https://cloud.r-project.org")
)
```

An [OpenAlex API key](https://openalex.org) is recommended but not required. The snowball search itself is cached in `data/snowball/` and only re-run when that folder is deleted; it is small enough to also run without a key. Rendering checks the key and the remaining API budget, and shows the result in the document (without a key: "No API key found").

## Final Documents

- [Snowballing for Literature Search and Analysis](https://ipbes-data.github.io/IPBES_TG_Snowballing/)
