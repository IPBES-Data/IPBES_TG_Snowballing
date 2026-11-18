# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is one TG repo in the IPBES TGs workspace; the parent `../CLAUDE.md` covers the shared TG structure, CI workflow and the Directory. Only what is specific to this TG is noted here.

## What this TG is

"Snowball Search for Literature Search and Analysis using OpenAlex": a single Quarto document (`TG.qmd`) showing a citation-based snowball search in R with the openalexPro ecosystem (`openalexSnowball::pro_snowball()` / `read_snowball()`, built on `openalexPro`; both from https://openalexpro.r-universe.dev, not CRAN), then static (`ggraph`/`tidygraph`) and interactive (`networkD3`) citation-network plots. The "supplemented" network is just `read_snowball(edge_type = c("core", "extended"))`. Edge direction: `from` cites `to`. Two helper functions in the document (`plot_snowball_static()`, `plot_snowball_interactive()`) draw both networks.

## Rendering

```bash
quarto render TG.qmd --to html
```

Unlike the template default, **this TG evaluates R code** (there is no `execute: eval: false` in the front matter), so rendering needs R plus the packages and, on a cold cache, an OpenAlex API key (via `openalexPro::opt_api_key()`, env var `openalexPro.api_key`, or keyring `API_openalex`) and network access. CI therefore installs R and everything `scripts/detect_r_packages.sh` finds.

## Cached results are committed

The long-running steps are guarded and read/write committed files:

- `data/snowball/`: Parquet output of `pro_snowball()` (~18 MB, mostly `nodes/` and `edges/edge_type=outside`). The search only runs if `data/snowball/edges/` is missing (written last, so it marks completion); `resume = TRUE` continues an interrupted run. With the cache present, the search is skipped; the only API calls are `pro_validate_credentials()` / `pro_rate_limit_status()` in the API-key chunk (`eval: true`, works without a key and then prints "No API key found").
- `figures/*.png|pdf`: static plots are only regenerated when the `.png` is missing (the PDF is written alongside; it needs a working `cairo_pdf` because `theme_graph()` uses Arial Narrow, which the plain `pdf` device rejects).
- `figures/*_interactive*.html`: written on every render.

To refresh results (e.g. after changing the key DOIs), delete `data/snowball/` and the figure files and re-render, then commit the new files.

The document links to `./figures/*_interactive*.html` ("Open in new window"); the CI workflow therefore also copies those files to `figures/` on `gh-pages`.

Code chunks (and plain ``` blocks) have an empty line before the opening and after the closing fence.

R package names in prose (TG.qmd and README.md) are always links of the form `` [`{igraph}`](https://r.igraph.org) ``. Not in code, and not for "the openalexPro ecosystem" (the GitHub organisation).

External links open in a new tab (`link-external-newwindow: true` in the front matter; Quarto adds `target="_blank"` via JavaScript at page load). Relative links count as internal, so links to local files that should open in a new tab need `{target="_blank"}`.

## R package detection gotcha

`detect_r_packages.sh` is a regex scan and does not skip R comments: commented-out `library(pandoc)` / `require(...)` lines in `TG.qmd` still land in `R.pkgs` and CI tries to install them (the commented-out draft block at the end of `TG.qmd` brings in `igraph`/`visNetwork`), and `pkg::` in prose counts too (`keyring`). Delete unused package references rather than commenting them out. The committed `R.pkgs` is overwritten by CI on each run.

## Differences from the template

`.github/workflows/quarto-gh-pages.yml` adds an apt step installing `libigraph-dev libglpk40 libxml2 libcurl4-openssl-dev` (for `igraph` etc.) and prepends `https://openalexpro.r-universe.dev` to the repos in the R install step, copies the interactive graphs to `public/figures/`, and passes the repo secret `OPENALEX_API_KEY` to the render as `openalexPro.api_key` (via `env`, since bash can't export dotted names; only used when the cache is missing). Keep all four if the workflow is re-synced from `IPBES_TG_TEMPLATE`.

Version history is in `NEWS.md` (matches the front-matter `version`).
