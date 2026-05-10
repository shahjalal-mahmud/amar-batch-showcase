# Data Flow Diagram — Amar Batch

> **How data moves through the app — from user action to database and back.**

This document traces the complete lifecycle of data in Amar Batch using text-based diagrams. Every major feature is covered: attendance, fees, exams, SMS, and backup.

---

## Table of Contents

- [Data Flow Diagram — Amar Batch](#data-flow-diagram--amar-batch)
  - [Table of Contents](#table-of-contents)
  - [1. Core Architecture Flow](#1-core-architecture-flow)
  - [2. Unidirectional Data Flow (UDF)](#2-unidirectional-data-flow-udf)
  - [3. Feature Flows](#3-feature-flows)
    - [3A. Onboarding \& Teacher Setup](#3a-onboarding--teacher-setup)
    - [3B. Student Admission](#3b-student-admission)
    - [3C. Daily Attendance](#3c-daily-attendance)
    - [3D. Fee Collection](#3d-fee-collection)
    - [3E. Exam Results](#3e-exam-results)
    - [3F. SMS Queue Processing](#3f-sms-queue-processing)
    - [3G. Google Drive Backup](#3g-google-drive-backup)
    - [3H. License Validation](#3h-license-validation)
  - [4. Database as Single Source of Truth](#4-database-as-single-source-of-truth)
  - [5. State Management Flow](#5-state-management-flow)
  - [6. Background Worker Lifecycle](#6-background-worker-lifecycle)
  - [7. Offline-First Data Strategy](#7-offline-first-data-strategy)
  - [Related Documentation](#related-documentation)

---

## 1. Core Architecture Flow

Data always flows top-to-bottom for writes and bottom-to-top (via reactive streams) for reads.

```
╔══════════════════════════════════════════════════════════════╗
║                         UI LAYER                             ║
║                     Jetpack Compose                          ║
║                                                              ║
║   ┌─────────────┐  ┌─────────────┐  ┌────────────────────┐   ║
║   │  Attendance │  │    Fee      │  │  Student / Exam /  │   ║
║   │   Screen   │  │   Screen   │  │  Dashboard / SMS   │     ║
║   └──────┬──────┘  └──────┬──────┘  └─────────┬──────────┘   ║
║          │  (User Actions) │                   │             ║
╚══════════╪═════════════════╪═══════════════════╪════════════╝
           │                 │                   │
           ▼                 ▼                   ▼
╔══════════════════════════════════════════════════════════════╗
║                      VIEWMODEL LAYER                         ║
║               StateFlow<UiState> + SharedFlow<Event>         ║
║                                                              ║
║   AttendanceViewModel   FeeViewModel   StudentViewModel ...  ║
║                                                              ║
║   • Holds UI state             • Calls Use Cases             ║
║   • Handles user actions       • Calls Repositories          ║
║   • viewModelScope coroutines  • Never holds Context/View    ║
╚══════════════════════╦═══════════════════════════════════════╝
                       ║
           ┌───────────╩──────────┐
           ▼                      ▼
╔═══════════════════╗   ╔═════════════════════════════════════╗
║   DOMAIN LAYER    ║   ║          DATA LAYER                 ║
║   (Use Cases)     ║   ║   (Repository Implementations)      ║
║                   ║   ║                                     ║
║ CalculateDue      ║   ║  StudentRepositoryImpl              ║
║ MonthsUseCase     ║   ║  FeeRepositoryImpl                  ║
║                   ║   ║  AttendanceRepositoryImpl           ║
║ CollectFeeUseCase ║   ║  SmsRepositoryImpl                  ║
║                   ║   ║                                     ║
║ Pure Kotlin —     ║   ║  Only layer that knows about        ║
║ zero Android      ║   ║  Room / Firebase / Drive            ║
║ dependencies      ║   ║                                     ║
╚═════════╦═════════╝   ╚═══════════════╦═════════════════════╝
          ║                             ║
          ╚═════════════╦═══════════════╝
                        ▼
╔══════════════════════════════════════════════════════════════╗
║                     LOCAL DATABASE                           ║
║                Room (SQLite) — 17 Tables                     ║
║                                                              ║
║    teachers │ batches │ classes │ shifts │ students          ║
║    attendance_sessions │ attendance_records                  ║
║    fee_transactions │ exams │ exam_results                   ║
║    sms_queue │ sms_templates │ sms_settings                  ║
║    expenses │ licenses │ app_settings │ backup_metadata      ║
║                                                              ║
║    ◉  Primary & only source of truth                         ║
║    ◉  Exposes reactive Flow<T> for live UI updates           ║
╚══════════════════════════╦═══════════════════════════════════╝
                           ║  (optional, when online)
                           ▼
              ┌────────────────────────┐
              │     REMOTE SERVICES    │
              │                        │
              │  Google Drive API      │
              │  (backup only)         │
              │                        │
              │  Firebase Auth         │
              │  (Google Sign-In)      │
              └────────────────────────┘
```

---

## 2. Unidirectional Data Flow (UDF)

The app enforces a strict one-way data flow. The UI never mutates state directly.

```
  ┌─────────────────────────────────────────────────────────┐
  │                    USER ACTION                          │
  │          (e.g. tap "Mark Absent", "Save Fee")           │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   VIEWMODEL                             │
  │      Public function called (e.g. saveAttendance())     │
  │      Launches coroutine in viewModelScope               │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │             USE CASE (if business logic)                │
  │      Validates input, coordinates multi-repo calls      │
  │      Returns Result<T>                                  │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   REPOSITORY                            │
  │      Converts domain model → entity                     │
  │      Calls DAO, wraps result in Result<T>               │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   ROOM (DAO)                            │
  │      Writes to SQLite                                   │
  │      Emits new value on observed Flow<T>                │
  └──────────────────────────┬──────────────────────────────┘
                             │
                     (Flow emission)
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   REPOSITORY                            │
  │      Maps entity → domain model                         │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   VIEWMODEL                             │
  │      Updates _uiState (StateFlow)                       │
  │      Emits one-time event via SharedFlow (if needed)    │
  └──────────────────────────┬──────────────────────────────┘
                             │
                             ▼
  ┌─────────────────────────────────────────────────────────┐
  │                    UI (Compose)                         │
  │      Recomposes with new UiState                        │
  │      Handles event (navigate, show Snackbar, etc.)      │
  └─────────────────────────────────────────────────────────┘
```

**Key rule:** At any point you can ask _"where does this state come from?"_ — the answer always traces back to Room.

---

## 3. Feature Flows

### 3A. Onboarding & Teacher Setup

New user registration is a single atomic transaction. Partial success is not allowed.

```
  App Launch (First Time)
         │
         ▼
  ┌──────────────────┐
  │  Check License   │  ──► No Teacher record found
  └──────────────────┘
         │
         ▼
  ┌──────────────────────────────────────────┐
  │          ONBOARDING SCREEN               │
  │  Google Sign-In → get account details    │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────┐
  │      @Transaction  (ATOMIC)              │
  │                                          │
  │  1. INSERT Teacher (name, phone, email)  │
  │  2. INSERT Batch (default batch)         │
  │  3. INSERT License (trial period)        │
  │  4. INSERT AppSettings (defaults)        │
  │  5. INSERT SmsSettings (disabled)        │
  │                                          │
  │  ✓ All succeed → proceed                 │
  │  ✗ Any fail   → full rollback            │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────┐
  │          DASHBOARD SCREEN                │
  │  "Create your first class to begin"      │
  └──────────────────────────────────────────┘
```

**Existing user restore flow:**

```
  App Launch → Google Sign-In
         │
         ▼
  Connect Google Drive
         │
         ▼
  Download backup file from Drive
         │
         ▼
  @Transaction: Restore DB
         │
         ├── Success → Load Dashboard
         └── Failure → Show error, retry option
```

---

### 3B. Student Admission

```
  Teacher fills admission form
  (name, class, shift, parent phone, start month)
         │
         ▼
  StudentViewModel.admitStudent(studentData)
         │
         ▼
  ┌──────────────────────────────────────────┐
  │      @Transaction  (ATOMIC)              │
  │                                          │
  │  1. INSERT Student                       │
  │     → shift_id, name, parent_phone,      │
  │       start_month (YYYYMM), is_active=1  │
  │                                          │
  │  2. Check: is SMS (admission) enabled?   │
  │     YES → INSERT SmsQueue entry          │
  │            (type: ADMISSION, priority: 2)│
  │     NO  → skip                           │
  │                                          │
  │  ✓ Both succeed → emit StudentAdded event│
  │  ✗ Any fail     → full rollback          │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  Room Flow emits updated student list
         │
         ▼
  StudentListScreen recomposes automatically
```

**Bulk CSV admission:**

```
  Teacher selects CSV file
         │
         ▼
  Parse CSV (validate columns, skip bad rows)
         │
         ▼
  @Transaction: INSERT all valid students at once
  (No SMS queued for bulk import — by design)
         │
         ├── Success → "X students added"
         └── Failure → full rollback, show error
```

---

### 3C. Daily Attendance

This is the most frequent operation in the app. Speed is critical.

```
  Teacher opens Attendance tab
         │
         ▼
  Select Class → Select Shift → Select Date
         │
         ▼
  AttendanceViewModel.loadAttendance(classId, shiftId, date)
         │
         ├── attendanceRepository.getSession(classId, shiftId, date)
         │       → Room query (existing session or null)
         │
         └── studentRepository.getActiveStudents(shiftId)
                 → Flow<List<Student>> (live list)
         │
         ▼
  UiState.Ready(students, presentIds, date, mode)
         │
         ▼
  ┌──────────────────────────────────────────────────────────┐
  │                 ATTENDANCE SCREEN                        │
  │                                                          │
  │  Mode: DEFAULT_PRESENT   │   Mode: DEFAULT_ABSENT        │
  │  (tap to mark absent)    │   (tap to mark present)       │
  │                                                          │
  │  [✓] Rana         [✓] Sharmin       [✗] Hasan           │
  └──────────────────────┬───────────────────────────────────┘
                         │ tap toggles local Set<studentId>
                         │ (in-memory — not saved yet)
                         ▼
  Teacher taps "Save"
         │
         ▼
  AttendanceViewModel.saveAttendance()
         │
         ▼
  ┌──────────────────────────────────────────┐
  │      @Transaction  (ATOMIC)              │
  │                                          │
  │  1. UPSERT AttendanceSession             │
  │     (classId, shiftId, date, totalCount) │
  │                                          │
  │  2. DELETE old AttendanceRecords         │
  │     for this session (if editing)        │
  │                                          │
  │  3. INSERT AttendanceRecords             │
  │     one row per student                  │
  │     (student_id, session_id, is_present) │
  │                                          │
  │  4. Check: is absent SMS enabled?        │
  │     YES → INSERT SmsQueue for each       │
  │            absent student                │
  │     NO  → skip                          │
  │                                          │
  │  ✓ All succeed → UiState saved           │
  │  ✗ Any fail    → full rollback           │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  Calendar view updates: green dot appears on date
  (Room Flow emits → Calendar recomposes)
```

---

### 3D. Fee Collection

```
  Teacher opens Fee tab → selects Class/Shift
         │
         ▼
  FeeViewModel loads:
  ┌────────────────────────────────────────────────────────┐
  │  For each active student:                              │
  │                                                        │
  │  CalculateDueMonthsUseCase.invoke(studentId, startMonth│
  │                                                        │
  │  Logic (pure Kotlin, no Android deps):                 │
  │    currentMonth = today in YYYYMM format               │
  │    paidMonths   = feeRepository.getPaidMonths(id)      │
  │    dueMonths    = [startMonth..currentMonth]            │
  │                   minus paidMonths                     │
  └────────────────────────────────────────────────────────┘
         │
         ▼
  FeeScreen shows:  Rana — 3 months due (৳2,997)
         │
  Teacher selects student → selects months → taps Collect
         │
         ▼
  CollectFeeUseCase.invoke(studentId, months, amount)
         │
         ├── Validate: months not empty, amount > 0
         │             no month already paid
         │
         ▼
  ┌──────────────────────────────────────────┐
  │      @Transaction  (ATOMIC)              │
  │                                          │
  │  For each selected month:                │
  │    INSERT FeeTransaction                 │
  │    (student_id, fee_month YYYYMM,        │
  │     amount, paid_at timestamp)           │
  │  → separate row per month               │
  │                                          │
  │  If confirmation SMS enabled:            │
  │    INSERT SmsQueue                       │
  │    (type: FEE_CONFIRMATION, priority: 1) │
  │                                          │
  │  ✓ Succeed → emit PaymentSuccess event   │
  │  ✗ Fail    → full rollback               │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  Room Flow emits updated fee_transactions
         │
         ├── FeeScreen: student moves to "paid" list
         └── Dashboard: monthly total updates automatically
```

**Fee reminder flow:**

```
  Teacher taps "Send Fee Reminders" for a month
         │
         ▼
  Query unpaid students:
    SELECT students WHERE is_active=1
    AND NOT EXISTS (fee_transaction for this month)
         │
         ▼
  @Transaction: INSERT SmsQueue for each unpaid student
  (type: FEE_REMINDER, priority: 1)
         │
         ▼
  SmsWorker picks up entries → sends SMS one by one
```

---

### 3E. Exam Results

```
  Teacher creates exam:
  (name, date, total marks, class/shift)
         │
         ▼
  INSERT Exam → get exam_id
         │
         ▼
  ExamScreen: list of all students, marks input fields
         │
  Teacher enters marks, taps Save
         │
         ▼
  ┌──────────────────────────────────────────┐
  │      @Transaction  (ATOMIC)              │
  │                                          │
  │  For each student:                       │
  │    INSERT ExamResult                     │
  │    (exam_id, student_id, marks_obtained, │
  │     is_absent flag)                      │
  │                                          │
  │  If result SMS enabled:                  │
  │    INSERT SmsQueue per student           │
  │    (type: EXAM_RESULT, priority: 2)      │
  │                                          │
  │  ✓ Succeed → ExamSaved event             │
  │  ✗ Fail    → full rollback               │
  └──────────────────┬───────────────────────┘
                     │
                     ▼
  ExamResult screen shows:
    Highest: 95 | Average: 78 | Absent: 2
  (computed in Repository layer — not DAO)
```

---

### 3F. SMS Queue Processing

SMS is never sent directly. Everything goes through a priority queue in Room, processed by a background worker.

```
  Any Feature  (Attendance / Fee / Exam / Admission)
       │
       │  [if SMS enabled for that event type]
       ▼
  INSERT SmsQueue
  ┌─────────────────────────────────────────────────────┐
  │  sms_queue table row:                               │
  │  id, recipient_phone, message_body,                 │
  │  type (ABSENT/FEE_DUE/FEE_CONFIRM/EXAM_RESULT/...)  │
  │  priority (1=high, 2=normal, 3=low)                 │
  │  status (PENDING)                                   │
  │  retry_count (0)                                    │
  │  created_at                                         │
  └─────────────────────────────────────────────────────┘
       │
       ▼
  ┌──────────────────────────────────────────────────────┐
  │              SmsWorker (Background)                  │
  │         Runs every 30 seconds via WorkManager        │
  │                                                      │
  │  1. Fetch next PENDING entry (priority ASC)          │
  │  2. Update status → SENDING                          │
  │  3. Send SMS via Android SmsManager (device SIM)     │
  │  4. Wait 5–10 seconds (prevents carrier blocking)    │
  │  5a. Success → UPDATE status = SENT                  │
  │  5b. Failure → retry_count++                         │
  │       if retry_count < 3 → status = PENDING (retry)  │
  │       if retry_count ≥ 3 → status = FAILED           │
  │  6. Repeat until queue is empty                      │
  └──────────────────────────────────────────────────────┘
       │
       ▼
  CleanupWorker (Weekly):
  DELETE old SENT/FAILED entries older than 30 days
```

**Priority levels:**

```
  Priority 1 (HIGH)    → Fee Confirmation, Fee Reminder
  Priority 2 (NORMAL)  → Absent Notification, Exam Result
  Priority 3 (LOW)     → Welcome/Admission, Monthly Summary
```

---

### 3G. Google Drive Backup

Backup is fully automatic when conditions are met. The teacher never has to think about it.

```
  BackupWorker
  (Scheduled: once daily, requires network + charging)
         │
         ▼
  1. Serialize entire Room DB → backup file
         │
         ▼
  2. Upload file to teacher's Google Drive
     (app-specific folder — only this app can read it)
         │
         ├── Success:
         │     UPDATE backup_metadata
         │     (last_backup_time, backup_file_id)
         │     → BackupScreen shows "Last backup: just now"
         │
         └── Failure:
               retry_count++
               if retry_count < 3 → retry tomorrow
               if retry_count ≥ 3 → notify teacher
```

**Restore flow (on new device):**

```
  New device → App launch → "Existing User" selected
         │
         ▼
  Google Sign-In (same account)
         │
         ▼
  List backup files from Drive
         │
         ▼
  Teacher selects backup → Download
         │
         ▼
  @Transaction: Replace local DB with backup
         │
         ├── Success → Load Dashboard with restored data
         └── Failure → Keep current state, show error
```

---

### 3H. License Validation

```
  App Launch (every time)
         │
         ▼
  ┌────────────────────────────────────────────────────┐
  │  LicenseCheckUseCase.invoke()                      │
  │                                                    │
  │  1. Is device online?                              │
  │                                                    │
  │     YES → Fetch license status from Firebase       │
  │            Update local License record             │
  │                                                    │
  │     NO  → Read local License table                 │
  │            (expiry_date stored as epoch ms)        │
  │                                                    │
  │  2. Is current_date > expiry_date + grace_period?  │
  │                                                    │
  │     NO  → Proceed to Dashboard normally            │
  │                                                    │
  │     YES → Show Blocked Screen                      │
  │           "Your license has expired"               │
  │           Device ID shown for teacher to share     │
  │           "Contact Developer" button               │
  └────────────────────────────────────────────────────┘
```

---

## 4. Database as Single Source of Truth

Room is the only place where application state lives. There is no in-memory cache that can drift out of sync.

```
  ┌───────────────────────────────────────────────────────────┐
  │                    ROOM DATABASE                          │
  │                                                           │
  │  Every Feature reads from Room via Flow<T>                │
  │  Every Feature writes to Room via suspend DAO functions   │
  │                                                           │
  │  Flow<T> is reactive — UI updates automatically           │
  │  when the underlying table changes                        │
  └───────────────────────────────────────────────────────────┘

  Example — what happens when a student pays a fee:

  FeeDao.insertTransaction()  ←  write
         │
         ▼ (Room emits change)
         │
         ├──► FeeScreen Flow updates     → paid list refreshes
         ├──► DashboardScreen Flow       → monthly total updates
         └──► StudentProfile Flow        → fee history updates

  All three screens update from ONE write — no manual refresh.
```

**The 17 tables and their owners:**

```
  ┌──────────────────────────────────────────────────┐
  │  TABLE                  │  OWNED BY              │
  ├──────────────────────────────────────────────────┤
  │  teachers               │  AuthRepository         │
  │  batches                │  BatchRepository        │
  │  licenses               │  LicenseRepository      │
  │  app_settings           │  SettingsRepository     │
  │  backup_metadata        │  BackupRepository       │
  ├──────────────────────────────────────────────────┤
  │  classes                │  ClassRepository        │
  │  shifts                 │  ShiftRepository        │
  ├──────────────────────────────────────────────────┤
  │  students               │  StudentRepository      │
  ├──────────────────────────────────────────────────┤
  │  attendance_sessions    │  AttendanceRepository   │
  │  attendance_records     │  AttendanceRepository   │
  ├──────────────────────────────────────────────────┤
  │  fee_transactions       │  FeeRepository          │
  │  expenses               │  ExpenseRepository      │
  ├──────────────────────────────────────────────────┤
  │  exams                  │  ExamRepository         │
  │  exam_results           │  ExamRepository         │
  ├──────────────────────────────────────────────────┤
  │  sms_queue              │  SmsRepository          │
  │  sms_templates          │  SmsRepository          │
  │  sms_settings           │  SmsRepository          │
  └──────────────────────────────────────────────────┘
```

---

## 5. State Management Flow

```
  ┌─────────────────────────────────────────────────────────┐
  │  THREE KINDS OF STATE                                   │
  └─────────────────────────────────────────────────────────┘

  1. Flow<T>  ──────────────────────────────────────────────
     Source:   Room DAO
     Purpose:  Live database queries that auto-update
     Example:  studentDao.getActiveStudents(shiftId)
               → UI always shows current data without refresh

  2. StateFlow<UiState>  ───────────────────────────────────
     Source:   ViewModel
     Purpose:  Current screen state (always has a value)
     States:
       Loading  →  "Getting data..."
       Ready    →  data loaded, UI interactive
       Error    →  "Something went wrong"

     Pattern:
       Loading
          │
          ├── success ──► Ready(data)
          │                  │
          │                  └── user edits ──► Ready(updatedData)
          │
          └── failure ──► Error(message)

  3. SharedFlow<Event>  ────────────────────────────────────
     Source:   ViewModel
     Purpose:  One-time side effects (don't repeat on recompose)
     Examples:
       SaveSuccess   → navigate back to previous screen
       Error(msg)    → show Snackbar for 3 seconds
       PaymentDone   → show confirmation dialog
```

---

## 6. Background Worker Lifecycle

```
  WorkManager manages three background workers:

  ┌─────────────────────────────────────────────────────────┐
  │  SmsWorker                                              │
  │  Schedule: PeriodicWork every 30 seconds                │
  │  Trigger:  Also kicked on new SmsQueue entry            │
  │  Constraint: None (runs even without network)           │
  │                                                         │
  │  PENDING → [SmsWorker picks up] → SENDING → SENT        │
  │                                       └──► FAILED       │
  │               (retry up to 3 times with delay)          │
  └─────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────┐
  │  BackupWorker                                           │
  │  Schedule: PeriodicWork once per day                    │
  │  Constraint: Network connected + device charging        │
  │                                                         │
  │  [Conditions met] → Serialize DB → Upload Drive         │
  │        │                                                │
  │        ├── Success → UPDATE backup_metadata             │
  │        └── Failure → retry next day (max 3)             │
  └─────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────┐
  │  CleanupWorker                                          │
  │  Schedule: PeriodicWork once per week                   │
  │  Constraint: None                                       │
  │                                                         │
  │  DELETE sms_queue WHERE status IN (SENT, FAILED)        │
  │           AND created_at < (now - 30 days)              │
  └─────────────────────────────────────────────────────────┘

  Worker failure handling:

  doWork() throws exception
       │
       ▼
  runAttemptCount < 3  →  Result.retry()
  runAttemptCount ≥ 3  →  Result.failure() + log error
```

---

## 7. Offline-First Data Strategy

Amar Batch is designed to work 100% offline. The internet is only needed for backup.

```
  ┌─────────────────────────────────────────────────────────┐
  │              OFFLINE vs ONLINE CAPABILITY               │
  └─────────────────────────────────────────────────────────┘

  ALWAYS works offline (no internet needed):
  ✓  Mark attendance
  ✓  Collect fees
  ✓  Enter exam results
  ✓  Add / manage students
  ✓  Send SMS (uses device SIM, not internet)
  ✓  License check (uses local expiry_date)
  ✓  View all history and reports
  ✓  Profit / due calculations

  Requires internet (optional, graceful degradation):
  ○  Google Drive backup / restore
  ○  License refresh from Firebase (falls back to local)

  ─────────────────────────────────────────────────────────

  Data precedence when conflicts exist:

  Room (local) is ALWAYS authoritative.
  Google Drive is a backup copy — it never overwrites
  local data automatically.

  Restore = explicit teacher action (on new device setup only)

  ─────────────────────────────────────────────────────────

  Why Room is the single source of truth:

  IF (cloud first) THEN:
    UI must wait for network → slow in rural Bangladesh
    Conflicts between local cache and cloud state
    App broken without internet

  BECAUSE (Room first):
    UI always has data instantly (Flow<T> from local DB)
    No race conditions — one source, one truth
    App fully functional even with zero connectivity
    Cloud backup is additive, not essential
```

---

## Related Documentation

| Document                                                             | Description                               |
| -------------------------------------------------------------------- | ----------------------------------------- |
| [ARCHITECTURE_OVERVIEW.md](../architecture/ARCHITECTURE_OVERVIEW.md) | Clean Architecture + MVVM layer breakdown |
| [TECH_STACK.md](../architecture/TECH_STACK.md)                       | All libraries and technology choices      |
| [SCHEMA_OVERVIEW.md](../database/SCHEMA_OVERVIEW.md)                 | All 17 database tables explained          |
| [SMS_QUEUE_SYSTEM.md](../technical/SMS_QUEUE_SYSTEM.md)              | Priority queue design in detail           |
| [OFFLINE_SYNC_STRATEGY.md](../technical/OFFLINE_SYNC_STRATEGY.md)    | Offline-first approach deep dive          |

---

_Last Updated: May 2026_
_App Version: Beta_
_Developer: MD SHAHAJALAL MAHMUD — [mahmud.nubtk@gmail.com](mailto:mahmud.nubtk@gmail.com)_
