# Architecture

## Design principle

The UI still lives in one file: `index.html` — no build step, no package
manager, HTML + inline `<style>` + inline `<script>` + one inlined
third-party library (SheetJS). What changed: the app is no longer offline
/ zero-infrastructure. It now talks to a Supabase project (Postgres + Auth
+ Realtime) over the network for auth and all persistence, loaded via the
`@supabase/supabase-js` CDN script tag in `<head>`. Opening the file still
requires no install/server of your own, but it does require a network
connection and a bankbazaar.com login.

## High-level component map

```
index.html
├── <style>            theming (CSS custom properties), layout, components,
│                       plus auth-screen / loading-screen / user-chip styles
├── HTML body
│   ├── #authScreen     sign in / create account card (shown pre-login)
│   ├── #loadingScreen  spinner shown while the initial data fetch runs
│   └── #appRoot         header + nav (4 tabs) + <section> per tab — hidden
│                        until a session exists
└── <script>
    ├── SheetJS (vendored, minified)      — xlsx export engine
    ├── Supabase client + auth state       — sb, currentUser, currentProfile, PROFILES
    ├── canonical lists                    — STATUSES/PRIORITIES/YESNO (fixed); ANALYSTS/
    │                                        DAILY_ANALYSTS derived from PROFILES via applyProfiles()
    ├── in-memory state                    — projects[], dailyLog[] (denormalized views of the DB)
    ├── render*() functions                — pure functions of state -> DOM (unchanged from before)
    ├── mutation functions                 — updateX()/addX()/deleteX(), each: await a Supabase
    │                                        call keyed by DB `id` -> update local state -> re-render -> toast
    ├── computeSummary() + chart code      — derives KPIs/chart data from projects[] each render
    ├── loadAllData()/subscribeRealtime()  — initial fetch + postgres_changes subscriptions
    └── export functions                   — downloadWorkbook() (xlsx), unchanged
```

## Data flow (edit -> save)

```
User edits a field (project row / daily log cell / team roster)
        │
        ▼
  update*()/add*()/delete*() mutates the in-memory array by DB id
  (projects / dailyLog / PROFILES -> ANALYSTS / DAILY_ANALYSTS)
        │
        ├──────────────► render*() re-renders affected DOM immediately (optimistic)
        │                 (table, KPIs, charts, team list, daily header)
        │
        └──────────────► await sb.from(table).insert/update/delete(...)
                              │
                              ├─► success: showToast(...) confirms the write
                              └─► error: showToast(..., "danger") surfaces it;
                                    local state is left as-is (no automatic rollback)
```

Row identity is the Postgres `id` (uuid) everywhere — `deleteProject(id)`,
`updateProject(id, field, value)`, etc. — not array index, since Realtime
events and concurrent edits from other teammates can reorder or splice the
local arrays at any time.

## Data flow (load / login)

```
Page load
   │
   ▼
initAuth(): sb.auth.getSession() — if a session exists, skip straight in;
otherwise show #authScreen and wait for sign in / sign up
   │
   ▼
onLoggedIn(user): look up this user's `profiles` row by auth_user_id,
show #loadingScreen, then loadAllData() — fetches ALL profiles/projects/
daily_logs rows in parallel and rebuilds ANALYSTS/DAILY_ANALYSTS/projects/dailyLog
   │
   ▼
renderEverything() (renderDailyHeader + renderTeam + renderDaily + renderAll)
   │
   ▼
subscribeRealtime() — opens 3 postgres_changes channels (profiles/projects/
daily_logs); every INSERT/UPDATE/DELETE from ANY signed-in teammate is
reconciled into local state by id and re-rendered, live, no refresh needed
   │
   ▼
showApp() swaps #loadingScreen for #appRoot
```

`sb.auth.onAuthStateChange` also drives this — a SIGNED_IN event (e.g. after
`signUp`/`signInWithPassword` resolves) calls `onLoggedIn`, and SIGNED_OUT
calls `onLoggedOut()`, which unsubscribes Realtime and clears local state
back to empty arrays before showing the auth screen again.

## Multi-user sync

This is now a real shared multi-user app: Supabase Realtime (`postgres_
changes` on all three tables) pushes every insert/update/delete to every
signed-in tab, so two people editing concurrently converge instead of
diverging. RLS allows any authenticated bankbazaar.com user to read/write
all rows in `profiles`/`projects`/`daily_logs` — there's no per-row
ownership model, matching the old single-shared-file mental model but with
real concurrency instead of "last save wins" file overwrites.

## Why the file is large

`index.html` is ~900KB, but only a few KB of that is dashboard logic — the
rest is the inlined SheetJS library (minified) that powers the Excel
export. This trade-off keeps the tool fully offline and dependency-free at
the cost of file size, which is irrelevant for a local file opened in a
browser.
