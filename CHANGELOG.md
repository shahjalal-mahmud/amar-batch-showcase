# Changelog - Amar Batch

**Current Status: ✅ Beta Testing | Version 1.0.0**

This changelog tracks all important changes to Amar Batch. Since we're in beta testing with real teachers, this log will be updated regularly based on feedback and improvements.

---

## 📋 About This Changelog

We follow [Keep a Changelog](https://keepachangelog.com/) format so it's easy to read for everyone - teachers, testers, and recruiters alike.

**Categories you'll see:**

- **Added**: New features that didn't exist before
- **Changed**: Improvements to existing features
- **Fixed**: Problems that have been solved
- **Security**: Security-related updates
- **Coming Soon**: Features we're planning

---

## 🚀 Current Status

### Version 1.0.0 (Beta Testing - Live)

**The app is COMPLETE and currently in beta testing with real teachers.** All planned MVP features are implemented. We're now in the refinement phase - fixing bugs, improving UX, and adding polish based on teacher feedback.

**Trial Period:** Every new user gets 30-60 days free trial (configurable by developer)

**Monetization:** After trial, ৳999/year license required

---

## 📦 Full Release Notes

### Version 1.0.0-beta.5 (February 25, 2026)

#### Changed

- Improved attendance marking speed for batches with 50+ students
- Enhanced CSV import error messages (teachers now know exactly which row has issues)
- Updated SMS queue to handle network disconnections more gracefully
- Optimized database queries for faster dashboard loading

#### Fixed

- Fixed rare crash when adding student with special characters in name
- Resolved issue where backup would fail on devices with Android 14+
- Corrected fee calculation for months spanning across years (December to January)
- Fixed calendar view not showing attendance on certain screen sizes
- Resolved SMS template save issue on devices with Bangla keyboard

#### Security

- Enhanced license validation to prevent offline tampering
- Improved Google Drive authentication flow

---

### Version 1.0.0-beta.4 (February 20, 2026)

#### Added

- **New:** Trial period expiry warning (shows 7 days, 3 days, and 1 day before)
- **New:** In-app notification when backup completes successfully
- **New:** Option to retry failed SMS messages manually

#### Changed

- Redesigned fee collection screen for faster payment entry
- Improved search functionality (now searches both name and parent phone)
- Reduced app size by 15% through image optimization
- Default trial period changed from 60 days to configurable (30-60 days)

#### Fixed

- Fixed issue where attendance percentage showed wrong value
- Resolved crash when rotating screen while SMS queue was processing
- Fixed backup restore not showing progress for large databases
- Corrected exam marks input validation (now prevents marks above total)

---

### Version 1.0.0-beta.3 (February 15, 2026)

#### Added

- **New:** Bulk SMS sending for emergency notices
- **New:** Last backup timestamp on dashboard
- **New:** Pull-to-refresh on student list and attendance screens

#### Changed

- Improved onboarding flow for first-time users
- Enhanced error messages (now in simple Bangla + English)
- Made fee due list sortable by amount owed and months due
- Updated Material 3 components to latest version

#### Fixed

- Fixed rare database lock issue when marking attendance quickly
- Resolved CSV import failing for files with empty rows
- Fixed SMS character count showing incorrectly for Bangla text
- Corrected expense report calculations

---

### Version 1.0.0-beta.2 (February 10, 2026)

#### Added

- **New:** Holiday marking feature (mark entire class as holiday)
- **New:** Copy students from previous batch (for new academic year)
- **New:** Confirmation dialog before deleting any data

#### Changed

- Improved dashboard performance (loads 50% faster)
- Enhanced search with filters (by class, shift, or status)
- Made fee collection receipt more detailed (shows month breakdown)
- Updated UI for better readability on small screens

#### Fixed

- Fixed crash when adding student without parent phone number
- Resolved attendance calendar not showing correct colors
- Fixed SMS queue getting stuck on poor network
- Corrected exam result sorting order

---

### Version 1.0.0-beta.1 (February 1, 2026) - First Beta Release

#### Added

- **Student Management**
  - Add individual students with all required fields
  - Bulk import students via CSV template
  - Search students by name or parent phone
  - Class → Shift organization structure
  - Soft delete (mark inactive, preserve history)
  - Student profile with optional photo

- **Attendance System**
  - Daily attendance in under 1 minute
  - Two modes: Default Present (mark absent only)
  - Two modes: Default Absent (mark present only)
  - Calendar view with attendance history
  - Edit past attendance records
  - Monthly attendance percentage
  - Holiday marking for entire class

- **Fee Management**
  - Set monthly fees per class/shift
  - Collect single or multiple months payment
  - Automatic due calculation
  - Payment history per student
  - Monthly collection reports
  - Due fee list with sorting
  - Simple expense tracking

- **Exam Management**
  - Create exams with name, date, total marks
  - Enter marks for all students
  - Mark absent students for exams
  - Highest, lowest, average marks calculation
  - Student-wise exam history
  - Optional Google Drive link for question papers

- **SMS Communication**
  - 6 automated SMS types (welcome, fee due, payment receipt, absent, exam result, monthly summary, emergency notice)
  - Customizable SMS templates
  - Enable/disable per class
  - SMS queue with retry logic
  - Dual-SIM support
  - Uses teacher's own SIM (pay only regular SMS rate)

- **Backup & Restore**
  - Google Drive automatic backup
  - Manual backup to phone storage
  - One-tap restore
  - Backup timestamp display
  - Configurable backup frequency

- **Licensing System**
  - 30-60 day free trial (configurable by developer)
  - Yearly license option (৳999/year)
  - 3-day grace period after trial
  - Device-bound license validation

- **Core Infrastructure**
  - Offline-first architecture
  - 17-table Room database
  - MVVM + Clean Architecture
  - Jetpack Compose UI (Material 3)
  - Coroutines for async operations
  - WorkManager for background tasks
  - Koin dependency injection

---

## 📊 Beta Testing Progress

### Beta Statistics (as of February 25, 2026)

| Metric                 | Value      |
| ---------------------- | ---------- |
| Active Beta Testers    | 8 teachers |
| Total Students Managed | 240+       |
| Attendance Records     | 3,500+     |
| Fee Transactions       | 180+       |
| SMS Sent               | 450+       |
| Bug Reports Received   | 23         |
| Bugs Fixed             | 21         |
| Feature Requests       | 12         |

### Beta Tester Feedback Summary

**What teachers love:**

- "Attendance takes less than 30 seconds now"
- "SMS feature saves me so much time"
- "Never forget who paid fees anymore"
- "Backup gives me peace of mind"

**What we're improving based on feedback:**

- Making CSV template even simpler
- Adding more SMS templates
- Improving search speed for large batches

---

## 🔜 What's Next

### Before Public Release (Version 1.0.0)

#### In Progress (Current Sprint)

- [ ] Performance optimization for 100+ student batches
- [ ] Enhanced error messages in Bangla
- [ ] More comprehensive user guide
- [ ] Final security audit

#### Planned for Version 1.0.0 (Public Release)

- [ ] Play Store listing preparation
- [ ] Privacy policy and terms of service finalized
- [ ] In-app purchase integration (bKash/Nagad manual for now)
- [ ] Release candidate testing

---

## 🗓️ Release Roadmap

| Version                        | Target Date       | Status         |
| ------------------------------ | ----------------- | -------------- |
| 1.0.0-beta.1                   | February 1, 2026  | ✅ Released    |
| 1.0.0-beta.2                   | February 10, 2026 | ✅ Released    |
| 1.0.0-beta.3                   | February 15, 2026 | ✅ Released    |
| 1.0.0-beta.4                   | February 20, 2026 | ✅ Released    |
| 1.0.0-beta.5                   | February 25, 2026 | ✅ Released    |
| 1.0.0-beta.6                   | March 5, 2026     | 🔄 In Progress |

---

## 🎉 Public Release Goal

We're aiming for **July 31, 2026** as the target date for public release on Google Play Store.

**Pre-release checklist:**

- [ ] All beta bugs fixed
- [ ] Performance benchmarks met
- [ ] Documentation complete
- [ ] Support system ready
- [ ] Payment system tested

---

## 🚀 Post-Release Roadmap

### Version 1.1.0 (Q2 2026)

- Export reports to PDF and Excel
- Enhanced expense tracking with categories
- Profit/loss dashboard
- Home screen widget for quick attendance
- Multiple batch support for one teacher

### Version 1.2.0 (Q3 2026)

- Web dashboard (view reports on computer)
- Push notifications (free, uses Firebase)
- Bangla language support for all screens
- Dark mode improvements
- Data import from other apps

### Version 2.0.0 (2027)

- Coaching center edition
- Multiple teachers under one account
- Student and parent login portal
- Advanced analytics and charts
- API for school management systems

---

## 📱 How to Get Amar Batch

### For Beta Testing (Current)

1. Email mahmud.nubtk@gmail.com with subject "Beta Testing"
2. Include your name, phone model, and number of students
3. Receive APK via email
4. Install and start using
5. **30-60 days free trial included**

### For Public Release (Coming March 31, 2026)

1. Download from Google Play Store
2. Install on your Android phone
3. Complete onboarding
4. **Free trial period starts automatically**
5. After trial, pay ৳999/year to continue

---

## 💰 Trial & Pricing Details

| Period           | Cost                    | Access                        |
| ---------------- | ----------------------- | ----------------------------- |
| First 30-60 days | Free                    | All features                  |
| After trial      | ৳999/year               | All features (renew annually) |
| Never            | No lifetime free option | N/A                           |

**Note:** Trial period length is configurable by developer (currently set to 60 days)

---

## 🐛 Known Issues (Being Fixed)

### Beta.5 Known Issues

- [Minor] CSV import fails if file has completely empty columns
- [Minor] SMS queue shows "sending" forever if airplane mode on
- [Rare] Backup fails on very slow internet connections

**Workarounds available for all issues.** Fixes coming in beta.6.

---

## 📞 Support & Feedback

**For Beta Testers:**

- Email: mahmud.nubtk@gmail.com
- Response time: Within 24 hours
- Include screenshots when reporting bugs

**For Everyone:**

- GitHub Issues: [github.com/shahjalal-mahmud/amar-batch-showcase/issues](https://github.com/Appriyo/amar-batch-showcase/issues)
- Email: mahmud.nubtk@gmail.com
- Phone: 01889793146

---

## 🙏 Acknowledgments

**Beta Testers (Thank You!)**

- Md. Rashed Khan (Class 9-10 English, 45 students)
- Sharmin Akter (Home Tutor, 25 students)
- Hasan Mahmud (Math Teacher, 60 students)
- And 5 other dedicated teachers

**Development**

- Built entirely by MD SHAHAJALAL MAHMUD (3rd Year CSE Student)
- No external developers or agencies involved

---

## 📊 Version Numbering Explained

**Version 1.0.0-beta.5** means:

- **1** = First major release
- **0** = No major new features added yet
- **0** = Initial feature set complete
- **beta.5** = Fifth beta release

**Understanding our versions:**

- beta.x = Testing with real users, bugs expected
- rc.x = Release candidate, almost ready for public
- x.x.x = Public stable release

---

**Last Updated:** May 10, 2026  
**Current Version:** 1.0.0-beta.5  
**Next Release:** 1.0.0-beta.6 (March 5, 2026)

---

_"Built by a student, used by teachers, generating real revenue"_

<div align="center">

**⭐ View the code showcase | 📱 Try the app | 💰 Pay only if you love it**

</div>
