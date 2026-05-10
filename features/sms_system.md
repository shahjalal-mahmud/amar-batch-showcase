# SMS System

> Parents stay informed. Teachers stay in control. Everything happens in the background.

---

## What This Feature Does

The SMS system sends automated messages to parents using the teacher's own phone and SIM card — no internet, no third-party service, no extra cost beyond the teacher's normal SMS rate. Every message goes through a queue that handles sending, retries, and cleanup automatically.

---

## Core Design

SMS is never sent directly from the action that triggers it. When a student is marked absent, the attendance is saved first. Only after that succeeds does an SMS entry get added to the queue. If the queue entry fails, the attendance record is unaffected. Main operations — attendance, fees, exams — can never be blocked or broken by messaging.

The queue is processed by a background worker that sends one message at a time with a short delay between each. This prevents the device or carrier from flagging the messages as spam.

---

## Message Types

Six types of SMS can be sent:

**Admission** — Sent when a new student is admitted manually. Informs the parent that their child has been enrolled.

**Absent Alert** — Sent after attendance is taken and a student is marked absent. One message per absent student per session.

**Fee Reminder** — Sent to parents of students who have not paid for a given month. Triggered manually by the teacher.

**Fee Confirmation** — Sent after a fee payment is recorded. Confirms to the parent the amount and month paid.

**Exam Result** — Sent after exam results are saved. Includes the student's mark and percentage.

**Monthly Summary** — Sent at month's end. Reports how many classes the student attended out of the total sessions held.

---

## Control Hierarchy

Three levels of control determine whether an SMS is actually sent:

The **global switch** in app settings is the master control. If SMS is globally disabled, nothing is sent regardless of any other setting.

**Class-level settings** let the teacher enable or disable each message type independently per class. A teacher might want fee reminders for one class but not another, or might want absent alerts only for younger students.

**Student eligibility** is the final check. A student must be active and must have a valid parent phone number. A student without a phone number on file is silently skipped — no error, no crash.

Bulk CSV imports do not send admission SMS regardless of settings. Sending dozens of welcome messages during a mass import would be disruptive and is not what teachers expect.

---

## Message Templates

Each of the six message types has a default template with placeholder fields. Templates use markers like `{student_name}`, `{date}`, `{amount}`, `{mark}`, `{teacher_name}`, and others that are filled in with real values before each message is sent.

Teachers can edit any template to match their preferred wording or language. A template that has been customized can be reset to its default at any time. Templates cannot be deleted or added — only the content of the six existing types can be changed.

---

## Queue and Retry Logic

Every outgoing message enters the queue as a Pending entry with a priority level. High-priority messages (absent alerts, fee confirmations) are processed before normal-priority ones (monthly summaries, reminders). Within the same priority, messages are sent oldest-first.

If a send attempt fails, the worker retries up to three times with increasing delays between attempts. After three failures, the message is marked Failed and no further attempts are made. The failure reason is stored for the teacher to see.

When a student is deactivated, any pending messages queued for that student are cancelled. There is no reason to send notifications for a student who is no longer active.

---

## Dual SIM Support

Teachers with dual-SIM phones can choose which SIM to use for SMS. The selected slot is stored in app settings and applied to every outgoing message. Teachers typically use their personal teaching number rather than their primary personal number for this.

---

## Queue Cleanup

Sent messages are removed from the queue after seven days. Failed and cancelled messages are removed after thirty days. This prevents the queue table from growing indefinitely without losing any meaningful audit history.

---

## What Is Not Built

The SMS system uses Android's built-in SmsManager — it sends messages as standard SMS through the device SIM. There is no WhatsApp integration, no cloud SMS gateway, no delivery receipt tracking, and no scheduled future messages outside the queue system. These were deliberately excluded to keep the system simple, free to use, and independent of third-party services.

---

## Related Features

- [Student Management](./student_management.md) — Parent phone number is required for any SMS to be sent
- [Attendance Tracking](./attendance_tracking.md) — Absent alerts and monthly summaries originate here
- [Fee Management](./fee_management.md) — Fee reminders and payment confirmations originate here
- [Exam Tracking](./exam_tracking.md) — Exam result notifications originate here
