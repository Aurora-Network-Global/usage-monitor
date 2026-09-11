# Aurora Portal Uptake

Usage report on the **Aurora Universities Alliance / OpenAIRE** portals — **CONNECT** (`aurora.openaire.eu`) and **MONITOR** (`monitor.openaire.eu/dashboard/aurora`) — prepared for the Aurora2030 review committee from Matomo analytics.

**[View the dashboard](https://aurora-network-global.github.io/usage-monitor/)**

## What's in this repo

- `site/Aurora-OpenAIRE-Usage-Dashboard-dynamic.html` — the report, live: a single, self-contained, dependency-free HTML page (monthly visits, depth of use, referrer channels, top pages, and geography for both portals) that loads the 10 CSVs itself, in the browser, via DuckDB-Wasm — nothing is pre-baked. First load takes a few seconds while the CSVs and query engine download. This is the default.
- `site/Aurora-OpenAIRE-Usage-Dashboard.html` — the same report as a static, pre-baked snapshot of the same data, kept as a frozen fallback/comparison point. Visit `index.html#static` to see it.
- `index.html` — redirects to the live dashboard above by default (`index.html#static` for the static snapshot), so GitHub Pages serves it at the site root. Either dashboard also links to the other via the "Version" link in its header.
- `data/Export _ *.csv` — the raw Matomo exports (Channel Type, City, Country, Main metrics, Page URLs, split by portal) that both dashboards' figures are built from.

The dashboard's look matches the [Aurora Universities Alliance](https://aurora-universities.eu/) brand — Archivo Black / Manrope typography, teal/blue/navy palette.

## Updating the data

The live dashboard (`Aurora-OpenAIRE-Usage-Dashboard-dynamic.html`) reads the 10 CSVs in `data/` at page-load time, so refreshing it is just a file swap — no code edit, no rebuild:

1. Re-export the same 10 reports from Matomo (Main metrics, Channel Type, Page URLs, Country, City — each for CONNECT and MONITOR).
2. Rename each export to the fixed filename it replaces (Matomo's own download names embed the export's date range, which changes every time, so the loader looks for a fixed name instead — see the exact 10 names in `docs/data.md`):

   ```
   Export _ Main metrics _ CONNECT.csv        Export _ Main metrics _ MONITOR.csv
   Export _ Channel Type _ CONNECT.csv        Export _ Channel Type _ MONITOR.csv
   Export _ Page URLs _ CONNECT.csv           Export _ Page URLs _ MONITOR.csv
   Export _ Country _ CONNECT.csv             Export _ Country _ MONITOR.csv
   Export _ City _ CONNECT.csv                Export _ City _ MONITOR.csv
   ```
3. Overwrite the matching files in `data/` and push. Everything else — the date range shown, daily-row counts, all figures — recomputes on next page load.

**Only have one portal's exports** (e.g. running this for a single-portal deployment rather than Aurora's CONNECT+MONITOR pair)? Drop in just that portal's 5 files and leave the other 5 out — the dashboard detects which portal(s) have data and hides the rest, rather than erroring. See "Reusing this dashboard for a different portal (or a single one)" in `docs/data.md` for what else needs a manual edit (branding, portal names/colors, Aurora's member-institution lists) versus what just works from data alone.

The static dashboard (`Aurora-OpenAIRE-Usage-Dashboard.html`) is a frozen snapshot and doesn't read `data/` at all — refreshing it means hand-editing its `RAW` literal, per `CLAUDE.md`.

See `CLAUDE.md` for a quick orientation, `docs/data.md` and `docs/design.md` for the data pipeline and design decisions in depth, and `TODO.md` for what's still planned.

## License

Released under the [EUPL-1.2](LICENSE).
