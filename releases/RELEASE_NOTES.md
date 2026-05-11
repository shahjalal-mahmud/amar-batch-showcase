# Release Notes — Amar Batch

---

## v1.0.0 — Beta Release

_May 2026_

**Status:** Live on teacher devices · Paid subscriptions active · Preparing for Play Store

---

### What's in This Release

**Core Features**

- Student management — single admission, bulk CSV import, class/shift assignment, soft deactivation, annual promotion
- Daily attendance — Default Present and Default Absent modes, past-date editing, calendar view
- Fee collection — single and multi-month payments, due calculation, fee reminders, profit tracking with expenses
- Exam tracking — exam creation with auto-generated result placeholders, mark entry, class statistics
- SMS system — 6 message types, priority queue, dual-SIM support, customizable templates
- Google Drive backup — automatic daily backup, full restore on reinstall
- License system — trial period, grace period, paid activation, device binding, offline validation

**Architecture**

- Clean Architecture + MVVM with Jetpack Compose (Material 3)
- Room database with 17 normalized tables
- 100% offline-capable — internet required only for backup and license sync
- WorkManager background jobs for backup, SMS queue, and cleanup
- Koin dependency injection

---

### Known Limitations in Beta

- Single teacher, single batch per installation (multi-batch planned for a future release)
- Manual APK distribution — Play Store submission in progress
- License activation is manual (developer activates after bKash/Nagad payment)
- No cloud SMS gateway — uses device SIM only

---

### Devices Tested

- Android 8.0 (API 26) and above
- Single-SIM and dual-SIM phones
- Tested on low-end to mid-range Android devices common in Bangladesh

---

## Upcoming — v1.1.0 (Planned)

- Play Store public release
- Multi-batch support
- Improved backup scheduling controls
- Performance improvements for large student lists (500+)

---

## Upcoming — v2.0.0 (Roadmap)

- Coaching center mode (multiple teachers)
- Cloud SMS gateway option
- Advanced reporting and analytics

---

_For questions or bug reports: mahmud.nubtk@gmail.com_
