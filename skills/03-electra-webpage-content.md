---
name: electra-webpage-content
description: Generate the next month's Electra Belgium reporting *webpage content* as a Markdown file. Reproduces, in MD, exactly the figures and commentary the webpage shows (the five sections of the `electra-reporting-belgium-V14.html` DATA object), with every number pulled 1:1 from the monthly raw-data export and the commentary grounded in that month's Notion "Operations Report". Appends a short editorial paragraph (NOT for the webpage) explaining what was included or omitted in the comments, saves the file to the "3. Updated content" folder, then asks the user for feedback. Use when the user asks to "generate the webpage content", "update the reporting page content for June/July/...", "prepare next month's webpage MD", or roll the Electra Belgium webpage content forward.
---

# Electra Belgium — Monthly Webpage Content Generator

Produces the **content of the monthly reporting webpage** (the page rendered by
`Webpage template/electra-reporting-belgium-V14.html`) for the next month, as a
**Markdown file** the user reviews before it is hand-carried into the HTML.

The MD reproduces **exactly** what appears on the webpage — every KPI card, every
table, and every commentary bullet for all five sections — plus a final
**Editorial Note** paragraph that is *not* part of the webpage and only explains
the inclusion/omission choices for the comments.

This sits one layer above the data skills:
- [[electra-monthly-data]] rolls the **raw figures** forward into
  `electra-belgium-raw-data_<MONTH>_<YY>.md`.
- [[electra-notion-ops-report]] turns that raw file into the **Notion narrative**.
- **This skill** combines both into the **webpage content MD**: figures from the
  raw export, commentary from the Notion Operations Report.

## Alignment convention (important)

**Same-month, raw-aligned.** The webpage content for `<MONTH> <YEAR>` takes every
figure from `electra-belgium-raw-data_<MONTH>_<YY>.md` (and MoM deltas from the
previous month's raw file). The qualitative **commentary bullets** are written
from `Operations Report – <MONTH> <YEAR>` on Notion. See
[[electra-notion-report-alignment]]. **If a narrative sentence states a € / % /
count that also lives in the raw export, it must match the raw figure** — the raw
export always wins.

## Versioning

The next webpage content is **one month after the latest existing version**:
1. Look in the **"3. Updated content"** folder for the newest
   `electra-webpage-content_<MONTH>_<YEAR>.md`. Target month = that month **+ 1**.
2. If the folder has none yet (first run), baseline on the **current period of the
   HTML template** — read the `meta.period` in
   `4. Webpage template/electra-reporting-belgium-V14.html` (e.g. `May 2026`) and
   produce the **next** month (→ `June 2026`).

So: May template → first run gives June; run again → July; and so on. Do not
overwrite a month that already exists in "3. Updated content" unless the user asks.

## Key locations

- **Raw-data folder:** `…/Wagon project - Electra/2. MD DATA/Raw`
  (files `electra-belgium-raw-data_<MONTH>_<YY>.md`).
- **Webpage template:** `…/Wagon project - Electra/4. Webpage template/electra-reporting-belgium-V14.html`.
- **Output folder:** `…/Wagon project - Electra/3. Updated content`.
- **Notion parent — "Electra Operations Report":** `3757b3fd-8978-80b4-abbd-ce74b13d23b6`
  (children are the `Operations Report – <MONTH> <YEAR>` pages). Re-resolve with
  `notion-search "Electra Operations Report"` if it ever moves.

## Procedure

### 1. Resolve the target month
Apply the **Versioning** rule above to get `<MONTH> <YEAR>` (e.g. `June 2026`) and
its file token `<MONTH>_<YY>` (e.g. `JUNE_26`). If the user named a month
explicitly, use that instead.

### 2. Get the raw-aligned figures
Run the helper from the raw-data folder:

```bash
python3 .claude/skills/electra-webpage-content/scripts/build_webpage_content.py \
  --dir "/abs/path/to/2. MD DATA/Raw" --target <MONTH>_<YY>
```
e.g. `--target JUNE_26`. Omit `--target` to use the latest raw file.

The script prints one JSON blob shaped like the webpage `DATA` object:
`meta`, `exec_kpis`, `ops_kpis`, `cluster_table`, `finance_kpis`, `income_table`,
`debt_kpis`, `covenant_table`, `next_steps`, `next_service`, `dsra`. **Every
number in the MD must be copied from this JSON — never re-estimate a figure.**

- If it returns `{"error": "no raw file for …"}`, the raw export for the target
  month doesn't exist yet → run [[electra-monthly-data]] first, then retry.

### 3. Read the Notion Operations Report for that month (commentary source)
`notion-fetch` the `Operations Report – <MONTH> <YEAR>` page under the parent
above. Its qualitative sections — **Financing, Commercial, Operations & Incidents,
Construction & Pipeline, Regulatory & Risk, Watchlist** — are the raw material for
the webpage commentary bullets and for enriching Next steps.

- If that month's Notion report doesn't exist yet, either run
  [[electra-notion-ops-report]] first, or fall back to evolving the previous
  webpage content / previous Notion report. Tell the user which path you took.

### 4. Compose the webpage-content Markdown
Write the file with the layout below. The five sections mirror the HTML `DATA`
object 1:1; render KPI cards and tables from the **step-2 JSON**, and write the
commentary bullets from the **step-3 narrative**, grounded on those figures.

Commentary guidance (match the webpage's existing voice — see the template's
`bullets` arrays):
- **4–5 bullets** per section, one short sentence each, **bold the key figure**.
- Lead each section with its headline (covenant/DSCR for Overview; the strongest
  cluster for Operations; revenue/EBITDA vs budget for Finance; covenant
  compliance for Debt).
- Only use narrative items with **investor materiality**. Pull the "why" behind a
  number from the Notion report (e.g. *why* uptime moved, *which* commercial wins
  drove energy). Never introduce a figure that contradicts the raw JSON.
- **Next steps**: start from the raw `next_steps`; you may merge in 1–2 forward
  items from the Notion **Watchlist** if they add investor-relevant color, keeping
  the quarter tags.

Output layout (Markdown):

```markdown
# Electra — Belgium Monthly Report — Webpage Content
## <MONTH YEAR>

> Reproduces the exact figures and comments shown on the reporting webpage
> (`electra-reporting-belgium-V14.html`). The final **Editorial Note** is for
> internal context and must **not** be published on the webpage.

**Report meta**
- Period: <meta.period>
- Issued: <meta.reportDate>
- Facility: <meta.facility>
- Classification: <meta.confid>

---

## 01 — Overview

**KPI cards**
- <l>: **<v>** — <s>          (one line per exec_kpis entry)
- …

**Key takeaways this month**
- <bullet 1>
- … (4–5 bullets)

---

## 02 — Operational performance

**KPI cards**
- … (ops_kpis)

**<cluster_table.caption>**

| Cluster | Stations | Points | Points vs target | Energy (MWh) | Energy actual vs budget | Utilisation | Uptime |
| --- | --- | --- | --- | --- | --- | --- | --- |
| … rows … |
| **Belgium total** | … |   (render cluster_table.total as a bold total row)

**Commentary**
- … (4 bullets)

---

## 03 — Financial performance

**KPI cards**
- … (finance_kpis)

**<income_table.caption>**

| Item | Month actual | Month budget | Actual vs Budget | YTD actual |
| --- | --- | --- | --- | --- |
| … rows … |
| **EBITDA** | … |   (income_table.total as bold row)

**Commentary**
- … (4 bullets)

---

## 04 — Debt facility & covenants

**KPI cards**
- … (debt_kpis)

**<covenant_table.caption>**

| Metric | Value | Covenant | Status |
| --- | --- | --- | --- |
| … rows … |

**Commentary**
- … (4 bullets; include the next service date/amount from next_service and the DSRA figure)

---

## 05 — Next steps  ·  <upcoming quarters>

- **<q>** — <p>     (one line per next_steps entry)
- …

---

## Editorial Note — NOT for the webpage

One short paragraph (3–6 sentences) explaining the **commentary** choices:
which narrative items from the Notion report were **included** in the bullets and
why (investor materiality), and which were **omitted** (internal HR/org,
negotiation step-by-step, granular task lists, operational micro-details). Close
with the traceability line:

*Figures trace 1:1 to `<raw_files.target>` (current) and `<raw_files.prior>`
(MoM comparison); commentary drawn from "Operations Report – <MONTH> <YEAR>" on
Notion.*
```

### 5. Save the file
Write to the **"3. Updated content"** folder as
`electra-webpage-content_<MONTH>_<YEAR>.md` (e.g.
`electra-webpage-content_JUNE_2026.md`). Don't overwrite an existing month unless
the user asked.

### 6. Ask the user for feedback, then iterate
Show a brief summary (month, headline KPIs, where figures and commentary came
from) and **ask the user whether they have any comments on the MD file**. If they
do, implement the edits in place and re-confirm. If not, you're done.

## Guardrails
- **Raw export wins.** Any figure in the MD (cards, tables, or stated inside a
  bullet) must equal the step-2 JSON. Never invent or round away from it.
- **Webpage vs editorial.** Everything above the "Editorial Note" is verbatim
  webpage content; the Editorial Note is the only non-webpage block.
- **Idempotent versioning.** Don't regenerate a month that already exists in
  "3. Updated content" unless explicitly told to replace it.
- This skill **reads** the raw files, the HTML template, and Notion; it **writes
  one MD file** in "3. Updated content". It does not edit the raw files, the HTML, or
  Notion.

