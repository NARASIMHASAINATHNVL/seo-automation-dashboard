# Changelog

All notable changes to the SEO Automation & Projects Dashboard.

## [Unreleased]

### Added
- `docs/` folder with full project documentation (spec, SOP, architecture,
  configuration, dependencies, troubleshooting, prompt library).
- `seo-automation-dashboard-data.sample.json` — example/reference data file.
- A registered Claude Skill for maintaining this project in future sessions.

## 2026-08-06 (v3) — Team management + persistence

### Added
- **Team tab**: add/remove people from the roster, toggle whether each
  person appears in the Daily Status Log.
- `ANALYSTS` and `DAILY_ANALYSTS` are now runtime-mutable and persisted —
  previously hard-coded constants.
- Daily Status Log table header is now generated dynamically from
  `DAILY_ANALYSTS` instead of being static HTML.
- `collectData()` / the saved JSON now includes `analysts` and
  `dailyAnalysts` alongside `projects` and `dailyLog`.

### Changed
- Adding/editing/deleting a project, daily log row, or person now all
  funnel through the same `persistAll()` call, so "everything" (people +
  projects + daily log) saves to the same JSON consistently.

## 2026-08-06 (v2) — JSON persistence

### Added
- `localStorage` autosave on every edit (safety net, survives page
  reloads).
- "Connect JSON file (autosave)" — uses the File System Access API to
  write live edits straight to a real `.json` file on disk.
- "Download JSON now" — manual snapshot export, used as the fallback in
  browsers without File System Access API support.
- Sync status bar under the tab nav showing autosave state.

## 2026-07-13 (v1) — Initial dashboard

### Added
- Dashboard tab: KPI cards, per-analyst status bar chart, status-split
  donut chart, "needs help" table.
- Project Tracker tab: editable table matching the master Excel's columns
  and dropdown lists.
- Daily Status Log tab: per-analyst free-text daily log.
- "Download Updated Excel (.xlsx)" export matching the master workbook
  layout (List of Projects / Summary / Projects Daily Status sheets), via
  an inlined SheetJS library.
