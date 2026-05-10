# Background Workers

> WorkManager handles the tasks that must keep running even when the teacher closes the app.

---

## Overview

Three WorkManager workers run in the background. Each has a single, clearly scoped responsibility. None of them touches the UI. All of them are designed to fail gracefully — a worker that crashes or exhausts its retries should never permanently break a teacher's workflow.

---

## The Three Workers

### AutoBackupWorker

Runs on a repeating schedule and uploads the local database to the teacher's Google Drive when the device has internet connectivity.

**Schedule:** Every hour, network required.

**What it does:** Checks that a signed-in teacher exists on this device, then delegates to the backup repository. On success, the local backup metadata is updated with the current timestamp. On failure, WorkManager retries with exponential backoff.

**Retry behavior:** Up to three retries per period with exponential backoff (30s → 60s → 120s). After three failures in one period, the worker exits — but it exits with a success result, not a failure result.

**Why success on exhausted retries:** Returning a permanent failure to WorkManager on a periodic job tells the system to never run that job again. For a teacher in a rural area where three consecutive hours of bad connectivity is normal, this would silently kill all future backups until the app is reinstalled. The correct behavior is to treat "I couldn't do it this hour" as a recoverable condition and try again next hour. Failures are still logged to Crashlytics for investigation.

**Scheduling policy:** On each app launch, the scheduler compares the currently running worker's app version against the current version. If the version has changed (app update), the old worker is cancelled and a fresh one is enqueued. If the version matches, the existing worker is left alone. This prevents app updates from leaving a stale worker running with outdated behavior.

---

### SmsWorker

Processes the SMS queue — sending bulk messages one at a time with a delay between each.

**Schedule:** One-time work, enqueued when bulk SMS is triggered (attendance summaries, fee reminders, exam results). Not periodic.

**What it does:** Reads app settings, checks that SMS is globally enabled, then drains the pending queue in priority order. Each message is handed to a sender abstraction that handles the actual Android radio-level call. After each send, the worker waits nine seconds before processing the next item to avoid carrier rate-limiting.

**Two dispatch paths exist:** One for bulk messages (attendance summaries, fee reminders, exam results) that go through WorkManager, and one for immediate single-message sends (fee confirmations, admission welcome) that go through a foreground service and bypass WorkManager's job scheduling entirely. This ensures time-sensitive confirmations are not delayed by JobScheduler.

**Deduplication:** Before enqueuing a new SmsWorker, the scheduler checks whether one is already running or queued. If an active worker exists, it will pick up the new queue entries on its next loop iteration without any additional scheduling.

**Foreground notification:** The worker promotes itself to foreground to prevent the system from killing it mid-send on low-memory devices. The notification is cancelled when the queue is empty.

---

### CleanupWorker

Removes old SMS queue entries to prevent the queue table from growing indefinitely.

**Schedule:** Weekly, no network requirement.

**What it does:** Deletes sent messages older than seven days, and failed or cancelled messages older than thirty days. This runs as a background database write with no user-facing impact.

---

## Worker Failure Philosophy

Each worker is designed with the same principle: fail quietly, recover automatically, never leave the system in a permanently broken state.

- Workers catch exceptions internally and log them to Crashlytics rather than letting them propagate to WorkManager as permanent failures.
- Periodic workers always exit with a success result after exhausting retries, so they remain scheduled for future runs.
- Workers that depend on app settings read those settings at the start of each run rather than caching them, so a teacher disabling SMS between runs is always respected.

---

## Sender Abstraction

The actual Android SMS send operation is hidden behind an interface. The production implementation wraps Android's SmsManager, handles permission checks, resolves the correct SIM for the selected slot, splits long messages into parts, and uses a broadcast receiver to confirm each part was accepted by the carrier.

This abstraction exists for testability — SmsManager is a final Android system class that cannot be instantiated in a pure JVM test environment. The workers themselves contain only logic that can be tested without a device.

---

## Related Documentation

- [SMS Queue System](./SMS_QUEUE_SYSTEM.md) — How the queue is structured and processed
- [Offline-First Strategy](./OFFLINE_SYNC_STRATEGY.md) — Why the backup worker is designed the way it is
- [Backup & Restore Feature](../features/backup_restore.md) — Teacher-facing view of the backup system
