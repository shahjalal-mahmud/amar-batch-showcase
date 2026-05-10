# Roadmap - Amar Batch

## Current Status: Beta Testing (Version 1.0.0)

The app is **fully built** and currently in beta testing with real teachers. We are in the refinement phase — fixing bugs, improving UX, and adding polish based on teacher feedback.

---

## ✅ Completed (Development Phase)

### Planning & Design (Dec 2025 - Jan 2026)

- [x] Market research with Bangladeshi teachers
- [x] Feature prioritization for MVP
- [x] User flow design
- [x] UI/UX mockups
- [x] Database schema design (17 tables)
- [x] Architecture planning (Clean MVVM)

### Core Development (Jan 2026 - Feb 2026)

- [x] Project setup and configuration
- [x] Room database implementation
- [x] Student management (CRUD + CSV import)
- [x] Class + Shift organization structure
- [x] Attendance system with calendar view
- [x] Fee management with due calculation
- [x] Exam and marks tracking
- [x] SMS system with queue and templates
- [x] Google Drive backup and restore
- [x] Licensing and trial system
- [x] Expense tracking
- [x] 800+ unit tests

---

## 🔄 Current Phase: Beta Testing (Feb 2026 - Mar 2026)

**Status: IN PROGRESS**

### Beta.5 (Current - Feb 25, 2026)

- [x] Performance optimization for large batches
- [x] Enhanced CSV import error messages
- [x] SMS queue reliability improvements
- [x] Database query optimizations

### Beta.6 (Target: Mar 5, 2026)

- [ ] Final performance benchmarking
- [ ] Crash reporting setup
- [ ] Edge case bug fixes
- [ ] Bangla error messages for common issues

### Beta.7 (Target: Mar 10, 2026)

- [ ] User guide completion
- [ ] In-app tutorial videos
- [ ] Feedback collection improvements
- [ ] Accessibility enhancements

### Beta Metrics

| Metric              | Current | Target |
| ------------------- | ------- | ------ |
| Active beta testers | 8       | 15+    |
| Bug fix rate        | 90%     | 99%    |
| Crash-free rate     | 98.5%   | 99.9%  |

---

## 🚀 Next: Public Release (March 2026)

### Version 1.0.0-rc.1 (Release Candidate) - Target: Mar 15, 2026

- [ ] All beta bugs fixed
- [ ] Memory leak testing completed
- [ ] Performance benchmarks met
- [ ] Documentation finalized
- [ ] Privacy policy and terms finalized
- [ ] Support system ready

### Version 1.0.0 (Public Release) - Target: Mar 31, 2026

- [ ] Google Play Store listing created
- [ ] App screenshots and descriptions ready
- [ ] Release APK signed and tested
- [ ] Launch announcement prepared
- [ ] Support channels established
- [ ] Payment system (bKash/Nagad) operational

### Launch Checklist

- [ ] Play Store optimization (ASO)
- [ ] Landing page live (appriyo.com/app/amarbatch)
- [ ] Demo videos recorded
- [ ] User guide PDF ready
- [ ] Support email monitored
- [ ] Feedback collection system active

---

## 📈 Post-Launch Roadmap

### Version 1.1.0 (Q2 2026 - Estimated: May 2026)

**Focus: Reports & Export**

| Feature                                | Status     | Priority |
| -------------------------------------- | ---------- | -------- |
| Export reports to PDF                  | 📅 Planned | High     |
| Export reports to Excel                | 📅 Planned | High     |
| Enhanced expense categories            | 📅 Planned | Medium   |
| Profit/Loss dashboard                  | 📅 Planned | Medium   |
| Home screen widget (quick attendance)  | 📅 Planned | Low      |
| Multiple batch support for one teacher | 📅 Planned | High     |

### Version 1.2.0 (Q3 2026 - Estimated: August 2026)

**Focus: Reach & Accessibility**

| Feature                           | Status     | Priority |
| --------------------------------- | ---------- | -------- |
| Web dashboard (basic)             | 📅 Planned | High     |
| Push notifications (Firebase)     | 📅 Planned | High     |
| Bangla language support (full UI) | 📅 Planned | High     |
| Dark mode improvements            | 📅 Planned | Low      |
| Data import from other apps       | 📅 Planned | Medium   |

### Version 1.3.0 (Q4 2026 - Estimated: November 2026)

**Focus: Automation & Smarts**

| Feature                                     | Status     | Priority |
| ------------------------------------------- | ---------- | -------- |
| Student promotion to next class (one-click) | 📅 Planned | High     |
| Automatic fee reminder scheduling           | 📅 Planned | High     |
| Attendance trends (weekly/monthly charts)   | 📅 Planned | Medium   |
| SMS delivery reports                        | 📅 Planned | Medium   |
| Bulk student edit/update                    | 📅 Planned | Low      |

---

## 🎯 Long-term Vision (2027+)

### Version 2.0.0 - Coaching Center Edition

**Target: 2027**

| Feature                       | Description                              |
| ----------------------------- | ---------------------------------------- |
| Multi-teacher accounts        | Multiple teachers under one subscription |
| Role-based access             | Owner, admin, teacher roles              |
| Student & parent login portal | Parents view attendance, fees, results   |
| Central dashboard             | Overview of all batches and teachers     |
| Advanced analytics            | Revenue, attendance, performance trends  |
| API for school integration    | Connect with existing school systems     |

### Version 3.0.0 - Platform Expansion

**Target: 2028+**

| Feature                     | Description                             |
| --------------------------- | --------------------------------------- |
| iOS app (Swift/KMM)         | Reach teachers on iPhones               |
| Desktop app (Windows/Mac)   | For teachers who prefer computers       |
| SMS gateway integration     | Alternative to teacher's SIM            |
| Payment gateway integration | Auto-reminders and online collection    |
| AI-powered insights         | Predict attendance issues, fee defaults |

---

## 📊 Feature Prioritization Framework

| Priority     | Meaning                  | Timeline         |
| ------------ | ------------------------ | ---------------- |
| **Critical** | Must have for launch     | Version 1.0.0    |
| **High**     | Urgent user need         | Within 3 months  |
| **Medium**   | Important but not urgent | Within 6 months  |
| **Low**      | Nice to have             | Within 12 months |

---

## 🔄 Feature Status Key

| Symbol | Meaning              |
| ------ | -------------------- |
| ✅     | Completed            |
| 🔄     | In Progress          |
| 📅     | Planned              |
| 🔮     | Future Consideration |
| ❌     | Not Planned          |

---

## 📝 Planned Features (Detailed)

### Coming in Version 1.1.0

#### Export to PDF/Excel

- Generate monthly attendance reports
- Export fee collection summaries
- Student performance reports
- Custom date range selection

#### Enhanced Expenses

- Categorize expenses (books, rent, materials)
- Attach receipts (photos)
- Monthly expense trends
- Profit calculation after expenses

#### Multiple Batches

- Switch between different batches
- Separate data per batch
- Combined dashboard view
- Per-batch pricing

### Coming in Version 1.2.0

#### Web Dashboard

- View reports on any device
- Basic student management
- No offline capability (view-only)
- Secure login with OTP

#### Push Notifications

- Free alternative to SMS
- Works on internet connection
- Attendance reminders
- Fee due alerts

#### Bangla Language

- Complete UI translation
- Bangla SMS templates
- Bangla date formats
- Local number formatting

---

## 🐛 Bug Tracking Priority

| Severity     | Response Time | Example                           |
| ------------ | ------------- | --------------------------------- |
| **Critical** | 24 hours      | App crashes, data loss            |
| **High**     | 3 days        | Feature broken, major UI issue    |
| **Medium**   | 1 week        | Minor UI glitch, slow performance |
| **Low**      | 2 weeks       | Typo, cosmetic issue              |

---

## 📈 Success Metrics

### Launch Goals (Version 1.0.0)

- [ ] 100+ downloads in first month
- [ ] 4.5+ star rating on Play Store
- [ ] < 0.5% crash rate
- [ ] 10+ paid subscriptions

### First Year Goals (2026)

- [ ] 500+ active users
- [ ] 25% conversion rate (trial → paid)
- [ ] 4.8+ star rating
- [ ] 100+ paid subscriptions

### Second Year Goals (2027)

- [ ] 2,000+ active users
- [ ] Coaching center edition launch
- [ ] Web dashboard live
- [ ] 500+ paid subscriptions

---

## 💬 Feedback Collection

### How We Gather Feedback

| Channel                | Purpose                        |
| ---------------------- | ------------------------------ |
| Beta tester interviews | Deep insights on UX            |
| In-app feedback form   | Bug reports + feature requests |
| Email support          | Direct user communication      |
| Google Play reviews    | Public sentiment               |
| WhatsApp group         | Quick iteration feedback       |

### Feature Request Process

1. User submits request (email/feedback form)
2. Categorized and prioritized
3. Added to roadmap if matches vision
4. User notified when implemented

---

## 🔄 Release Cadence

| Version Type      | Frequency         | Example       |
| ----------------- | ----------------- | ------------- |
| **Patch** (1.0.x) | As needed (bugs)  | 1.0.0 → 1.0.1 |
| **Minor** (1.x.0) | Every 2-3 months  | 1.0.0 → 1.1.0 |
| **Major** (x.0.0) | Every 6-12 months | 1.x.x → 2.0.0 |

---

## 📅 Key Dates (Targets)

| Milestone                        | Date               |
| -------------------------------- | ------------------ |
| Beta testing ends                | March 15, 2026     |
| Release Candidate                | March 15, 2026     |
| **Public Release**               | **March 31, 2026** |
| Version 1.1.0                    | May 31, 2026       |
| Version 1.2.0                    | August 31, 2026    |
| Version 1.3.0                    | November 30, 2026  |
| Version 2.0.0 (Coaching Edition) | June 30, 2027      |

---

## 🤝 How to Contribute

### For Beta Testers

- Email mahmud.nubtk@gmail.com with subject "Beta Testing"
- Include your device model and Android version
- Report bugs with screenshots when possible

### For Feature Suggestions

- Email with subject "Feature Request: [name]"
- Describe the problem you're solving
- Explain how it helps teachers

### For Security Issues

- Use private reporting (see SECURITY.md)
- Do NOT create public issues

---

## 📞 Questions About Roadmap?

**Developer:** MD SHAHAJALAL MAHMUD
**Email:** mahmud.nubtk@gmail.com

---

_Last Updated: May 2026_
_Current Version: 1.0.0-beta.5_
_Next Release: 1.0.0-beta.6 (March 5, 2026)_

---

<div align="center">

**🎯 Building the best tool for Bangladeshi teachers — one feature at a time**

</div>
