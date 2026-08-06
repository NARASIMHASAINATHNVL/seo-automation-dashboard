# SOP — Using the SEO Automation & Projects Dashboard

Audience: everyone on the SEO team. No technical background required.

## Opening the dashboard

1. Go to the `seo-automation-dashboard` folder.
2. Double-click `index.html`. It opens in your default browser.

## First-time setup (recommended, one-time per person)

1. Click **"Connect JSON file (autosave)"** in the bar under the tabs.
2. Choose (or create) a file, e.g. `seo-automation-dashboard-data.json`, in
   the same folder as `index.html`.
3. From now on, every change you make writes to that file automatically —
   no save button needed.

If your browser doesn't support this (see `TROUBLESHOOTING.md`), skip this
step and just remember to click **"Download JSON now"** at the end of a
session.

## Daily tasks

### Adding a new project
1. Go to the **Project Tracker** tab.
2. Click **"+ Add Project"**.
3. Fill in the name, assign it to a person, set status/priority/dates.

### Updating a project's status
1. Go to **Project Tracker**.
2. Change the **Current Status** dropdown on that row. The Dashboard tab's
   KPIs and charts update immediately.

### Logging today's work
1. Go to **Daily Status Log**.
2. Click **"+ Add Date Row"** if today isn't listed yet.
3. Type what you worked on in your column.

### Adding a new person to the team
1. Go to the **Team** tab.
2. Type their name, leave "Include in Daily Log" checked (uncheck only for
   people who won't log daily status — e.g. the team lead), click
   **"+ Add Person"**.
3. They now appear in the "Assigned to" dropdown and the per-analyst chart.

### Removing someone from the team
1. Go to **Team**, click **Delete** next to their name.
2. If they still have projects assigned, you'll be asked to confirm — their
   name stays on those existing rows until you manually reassign them.

## Weekly / as-needed

### Exporting to the shared Excel workbook
1. Go to **Project Tracker**.
2. Click **"Download Updated Excel (.xlsx)"**.
3. Open the downloaded file, copy the rows you need, paste into
   `SEO_Automation_Tracker.xlsx` on OneDrive.

### Backing up your data
- Click **"Download JSON now"** any time to save a snapshot you can hand to
  someone else or restore from later.

## Sharing the dashboard with someone else

The dashboard only shows what's in *your* browser's local storage / your
connected JSON file. To hand your current data to a teammate:
1. Click **"Download JSON now"**.
2. Send them the `.json` file.
3. They open `index.html`, click **"Connect JSON file (autosave)"**, and
   select the file you sent them (overwrite prompt is expected — say yes).

There is no automatic multi-user sync — see `ARCHITECTURE.md` for why, and
the SEO Automation Dashboard's suggestion list for how this could be added
later.
