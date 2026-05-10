# Offline-First Strategy

## Why Offline-First?

Amar Batch is designed for teachers in Bangladesh who often face:

- **Unreliable internet connectivity** in rural and semi-urban areas
- **Expensive mobile data** — not something teachers want to spend on
- **Classrooms without WiFi** — most teaching happens offline

**The principle:** Internet is a bonus, not a requirement. The app must work 100% without it.

---

## Core Strategy

### Single Source of Truth

**Room database (SQLite) is the ONLY source of truth.**

- All data is stored locally on the device
- UI never holds its own copy of data — it just observes Room via Flows
- No caching layer between Room and UI
- No race conditions between local and remote state

### What Internet is Used For (Optional)

| Feature                 | Requires Internet? | Offline Fallback                     |
| ----------------------- | ------------------ | ------------------------------------ |
| Mark attendance         | ❌ No              | Works completely offline             |
| Add students            | ❌ No              | Works completely offline             |
| Collect fees            | ❌ No              | Works completely offline             |
| Record exam marks       | ❌ No              | Works completely offline             |
| Send SMS                | ❌ No              | Uses SIM, not internet               |
| **Google Drive backup** | ✅ Yes             | Manual backup to local storage       |
| **License validation**  | ⚠️ Occasional      | Works offline, validates when online |

---

## Data Flow Architecture

```
User Action (e.g., mark attendance)
↓
UI sends event to ViewModel
↓
ViewModel calls Repository
↓
Repository writes to Room DAO
↓
Room database updated (SQLite)
↓
Room Flow emits new data
↓
Repository maps Entity → Domain Model
↓
ViewModel receives new StateFlow value
↓
UI recomposes with fresh data

NO INTERNET REQUIRED AT ANY STEP
```

---

## Key Implementation Decisions

### 1. Room is the Only Database

| Decision          | Why                                  |
| ----------------- | ------------------------------------ |
| No cloud database | Offline-first means local is primary |
| No separate cache | Room Flow IS the cache               |
| No sync conflicts | No remote data to conflict with      |

### 2. Reactive UI with Flows

```kotlin
// In DAO — Room automatically emits when data changes
@Query("SELECT * FROM students WHERE shift_id = :shiftId")
fun getStudents(shiftId: Long): Flow<List<StudentEntity>>

// In Repository — map to domain models
fun getStudents(shiftId: Long): Flow<List<Student>> =
    dao.getStudents(shiftId).map { entities ->
        entities.map { it.toDomain() }
    }

// In ViewModel — collect and expose as StateFlow
val students: StateFlow<List<Student>> =
    repository.getStudents(shiftId).stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )
```

### 3. No Hard Deletions

Data is never permanently deleted from the database:

- `is_active = false` instead of `DELETE`
- Historical records preserved for reports
- Users can "hide" inactive students

### 4. Soft State Management

All UI state is derived from Room data:

| State Type        | Source                    |
| ----------------- | ------------------------- |
| Student list      | Room query → Flow         |
| Attendance status | Room query → Flow         |
| Fee due status    | Calculated from Room data |
| Exam marks        | Room query → Flow         |

---

## Backup Strategy (Optional Internet)

Backup is **optional** — not required for app functionality.

```
Manual Backup → User taps "Backup" → Saves to local storage
Auto Backup (when internet available) → User's Google Drive → Private folder
Restore (when internet available) → From Google Drive → Local database
```

### Backup Design Principles

- **User-controlled** — explicit consent required for Google Drive access
- **User's own storage** — data goes to teacher's personal Google Drive, not our servers
- **No automatic sync** — backup is one-way, not continuous sync
- **Restore is explicit** — user chooses when to restore

---

## What We Avoided

| Pattern                      | Why We Avoided It                             |
| ---------------------------- | --------------------------------------------- |
| **Cloud-first sync**         | Teachers have unreliable internet             |
| **Real-time collaboration**  | Not needed for single teacher                 |
| **Offline queue for server** | No server to queue to                         |
| **Conflict resolution**      | No conflicts when there's only one source     |
| **Optimistic updates**       | Not needed — writes are immediate to local DB |

---

## Offline-First Benefits

| Benefit                  | How It Helps Teachers                                       |
| ------------------------ | ----------------------------------------------------------- |
| **No downtime**          | App works in villages, remote areas, underground classrooms |
| **No data costs**        | No internet required for daily use                          |
| **Fast performance**     | Local SQLite is faster than network calls                   |
| **Privacy**              | Data never leaves the device unless user chooses backup     |
| **No server dependency** | App continues working even if our services are down         |

---

## Limitations (Acceptable for MVP)

| Limitation                 | Why Acceptable                        |
| -------------------------- | ------------------------------------- |
| No multi-device sync       | Teachers use one primary device       |
| No web access              | Teachers use phone, not computer      |
| Manual backup required     | Teachers can backup weekly or monthly |
| No real-time parent portal | SMS provides async communication      |

---

## Future Enhancements (Not in MVP)

When internet becomes more reliable or teachers request it:

| Feature                   | When                              |
| ------------------------- | --------------------------------- |
| Optional cloud sync       | If teachers want multiple devices |
| Web dashboard (view-only) | If teachers request it            |
| Real-time parent app      | Future expansion                  |

**Note:** These will be **add-ons**, not replacements for offline-first design.

---

## Key Takeaway

> **"The app works exactly the same with or without internet."**

Internet is used only for:

1. Optional Google Drive backup
2. Occasional license validation

Everything else — attendance, fees, exams, SMS — works completely offline.

---

_Last Updated: May 2026_
