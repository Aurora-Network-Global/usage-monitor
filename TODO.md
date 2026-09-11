# TODO

1. **Redesign** `site/Aurora-OpenAIRE-Usage-Dashboard.html` to use the style, fonts, and colors of [aurora-universities.eu](https://aurora-universities.eu/), using the `/taste-skill` skill (`.claude/skills/taste-skill/`).
2. **Reorganize the repo** into a folder for the source data and a folder for the HTML/site files, instead of everything living at the repo root.
3. **Load the CSV data dynamically** in the HTML instead of hand-transcribing it into the `RAW` JS literal. Candidates to evaluate: WASM + DuckDB-Wasm (SQL over the CSVs in-browser), Pyodide (pandas in-browser), or an Altair/Vega-Lite chart layer on top of either. Whatever is chosen must still work as a static GitHub Pages site (no server).
