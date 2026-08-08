# نحو الجنان — Admin Console (Vue Edition)

> A zero-build, reactive database console for direct Supabase operations — built with Vue 3.

## Overview

This is an internal, developer-only control panel for **نحو الجنان**, a Quran memorization app. It gives the maintainer direct access to the production Supabase database — live KPIs, table exploration, storage cleanup, CSV export, and guarded bulk deletes — without ever opening the Supabase dashboard. It's a single static page with no build step: open `index.html` and it runs.

It exists purely as internal tooling, not a customer-facing product, and is meant for one audience: the developer who owns the database.

## My Role

I designed and built the entire tool solo, including a from-scratch architectural rewrite:

- Designed all six operational panels (dashboard, explorer, database stats, storage, export, delete) and the Supabase queries behind each one.
- Originally built the tool with vanilla JavaScript and manual DOM manipulation, then **redesigned it around Vue 3's reactivity model** — replacing imperative `el()`/`innerHTML` DOM-building with declarative templates, `computed` properties, and a reusable component.
- Built the safety layer for destructive actions: type-to-confirm delete guards, backup-before-delete CSV export, and a hard-coded whitelist of tables that can never be bulk-deleted from this tool.
- Wired direct Supabase integration: paginated table queries, custom Postgres RPC calls for table-size introspection, and Storage API calls for orphaned-file cleanup.

## Key Features

- 📊 **Live KPI dashboard** — user/halaqah counts by type, daily/weekly/monthly report volume, message totals, assessment and follow-up counts, average quiz score, and financial report counts, plus 14-day trend charts for messages and reports
- 🔍 **Table explorer** — pick any table to get an exact row count, or group by any column to see a value distribution (e.g. users by role)
- 🗄️ **Database stats** — total/data/index size and row estimates per table, pulled via a custom Postgres RPC function
- 🧹 **Storage & cleanup** — bucket overview (file count, total size), and an orphaned-file scanner that cross-references every stored file against message/announcement references, with multi-select deletion
- ⬇️ **CSV export** — export any table in full, with an optional date-range filter when the table has a suitable date column
- 🗑️ **Guarded delete by date** — bulk-delete rows in a date range from a strict whitelist of renewable tables (messages, reactions, calls, poll votes, quiz sessions/results) — deliberately excludes anything that's a "real record" (reports, assessments, follow-ups, financial data); requires typing the exact affected-row count to unlock the delete button, with an optional one-click CSV backup first

## Tech Stack

![Vue.js](https://img.shields.io/badge/Vue.js%203-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chart.js&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)

| Layer | Technology |
|---|---|
| UI framework | Vue 3 (global build via CDN — no bundler, no build step) |
| Data layer | Supabase JS client — Postgres queries, Storage API, RPC (SECURITY DEFINER functions) |
| Charts | Chart.js (line charts for 14-day trends) |
| Markup/styling | Static HTML + CSS, RTL Arabic UI |
| Hosting/runtime | Fully static — opens directly as a local file, or served by any static file server |

## Screenshots

> One screen per tab. Drop your images into a `screenshots/` folder using these filenames (or your own) and rename the headings as needed.

### Dashboard
![Dashboard](screenshots/dashboard.png)
Live KPI cards (users, halaqat, reports, messages, assessments, follow-ups, quiz average, financial reports) plus 14-day trend charts for messages and daily reports.

### Explorer
![Explorer](screenshots/explorer.png)
Pick any table to get an exact row count, or group rows by any column to see a breakdown (e.g. users by role).

### Database
![Database](screenshots/database.png)
Per-table storage stats — total size, data size, index size, and estimated row counts.

### Storage & Files
![Storage and Files](screenshots/storage-files.png)
Bucket overview plus an orphaned-file scanner that finds media not referenced by any message or announcement, ready for multi-select cleanup.

## Technical Highlights & Challenges

- **Rebuilt around reactivity, not just re-skinned** — the original version manually rebuilt DOM subtrees (`innerHTML = ''` then re-append) after every data change. The Vue rewrite moves all state into a single reactive `data()` object and lets `computed` properties (like the selected-files list, its total size, or which table/date-column pairing is active) recalculate automatically — no more manually calling a `render...()` function after every state change.
- **Chart.js instances kept outside Vue's reactivity on purpose** — wrapping a Chart.js instance in Vue's reactive Proxy causes problems with its internal circular references, so chart handles are stored in a plain module-level object instead of `data()`, while still using Vue `$refs` (not global `getElementById` lookups) to reach the canvas elements.
- **Hidden pagination cap on RPC calls** — discovered that PostgREST's default 1000-row response limit applies even to RPC calls that return table-shaped data (like the storage-object listing), not just plain table selects — so pagination via `.range()` had to be implemented consistently across both query paths.
- **Guarded destructive actions by design** — every irreversible action (bulk delete, file removal) requires typing the exact affected-row/file count before the action button unlocks, and the date-range delete tool offers a one-click CSV backup before anything is removed. The delete tool's table list is a hard-coded whitelist, not the full schema, so "real" records (reports, assessments, follow-ups, financial data) are structurally impossible to bulk-delete from here.
- **Zero-build by constraint, not by accident** — as an internal single-developer tool, adding a bundler/build pipeline would have been pure overhead. Vue 3's CDN global build made a genuinely componentized, reactive architecture possible while keeping the "open the HTML file and it just works" property of the original.

## Status

🛠 **Internal tool — actively used to manage the live production database** (not a public-facing product; no release/store status applies)
