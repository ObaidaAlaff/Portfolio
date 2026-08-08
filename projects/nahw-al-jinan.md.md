# نحو الجنان (Nahw Al-Jinan)

> A full-stack Quran memorization platform connecting teachers and students inside structured study circles (halaqat) — daily progress tracking, assessments, and communication in one app.

## Overview

نحو الجنان is a mobile app built for Quran memorization schools and study circles. Students log daily memorization (wird) progress and review it on a calendar; teachers track assessments, follow-ups, and attendance for the halaqat they own; admins manage curriculum levels, quizzes, announcements, and finances. Everyone communicates through role-aware group and private chat, and the whole system runs on a real-time Postgres backend with strict role-based data access.

It's built for Islamic education institutions running multiple concurrent memorization circles, where a teacher's "official" ownership of a halaqah — not just chat membership — needs to drive what data they see.

## My Role

I designed and built the entire product end-to-end, solo:

- **Architecture** — a modular Flutter app (GetX) organized by feature/role, backed by a 30+ table Supabase (Postgres) schema with row-level security and SECURITY DEFINER RPC functions for privileged aggregate queries.
- **Backend & data model** — designed the schema for halaqat, memberships, daily reports, assessments, follow-ups, quizzes/exams, polls, announcements, chat, and finances, including the migration from a legacy single-group-per-user model to a proper many-to-many membership model with explicit "primary teacher" ownership.
- **Feature development** — built the student, teacher, and admin experiences: daily wird reporting, teacher dashboards, assessments/follow-up tracking, quizzes and exams, polls, announcements, and in-app chat with calls.
- **Notifications pipeline** — wired push notifications end-to-end via a Supabase Edge Function talking to Firebase Cloud Messaging.
- **Internal tooling** — built a standalone, no-build-step admin console for direct database operations (stats, exports, safe deletes, storage cleanup), and later rebuilt it in Vue 3 to compare architectures.

## Key Features

- 📖 **Daily memorization reports** — students log daily wird (recitation portion) with review-range tracking and a juz-fixed mode for advanced tracks
- 👩‍🏫 **Teacher dashboards** — assessments, follow-ups, and student progress scoped to the halaqat a teacher actually owns, not just any group they're a member of
- 🗓️ **Progress calendar** — students browse their full report history by date
- 📝 **Quizzes & exams** — question banks, timed sessions, auto-scored results
- 📊 **Polls & announcements** — school-wide or halaqah-scoped, with reactions
- 💬 **Role-aware chat** — group chats per halaqah, private messaging, voice/video calls, mentions and reactions
- 💰 **Financial reports** — per-halaqah/institution financial tracking for admins
- 🤲 **Adhkar reminders** — ambient dhikr popups with randomized placement, lifecycle-aware timing, and auto-dismiss
- 🛠️ **Internal admin console** — a separate zero-build web tool (built twice: vanilla JS and Vue 3) for KPIs, table exploration, storage cleanup, CSV export, and guarded bulk deletes — without touching the Supabase dashboard

## Tech Stack

![Flutter](https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![GetX](https://img.shields.io/badge/GetX-8A2BE2?style=for-the-badge&logo=flutter&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=black)
![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chart.js&logoColor=white)
![Android](https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white)

| Layer | Technology |
|---|---|
| Mobile app | Flutter (Dart), GetX (state management, routing, DI) |
| Backend | Supabase — Postgres, Auth, Realtime, Storage, Row-Level Security, RPC (SECURITY DEFINER) |
| Notifications | Firebase Cloud Messaging via a Supabase Edge Function |
| Internal admin tool | Vanilla JS + Supabase-js, and a parallel Vue 3 (CDN, no build step) rewrite; Chart.js for KPI trends |
| Platform | Android (Gradle/Kotlin, compileSdk 36) |

## Screenshots

> Suggested screens based on the app's actual feature set — rename, reorder, or add more, then drop your images into a `screenshots/` folder using these filenames (or your own).

### Teacher Home
![Teacher Home](screenshots/teacher-home.png)
The teacher's landing screen — halaqah overview with follow-up and assessment summary cards.

### Daily Report (Wird Tracking)
![Daily Report](screenshots/daily-report.png)
A student's daily memorization report — portion recited, review range, and rating stars.

### Assessments
![Assessments](screenshots/assessments.png)
A teacher's assessment log for their halaqah — scores and evaluation notes per student.

### Follow-ups
![Follow-ups](screenshots/follow-ups.png)
Meeting and follow-up records tracked per halaqah.

### Communication
![Communication](screenshots/communication.png)
Group and private chat list, organized by halaqah, with support for calls and reactions.


### Admin Dashboard (Internal Tool)
![Admin Dashboard](screenshots/admin-dashboard.png)
The developer-only console: live KPIs, table explorer, storage cleanup, and guarded CSV exports/deletes.

## Technical Highlights & Challenges

- **Ownership vs. membership** — introduced a `primary_teacher_id` concept on top of chat group membership, so a teacher's reports/assessments/follow-up dashboards reflect the halaqat they officially own, not every group they happen to be a participant in. This required auditing and updating every dependent controller (home, reports, assessments, follow-ups) to stay consistent.
- **Legacy data migration** — migrated group membership off a single `users.group_id` column onto a proper `group_members` join table, including a backfill migration for users whose membership had drifted out of sync, verified against the live database rather than assumed.
- **Subtle reactive-state bug** — root-caused a GetX `Obx` crash traced to short-circuit `&&` evaluation skipping a reactive read entirely under certain conditions, and fixed the underlying pattern rather than papering over the crash.
- **Background timer correctness** — found that `Timer.periodic` kept firing while the app was backgrounded (unlike Flutter's `AnimationController` tickers, which auto-mute), causing queued UI to burst on resume; fixed by syncing the timer's lifecycle to `AppLifecycleState` via `WidgetsBindingObserver`.
- **Hidden pagination cap** — discovered PostgREST's default 1000-row response limit applies even to RPC calls returning table-shaped data, not just direct table queries, and implemented consistent manual pagination across both.
- **Architecture comparison in practice** — built the same internal admin console twice: once with imperative DOM manipulation, once as a reactive Vue 3 app (directives, computed properties, component extraction) — both as zero-build static pages talking directly to Supabase.

## Status

✅ **Published on Google Play**
