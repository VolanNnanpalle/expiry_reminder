# FreshCheck — Development Log

> Reverse chronological order. Newest entries at the top.
> Every session must add an entry here. See CLAUDE.md for the required format.

---

## [2026-02-22] Session Summary — Staff Engineer Plan Review & Gap Analysis

### What Was Done
- Conducted thorough staff-engineer-level review of the entire project plan
- Identified 19 gaps across 5 categories: launch blockers, architecture, UX, operations, and business
- Updated `PROJECT_PLAN.md` with all fixes (Phase 0 added, tasks restructured, testing gates updated)
- Updated `CLAUDE.md` with new architecture decisions, schema changes, and gotchas
- Added success metrics table to PROJECT_PLAN.md

### Critical Gaps Found & Fixed

#### LAUNCH BLOCKERS FIXED:
1. **Google Play 20-tester/14-day requirement** — Was going to start testing in Week 8, which would add 2+ weeks delay. Moved to Week 5.
2. **Apple Developer account timing** — Was in Week 7, moved to Phase 0 (before any code). Verification takes 24-48 hours.
3. **App name validation** — Was never planned. Added to Phase 0. Could have built entire app and found name taken.

#### ARCHITECTURE GAPS FIXED:
4. **Database migration strategy** — Added schemaVersion: 1 from Day 1 with migration callbacks
5. **Error handling pattern** — Added sealed Result class requirement for repository layer
6. **Image storage** — Added `localImagePath` column separate from `imageUrl`, with compression and cleanup rules
7. **Barcode cache** — Added `BarcodeCache` table with 30-day TTL instead of vague "cache lookups" bullet
8. **Notification deep linking** — Added specific implementation steps for passing item ID through notification payload
9. **Expired item lifecycle** — Added collapsible "Expired" section with "Clear all" option

#### UX GAPS FIXED:
10. **No batch/quick add flow** — Added "Save & Add Another" as PRIMARY save action
11. **Fresh produce unaddressed** — Added "Common Items" quick-select with preset shelf lives
12. **Permission timing backwards** — Changed from onboarding to just-in-time (after first item save)
13. **Home screen not optimized for glancing** — Added summary header ("X items expiring this week")
14. **Swipe-to-delete pattern** — Changed from confirmation dialog to SnackBar with Undo button

#### OPERATIONAL GAPS FIXED:
15. **No crash reporting until Week 9** — Moved Firebase Crashlytics to Phase 1 Day 1
16. **No analytics until Week 9** — Moved Firebase Analytics to Phase 1 Day 1
17. **Git not prioritized** — Moved to Phase 0 as non-negotiable first action
18. **App binary size unchecked** — Added size check after Phase 1 build
19. **build_runner workflow** — Changed from manual `build` to `watch` for development

### Decisions Made

#### Decision: Firebase Crashlytics + Analytics from Day 1 (not post-launch)
- **Problem:** Original plan had crash reporting and analytics as post-launch (Week 9). During beta testing, crash reports would have no stack traces, and post-launch optimization would be guesswork.
- **Options Considered:**
  1. **Keep analytics post-launch** — Simpler initial setup. Cons: flying blind during beta AND first critical weeks of launch.
  2. **Add Firebase from Day 1** — Extra 30-60 min setup in Week 1. Cons: slightly more complex initial setup, Firebase dependency.
  3. **Use Sentry instead of Firebase** — Better error grouping. Cons: paid for higher volume, doesn't include analytics.
- **Chosen:** Firebase from Day 1
- **Reasoning:** Firebase Crashlytics is free and gives crash reports with stack traces from TestFlight/beta testing. Firebase Analytics is free and tracks the exact metrics needed to optimize conversion (items added per session, scan success rate, paywall views). The 30-60 minute setup cost is trivial compared to weeks of guessing post-launch. Sentry is better for large teams but overkill for solo dev.
- **Tradeoffs Accepted:** Firebase adds binary size and Google dependency. Acceptable for the value of crash visibility and analytics.

#### Decision: Just-in-Time Notification Permissions
- **Problem:** Original plan asked for notification permission during onboarding screen 3, before user had any items to be notified about.
- **Options Considered:**
  1. **Onboarding permission request** — Standard pattern. Cons: user has no context for why they need notifications yet. Lower acceptance rate.
  2. **Just-in-time after first item** — Ask when user saves first item: "Want to be reminded when [milk] expires?" Cons: slightly more complex to implement (need to track first-save state).
  3. **Settings-only** — Only ask in settings. Cons: most users never visit settings, very low notification adoption.
- **Chosen:** Just-in-time after first item
- **Reasoning:** Research consistently shows contextual permission requests convert 2-3x better than upfront asks. The user just added milk with a date — asking "want a reminder?" is helpful, not invasive. This also shortened onboarding from 3 screens to 2, reducing drop-off.
- **Tradeoffs Accepted:** Slightly more implementation complexity. Need shared_preferences flag to avoid re-asking.

#### Decision: Add Phase 0 (Pre-Flight) Before Any Code
- **Problem:** Multiple tasks that must happen before code but weren't scheduled: name validation, Apple account, Flutter install, git setup.
- **Options Considered:**
  1. **Keep them as Week 1 tasks** — Mix setup with coding. Cons: if Apple account verification fails or name is taken, you waste an entire weekend.
  2. **Dedicated Phase 0 (one weekday evening)** — All pre-flight checks done before Weekend 1. Cons: "one more thing before starting" feels slow.
- **Chosen:** Phase 0
- **Reasoning:** Every item in Phase 0 is either async (Apple account verification takes 24-48 hours, runs while you work) or a potential blocker (name taken = rename everything). Doing these on a Tuesday evening means Weekend 1 is pure coding with zero blockers. The cost of NOT doing Phase 0 is potentially losing an entire weekend to account verification or a name collision.
- **Tradeoffs Accepted:** Feels like an extra step before "real" work starts. But it's 1-2 hours that can save entire weekends.

### Issues Encountered
- None (review phase only)

### Tests Written/Run
- N/A (no code written yet)

### Next Steps
- **Immediate (one weekday evening):** Complete Phase 0 checklist — validate app name, register Apple Developer account, register Google Play Console, install Flutter, create GitHub repo
- **Weekend 1 Saturday:** Phase 1, Week 1 — project scaffolding with Firebase setup
- **Weekend 1 Sunday:** Phase 1, Week 1 — database + models with new schema (including BarcodeCache, localImagePath, migration strategy)

---

## [2026-02-22] Session Summary — Project Planning & Setup

### What Was Done
- Created `CLAUDE.md` — project context file for Claude sessions
- Created `PROJECT_PLAN.md` — 8-week detailed plan with task checkboxes
- Created `DEVLOG.md` — this development log
- Evaluated 14 app ideas and selected Expiry Reminder (FreshCheck) as the highest-priority project
- Defined initial tech stack, database schema, project structure, and monetization strategy

### Decisions Made

#### Decision: Which App Idea to Build First
- **Problem:** Owner has 14+ app ideas and limited time. Need to pick the one most likely to succeed in the App Store and generate revenue.
- **Options Considered:**
  1. **Friend Calendar/Planner** — Good problem but requires network effect (useless until friends also download). High risk of zero traction.
  2. **Expiry Reminder (FreshCheck)** — Clear daily-use problem, no network effect needed, simple MVP, mature tech (OCR/barcode), food waste is trending.
  3. **Anime Voice/Accent Converter** — Viral potential but copyright risk, novelty may not retain users, harder to monetize.
  4. **Video Context (Kontex)** — Cool concept but technically very hard (reverse video search at scale) and copyright minefield.
- **Chosen:** Expiry Reminder (FreshCheck)
- **Reasoning:** Works as a standalone app (no network effect), has a clear daily habit loop (people open their fridge every day), simple MVP scope achievable in 8 weeks with limited time, clear freemium monetization path, food waste reduction is a growing movement providing marketing angles.
- **Tradeoffs Accepted:** Less viral potential than some ideas. Existing competitors exist (NoWaste, Fridgely). Must differentiate on scanning UX quality.

#### Decision: Tech Stack — Flutter + Riverpod + Drift
- **Problem:** Need a cross-platform framework and supporting libraries to ship to both iOS and Android from one codebase.
- **Options Considered:**
  1. **React Native** — Large ecosystem, JS/TS familiarity. Cons: bridge performance for camera ops, Expo limitations with native modules.
  2. **Flutter + Bloc** — Flutter is mature for mobile, Bloc is battle-tested. Cons: Bloc has significant boilerplate (events, states, blocs per feature).
  3. **Flutter + Riverpod + Drift** — Flutter for cross-platform, Riverpod for lightweight state management, Drift for type-safe SQL. Cons: Riverpod has a learning curve for newcomers.
- **Chosen:** Flutter + Riverpod + Drift
- **Reasoning:** Owner wants Flutter (stated preference). Riverpod over Bloc because solo developer needs speed — Riverpod has less boilerplate, is more intuitive for simple CRUD apps, and has excellent code generation support. Drift over Hive because we need proper SQL queries (filter by category, sort by expiry date, search) and Drift provides type-safe reactive streams + migration support for schema changes.
- **Tradeoffs Accepted:** Riverpod learning curve for a Flutter beginner. Drift requires code generation step (adds build complexity). Could have gone simpler with Provider + sqflite but would hit walls when adding features later.

#### Decision: Local-First Architecture (No Backend for MVP)
- **Problem:** Should we build a backend/cloud sync for v1.0?
- **Options Considered:**
  1. **Firebase backend from day 1** — Cloud sync, auth, analytics built in. Cons: adds 2+ weeks of work, requires account system, ongoing cost.
  2. **Local-only for MVP, add cloud later** — Ship faster, zero backend cost, works offline by default. Cons: no cross-device sync, no backup.
  3. **Supabase backend** — Open source Firebase alternative. Cons: same time cost as Firebase for MVP.
- **Chosen:** Local-only for MVP
- **Reasoning:** The core value proposition (scan → track → get reminded) requires zero network dependency except barcode lookups. Adding auth + cloud sync would delay launch by 2-3 weeks minimum. Users don't expect sync in a v1.0 utility app. Cloud sync becomes a premium feature post-launch.
- **Tradeoffs Accepted:** No data backup in v1.0 (users lose data if they delete the app). No cross-device sync. Will need to add this as premium feature — the migration path from local-only to cloud-synced needs to be considered in schema design.

### Issues Encountered
- None (planning phase only)

### Tests Written/Run
- N/A (no code written yet)

### Next Steps
- Start Phase 1, Week 1: Create Flutter project, set up folder structure, configure dependencies
- Before writing any code, verify Flutter is installed and `flutter doctor` passes
- First coding task: project scaffolding + theme + routing setup (Saturday)
- Second coding task: Drift database tables + repository + unit tests (Sunday)
