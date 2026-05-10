# Fee Management

> The feature that prevents the most common source of tension between teachers and parents — "I already paid" disputes.

---

## What This Feature Does

Fee Management tracks monthly tuition payments for every student, shows who has paid and who has not, calculates outstanding dues, and coordinates fee reminder and confirmation SMS.

---

## Core Capabilities

### Collecting a Fee Payment

The teacher selects a class and shift, then chooses a student. The app automatically calculates which months are due — from the student's start month to the current month — and shows the expected amount based on the shift's fee.

The teacher selects the month or months being paid. For a single month, one record is saved. For multiple months paid at once, a separate record is created per month. This makes each payment individually visible, individually correctable, and individually reportable.

The amount shown is the expected fee, but the teacher can adjust it before confirming. If a teacher accepts a different amount — less or more — that actual amount is saved alongside the expected amount. The difference is visible in reports.

### Fee Structure by Class and Shift

Each class has a default monthly fee. All shifts inside that class inherit this fee automatically. If a specific shift warrants a different amount — perhaps an advanced group or a smaller batch — the teacher can override that shift's fee independently. Once overridden, that shift's fee stays separate and is no longer affected by changes to the class default.

When the class default fee is updated, all shifts that have not been individually overridden are updated together in a single atomic operation.

### Snapshot Principle

Every fee transaction records not just the amount but also the class ID, shift ID, and expected amount at the moment of payment. This means past records never change even if the student later moves to a different shift, or the teacher changes the fee structure. January's payment always reflects what January's fee actually was.

### Editing and Deleting Payments

Individual fee transactions can be edited or deleted with teacher confirmation. Deletion is allowed because fee entry mistakes happen — a teacher recording the same month twice should be able to fix that. Deleting a fee record has no side effects on attendance or exam data.

### Viewing What Is Due

The fee screen shows each student's outstanding months at a glance. The calculation is derived at query time: expected months since the student's start month, minus months that have a payment record. There is no running balance stored anywhere — the due amount is always computed fresh from actual data.

Summary figures available at the class level include total collected for the current month, total outstanding from previous months, and a lifetime outstanding figure across all students.

### Fee Reminders

Teachers can trigger fee reminder SMS for all unpaid students in a given month. The app queries for active, non-exempt students who do not have a payment record for that month and enqueues a reminder message for each one. This does not modify any financial data — it only writes to the SMS queue.

---

## Fee Exemption

Students marked as fee-exempt are excluded from all due calculations and reminder SMS. Their attendance and exam tracking continue normally. Teachers use this for students who receive free tuition — the student remains fully tracked without creating phantom dues.

---

## Profit Calculation

The app tracks expenses separately as a flat list of amounts tied to a month. The profit for any month is the total collected minus the total expenses for that month. This calculation lives in the repository layer, not the database — no derived figures are stored.

---

## What Is Deliberately Not Built

There are no partial payment fields, no discount or waiver tracking, no late fee penalties, and no installment plans. These were evaluated and rejected. The teachers this app serves negotiate exceptions verbally — they do not need a system to model every possible variation. The `amount_paid` field provides a manual escape hatch for any one-off adjustment.

---

## Related Features

- [Student Management](./student_management.md) — All student data is included in the backup
- [Attendance Tracking](./attendance_tracking.md) — Full session and record history is backed up
- [Exam Tracking](./exam_tracking.md) — All exam and result data is backed up
- [SMS System](./sms_system.md) — Absence alerts and monthly summaries are sent through the SMS queue
- [Backup Restore](./backup_restore.md) — Covers automatic backup, what gets backed up, the restore flow on a new device
