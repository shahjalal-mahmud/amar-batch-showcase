# Amar Batch - Teacher Batch Management App

<div align="center">

**A silent assistant for teachers — simple, reliable, and respectful of their time.**

[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)]()
[![Status](https://img.shields.io/badge/Status-Beta-blue.svg)]()
[![Monetized](https://img.shields.io/badge/Monetized-Active-success.svg)]()
[![License](https://img.shields.io/badge/License-Proprietary-red.svg)]()

**🎯 Live Product | 💰 Generating Revenue | 👨‍💻 Built by a 3rd Year CSE Student**

</div>

---

## 📱 What is Amar Batch?

**Amar Batch** is a production Android app for Bangladeshi batch teachers. It helps manage attendance, tuition fees, exams, and parent communication through SMS — **all without requiring internet**.

**I built this alone** as a 3rd year CSE student, and teachers are **paying for it**.

---

## 🎯 Quick Links

| What             | Where                                                          |
| ---------------- | -------------------------------------------------------------- |
| **Live Product** | [appriyo.com/app/amarbatch](https://appriyo.com/app/amarbatch) |
| **Demo Video**   | [Watch 2-min Demo](./videos/quick_demo.mp4)                    |
| **Screenshots**  | [View Gallery](./screenshots/)                                 |
| **User Guide**   | [Read Documentation](./user_guide/)                            |

---

## 💰 Monetization Status

- ✅ **Live on Teacher's phones** (Manual APK distribution)
- ✅ **Paid subscriptions active** (৳999/year)
- ✅ **Revenue generating**
- 🔄 **Preparing for Play Store launch**

---

## 🏗️ Technical Highlights

| Aspect              | Implementation                 |
| ------------------- | ------------------------------ |
| **Architecture**    | Clean Architecture + MVVM      |
| **UI**              | Jetpack Compose (Material 3)   |
| **Database**        | Room with 17 normalized tables |
| **Offline-first**   | 100% offline capable           |
| **Background Jobs** | WorkManager with retry logic   |
| **Backup**          | Google Drive integration       |
| **Testing**         | Unit + Instrumentation tests   |

[📐 View Architecture Details](./docs/architecture/ARCHITECTURE_OVERVIEW.md)

---

## 📸 Screenshots

| Dashboard                                 | Attendance                                          | Fee Collection                            |
| ----------------------------------------- | --------------------------------------------------- | ----------------------------------------- |
| ![Dashboard](./screenshots/dashboard.png) | ![Attendance](./screenshots/attendance_marking.png) | ![Fees](./screenshots/fee_collection.png) |

[View all screenshots →](./screenshots/)

---

## 🎥 Demo Videos

- [Quick Demo (1 min)](./videos/quick_demo.mp4)
- [Attendance Marking](./videos/attendance_demo.mp4)
- [Fee Collection](./videos/fee_demo.mp4)
- [SMS System](./videos/sms_demo.mp4)

---

## File Structure For this Repo

```
amar-batch-showcase/
│
├── README.md                                    # Main entry point
│
├── SHOWCASE.md                                  # Quick overview
│
├── docs/
│   ├── OVERVIEW.md                              # Product concept (from your docs)
│   ├── PROBLEM_STATEMENT.md                     # The problem I solved
│   ├── SOLUTION.md                              # How Amar Batch solves it
│   │
│   ├── architecture/
│   │   ├── ARCHITECTURE_OVERVIEW.md             # Clean Architecture + MVVM
│   │   ├── DATA_FLOW_DIAGRAM.md                 # Text-based diagram
│   │   ├── OFFLINE_FIRST_STRATEGY.md            # Your offline-first approach
│   │   └── TECH_STACK.md                        # From your TECH_STACK.md
│   │
│   ├── database/
│   │   ├── SCHEMA_OVERVIEW.md                   # 17 tables explained
│   │   ├── ENTITY_RELATIONSHIPS.md              # Visual/text relationships
│   │   ├── TABLE_STRUCTURES.md                  # Column definitions (no SQL)
│   │   └── DESIGN_DECISIONS.md                  # Why I designed it that way
│   │
│   ├── features/
│   │   ├── student_management.md
│   │   ├── attendance_tracking.md
│   │   ├── fee_management.md
│   │   ├── exam_tracking.md
│   │   ├── sms_system.md
│   │   └── backup_restore.md
│   │
│   ├── ui_ux/
│   │   ├── USER_FLOW.md                         # Step-by-step user journey
│   │   ├── SCREEN_STRUCTURE.md                  # Navigation hierarchy
│   │   └── DESIGN_PHILOSOPHY.md                 # Simple, offline-first UX
│   │
│   └── technical/
│       ├── OFFLINE_SYNC_STRATEGY.md
│       ├── BACKGROUND_WORKERS.md                # WorkManager implementation
│       ├── LICENSING_SYSTEM.md                  # How licensing works
│       └── SMS_QUEUE_SYSTEM.md                  # Priority-based queue
│
├── screenshots/
│   ├── dashboard.png
│   ├── student_list.png
│   ├── add_student.png
│   ├── attendance_marking.png
│   ├── fee_collection.png
│   ├── exam_results.png
│   ├── sms_settings.png
│   ├── backup_screen.png
│   └── calendar_view.png
│
├── videos/
│   ├── quick_demo.mp4                           # 1-minute overview
│   ├── attendance_demo.mp4                      # Marking attendance
│   ├── fee_demo.mp4                             # Collecting fees
│   └── sms_demo.mp4                             # Sending notifications
│
├── assets/
│   ├── app_logo.png
│   ├── app_icon.png
│   ├── feature_graphic.png
│   └── promo_banner.png
│
├── architecture_diagrams/
│   ├── app_architecture.png                     # MVVM + Clean Architecture
│   ├── database_schema.png                      # Visual DB design
│   ├── user_flow.png                            # User journey map
│   └── sms_workflow.png                         # SMS queue system
│
├── user_guide/
│   ├── getting_started.md
│   ├── student_management_guide.md
│   ├── attendance_guide.md
│   ├── fee_management_guide.md
│   ├── exam_guide.md
│   ├── sms_guide.md
│   └── backup_guide.md
│
├── business/
│   ├── MONETIZATION_STRATEGY.md                 # Pricing model
│   ├── TARGET_MARKET.md                         # Bangladeshi teachers
│   ├── COMPETITOR_ANALYSIS.md
│   └── METRICS_AND_GROWTH.md
│
├── CHANGELOG.md                                 # Version history
├── NOTICE.txt                                  
├── ROADMAP.md                                   # Future plans (from your docs)
├── SECURITY.md
└── LICENSE.md                                   # Proprietary - All rights reserved

```

---

## 📚 Documentation

- [Product Overview](./docs/OVERVIEW.md)
- [Problem Statement](./docs/PROBLEM_STATEMENT.md)
- [Tech Stack](./docs/architecture/TECH_STACK.md)
- [Database Design](./docs/database/SCHEMA_OVERVIEW.md)
- [User Guide](./user_guide/getting_started.md)

---

## 🔐 License

This is a **proprietary commercial product**. The source code is not public. This repository contains **documentation, screenshots, and architecture details only**.

**© 2024 MD SHAHAJALAL MAHMUD. All Rights Reserved.**

---

## 📞 Contact

**Developer:** MD SHAHAJALAL MAHMUD
**Email:** mahmud.nubtk@gmail.com  
**Portfolio:** [Portfolio](https://shahajalalmahmud.netlify.app/)

---

<div align="center">

**⭐ This showcase demonstrates what a single student developer can build**

_Built with Kotlin, Jetpack Compose, and Room_

</div>
