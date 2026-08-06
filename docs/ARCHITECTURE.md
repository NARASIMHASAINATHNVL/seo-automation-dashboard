# Architecture

## Design principle

Everything lives in one file: `index.html`. No backend, no build step, no
package manager. HTML + inline `<style>` + inline `<script>` + one inlined
third-party library (SheetJS). This keeps the "install" story at "double-
click the file" for a non-technical team.

## High-level component map

```
index.html
├── <style>            theming (CSS custom properties), layout, components
├── HTML body
│   ├── header + nav (4 tabs: Dashboard / Project Tracker / Daily Status Log / Team)
│   ├── syncBar        autosave status + Connect/Download JSON buttons
│   └── <section> per tab (only one .view.active shown at a time)
└── <script>
    ├── SheetJS (vendored, minified)   — xlsx export engine
    ├── canonical lists                — ANALYSTS, STATUSES, PRIORITIES, YESNO, DAILY_ANALYSTS
    ├── in-memory state                — projects[], dailyLog[], ANALYSTS[], DAILY_ANALYSTS[]
    ├── restoreFromLocalStorage()      — runs once on load, before first render
    ├── render*() functions            — pure functions of state -> DOM (re-run in full on every change)
    ├── mutation functions             — updateX()/addX()/deleteX(), each: mutate state -> re-render -> persistAll()
    ├── computeSummary() + chart code  — derives KPIs/chart data from projects[] each render
    ├── persistence layer              — collectData(), saveToLocalStorage(), writeToFileHandle(), persistAll()
    └── export functions               — downloadWorkbook() (xlsx), downloadJsonFallback() (json)
```

## Data flow (edit -> save)

```
User edits a field (project row / daily log cell / team roster)
        │
        ▼
  update*()/add*()/delete*() mutates the in-memory array
  (projects / dailyLog / ANALYSTS / DAILY_ANALYSTS)
        │
        ├──────────────► render*() re-renders affected DOM
        │                 (table, KPIs, charts, team list, daily header)
        │
        └──────────────► persistAll()
                              │
                              ├─► saveToLocalStorage()
                              │     always runs — safety net, per-browser
                              │
                              └─► writeToFileHandle()  (only if a file is connected)
                                    │
                                    ▼
                              real .json file on disk, rewritten in full
                              on every single edit
```

## Data flow (load)

```
Page load
   │
   ▼
seed data assigned to projects/dailyLog/ANALYSTS/DAILY_ANALYSTS (defaults)
   │
   ▼
restoreFromLocalStorage() — if a prior autosave exists in this browser,
it overwrites the seed data above (project/dailyLog/analysts/dailyAnalysts)
   │
   ▼
renderAll() + renderDailyHeader() + renderDaily() + renderTeam()
   │
   ▼
(user may click "Connect JSON file" to also link a real file — this does
 NOT re-import from that file; it only starts writing future edits to it.
 See TROUBLESHOOTING.md if you need file -> app import.)
```

## Why no multi-user sync

Each browser tab holds its own copy of the state in memory + localStorage.
"Connect JSON file" makes one browser's edits land in a real file, but
there's no listener that re-reads that file if someone else changes it —
so two people editing concurrently will diverge. This was an intentional
trade-off (see `PROJECT_SPECIFICATION.md` §3) to keep the tool at
zero-infrastructure. See the SEO Automation Dashboard's improvement
backlog for how real sync could be added (a small backend or a connector
like Google Sheets/Notion) if the team outgrows single-file sharing.

## Why the file is large

`index.html` is ~900KB, but only a few KB of that is dashboard logic — the
rest is the inlined SheetJS library (minified) that powers the Excel
export. This trade-off keeps the tool fully offline and dependency-free at
the cost of file size, which is irrelevant for a local file opened in a
browser.
