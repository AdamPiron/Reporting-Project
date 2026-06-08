---
name: update-electra-report
description: >-
  Generate the next monthly version of the Electra Belgium reporting webpage.
  Use whenever the user wants to update, build, refresh, or produce the next
  month's Electra reporting page, or says things like "update the Electra report",
  "generate the next month", "make the June/July report", "régénère la webpage
  Electra", "crée la nouvelle version du reporting Electra". Reads the structured
  markdown in the "3. Updated content" folder and writes a new HTML version into the
  "4. Webpage template" folder, advancing the month by one.
---

# Update Electra Belgium Monthly Report

This skill produces the **next monthly version** of the Electra Belgium debt-investor
reporting webpage. It takes the structured markdown content for a given month
(`3. Updated content/`) and renders it into a new self-contained HTML file
(`4. Webpage template/`), keeping the exact same design system as the previous version.

## Folder layout

Two sibling folders under `Wagon project - Electra/`:

- **`3. Updated content/`** — one markdown file per month, named
  `electra-webpage-content_<MONTH>_<YEAR>.md` (e.g. `electra-webpage-content_JUNE_2026.md`).
  Each file holds all the figures and commentary for that month.
- **`4. Webpage template/`** — the published HTML pages, named
  `electra-reporting-belgium-<MONTH> <YEAR>.html` (e.g. `electra-reporting-belgium-MAY 2026.html`).
  These are the deliverables.

The skill is typically invoked from the `3. Updated content/` directory, so
`4. Webpage template/` is reachable as `../Webpage template/`.

## Versioning — determine the target month

1. List `4. Webpage template/` and find the **latest existing** HTML version by month/year
   (parse the `<MONTH> <YEAR>` in each filename; months are calendar order).
2. The **target month = latest existing month + 1** (roll the year over after December).
   Example: if the latest page is `MAY 2026`, the target is **June 2026**.
3. Find the matching content file in `3. Updated content/`:
   `electra-webpage-content_<TARGET_MONTH>_<YEAR>.md` (month token is UPPERCASE).
   - If it is missing, stop and tell the user the content for that month isn't available yet,
     and list which content files **are** present.
4. **Always confirm the resolved target month with the user before writing** (state
   "latest page = X, so I'll generate Y from `<file>`"), unless they already named the month
   explicitly in their request.

## How to generate the page

The HTML is a single file whose entire content is driven by one JavaScript object,
`const DATA = {…}`, near the bottom (inside `<script>`). The rendering functions below
it never change. **You only ever rebuild the `DATA` object and a few header strings.**

The reliable procedure:

1. **Read the latest existing HTML** in `4. Webpage template/` — this is your structural base.
   Copy it verbatim, then edit only what changes.
2. **Read the target month's markdown** in `3. Updated content/`.
3. Replace the `DATA` object so every field reflects the markdown. Section-by-section
   mapping (markdown → `DATA`):

   | Markdown section | `DATA` field |
   | --- | --- |
   | Report meta (Period / Issued / Facility / Classification) | `meta.period`, `meta.reportDate`, `meta.facility`, `meta.confid` |
   | 01 — Overview · KPI cards | `exec.kpis` |
   | 01 — Overview · Key takeaways | `exec.bullets` |
   | 02 — Operational performance · KPI cards | `ops.kpis` |
   | 02 — Activity by cluster (table) | `ops.table` (`caption`, `head`, `rows`, `total`, `colHighlight`) |
   | 02 — Commentary | `ops.bullets` |
   | 03 — Financial performance · KPI cards | `finance.kpis` |
   | 03 — Income statement (table) | `finance.table` |
   | 03 — Commentary | `finance.bullets` |
   | 04 — Debt facility & covenants · KPI cards | `debt.kpis` |
   | 04 — Covenant monitoring (table) | `debt.table` |
   | 04 — Commentary | `debt.bullets` |
   | 05 — Next steps | `nextSteps` (each `{q, p}`) |

4. **Also update** the footer/period strings — these come from `m.period` / `m.confid`
   automatically, so editing `meta` is enough. Update the table `caption`s to the new
   month (e.g. "Activity by cluster — June 2026", "Covenant monitoring — June 2026").

### Field formatting rules (match the existing file exactly)

- **KPI cards** (`{l, v, s, t}`): `l` = label, `v` = big value, `s` = sub-line.
  Add `t:"up"` only to cards that show a positive/favourable movement or a comfortably-met
  covenant (carry over the same `t:"up"` choices the previous version used for the same KPI;
  use judgement for new ones). Plain descriptive sub-lines (e.g. "of revenue", "stable",
  "75% cap", "8% margin") get **no** `t`.
- **Table cells with deltas**: wrap positive variances in
  `<span class='var pos'>+3.9%</span>` and negative ones in
  `<span class='var neg'>−3.7%</span>`. Use the real minus sign `−` (U+2212), as in the
  source. The `total` row is the bold "Belgium total" / "EBITDA" line.
- **`colHighlight`**: keep the same highlighted column indices as the previous version
  (the ops table uses `colHighlight:[3,5]`).
- **Covenant status cells**: `<span class='pill ok'>Compliant</span>` (also Funded / On track).
  Use `<span class='pill watch'>…</span>` if a status indicates a breach or watch item.
- **Commentary bullets** (`exec/ops/finance/debt .bullets`): convert each markdown bullet to
  an HTML string. Wrap the leading emphasis phrase in `<b>…</b>` and wrap key **numeric
  figures** (rates, €, x-multiples, %, MWh, counts) in `<span class='f'>…</span>`, mirroring
  how the previous version styles its bullets. Keep bullets at headline/investor altitude.
- **`nextSteps`**: `q` = the quarter tag (e.g. "Q3 2026"), `p` = the action text.

### Critical exclusions

- **Never publish the "Editorial Note — NOT for the webpage" section.** It is internal
  context only and must not appear anywhere in the HTML.
- Do not invent figures. Every number on the page must trace to the markdown. If the
  markdown is missing a value the template expects, flag it to the user rather than guessing.
- Keep the disclaimer banners ("Illustrative template using randomized figures…") exactly
  as in the base file.

## Output

Write the new file to:

```
Webpage template/electra-reporting-belgium-<MONTH> <YEAR>.html
```

with `<MONTH>` capitalised like the existing files (e.g. `June`) — match the existing
naming/casing pattern. Do **not** overwrite or delete previous months' pages.

## Verify before finishing

After writing, re-read the new file and sanity-check:
- `meta.period`, `meta.reportDate`, and both table captions all show the new month.
- KPI values, table rows/totals, and bullet figures match the markdown 1:1 (spot-check a few).
- No "Editorial Note" content leaked in.
- The `<script>` rendering block below `DATA` is byte-for-byte identical to the base file.

Then tell the user the path of the file you created and give a one-line summary of the
month's headline figures (e.g. DSCR, stations live, EBITDA).

