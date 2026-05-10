# Licensing System

> Access control for a solo-developer commercial product — simple, reliable, and offline-tolerant.

---

## Overview

The licensing system controls whether a teacher can use the app. It supports a free trial period, a paid subscription, and a blocked state. The system works offline by caching license state locally, and syncs with a remote server when connectivity is available.

---

## License States

A license is always in one of four states:

**Trial** — the default state after first setup. The teacher has full access for the duration of the trial period (several months).

**Paid** — activated manually by the developer after the teacher pays via bKash or Nagad. Full access for the subscription period (one year).

**Grace Period** — a buffer window after the expiry date. The teacher still has full access during grace, but sees a warning to renew. This exists to handle edge cases: the developer being briefly unavailable, the teacher being on a trip, or a payment processing delay.

**Blocked** — access revoked. The app shows a lock screen with the developer's contact information and the device ID. The teacher's data is never deleted — if the license is reinstated, everything is exactly as they left it.

---

## How Validation Works

License validation runs on every app launch and periodically while the app is in use.

**When online:** The app contacts the license server and fetches the current license status. The local license record is updated with whatever the server returns. This is how paid activations and trial extensions propagate to the device — the developer updates the record on the server, and the app picks it up on the next online check.

**When offline:** The app reads the locally cached expiry date and computes the state from that. A teacher with a valid license can always open and use the app, even with no internet, because the relevant state is stored on the device.

The grace period provides a window for the offline fallback to remain valid even if the teacher is offline across a renewal deadline. It also prevents the situation where a teacher cannot access the app simply because the server is temporarily unreachable.

---

## Device Binding

Each license record stores the device ID of the phone it was activated on. This ties a license to a specific device and prevents one license from being shared across multiple phones.

When a teacher reinstalls on a new phone, the developer manually transfers the license to the new device ID. This is a deliberate manual step rather than an automatic one — it keeps the activation flow simple and gives the developer visibility into device changes.

---

## Identity Layer

Teacher identity is stored in two places. The license and teacher profile are kept in Firestore (the remote source of truth for identity). The local Room database caches this information for offline access.

When a teacher signs in on a new device, the app authenticates with Firebase, waits briefly for the authentication session to fully stabilize, then fetches the teacher document from Firestore. If a document is found, the teacher is on a reinstall — their profile and license are restored to the local database, and then their academic data backup is downloaded from Drive.

**Why the stabilization delay:** immediately after a sign-in, the Firestore SDK's internal token listener has not yet processed the new credentials. Calling Firestore at that exact moment causes a type-cast exception inside the SDK. A short delay after successful authentication prevents this reliably.

---

## Concurrent Sign-In Protection

The sign-in flow is guarded against being called twice concurrently. If two sign-in attempts happen simultaneously — which can occur if a screen is tapped multiple times during a slow network response — only the first is allowed to proceed. Subsequent concurrent calls return immediately without executing.

---

## Activation Flow (Manual)

The developer activates paid licenses manually:

1. Teacher pays via bKash or Nagad and sends confirmation
2. Developer updates the license record on the server (status, expiry date)
3. On the teacher's next app launch with internet, the license sync pulls the update
4. The app unlocks immediately — no reinstall, no action required from the teacher

This manual flow is intentional. The product has a small enough user base that automated payment integration would add complexity with little practical benefit. The teacher pays once a year; the manual confirmation takes under a minute.

---

## What Happens When a License Expires

When the computed state transitions to Expired, the local license record is updated to Blocked status. The next app launch shows the lock screen.

The lock screen displays:

- A message that access has expired
- The developer's contact information
- The device's unique ID (so the developer can look up the license)

The teacher's data is completely intact. Reinstating the license on the server and opening the app restores full access with no data loss.

---

## Related Documentation

- [Offline-First Strategy](./OFFLINE_SYNC_STRATEGY.md) — How offline license validation fits into the broader offline strategy
- [Background Workers](./BACKGROUND_WORKERS.md) — Periodic license sync via the backup worker
