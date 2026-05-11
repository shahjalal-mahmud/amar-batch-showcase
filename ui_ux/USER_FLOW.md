# User Flow — Amar Batch

> Step-by-step journeys through the app, from first launch to daily use.

---

## Table of Contents

- [1. First Launch](#1-first-launch)
- [2. Daily Routine](#2-daily-routine)
- [3. Setting Up Classes and Shifts](#3-setting-up-classes-and-shifts)
- [4. Adding Students](#4-adding-students)
- [5. Taking Attendance](#5-taking-attendance)
- [6. Collecting Fees](#6-collecting-fees)
- [7. Managing Exams](#7-managing-exams)
- [8. Sending SMS](#8-sending-sms)
- [9. End of Month](#9-end-of-month)
- [10. New Academic Year](#10-new-academic-year)
- [11. Restore on a New Phone](#11-restore-on-a-new-phone)

---

## 1. First Launch

```
Open App
    │
    ├── New User
    │       │
    │       ▼
    │   Enter name, phone number, batch name
    │       │
    │       ▼
    │   Sign in with Google (for backup)
    │       │
    │       ▼
    │   Default batch created automatically
    │       │
    │       ▼
    │   → Dashboard (empty, ready to set up)
    │
    └── Existing User (reinstall / new phone)
            │
            ▼
        Sign in with Google
            │
            ▼
        Restore from Google Drive
            │
            ▼
        → Dashboard (all previous data restored)
```

After first launch as a new user, the teacher sees an empty dashboard with a prompt to create their first class.

---

## 2. Daily Routine

The typical flow a teacher follows on any teaching day:

```
Open App
    │
    ▼
Select Class → Select Shift
    │
    ▼
Take Attendance (under 1 minute)
    │
    ▼
Save → Absent SMS sent to parents automatically (if enabled)
    │
    ▼
If any student pays fees today:
    │
    ▼
Collect Fee → Confirmation SMS sent (if enabled)
    │
    ▼
Close App
```

---

## 3. Setting Up Classes and Shifts

Done once at the start, or whenever a new class is added.

```
Settings → Classes
    │
    ▼
Create Class
    ├── Class Name (e.g. "Class 9 English")
    ├── Academic Year (e.g. 2026)
    └── Monthly Fee (e.g. ৳800)
    │
    ▼
Default Shift created automatically (hidden)
    │
    ├── Teacher teaches one group → done, add students directly
    │
    └── Teacher teaches multiple groups at different times
            │
            ▼
        Create Shift inside the Class
            ├── Shift Name (e.g. "Morning", "Evening")
            ├── Days of week (optional)
            └── Time (optional)
            │
            ▼
        Create more shifts as needed
```

A class always has at least one shift. If the teacher never creates a custom shift, the default shift works invisibly — students just belong to the class.

---

## 4. Adding Students

### Single Admission

```
Add Student
    │
    ▼
Fill required fields:
    ├── Student Name
    ├── Select Class
    ├── Select Shift
    └── Parent's Phone Number
    │
    ▼
Fill optional fields (if available):
    ├── Student's Phone
    ├── School Name
    ├── Admission Fee
    └── Starting Month (first month fees are tracked from)
    │
    ▼
Save
    │
    ▼
Welcome SMS sent to parent (if admission SMS enabled)
    │
    ▼
Student appears in attendance sheets and fee tracking
```

### Bulk Admission (CSV)

```
Students → CSV Import → Download Template
    │
    ▼
Fill template in Excel or Google Sheets
(Name, Class, Shift, Parent Phone — minimum required)
    │
    ▼
Upload CSV file → Preview → Import
    │
    ▼
All valid students added in one operation
(No welcome SMS sent for bulk imports)
```

---

## 5. Taking Attendance

### Default Present Mode (most students attend)

```
Attendance → Select Class → Select Shift
    │
    ▼
All students shown with green tick (present)
    │
    ▼
Tap only the absent students → they turn red
    │
    ▼
Save
    │
    ▼
Session recorded → Calendar shows green dot for today
Absent SMS queued for each absent student (if enabled)
```

### Default Absent Mode (many students absent)

```
Attendance → Select Class → Select Shift
    │
    ▼
Toggle to "Default Absent" mode
    │
    ▼
All students shown as absent (red)
    │
    ▼
Tap only the present students → they turn green
    │
    ▼
Save
```

### Editing Past Attendance

```
Attendance → Select Class → Select Shift → Select Date
    │
    ▼
Existing session loads with current records
    │
    ▼
Make corrections → Save
    │
    ▼
Records updated atomically (no SMS sent for edits)
```

---

## 6. Collecting Fees

### Single Month Payment

```
Fees → Select Class → Select Shift
    │
    ▼
Student list shows who has paid and who has not
    │
    ▼
Tap student → "Collect Fee"
    │
    ▼
Select month → Amount auto-fills from class fee
    │
    ▼
Adjust amount if needed → Save
    │
    ▼
Payment recorded → Fee Confirmation SMS sent (if enabled)
```

### Multi-Month Payment

```
Fees → Select student → "Collect Fee"
    │
    ▼
Select multiple months (e.g. January, February, March)
    │
    ▼
Total auto-calculates → Save
    │
    ▼
Separate row created per month in the database
Confirmation SMS sent
```

### Sending Fee Reminders

```
Fees → Due List
    │
    ▼
See all students with outstanding months
    │
    ├── Single student → tap SMS icon → message previews → Send
    │
    └── All due students → "Send Reminder to All" → confirm → Send
            │
            ▼
        Reminders queued and sent one by one in background
```

---

## 7. Managing Exams

### Creating an Exam and Entering Marks

```
Exams → Create Exam
    │
    ▼
Fill details:
    ├── Exam Name (e.g. "Monthly Test 1")
    ├── Class and Shift
    ├── Date
    ├── Total Marks
    └── Google Drive link (optional — for exam paper)
    │
    ▼
Exam created → result placeholders generated for all active students
    │
    ▼
Open Exam → Enter marks for each student
    │
    ├── Present → enter mark (must be ≤ total marks)
    └── Absent → check absent box → mark stays blank
    │
    ▼
Save → Results stored atomically
    │
    ▼
Exam Result SMS sent to parents (if enabled)
```

### Viewing Results

```
Open Exam
    │
    ▼
See summary:
    ├── Highest mark
    ├── Lowest mark
    ├── Class average
    └── Present / absent count
    │
    ▼
Tap any student → see their individual result history
```

---

## 8. Sending SMS

### How SMS Reaches the Parent

```
Any event (absent / fee collected / exam saved / reminder triggered)
    │
    ▼
SMS entry written to queue (database)
    │
    ▼
Background worker picks up queue
    │
    ▼
Worker checks: global SMS enabled? → class SMS type enabled? → parent phone valid?
    │
    ├── All yes → send via device SIM → mark Sent
    └── Any no  → skip silently (no error to teacher)
```

### Customizing Message Templates

```
Settings → SMS Templates
    │
    ▼
Select template type (e.g. Fee Reminder)
    │
    ▼
Edit message text
Use placeholders: {student_name}, {month}, {amount}, {teacher_name}, etc.
    │
    ▼
Save → new wording used for all future messages of that type
    │
    ▼
Reset to default anytime
```

### Enabling or Disabling SMS per Class

```
Settings → Classes → Select Class → SMS Settings
    │
    ▼
Toggle each type independently:
    ├── Absent Alert
    ├── Fee Reminder
    ├── Fee Confirmation
    ├── Exam Result
    ├── Admission Welcome
    └── Monthly Summary
```

---

## 9. End of Month

```
Check fee collection report → see who paid, who didn't
    │
    ▼
Send fee reminders to all unpaid students
    │
    ▼
Send monthly attendance summary SMS to all parents (optional)
    │
    ▼
Add any expenses for the month (chair rent, materials, etc.)
    │
    ▼
View profit calculation:
    Total Collected − Total Expenses = Net Profit
    │
    ▼
Manual backup (optional — auto backup also runs in background)
```

---

## 10. New Academic Year

At the start of each year, the teacher has two options:

### Option A — Same students continue (most move up)

```
Settings → Classes → Create New Class
    (e.g. "Class 10 English - 2027")
    │
    ▼
Create shifts inside new class
    │
    ▼
Students → Select students to promote → Promote
    │
    ▼
Students reassigned to new class and shift atomically
All historical records (attendance, fees, exams) remain intact
```

### Option B — Mostly new students

```
Mark old students inactive one by one
(or deactivate entire class)
    │
    ▼
Create new class for the year
    │
    ▼
Add new students (manual or CSV)
```

---

## 11. Restore on a New Phone

```
Install app on new phone
    │
    ▼
Open App → "Existing User"
    │
    ▼
Sign in with same Google account
    │
    ▼
App fetches teacher profile and license from server
    │
    ▼
App downloads academic data backup from Google Drive
    │
    ▼
Validation: check backup is not empty or corrupt
    │
    ▼
Restore: clear local database → insert all backup data → verify counts
    │
    ▼
→ Dashboard with all data exactly as it was on last backup
```

If no backup exists on Drive (first phone, never backed up), the teacher starts fresh with their profile and license intact but no academic history.

---

## Screen Navigation Summary

```
Bottom Navigation Bar
    │
    ├── Dashboard   → monthly summary, quick stats
    ├── Students    → full list, search, profiles
    ├── Attendance  → mark present/absent, calendar view
    ├── Fees        → collect payments, view dues
    └── Exams       → create tests, enter marks, view results

Top-level Settings (via menu)
    ├── Profile
    ├── Classes & Shifts
    ├── SMS Settings & Templates
    ├── Backup & Restore
    └── License Info
```

---

_Last Updated: May 2026_
_Developer: MD SHAHAJALAL MAHMUD — [mahmud.nubtk@gmail.com](mailto:mahmud.nubtk@gmail.com)_
