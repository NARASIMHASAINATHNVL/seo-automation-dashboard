# Project Specification — SEO Automation & Projects Dashboard

## 1. Purpose

A single-file, offline-first dashboard for the BankBazaar SEO team to track
automation projects, daily analyst status, and team roster — replacing manual
updates to a shared "SEO_Automation_Tracker.xlsx" workbook with a live,
editable, chart-backed view that any team member can open with a
double-click, no install and no server required.

## 2. Owner & audience

- **Owner:** Narasimha Sainath, Head of SEO, BankBazaar.com
- **Users:** SEO team analysts (currently: Lochan, Divakar, Mahalakshmi,
  Bhagya, Pradeepa, Neeraja S) plus the owner.

## 3. Scope

In scope:
- Tracking automation/SEO projects: name, assignee, status, priority, key
  dates, help-needed flag, notes.
- Daily per-analyst status log (free text).
- Team roster management (add/remove people, control who appears in the
  Daily Log).
- Dashboard visualizations: KPI counts, per-analyst status bar chart,
  overall status donut chart, "needs help" list.
- Data export to `.xlsx` (matching the master workbook's column layout) and
  to `.json` (full raw state).
- Local persistence so edits survive closing/reopening the file.

Out of scope (by design, for a zero-backend tool):
- Real-time multi-user sync — each person's browser has its own local copy
  unless they explicitly share/import the same JSON file.
- User authentication / per-user attribution of edits.
- Server-side validation or storage.

## 4. Functional requirements

| # | Requirement | Status |
|---|---|---|
| F1 | Add / edit / delete a project row | Done |
| F2 | Add / edit / delete a daily log row | Done |
| F3 | Add / remove a person from the team roster | Done |
| F4 | Toggle whether a person appears in the Daily Log | Done |
| F5 | Dashboard KPIs and charts recompute live on every edit | Done |
| F6 | Export current state to `.xlsx` in master-workbook layout | Done |
| F7 | Export current state as `.json` | Done |
| F8 | Edits persist across page reloads without any user action | Done (localStorage) |
| F9 | Edits persist to a real file on disk, updated automatically | Done, where the browser supports the File System Access API (see Dependencies) |

## 5. Non-functional requirements

- **Zero install:** must open by double-clicking `index.html`. No build
  step, no `npm install`, no server.
- **Self-contained:** all CSS/JS, including the SheetJS export library, is
  inlined in the single HTML file.
- **Offline:** makes no network requests.
- **Resilient:** if the file-write path (File System Access API) is
  unavailable, the app still functions fully via localStorage + manual
  "Download JSON" / "Download Updated Excel".

## 6. Data model

See `CONFIGURATION.md` for the full JSON schema. Summary:

- `projects[]` — one row per project/automation.
- `dailyLog[]` — one row per date, one field per analyst included in the
  Daily Log.
- `analysts[]` — full team roster (used for "Assigned to" dropdown and the
  per-analyst chart).
- `dailyAnalysts[]` — subset of `analysts` shown as columns in Daily Log.

## 7. Success criteria

- Any analyst can open the file cold and understand project status within
  10 seconds (dashboard tab).
- No data loss on accidental tab close (localStorage safety net).
- Exported `.xlsx` pastes cleanly into the existing shared workbook without
  column remapping.

## 8. Related documents

- `SOP.md` — day-to-day usage procedure for the team.
- `ARCHITECTURE.md` — technical design and data flow.
- `CONFIGURATION.md` — data schema / canonical lists.
- `DEPENDENCIES.md` — libraries and browser APIs relied on.
- `TROUBLESHOOTING.md` — common issues and fixes.
- `CHANGELOG.md` — version history.
- `PROMPT_LIBRARY.md` — ready-to-use prompts for asking Claude to extend
  this dashboard.
