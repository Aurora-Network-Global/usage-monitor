# Aurora Portal Uptake

Usage report on the **Aurora Universities Alliance / OpenAIRE** portals — **CONNECT** (`aurora.openaire.eu`) and **MONITOR** (`monitor.openaire.eu/dashboard/aurora`) — prepared for the Aurora2030 review committee from Matomo analytics.

**[View the dashboard](https://aurora-network-global.github.io/usage-monitor/)**

## What's in this repo

- `site/Aurora-OpenAIRE-Usage-Dashboard.html` — the report itself: a single, self-contained, dependency-free HTML page (monthly visits, depth of use, referrer channels, top pages, and geography for both portals).
- `site/Aurora-OpenAIRE-Usage-Dashboard-dynamic.html` — a work-in-progress version that loads the CSVs live in the browser (DuckDB-Wasm) instead of a pre-baked copy of the data. Visit `index.html#dynamic` to see it.
- `index.html` — redirects to the static dashboard above by default, so GitHub Pages serves it at the site root.
- `data/Export _ *.csv` — the raw Matomo exports (Channel Type, City, Country, Main metrics, Page URLs, split by portal) that the dashboard's figures were built from.

The dashboard's look matches the [Aurora Universities Alliance](https://aurora-universities.eu/) brand — Archivo Black / Manrope typography, teal/blue/navy palette.

See `CLAUDE.md` for a quick orientation, `docs/data.md` and `docs/design.md` for the data pipeline and design decisions in depth, and `TODO.md` for what's still planned.

## License

Released under the [EUPL-1.2](LICENSE).
