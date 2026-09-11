# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-page, dependency-free analytics report: `site/Aurora-OpenAIRE-Usage-Dashboard.html`. It presents Matomo usage stats for two Aurora Universities Alliance / OpenAIRE portals — **CONNECT** (`aurora.openaire.eu`) and **MONITOR** (`monitor.openaire.eu/dashboard/aurora`) — for the Aurora2030 review committee. The raw Matomo CSV exports that the dashboard's numbers were derived from live in `data/`.

There is no package.json, build tool, linter, or test suite. "Developing" here means editing the HTML file directly and opening it in a browser.

## Layout

- `index.html` — GitHub Pages entry point; redirects to `site/Aurora-OpenAIRE-Usage-Dashboard.html`.
- `site/` — the dashboard HTML (and, once the redesign/dynamic-loading TODOs land, its assets/scripts).
- `data/` — the raw Matomo CSV exports the dashboard's figures were built from.
- `.claude/skills/` — vendored skills for follow-up work on this repo (see below).
- `TODO.md` — planned work: brand redesign, further reorg, dynamic CSV loading.

## Running / previewing

Just open the file — no server or build step required:

```
open site/Aurora-OpenAIRE-Usage-Dashboard.html   # or: xdg-open / drag into a browser
```

There is nothing to install, compile, lint, or test.

## Architecture of the dashboard file

The HTML file is entirely self-contained (one `<style>` block, one `<script>` block at the bottom). The only external resources are Google Fonts (Archivo Black, Manrope, IBM Plex Mono) — everything else, including charts, is hand-rolled.

**Brand**: matches aurora-universities.eu — Archivo Black for headings/display numbers, Manrope for body text, IBM Plex Mono kept for tabular/numeric UI (Aurora's own site has no brand mono font). Core palette: `--s1`/CONNECT teal→blue `#008dff`, `--s2`/MONITOR `#01c8b1`, ink `#0d0a46`, body text `#555371`, background `#f7f8fc`, plus `#de53ae`/`#7141f1`/`#ffa255` categorical accents. All colors live in the `:root` custom properties (see Theming below) — redo the palette there, not by hunting for hardcoded hex values in the rules.

**Data is baked in, not fetched at runtime.** The `<script>` starts with a `RAW` object literal — arrays of arrays, one entry per month/dimension row, keyed by dataset (`CONNECT`, `CONNECT_ch`, `MONITOR`, `MONITOR_ch`, `pages`, `mpage`, `CONNECT_country`, `CONNECT_city`, `MONITOR_country`, `MONITOR_city`). This was manually transcribed from the CSV exports; the page never reads the `.csv` files directly. **To refresh the dashboard with new data, you must re-export from Matomo, hand-condense the rows, and edit the `RAW` literal (and the `mpage` object, and `ALL_MONTHS` range in `monthsBetween('2023-12','2026-08')`) yourself.**

Key pieces in the script, top to bottom:
- `RAW` — the embedded dataset described above.
- `PORTALS` — the two-portal config (key, display name, URL, CSS color variable, role blurb). Iterating this array drives most sections.
- `CH` — referrer-channel labels/colors used by the channel-mix charts.
- `monthsBetween` / `ALL_MONTHS` — builds the full continuous month axis (Dec 2023–Aug 2026); months with no data become gaps, not zeros.
- `D` — `RAW` re-indexed by month per portal, with derived fields (`bounceRate`, `directShare`, `apv` = actions/visit) and an automated-traffic `flag`:
  - `'bot'` when bounce rate ≥ 85% AND direct-entry share ≥ 85% AND visits ≥ 120
  - `'crawl'` when actions-per-visit ≥ 15
  This heuristic is the basis for the "flagged month" markers throughout the UI — keep it in sync if the underlying data changes shape.
- `summarise(key)` → `S` — per-portal rollups (totals, last-12-vs-prior-12 delta, peak month, "clean" peak excluding flagged months, averages) used by the summary cards.
- Chart code is all inline SVG built with the `el(tag, attrs, text)` DOM helper — no charting library. Each chart (`drawVisits`, `drawDepth`, the channel-mix charts, sparklines via `sparkPath`) manually computes scales/ticks (`niceTicks`) and wires its own `mousemove` tooltip handler.
- Static tables (top pages, countries, cities) are rendered by string-templating `RAW` arrays into `<table>` HTML.

**Theming**: CSS variables on `:root`, redefined under `@media (prefers-color-scheme: dark)` (guarded by `:root:not([data-theme="light"])`) and again under `:root[data-theme="dark"]`. If you add colors, define them in all three places following the existing pattern.

## Important data caveat baked into the report

The dashboard explicitly calls out (in the banner near the top of the page) that across all 887 daily rows, `Unique visitors == Visits == New Visits` exactly — Matomo is not distinguishing returning visitors on these two properties. The dashboard therefore reports **visits**, not "unique visitors," and states this limitation rather than hiding it. Preserve this caveat (and re-verify it) if the underlying CSVs are refreshed.

## CSV exports in data/

The `data/Export _ *.csv` files are raw Matomo exports (Channel Type, City, Country, Main metrics, Page URLs — each split by portal) and are the source material `RAW` was built from. They are **UTF-16 encoded**; reading them with a UTF-8-assuming tool (`cat`, `grep`, naive `csv` libraries) will show garbled/spaced-out text. Decode as UTF-16 (e.g. Python `open(path, encoding='utf-16')`, or `iconv -f utf-16`) before parsing.

## Vendored skills

`.claude/skills/taste-skill/` — frontend design-taste skill (MIT, from leonxlnx/taste-skill), for the brand-matching redesign in `TODO.md` item 1. Note it's written for landing pages/portfolios, not dashboards, so apply its typography/color-calibration/redesign-audit guidance rather than its landing-page layout and motion rules.

`.claude/skills/labeling-ai-generated-content/` — EU AI Act Article 50 disclosure-labeling skill (CC0, from ubvu/wibt-communicatie).

## Open work (see TODO.md)

Only TODO item 3 (dynamic CSV loading in-browser) is still open. Note for future fetches: this sandbox's network egress policy blocks arbitrary external domains outright (confirmed even `example.com` is blocked) — getting the actual aurora-universities.eu colors/fonts for the redesign required the user to upload a saved copy of the page rather than fetching it live; the same constraint will apply to any future live-site lookups from this repo.
