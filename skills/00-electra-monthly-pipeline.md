---
name: electra-monthly-pipeline
description: >-
  Run the full Electra Belgium monthly report pipeline end-to-end, in the correct
  sequence. Generates raw data → Notion ops report → webpage content MD (with user
  feedback) → HTML webpage → published to GitHub Pages. Use when the user says
  "run the full pipeline", "do the full monthly update", "generate everything for
  June/July/...", "lance le pipeline Electra", "fais le reporting complet", or any
  phrase meaning "run all five Electra reporting steps together". Also use if the
  user says "update the Electra report from scratch" or "redo the whole monthly
  process".
---

# Electra Belgium — Full Monthly Report Pipeline

Runs the five Electra reporting skills **in sequence**, one per month, producing:

1. A new raw-data export (`2. MD DATA/Raw/`)
2. A new Notion "Operations Report" page
3. A reviewed webpage-content Markdown file (`3. Updated content/`)
4. A new HTML reporting webpage (`4. Webpage template/`)
5. The HTML file published to GitHub Pages (`AdamPiron/Reporting-Project`)

Each step depends on the previous one. The pipeline pauses after step 3 for user
feedback on the MD file, then resumes with step 4 once approved.

---

## Pipeline steps

### Step 1 — Generate raw data  ›  `electra-monthly-data`

Determine the target month (one month after the latest raw file in
`2. MD DATA/Raw/`). Run the [[electra-monthly-data]] skill to produce:

```
MD DATA/Raw/electra-belgium-raw-data_<MONTH>_<YY>.md
```

Before running, tell the user: *"Step 1/5 — Generating raw data for \<MONTH\>."*

If the file for the target month already exists, tell the user and ask whether to
**skip** this step (use the existing file) or **overwrite** it. Default: skip.

---

### Step 2 — Create Notion Operations Report  ›  `electra-notion-ops-report`

With the raw file from step 1 confirmed, run the [[electra-notion-ops-report]]
skill to create the `Operations Report – <MONTH> <YEAR>` page on Notion under the
"Electra reporting" folder.

Before running, tell the user: *"Step 2/5 — Creating the Notion Operations Report
for \<MONTH\>."*

If the Notion page for this month already exists, ask whether to **skip** or
**replace**. Default: skip.

---

### Step 3 — Generate webpage content MD  ›  `electra-webpage-content`

Run the [[electra-webpage-content]] skill to produce:

```
Updated content/electra-webpage-content_<MONTH>_<YEAR>.md
```

Before running, tell the user: *"Step 3/5 — Generating the webpage content
Markdown for \<MONTH\>."*

**This step includes a feedback loop.** After the skill writes the file it will
ask the user for comments. **Do not proceed to step 4 until the user explicitly
confirms the MD file is ready** (e.g. "looks good", "proceed", "ça marche", or
similar). If they request changes, apply them in place and re-confirm.

If the MD file for this month already exists in `3. Updated content/`, ask whether
to **skip** (use the existing file) or **regenerate**. Default: skip.

---

### Step 4 — Generate HTML webpage  ›  `update-electra-report`

Run the [[update-electra-report]] skill to render the approved MD into:

```
Webpage template/electra-reporting-belgium-<MONTH> <YEAR>.html
```

Before running, tell the user: *"Step 4/5 — Generating the HTML webpage for
\<MONTH\>."*

If an HTML file for this month already exists, ask whether to **skip** or
**overwrite**. Default: skip.

---

### Step 5 — Publish to GitHub Pages  ›  `publish-electra-to-github`

Run the [[publish-electra-to-github]] skill to push the HTML file generated in
step 4 to the **AdamPiron/Reporting-Project** repo.

Before running, tell the user: *"Step 5/5 — Publishing the HTML report for
\<MONTH\> to GitHub Pages."*

If the file is already present in the repo, tell the user and skip (no overwrite).

---

## End of pipeline

Once all five steps are done, summarise:

- The target month
- Files created (raw MD, Notion URL, content MD, HTML path)
- One-line headline (DSCR, stations live, EBITDA vs budget)
- Public URL of the published report: `https://adampiron.github.io/Reporting-Project/<clean_name>`

---

## Guardrails

- **Always confirm the target month** before starting step 1, unless the user
  named it explicitly.
- **Never skip step 3's feedback gate.** The MD must be approved before HTML
  generation starts.
- If any step fails, stop the pipeline, surface the error clearly, and tell the
  user which steps completed and which remain. Do not silently continue.

