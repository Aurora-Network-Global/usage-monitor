# Aurora Portal Uptake

Usage report on the **Aurora Universities Alliance / OpenAIRE** portals — **CONNECT** (`aurora.openaire.eu`) and **MONITOR** (`monitor.openaire.eu/dashboard/aurora`) — prepared for the Aurora2030 review committee from Matomo analytics.

**[View the dashboard](https://aurora-network-global.github.io/usage-monitor/)**

## What's in this repo

- `site/Aurora-OpenAIRE-Usage-Dashboard-dynamic.html` — the report, live: a single, self-contained, dependency-free HTML page (monthly visits, depth of use, referrer channels, top pages, and geography for both portals) that loads the 10 CSVs itself, in the browser, via DuckDB-Wasm — nothing is pre-baked. First load takes a few seconds while the CSVs and query engine download. This is the default.
- `site/Aurora-OpenAIRE-Usage-Dashboard.html` — the same report as a static, pre-baked snapshot of the same data, kept as a frozen fallback/comparison point. Visit `index.html#static` to see it.
- `index.html` — redirects to the live dashboard above by default (`index.html#static` for the static snapshot), so GitHub Pages serves it at the site root. Either dashboard also links to the other via the "Version" link in its header.
- `data/Export _ *.csv` — the raw Matomo exports (Channel Type, City, Country, Main metrics, Page URLs, split by portal) that both dashboards' figures are built from.

The dashboard's look matches the [Aurora Universities Alliance](https://aurora-universities.eu/) brand — Archivo Black / Manrope typography, teal/blue/navy palette.

See `CLAUDE.md` for a quick orientation, `docs/data.md` and `docs/design.md` for the data pipeline and design decisions in depth, and `TODO.md` for what's still planned.

## License

Released under the [EUPL-1.2](LICENSE).
