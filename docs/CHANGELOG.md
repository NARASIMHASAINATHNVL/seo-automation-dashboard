# Changelog

All notable changes to the SEO Automation & Projects Dashboard.

## 2026-08-18 — Tools Usage tab (UI-complete, inert until `tool_usage_events` migration lands)

### Added
- New **Tools Usage** nav tab (`data-view="toolsusage"`, after Leaderboard)
  built entirely against the `tool_usage_events` table shape from the
  queued migration (`id`, `tool_name`, `user_id` -> `profiles.id`,
  `used_at`), even though that table does not exist in the database yet.
  - **Log a Tool Use** — one row per tool with a live use-count and a
    "Using the tool now" button (`logToolUsage()`). Tool catalog is the
    distinct set of `project` names already loaded from `projects`
    (`toolCatalog()`) rather than a second `select distinct` round trip,
    since `projects` is already in memory and kept in sync via the
    existing realtime subscription.
  - Clicking the button inserts `{tool_name, user_id: currentProfile.id}`
    into `tool_usage_events` (same "resolve current profile id from
    `currentProfile`" pattern used by `addProjectRow()`/comments), shows
    `showToast("Logged your use of <tool>")` on success, or a friendly
    error toast on failure.
  - **Aggregate stats**: total uses + unique users (all-time) KPI cards,
    a per-tool breakdown table (total uses / unique users / last used),
    an 8-week and a 6-month usage trend chart (`renderToolUsageTrendChart()`,
    reusing the exact Chart.js line/area config, CSS-var theming, and
    empty-state overlay pattern from `renderTrendChart()`), and a
    per-person leaderboard (`renderToolUsageLeaderboard()`), styled the
    same as the existing Leaderboard tab rows.
  - All new CSS reuses existing custom properties only (`--surface-1/2`,
    `--surface-border`, `--text-primary/secondary/muted`, `--good`,
    `--accent`, etc.) via existing classes (`.card`, `.chart-card`,
    `.kpi-row`/`.kpi`, `.leaderboard-row`, `.table-wrap`, `.hint`) — no new
    hardcoded colors were introduced.
- **Graceful "table not set up yet" handling** — every
  `sb.from('tool_usage_events')` call (`loadToolUsageEvents()`,
  `logToolUsage()`) is wrapped in try/catch. Any read/write failure
  (missing-table `42P01`, "relation does not exist", PostgREST schema-cache
  misses, RLS denials, etc. — see `isMissingTableError()` /
  `friendlyToolUsageError()`) sets `toolUsageAvailable = false` and every
  stats section falls back to a shared friendly message via
  `toolUsageStatusMessage()`: "Usage tracking isn't set up yet — check back
  soon." when the table is missing, or "No usage logged yet." when the
  table works but is empty — mirroring the existing `renderDonut()` "No
  data yet" empty-state convention. The rest of the tab (tool list, nav,
  other tabs) renders normally regardless.

### Known limitation
- **This feature is UI-complete but functionally inert until the
  `tool_usage_events` table migration (queued separately, not part of this
  change) is applied to the database.** Until then, every section of the
  Tools Usage tab will show the "Usage tracking isn't set up yet — check
  back soon." message and the "Using the tool now" button will show a
  matching error toast instead of logging anything.

## 2026-08-18 — Footer credit

### Added
- App footer (`.app-footer`, below the main content, above the toast stack):
  "Developed by Sainath Nuvvula · For suggestions and feedback, reach out at
  +91-9535745456 or narasimha.sainath@bankbazaar.com" with working `tel:`
  and `mailto:` links.

## 2026-08-18 — BankBazaar branding + internal-tool disclaimer

### Added
- Favicon now points to the official BankBazaar favicon
  (`https://www.bankbazaar.com/images/favicon.ico`) instead of a generated
  inline SVG icon.
- Header logo badge (auth screen and main app header) now shows the
  BankBazaar mark instead of the generic checkmark icon.
- Added an "Internal BankBazaar tool — for authorized team members only.
  Do not share access or data outside the organization." disclaimer line,
  shown on both the login screen and the main app header.

## 2026-08-18 — Self-service password change, Time Taken column, status row tints

### Added
- **Self-service password change** — a "Change Password" button next to
  "Sign out" in the header opens a small modal (`openChangePasswordModal()`
  / `closeChangePasswordModal()` / `handleChangePassword()`) with new
  password + confirm password fields. Validates 8+ chars and that the two
  fields match, then calls `sb.auth.updateUser({ password })` directly —
  works for any logged-in user (analyst or admin) with no admin/service-role
  access required. Shows a toast on success ("Password updated") or surfaces
  the real Supabase error on failure. Entirely separate from the existing
  admin-only "Reset password" button on the Team tab
  (`admin-manage-users` edge function) — that flow is untouched.
- **Project Tracker "Time Taken" column** (`timeTakenHtml()`) — shows days
  between Date Assigned and Date Completed for "Completed & Approved"
  projects with both dates set, "—" otherwise. The day-diff calculation
  was extracted out of `computeTimeToCompletion()` into a new shared
  helper, `projectDaysTaken()`, so both the per-row column and the
  existing aggregate stats use the exact same logic (no duplication).
- **Status-based row tinting on Project Tracker** — each row gets
  `.row-not-started` (status "Assigned") or `.row-in-progress` (Ongoing /
  Review Pending / Completed & Approved) via `color-mix()` against the
  existing `--neutral` / `--blue` CSS vars, so "not started" work is
  clearly distinct from "in progress/done" work in both light and dark
  theme. No new hardcoded colors; existing `--good`/`--warning`/
  `--critical`/`--surface-*` vars untouched.

## [Unreleased]

### Added
- `docs/` folder with full project documentation (spec, SOP, architecture,
  configuration, dependencies, troubleshooting, prompt library).
- `seo-automation-dashboard-data.sample.json` — example/reference data file.
- A registered Claude Skill for maintaining this project in future sessions.

## 2026-08-18 — Overdue banners, smart inbox, workload/pipeline analytics, predicted dates, print export

All client-side, computed from data already fetched from Supabase — no
schema changes (the DB migration for a proper audit-log table is still
pending, so the "pipeline snapshot" chart below is explicitly labeled as a
current-state snapshot, not historical cycle time).

### Added
- **Overdue banner** (`renderOverdueBanners()`) — a red/critical banner
  pinned at the top of My Work and Dashboard, listing the signed-in user's
  own overdue/stale projects (reuses `getProjectStaleness`/
  `computeStaleness`, not reimplemented), each entry clickable
  (`jumpToProject()`) to scroll straight to that row in Project Tracker
  with a brief highlight flash. Admins see a team-wide version on the
  Dashboard tab instead of just their own; the banner renders nothing at
  all when there's nothing overdue.
- **"Needs Your Attention" smart inbox** (`computeNeedsAttention()` /
  `renderNeedsAttention()`) — merges overdue/stale projects, projects with
  Help Needed set, and (admin/team-wide scope only) unassigned projects
  into one de-duplicated, urgency-sorted list. Shown on the Dashboard
  (team-wide for admins, personal for analysts) and as a personal slice on
  My Work.
- **Time-to-completion analytics** (`computeTimeToCompletion()` /
  `renderTimeToCompletion()`) — average days from Date Assigned to Date
  Completed for "Completed & Approved" projects with both dates set,
  shown overall and per analyst in a new Dashboard card.
- **Analyst workload heatmap** (`renderWorkloadHeatmap()`) — a compact
  analyst-by-status grid next to the existing "Projects per Analyst by
  Status" bar chart, cell intensity driven by `color-mix()` against the
  existing `STATUS_COLOR` CSS-variable references (no new hardcoded
  colors).
- **Pipeline snapshot funnel** (`renderFunnelChart()`) — a new Chart.js
  horizontal bar chart on the Dashboard showing current project counts
  per stage (Assigned → Ongoing → Review Pending → Completed & Approved),
  explicitly labeled as a current-state snapshot rather than historical
  cycle time.
- **Predicted completion date** (`estimateCompletionDate()` /
  `predictedBadgeHtml()`) — for Ongoing/Review Pending projects, a muted
  dashed "≈ Est. <date>" badge under Target Date in both Project Tracker
  and My Work, computed as Date Assigned + the assigned analyst's own
  historical average days-to-complete, falling back to Target Date
  (clearly marked "(target)") when that analyst has no completion history
  yet.
- **Adjustable trend range** — 4/8/12-week preset buttons above the
  Leaderboard "Completions" chart, re-calling the existing
  `computeWeeklyTrend(projects, weeks)` (already parameterized) via a new
  `trendWeeks` module variable and `setTrendRange()`.
- **Print Report** — an admin-only "Print Report" button next to
  "Download Updated Excel" that switches to the Dashboard tab and calls
  `window.print()`; a new `@media print` block hides nav/toolbar/buttons
  and lays out cards cleanly. No new PDF library added.

## 2026-08-18 — Deep design-system rewrite + Chart.js migration

Third visual pass after two rounds of tightening didn't move the needle
enough — this one is a genuine step-change rather than incremental polish.
No business logic, Supabase calls, element ids, `data-view` wiring, or CSS
custom property *names* changed; only presentation.

### Added
- Google Fonts `Inter` (weights 400–900) now actually drives the whole
  UI's typographic hierarchy: 400 body copy, 600–700 headings/labels,
  800–900 for KPI numbers and brand marks (font link tag already existed
  from a prior pass; bumped to include weight 900 and applied consistently).
- New additive CSS custom properties (existing ones like `--shadow`,
  `--brand-text`, `--good` etc. were **not** renamed or removed):
  `--brand-gradient` (rich purple-to-violet gradient, light + dark
  variants) and a layered shadow scale `--shadow-sm` / `--shadow-md` /
  `--shadow-lg`, applied to the header logo, nav active-pill, buttons,
  auth card, and the import modal for more depth than the previous flat
  `--shadow`/`--shadow-hover` pair.
- KPI cards: numbers bumped to font-weight 900, icon badges now sit in a
  soft `color-mix()` tinted rounded-square (15% of the status color)
  instead of a flat neutral swatch, with a small hover scale-up on the icon.

### Changed
- **All three dashboard charts (`renderBarChart`, `renderDonut`,
  `renderTrendChart`) migrated from hand-rolled SVG DOM manipulation to
  Chart.js 4.4.0** (loaded via `<script src="…/chart.umd.min.js">` in
  `<head>`, before the Supabase/app script at the bottom of `<body>`).
  Function names/signatures and call sites (`renderAll()`) are unchanged.
  - `#barChart` → stacked bar chart, one dataset per status
    (`STATUSES`/`STATUS_HEX`), animated entry.
  - `#donutChart` → doughnut chart with a custom `centerText` Chart.js
    plugin drawing the total + "projects" label in the hole (replaces the
    manual center `<text>` elements).
  - `#trendChart` → line/area chart with a canvas gradient fill, driven by
    the unchanged `computeWeeklyTrend()`.
  - The `<svg>` elements for all three charts became `<canvas>` elements
    with the **same ids**. The old manual SVG tooltip divs
    (`#barTooltip`/`#donutTooltip`/`#trendTooltip`) were removed — nothing
    else referenced them — since Chart.js ships its own tooltips.
  - Each chart keeps a module-level instance variable
    (`barChartInstance`/`donutChartInstance`/`trendChartInstance`) that is
    destroyed before every re-render, and still reads grid/tick/label
    colors from `getComputedStyle(document.body)` custom properties at
    render time so both light and dark theme stay correct — mirroring how
    the old SVG code sourced its colors.
  - Empty states preserved: donut/trend charts skip creating a Chart
    instance (and clear the canvas) when there's no data yet, same as the
    old "No data yet" / "No completions yet" behavior.
- Nav active-pill, header logo, primary buttons, and the auth-tab active
  state now use `var(--brand-gradient)` instead of a repeated inline
  `linear-gradient(135deg, var(--navy), var(--navy-2))` literal — same
  visual family, single source of truth.

## 2026-08-17 — Bulk project import (Excel/CSV)

Admin-only "Import Projects" button on the Project Tracker toolbar. Parses
an `.xlsx`/`.xls`/`.csv` file (reusing the already-inlined SheetJS library —
no new dependency), matching the same column layout as "Download Updated
Excel" (`List of Projects` sheet if present, else the first sheet). Every
row is validated client-side before anything touches the database:
- Project name required; blank status defaults to `Assigned` (matches the
  existing single-row Add Project behavior and the DB check constraint);
  assignee must match an existing team member name; status/priority/help
  values must match the app's canonical lists; dates are parsed leniently
  (Excel date cells via `cellDates:true`, or best-effort string parsing)
  with a non-blocking warning if a date can't be read.
- A new preview modal (`#importModalOverlay` — the app's first modal
  component) lists every parsed row with a Ready / Warning / Error tag and
  the reason, before a single batched `insert()` commits only the valid
  rows. Wired via `handleImportFile`, `validateImportRow`,
  `renderImportPreview`, `confirmImport`, `closeImportModal`.
- The Import Projects button/hint are admin-only, toggled in
  `renderProjects()` alongside the existing "only my rows" toggle.

## 2026-08-14 — Overdue/stale alerts, streaks, badges, weekly trend chart, My Work view

Purely additive analytics/personalization pass on top of the data-dense
redesign — no schema changes, no new Supabase queries; everything is
derived client-side from `projects`/`ANALYSTS`/`PROFILES`/`currentProfile`
already loaded into memory.

### Added
- **Overdue/Stale detection** — `getProjectStaleness(p)` / `computeStaleness(projects)`.
  A project is **Overdue** if it has a `targetDate` in the past and isn't
  `Completed & Approved` ("Overdue by N days"); it's **Stale** (only if not
  overdue) if it's `Ongoing`/`Review Pending`, has a `dateAssigned`, and has
  been open 14+ days ("Stale — N days open"). `Assigned` and
  `Completed & Approved` rows are never flagged. Surfaced as: a new
  Dashboard alert banner (`.dash-alert-card`, wired via
  `renderStalenessAlert()`) with an "Overdue / Stale Projects" table
  (empty state: "Nothing overdue or stale right now — clean."), and a
  small `.staleness-tag` badge next to the status pill on flagged rows in
  the Project Tracker table and the new My Work table.
- **Streaks** — `computeStreak(analystName, projects)` counts consecutive
  weeks (Monday-based) with at least one `Completed & Approved` project,
  walking backward from the current week (falling back to last week once
  if the current week has zero completions yet, so a streak isn't lost
  mid-week), capped at a 26-week lookback.
- **Badges** — `computeBadges(analystName, projects)` derives chip badges
  from completion totals and streak: 🎯 First Completion (1+), ⭐ High Five
  (5+), 🏅 Perfect Ten (10+), 🔥 On Fire (3wk streak), 💎 Consistent (6wk
  streak). Rendered as `.badge-chip` pills next to the analyst's name on
  the Leaderboard ranked list and the Team tab.
- **Weekly trend chart** — `computeWeeklyTrend(projects, weeks=8)` buckets
  `Completed & Approved` projects by ISO week of `dateCompleted`. Rendered
  by a new hand-rolled SVG line/area chart (`renderTrendChart()`, same
  plain-DOM/gridline/tooltip conventions as `renderBarChart`/`renderDonut`)
  in a `.card.chart-card` on the Leaderboard tab: "Completions — Last 8
  Weeks", with a clean "No completions yet" empty state.
- **My Work tab** — new nav button/section (`data-view="mywork"`,
  positioned first in the nav), rendered by `renderMyWork()`: a personal
  KPI strip (Assigned/Ongoing/Review Pending/Completed/Overdue-or-Stale),
  a progress bar + streak + earned badges, and a compact table of the
  signed-in user's own projects. Empty state: "No projects assigned to you
  yet." Non-admins are automatically switched to this tab on their first
  successful sign-in (`onLoggedIn()` calls `switchTab('mywork')` once);
  admins keep the existing Dashboard default, and manual tab switches are
  never overridden on subsequent re-renders.

## 2026-08-14 — Visual redesign: data-dense analytics style

Full CSS/HTML visual pass across the whole app, at the product owner's
request, moving the look from a spacious/marketing-leaning style toward a
denser BI-tool feel (Looker/Metabase/Grafana-ish) — no Supabase, data, or
business-logic changes.

### Changed
- **KPI cards (Dashboard):** tighter padding, bold tabular-figure numbers,
  a new thin proportion mini-bar per card (`.kpi-bar`/`.kpi-bar-fill`,
  colored to match each card's status), and a new small trend/context line
  (`.kpi-trend`) — Total and Completed show "▲ N new/done this wk" (derived
  from existing `dateAssigned`/`dateCompleted` fields, Monday-based week
  window), the other four show "N% of total" as scannable context.
- **Charts:** the two Dashboard chart cards now carry a `chart-card` class
  with an accent top border and a slightly wider bar-chart column
  (`.charts-row` ratio 1.4fr→1.5fr) so they read as the page's centerpiece
  rather than incidental widgets. Fixed several hardcoded hex colors in
  `renderBarChart()`/`renderDonut()` (gridlines, axis labels, "No data
  yet"/"projects" center text) to instead read `--surface-border`,
  `--text-muted`, `--text-secondary` at render time so charts now stay
  legible in dark mode instead of using fixed light-mode grays.
- **Tables:** denser rows (`td` padding 7px→5px, table font 13px→12.5px),
  tighter sticky header with a subtle bottom shadow, numeric columns
  right-aligned (`Projects Assigned` on Team, and the trailing
  action/button column via `th:empty`/`td[data-label=""]`), taller
  `.table-wrap` viewport (520px→560px). Existing hairline-divider + row
  hover + `--stripe-color` status-stripe conventions kept as the single
  consistent row treatment (no zebra striping added).
- **Layout density:** tightened header/nav margins and padding, `main`
  max-width 1320px→1400px with less vertical padding, all `.card` panels
  denser (padding/margin trimmed), section headings restyled as small
  uppercase "panel titles" (BI-tool convention) instead of larger sentence
  case.
- **Nav:** kept as the existing top pill nav (all `data-view`/`tab-btn`/
  `switchTab()` wiring untouched) but restyled from a full pill to a
  tighter rounded-rectangle toolbar with uppercase labels, matching the
  denser overall rhythm.
- **Background:** simplified the multi-layer radial-gradient body
  background (dropped one gradient layer, reduced tint intensity) for a
  more sober, professional data-tool backdrop in both themes.
- Brand palette (navy/blue/purple accents, status colors) left untouched
  per the ask — this was a density/hierarchy/typography pass, not a
  rebrand.

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
