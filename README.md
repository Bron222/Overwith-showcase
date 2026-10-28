# Overwith

A Flutter task manager for Android and iOS, for planning what to do today and across the week. Tasks can be due on a specific day or "sometime this week", and they can repeat and send reminders. Everything is stored on the device.

Website: [overwith.app](https://overwith.app) · [Privacy policy](https://overwith.app/privacy) · [Support](https://overwith.app/support)

> **The source code is private.** This page shows the project's features, design and engineering approach. Code access is available on request.

<p align="center">
  <img src="screenshots/today.png" width="200" alt="Today screen">
  <img src="screenshots/week.png" width="200" alt="Week screen">
  <img src="screenshots/edit-task.png" width="200" alt="Edit task form">
  <img src="screenshots/today-dark.png" width="200" alt="Today screen in dark mode">
</p>

## Features

- **Today view**: today's tasks with a done count, plus any unfinished "sometime this week" tasks.
- **Week view**: flexible tasks for the week, then Monday to Sunday with today highlighted. Arrows move to any past or future week, one week at a time.
- **Day or week tasks**: give a task a specific date, or just the week it belongs to.
- **Repeating tasks**: when you tick off a daily or weekly task, the next copy is created automatically, with its reminder moved forward.
- **Roll-forward**: unfinished tasks from past days move to today automatically, and "sometime this week" tasks move to the current week. Weekly tasks keep their weekday, and missed repeats don't pile up.
- **Reminders**: local notifications at a time you choose. The app asks for notification permission the first time you turn one on. On a locked phone set to hide sensitive content, the task name is hidden.
- **Quick editing**: tap a task to edit it, tick it off with the checkbox, or swipe left to delete it (with Undo).
- **Offline**: all data is saved in SQLite on the device. There's no account or server.
- **Light and dark mode**, following the phone's setting.
- **About screen** with the open-source licenses of the packages the app uses.

## Tech stack

| Area | Choice |
| --- | --- |
| Framework | Flutter 3.47 / Dart 3.13 |
| UI | Material 3 |
| State | `ChangeNotifier` and `InheritedNotifier` (built into Flutter, no extra package) |
| Storage | SQLite via `sqflite` |
| Notifications | `flutter_local_notifications` and `timezone` |
| Testing | `flutter_test` with `sqflite_common_ffi` for fast tests on a computer; `integration_test` for end-to-end tests on a real device |
| CI | GitHub Actions (`flutter analyze` and `flutter test` on every push and pull request) |

## Architecture

```
Screens ──read & listen──▶ TaskStore ──save──▶ TaskDatabase (SQLite)
                              │
                              └──schedule/cancel──▶ ReminderScheduler (notifications)
```

The app has four layers, each with one job:

| Layer | Responsibility |
| --- | --- |
| **Model** | The `Task` class, date helpers, and recurrence rules (pure Dart, no Flutter) |
| **Data** | One SQLite table and the queries for "today" and "this week" |
| **State** | `TaskStore`, the single source of truth that every screen reads from |
| **UI** | Today and Week tabs, the add/edit form, and reusable task rows |

Screens never talk to the database directly. Every change goes through `TaskStore`, which saves it, updates its list of tasks in memory, tells the screens to redraw, and schedules or cancels the task's reminder.

## Design decisions

- **Built-in state management, no extra package.** Flutter's own `ChangeNotifier` was enough for one shared store and kept the app simple and easy to follow.
- **Week tasks are stored on the Monday of their week.** That lets one date-range query return everything in a week, both day tasks and "sometime this week" tasks.
- **Dates are moved by calendar day, not by 24 hours.** Adding a fixed 24 hours can shift a time by an hour across daylight-saving changes, so repeating tasks and reminders move by calendar day instead.
- **Reminders sit behind an interface.** The store depends on a `ReminderScheduler` interface rather than the notification plugin directly, so tests use a fake scheduler, and a notification failure can never stop a task from being saved.
- **Reminders are rebuilt at every launch.** All scheduled reminders are cleared and recreated from the database on start-up, so they can't drift out of sync.
- **Inexact alarms on Android.** Reminders may arrive a few minutes late, but the app avoids Google Play's restricted "exact alarm" permission.
- **Missed tasks catch up once, not many times.** If a daily task is missed for three days, it shows up once today rather than three times. The check runs when the app opens and again when it comes back to the front, in case the day changed while it was open.
- **Permissions are asked for in context.** Notification permission is requested the first time a reminder is turned on, not when the app launches.

## Testing and quality

- **68 automated tests** across four levels:
  - model rules (dates, recurrence, validation)
  - database queries against an in-memory SQLite database
  - store behaviour, including reminder scheduling with a fake scheduler
  - widget tests that drive the real screens: adding, editing, deleting, ticking off and switching weeks
- **Static analysis** with Flutter's recommended lint rules; the analyzer reports no issues.
- **End-to-end tests** run the real app on an Android 36 emulator, with the real database and the real notification system. They add, tick off, edit and delete tasks, switch weeks, and restart the app to check that everything was saved. They also check that a task left over from an earlier day moves to today in the real database. They also check that reminders are scheduled with Android and cancelled again. Testing caught a real crash before release: two "add" buttons conflicted during screen transitions.
- **Continuous integration** runs the analyzer and the unit and widget tests on every push and pull request. All changes go through pull requests that must pass CI.
- **Checks before merging**: the analyzer, the unit and widget tests and the end-to-end tests must all pass before a change is merged into `main`. A pull request template keeps this checklist.

## Privacy and security

- **No network access**: the release build doesn't request internet permission.
- **No accounts or tracking**: tasks stay on the device.
- **Private lock-screen reminders**: the task name is hidden when the phone is set to hide sensitive content.
- **Published privacy policy** at [overwith.app/privacy](https://overwith.app/privacy).
- **Parameterised SQL queries**: text typed into a task can't be used to attack the database (no SQL injection).
- **Locked-down notification receivers**: other apps can't trigger them.
- **Read-only CI permissions**: the build pipeline can read the code but can't change it.
- **Dependabot**: dependencies are checked weekly, and each update arrives as a pull request that CI tests. Security alerts and automatic security fixes are turned on.

## What's next

- Build and test the iOS version
- Add a custom app icon and a release signing key
- Publish on Google Play and the App Store

## About this project

I built Overwith to plan my days and the weeks ahead in one place, and as a hands-on way to learn Flutter app development from start to finish. Along the way I practised structuring an app around a single source of state, storing data locally with SQLite, scheduling notifications, writing unit and widget tests, and running them automatically with GitHub Actions.

Feedback and suggestions are welcome. Feel free to open an issue.

---

© 2026 Bron222. All rights reserved. Screenshots and text on this page may not be reused without permission.
