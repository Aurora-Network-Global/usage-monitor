# Data pipeline

Two versions of the dashboard live in `site/`:

- **`Aurora-OpenAIRE-Usage-Dashboard-dynamic.html`** (the default, linked from `index.html`) — has no `RAW` literal. A `loadRAW()` function fetches the 10 CSVs from `../data/` at page-load time, loads them into an in-browser DuckDB (via DuckDB-Wasm), runs the SQL below, and assembles a `RAW` object with the **exact same shape** the static file hardcodes. Everything downstream of that point — every chart, table, and computed stat — is identical code between the two files; only how `RAW` gets populated differs. Confirmed working against the live GitHub Pages deployment (see the jsDelivr/`apache-arrow` quirk below, which blocked it until fixed).
- **`Aurora-OpenAIRE-Usage-Dashboard.html`** (static snapshot, `index.html#static`) — the 10 CSVs were read once, hand-aggregated, and the result baked into a `RAW` JS object literal near the top of the `<script>` block. Nothing is fetched at runtime. Kept as a frozen fallback/comparison point.

This split means the rest of this document (how `RAW`'s fields map to variables and charts) applies to both files equally.

## Source files → RAW keys → SQL → what renders

All 10 files are UTF-16 encoded (see the root `CLAUDE.md`). The dynamic loader decodes each with `TextDecoder('utf-16le'|'utf-16be')` (BOM-sniffed) before handing the text to DuckDB.

Filenames are **fixed strings**, not a pattern — Matomo's own default export name embeds the report's date range (`Export _ Main metrics_CONNECT_Aurora _ January 1, 2024 – August 31, 2026.csv`), which changes every time you re-export, so the loader can't just glob for "a CONNECT main-metrics file." Every file below has had that variable part stripped from its name on disk; refreshing data means **overwriting these exact 10 filenames**, not adding new ones. There is no manifest and no directory listing involved — the loader tries each exact name in turn.

| CSV (in `data/`) | DuckDB table | `RAW` key | Populates | Renders in |
|---|---|---|---|---|
| `Export _ Main metrics _ CONNECT.csv` | `mm_connect` | `CONNECT` | `D.CONNECT` (via `summarise()`) | Portal summary card, "Monthly visits" chart/table, "Depth of use" chart, notes |
| `Export _ Main metrics _ MONITOR.csv` | `mm_monitor` | `MONITOR` | `D.MONITOR` | same, MONITOR side |
| `Export _ Channel Type _ CONNECT.csv` | `ch_connect` | `CONNECT_ch` | `D.CONNECT[month].ch` | "How visitors arrive" — CONNECT chart |
| `Export _ Channel Type _ MONITOR.csv` | `ch_monitor` | `MONITOR_ch` | `D.MONITOR[month].ch` | "How visitors arrive" — MONITOR chart |
| `Export _ Page URLs _ CONNECT.csv` | `pages_connect` | `pages` | — | "What people land on — CONNECT" table |
| `Export _ Page URLs _ MONITOR.csv` | `pages_monitor` | `mpage` | — | "What people land on — MONITOR" panel (single row: MONITOR only has one page) |
| `Export _ Country _ CONNECT.csv` | `country_connect` | `CONNECT_country` | — | Alliance reach split, Aurora member/partner tables, Top countries |
| `Export _ Country _ MONITOR.csv` | `country_monitor` | `MONITOR_country` | — | same, MONITOR side |
| `Export _ City _ CONNECT.csv` | `city_connect` | `CONNECT_city` | — | Top cities |
| `Export _ City _ MONITOR.csv` | `city_monitor` | `MONITOR_city` | — | same, MONITOR side |

### Any file may simply be absent

Each of the 10 files above is loaded independently: `loadTable()` treats a 404 as "this dataset isn't available," not an error, and every downstream SQL query only runs for a table that actually loaded (falling back to an empty result otherwise). Concretely:

- **A whole portal can be missing.** Drop only the 5 CONNECT files into `data/` and leave the 5 MONITOR ones out entirely — `PORTALS` (near the top of `main()`) is filtered down to `PORTALS_ALL.filter(p => RAW[p.key] && RAW[p.key].length)`, so every chart, table, and summary card that iterates `PORTALS` (which is nearly all of them) just renders for the one portal that exists. The pieces that don't naturally iterate `PORTALS` — the "How visitors arrive" duo, the "What people land on" sections, the CONNECT/MONITOR geography toggle — have their missing-portal half explicitly hidden (`main()`, right after the `PORTALS` filter) instead of rendering an empty or zeroed card, since a zeroed MONITOR card would misleadingly read as "MONITOR really did have 0 visits" rather than "MONITOR wasn't loaded at all."
- **A single dataset for an otherwise-present portal can be missing too** (e.g. you have CONNECT's main metrics but not its Page URLs export) — that one table/section degrades to an empty result (an empty table, a skipped note) rather than throwing.
- **At least one Main metrics file (CONNECT's or MONITOR's) must be present** — `loadRAW()` throws a clear error naming both expected filenames if neither loads, since there'd be no date range to build the page around otherwise.
- Verified with a CONNECT-only fixture through the full rendering pipeline (portal cards, all charts, geography, notes, cost benchmark) — no console errors, no leftover "MONITOR" text, no broken zero/NaN values.

### Exact column mapping (verified row-for-row against the raw CSVs)

These were reverse-engineered by diffing computed output against the static page's `RAW` literal until every dataset matched exactly — some are *not* what the column names next to them would suggest:

- **`CONNECT`/`MONITOR`** — one row per month: `[month, SUM(Visits), SUM(Actions), SUM(Bounces), SUM(Pageviews), SUM("Unique Pageviews"), SUM("Total time spent by visitors (in seconds)"), COUNT(*)]`. The last field (`days`) is the number of daily rows Matomo exported for that month, **not** calendar days — a month with no traffic on some days is simply missing those rows.
- **`CONNECT_ch`/`MONITOR_ch`** — one row per month: `[month, visits where Label='Direct Entry', 'Search Engines', 'Websites', 'Social Networks', 'Campaigns']` (fixed order, matches the `CH` array in the script).
- **`pages`** — one row per URL, whole period: `[Label, "Unique Pageviews", Pageviews, Entrances, Exits, "Bounce Rate", "Avg. time on page"]`. The 4th field is **Exits**, not "Actions after entering here" — easy to get wrong, since both exist as separate columns in the CSV and are close together.
- **`mpage`** — same shape as one `pages` row, but as an object (`{label, upv, pv, entrances, bounce, avgtime}`) since MONITOR's Page URLs export has exactly one row (`dashboard`).
- **`CONNECT_country`/`MONITOR_country`** — one row per country: `[Label, "Metadata: code", Visits, Actions, Bounces, "Total time spent by visitors (in seconds)"]`.
- **`CONNECT_city`/`MONITOR_city`** — one row per city: `["Metadata: city_name", "Metadata: country_name", "Metadata: country", Visits, Actions, Bounces]`. The CSV's own `Label` column (e.g. `"Créteil, Île-de-France, France"`) is not used directly — the already-split metadata columns are cleaner.

All tables are sorted by their primary count descending, with source-row order as an explicit tie-break (`row_number() OVER ()` captured at load time, ordered ascending) — plain `ORDER BY count DESC` alone reorders tied rows non-deterministically and does not reproduce the original order.

### Two things the dynamic version derives instead of hand-typing

The static page hand-types a few facts about the data (its date range, and per-portal row counts) directly into prose. The dynamic version computes them instead, from the loaded rows:

- **Period / month axis** (`ALL_MONTHS`) — the header's "Period" field and the chart's x-axis range come from `min`/`max` of the month keys across `CONNECT` and `MONITOR`, not a hardcoded `'2023-12'`/`'2026-08'`.
- **Daily-row counts** (the "In all N daily rows..." banner, and "Across X CONNECT and Y MONITOR daily rows" in the notes) — summed from each row's `days` field (see above), not typed by hand. This surfaced a real bug: the static page's notes said "333 MONITOR daily rows," but the actual file has 569 (887 total should be 1,123) — fixed in both files once found.

Per-portal start dates (each portal's own "data starts …" in the notes, and the "visits, … – …" range on each portal card) are derived the same way in **both** files — from `Object.keys(D[p.key]).sort()[0]` and `ALL_MONTHS[ALL_MONTHS.length-1]`, not hand-typed prose — so a future data refresh can't leave a stale specific date behind the way the original static page's notes once hardcoded "MONITOR data starts 1 Dec 2023, CONNECT data starts 25 Apr 2024" independently of its own `RAW` literal. The trade-off: this is month-level precision (from the monthly-aggregated `D`), not the exact day Matomo's daily export would give you.

## Known CSV quirks that affect the query, not just the numbers

Found while getting the dynamic loader's output to match the static `RAW` literal exactly (verified with a local DuckDB against the real files, not guessed):

- **`null_padding: true` is required.** Some Main metrics rows omit trailing optional columns entirely (rather than leaving them empty), which otherwise breaks DuckDB's delimiter sniffing for that file — without it, the whole header is read back as a single column.
- **"Avg. time on page" needs an explicit `VARCHAR` type override.** Its values look like clock times (`"00:00:53"`), so DuckDB's auto-detection reads the column as `TIME` instead of display text, which breaks once a value falls outside `TIME`'s range.
- Numeric aggregates missing due to `null_padding` (a genuinely absent trailing column) are coalesced to `0` — matching what the static page's original numbers assumed absence meant.
- **`dist/duckdb-browser.mjs` needs a `/+esm` suffix (or an import map) on jsDelivr.** That file is not fully bundled — it contains its own bare `import ... from "apache-arrow"`, which a plain unbundled `import()` in the browser can't resolve ("Failed to resolve module specifier"). Confirmed by installing the real published package locally and grepping the shipped file. jsDelivr's `/+esm` combine mode rewrites nested bare imports to resolvable CDN URLs; an `<script type="importmap">` mapping `apache-arrow` to its own jsDelivr `/+esm` URL is kept alongside it as a fallback.

## Cost-per-visit benchmark

The "Cost per visit" section doesn't come from the CSVs at all — `ANNUAL_COST_EUR` is a constant supplied by the Aurora office (currently €10,000), divided by `PORTALS.reduce((s,p)=>s+S[p.key].l, 0)` (last-12-month visits summed across whichever portals are active, already computed by `summarise()`). The three comparison figures in that section are cited industry reference points, not derived from this dataset — see the section's own "Sources" line. `ANNUAL_COST_EUR` itself is deployment-specific (it's what the Aurora office is actually billed for the CONNECT+MONITOR bundle) — a single-portal or differently-scoped deployment needs its own number here.

## Refreshing the data

- **Static page**: re-export the 10 CSVs, then hand-update the `RAW` literal and `ALL_MONTHS` range (see root `CLAUDE.md`).
- **Dynamic page**: overwrite the same 10 filenames in `data/` with the new export — everything else (date range, row counts, per-portal totals) recomputes automatically on next page load. No code edit needed. (If your Matomo export downloads with its own date range baked into the filename, rename it to the fixed name in the table above before dropping it in — see the root `CLAUDE.md`'s "Running / previewing" section.)

## Reusing this dashboard for a different portal (or a single one)

The rendering pipeline (charts, tables, summary cards, notes, the geography section) is written against `PORTALS`/`RAW[key]` generically and was verified to work with just one portal's data present (see "Any file may simply be absent" above) — so pointing this at, say, a single research portal instead of Aurora's CONNECT+MONITOR pair mostly works by just supplying that one portal's 5 CSVs and leaving the other 5 out. That said, several things in both dashboard files are still specific to *this* Aurora CONNECT/MONITOR deployment and need a manual edit for any other reuse, not a data change:

- **Branding and positioning copy** — the `<title>`, `<h1>`, `.eyebrow`, and the header's lede paragraph explicitly name "Aurora Portal Uptake," "Aurora Universities Alliance," "CONNECT," and "MONITOR." A few section captions also say "every portal tracked here" generically, but the ones naming the portals by name do not rewrite themselves.
- **`PORTALS_ALL`** (`main()`, near the top) — the display name, URL, brand color, and role blurb for each portal are hardcoded here; a different portal needs its own entry.
- **`AURORA`, `PARTNERS`, `AURORA_CITY`, `OPSITE`, `DCNAMES`** (geography section) — Aurora's specific member/associate-partner universities and OpenAIRE's own operating cities. These drive the "Alliance reach" split, the member/partner tables, and the "flag automated-traffic-looking cities" logic. A different deployment's notion of "our own institutions" needs its own lookup here.
- **Two notes in "Notes and caveats"** are marked in a code comment as Aurora-specific: the one that flags whether each portal's top city is an Aurora member vs. OpenAIRE's own operator (built from the `AURORA_CITY`/`OPSITE` lookups above), and the general framing throughout assumes this alliance's specific membership. The rest of the notes are built generically from `PORTALS`/`S`/`D` and read correctly for any portal count without editing.
- **`ANNUAL_COST_EUR`** (cost-per-visit section) — see above.
