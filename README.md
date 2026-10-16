<p align="center"><img src="banner.svg" alt="CampusCheck: Bluetooth-verified campus attendance" width="100%"></p>

# CampusCheck

**A production attendance app that stops students from signing in for absent friends.**

QR codes, shared PINs and GPS can all be beaten from the corridor or a WhatsApp group. CampusCheck requires a student's phone to **hear a Bluetooth beacon broadcast from the lecturer's phone inside the classroom** *and* enter a short-lived 6-character code. The server checks both before it marks anyone present.

> The source code is private. This repo shows what the app does and how it's built. Code walkthroughs are available on request.

---

## How it works

```mermaid
sequenceDiagram
    participant L as Lecturer phone
    participant S as Student phone
    participant API as Supabase Edge Function
    participant DB as Postgres (RLS)
    L->>L: Generate 6-char code
    L-->>S: Broadcast rotating BLE token (every 30 s)
    S->>S: Detect beacon + read signal strength
    S->>API: Submit code + beacon token
    API->>DB: Check enrolment, code freshness, duplicate,<br/>HMAC token, room match, min RSSI
    DB-->>L: Live register updates in under 1 s (Realtime)
    Note over DB: pg_cron marks non-submitters Absent when the code expires
```

## Features

**Students:** check in with code + beacon, see attendance % per module with at-risk warnings, history, absence notes and appeals, GPA tracker, a "you can safely miss N more" calculator, badges, lecturer ratings, announcements, calendar, reminders.

**Lecturers:** generate codes that start the phone beacon automatically, a live register that updates in real time, CSV export, review absence notes (including bulk review), announcements, class events.

**Admins (14 tabs):** dashboard and 12-week trends, lecturer performance, full management of students, lecturers, programmes, modules, sessions, venues, beacons and terms, CSV imports with per-row results, mass enrolment, year promotion, password resets, attendance rules.

**Everywhere:** light/dark theme, forced password change on first login, merged calendar, push notifications.

## Engineering highlights

- **Custom native Android module (Kotlin)** for BLE advertising and scanning, with a time-windowed HMAC token so a beacon can't be replayed later.
- **Security enforced in the database, not the app.** Row Level Security is on all 25 tables. Students have zero access to attendance codes, and attendance rows can only be written by a validating edge function.
- **Server-side validation chain:** auth, then enrolment, code freshness, duplicate check, beacon token, room binding and signal strength, with clear error messages for each failure.
- **Realtime across roles:** a student's check-in appears on the lecturer's register in under a second, with a polling fallback.
- **Scheduled jobs:** `pg_cron` closes expired sessions and marks absences every minute.
- **Tested:** 48 automated tests covering auth, timing windows, the API contract and post-MVP features.

## Tech stack

| Layer | Tech |
|---|---|
| Mobile | React Native 0.86, Expo SDK 57, TypeScript/JavaScript, NativeWind, Reanimated, TanStack Query |
| Native | Kotlin Expo module (Bluetooth Low Energy advertise + scan) |
| Backend | Supabase: Postgres, Auth, Row Level Security, Edge Functions (Deno), Realtime, pg_cron |
| Builds | EAS Build → signed Android APK / App Bundle |

## Demo

<p align="center"><img src="media/campuscheck-demo.gif" alt="Lecturer walkthrough: switching modules, sessions, notes, announcements, rooms, and live light/dark theme switch" width="300"></p>

<p align="center"><a href="media/campuscheck-demo.mp4">▶ Watch the full-quality video (MP4)</a></p>

Recorded on a Samsung Galaxy S21+ running the production build against the live Supabase backend. Student names and numbers are blurred.

### Student app

<p align="center"><img src="media/campuscheck-student-demo.gif" alt="Student walkthrough: check-in, attendance stats, modules, marks, announcements, calendar, rooms, dark mode" width="300"></p>

<p align="center"><a href="media/campuscheck-student-demo.mp4">▶ Watch the student video (MP4)</a></p>

| Home & check-in | Modules | Attendance history | Marks & badges |
|---|---|---|---|
| <img src="screenshots/11-student-home.png" width="160" alt="Student home with code entry and attendance percentage"> | <img src="screenshots/12-student-classes.png" width="160" alt="Per-module attendance"> | <img src="screenshots/13-student-attendance.png" width="160" alt="Attendance history with dispute and rating"> | <img src="screenshots/14-student-marks.png" width="160" alt="Badges and marks tracker"> |

| Announcements | Calendar | Dark mode | Settings |
|---|---|---|---|
| <img src="screenshots/15-student-announcements.png" width="160" alt="Announcements"> | <img src="screenshots/16-student-calendar.png" width="160" alt="Day calendar"> | <img src="screenshots/17-student-dark-home.png" width="160" alt="Student home in dark mode"> | <img src="screenshots/18-student-settings.png" width="160" alt="Settings"> |

### Admin app

<p align="center"><img src="media/campuscheck-admin-demo.gif" alt="Admin walkthrough: attendance rules, trends, announcements, terms, events, rooms, sessions, curriculum, modules, programmes, dark mode" width="300"></p>

<p align="center"><a href="media/campuscheck-admin-demo.mp4">▶ Watch the admin video (MP4)</a></p>

| Overview | Attendance rules | Trends heatmap | Programme comparison | Announce |
|---|---|---|---|---|
| <img src="screenshots/19-admin-overview.png" width="160" alt="Admin control centre"> | <img src="screenshots/20-admin-rules.png" width="160" alt="Beacon and code rules"> | <img src="screenshots/21-admin-trends.png" width="160" alt="Attendance heatmap and 12-week trend"> | <img src="screenshots/22-admin-comparisons.png" width="160" alt="Programme and semester comparison"> | <img src="screenshots/23-admin-announce.png" width="160" alt="Broadcast announcements"> |

| Events | Modules | Programmes | Dark mode |
|---|---|---|---|
| <img src="screenshots/24-admin-events.png" width="160" alt="Events management"> | <img src="screenshots/25-admin-modules.png" width="160" alt="Modules management"> | <img src="screenshots/26-admin-programmes.png" width="160" alt="Programmes management"> | <img src="screenshots/27-admin-dark.png" width="160" alt="Admin in dark mode"> |

## Lecturer screenshots

| Lecturer dashboard | Live register | Sessions | Announcements | Rooms |
|---|---|---|---|---|
| <img src="screenshots/01-lecturer-dashboard.png" width="160" alt="Lecturer dashboard with attendance code and countdown"> | <img src="screenshots/02-live-register.png" width="160" alt="Live register with Present, Late and Absent filters"> | <img src="screenshots/03-sessions.png" width="160" alt="Scheduled class sessions"> | <img src="screenshots/04-announcements.png" width="160" alt="Module announcements and session rating"> | <img src="screenshots/05-venues.png" width="160" alt="Venues by floor with live availability"> |

| Reports | Settings | Dark: sessions | Dark: rooms | Dark: announcements |
|---|---|---|---|---|
| <img src="screenshots/06-reports.png" width="160" alt="Attendance by module chart"> | <img src="screenshots/07-settings.png" width="160" alt="Settings with reminders, dark mode and Bluetooth diagnostics"> | <img src="screenshots/08-dark-sessions.png" width="160" alt="Sessions in dark mode"> | <img src="screenshots/09-dark-venues.png" width="160" alt="Rooms in dark mode"> | <img src="screenshots/10-dark-announcements.png" width="160" alt="Announcements in dark mode"> |

---

## Hire me

I build mobile apps with React Native + Supabase, from idea to Play Store.

**GitHub:** [@yaseenselemane](https://github.com/yaseenselemane)
