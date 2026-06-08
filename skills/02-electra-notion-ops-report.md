---
name: electra-notion-ops-report
description: Generate the next month's Electra Belgium "Operations Report" page on Notion, under the "Electra reporting" folder, with every figure pulled 1:1 from the monthly raw-data export so the numbers are aligned to the raw data. Finds the latest "Operations Report – <MONTH> <YEAR>" on Notion, computes the next month, reads the matching electra-belgium-raw-data_<MONTH>_<YY>.md file, and creates the new report page. Use when the user asks to "generate the Notion operations report", "create next month's ops report on Notion", "publish the June/July/... operations report", or roll the Electra investor ops report forward.
---

# Electra Belgium — Notion Operations Report Generator

Creates the monthly **investor "Operations Report"** page on Notion (under
`Projects n8n → Electra reporting`), one per month, with the headline numbers
sourced **directly from the monthly raw-data export** so they are always aligned
to the raw data — no hand-typed figures.

It is the publishing layer that sits on top of the [[electra-monthly-data]]
skill: that skill rolls the **raw figures** forward into a new
`electra-belgium-raw-data_<MONTH>_<YY>.md` file; this skill turns the latest such
file into a polished Notion report.

## Alignment convention (important)

**Same-month alignment.** The report titled `Operations Report – <MONTH> <YEAR>`
takes its figures from `electra-belgium-raw-data_<MONTH>_<YY>.md`, and the
comparison column from the **previous month's** raw file. Both columns are
therefore traceable 1:1 to a raw export.

- Report **June 2026** → current column from `…_JUNE_26.md`, prior column from `…_MAY_26.md`.
- Numbers that do **not** exist in the raw export (e.g. NPS, qualitative
  narrative) are evolved from the previous Notion report, never invented to
  contradict the raw data.

## Versioning

The next report is **one month after the latest existing Operations Report on
Notion**. If the newest page is `Operations Report – May 2026`, this skill
produces `Operations Report – June 2026`; run it again next time and it produces
July, and so on. It refuses to overwrite a month that already exists (unless the
user explicitly asks to replace it).

## Known Notion location

- **Parent folder page** — "Electra reporting": `3757b3fd-8978-80b4-abbd-ce74b13d23b6`
  (under "Projects n8n"). New report pages are created with this as `page_id` parent.
- Re-resolve it if it ever moves: `notion-search` for `"Electra reporting"` and
  pick the page whose children are the `Operations Report – …` pages.

## Procedure

### 1. Find the latest report and compute the next month
- `notion-fetch` the "Electra reporting" parent page (ID above). Read its child
  pages, find every `Operations Report – <MONTH> <YEAR>`, and pick the most
  recent by date. The target month = that month **+ 1**.
- (Equivalently, ask the user which month if they specified one.)

### 2. Get the raw-aligned figures
Run the helper from the raw-data folder (`2. MD DATA/Raw`):

```bash
python3 .claude/skills/electra-notion-ops-report/scripts/build_report_data.py \
  --dir "/path/to/2. MD DATA/Raw" --target <MONTH>_<YY>
```
e.g. `--target JUNE_26`. Omit `--target` to use the latest raw file.

The script prints one JSON blob with `target` (the month), `prior` (the
comparison month), and `deltas` (MoM %). **Every numeric figure in the report
must come from this JSON** — copy values, do not re-estimate them.

- If the script returns `{"error": "no raw file for …"}`, the raw export for the
  target month doesn't exist yet → run the [[electra-monthly-data]] skill first
  to generate it, then retry.

### 3. Read the previous report for narrative continuity
`notion-fetch` the previous `Operations Report – …` page. The **qualitative
sections** (Financing, Commercial, Operations & Incidents, Construction &
Pipeline, Regulatory & Risk, Watchlist) carry a storyline forward — resolve last
month's open items, advance pipeline sites one stage, roll all dates into the new
month, and update the watchlist. If a `Call transcripts - <MONTH> <YEAR>` page
exists under "Electra reporting", skim it to ground the narrative in that month's
actual events; otherwise evolve the prior report plausibly.

### 4. Compose the report (Markdown for Notion)
Mirror the structure of the existing report. Use the JSON figures verbatim:

- **Title line:** `**Investor Operations Report — <MONTH> <YEAR>**` then `---`
- **## Financial Snapshot (<MONTH> <YEAR>)** — a table with columns
  `Metric | <MONTH> <YEAR> | <PRIOR MONTH>`. Rows (all from JSON):
  - Revenue → `target.revenue_eur_k` vs `prior.revenue_eur_k` (add `(+X% MoM)` from `deltas.revenue_meur_mom_pct`)
  - Gross Margin → `gross_margin_pct`
  - Energy Cost Ratio → `energy_cost_ratio_pct`
  - Network Uptime → `fleet_uptime_pct`
  - NPS → not in raw; carry/evolve from the previous report (clearly a soft KPI)
- **## Network & Operations KPIs** — a short table or bullets, all from JSON:
  stations_live (`+deltas.stations_added`), charge_points (`+charge_points_added`),
  energy_delivered_mwh (`+deltas.energy_delivered_mwh_mom_pct`), charge_sessions,
  utilisation_pct, ebitda_month_meur (`ebitda_margin_pct`), dscr,
  facility_drawn_meur / facility_total_meur (`facility_drawn_pct`).
- **## Financing**, **## Commercial**, **## Operations & Incidents**,
  **## Construction & Pipeline** (table), **## Regulatory & Risk** — narrative,
  evolved per step 3. Any € / % / count stated here that also lives in the raw
  data must match the JSON.
- **## <NEXT MONTH> Watchlist** — use `next_month_label` from the JSON; a numbered
  list of the month's key forward items.
- **## Editorial Note** — keep the existing "Included / Omitted" framing,
  refreshed for this month.

Follow the Notion-flavored Markdown spec (fetch `notion://docs/enhanced-markdown-spec`
if unsure about table syntax).

### 5. Create the page on Notion
Use `notion-create-pages` with:
- `parent`: `{ "type": "page_id", "page_id": "3757b3fd-8978-80b4-abbd-ce74b13d23b6" }`
- `icon`: `📊`
- `properties`: `{ "title": "Operations Report – <MONTH> <YEAR>" }`
- `content`: the Markdown from step 4 (do **not** repeat the title at the top).

### 6. Report back
Give the user the new page URL and a one-line summary (month, revenue, stations,
DSCR), and note that all figures trace to `electra-belgium-raw-data_<MONTH>_<YY>.md`.

## Guardrails
- **Never contradict the raw data.** If a narrative figure and the raw export
  disagree, the raw export wins.
- **Idempotent versioning:** don't create a month that already exists on Notion
  unless explicitly told to replace it.
- This skill **reads** the raw files and **writes one Notion page**; it does not
  modify the raw-data files or the HTML report. To roll the raw figures forward,
  use [[electra-monthly-data]] first.

