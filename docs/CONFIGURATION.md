# Configuration — Supabase Backend, Data Schema & Canonical Lists

## Supabase project

The dashboard is backed by a Supabase project (Postgres + Auth + Realtime).
Credentials are hard-coded near the top of the `<script>` in `index.html`
(the anon key is safe to expose — access is enforced by Row Level Security):

```js
const SUPABASE_URL = "https://lqzwfequiylhdlvfjuce.supabase.co";
const SUPABASE_ANON_KEY = "eyJ..."; // anon/public key, protected by RLS
const sb = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

Tables: `profiles`, `projects`, `daily_logs`. RLS lets any authenticated
Supabase user select/insert/update/delete all rows in all three tables (this
is an internal trusted-team tool, not a per-user-ownership app). Auth signup
is restricted server-side to `@bankbazaar.com` emails via a Postgres trigger;
a non-matching signup is rejected and the raw Postgres error is surfaced in
the auth form. Realtime (`postgres_changes`) is enabled on all three tables
so edits from one signed-in teammate appear live for everyone else.

## Canonical lists (in `index.html`)

```js
let ANALYSTS = [];          // derived from the `profiles` table after login (applyProfiles())
const STATUSES = ["Assigned","Ongoing","Review Pending","Completed & Approved"];
const PRIORITIES = ["", "High","Medium","Low"];
const YESNO = ["", "Yes","No"];
let DAILY_ANALYSTS = [];    // profiles where include_in_daily_log = true
const THEME_KEY = "seo-automation-dashboard-theme"; // the ONLY app value still in localStorage
```

`STATUSES`, `PRIORITIES`, and `YESNO` are intentionally fixed — they mirror
the dropdown lists in the hidden "Lists" sheet of the master Excel workbook.
Changing them here would make exports inconsistent with that workbook, so
edit them only if the master workbook's lists also change.

`ANALYSTS` and `DAILY_ANALYSTS` are no longer hand-edited or read from
localStorage — they're rebuilt by `applyProfiles()` from the live
`PROFILES` array (fetched from Supabase on login, kept in sync via
Realtime) every time a profile is added, removed, or its
`include_in_daily_log` flag changes via the **Team** tab.

## Supabase table schema (source of truth, not editable from `index.html`)

```
profiles(id uuid pk, full_name text, email text unique, auth_user_id uuid unique nullable,
         include_in_daily_log boolean, created_at)
projects(id uuid pk, project text, assigned_to uuid->profiles.id, status text, priority text,
         date_assigned date, target_date date, date_completed date, help_needed text,
         notes text, created_by uuid->profiles.id, created_at, updated_at)
daily_logs(id uuid pk, log_date date, analyst_id uuid->profiles.id, task text,
           created_at, updated_at, unique(log_date, analyst_id))
```

The UI's in-memory `projects[]` array is a denormalized view of `projects`
(with `assigned_to` resolved to a name via `profileNameById()`), and
`dailyLog[]` is a wide-format pivot of `daily_logs` (one array entry per
`log_date`, one key per analyst name) built by `buildDailyLogFromRows()`.
Both are rebuilt on login (`loadAllData()`) and kept in sync afterwards by
the Realtime subscriptions in `subscribeRealtime()`.

`../seo-automation-dashboard-data.sample.json` (from the pre-Supabase
version of this app) is kept only as a reference for the old shape of a
`projects`/`dailyLog` row — it's no longer read or written by the app.

## Where each setting takes effect

| Setting | Used by |
|---|---|
| `ANALYSTS` | Project Tracker "Assigned to" dropdown, Dashboard per-analyst bar chart, Team tab |
| `DAILY_ANALYSTS` | Daily Status Log columns, Team tab checkbox |
| `STATUSES` | Project Tracker status dropdown, KPI cards, both charts |
| `PRIORITIES` | Project Tracker priority dropdown |
| `YESNO` | Project Tracker "Help Needed" dropdown |
| `THEME_KEY` | localStorage key for the light/dark toggle only — the one remaining localStorage use in the app |
