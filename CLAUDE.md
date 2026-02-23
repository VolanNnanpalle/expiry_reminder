# Expiry Reminder App — PantryPing

## Project Overview
A Flutter mobile app (iOS + Android) that helps users track food expiration dates to reduce waste and save money. Users can scan barcodes, use OCR to read expiry dates, or manually enter items. The app sends push notifications before items expire.

**Goal:** Ship to App Store + Google Play, then monetize with freemium model.
**Owner:** Solo developer (limited time — 9-5 job, master's course, gym 4x/week). Primary dev time is weekends.

## Current Status
See `PROJECT_PLAN.md` for detailed progress tracking with checkboxes. Check that file FIRST to understand what's been completed and what's next.

---

## CRITICAL: Agent Workflow Rules

Every Claude session working on this project MUST follow these rules. No exceptions.

### 0. Session Resumption Protocol
When the user says ANYTHING that implies continuing work on this project — including but not limited to: "continue," "keep working," "pick up where I left off," "what's next," "let's work on this," "resume," "keep going," or just opens a session in this directory — follow this exact protocol:

1. Read this file (`CLAUDE.md`) — you're already doing this
2. Read `PROJECT_PLAN.md` — scan the **Quick Status Dashboard** for current phase, then find the **first unchecked `[ ]` task** in that phase
3. Read **ONLY the latest entry** (the first `##` section) in `DEVLOG.md` — this has what was done last session, any in-progress work, and recommended next steps
4. **Tell the user** what you found:
   - Current phase and progress (e.g., "We're in Phase 1, Week 1. Saturday scaffolding is done.")
   - What was done last session (from DEVLOG)
   - Any tasks marked `[~]` (in-progress / partially done from prior session)
   - What you'll work on now (the next unchecked task)
5. **Wait for user confirmation** before starting, in case they want to redirect
6. Begin work on the next task

**Task status convention in PROJECT_PLAN.md:**
- `[ ]` = Not started
- `[~]` = In progress / partially done (prior session started but didn't finish)
- `[x]` = Complete and tested

When ending a session or when the user says they're done, you MUST:
- Mark any unfinished tasks as `[~]` in PROJECT_PLAN.md
- Update the Quick Status Dashboard (Current Phase, Current Week, Blockers)
- Write a session summary to DEVLOG.md (see format below)

### 1. Read Context Before Doing Anything
Before writing ANY code, read these files in this order:
1. This file (`CLAUDE.md`) — understand the project
2. `PROJECT_PLAN.md` — find what's been completed and what's next
3. `DEVLOG.md` — read **ONLY the latest entry** (top of file) to understand recent work, decisions, and any open issues. Do NOT read the entire file — it grows over time and older entries are not relevant to your current session.

### 2. Decision Log Protocol
**Every implementation decision must be scrutinized before committing to it.** This applies to:
- Package/library choices
- Architecture patterns
- UI/UX approaches
- Data model changes
- API integration approaches
- Any non-trivial "how should we do this?" moment

**Before implementing a decision, you MUST:**
1. **State the problem** — What are we trying to solve?
2. **List alternatives** — At least 2-3 options. No decision is made in a vacuum.
3. **Evaluate each option critically** — Pros, cons, tradeoffs. Be harsh. Consider: performance, maintainability, complexity, time to implement, future flexibility.
4. **Justify the choice** — Why this option over the others? What convinced you?
5. **Log it** — Write the decision to `DEVLOG.md` in the format specified below.

Do NOT blindly follow the initial tech stack recommendations in this file. If during implementation you discover a better approach, scrutinize it, and if it wins, update both `CLAUDE.md` and `DEVLOG.md` with the reasoning.

### 3. Testing Before Moving On
**No task is complete until it is tested.** Before marking any task as done:
- Write and run relevant unit tests for logic/data code
- Manually verify UI changes work on at least one platform (iOS simulator or Android emulator)
- Run `flutter analyze` to catch lint issues
- Run `flutter test` to verify no existing tests broke
- If the task involves a new screen/flow, verify the happy path AND at least one error/edge case

**If tests fail, the task is NOT done.** Fix the tests before moving on.

### 4. Update Logs After Every Task
After completing each task (or group of small related tasks), update `DEVLOG.md` with:
- What was done
- Any decisions made (using the decision format below)
- Any issues encountered and how they were resolved
- What should be done next
- Update the task checkboxes in `PROJECT_PLAN.md`

### 5. Update Status on Session End
At the end of every session (or if the user says they're done for the day):

**Step 1: Update PROJECT_PLAN.md:**
- Mark completed tasks as `[x]`
- Mark partially-done tasks as `[~]`
- Update the Quick Status Dashboard at the top:
  - Current Phase
  - Current Week
  - Blockers (if any)

**Step 2: Write session summary to DEVLOG.md** (at the top, newest first) that includes:
- What was accomplished this session
- What's in progress / partially done (reference specific `[~]` tasks)
- Any blockers or open questions
- **Explicit recommended next steps** for the next session — be specific about which task to start with

---

## DEVLOG.md Format

The file `DEVLOG.md` is the living development log. It is written in reverse chronological order (newest entries at the top). Every entry follows this structure:

```markdown
## [YYYY-MM-DD] Session Summary — [Brief Title]

### Current State
- **Phase:** [phase number and name]
- **Dashboard updated:** Yes/No
- **Tasks marked `[~]` (in-progress):** [list any partially done tasks, or "None"]

### What Was Done
- Completed task X (checkbox reference from PROJECT_PLAN.md)
- Completed task Y

### Decisions Made

#### Decision: [Title of Decision]
- **Problem:** What we needed to solve
- **Options Considered:**
  1. **Option A** — [description]. Pros: ... Cons: ...
  2. **Option B** — [description]. Pros: ... Cons: ...
  3. **Option C** — [description]. Pros: ... Cons: ...
- **Chosen:** Option B
- **Reasoning:** [Why this won. Be specific.]
- **Tradeoffs Accepted:** [What we're giving up]

### Issues Encountered
- [Description of issue and resolution, or "None"]

### Tests Written/Run
- [List of tests added or run, and their results]

### Next Steps (for the NEXT session to pick up)
- **Start with:** [exact task name/description to begin with]
- **Then:** [what follows after that]
- **Blockers to resolve first:** [any blockers, or "None"]
```

---

## Tech Stack
- **Framework:** Flutter (Dart)
- **State Management:** Riverpod (flutter_riverpod + riverpod_annotation)
- **Local Database:** Drift (formerly Moor) — SQLite-based, type-safe, reactive
- **Barcode Scanning:** mobile_scanner
- **OCR:** google_mlkit_text_recognition
- **Product Database:** Open Food Facts API (free, no key needed)
- **Notifications:** flutter_local_notifications
- **Navigation:** go_router
- **HTTP:** dio
- **Image Handling:** cached_network_image, image_picker
- **Crash Reporting:** Firebase Crashlytics (from Day 1, NOT post-launch)
- **Analytics:** Firebase Analytics (from Day 1, NOT post-launch)
- **Error Handling:** Sealed Result class pattern for repository layer

## Project Structure
```
lib/
├── main.dart                    # Entry point
├── app.dart                     # MaterialApp + router setup
├── core/
│   ├── constants/app_constants.dart
│   ├── theme/app_theme.dart
│   ├── routing/app_router.dart
│   ├── utils/date_parser.dart   # Critical: multi-format expiry date parsing
│   └── utils/expiry_helpers.dart
├── features/
│   ├── items/
│   │   ├── data/
│   │   │   ├── database/food_item_dao.dart
│   │   │   ├── database/app_database.dart
│   │   │   └── repositories/item_repository.dart
│   │   ├── domain/models/food_item.dart
│   │   └── presentation/
│   │       ├── screens/item_list_screen.dart
│   │       ├── screens/add_item_screen.dart
│   │       ├── screens/item_detail_screen.dart
│   │       ├── widgets/item_card.dart
│   │       ├── widgets/expiry_badge.dart
│   │       └── providers/items_provider.dart
│   ├── scanner/
│   │   ├── data/open_food_facts_api.dart
│   │   └── presentation/
│   │       ├── screens/barcode_scanner_screen.dart
│   │       ├── screens/ocr_scanner_screen.dart
│   │       └── providers/scanner_provider.dart
│   ├── notifications/
│   │   ├── services/notification_service.dart
│   │   └── providers/notification_provider.dart
│   ├── settings/
│   │   ├── presentation/screens/settings_screen.dart
│   │   └── providers/settings_provider.dart
│   └── onboarding/
│       └── presentation/screens/onboarding_screen.dart
└── shared/
    ├── widgets/
    └── providers/
```

## Key Architecture Decisions
1. **Feature-first folder structure** — each feature is self-contained for easy navigation
2. **Riverpod over Bloc** — less boilerplate, better for solo dev speed, excellent testability
3. **Drift over Hive** — SQL gives us proper queries, migrations, and reactive streams
4. **mobile_scanner over google_mlkit_barcode** — better maintained, simpler API, uses MLKit under the hood
5. **Local-first, no backend for MVP** — ship fast, add cloud sync post-launch
6. **Firebase Crashlytics + Analytics from Day 1** — not post-launch. Need crash reports during beta and analytics to optimize conversion.
7. **Just-in-time notification permissions** — ask after first item save, not during onboarding. 2-3x better conversion rate.
8. **Consistent error handling** — use sealed Result class or equivalent for all repository methods. No raw exceptions leaking to UI.
9. **Barcode cache table** — Open Food Facts is slow (1-3s) and unreliable. Cache lookups locally with 30-day TTL.
10. **Schema versioning from Day 1** — schemaVersion: 1 with migration callbacks. Never modify existing columns, only add.

## Database Schema (Drift)

**CRITICAL:** Schema version must be set to 1 from Day 1. Migration strategy must exist even if empty. Never modify existing columns in updates — only add new ones.

```dart
// schemaVersion: 1

// food_items table
class FoodItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().withLength(min: 1, max: 200)();
  TextColumn get barcode => text().nullable()();
  DateTimeColumn get expiryDate => dateTime()();
  DateTimeColumn get addedDate => dateTime().withDefault(currentDateAndTime)();
  TextColumn get category => text().withDefault(const Constant('fridge'))(); // fridge, freezer, pantry
  // NOTE: org is com.pantryping, project name is pantryping
  IntColumn get quantity => integer().withDefault(const Constant(1))();
  TextColumn get imageUrl => text().nullable()();        // Remote image (Open Food Facts)
  TextColumn get localImagePath => text().nullable()();  // User-taken photo (stored in app docs dir)
  TextColumn get notes => text().nullable()();
  TextColumn get brand => text().nullable()();
  BoolColumn get isConsumed => boolean().withDefault(const Constant(false))();
}

// notification_settings table
class NotificationSettings extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get daysBeforeExpiry => integer()(); // 0, 1, 3, 7
  IntColumn get hourOfDay => integer().withDefault(const Constant(9))();
  IntColumn get minuteOfHour => integer().withDefault(const Constant(0))();
  BoolColumn get isEnabled => boolean().withDefault(const Constant(true))();
}

// barcode_cache table — avoids repeated slow API calls to Open Food Facts
class BarcodeCache extends Table {
  TextColumn get barcode => text()();
  TextColumn get productName => text()();
  TextColumn get brand => text().nullable()();
  TextColumn get imageUrl => text().nullable()();
  DateTimeColumn get cachedAt => dateTime()();

  @override
  Set<Column> get primaryKey => {barcode};
}
```

## Commands
- `flutter run` — run app in debug mode
- `flutter build ios` — build for iOS
- `flutter build appbundle` — build for Play Store
- `dart run build_runner build` — generate Drift/Riverpod code
- `flutter test` — run tests
- `flutter analyze` — lint check

## Key Gotchas
- **Date parsing is the hardest part.** Expiry dates come in 20+ formats globally. ALSO: OCR picks up manufacturing dates, batch codes that look like dates, and "packed on" dates. Use keyword proximity scoring (EXP, BB, USE BY) to pick the right date. NEVER auto-select — always confirm with user.
- **Camera permissions:** iOS requires `NSCameraUsageDescription` in Info.plist. Android requires `CAMERA` permission in AndroidManifest.
- **Notification permissions:** iOS 16+ and Android 13+ require explicit permission. Request JUST-IN-TIME (after first item save), not during onboarding.
- **Open Food Facts API** may not have all products. Always fall back to manual entry. Check barcode cache FIRST before hitting API.
- **Code generation:** Use `dart run build_runner watch` during development (not `build` each time).
- **Google Play requires 20 testers for 14 days** before you can publish publicly from a new developer account. Start closed testing track in Week 5.
- **App binary size:** google_mlkit_text_recognition bundles ML models on-device. Can push app to 80-100MB+. Check size after Phase 1 setup.
- **Fresh produce (bananas, avocados, etc.)** has no barcode or printed expiry date. The "Common Items" quick-select with preset shelf lives handles this case.
- **Local image storage:** User-taken photos go in app documents directory. Compress to max 500KB. Clean up orphaned images when items are deleted.
- **Database migrations:** NEVER modify existing columns. Only add new ones. Always increment schemaVersion and add migration logic.
