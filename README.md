# Aurora Portal Uptake

Usage report on the **Aurora Universities Alliance / OpenAIRE** portals — **CONNECT** (`aurora.openaire.eu`) and **MONITOR** (`monitor.openaire.eu/dashboard/aurora`) — prepared for the Aurora2030 review committee from Matomo analytics.

**[View the dashboard](https://aurora-network-global.github.io/usage-monitor/)**

## What's in this repo

- `site/Aurora-OpenAIRE-Usage-Dashboard.html` — the report itself: a single, self-contained, dependency-free HTML page (monthly visits, depth of use, referrer channels, top pages, and geography for both portals).
- `index.html` — redirects to the dashboard above, so GitHub Pages serves it at the site root.
- `data/Export _ *.csv` — the raw Matomo exports (Channel Type, City, Country, Main metrics, Page URLs, split by portal) that the dashboard's figures were built from.

The dashboard's look matches the [Aurora Universities Alliance](https://aurora-universities.eu/) brand — Archivo Black / Manrope typography, teal/blue/navy palette.

See `CLAUDE.md` for how the dashboard is built and how to update it with new data, and `TODO.md` for planned work (dynamic CSV loading).

## License

Released under the [EUPL-1.2](LICENSE).
