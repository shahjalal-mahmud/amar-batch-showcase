# Screen Structure — Amar Batch

> Navigation hierarchy and screen organization.

---

## App-Level Navigation States

Before reaching the main app, the user passes through a linear gate of screens based on their state:

```
App Launch
    │
    ▼
Splash Screen
    │
    ├── No teacher on device ──────────────────► Onboarding Screen
    │                                                   │
    │                                                   ▼
    │                                            Google Sign-In
    │                                                   │
    │                                    ┌──────────────┴──────────────┐
    │                                    │                             │
    │                               New User                    Existing User
    │                            (fill details)               (restore from Drive)
    │                                    │                             │
    │                                    └──────────────┬──────────────┘
    │                                                   │
    │                                                   ▼
    │                                             Restore Screen
    │                                          (shows restore progress)
    │                                                   │
    ├── Teacher exists, license valid ─────────────────►┤
    │                                                   │
    │                                                   ▼
    └── License blocked / expired ──────► License Blocked Screen
                                          (contact info + device ID)
```

Once through this gate, the teacher lands on the main scaffold with the bottom navigation bar.

---

## Main Navigation — Bottom Bar

Five tabs are always visible in the bottom navigation bar:

| Tab     | Route        | Purpose                                    |
| ------- | ------------ | ------------------------------------------ |
| Home    | `dashboard`  | Monthly overview, quick stats, expenses    |
| Classes | `classes`    | Student lists organized by class and shift |
| Attend  | `attendance` | Mark daily attendance, view calendar       |
| Fees    | `fees`       | Collect payments, view dues                |
| Exams   | `exams`      | Create exams, enter marks                  |

Two routes hide the bottom bar entirely and go full-screen: the global search overlay and the notifications screen.

---

## Screen Hierarchy by Tab

### Home Tab

```
Dashboard
    ├── Global Search (full-screen overlay)
    ├── Expense History
    ├── Profile
    │       └── Edit Profile
    └── Settings (More)
            ├── SMS Settings
            ├── SMS Actions
            │       ├── Fee Reminder
            │       ├── Attendance Summary
            │       ├── Exam Marks SMS
            │       └── Notice SMS
            ├── SMS Templates (list)
            │       └── Edit Template (per type)
            ├── Backup & Restore
            ├── License Info
            ├── Contact Us
            ├── About
            ├── Privacy Policy
            └── Terms & Conditions
```

---

### Classes Tab

```
Classes Home
(all classes and shifts listed)
    │
    ├── Class Students Screen (by class)
    │       ├── Class Detail (edit class info)
    │       ├── Shift Detail (edit shift info)
    │       └── Student Detail
    │               └── Edit Student
    │
    ├── Class Students Screen (by shift, direct deep-link)
    │
    └── Bulk Admit Screen (CSV import)
```

Students are accessed through their class and shift. There is no standalone "all students" screen — students always appear in the context of a class.

---

### Attend Tab

```
Attendance Home
(class/shift selector + mark attendance)
    │
    └── Attendance Calendar
            └── Session Detail
                (view or edit a specific past session)
```

---

### Fees Tab

```
Fee Home
(student list with payment status)
    │
    └── Fee History
        (full transaction log, filterable by class/month)
```

---

### Exams Tab

```
Exams Home
(list of exams per class/shift)
    │
    └── Exam Detail
        (enter marks, view results, send SMS)
```

---

## Navigation Patterns

**Push navigation** — most screens are pushed onto the back stack. The back button or back arrow returns to the previous screen.

**Tab switching** — switching between the five bottom tabs saves and restores each tab's scroll and navigation state independently. Returning to a tab picks up where the teacher left off.

**Deep links within tabs** — some screens can be reached from multiple entry points. Student Detail, for example, is reachable from the Classes tab (via class → shift → student) and from Global Search (directly by student ID).

**Full-screen overlays** — Global Search and Notifications hide the bottom bar and take over the full screen. The back gesture dismisses them and restores the previous tab.

---

## Screen Count Summary

| Category                | Screens |
| ----------------------- | ------- |
| Pre-auth / onboarding   | 4       |
| Main bottom tabs        | 5       |
| Dashboard & profile     | 3       |
| Classes & students      | 5       |
| Attendance              | 3       |
| Fees                    | 2       |
| Exams                   | 2       |
| SMS actions & templates | 6       |
| Settings & legal        | 6       |
| **Total**               | **36**  |

---

## Related Documentation

- [User Flow](./USER_FLOW.md) — Step-by-step journeys through each feature
- [Design Philosophy](./DESIGN_PHILOSOPHY.md) — UX decisions behind the screen structure
