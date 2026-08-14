# Changelog

All notable changes to the SEO Automation & Projects Dashboard.

## [Unreleased]

### Added
- `docs/` folder with full project documentation (spec, SOP, architecture,
  configuration, dependencies, troubleshooting, prompt library).
- `seo-automation-dashboard-data.sample.json` — example/reference data file.
- A registered Claude Skill for maintaining this project in future sessions.

## 2026-08-14 — Admin-only login provisioning; self-signup removed

### Changed
- **Self-signup removed entirely.** The login screen no longer offers any
  path to create an account — `#authScreen` now shows only the username +
  password sign-in form. Removed `.auth-tabs` (the `#authTabSignin` /
  `#authTabSignup` tab switcher), the entire `<form id="signupForm">`
  block, and the `.auth-switch-hint` paragraph (`#switchToSignup` /
  `#switchToSignin`) from the HTML.
- In `<script>`: removed the now-dead `setAuthMode(mode)`,
  `handleSignup(evt)`, and `isValidUsername(username)` functions, and the
  unused `authMode` state variable. Simplified `setAuthBusy()` (no longer
  branches on signin vs signup button copy). `onLoggedOut()` no longer
  calls `setAuthMode("signin")` (redundant with its existing
  `clearAuthMessages()` call). Kept `usernameToEmail()`,
  `friendlyAuthError()`, `handleSignin()`, `showAuthError()`,
  `showAuthNotice()`, `clearAuthMessages()`, `setAuthBusy()` unchanged —
  sign-in still depends on all of them.

### Added
- **Admin-only login provisioning on the Team tab.** `renderTeam()`'s
  actions cell now shows, admin-only, either **"Set up login"** (for a
  profile with no `auth_user_id` yet) or **"Reset password"** (for a
  profile that already has one), alongside the existing Delete button.
- `promptSetupLogin(profileId, fullName)` and
  `promptResetPassword(profileId, fullName)` (defined after
  `deletePerson()`): prompt for a username/password (or just password for
  reset), validate the 6-character minimum client-side, then call the
  already-deployed `admin-manage-users` Supabase Edge Function via
  `sb.functions.invoke('admin-manage-users', { body: { action:
  'create_login'|'reset_password', ... } })`. Real server errors are
  unwrapped via `await error.context.json()` (the SDK's `error.message`
  only ever says "Edge Function returned a non-2xx status code") and shown
  via `showToast(msg, "danger")`. On successful login creation, the
  profile row is re-fetched from `profiles` and `PROFILES` is patched in
  place so the "Set up login" button flips to "Reset password" without a
  full reload.
- The Edge Function itself verifies the caller is an admin server-side
  (via session JWT + `profiles.is_admin`) — the frontend still gates
  button visibility to admins for UX, but is not the security boundary.

## 2026-08-14 — Gamified leaderboard + UI polish

### Added
- **Leaderboard tab** (5th nav item, `data-view="leaderboard"`): a new
  `<section id="leaderboard">` with two "spotlight" cards — **Champion of
  the Week** and **Champion Overall** — plus a ranked "Analyst Progress"
  list showing every analyst with a rank (medal icons for top 3), avatar,
  an animated progress bar, and "`completed / total · pct%`" text.
- `computeLeaderboard()` (in `<script>`, right after `computeSummary()`):
  purely derived from the in-memory `projects` and `ANALYSTS` arrays — no
  new Supabase queries, schema changes, or persisted state. Computes, per
  analyst, total assigned, all-time completed count, completion %, and
  completed-this-week count (Mon 00:00 → next Mon 00:00, based on
  `dateCompleted`, a `YYYY-MM-DD` string from the native date input).
  - **Champion Overall** = analyst(s) with the highest all-time completed
    count, only crowned if that count is > 0; ties are joint champions.
  - **Champion of the Week** = same logic using the weekly count,
    independently of Champion Overall.
  - Friendly, theme-safe empty states when nobody has completed anything
    yet (overall) or this week.
- `renderLeaderboard()`: renders both spotlight cards, the ranked
  progress list, and a new compact "This week's champion" callout card on
  the Dashboard tab (`#dashChampionCard`, placed between the KPI row and
  the charts row) with a "View full leaderboard →" button. Wired into
  `renderAll()` alongside the existing render calls.
- `switchTab(view)`: extracted the nav click-handler's tab-switching logic
  into a reusable function so the new dashboard callout's "View full
  leaderboard →" button reuses the exact same view-switching code as the
  nav buttons instead of duplicating it.
- New CSS: `.progress-bar` / `.progress-fill` (reusable animated
  completion bars, `transition: width .5s ease`), `.spotlight-row` /
  `.spotlight-card` (gold-gradient "trophy" cards, new `--gold` custom
  property added to both the light and `body.dark` palettes),
  `.leaderboard-row` / `.rank-medal` / `.you-tag` (ranked list styling),
  `.dash-champion-card` (dashboard callout). All new classes have explicit
  `body.dark` overrides and follow the existing `.charts-row`/`.kpi-row`
  responsive pattern (`@media(max-width:900px)` and `720px`).

### Changed (visual polish)
- `.card` now has the same hover lift as `.kpi` (`transform:
  translateY(-2px)` + `box-shadow: var(--shadow-hover)` on hover) — it
  only had a box-shadow transition before.
- `.view.active` now cross-fades/slides in (`animation: view-in .2s`,
  6px translateY + opacity) on every tab switch instead of an instant
  `display:block` snap.

### Assumptions
- `dateCompleted` is a `YYYY-MM-DD` string (confirmed via the `<input
  type="date">` binding in the Project Tracker row template) or empty —
  empty/missing values are skipped when computing the weekly count rather
  than treated as errors.

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
