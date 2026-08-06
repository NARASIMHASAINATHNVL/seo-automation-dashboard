# Configuration — Data Schema & Canonical Lists

This app has no separate config file for its own behavior — the
"configuration" is the canonical lists hard-coded near the top of the
`<script>` in `index.html`, plus the JSON data schema used for persistence.

## Canonical lists (in `index.html`)

```js
let ANALYSTS = [...];       // full team roster — editable at runtime via the Team tab
const STATUSES = ["Assigned","Ongoing","Review Pending","Completed & Approved"];
const PRIORITIES = ["", "High","Medium","Low"];
const YESNO = ["", "Yes","No"];
let DAILY_ANALYSTS = [...]; // subset of ANALYSTS shown in Daily Status Log — editable via Team tab
const STORAGE_KEY = "seo-automation-dashboard-data-v1";
```

`STATUSES`, `PRIORITIES`, and `YESNO` are intentionally fixed — they mirror
the dropdown lists in the hidden "Lists" sheet of the master Excel workbook.
Changing them here would make exports inconsistent with that workbook, so
edit them only if the master workbook's lists also change.

`ANALYSTS` and `DAILY_ANALYSTS` are `let` (mutable) and are meant to be
edited through the **Team** tab in the UI, not by hand-editing the file.

## Persisted JSON schema

This is what `collectData()` produces, what gets written to localStorage
and to a connected `.json` file, and what a downloaded snapshot looks like:

```json
{
  "projects": [
    {
      "project": "FAQ Schema Generator",
      "assignedTo": "Bhagya",
      "status": "Review Pending",
      "priority": "",
      "dateAssigned": "",
      "targetDate": "",
      "dateCompleted": "",
      "help": "",
      "notes": ""
    }
  ],
  "dailyLog": [
    {
      "date": "2026-07-13",
      "Lochan": "",
      "Mahalakshmi": "Image SEO Auditor"
    }
  ],
  "analysts": ["Lochan", "Divakar", "Mahalakshmi", "Bhagya", "Pradeepa", "Neeraja S", "Narasimha Sainath"],
  "dailyAnalysts": ["Lochan", "Divakar", "Mahalakshmi", "Bhagya", "Pradeepa", "Neeraja S"],
  "savedAt": "2026-08-06T08:23:00.000Z"
}
```

Field notes:
- `projects[].status` must be one of `STATUSES` (or `""`).
- `projects[].priority` must be one of `PRIORITIES`.
- `projects[].help` must be one of `YESNO`.
- `dailyLog[]` rows are sparse objects — only analysts in `dailyAnalysts` at
  the time the row was created are guaranteed to have a key; the UI treats
  a missing key the same as an empty string.
- `savedAt` is informational only (last-write timestamp), not read back in.

A ready-to-copy example lives at `../seo-automation-dashboard-data.sample.json`
in the project root.

## Where each setting takes effect

| Setting | Used by |
|---|---|
| `ANALYSTS` | Project Tracker "Assigned to" dropdown, Dashboard per-analyst bar chart, Team tab |
| `DAILY_ANALYSTS` | Daily Status Log columns, Team tab checkbox |
| `STATUSES` | Project Tracker status dropdown, KPI cards, both charts |
| `PRIORITIES` | Project Tracker priority dropdown |
| `YESNO` | Project Tracker "Help Needed" dropdown |
| `STORAGE_KEY` | localStorage key — change this (and ship it as an update) if you ever need to run two independent copies of the dashboard in the same browser without them overwriting each other's saved data |
