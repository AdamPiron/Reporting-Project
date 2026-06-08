---
name: electra-monthly-data
description: Generate the next month's Electra Belgium raw-data reporting file. Finds the latest electra-belgium-raw-data_<MONTH>_<YY>.md, evolves every figure coherently month-over-month (stations, charge points, energy, revenue, EBITDA, debt drawn, YTD totals, covenants, dates), and writes a new file with the next month in its name. Use when the user asks to "generate next month's data", "create the June/July/... reporting data", "roll the Electra Belgium figures forward", or update the monthly Electra reporting dataset.
---

# Electra Belgium — Monthly Data Generator

Generates plausible-but-fake monthly figures for the Electra Belgium debt-investor
report, keeping the numbers **coherent from one month to the next** (the station
count grows, energy and revenue follow, the facility draws down, YTD totals
accumulate, covenants recompute, and all dates roll forward).

## What it does

1. Scans the data directory for `electra-belgium-raw-data_<MONTH>_<YY>.md` files
   and picks the most recent one (by its `period: YYYY-MM` line).
2. Parses every figure and evolves them with a coherent model:
   - **Stations / charge points** grow by a few per month (Wallonia catching up),
     and the three clusters always sum to the Belgium total.
   - **Energy** grows ~3–8% MoM; **sessions, revenue, EBITDA** follow.
   - **Income statement** recomputes margins, budget variances, and **YTD** totals
     accumulate from the previous file.
   - **Debt**: cumulative drawn creeps up toward the €75M cap; DSCR / LLCR / gearing
     drift slightly; **covenant statuses recompute** against their thresholds.
   - **Dates**: `period` +1 month, `report_date` = the 10th of the following month,
     `exported_at` updated, and `next_service` rolls to the next 30-Jun / 31-Dec.
3. Writes a new file named with the next month, e.g. `electra-belgium-raw-data_JUNE_26.md`.

The randomness is **seeded by the target month**, so re-running for the same month
reproduces identical numbers (idempotent).

## How to run

From the directory that holds the raw-data files (the `2. MD DATA/Raw` folder):

```bash
python3 .claude/skills/electra-monthly-data/scripts/generate_next_month.py
```

Or point it at the folder explicitly:

```bash
python3 .claude/skills/electra-monthly-data/scripts/generate_next_month.py \
  --dir "/path/to/2. MD DATA/Raw"
```

Flags:
- `--dir DIR` — directory containing the raw-data files (default: current directory).
- `--force` — overwrite the target file if it already exists (otherwise it refuses,
  to avoid clobbering an existing month).

The script prints the source file, the created file, and its full path.

## After running

The generated file is the **source of truth (raw export)**. To publish a report,
its values still need to be carried into:
- `electra-reporting-belgium-data <MONTH> <YY>.md` (the human/HTML-mapped template), and
- the `DATA` object inside `electra-reporting-belgium-V14.html`.

These narrative/HTML layers are not auto-updated by this skill — only the raw dataset is.
If the user wants those refreshed too, update them to match the new raw file's figures
(month labels, KPI values, cluster table, income statement, covenant table, dates).

