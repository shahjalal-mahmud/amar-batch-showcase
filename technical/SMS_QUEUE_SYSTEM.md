# SMS Queue System

> Reliable message delivery without blocking the teacher or overwhelming the carrier.

---

## Why a Queue Exists

The naive approach to SMS — send the message immediately when an event happens — fails in practice for several reasons. Sending many messages at once triggers carrier rate-limiting. A failed send has no retry mechanism. The teacher's UI freezes while waiting for the radio confirmation. And if the app is killed mid-send, any unsent messages are lost.

The queue solves all of these. Messages are written to the database first, then processed by a background worker asynchronously. The teacher's action completes instantly. The sending happens quietly in the background. If a message fails, it is retried automatically. If the app is killed, the queue survives in the database and processing resumes when the app restarts.

---

## Queue Structure

Each entry in the queue represents one SMS to one recipient. The key fields are:

- The recipient's phone number (the parent's number, captured at admission)
- The fully rendered message body (placeholders already substituted)
- The message type (absent alert, fee reminder, fee confirmation, etc.)
- A priority level (high or normal)
- A status (pending, sent, failed, cancelled)
- A retry count

Messages are processed in priority order, oldest first within the same priority.

---

## Priority Levels

Two priority levels exist:

**High priority** — fee confirmations and absence alerts. These are time-sensitive. A parent wants to know immediately if their child did not attend, or if a payment was received.

**Normal priority** — fee reminders, exam results, and monthly attendance summaries. These are informational and can tolerate a short delay.

The worker always processes high-priority messages before normal-priority ones.

---

## Two Dispatch Paths

Not all messages go through WorkManager. Two dispatch paths exist based on the urgency of the message:

**Immediate path** — used for fee confirmations and admission welcome messages. These are enqueued and a foreground service is started immediately. The foreground service bypasses WorkManager's job scheduling, which can delay a job by up to 15 minutes depending on battery state and Doze mode. A teacher who just collected a fee expects the parent to receive the confirmation right away, not after a 15-minute wait.

**Bulk path** — used for attendance summaries, fee reminders, and exam results. These are enqueued and a WorkManager one-time worker is triggered. WorkManager is appropriate here because slight delays are acceptable and the system constraints (battery, network) should be respected.

Before enqueuing a new bulk worker, the scheduler checks whether one is already running. If an active worker exists, it will pick up the new queue entries on its next iteration without scheduling a duplicate.

---

## Sending Process

The worker processes one message at a time with a nine-second pause between each. This delay is intentional — sending many SMS in rapid succession is interpreted as spam by carriers on some networks.

For each message, the worker:

1. Reads the current app settings to verify SMS is globally enabled
2. Fetches the highest-priority pending entry from the queue
3. Sanitizes the message body (strips invisible characters that cause encoding issues)
4. Resolves which SIM to use based on the teacher's configured slot
5. Delegates the actual radio-level send to a sender component
6. Marks the entry as sent or increments its retry count based on the result

If the queue is empty at any point, the worker exits. If global SMS is disabled, the worker exits immediately without processing anything.

---

## Retry Logic

If a send attempt fails, the retry count is incremented. If the retry count is below the maximum (three attempts), the entry remains in the pending state and will be picked up on the next worker run. After three failures, the entry is marked as failed and the failure reason is stored. No further attempts are made.

The failure reason is available in the database for diagnostic purposes. Teachers can see failed messages in the app.

---

## SIM Selection

For dual-SIM devices, the teacher selects which SIM slot to use for sending. This selection is stored in app settings and applied to every outgoing message. If the configured slot is not available on the device (SIM removed, for example), the sender falls back to the default SIM and logs the fallback for investigation.

---

## Message Cancellation

When a student is deactivated, any pending queue entries for that student are cancelled. There is no point sending a message about a student who is no longer active — it would confuse the parent and potentially send stale information.

---

## Queue Cleanup

The queue is not a permanent log. Entries are cleaned up automatically:

- Sent messages are deleted after seven days
- Failed and cancelled messages are deleted after thirty days

This prevents the queue table from accumulating indefinitely while still providing a reasonable window for the teacher to see what was sent and what failed.

---

## Sender Abstraction

The component that performs the actual radio-level send is separated behind an interface. The implementation wraps Android's SmsManager, checks permissions, resolves the correct SIM subscription, splits long messages into multiple parts if needed, and uses broadcast receivers to confirm that each part was acknowledged by the carrier.

This separation exists so the queue processing logic can be tested independently of Android framework classes that cannot be instantiated in a standard test environment.

---

## Related Documentation

- [SMS System Feature](../features/sms_system.md) — Teacher-facing explanation of the SMS feature
- [Background Workers](./BACKGROUND_WORKERS.md) — How SmsWorker is scheduled and managed
