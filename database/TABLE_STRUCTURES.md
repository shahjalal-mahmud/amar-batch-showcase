# Table Structures

## Column Definitions (No SQL)

**For Recruiters & Technical Evaluators**

This document describes the **purpose and business rules** of each table without exposing actual SQL implementation. Column definitions are described conceptually.

> **Note:** Actual column names, data types, and SQL statements are not included to protect intellectual property.

---

## Table Categories

| Category               | Tables                                                                         | Purpose                    |
| ---------------------- | ------------------------------------------------------------------------------ | -------------------------- |
| **Core Identity**      | teachers, batches                                                              | Who is using the app       |
| **Academic Structure** | classes, shifts, students                                                      | How students are organized |
| **Operational**        | attendance_sessions, attendance_records, fee_transactions, exams, exam_results | Daily teaching activities  |
| **Financial**          | expenses                                                                       | Cost tracking              |
| **Communication**      | sms_settings, sms_templates, sms_queue                                         | Parent messaging           |
| **System**             | app_settings, backup_metadata, licenses                                        | App management             |

---

## Core Identity Tables

### Teachers

**Purpose:** Stores teacher profile and authentication information.

**Key columns (conceptual):**

- Unique identifier (auto-generated)
- Teacher's full name
- Contact phone number (Bangladesh format: 01XXXXXXXXX)
- Profile picture reference (optional)
- Google Sign-In identifier (optional, for backup)
- Guest mode flag (authenticated vs. local-only)
- Creation timestamp

**Business rules:**

- One teacher record per app installation (MVP)
- Phone number must be unique
- Cannot be deleted (core identity)
- Guest mode users have no Google account linked

---

### Batches

**Purpose:** Groups classes together. Enables future multi-batch support.

**Key columns (conceptual):**

- Unique identifier
- Reference to teacher
- Batch name (e.g., "Main Batch")
- Creation timestamp

**Business rules:**

- One batch auto-created on first launch (MVP)
- Cannot delete if classes exist
- Future: multiple batches per teacher

---

## Academic Structure Tables

### Classes

**Purpose:** Represents academic classes with year tracking.

**Key columns (conceptual):**

- Unique identifier
- Reference to batch
- Class name (e.g., "Class 8", "SSC Batch")
- Academic year (e.g., 2026)
- Default monthly tuition fee
- Active status flag (soft delete)
- Creation timestamp

**Business rules:**

- Each class tied to specific academic year
- Prevents historical data corruption
- Same class name can exist in different years
- Cannot delete if students exist (use soft delete)

---

### Shifts

**Purpose:** Time-based groupings of students within a class.

**Key columns (conceptual):**

- Unique identifier
- Reference to class
- Shift name (e.g., "Morning Batch", "Shift A")
- Weekly schedule (optional)
- Start/end times (optional)
- Custom monthly fee (overrides class default if set)
- Default shift flag (auto-created)
- Active status flag
- Creation timestamp

**Business rules:**

- Every class has at least 1 shift (auto-created default)
- Default shift cannot be manually deleted
- Students only attach to shifts, not directly to classes
- Cannot delete shift if students exist

---

### Students

**Purpose:** Core student records with parent contact information.

**Key columns (conceptual):**

- Unique identifier
- Reference to class and shift
- Student's full name
- Parent/guardian name (optional)
- Parent phone number (required for SMS)
- Student's own phone (optional)
- School name (optional)
- Admission fee and date (optional)
- Starting month (optional, YYYYMM format)
- Active status flag (soft delete)
- Fee exemption flag
- Creation timestamp

**Business rules:**

- Minimum required: name, class, shift, parent phone
- Parent phone must be valid Bangladesh number for SMS
- **No hard deletes allowed** — use active status flag
- Inactive students excluded from daily views
- Historical records (attendance, fees) remain intact

---

## Operational Tables

### Attendance Sessions

**Purpose:** Represents a single attendance-taking event.

**Key columns (conceptual):**

- Unique identifier
- Reference to class and shift
- Session date (epoch timestamp, midnight)
- Creation timestamp

**Business rules:**

- One session per (class, shift, date) combination
- Prevents duplicate attendance on same day
- Cannot be deleted (preserve history)
- Created automatically when teacher marks attendance

---

### Attendance Records

**Purpose:** Individual student attendance status within a session.

**Key columns (conceptual):**

- Unique identifier
- Reference to attendance session
- Reference to student
- Attendance status (present or absent)
- Creation timestamp

**Business rules:**

- One record per (session, student)
- Only active students get records
- Absent status may trigger SMS (if enabled)
- Cannot be deleted (preserve history)

---

### Fee Transactions

**Purpose:** Records tuition fee payments with snapshots.

**Key columns (conceptual):**

- Unique identifier
- Reference to student
- Snapshot: class at payment time
- Snapshot: shift at payment time
- Fee month being paid (YYYYMM format)
- Snapshot: expected amount at payment time
- Actual amount paid
- Payment date timestamp
- Creation timestamp

**Business rules:**

- One transaction per (student, fee month) — no duplicates
- **Snapshots prevent historical corruption:**
  - If student changes class/shift, past payments reference old data
  - If fees increase, past payments show old amount
- Hard delete allowed (with confirmation) for mistakes
- Multi-month payment creates separate transaction per month
- Payment triggers SMS confirmation (if enabled)

---

### Expenses

**Purpose:** Flat expense tracking for profit calculation.

**Key columns (conceptual):**

- Unique identifier
- Reference to batch
- Expense title/description
- Amount in Taka
- Expense month (YYYYMM format)
- Creation timestamp

**Business rules:**

- Intentionally simple — no categories or complex accounting
- Hard delete allowed
- Freely editable by teacher
- Used for monthly profit calculation

---

### Exams

**Purpose:** Exam definitions with metadata.

**Key columns (conceptual):**

- Unique identifier
- Reference to class and shift
- Exam name (e.g., "Weekly Test 1")
- Exam date timestamp
- Total possible marks
- Google Drive link to exam paper (optional)
- Creation timestamp

**Business rules:**

- Must belong to specific shift
- Cannot be deleted (preserve results)
- Teacher can edit name, date, total marks
- Results auto-created for all active students when exam created

---

### Exam Results

**Purpose:** Individual student marks for an exam.

**Key columns (conceptual):**

- Unique identifier
- Reference to exam
- Reference to student
- Marks obtained (decimal, null if absent)
- Status (present or absent)
- Creation timestamp

**Business rules:**

- One result per (exam, student)
- If absent: status = ABSENT, marks = null
- If present: status = PRESENT, marks has value
- Marks must be ≤ exam's total marks
- Cannot be deleted — can only modify marks
- Results trigger SMS (if enabled)

---

## Financial Table

### Expenses

_(See Operational Tables — expenses moved to Operational in actual implementation)_

**Key columns (conceptual):**

- Unique identifier
- Reference to batch
- Expense title
- Amount
- Expense month (YYYYMM)
- Creation timestamp

---

## Communication Tables

### SMS Settings

**Purpose:** Class-level SMS preferences.

**Key columns (conceptual):**

- Unique identifier
- Reference to class (one-to-one)
- Flags for each SMS type (admission, fee reminder, payment confirmation, absent alert, exam result, monthly summary)
- Creation timestamp

**Business rules:**

- One settings record per class (auto-created)
- Each SMS type can be enabled/disabled independently
- Global SMS master switch overrides all class settings

---

### SMS Templates

**Purpose:** Customizable message templates.

**Key columns (conceptual):**

- Unique identifier
- SMS type (enum: 6 types total)
- Template content with placeholders (e.g., `{student_name}`)
- Default flag (true if teacher hasn't modified)
- Creation timestamp

**Business rules:**

- 6 system-controlled types (cannot add/remove)
- Teacher can modify content only
- Placeholders replaced at runtime with actual data
- Cannot be deleted
- Character limit: 160 per SMS (longer messages split)

---

### SMS Queue

**Purpose:** Queues outbound SMS for background sending.

**Key columns (conceptual):**

- Unique identifier
- Reference to student
- Parent phone number (recipient)
- Rendered message content
- SMS type (for tracking)
- Priority (high, normal)
- Status (pending, sent, failed, cancelled)
- Retry count
- Failure reason (if failed)
- Creation timestamp
- Sent timestamp

**Business rules:**

- Processed by WorkManager in background
- One SMS every 5-10 seconds (prevents blocking)
- Max 3 retries with exponential backoff
- Cleanup: sent/removed after 7-30 days
- Global SMS disable pauses queue

---

## System Tables

### App Settings

**Purpose:** Global app configuration. Single row.

**Key columns (conceptual):**

- Primary key (always 1)
- Global SMS enable/disable
- Selected SIM slot for dual-SIM phones
- Auto-backup enable/disable
- Creation timestamp

**Business rules:**

- Only one row exists (created on first launch)
- Never deleted
- Global SMS disables ALL SMS regardless of class settings

---

### Backup Metadata

**Purpose:** Tracks backup and restore operations. Single row.

**Key columns (conceptual):**

- Primary key (always 1)
- Last successful backup timestamp
- Last restore timestamp
- Backup format version (for migrations)
- Creation timestamp

**Business rules:**

- Only one row exists
- Updated after successful backup/restore
- Version field handles schema changes during restore

---

### Licenses

**Purpose:** Access control and trial management.

**Key columns (conceptual):**

- Unique identifier
- Reference to teacher
- Device ID (Android unique identifier)
- License status (trial, paid, blocked)
- Trial start date
- Expiry date
- Grace period days (default 3)
- Last server check timestamp
- Paid flag
- Last update timestamp

**Business rules:**

- One license per teacher/device combination
- Trial period: 30-90 days (configurable)
- Grace period: 3 days after expiry
- Status = BLOCKED after grace period → app locks
- Device binding prevents license sharing
- Server sync validates license when online
- Manual payment → developer updates license

---

## Summary: Table Count

| Category           | Count  |
| ------------------ | ------ |
| Core Identity      | 2      |
| Academic Structure | 3      |
| Operational        | 5      |
| Financial          | 1      |
| Communication      | 3      |
| System             | 3      |
| **Total**          | **17** |

---

## What This Demonstrates

| Skill                       | Evidence                                       |
| --------------------------- | ---------------------------------------------- |
| **Database normalization**  | 17 tables with clear separation of concerns    |
| **Data integrity**          | Foreign keys, unique constraints, soft deletes |
| **Historical preservation** | Soft delete strategy, fee snapshots            |
| **Practical design**        | Simplified expenses (no complex accounting)    |
| **Offline-first**           | All tables designed for local SQLite           |

---

## Related Documentation

- [Schema Overview](./SCHEMA_OVERVIEW.md) — Table categories and design principles
- [Entity Relationships](./ENTITY_RELATIONSHIPS.md) — Foreign keys and constraints
- [Architecture Overview](../architecture/ARCHITECTURE_OVERVIEW.md) — System design

---

**Developer:** MD SHAHAJALAL MAHMUD
**Email:** mahmud.nubtk@gmail.com

---

_Last Updated: May 2026_
_Total Tables: 17_
