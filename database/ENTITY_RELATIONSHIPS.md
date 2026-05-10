# Entity Relationships & Data Integrity

## For Recruiters & Technical Evaluators

**Foreign Key Constraints | Unique Constraints | Soft Delete Pattern**

---

## 📋 Overview

This document describes how database entities relate to each other and the rules that maintain data integrity. These relationships are designed to preserve historical records while preventing data corruption.

**Key principle:** Student, attendance, and fee history is NEVER permanently deleted.

---

## 📊 Parent-Child Relationships

### Core Hierarchy Chain

| Parent  | Child   | Relationship | Delete Rule | Rationale                                   |
| ------- | ------- | ------------ | ----------- | ------------------------------------------- |
| Teacher | Batch   | One-to-many  | RESTRICT    | Cannot delete teacher with existing batches |
| Batch   | Class   | One-to-many  | RESTRICT    | Cannot delete batch with existing classes   |
| Class   | Shift   | One-to-many  | CASCADE     | Shifts belong exclusively to a class        |
| Shift   | Student | One-to-many  | RESTRICT    | Cannot delete shift with enrolled students  |

### Attendance Chain

| Parent             | Child              | Relationship | Delete Rule | Rationale                   |
| ------------------ | ------------------ | ------------ | ----------- | --------------------------- |
| Class              | Attendance Session | One-to-many  | RESTRICT    | Preserve attendance history |
| Shift              | Attendance Session | One-to-many  | RESTRICT    | Preserve attendance history |
| Attendance Session | Attendance Record  | One-to-many  | CASCADE     | Records belong to session   |
| Student            | Attendance Record  | One-to-many  | RESTRICT    | Preserve student history    |

### Financial Chain

| Parent  | Child           | Relationship | Delete Rule | Rationale                                  |
| ------- | --------------- | ------------ | ----------- | ------------------------------------------ |
| Student | Fee Transaction | One-to-many  | RESTRICT    | Cannot delete student with payment history |
| Batch   | Expense         | One-to-many  | CASCADE     | Expenses belong to batch                   |

### Assessment Chain

| Parent  | Child       | Relationship | Delete Rule | Rationale                            |
| ------- | ----------- | ------------ | ----------- | ------------------------------------ |
| Class   | Exam        | One-to-many  | RESTRICT    | Cannot delete class with exams       |
| Shift   | Exam        | One-to-many  | RESTRICT    | Cannot delete shift with exams       |
| Exam    | Exam Result | One-to-many  | CASCADE     | Results belong to exam               |
| Student | Exam Result | One-to-many  | RESTRICT    | Preserve student performance history |

### License Chain

| Parent  | Child   | Relationship | Delete Rule | Rationale                    |
| ------- | ------- | ------------ | ----------- | ---------------------------- |
| Teacher | License | One-to-many  | RESTRICT    | Preserve license audit trail |

---

## 🔒 Unique Constraints

Prevent duplicate data entries across critical combinations.

| Table                   | Constraint (Conceptual)              | Purpose                                   |
| ----------------------- | ------------------------------------ | ----------------------------------------- |
| **teachers**            | One Google account per teacher       | Prevent duplicate teacher profiles        |
| **attendance_sessions** | One session per (class, shift, date) | Cannot mark attendance twice for same day |
| **attendance_records**  | One record per (session, student)    | Student has only one status per session   |
| **fee_transactions**    | One payment per (student, month)     | No duplicate payments for same month      |
| **exam_results**        | One result per (exam, student)       | Student has only one mark per exam        |

---

## 🗑️ Deletion Rules by Entity

### Cannot Delete (Historical Preservation)

| Entity             | Why                  |
| ------------------ | -------------------- |
| Teacher            | Core identity        |
| Attendance Session | Historical record    |
| Attendance Record  | Historical record    |
| Exam               | Results depend on it |
| SMS Template       | System controlled    |

### Soft Delete (Hide but Preserve)

| Entity  | Using `is_active` flag | When Used                |
| ------- | ---------------------- | ------------------------ |
| Batch   | Yes                    | Teacher disables a batch |
| Class   | Yes                    | Class no longer offered  |
| Shift   | Yes                    | Shift discontinued       |
| Student | Yes                    | Student leaves batch     |

**Soft delete means:** Data remains in database but filtered out of daily views. Historical reports still work.

### Hard Delete (Permanent Removal)

| Entity          | Conditions                   | Why Allowed                    |
| --------------- | ---------------------------- | ------------------------------ |
| Fee Transaction | Confirmation required        | Teacher added wrong payment    |
| Expense         | Freely deletable             | Operational cost entry error   |
| SMS Queue       | Auto-cleanup after 7-30 days | Prevent unlimited queue growth |

### Edit Only (No Delete)

| Entity      | Notes                                         |
| ----------- | --------------------------------------------- |
| Exam Result | Can modify marks but cannot delete the record |

---

## ✅ Business Rules Summary

| Rule                         | Description                                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------ |
| **Default Shift**            | Every class must have at least 1 shift (auto-created if missing)                     |
| **Soft Deletes Only**        | Students, classes, shifts use `is_active` flag — never hard deleted                  |
| **Active Filtering**         | Most operations only show entities with `is_active = true`                           |
| **Fee Snapshots**            | Fee amount, class, shift captured at payment time (future changes don't affect past) |
| **Academic Year Protection** | Classes tied to specific years — data doesn't corrupt during transitions             |
| **Foreign Key Enforcement**  | All relationships protected by database-level constraints                            |
| **Duplicate Prevention**     | Unique constraints block duplicate sessions, payments, results                       |
| **Phone Validation**         | Bangladesh format (11 digits, starts with 01) enforced in application layer          |
| **SMS Gating**               | SMS sent only if enabled globally AND per class AND student active AND phone exists  |
| **Timestamps**               | All tables have `created_at`; critical tables have `updated_at`                      |

---

## 🎯 Design Rationale

### Why RESTRICT on Most Parent Deletions?

> _"Don't let teachers accidentally delete data they'll need next month for reports."_

- Attendance and fee history is valuable for parent communication
- Soft delete allows "hiding" without losing data
- Historical reports require complete data

### Why CASCADE on Some Relationships?

> _"If the parent doesn't exist, the child has no meaning."_

- Shifts without a class don't make sense
- Attendance records without a session are meaningless
- Exam results without an exam have no context

### Why No Hard Delete for Students?

> _"A student who left last month still has fee and attendance history."_

- Teachers need year-end reports including all students
- Legal/financial records may require historical data
- Soft delete preserves integrity while hiding from daily views

---

## 📊 What This Design Demonstrates

For recruiters evaluating this schema:

| Skill                                      | Evidence                                                  |
| ------------------------------------------ | --------------------------------------------------------- |
| **Understanding of referential integrity** | Proper foreign key usage with appropriate ON DELETE rules |
| **Data preservation thinking**             | Soft delete strategy for critical entities                |
| **Practical trade-offs**                   | When to use RESTRICT vs CASCADE                           |
| **Business logic integration**             | Fee snapshots, academic year protection                   |
| **Duplicate prevention**                   | Composite unique constraints                              |

---

## 📚 Related Documentation

- [Schema Overview](./SCHEMA_OVERVIEW.md) — Table categories and design principles
- [Architecture Overview](../architecture/ARCHITECTURE_OVERVIEW.md) — System design and layer separation

---

## 📧 Contact for Discussion

**Developer:** MD SHAHAJALAL MAHMUD (3rd Year CSE Student)
**Email:** mahmud.nubtk@gmail.com

---

_Last Updated: May 2026_
