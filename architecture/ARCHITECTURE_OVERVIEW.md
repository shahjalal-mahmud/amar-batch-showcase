# Architecture Overview - Amar Batch

## For Recruiters & Technical Evaluators

**Clean Architecture + MVVM | Offline-First | 25,000+ lines of Kotlin | Built by a 3rd year CSE student**

---

## 📋 Executive Summary

Amar Batch follows **Clean Architecture** combined with **MVVM (Model-View-ViewModel)**. This architecture was chosen to achieve three goals:

| Goal                    | How Architecture Achieves It                                                   |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Testability**         | Business logic lives in pure Kotlin classes with no Android dependencies       |
| **Offline Reliability** | Room database is the single source of truth — no race conditions               |
| **Maintainability**     | Clear layer separation — a solo developer can understand any part months later |

The architecture deliberately avoids over-engineering. Every abstraction is justified by a real benefit.

---

## 🏛️ High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                        UI LAYER                          │
│                   Jetpack Compose                        │
│                                                          │
│  AttendanceScreen   FeeScreen   StudentScreen   ...      │
│                                                          │
│  Observes StateFlow │ Sends Events/Actions               │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    VIEWMODEL LAYER                        │
│             State Management & UI Logic                  │
│                                                          │
│  AttendanceViewModel   FeeViewModel   StudentViewModel   │
│                                                          │
│  Holds UiState │ Calls Use Cases or Repository           │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    DOMAIN LAYER                           │
│               Business Rules & Use Cases                 │
│                                                          │
│  CollectFeeUseCase   CalculateDueMonthsUseCase   ...     │
│                                                          │
│  Pure Kotlin │ No Android framework dependencies         │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    DATA LAYER                             │
│             Repository Implementations                   │
│                                                          │
│  StudentRepository   FeeRepository   AttendanceRepo...   │
│                                                          │
│  Abstracts data source │ Only layer that knows about     │
│  Room / Firebase / Google Drive                          │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────┐   ┌────────────────────┐
│      LOCAL DATABASE              │   │  REMOTE SERVICES   │
│    Room (SQLite) — 17 tables     │   │  Firebase Firestore │
│    Primary & only source of      │   │  Google Drive API  │
│    truth for all app data        │   │  (backup only)     │
└──────────────────────────────────┘   └────────────────────┘
```

---

## 📐 The Four Layers Explained

### 1. Data Layer (Bottom)

**Location:** `data/` (entities, DAOs, repository implementations)

**Responsibility:** Knows where data comes from and how to store it.

**Components:**

- **Room Entities** — Database table definitions (17 tables)
- **DAOs** — Database queries (Room)
- **Repository Implementations** — Concrete implementations of domain interfaces
- **Remote Services** — Firebase Auth, Firestore, Google Drive API

**Key Rules:**

- This is the ONLY layer that knows about Room, Firebase, or any data source
- Accepts and returns Domain Models (not Entities)
- Wraps failures in `Result<T>` — never throws exceptions
- Maps Entities ↔ Domain Models

---

### 2. Domain Layer

**Location:** `domain/` (models, repository interfaces, use cases)

**Responsibility:** Contains business rules and use cases. **Zero Android framework dependencies** — pure Kotlin.

**Components:**

- **Domain Models** — Pure data classes representing business concepts
- **Repository Interfaces** — Contracts that data layer implements
- **Use Cases** — Single business operations (e.g., `CalculateDueMonthsUseCase`)

**When to create a Use Case vs. calling Repository directly:**

| Scenario                                        | Where Logic Goes                        |
| ----------------------------------------------- | --------------------------------------- |
| Simple CRUD (get, save, delete)                 | Call repository directly from ViewModel |
| Business calculation (due months, attendance %) | Use Case                                |
| Cross-repository operation (fee + SMS)          | Use Case                                |
| Validation with multiple rules                  | Use Case                                |

---

### 3. ViewModel Layer

**Location:** `viewmodel/`

**Responsibility:** Holds UI state, handles user actions, orchestrates calls to domain/data layers.

**Components:**

- **StateFlow\<UiState\>** — Current screen state (Loading, Ready, Error)
- **SharedFlow\<Event\>** — One-time events (navigation, toasts)
- **Public functions** — Called by UI (e.g., `saveAttendance()`, `loadStudents()`)

**Key Rules:**

- All coroutines use `viewModelScope` (auto-cancelled when screen is destroyed)
- Never reference `View`, `Context`, or Composables
- Always handle loading and error states explicitly

---

### 4. UI Layer (Top)

**Location:** `ui/` (screens, components, navigation, theme)

**Responsibility:** Renders UI and sends user actions to ViewModels.

**Components:**

- **Screens** — Compose functions that observe ViewModel state
- **Reusable Composables** — Buttons, cards, dialogs
- **Navigation** — Compose Navigation with type-safe routes
- **Theme** — Material 3 theming

**Key Rules:**

- Observe state with `collectAsStateWithLifecycle()` (lifecycle-aware)
- No business logic in Composables — only UI rendering
- Separate screen composable (with ViewModel) from content composable (previewable)

---

## 🔄 Unidirectional Data Flow

Data flows in ONE direction. The UI never directly mutates state:

```
User Action (tap "Save")
        ↓
ViewModel function called (saveAttendance())
        ↓
Repository called (attendanceRepository.saveSession(...))
        ↓
DAO writes to Room
        ↓
Room emits new data via Flow
        ↓
Repository maps entity → domain model
        ↓
ViewModel updates StateFlow
        ↓
UI recomposes with new state
```

**Why this matters:** At any point, you can answer "where does this state come from?" — it always traces back to Room, the single source of truth.

---

## 📊 State Management Strategy

### Three Types of State

| Type                     | Location  | Purpose                                 |
| ------------------------ | --------- | --------------------------------------- |
| **Flow\<T\>**            | DAOs      | Live database queries (auto-updating)   |
| **StateFlow\<UiState\>** | ViewModel | Current screen state (always has value) |
| **SharedFlow\<Event\>**  | ViewModel | One-time events (navigation, toasts)    |

### UiState Pattern

Each screen defines a sealed class for its state:
Loading → Ready (data loaded) → Error (if something fails)

Example structure:

- `object Loading` — Initial or refreshing state
- `data class Ready(...)` — Data loaded successfully
- `data class Error(message)` — Error with user-friendly message

---

## 🧵 Background Processing

**WorkManager** handles tasks that must survive app restarts:

| Worker            | Schedule                        | Failure Handling    |
| ----------------- | ------------------------------- | ------------------- |
| **SmsWorker**     | Every 30 seconds + trigger      | Retry up to 3 times |
| **BackupWorker**  | Daily (when charging + on WiFi) | Retry next day      |
| **CleanupWorker** | Weekly                          | Log and continue    |

**SMS Queue Pattern:** Messages are stored in Room's `sms_queue` table, processed one by one in background, with retry logic for network failures.

---

## 🔗 Dependency Injection (Koin)

Modules are organized by layer:

| Module               | Provides                     |
| -------------------- | ---------------------------- |
| **DatabaseModule**   | Room database, all 17 DAOs   |
| **RepositoryModule** | Repository implementations   |
| **UseCaseModule**    | Business logic use cases     |
| **ViewModelModule**  | ViewModels with dependencies |

**Why Koin over Dagger/Hilt:**

- Simpler — no annotation processing or code generation
- Debuggable — clear stack traces
- Lightweight — adds ~200KB vs 1MB+

---

## 🧪 Testability by Layer

| Layer          | Test Approach               | Mocking Strategy                 |
| -------------- | --------------------------- | -------------------------------- |
| **Domain**     | Unit tests (JUnit + Kotest) | No mocks needed (pure functions) |
| **ViewModel**  | Unit tests + Turbine        | Mock repositories via MockK      |
| **Repository** | Integration tests           | In-memory Room database          |
| **UI**         | Compose UI tests            | Mock ViewModels                  |

**Test coverage:** 800+ unit tests, 250+ integration tests, 150+ UI tests

---

## 🚫 What's NOT in This Architecture

| Exclusion                 | Reason                                       |
| ------------------------- | -------------------------------------------- |
| **EventBus**              | StateFlow is more predictable and debuggable |
| **Data Binding**          | Compose eliminates the need                  |
| **Cloud-first sync**      | Offline-first is priority                    |
| **Complex caching layer** | Room Flow is the cache                       |
| **Fragment-based UI**     | Compose is fully declarative                 |

---

## 📈 Key Architecture Decisions

### Decision 1: Offline-First, Not Cloud-First

**Why:** Target teachers have unreliable internet. App must work 100% offline.

**Implementation:** Room is the single source of truth. Cloud (Google Drive) is optional backup only.

### Decision 2: No Hard Deletions

**Why:** Historical attendance and fee records must be preserved for reporting.

**Implementation:** Soft delete flag `is_active` on all major entities.

### Decision 3: Repository Pattern for All Data Access

**Why:** ViewModels shouldn't know if data comes from Room, Firebase, or a future API.

**Implementation:** Domain defines interfaces → Data layer implements them.

### Decision 4: Use Cases Only for Complex Logic

**Why:** Too many use cases create unnecessary indirection.

**Implementation:** Simple CRUD calls repository directly. Business calculations get use cases.

---

## 📁 Package Structure (High-Level)

```
Amar_Batch/                              ← Repo root
│
├── .github/                             ← GitHub special folder
│   │
│   ├── ISSUE_TEMPLATE/                  ← all issue templates
│   ├── PULL_REQUEST_TEMPLATE/
│   ├── workflows/
│   ├── DISCUSSION_TEMPLATE.md           ← pin this post in Discussions tab
│   └── FUNDING.yml                      ← shows a "Sponsor" button (optional)
│
├── app/                                      # Android application module
│   ├── src/
│   │   ├── main/
│   │   │   ├── AndroidManifest.xml            # App manifest
│   │   │   ├── java/com/appriyo/amarbatch/    # Main application source code
│   │   │   │   ├── data/                      # Data layer (data sources & implementations)
│   │   │   │   ├── di/                        # Dependency Injection (Koin modules)
│   │   │   │   ├── domain/                    # Domain layer (business rules)
│   │   │   │   ├── ui/                        # UI layer (Jetpack Compose)
│   │   │   │   ├── viewmodel/                 # ViewModels (state management)
│   │   │   │   ├── utils/                     # Utility classes & helper functions
│   │   │   │   ├── worker/                    # WorkManager background tasks
│   │   │   │   ├── AmarBatchApp.kt            # Application class
│   │   │   │   └── MainActivity.kt            # App entry activity
│
├── docs/                                      # Project documentation
│   ├── auth/
│   ├── crashlytics/
│   ├── database/                              # Database design documentation
│   ├── legal/                                 # Legal documentation
│   ├── prompt/                                # AI-assisted development conversations
│   ├── repository/                            # Repository architecture documentation
│   ├── test/                                  # Testing documentation
│   ├── ui/                                    # UI architecture planning
│   ├── viewmodel/                             # ViewModel documentation
│   │
│   ├── PRODUCT_OVERVIEW.md                    # Product concept and philosophy
│   ├── SETUP.md                               # Development setup guide
│   ├── TECH_STACK.md                          # Technology stack explanation
│   └── USER_GUIDE.md                          # User documentation
│
├── .editorconfig
├── .gitignore
├── ARCHITECTURE.md
├── build.gradle.kts                           # Root Gradle configuration
├── CHANGELOG.md                               # Version history
├── CODE_OF_CONDUCT.md
├── CODEOWNERS.md
├── DEVELOPMENT.md
├── LICENSE.txt                                # Project license
├── README.md                                  # Project overview
├── SECURITY.md                                # Security policy
├── TESTING.md
```

---

## 🎯 What This Architecture Demonstrates

For recruiters evaluating this project:

| Skill                                   | Evidence in Architecture                         |
| --------------------------------------- | ------------------------------------------------ |
| **Understanding of Clean Architecture** | Clear layer separation with dependency inversion |
| **MVVM mastery**                        | StateFlow + SharedFlow pattern, lifecycle-aware  |
| **Offline-first design**                | Room as single source of truth                   |
| **Database design**                     | 17 normalized tables with proper relationships   |
| **Background processing**               | WorkManager + retry logic + queue pattern        |
| **Testability**                         | Each layer independently testable                |
| **Pragmatic decision-making**           | Use cases only when justified, not dogma         |

---

## 📚 Related Documentation

- [Tech Stack](./TECH_STACK.md) — Detailed technology choices
- [Database Schema](../database/SCHEMA_OVERVIEW.md) — 17 tables explained
- [SMS Queue System](../technical/SMS_QUEUE_SYSTEM.md) — Priority-based queue design
- [Offline-First Strategy](../technical/OFFLINE_SYNC_STRATEGY.md) — Data synchronization approach

---

## 📧 Contact for Architecture Discussion

**Developer:** MD SHAHAJALAL MAHMUD (3rd Year CSE Student)  
**Email:** mahmud.nubtk@gmail.com  
**Availability:** Open for Android Developer roles (Intern/Junior)

---

_Last Updated: May 2026_  
_Architecture: Clean Architecture + MVVM_  
_Status: Beta testing with paying users_
