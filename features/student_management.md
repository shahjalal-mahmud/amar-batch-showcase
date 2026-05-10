# Student Management

> The foundation of everything else. Every attendance record, fee payment, and exam result belongs to a student.

---

## What This Feature Does

Student Management handles the complete lifecycle of a student inside a teacher's batch — from the moment they are admitted to the moment they leave, and everything in between.

---

## Core Capabilities

### Admitting Students

Students can be admitted one at a time through a simple form, or in bulk by uploading a CSV file. The minimum information required to admit a student is their name, their parent's phone number, and which class and shift they belong to.

Optional fields include the student's own phone number, their school name, a one-time admission fee, and the month from which their tuition fees should start being tracked.

Bulk CSV admission intentionally does not send welcome SMS — sending dozens of messages at once during a mass import is disruptive. Individual admission does trigger a welcome SMS if the teacher has enabled it.

### Class & Shift Assignment

Every student must belong to a class and a shift. Students cannot exist without this assignment. When a teacher creates a class without adding shifts, a default shift is created silently in the background so student data always has a valid home.

Teachers can move a student from one shift to another at any time. The historical attendance and fee records stay intact — only future events are affected by the new assignment.

### Promotion to a New Year

At the end of an academic year, teachers select which students to promote. A new class is created for the new year, and the selected students are reassigned to it in a single atomic operation. If the operation fails for any reason, no students are moved — promotion either completes fully or not at all.

Students who are not promoted — those who leave, repeat a year, or simply are not continuing — can be deactivated instead.

### Deactivating and Reactivating Students

When a student stops attending, the teacher marks them inactive. The student disappears from all active lists — attendance sheets, fee collection screens, exam entry — but every historical record is preserved exactly as it was. Past attendance, fee payments, and exam marks remain visible and reportable.

Inactive students are accessible from a dedicated screen. They can be reactivated with a single tap if the student returns.

The app never hard-deletes a student. This is a deliberate design decision — data that has been built up over months must not be destroyable by accident.

---

## Validation Rules

The app enforces a small set of rules before admitting a student:

- Name cannot be blank
- Parent phone must be an 11-digit Bangladeshi number starting with 01
- The selected shift must belong to the selected class
- The same student (matching name and parent phone) cannot be admitted twice as an active student

Phone validation exists specifically to prevent silent SMS failures. An SMS sent to a malformed number fails without any obvious error — catching it at admission time is far better than discovering it later.

---

## Fee Exemption

Individual students can be marked as fee-exempt. Exempt students remain fully tracked for attendance and exams but are excluded from all fee due calculations and fee reminder SMS. Teachers use this for students who receive free tuition.

---

## Search

Students can be searched by name or by parent phone number. Search is case-insensitive and supports partial matches — typing "ran" finds "Rana" and "Rania". Only active students appear in search results by default.

---

## Data Preserved When a Student Leaves

When a student is deactivated, the following history is never touched:

- All attendance sessions they were part of
- All fee payments they made
- All exam results recorded under their name

This means the teacher can always look back at a past student's complete record, even years later.

---

## Related Features

- [Attendance Tracking](./attendance_tracking.md) — Full session and record history is backed up
- [Fee Management](./fee_management.md) — Every payment record is backed up
- [Exam Tracking](./exam_tracking.md) — All exam and result data is backed up
- [SMS System](./sms_system.md) — Absence alerts and monthly summaries are sent through the SMS queue
- [Backup Restore](./backup_restore.md) — Covers automatic backup, what gets backed up, the restore flow on a new device
