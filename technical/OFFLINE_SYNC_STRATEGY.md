# Offline-First Strategy

> The app works completely offline. Internet is optional.

---

## The Core Principle

Every feature — attendance, fees, exams, student management — works without any network connection. Google Drive is used for backup only, not for day-to-day operation.

This is not a technical accident. It is the primary product decision. The target users are private tutors in Bangladesh, many of whom teach in buildings with poor or expensive mobile data. An app that requires internet to mark attendance is an app that fails its users on the days they need it most.

---

## Room Is the Single Source of Truth

All application state lives in the local SQLite database managed by Room. There is no in-memory cache that can drift out of sync. There is no reconciliation logic between local and remote state.

When a teacher marks a student absent, that record goes directly into Room. The UI updates from a reactive Flow that Room exposes — no manual refresh, no polling, no network call required. The data is always exactly what is in the local database.

---

## What Works Offline

Everything that matters day-to-day:

- Taking and editing attendance
- Collecting and tracking fees
- Recording exam results
- Adding, moving, and deactivating students
- Sending SMS (uses device SIM, not internet)
- Viewing all history and reports
- Calculating dues and profit
- License validation (falls back to locally cached expiry date)

---

## What Requires Internet

Two things, both optional and gracefully degraded:

**Google Drive backup** — runs automatically when the device has network access. If there is no internet, no backup runs, but the app continues working normally. The backup picks up on the next opportunity.

**License refresh from the server** — the app checks the license server on launch when online. If offline, it uses the cached expiry date stored locally. This means a teacher with a valid license can always open the app even without connectivity.

---

## The Backup Strategy

The backup system is designed around the reality that internet connectivity is intermittent, not guaranteed.

When the device is online, a background worker serializes the entire Room database into a single JSON document and uploads it to the teacher's personal Google Drive folder. No third-party server is involved. The backup file lives in the teacher's own account.

The worker runs on a schedule but only when connectivity is available. If a backup fails — network drops mid-upload, token expired, Drive quota exceeded — the worker retries with exponential backoff. After exhausting retries for a given period, it exits cleanly and tries again on the next scheduled run. It never permanently dies.

**Critical design choice:** the backup worker never returns a permanent failure state to WorkManager. A missed backup in one period is recoverable — the next run will try again. A permanently dead periodic job would silently stop all future backups, and the teacher would only discover this when their phone breaks and they try to restore.

---

## Restore on a New Device

Restore is a deliberate, one-time action during onboarding. It is not automatic sync.

When a teacher reinstalls or gets a new phone, they sign in with their Google account. The app fetches their teacher profile and license from Firestore (identity layer), then downloads the academic data backup from Drive and replaces the empty local database with the restored content.

The restore process validates the backup document before touching the local database. If validation fails — empty document, structurally corrupt data, incompatible schema version — the teacher's current local data is never erased. After the transaction commits, a count verification pass checks that the number of rows written matches the number expected from the backup document.

---

## Conflict Avoidance

Because there is only ever one device (single teacher, single phone), there are no sync conflicts. The local database is always authoritative. The Drive backup is a point-in-time snapshot, not a live mirror.

This eliminates an entire class of complexity that offline-first systems typically struggle with. There is no last-write-wins logic, no merge strategy, no conflict resolution UI.

---

## Why Not Cloud-First

A cloud-first design — where Firestore or a similar service is the primary store — was considered and rejected for three reasons:

The first is the connectivity reality described above. Teachers in rural Bangladesh cannot depend on a stable connection during class.

The second is cost and complexity. A cloud-first database for a solo-developer app requires server infrastructure, real-time sync logic, conflict resolution, and ongoing operational overhead that is disproportionate to the product's current scale.

The third is data ownership. With Room as the source of truth, the teacher's data lives on their device and in their own Google Drive. The developer has no access to it. This is a privacy property worth preserving.

---

## Related Documentation

- [Background Workers](./BACKGROUND_WORKERS.md) — How the backup worker is scheduled and managed
- [Backup & Restore Feature](../features/backup_restore.md) — Teacher-facing explanation of the backup system
