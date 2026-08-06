# SEO Automation & Projects Dashboard

A single-file, no-server dashboard for tracking the BankBazaar SEO team's
automation projects and daily status — KPIs, charts, an editable project
tracker, and a daily status log, all matching the layout of the shared
"SEO_Automation_Tracker.xlsx" workbook.

## How to open it

Just double-click **`index.html`** in this folder. It opens directly in your
browser — no install, no server, no Python required.

## What's inside

- **Dashboard tab** — KPI counts (Total / Assigned / Ongoing / Review Pending
  / Completed & Approved / Need Help), a bar chart of projects per analyst by
  status, a status-split donut chart, and a table of projects flagged as
  needing help.
- **Project Tracker tab** — an editable table (project name, assigned
  analyst, status, priority, dates, help-needed flag, notes). Dropdowns are
  locked to the same analyst/status/priority lists as the master Excel.
- **Daily Status Log tab** — free-text log of what each analyst worked on,
  per day.

## Saving your edits

Every edit (adding/editing/deleting a project row or daily log entry)
auto-saves to this browser's local storage, so reopening the file later
restores your last changes automatically.

To also keep the data as a real file on disk:

1. Click **"Connect JSON file (autosave)"** in the bar under the nav and
   pick (or create) a `.json` file — e.g. `seo-automation-dashboard-data.json`
   in this folder.
2. From then on, every edit silently writes straight to that file — no
   further clicks needed.
3. If your browser doesn't support that (needs a Chromium-based browser,
   e.g. Chrome/Edge), use **"Download JSON now"** instead for a one-off
   snapshot each time you want one.

Use **Download Updated Excel (.xlsx)** in the Project Tracker tab to export
the table in the same column layout as the master workbook, ready to paste
back into the shared file on OneDrive.

## Notes

- Nothing here talks to a server or the internet — it's a static file, fully
  self-contained (the SheetJS library used for the Excel export is embedded
  inline).
- This is a separate, unrelated project from the "Bot Page-View Checker"
  tool — that one lives in its own folder and needs Python/Flask running.
