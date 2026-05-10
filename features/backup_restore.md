# Backup & Restore

> A teacher's records are months of careful work. Losing them to a broken or stolen phone is not acceptable.

---

## What This Feature Does

Backup & Restore protects the teacher's entire database by saving a copy to their personal Google Drive account. If the teacher gets a new phone, loses their current one, or needs to reinstall the app, they can restore everything exactly as it was.

---

## Core Design

The app is offline-first — the local database on the device is always the source of truth. Google Drive is a safety copy, not a live sync target. The teacher's data never depends on Drive being reachable. Backup happens when the conditions are right; the rest of the time it is invisible.

The backup file lives in an app-specific folder on the teacher's own Google Drive account. Only the app can access this folder — it is not visible to other apps or to anyone else who might have access to the teacher's Drive.

---

## Automatic Backup

After the teacher connects their Google account, backup runs automatically in the background once per day. The worker runs only when the device has network connectivity, ensuring no mobile data is consumed unexpectedly.

The teacher does not need to think about this. They open the app, do their work, and the backup happens on its own when conditions are met. The backup screen shows when the last successful backup occurred — "Last backup: 3 hours ago" — so teachers can verify that their data is protected.

If a backup attempt fails, the worker retries on subsequent days. After repeated failures, the teacher is notified so they can investigate (a Google account sign-in issue, for example).

---

## What Gets Backed Up

The entire local Room database is backed up — every student, every attendance session, every fee payment, every exam result, every SMS template the teacher has customized, and all settings. Nothing is left out.

---

## Restoring on a New Device

When a teacher opens the app on a new phone for the first time, they choose the "Existing User" path. After signing in with the same Google account, they connect to Drive and select their backup file. The app downloads the backup and replaces the empty local database with the restored data.

Restore is a deliberate action — it never happens automatically. The teacher chooses when and what to restore. Once restored, the app is in exactly the state it was at the time of the last backup.

---

## Onboarding and First Backup

During initial setup, the teacher signs in with Google, creates their profile, and the app sets up the database and backup metadata in a single atomic operation. If any part of setup fails, nothing is left in a half-created state.

---

## Backup Format Versioning

The backup file carries a version number. When a newer version of the app introduces database changes, the restore process can detect the version of the backup and handle any necessary adjustments. This means a backup taken with an older version of the app can still be restored correctly after an app update.

---

## Privacy

The backup is stored in the teacher's own Google Drive account. Anthropic does not have access to it. The app developer does not have access to it. No third party has access to it. The data belongs entirely to the teacher.

---

## Related Features

- [Student Management](./student_management.md) — All student data is included in the backup
- [Attendance Tracking](./attendance_tracking.md) — Full session and record history is backed up
- [Fee Management](./fee_management.md) — Every payment record is backed up
- [Exam Tracking](./exam_tracking.md) — All exam and result data is backed up
- [SMS System](./sms_system.md) — Absence alerts and monthly summaries are sent through the SMS queue
