---
name: send-electra-report
description: Send the latest Electra Belgium monthly reporting webpage as a PDF email attachment to a predefined recipient list. Use this skill whenever the user says things like "send the report by email", "email the Electra report", "send the latest Electra webpage", "distribute the monthly report", "envoie le rapport par email", or any phrase meaning "email the latest HTML reporting file from the Electra Wagon project folder". Always uses the most recent HTML file found in the reports folder, rendered to PDF.
---

# send-electra-report

Sends the latest Electra Belgium monthly report by email, rendered to a **PDF attachment**, to a predefined recipient list.

- **Reports folder**: `/Users/adampiron/Desktop/VI. CLAUDE LOCAL 🤖🤖🤖/4- Applications deliverables/7- Electra/Wagon project - Electra/4. Webpage template/`
- **File pattern**: `electra-reporting-belgium-*.html`
- **Config file**: `~/.claude/skills-data/send-electra-report/config.json`
- **Recipient list** (production): *(add final recipients once confirmed)*
- **Test recipient**: `pironadam@gmail.com`

## Why PDF (important)

The HTML report renders its content with **JavaScript** (a `DATA` object → `innerHTML`) and styles everything with **CSS custom properties** (`var(--slate-70)`). Email clients strip `<script>` and don't support CSS variables, so sending the raw or even pre-rendered HTML in the email body shows up **blank or broken**. The reliable solution is to render the page in a headless browser (Playwright/Chromium) to a PDF and attach that. The PDF is a faithful, self-contained snapshot of the report — no images, no scripts, no broken styles.

## Prerequisites

Playwright + Chromium must be installed (one-time):

```bash
python3 -c "import playwright" 2>/dev/null || pip3 install playwright --quiet
python3 -m playwright install chromium
```

## Workflow

Run these steps in order. Stop and surface any problem clearly if a step fails.

### 1. Load credentials

Read the config file at `~/.claude/skills-data/send-electra-report/config.json`. If it doesn't exist, jump to **Credentials setup** below.

Expected structure:
```json
{
  "sender_email": "pironadam@gmail.com",
  "app_password": "xxxx xxxx xxxx xxxx",
  "recipients": ["pironadam@gmail.com"]
}
```

### 2. Render the latest report to PDF and send

This single script: finds the most recent `electra-reporting-belgium-*.html`, renders it to PDF with Playwright, and emails it as an attachment via Gmail SMTP.

```python
import smtplib, json, os, re
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.application import MIMEApplication
from datetime import datetime
from playwright.sync_api import sync_playwright

CONFIG_PATH = os.path.expanduser("~/.claude/skills-data/send-electra-report/config.json")
with open(CONFIG_PATH) as f:
    config = json.load(f)

FOLDER = "/Users/adampiron/Desktop/VI. CLAUDE LOCAL 🤖🤖🤖/4- Applications deliverables/7- Electra/Wagon project - Electra/4. Webpage template"

MONTHS = {"january":1,"february":2,"march":3,"april":4,"may":5,"june":6,
          "july":7,"august":8,"september":9,"october":10,"november":11,"december":12}

def parse_date(fn):
    m = re.search(r'electra-reporting-belgium-(\w+)\s+(\d{4})\.html', fn, re.I)
    if m:
        mo = MONTHS.get(m.group(1).lower())
        if mo: return datetime(int(m.group(2)), mo, 1)
    return datetime.min

files = sorted([f for f in os.listdir(FOLDER)
                if f.startswith("electra-reporting-belgium-") and f.endswith(".html")],
               key=parse_date, reverse=True)
if not files:
    raise SystemExit("No electra-reporting-belgium-*.html files found in folder.")
latest = files[0]
period = re.search(r'electra-reporting-belgium-(.*?)\.html', latest, re.I).group(1).title()
html_path = os.path.join(FOLDER, latest)
pdf_name = f"Electra-Belgium-Report-{period.replace(' ','-')}.pdf"
pdf_path = os.path.join("/tmp", pdf_name)

print(f"Rendering PDF: {latest}")
with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(viewport={"width": 1100, "height": 900})
    page.goto(f"file://{html_path}", wait_until="networkidle")
    page.wait_for_timeout(2500)            # let JS finish rendering
    page.emulate_media(media="screen")     # keep the on-screen design
    page.pdf(path=pdf_path, format="A4", print_background=True,
             margin={"top":"12mm","bottom":"12mm","left":"10mm","right":"10mm"})
    browser.close()

print(f"PDF created: {pdf_name} ({os.path.getsize(pdf_path)/1024:.0f} KB)")

# Build the email
msg = MIMEMultipart()
msg["Subject"] = f"Electra Belgium — Monthly Report {period}"
msg["From"] = config["sender_email"]
msg["To"] = ", ".join(config["recipients"])

body = (f"Bonjour,\n\nVeuillez trouver ci-joint le rapport mensuel Electra Belgium — {period}.\n\n"
        f"Cordialement,\nElectra Reporting")
msg.attach(MIMEText(body, "plain", "utf-8"))

with open(pdf_path, "rb") as f:
    part = MIMEApplication(f.read(), _subtype="pdf")
    part.add_header("Content-Disposition", "attachment", filename=pdf_name)
    msg.attach(part)

with smtplib.SMTP_SSL("smtp.gmail.com", 465) as smtp:
    smtp.login(config["sender_email"], config["app_password"])
    smtp.sendmail(config["sender_email"], config["recipients"], msg.as_string())

print(f"Sent: {pdf_name} -> {config['recipients']}")
```

### 3. Report success

Tell the user:
- Which report was sent (PDF filename)
- To whom (recipient list)
- Subject line used

---

## Credentials setup (first-run only)

If `~/.claude/skills-data/send-electra-report/config.json` does not exist:

1. Tell the user they need a **Gmail App Password** (not their regular Gmail password), with 2-Step Verification enabled.
2. Walk them through creating one:
   - Go to https://myaccount.google.com/apppasswords
   - App name: "Electra Report Sender"
   - Copy the 16-character password generated
3. Once the user provides the app password, create the config file:

```bash
mkdir -p ~/.claude/skills-data/send-electra-report
```

Then write the JSON config with the values provided. Use `pironadam@gmail.com` as sender and start with `["pironadam@gmail.com"]` as the recipients list (this can be expanded later).

4. Proceed with **Step 2**.

---

## Adding or changing recipients

To update the recipient list, edit `~/.claude/skills-data/send-electra-report/config.json` and modify the `"recipients"` array. The skill reads it fresh each run, so no other changes are needed. When the production recipient list is confirmed, replace `["pironadam@gmail.com"]` with the full list.

## Notes

- The report is sent as a **PDF attachment** — a faithful snapshot of the rendered webpage. This is the only reliable method because the source HTML relies on JavaScript and CSS variables that email clients strip/don't support.
- Rendering uses Playwright + Chromium headless. The 2.5s wait gives the page's JS time to populate the DOM before the PDF is captured.
- Gmail SMTP requires an App Password, not the account password (2FA must be enabled on the Google account).
- Never commit or log the `app_password` value.
- The config file is stored locally only at `~/.claude/skills-data/send-electra-report/config.json` — it is never pushed to any repo.
