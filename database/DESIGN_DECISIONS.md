# Database Design Decisions — Amar Batch

> **Why the database is designed the way it is.**

Every significant design choice in this schema was made deliberately. This document explains the _reasoning_ behind those choices — the tradeoffs considered, the alternatives rejected, and the real-world constraints that drove each decision.

---

## Table of Contents

- [Database Design Decisions — Amar Batch](#database-design-decisions--amar-batch)
  - [Table of Contents](#table-of-contents)
  - [1. Why SQLite (Room) Instead of a Cloud Database](#1-why-sqlite-room-instead-of-a-cloud-database)
  - [2. Why Soft Deletes Instead of Hard Deletes](#2-why-soft-deletes-instead-of-hard-deletes)
  - [3. Why Timestamps Are Stored as Epoch Milliseconds](#3-why-timestamps-are-stored-as-epoch-milliseconds)
  - [4. Why Fee Month Is Stored as YYYYMM Integer](#4-why-fee-month-is-stored-as-yyyymm-integer)
  - [5. Why Fee Transactions Store Snapshots of Class and Amount](#5-why-fee-transactions-store-snapshots-of-class-and-amount)
  - [6. Why Classes Are Tied to Academic Years](#6-why-classes-are-tied-to-academic-years)
  - [7. Why Every Class Always Has at Least One Shift](#7-why-every-class-always-has-at-least-one-shift)
  - [8. Why the Default Shift Is Hidden](#8-why-the-default-shift-is-hidden)
  - [9. Why Attendance Is Split Into Sessions and Records](#9-why-attendance-is-split-into-sessions-and-records)
  - [10. Why Multi-Month Fee Payments Create Separate Rows](#10-why-multi-month-fee-payments-create-separate-rows)
  - [11. Why There Are No Partial Payments or Discounts](#11-why-there-are-no-partial-payments-or-discounts)
  - [12. Why Expenses Are a Flat List](#12-why-expenses-are-a-flat-list)
  - [13. Why SMS Goes Through a Queue Table Instead of Sending Directly](#13-why-sms-goes-through-a-queue-table-instead-of-sending-directly)
  - [14. Why SMS Settings Are Per Class, Not Per Student](#14-why-sms-settings-are-per-class-not-per-student)
  - [15. Why the License Table Lives Inside the App Database](#15-why-the-license-table-lives-inside-the-app-database)
  - [16. Why DAOs Are Grouped by Domain, Not by Table](#16-why-daos-are-grouped-by-domain-not-by-table)
  - [17. Why There Are 17 Tables and Not Fewer](#17-why-there-are-17-tables-and-not-fewer)
  - [18. What Was Deliberately Left Out](#18-what-was-deliberately-left-out)

---

## 1. Why SQLite (Room) Instead of a Cloud Database

The target users are private tutors in Bangladesh, many of whom teach in areas with unreliable or expensive mobile data. The single most important product requirement is that the app must work completely offline, every single day, without any degradation.

A cloud-first database (Firestore, Supabase, etc.) would mean the app breaks when the internet does. Even with aggressive local caching, cloud-first architectures introduce sync conflicts, stale reads, and loading states that slow down a workflow that should take under one minute per day.

Room over SQLite gives us a local database that is always fast, always consistent, and always available. Google Drive is used purely as a backup destination — the teacher's data is never _dependent_ on it being reachable.

The tradeoff is that there is no real-time sync between devices, and restoring data requires a manual step. For a single teacher on a single phone, this tradeoff is entirely acceptable.

---

## 2. Why Soft Deletes Instead of Hard Deletes

For the entities that matter most — students, classes, and shifts — we never delete rows from the database. Instead, we set an `is_active` flag to false.

The reason is historical integrity. A student who stopped coming to class six months ago still has attendance records, fee payment history, and exam results from when they were active. If that student's row were hard-deleted, all of those records would either become orphaned (violating referential integrity) or cascade-deleted (destroying history the teacher may need for reference or dispute resolution).

Soft deletes mean the past is always preserved. Inactive students simply disappear from daily screens — attendance lists, fee collection screens, exam entry — but their history remains intact and reportable.

This also protects teachers against accidents. Marking a student inactive is reversible with one tap. Hard deletion is not.

The entities that _do_ allow hard deletes — fee transactions and expenses — have no downstream dependents and represent correctable mistakes. A teacher who records a fee payment twice should be able to undo that. But attendance history cannot be "undone."

---

## 3. Why Timestamps Are Stored as Epoch Milliseconds

All dates and times in the schema are stored as `bigint` values representing epoch milliseconds (the number of milliseconds since January 1, 1970 UTC).

The main reason is timezone safety. Bangladesh is UTC+6, but Android devices can have their timezone changed, and SQLite has no native timezone support. If we stored timestamps as formatted strings ("2026-01-15 08:30") or as SQLite's TEXT dates, the values would be ambiguous when a device's timezone setting changes. Epoch milliseconds are always absolute — they represent the same instant in time regardless of where the device is.

Epoch milliseconds also make arithmetic trivial. Comparing whether one date is before another, calculating the difference between two dates, and sorting records chronologically all work as simple integer comparisons. There is no string parsing, no date library calls, and no room for formatting bugs.

The slight downside is that epoch values are not human-readable in raw database queries. We accept this tradeoff because the app's Kotlin code handles all display formatting, and raw database inspection is a developer concern, not a user concern.

---

## 4. Why Fee Month Is Stored as YYYYMM Integer

Fee months (and expense months, and student start months) are stored as integers in YYYYMM format. For example, January 2026 is stored as `202601`.

The alternative was to store the first day of each month as a full epoch timestamp. We rejected this because it introduces timezone ambiguity — midnight on January 1, 2026 in Bangladesh is a different epoch value than midnight on January 1, 2026 in UTC, and a device timezone change could cause a January payment to appear as a December payment in a display query.

YYYYMM integers have no timezone component at all. January 2026 is always `202601` regardless of the device's timezone setting. Sorting works correctly because the integers are ordered the same way the months are. Comparing whether a month is before or after another month is a simple integer comparison. Generating a sequence of months from a start month to the current month is straightforward arithmetic.

The format is also compact (6 digits), easy to read in database inspection, and avoids the overhead of parsing and formatting date strings.

---

## 5. Why Fee Transactions Store Snapshots of Class and Amount

Each fee transaction stores the `class_id`, `shift_id`, and `expected_amount` at the time the payment was recorded — even though all three of those can be looked up from the student record.

This is the snapshot principle, and it exists to protect historical records from being corrupted by future changes.

Consider what happens without snapshots. A teacher charges ৳1,000/month in 2026 and records twelve months of payments. In 2027 they raise the fee to ৳1,200. Without snapshots, a report of 2026 payments would now show the wrong expected amount. The teacher would not be able to verify that students had paid the correct amount for their time.

Or consider a student who moves from one shift to another mid-year. Without snapshots, past fee records would appear to be for the new shift, which had a different fee rate.

With snapshots, past records are frozen at the moment of payment. They reflect the state of the world at that time, not the current state. This is the same principle used in financial accounting — once a transaction is recorded, it is immutable.

---

## 6. Why Classes Are Tied to Academic Years

Each class record includes an `academic_year` field. This means "Class 9" in 2025 and "Class 9" in 2026 are two separate rows in the database.

The reason is that promoting students from one year to the next is a common and important operation. When a teacher's Class 9 students finish their year and move to Class 10, the teacher needs to create new class and shift records for the new year and reassign students to them.

If classes were year-agnostic, promoting students would mean modifying the same class record, which would retroactively change the class association on all past attendance and fee records. A student's January attendance would appear to be in "Class 10" even though they were in "Class 9" at the time.

By tying classes to academic years, we ensure that historical records always reflect which class a student was actually in when those records were created. The past cannot be accidentally rewritten by a present-day restructuring.

---

## 7. Why Every Class Always Has at Least One Shift

Students must belong to a shift, not directly to a class. This was a deliberate structural decision made to support teachers who teach the same class at different times of day.

A teacher might have Class 9 students in the morning and a different group of Class 9 students in the evening. Without shifts, there is no clean way to separate these two groups while keeping them associated with the same class structure.

However, most teachers — especially those just getting started — do not need multiple shifts at all. Forcing them to create and name a shift before they can add students would be a barrier to adoption.

The solution is the default shift. When a class is created, a shift is automatically created alongside it. The teacher never sees or interacts with this default shift — from their perspective, they just created a class and can start adding students. Only when a teacher explicitly creates a second shift does the concept of shifts become visible in the UI.

This gives every teacher a consistent underlying data model (all students belong to shifts) without exposing complexity to teachers who do not need it.

---

## 8. Why the Default Shift Is Hidden

The default shift has a flag `is_default = true`. The UI uses this flag to decide whether to show shift names at all.

When a class has only one shift and that shift is the default, the app behaves as if shifts do not exist. The teacher sees "Class 9" with a list of students. There is no mention of shifts.

When a teacher creates a second shift, the app reveals the shift layer. Both shifts become visible, students can be distributed between them, and attendance is tracked separately per shift.

This progressive disclosure keeps the app simple for simple use cases while remaining fully capable for complex ones. A teacher with one group of students per class should never have to think about a concept called "shift."

---

## 9. Why Attendance Is Split Into Sessions and Records

Attendance data is stored in two tables rather than one. `attendance_sessions` holds the metadata for each attendance-taking event (which class, which shift, which date). `attendance_records` holds the individual present/absent status for each student in that session.

The main reason for the split is that sessions and records have different cardinalities and different access patterns. A session is a single event that might have 30 associated records. Queries for "which days did this class take attendance this month" only need to touch the sessions table — there is no reason to join or scan thousands of individual records for that question.

The calendar view, for example, shows green dots on dates where attendance was taken. That query only needs `attendance_sessions`. A query for an individual student's attendance history only needs `attendance_records`. The split allows each query to be efficient and precise.

The split also makes the unique constraint clean and simple: one session per `(class_id, shift_id, session_date)`. This prevents a teacher from accidentally creating duplicate attendance for the same class on the same day.

---

## 10. Why Multi-Month Fee Payments Create Separate Rows

When a teacher collects fees for multiple months in a single transaction — for example, a student paying January, February, and March at once — the app creates three separate rows in `fee_transactions`, one per month.

The alternative was to store a single row with a comma-separated list of months or a JSON array. We rejected this for several reasons.

First, the unique constraint `(student_id, fee_month)` only works if each month is its own row. This constraint prevents a teacher from accidentally recording two payments for the same student for the same month. You cannot enforce that constraint on a comma-separated field.

Second, querying "which students have not paid for March" is a simple set-difference query when each month is a row. It becomes a string parsing problem if months are packed into a single field.

Third, individual months can be corrected independently. If a teacher records three months at once and then realizes one of the months was a mistake, they can delete just that one row without affecting the other two.

The tradeoff is a slightly more complex insert operation — the app must loop and insert multiple rows. This is handled in a single atomic transaction, so from the teacher's perspective it is still one action.

---

## 11. Why There Are No Partial Payments or Discounts

The fee schema has no field for partial payments, discounts, waivers, or late fees. The teacher collects the full month's fee or nothing.

This was a deliberate simplification. Private tutors in Bangladesh typically negotiate fee adjustments informally and verbally. Building a system to track every possible fee variation — sibling discounts, hardship waivers, partial month fees, late penalties — would make the fee collection screen significantly more complex and would cover scenarios that most teachers encounter rarely or never.

The app does allow the `amount_paid` field to differ from the `expected_amount` field, which gives teachers a manual escape hatch. If a teacher wants to accept a partial payment, they can simply enter the amount that was actually paid. The difference is visible in reports. But the app does not try to model or automate the logic behind why the amounts differ.

Simplicity here is a product feature. Teachers should be able to record a fee payment in five seconds. Adding discount logic would slow that down for the majority of teachers who never use it.

---

## 12. Why Expenses Are a Flat List

The expenses table has three fields that matter: a title, an amount, and a month. There are no categories, no tags, no recurring expense templates, and no sub-items.

Most teachers track a small number of monthly costs: perhaps a room rental, a whiteboard marker purchase, or a printer expense. They do not need a double-entry accounting system or an expense categorization hierarchy. They need to subtract their costs from their collected fees to know whether they made money this month.

A flat list satisfies that need with the minimum possible complexity. Adding categories would require teaching the teacher a taxonomy. Adding recurring expenses would require scheduling logic. Neither adds enough value to justify the added complexity.

If a teacher has a recurring expense — like a monthly room rent — they enter it manually each month. This takes ten seconds and keeps the expense table simple and understandable.

---

## 13. Why SMS Goes Through a Queue Table Instead of Sending Directly

When something triggers an SMS — a student is marked absent, a fee is collected, an exam result is saved — the app does not send the SMS immediately. Instead it inserts a row into the `sms_queue` table, and a background WorkManager job processes that queue independently.

There are two reasons for this architecture.

The first is reliability at the Android layer. Android's SmsManager can fail silently, return errors, or be throttled by the device or carrier if messages are sent too quickly in succession. By queuing messages and sending them one at a time with a delay between each, we avoid triggering spam detection and give each message a proper retry opportunity if it fails.

The second reason is atomicity. When a teacher saves attendance for 30 students and 5 are absent, we want either all 5 absence SMS entries to be enqueued or none of them — never a partial state where some parents are notified and others are not. Because the queue entries are inserted inside the same database transaction as the attendance records, they succeed or fail together. The SMS worker then processes them asynchronously, decoupling the sending behavior from the saving behavior.

This also means the teacher's UI never has to wait for SMS sending. The attendance screen saves instantly. The SMS happens in the background.

---

## 14. Why SMS Settings Are Per Class, Not Per Student

The `sms_settings` table has one row per class. Teachers can turn different SMS types on or off for each class independently.

The alternative was to have per-student SMS settings. We rejected this because it creates too much management burden. A teacher with 60 students should not have to configure SMS preferences 60 times. Class-level settings let them make one decision ("I want absent notifications for my Class 9 students but not for my Class 10 students") that applies to everyone in that class.

Individual student SMS is still effectively controlled through the `parent_phone` field. A student with no parent phone number never receives SMS, regardless of what the class settings say. This provides a natural per-student override without requiring a settings screen per student.

The global `sms_global_enabled` flag in `app_settings` acts as a master switch. If SMS is globally disabled, nothing is sent regardless of class settings. This lets teachers temporarily pause all SMS — during a holiday, or while they are troubleshooting — with a single toggle.

---

## 15. Why the License Table Lives Inside the App Database

License and trial information is stored in a table inside the local Room database, not fetched exclusively from a remote server.

The reason is the offline-first requirement. If the app required a live server connection to validate the license on every launch, it would fail to open for teachers in areas with no internet. That would be catastrophic — teachers rely on this app during class, which may be in a building with no signal.

The local license table stores the `expiry_date` as a cached value. When the device is online, the app fetches the current license status from Firebase and updates this local cache. When the device is offline, it falls back to the cached value. This means a teacher whose license is valid can always open the app, even without internet.

The device ID binding prevents the local record from being easily manipulated — the license is tied to the hardware identifier of the phone, not just to an account.

The tradeoff is that revoking a license is not instant if the teacher is offline. The grace period (3 days by default) is a deliberate buffer that acknowledges this. A teacher will eventually come online, the app will check the server, and the block will take effect.

---

## 16. Why DAOs Are Grouped by Domain, Not by Table

The app has 17 database tables but organizes its DAOs into 10 groups: `AuthDao`, `BatchDao`, `ClassStructureDao`, `StudentDao`, `AttendanceDao`, `FeeDao`, `ExamDao`, `ExpenseDao`, `SmsDao`, and `SettingsDao`.

If DAOs were one-to-one with tables, there would be 17 DAO interfaces. This creates two problems. First, ViewModels and repositories would need to inject many more dependencies. Second, operations that naturally span two tables in the same domain (like inserting an attendance session and its records together) would require coordinating across two separate DAO classes.

Grouping by domain means `AttendanceDao` handles both `attendance_sessions` and `attendance_records`. A single `@Transaction` method in that DAO can atomically insert both. The repository that uses `AttendanceDao` sees one coherent interface for all attendance operations rather than two disconnected pieces.

This grouping mirrors how the app thinks about its data — in terms of features (attendance, fees, exams) rather than in terms of table names.

---

## 17. Why There Are 17 Tables and Not Fewer

A first instinct when looking at 17 tables might be to ask whether some of them could be merged to simplify the schema. A few alternatives were considered and rejected.

Merging `attendance_sessions` and `attendance_records` into one table would mean storing redundant class, shift, and date information on every individual record. For a class of 30 students, that is 30 copies of the same session metadata per day. Worse, it would make the "which days did this class take attendance" query require a GROUP BY with a DISTINCT scan rather than a simple read from a small table.

Merging `sms_settings` into the `classes` table was considered. The reason to keep them separate is that `sms_settings` has a different lifecycle and access pattern from class metadata. A settings query fetches one row by class ID. A class listing query fetches many rows and needs name, fee, and year — it does not need six boolean SMS flags for each row. Keeping them separate keeps the classes table lean.

Merging `sms_templates` into `app_settings` was also considered. Templates are a fixed set of six customizable strings with their own type system. Embedding them as columns in `app_settings` would mean a single very wide row with poorly named columns. A separate table with type-indexed rows is cleaner and extensible.

Each of the 17 tables exists because it represents a genuinely distinct entity with its own cardinality, access patterns, and lifecycle.

---

## 18. What Was Deliberately Left Out

Some decisions are about what the schema does _not_ include. These omissions are just as intentional as what was included.

**No student promotion history table.** When students are promoted from one class to another, their `class_id` and `shift_id` are updated in place. There is no audit trail of which class a student was in on which date. The rationale is that fee transactions already snapshot `class_id` and `shift_id` at payment time, so historical financial records are accurate. Attendance records are linked through the session, which is linked to a specific class and shift. The full history is implicit in the operational records.

**No teacher-to-student messaging history.** SMS templates and the queue track outbound messages, but once a message is sent it is cleaned up after a short retention period. There is no inbox, no conversation thread, and no permanent log of what was sent to which parent. This keeps the database lean and avoids creating a communication record that the teacher has no use for after the fact.

**No multi-teacher support in the schema.** The `teachers` table is designed for one teacher per installation. While the schema has a `teacher_id` on batches (enabling future expansion), the current app assumes a single teacher. Adding multi-teacher support would require authentication changes, permission scoping on every query, and UI changes throughout — it is correctly deferred.

**No ranking or analytics tables.** There are no precomputed leaderboard tables, no grade distribution snapshots, and no trend tables. All reporting in the app is computed at query time from the operational tables. For the data volumes expected (see the estimates in the schema overview), this is fast enough and keeps the schema from accumulating derived data that could become stale or inconsistent.

**No notification scheduling table.** Fee reminders and monthly summaries are triggered manually by the teacher or on a simple schedule, not by a complex rule engine. There is no table that stores "send a reminder on the 5th of every month if the student has not paid." The simplicity of the trigger logic means it can live entirely in the application layer without needing its own persistence.

---

_Last Updated: May 2026_
_Developer: MD SHAHAJALAL MAHMUD — [mahmud.nubtk@gmail.com](mailto:mahmud.nubtk@gmail.com)_
