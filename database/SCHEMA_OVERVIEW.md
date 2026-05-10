# Database Schema Overview

## For Recruiters & Technical Evaluators

**17 Tables | Normalized Design | Offline-First | Soft Delete Pattern**

---

## 📋 Introduction

This document provides a **high-level overview** of the database schema for Amar Batch. The schema is designed to support offline-first operations while maintaining data integrity.

| Metric              | Value                          |
| ------------------- | ------------------------------ |
| **Database Type**   | SQLite (via Room)              |
| **Total Tables**    | 17                             |
| **Design Pattern**  | Normalized relational database |
| **Delete Strategy** | Soft deletes for critical data |

> **Note:** This document describes the **structure and relationships** only. Actual column definitions, SQL queries, and implementation details are not included.

---

## 🎯 Design Principles

### 1. Offline-First

- All data stored locally on device
- No internet required for core operations
- Backup/restore optional via Google Drive

### 2. Data Integrity

- Foreign key constraints prevent orphaned records
- Unique constraints prevent duplicates
- Timestamps (epoch milliseconds) for all records

### 3. Soft Delete Strategy

- Critical entities use `is_active` flags
- Historical data preserved for reporting
- No hard deletes for student data

### 4. Academic Year Protection

- Classes tied to academic years
- Historical records remain accurate
- Supports year-over-year transitions

---

## 📊 Entity Relationship Overview

```
Teacher (1) → Batch (N)
  └─ ON DELETE: RESTRICT (cannot delete teacher if batches exist)

Batch (1) → Class (N)
  └─ ON DELETE: RESTRICT (cannot delete batch if classes exist)

Class (1) → Shift (N)
  └─ ON DELETE: CASCADE (when class deleted, shifts auto-deleted)

Shift (1) → Student (N)
  └─ ON DELETE: RESTRICT (cannot delete shift if students exist)

Student (1) → Attendance Record (N)
  └─ ON DELETE: RESTRICT (cannot delete student if records exist)

Student (1) → Fee Transaction (N)
  └─ ON DELETE: RESTRICT (cannot delete student if transactions exist)

Student (1) → Exam Result (N)
  └─ ON DELETE: RESTRICT (cannot delete student if results exist)

Exam (1) → Exam Result (N)
  └─ ON DELETE: CASCADE (when exam deleted, results auto-deleted)

Batch (1) → Expense (N)
  └─ ON DELETE: CASCADE (when batch deleted, expenses auto-deleted)

Teacher (1) → License (N)
  └─ ON DELETE: RESTRICT (cannot delete teacher if license exists)
```

### Key Relationships

| Parent             | Child             | Type        | Notes                            |
| ------------------ | ----------------- | ----------- | -------------------------------- |
| Teacher            | Batch             | One-to-many | Future multi-batch support       |
| Batch              | Class             | One-to-many | Academic year scoped             |
| Class              | Shift             | One-to-many | Always has at least 1 shift      |
| Shift              | Student           | One-to-many | Every student belongs to a shift |
| Student            | Fee Transaction   | One-to-many | One per month possible           |
| Student            | Exam Result       | One-to-many | One per exam                     |
| Attendance Session | Attendance Record | One-to-many | One per student per session      |

---

## 📁 Table Categories

### Core Identity (2 tables)

| Table        | Purpose                                  |
| ------------ | ---------------------------------------- |
| **teachers** | Teacher profile and authentication       |
| **batches**  | Batch organization (single batch in MVP) |

### Academic Structure (3 tables)

| Table        | Purpose                             |
| ------------ | ----------------------------------- |
| **classes**  | Academic classes with year tracking |
| **shifts**   | Time-based student groupings        |
| **students** | Student master records              |

### Operational Data (5 tables)

| Table                   | Purpose                       |
| ----------------------- | ----------------------------- |
| **attendance_sessions** | Attendance day records        |
| **attendance_records**  | Individual student attendance |
| **fee_transactions**    | Tuition fee payments          |
| **exams**               | Exam definitions              |
| **exam_results**        | Student exam marks            |

### Financial Management (1 table)

| Table        | Purpose                              |
| ------------ | ------------------------------------ |
| **expenses** | Batch operational costs (simplified) |

### Communication System (3 tables)

| Table             | Purpose                           |
| ----------------- | --------------------------------- |
| **sms_settings**  | Class-level SMS preferences       |
| **sms_templates** | Customizable message templates    |
| **sms_queue**     | Outbound message queue with retry |

### System Management (3 tables)

| Table               | Purpose                             |
| ------------------- | ----------------------------------- |
| **app_settings**    | Global app configuration            |
| **backup_metadata** | Backup/restore tracking             |
| **licenses**        | Access control and trial management |

---

## 🔧 Data Type Conventions

### Common Data Types

| Data                  | Type                      | Example               |
| --------------------- | ------------------------- | --------------------- |
| **ID**                | Integer (auto-generated)  | 1, 2, 3               |
| **Timestamps**        | Long (epoch milliseconds) | 1704067200000         |
| **Month identifiers** | Long (YYYYMM format)      | 202601 (January 2026) |
| **Phone numbers**     | String                    | "01712345678"         |
| **Currency**          | Decimal (10,2)            | 999.00                |
| **Boolean flags**     | Boolean                   | true/false            |
| **Status enums**      | String                    | "PENDING", "SENT"     |

### Status Values

| Category           | Possible Values                       |
| ------------------ | ------------------------------------- |
| **Attendance**     | PRESENT, ABSENT                       |
| **SMS Queue**      | PENDING, SENT, FAILED, CANCELLED      |
| **License**        | TRIAL, PAID, BLOCKED, EXPIRED         |
| **Student status** | ACTIVE, INACTIVE (via is_active flag) |

---

## 🔗 Key Constraints (Conceptual)

### Foreign Key Constraints

Foreign keys maintain referential integrity:

| Child Table        | Parent Table     | Delete Rule      |
| ------------------ | ---------------- | ---------------- |
| Batch              | Teacher          | RESTRICT         |
| Class              | Batch            | RESTRICT         |
| Shift              | Class            | CASCADE          |
| Student            | Shift            | RESTRICT         |
| Attendance Session | Class, Shift     | RESTRICT         |
| Attendance Record  | Session, Student | CASCADE          |
| Fee Transaction    | Student          | RESTRICT         |
| Exam Result        | Exam, Student    | CASCADE/RESTRICT |

### Unique Constraints

Prevent duplicate entries:

| Table               | Unique Fields                      |
| ------------------- | ---------------------------------- |
| attendance_sessions | class_id + shift_id + session_date |
| attendance_records  | session_id + student_id            |
| fee_transactions    | student_id + fee_month             |
| exam_results        | exam_id + student_id               |

---

## 🗑️ Deletion Strategy

### Hard Delete vs Soft Delete

| Entity             | Delete Type  | Notes                                 |
| ------------------ | ------------ | ------------------------------------- |
| Teacher            | None         | Core identity — cannot delete         |
| Class              | Soft         | `is_active` flag                      |
| Shift              | Soft         | `is_active` flag                      |
| Student            | Soft         | `is_active` flag — never hard deleted |
| Attendance Session | None         | Historical data preserved             |
| Attendance Record  | None         | Historical data preserved             |
| Fee Transaction    | Hard         | Allowed with confirmation             |
| Expense            | Hard         | Freely deletable                      |
| Exam               | None         | Results depend on it                  |
| SMS Queue          | Auto-cleanup | Deleted after 7-30 days               |

**Key principle:** Student, attendance, and fee history is NEVER permanently deleted.

---

## 📈 Data Volume Expectations

### Estimated Size Per Teacher (1 Year)

| Entity              | Volume          |
| ------------------- | --------------- |
| Students            | 50-200          |
| Attendance Sessions | 200-400         |
| Attendance Records  | 10,000 - 80,000 |
| Fee Transactions    | 600 - 2,400     |
| Exams               | 10-50           |
| Exam Results        | 500 - 10,000    |
| SMS Queue           | 1,000 - 50,000  |

**Total database size:** 5-50 MB per year (SQLite, compressed)

---

## ✅ What This Design Demonstrates

For recruiters evaluating this schema:

| Skill                            | Evidence                                    |
| -------------------------------- | ------------------------------------------- |
| **Database normalization**       | 17 tables with proper relationships         |
| **Data integrity understanding** | Foreign key constraints, unique constraints |
| **Offline-first design**         | Local SQLite, no cloud dependency           |
| **Practical trade-offs**         | Soft deletes over hard deletes              |
| **Scalability thinking**         | Index strategy, volume estimates            |
| **Business logic integration**   | Fee snapshots, academic year protection     |

---

## 🚫 What's NOT Included (Intentionally)

This document does NOT contain:

- ❌ Actual SQL CREATE TABLE statements
- ❌ Column names or data type specifications
- ❌ Index definitions
- ❌ Room entity code
- ❌ DAO query implementations
- ❌ Migration SQL scripts

These details remain in the private codebase to protect intellectual property.

---

## 📚 Related Documentation

- [Architecture Overview](../architecture/ARCHITECTURE_OVERVIEW.md) — System design and layer separation
- [Tech Stack](../architecture/TECH_STACK.md) — Technology choices and rationale
- [Offline-First Strategy](../architecture/OFFLINE_FIRST_STRATEGY.md) — Data synchronization approach

---

## 📧 Contact for Schema Discussion

**Developer:** MD SHAHAJALAL MAHMUD
**Email:** mahmud.nubtk@gmail.com

---

_Last Updated: May 2026_
_Version: 1.0 (Public Showcase)_
