# Design decisions

The dashboard's look was redesigned (see `TODO.md` item 1) to match the [Aurora Universities Alliance](https://aurora-universities.eu/) brand, using the `/taste-skill` skill. The brand values below were **not guessed** — this sandbox's network policy blocks fetching `aurora-universities.eu` directly, so they were read from a saved copy of the live site (its HTML plus `theme.1.css` and `wp-custom-css`) that the user uploaded, then applied.

## Typography

| Role | Font | Notes |
|---|---|---|
| Headings, display numbers (`h1`–`h3`, portal-card titles, hero stat numbers) | **Archivo Black** | Single weight (400 in the family, reads as black/900 visually). Set `font-weight:400` explicitly wherever it's used — letting the browser synthesize bold on a font that has no heavier weight looks worse than not asking for one. |
| Body text, UI labels | **Manrope** | Loaded at weights 400/500/700/800. |
| Tabular/numeric UI (`.num`, table figures, chart axis labels, mono badges) | **IBM Plex Mono** | Kept from the original design — Aurora's own site has no brand monospace font, and tabular figures genuinely benefit from a mono face (`font-variant-numeric:tabular-nums` pairs with it throughout). This is the one deliberate departure from a pure brand match, and it's a functional one. |

All three are loaded from Google Fonts in one `<link>` in `<head>`.

## Color

Every color is a CSS custom property on `:root`, redefined for dark mode in two places (`@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, and again under `:root[data-theme="dark"]` so an explicit toggle wins either direction). **Add new colors in all three places, following the existing pattern** — see `CLAUDE.md`.

| Variable | Light | Dark | Role |
|---|---|---|---|
| `--ground` | `#f7f8fc` | `#0b0920` | Page background |
| `--surface` | `#ffffff` | `#151233` | Card/panel background |
| `--surface-2` | `#eceef8` | `#1c1840` | Secondary surface (segmented control track, table stripes) |
| `--ink` | `#0d0a46` | `#f1f1fb` | Primary text, headings |
| `--ink-2` | `#555371` | `#b9b8d6` | Body text |
| `--ink-3` | `#747a96` | `#8482a8` | Muted/secondary text |
| `--line` | `#e5e9ed` | `#2b2650` | Hairline borders |
| `--line-strong` | `#c9cfe6` | `#3c356b` | Stronger borders, focus rings |
| `--accent` | `#008dff` | `#4db8ff` | Links, focus outline |
| `--s1` (CONNECT) | `#008dff` | `#4db8ff` | Aurora's blue — CONNECT series across every chart |
| `--s2` (MONITOR) | `#01c8b1` | `#22e0c4` | Aurora's teal — MONITOR series |
| `--c1`…`--c5` | `#008dff #01c8b1 #de53ae #7141f1 #ffa255` | brightened equivalents | Categorical palette for referrer channels, member/partner flags, geography bars — reuses `--s1`/`--s2` as `--c1`/`--c2` then extends with Aurora's pink/purple/orange accents |
| `--warn` / `--warn-ink` / `--warn-bg` | `#ffa255` / `#8a5200` / `#fff2e4` | brightened / `#ffd9ad` / `#3a2410` | Flagged-month markers, "mostly automated" pills, the unique-visitors banner |
| `--crit` / `--crit-bg` | `#ff4151` / `#ffe9eb` | `#ff6b78` / `#3a1418` | Reserved for error states (not currently shown anywhere) |
| `--good` | `#32d296` | `#4fe0ad` | Reserved for positive deltas (not currently shown anywhere) |

Source palette these all trace back to (from the real site's CSS): teal `#01c8b1`, blue `#008dff` / link-blue `#0077c9`, deep navy ink `#0d0a46`, body text `#555371`, muted text `#747a96`, background `#f7f8fc`, pink `#de53ae`, purple accent seen in nav hovers, orange/amber from the site's tile accents.

Dark-mode values are **not** from the brand (Aurora's site has no dark theme to copy) — they're the same hues lightened/desaturated by hand for contrast against a dark ground, keeping each series recognizable against its light-mode counterpart.

## Shape and elevation

- **Corner radius**: 12px for cards/panels (`.panel`, `.pcard`), 8–10px for smaller elements, `999px` (pill) for segmented controls, badges, and flagpills — matching Aurora's own button/pill radius.
- **Shadow** (`--shadow`): light mode uses a two-tone "neumorphic" shadow — a light highlight (`-8px -8px 16px rgba(255,255,255,.7)`) plus a dark, ink-tinted shadow (`8px 8px 20px rgba(13,10,70,.1)`, using the ink color's RGB rather than plain black) — copied from Aurora's own card style (`.uk-card-default` on the real site uses the same dual-shadow technique). Dark mode drops back to a conventional single drop-shadow; a light-on-dark highlight shadow doesn't read the same way, and Aurora's own site has no dark-mode version to match.

## Responsive

The `.portals` grid (`repeat(auto-fit,minmax(330px,1fr))`), the flex-wrapping `.meta` row, and the `overflow-x:auto` `.tablewrap` containers around every table were already written to collapse to a single column and scroll internally on a narrow screen. None of it activated on a real phone until a `<meta name="viewport" content="width=device-width, initial-scale=1">` tag was added to both dashboard files &mdash; without it, mobile browsers render the page at a virtual desktop-width viewport and scale the whole thing down, which is what actually produced the cramped, tiny-text mobile screenshot that prompted this fix. Verified at a 412px CSS viewport (Pixel 7 emulation): no horizontal page overflow, portal cards stack to one column, and wide tables scroll within their own `.tablewrap` instead of the page.

## Components

- **Segmented controls** (`.seg`) — pill-shaped container, pill-shaped buttons, active state gets `--surface` background + shadow. Used for the time-range/chart-table toggles, the CONNECT/MONITOR geography toggle, and the "All traffic / Aurora members only" filter.
- **Flagpills** (`.flagpill`) — small pill badges in IBM Plex Mono, used for "flagged" months, "mostly automated" countries, and the member (`A`) / associate-partner (`P`) markers, each recolored via `color-mix()` against the relevant `--c*` variable rather than a separate hardcoded color per badge type.
- **Collapsible banner** (`.banner`, a native `<details>`/`<summary>`) — the "unique visitors" caveat is collapsed by default so it doesn't dominate the page, but expands to the full original callout with one click (and is screenshot-friendly once open).
- **AI-disclosure strip** (`.aidisclosure`, a `<details>`/`<summary>` like `.banner`) — a small bordered strip directly under the header, visible without scrolling, per the `labeling-ai-generated-content` skill's placement guidance (perceivable at first exposure, not buried in a footer). The tag and one-line summary are always visible even collapsed, so the disclosure itself is never hidden; only the fuller explanation (how the data loads, the wait, the static/live alternative) collapses by default.
- **Collapsible section** (`.sec-collapse`) — same `<details>`/`<summary>` accordion mechanics as `.banner`, applied to a full `<section>` (currently just "Cost per visit", collapsed by default): the `.sechead` (heading + description) sits inside the `<summary>` so the section's purpose is still readable collapsed, with a chevron that rotates on open.
- **Floating table of contents** (`.toc`) — a small round button fixed to the right edge of the viewport (vertically centered), styled after Marimo notebook's floating outline. Collapsed by default; click opens a card listing every section, click a link (or Escape, or an outside click) closes it again. An `IntersectionObserver` highlights the link for whichever section is currently in view. Hidden below 900px viewport width — there is no room for a floating side panel next to single-column mobile content, and every section is one scroll away regardless.

## Assets

There are none — no logo, icon, or image is embedded anywhere in either dashboard file. The Aurora and OpenAIRE names appear as plain text (`.eyebrow`, portal titles). This was a deliberate choice, not an oversight: embedding the actual Aurora logo file would mean vendoring a trademarked brand asset and keeping it in sync, for a purely typographic/color match that doesn't need it. If a logo is wanted later, `logo_aurora_color.svg` is the file referenced on the real site's header/footer.
