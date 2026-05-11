# Frequently Asked Questions (FAQ)

Got questions? We've got answers. This FAQ covers the most common questions about Amar Batch.

---

## 📱 General Questions

### Q1: What is Amar Batch?

**A:** Amar Batch is an offline-first Android app for Bangladeshi teachers to manage student attendance, tuition fees, exam results, and communicate with parents via SMS. It's designed to be simple, reliable, and respectful of teachers' time.

### Q2: Who is Amar Batch for?

**A:** Amar Batch is built for:

- Home tuition teachers
- Single-subject batch teachers (Class 1-12)
- Freelance tutors
- Small coaching center instructors

It is **not** designed for large coaching centers or schools with complex administrative needs.

### Q3: Is Amar Batch free?

**A:** Amar Batch offers a **free trial period** (1-2 months, configurable). After the trial, a **paid yearly subscription** is required. Pricing: ৳999/year for a single batch.

### Q4: Do I need an internet connection?

**A:**

- **Most features work offline** (attendance, fees, student records)
- **Internet needed for:** License validation (occasionally), Google Drive backup, Firebase authentication
- **SMS:** Uses your SIM card - internet not required
- **You can use the app 90% of the time without internet**

### Q5: Is my data safe?

**A:** Yes. All data is stored **locally on your device** only. We do not collect, upload, or share your data. Optional Google Drive backups are encrypted and stored in **your personal Google Drive** (not our servers). No analytics, no tracking, no ads.

### Q6: What happens if I lose my phone?

**A:**

- Without backup → Data is lost (like a lost diary)
- **With Google Drive backup** → You can restore data on a new phone
- **Recommendation:** Enable automatic backups in Settings

---

## 💾 Backup & Data

### Q7: Does Amar Batch automatically back up my data?

**A:** Yes, if you enable it. Go to **Settings → Backup** to:

- Enable automatic Google Drive backups
- Set backup frequency (daily, weekly, monthly)
- Test your backup

### Q8: How do I restore data after changing phones?

**A:**

1. Install Amar Batch on new phone
2. During setup, choose **"Restore from Google Drive"**
3. Sign in to the same Google account
4. Select your backup file
5. Wait for restore to complete

**Note:** Backup and restore use your personal Google Drive. Keep your Google account credentials secure.

### Q9: Can I backup to something other than Google Drive?

**A:** Not currently. Google Drive is the only cloud backup option. Manual backup to device storage is also available (Settings → Backup → Manual Backup).

### Q10: How often should I backup?

**A:**

- **Daily recommended** if you enter data every day
- **Weekly minimum** for most teachers
- **After every fee collection session** (best practice)

### Q11: Can I export data to Excel/PDF?

**A:** This feature is planned for **Phase 4 (Post-Launch)** . For MVP, you can view reports within the app.

---

## 💰 Fees & Payments

### Q12: How does fee tracking work?

**A:**

- Set monthly fee per class/shift
- Record payments by selecting student(s) and months paid
- App automatically calculates due amounts
- Simple, no complex accounting

### Q13: Can I track partial payments?

**A:** No. You cannot record partial payment amount.

### Q14: Can I track expenses?

**A:** Yes. Amar Batch includes flat expense tracking (e.g., photocopy, transport, snacks). Not designed for detailed business accounting.

### Q15: What happens if a student leaves mid-month?

**A:** You can mark a student as **inactive** (soft delete). Their records remain in the database for history but don't appear in active lists. Fee adjustments are manual (record partial payment or refund).

---

## 📝 Attendance

### Q16: How do I take attendance?

**A:** Two modes available:

- **Default Present:** Only mark absent students (fastest for most classes)
- **Default Absent:** Only mark present students (useful for large classes with few attendees)

Attendance takes **under 1 minute** for a typical batch.

### Q17: Can I edit past attendance?

**A:** Yes. Go to the **Attendance Calendar**, select the date, and edit any student's attendance record.

### Q18: Can I see attendance percentage?

**A:** Yes. Student profiles show attendance percentage, and monthly summaries are available.

---

## 📱 SMS System

### Q19: Does SMS cost money?

**A:** **YES.** Amar Batch uses **your phone's SIM card** to send SMS. Your mobile carrier (Grameenphone, Robi, Banglalink, Teletalk) charges you normally, just as if you sent the message manually from your messaging app.

**The developer does not charge for SMS.** All costs go to your carrier.

### Q20: How much does each SMS cost?

**A:** Depends on your mobile carrier's rates. Typically ৳0.50–৳1.00 per SMS in Bangladesh. Check with your carrier for exact pricing. For Example: RObi Sim: 500 SMS 60tk for 1 month.

### Q21: Can I use dual SIM phones?

**A:** Yes. In SMS settings, you can select which SIM card to use for Amar Batch messages. Ensure the selected SIM has sufficient balance.

### Q22: What types of SMS can I send?

**A:** 7 types:

1. Admission welcome message
2. Fee due reminder
3. Fee payment confirmation
4. Absent notification
5. Exam result notification
6. Monthly attendance summary
7. Emergency Notices Bulk send

### Q23: Can I customize SMS templates?

**A:** Yes. Go to **Settings → SMS Templates** to edit any message template.

### Q24: Can I disable SMS for certain messages?

**A:** Yes. You can enable/disable SMS globally or per class/student in Settings.

### Q25: What if an SMS fails to send?

**A:** Amar Batch has a **background SMS queue** with retry logic. Failed messages are retried automatically. You can view failed messages in SMS history.

---

## 🔐 License & Account

### Q26: How do I register for Amar Batch?

**A:**

1. Install the app
2. Open and complete onboarding
3. Enter your email and phone number
4. Start the free trial

### Q27: How long is the free trial?

**A:** 1-2 months (configurable). You'll see remaining trial days in Settings.

### Q28: How do I pay for the subscription?

**A:**

- Payment methods: bKash or Nagad (Bangladesh)
- Price: ৳999/year
- Process: Contact support at mahmud.nubtk@gmail.com to arrange payment
- After payment, developer activates your license within 24 hours

### Q29: Can I use Amar Batch on multiple phones?

**A:** No. The license is **device-bound** (one license per phone). For multiple devices, you need separate licenses.

### Q30: What happens when my license expires?

**A:**

- **3-day grace period** after expiry
- During grace period, full functionality remains
- After grace period, the app locks and requires renewal
- Your data remains safe (no deletion)

### Q31: Can I get a refund?

**A:** Refunds are handled case-by-case. Contact support within 7 days of payment. No refunds after 7 days or if app has been actively used.

---

## 📚 Features & Limitations

### Q32: Can I manage multiple batches?

**A:** **MVP supports a single batch only.** Multi-batch support is planned for **Phase 4 (Post-Launch)** .

### Q33: Does it support Bangla language?

**A:** Not yet. MVP is English-only. Bangla language support is planned for a future update.

### Q34: Can I import students from Excel/CSV?

**A:** Yes! You can bulk import students using a CSV file. Template available in the app.

### Q35: Can I attach files or photos to student records?

**A:** Not in MVP. Future versions may support document attachments.

### Q36: Does the app have ads?

**A:** **NO.** Amar Batch is completely ad-free. We believe teachers deserve a distraction-free tool.

### Q37: Can I use Amar Batch on iPhone?

**A:** No. Amar Batch is Android-only. No iOS version is currently planned.

### Q38: What Android versions are supported?

**A:** Android 7.0 (API 24) through Android 15 (API 35). Most phones from 2017 onward work.

---

## 🐛 Problems & Troubleshooting

### Q39: The app is crashing. What should I do?

**A:**

1. Restart your phone
2. Clear app cache (Settings → Apps → Amar Batch → Clear Cache)
3. Update to latest version
4. If problem persists, contact support with details

### Q40: I forgot my backup Google account. How do I recover?

**A:** We cannot recover your Google account credentials. Try Google's account recovery. Without the original Google account, backups cannot be restored.

### Q41: My SMS isn't sending. What's wrong?

**A:** Check:

- SIM has sufficient balance
- Correct SIM selected in SMS settings
- Phone has network signal
- SMS feature is enabled in settings
- You haven't exceeded carrier daily SMS limits

### Q42: Attendance/fee data looks wrong. How to fix?

**A:**

- Check if student is active (inactive students don't appear in some lists)
- Verify class/shift assignments
- Edit individual records manually
- If widespread issue, contact support with screenshots

### Q43: The app says "License Expired" but I paid.

**A:**

- Ensure you have internet connection (for license check)
- Wait 5 minutes (license activation can take time)
- Contact support with your payment details

### Q44: Can I recover deleted data?

**A:** No. Deletion is permanent unless you have a backup. Always backup before major deletions.

---

## 🔒 Privacy & Security

### Q45: Does Amar Batch collect my personal data?

**A:** **NO.** We collect:

- ✅ Your email (for license and support)
- ✅ Basic usage data (anonymized, for debugging only)

We do **NOT** collect:

- ❌ Student names or phone numbers
- ❌ Attendance or fee records
- ❌ SMS content
- ❌ Location data
- ❌ Contact lists

### Q46: Is my Google Drive data safe?

**A:** Yes. Backups are stored in **your personal Google Drive** with your encryption. We cannot access your Drive without your permission.

### Q47: Can other teachers see my data?

**A:** No. All data is local to your device. No sharing or syncing between devices.

### Q48: What if my phone is stolen?

**A:**

- **If backed up** → Restore on new phone
- **If not backed up** → Data is lost (like a physical diary)
- **Remotely wipe data?** Not possible (no internet-based wipe feature)

---

## 🚀 Future Updates

### Q49: When will multi-batch support be available?

**A:** Planned for Phase 4 (Post-Launch), approximately 3-6 months after public release.

### Q50: Will there be a web version?

**A:** Not currently planned. Focus is on Android mobile-first experience.

### Q51: Can I request a feature?

**A:** Absolutely! Open a GitHub Issue with label `enhancement` or email mahmud.nubtk@gmail.com. We prioritize features requested by multiple teachers.

---

## 📞 Still Have Questions?

If your question isn't answered here:

- 📧 Email: mahmud.nubtk@gmail.com
- 💬 WhatsApp: 01889793146
- 🐛 GitHub Issues: [github.com/shahjalal-mahmud/Amar_Batch/issues](https://github.com/Appriyo/amar-batch-showcase/issues)

**Before contacting support, please check:**

- [User Guide](docs/USER_GUIDE.md) for step-by-step instructions
- [SUPPORT.md](SUPPORT.md) for response times and procedures

---

**Amar Batch — Clear answers for busy teachers** ❤️

---

_Last Updated: May 2026_
