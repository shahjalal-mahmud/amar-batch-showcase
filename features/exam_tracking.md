# Exam Tracking

> Academic records that teachers can show parents with confidence — no missing students, no invalid marks.

---

## What This Feature Does

Exam Tracking lets teachers create tests and exams, record marks for every student, and view per-exam statistics. Each exam is tied to a specific class and shift, and result records are auto-generated for every active student when the exam is created.

---

## Core Capabilities

### Creating an Exam

The teacher provides a name (for example, "Weekly Test 3" or "Monthly Exam"), a date, and the total marks possible. An optional Google Drive link can be attached — useful for storing the exam paper PDF for the teacher's own reference.

When the exam is saved, the app immediately generates a result placeholder for every active student in the selected shift. Each placeholder starts with a status of Absent and no mark. This guarantees that every student has a result row from the beginning — there are no missing entries to fill in later, and no risk of accidentally omitting a student.

### Entering Marks

The teacher opens the exam and sees the full list of students with their current status. For each student they mark as present, a numeric mark is entered. For students who did not take the exam, the absent status is retained.

The app enforces two rules on every mark: it cannot be negative, and it cannot exceed the exam's total mark. These rules catch data entry mistakes before they reach the database.

### Editing Results

Results can be corrected at any time. A teacher who entered the wrong mark for a student, or forgot to change a student from absent to present, can reopen the exam and fix it. When results are saved, all entries are updated together in a single atomic operation — a partial save is not possible.

### Exam Statistics

After marks are entered, the exam screen shows:

- The highest mark in the class
- The lowest mark among students who were present
- The average mark across present students
- How many students were present and how many were absent

These statistics are calculated from the actual result records at query time. Nothing is cached or pre-computed — the figures always reflect the current state of the data.

### Student Performance History

A student's profile shows their complete exam history — every exam they have participated in, their mark, and their percentage. This history is built from the result records and never affected by changes to the exam structure itself.

---

## What Cannot Be Changed After Creation

Exams cannot be deleted. This constraint exists because deleting an exam would either orphan its result records or cascade-delete them, destroying academic history. A teacher who creates an exam by mistake can edit its name, date, and total mark — but the record and its results stay in the database.

Individual result records also cannot be deleted. They can only be updated. A student who is marked absent can be changed to present with a mark, but their result row always exists.

If the total mark of an exam is lowered after marks have been entered, the app checks that no existing mark would exceed the new total. If any mark would become invalid, the update is rejected with a clear message — the teacher must correct those marks first.

---

## Drive Link

The optional Google Drive link on an exam is purely for the teacher's reference. It is not sent to parents or students. Teachers use it to store exam paper files so they can find them later from the app without searching their Drive manually.

---

## What Is Deliberately Not Built

There is no GPA system, no grade letter assignment, no ranking table, and no cross-exam trend analytics. These were evaluated and left out. The teachers this app serves want to know how students did on a test — they do not need an academic analytics engine. All statistics are simple aggregates computed on demand.

---

## Related Features

- [Student Management](./student_management.md) — Only active students get result placeholders when an exam is created
- [SMS System](./sms_system.md) — Exam results can be sent to parents via SMS after entry
