[![DOI](https://zenodo.org/badge/DOI/99999.svg)](https://doi.org/99999)
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

- **[data](data/)**: data files created during the running of the code file and contains cached as well as final data files.
- **`figures`**: figures created during the running of the code` file in low-res as well as high-res.
- **`R`**: R scripts used to run the code in the repo. Files in this folder will be sourced initially.

## Abstract

In addition to the typical literature search using search terms,
  one can conduct a snowballing search, which is searching, starting
  from a set of identified key-papers, the publications cited in the key papers
  as well as the publictions citing the key-paper. This search is much more
  focussed on the topic of the key-papers and provides, as a ind of by-product,
  the citation network of the papers. Here we will discuss how this can be
  achieved with code examples and mention some shortcomings and abvantages of
  this approach. Furthermore, we will outline some analysis approaches and
  possibilities of a citation network without going into to moch detail

## Google Doc for Reviev

[IPBES_TG_Snowballing](https://docs.google.com/document/d/10Xrxo4wM79GyNFQDnM0sZJtGVnR1qlBdNYCeSC55Y8E/edit)

## Final Documents

- [Snowballing for Literature Search and Analysis](IPBES_TG_Snowballing.html)
