# Security Policy

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability in Amar Batch, please report it responsibly.

**DO NOT** create a public GitHub issue. Instead, report via:

- **Email**: mahmud.nubtk@gmail.com (preferred)
- **GitHub Security Advisories**: Use "Report a vulnerability" feature

Please include:

- Description of the vulnerability
- Steps to reproduce
- Potential impact

**Response Timeline:**

- Acknowledgment: Within 48 hours
- Initial assessment: Within 5 business days
- Status updates: Every 7 days until resolved

---

## Supported Versions

| Version        | Supported          |
| -------------- | ------------------ |
| 1.x.x (Latest) | ✅ Yes             |
| Beta versions  | ⚠️ Limited support |

Only the latest version receives security updates. Users should always update to the newest version.

---

## Security Best Practices

### For Users

- ✅ Download only from trusted sources (Google Play Store)
- ✅ Keep your device's Android OS updated
- ✅ Use device lock screen (PIN/pattern/fingerprint)
- ✅ Enable Google Play Protect
- ✅ Backup regularly to Google Drive
- ❌ Never install modified/cracked versions
- ❌ Don't root your device unnecessarily

### For Developers (Contributing)

This repository contains **documentation only**. The actual app source code is private.

If you are a developer reviewing this showcase:

- Never hardcode secrets or API keys
- Always validate user input
- Use parameterized database queries
- Enable ProGuard for release builds

---

## Data Privacy

### What Data is Stored

**On the device (local only)**:

- Student names and phone numbers
- Attendance records
- Fee transactions
- Exam marks
- SMS history

**Not Collected:**

- No data is collected on external servers
- No user tracking or analytics
- No third-party data sharing
- No telemetry

### Data Storage

- **Location**: Device only (SQLite database)
- **Backup**: Optional Google Drive (user's own account)
- **Transmission**: No network transmission except optional backup

### User Rights

Users have full control:

- **View**: All data visible in the app
- **Export**: Backup to Google Drive or file
- **Modify**: Edit any information
- **Delete**: Uninstalling removes all data

---

## Known Security Considerations

| Risk                    | Mitigation                                                                             |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **Device theft/loss**   | Relies on Android's Full Disk Encryption; device lock screen recommended               |
| **Rooted devices**      | Standard Android security applies; sensitive data not collected                        |
| **SMS content**         | Teacher controls all messages; no sensitive information auto-sent; SMS can be disabled |
| **Google Drive backup** | User's personal Google account; standard Google security applies                       |

---

## Security Contact

**Email**: mahmud.nubtk@gmail.com
**Response Time**: Within 48 hours

---

## Policy Updates

This security policy is reviewed quarterly.

**Last Updated**: May 2026
**Version**: 1.0

---

_Questions about security? Contact the developer directly._
