---
name: publish-electra-to-github
description: Publish the latest Electra Belgium monthly reporting HTML page to GitHub Pages. Use this skill whenever the user says things like "publie le reporting sur GitHub", "push the Electra HTML to GitHub", "publish the latest Electra report to the repo", "envoie le dernier rapport sur GitHub", "update the Reporting-Project repo", "mets le rapport sur GitHub Pages", "pousse la page HTML sur GitHub", or any phrase meaning "push/publish the latest Electra HTML report to GitHub so it's viewable online". Always uses the most recent HTML file found in the "4. Webpage template" folder. The target repo is AdamPiron/Reporting-Project and the public URL is https://adampiron.github.io/Reporting-Project/.
---

# publish-electra-to-github

Publishes the latest Electra Belgium monthly HTML report from the local "4. Webpage template" folder to the GitHub repo **AdamPiron/Reporting-Project**, served as a static page via GitHub Pages.

Each run adds only the most recent HTML file (if it isn't already in the repo), so the repo accumulates all monthly reports. Each file is immediately viewable online once pushed.

## Paths

- **Source folder**: `/Users/adampiron/Desktop/VI. CLAUDE LOCAL 🤖🤖🤖/4- Applications deliverables/7- Electra/Wagon project - Electra/4. Webpage template/`
- **File pattern**: `electra-reporting-belgium-MONTH YEAR.html` (e.g. `electra-reporting-belgium-August 2026.html`)
- **Target repo**: `AdamPiron/Reporting-Project` — https://github.com/AdamPiron/Reporting-Project
- **Public URL base**: `https://adampiron.github.io/Reporting-Project/`
- **GitHub Pages**: already enabled on `main` branch root

## Filename normalization

Source filenames contain spaces (e.g. `electra-reporting-belgium-August 2026.html`). Before pushing, normalize spaces to dashes: `electra-reporting-belgium-August-2026.html`. This keeps the URL clean and avoids encoding issues.

## Workflow

Run these steps in order. Stop and surface any problem clearly if a step fails.

### 1. Find the latest HTML file

Run this Python snippet to identify the most recent HTML file:

```python
import os, re

FOLDER = "/Users/adampiron/Desktop/VI. CLAUDE LOCAL 🤖🤖🤖/4- Applications deliverables/7- Electra/Wagon project - Electra/4. Webpage template"

MONTHS = {
    "january":1,"february":2,"march":3,"april":4,"may":5,"june":6,
    "july":7,"august":8,"september":9,"october":10,"november":11,"december":12
}

def parse_date(fn):
    m = re.match(r'electra-reporting-belgium-([a-zA-Z]+)\s+(\d{4})\.html', fn, re.I)
    if m:
        mo = MONTHS.get(m.group(1).lower())
        if mo:
            return (int(m.group(2)), mo)
    return (0, 0)

files = [f for f in os.listdir(FOLDER)
         if re.match(r'electra-reporting-belgium-\w+\s+\d{4}\.html', f, re.I)]
if not files:
    raise SystemExit("No matching HTML files found in source folder.")

latest = max(files, key=parse_date)
clean_name = latest.replace(" ", "-")
print(f"Source: {latest}")
print(f"Target: {clean_name}")
```

### 2. Clone the repo to a temp directory

```bash
TMPDIR=$(mktemp -d)
gh repo clone AdamPiron/Reporting-Project "$TMPDIR/repo"
```

### 3. Check if the file is already published

If `<clean_name>` already exists in `$TMPDIR/repo/`, tell the user "Ce rapport est déjà publié : `<clean_name>`" with its public URL, clean up, and stop.

### 4. Copy the file and push

```bash
SOURCE_FOLDER="/Users/adampiron/Desktop/VI. CLAUDE LOCAL 🤖🤖🤖/4- Applications deliverables/7- Electra/Wagon project - Electra/4. Webpage template"

cp "$SOURCE_FOLDER/<latest>" "$TMPDIR/repo/<clean_name>"

git -C "$TMPDIR/repo" \
  -c user.email="pironadam@gmail.com" \
  -c user.name="AdamPiron" \
  add "<clean_name>"

git -C "$TMPDIR/repo" \
  -c user.email="pironadam@gmail.com" \
  -c user.name="AdamPiron" \
  commit -m "Add <clean_name>"

git -C "$TMPDIR/repo" push
```

### 5. Clean up and report

```bash
rm -rf "$TMPDIR"
```

Tell the user:
- Which file was published (clean filename)
- Its public URL: `https://adampiron.github.io/Reporting-Project/<clean_name>`
- The repo URL: https://github.com/AdamPiron/Reporting-Project
- Note: GitHub Pages may take 1–2 minutes to deploy the new file

## Notes

- The `gh` CLI is already authenticated as `AdamPiron` — no login step needed.
- GitHub Pages is already enabled on the `main` branch root — no setup needed.
- Never modify the source files in "4. Webpage template" — treat them as read-only.
- Never push config files, credentials, or anything other than the target `.html` file.
- The repo currently contains `electra-reporting-belgium-August-2026.html` (first file, pushed 2026-06-08).
