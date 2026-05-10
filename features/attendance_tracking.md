# Attendance Tracking

> The most frequently used feature in the app. Designed to take under one minute for an entire batch.

---

## What This Feature Does

Attendance Tracking lets teachers record which students were present or absent for each class session, view history on a calendar, and notify absent students' parents via SMS automatically.

---

## Core Capabilities

### Taking Attendance

The teacher selects a class and shift, and the full list of active students appears. Attendance is recorded with a single tap per student.

Two modes exist to match how different teachers think:

**Default Present mode** — assumes everyone is present. The teacher only taps the students who are absent. This suits classes where most students usually attend.

**Default Absent mode** — assumes everyone is absent. The teacher taps each student who is present. This suits smaller classes or sessions where turnout is unpredictable.

The teacher selects the mode that matches the day's situation. After marking, they tap Save — and the session is stored atomically. The session and all individual records succeed together or neither is saved.

### Taking Attendance for Past Dates

Teachers do not have to take attendance in real time. The date selector defaults to today but can be changed freely. A teacher can catch up on a session they forgot to record, or correct a day from last week.

### Editing Attendance

Any past session can be reopened and corrected. When attendance is edited, the old records are replaced entirely with the new ones in a single atomic update. Editing does not trigger absence SMS — notifications are only sent on the original save.

---

## How Data Is Structured

Each time attendance is taken, two things are created:

An **Attendance Session** records which class, which shift, and which date. There can only be one session per class-shift-date combination — trying to create a duplicate returns a clear error rather than silently overwriting.

An **Attendance Record** is created for each student in that session, storing whether they were present or absent. Records cannot be deleted individually — the only way to change past attendance is to edit the entire session.

This two-table structure keeps the calendar view fast (it only reads sessions, not individual records) while keeping student-level history detailed and accurate.

---

## Calendar View

Sessions are shown as visual indicators on a calendar. Tapping a date shows the session taken for that day. This gives teachers an at-a-glance view of how consistently attendance has been recorded over the month, and makes it easy to spot days that were missed.

---

## Monthly Attendance Summary

At the end of each month, teachers can trigger a summary SMS to all parents in a class. The message tells each parent how many classes their child attended out of the total sessions that month. This summary is calculated from the actual session and record data — no running totals are maintained separately.

---

## Absence Notifications

If absent SMS is enabled for a class, each student marked absent triggers a message to their parent's phone. This happens after the attendance is saved, not during — so a failure in the SMS queue never prevents attendance from being recorded.

---

## What Stays Fixed

Attendance sessions and records are never hard-deleted. This is intentional. Attendance history forms the foundation of monthly summaries, student profiles, and parent trust. Making it permanently deletable would undermine all three.

---

## Related Features

- [Student Management](./student_management.md) — Only active students appear on attendance sheets
- [SMS System](./sms_system.md) — Absence alerts and monthly summaries are sent through the SMS queue
