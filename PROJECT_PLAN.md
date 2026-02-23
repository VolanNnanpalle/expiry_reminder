# FreshCheck — Complete Project Plan

> **Goal:** Ship a food expiry reminder app to both App Stores and monetize with freemium.
> **Estimated Timeline:** 8 weeks to MVP launch (10-15 hrs/week, primarily weekends)
> **Last Updated:** 2026-02-22

---

## Workflow Rules (Read CLAUDE.md for full details)

1. **Read context first** — `CLAUDE.md` → `PROJECT_PLAN.md` (this file) → `DEVLOG.md` (latest entry only)
2. **Scrutinize every decision** — Document problem, alternatives, pros/cons, and reasoning in `DEVLOG.md`
3. **Test before marking done** — Run `flutter analyze` + `flutter test` + manual verification. If tests fail, task is NOT done.
4. **Log after every task** — Update `DEVLOG.md` with what was done, decisions, issues, and next steps
5. **Update checkboxes** — Mark completed tasks in this file after each task
6. **Testing gates block progress** — Each phase has a testing gate. ALL items must pass before starting the next phase.

**Task status legend:**
- `[ ]` = Not started
- `[~]` = In progress / partially done (started in a prior session but not finished)
- `[x]` = Complete and tested

---

## Quick Status Dashboard

| Phase | Status | Target |
|-------|--------|--------|
| Phase 1: Foundation | `NOT STARTED` | Weeks 1-2 |
| Phase 2: Scanning | `NOT STARTED` | Weeks 3-4 |
| Phase 3: Notifications & Polish | `NOT STARTED` | Weeks 5-6 |
| Phase 4: App Store Launch | `NOT STARTED` | Weeks 7-8 |
| Phase 5: Post-Launch & Premium | `NOT STARTED` | Weeks 9+ |

**Current Phase:** Phase 0 (Pre-flight)
**Current Week:** 0 (Not started)
**Blockers:** None

---

## Phase 0: Pre-Flight Checklist (Before Writing Any Code)

> **Time needed:** 1-2 hours, one weekday evening. DO NOT skip this.

- [ ] **Validate app name:** Search "FreshCheck" on App Store + Google Play. Search USPTO trademark database. If taken, brainstorm alternatives NOW (e.g., ExpiryPal, FreshAlert, ShelfLife, PantryPing). Pick a name that's available on both stores.
- [ ] **Register Apple Developer Account** ($99/year) — developer.apple.com. Verification can take 24-48 hours. Do this FIRST so it's ready by Phase 4.
- [ ] **Register Google Play Console** ($25 one-time) — play.google.com/console. Activate immediately.
- [ ] **Install Flutter SDK** and run `flutter doctor` — resolve all issues before Weekend 1.
- [ ] **Create GitHub repository** — `git init`, create `.gitignore` for Flutter, first commit, push. This is non-negotiable. Every meaningful change gets committed.
- [ ] **Verify you have Xcode + iOS simulator AND Android Studio + emulator** working. Resolving simulator issues eats entire weekends if not done upfront.

**Phase 0 Gate:** All items above must be checked before starting Phase 1. Apple Developer account can be pending (it processes in background) but must be submitted.

---

## Schedule Assumptions

The developer has limited availability:
- **Monday-Friday:** Full-time job + master's course + gym 4x/week
- **Realistic weekday availability:** 2-3 evenings × 1-1.5 hours = ~3 hours
- **Saturday:** 4-6 hours of focused dev time
- **Sunday:** 4-6 hours of focused dev time
- **Total weekly budget:** ~10-15 hours

**Rule:** Each weekend should have ONE major deliverable. Weekday evenings are for small tasks, bug fixes, and polish only.

---

## Phase 1: Foundation (Weeks 1-2)

### Week 1 — Project Setup + Data Layer

**Saturday (4-6 hrs): Project Scaffolding**
- [ ] Create Flutter project: `flutter create --org com.freshcheck freshcheck`
- [ ] `git init` + initial commit + push to GitHub (if not done in Phase 0)
- [ ] Set up folder structure per CLAUDE.md architecture
- [ ] Run `flutter pub outdated` and resolve to latest stable versions BEFORE adding to pubspec
- [ ] Configure `pubspec.yaml` with all dependencies:
  ```yaml
  dependencies:
    flutter_riverpod: ^2.5.1
    riverpod_annotation: ^2.3.5
    drift: ^2.16.0
    sqlite3_flutter_libs: ^0.5.20
    path_provider: ^2.1.2
    path: ^1.9.0
    mobile_scanner: ^5.1.1
    google_mlkit_text_recognition: ^0.13.0
    flutter_local_notifications: ^17.2.1
    go_router: ^14.2.0
    dio: ^5.4.1
    cached_network_image: ^3.3.1
    image_picker: ^1.0.7
    intl: ^0.19.0
    shared_preferences: ^2.2.2
    permission_handler: ^11.3.0
    url_launcher: ^6.2.4
    firebase_core: # run `flutter pub add firebase_core` to get latest
    firebase_crashlytics: # run `flutter pub add firebase_crashlytics` to get latest
    firebase_analytics: # run `flutter pub add firebase_analytics` to get latest

  dev_dependencies:
    drift_dev: ^2.16.0
    build_runner: ^2.4.8
    riverpod_generator: ^2.4.0
    flutter_test:
      sdk: flutter
    mocktail: ^1.0.3
  ```
- [ ] Set up Firebase project (Crashlytics + Analytics) — follow FlutterFire CLI setup
- [ ] Initialize Crashlytics in main.dart (wrap runApp in runZonedGuarded, FlutterError.onError)
- [ ] Set up app theme (colors, typography, dark mode support)
- [ ] Set up go_router with placeholder screens
- [ ] Create main.dart and app.dart entry points
- [ ] Verify app runs on both iOS simulator and Android emulator
- [ ] Check app binary size: `flutter build apk --release` → note size. If > 80MB, flag for investigation.
- [ ] Start `dart run build_runner watch` as default dev workflow (not `build` each time)

**Sunday (4-6 hrs): Database + Models**
- [ ] Define Drift database tables (FoodItems, NotificationSettings, BarcodeCache)
- [ ] **Set `schemaVersion: 1`** in database class — critical for future migrations
- [ ] Add migration strategy callback (empty for now, but the structure must exist from Day 1):
  ```dart
  @override
  int get schemaVersion => 1;
  @override
  MigrationStrategy get migration => MigrationStrategy(
    onUpgrade: (migrator, from, to) async {
      // Future migrations go here
    },
  );
  ```
- [ ] Schema must include `localImagePath` column (TEXT, nullable) separate from `imageUrl` — for user-taken photos stored on device
- [ ] Schema must include `BarcodeCache` table: barcode (TEXT PRIMARY KEY), productName (TEXT), brand (TEXT nullable), imageUrl (TEXT nullable), cachedAt (DATETIME)
- [ ] Run code generation: `dart run build_runner build`
- [ ] Create DAOs with CRUD operations
- [ ] Create `ItemRepository` with methods:
  - `addItem(FoodItem)`
  - `updateItem(FoodItem)`
  - `deleteItem(int id)`
  - `getAllItems()` → Stream
  - `getItemsByCategory(String category)` → Stream
  - `getExpiringItems(int withinDays)` → Stream
  - `getExpiredItems()` → Stream (items past expiry, not consumed)
  - `markAsConsumed(int id)`
  - `archiveExpiredItems(int olderThanDays)` → bulk update
- [ ] Create `BarcodeCacheRepository` with methods:
  - `getCachedProduct(String barcode)` → nullable
  - `cacheProduct(String barcode, productData)`
  - `clearOldCache(int olderThanDays)`
- [ ] Define a consistent error handling pattern: use Dart `sealed class` or `Result` type for repository returns
- [ ] Write 5-8 unit tests for repository (CRUD + expiry queries + cache)
- [ ] Create Riverpod providers for database access

**Weekday Evenings (~3 hrs total):**
- [ ] Create `app_constants.dart` (categories list, color codes, default notification times)
- [ ] Create `expiry_helpers.dart`:
  - `daysUntilExpiry(DateTime date)` → int
  - `expiryStatus(DateTime date)` → enum (expired, today, soon, ok, fresh)
  - `expiryColor(ExpiryStatus status)` → Color
- [ ] Create shared widgets: `ExpiryBadge`, `CategoryChip`

### Week 2 — Item CRUD Screens

**Saturday (4-6 hrs): Add Item Screen**
- [ ] Build `AddItemScreen` with form:
  - Product name (required, text field)
  - Expiry date (required, date picker)
  - Category selector (fridge/freezer/pantry — chip selector)
  - Quantity (number stepper, default 1)
  - Notes (optional text field)
  - Brand (optional text field)
- [ ] **"Save & Add Another" button** — clears form, stays on add screen. CRITICAL for grocery haul UX. This is the primary save action. "Save & Done" is secondary.
- [ ] **"Common Items" quick-select** — grid of frequent produce with preset shelf lives: banana (5 days), avocado (4 days), milk (7 days), bread (5 days), eggs (21 days), lettuce (5 days), chicken (2 days), yogurt (14 days). Tapping auto-fills name + default expiry. User can adjust.
- [ ] Form validation
- [ ] Save to database via Riverpod provider
- [ ] Success feedback (haptic + snackbar). If "Save & Add Another": clear form, keep category selection. If "Save & Done": navigate back to list.

**Sunday (4-6 hrs): Item List Screen (Home Screen)**
- [ ] Build `ItemListScreen` as the home screen
- [ ] **Summary header at top:** "X items expiring this week" with count badge — this is what users glance at daily. Make it prominent.
- [ ] Display items grouped/sorted by expiry date (soonest first)
- [ ] Color-coded cards: red (expired/today), orange (1-3 days), yellow (4-7 days), green (7+ days)
- [ ] **Expired items section:** Collapsed "Expired (X)" section at bottom of list. Tap to expand. "Clear all expired" button. Individual swipe to dismiss.
- [ ] Category filter tabs (All, Fridge, Freezer, Pantry)
- [ ] Swipe to delete → **SnackBar with "Undo" button** (NOT confirmation dialog — Material Design pattern, faster UX)
- [ ] Swipe to mark as consumed
- [ ] Empty state with illustration/message
- [ ] FAB to add new item

**Weekday Evenings (~3 hrs total):**
- [ ] Build `ItemDetailScreen` (view/edit single item)
- [ ] Add pull-to-refresh on list
- [ ] Add search/filter functionality
- [ ] Handle edge cases (no items, all expired, etc.)

**Phase 1 Testing Gate — DO NOT PROCEED TO PHASE 2 UNTIL ALL PASS:**
- [ ] `flutter analyze` returns no errors
- [ ] `flutter test` — all unit tests pass (minimum 5 tests for repository CRUD)
- [ ] Manual test: add item → appears in list sorted correctly
- [ ] Manual test: edit item → changes persist after app restart
- [ ] Manual test: delete item → removed from list
- [ ] Manual test: filter by category → correct items shown
- [ ] Manual test: empty state displays correctly when no items
- [ ] App runs without crashes on at least one simulator/emulator
- [ ] All decisions documented in `DEVLOG.md`
- [ ] `PROJECT_PLAN.md` checkboxes updated

**Phase 1 Checkpoint:** App runs, you can manually add/edit/delete food items, see them in a list sorted by expiry, and filter by category. This is a working app already.

---

## Phase 2: Scanning (Weeks 3-4)

### Week 3 — Barcode Scanning + Product Lookup

**Saturday (4-6 hrs): Barcode Scanner**
- [ ] Configure camera permissions:
  - iOS: Add `NSCameraUsageDescription` to Info.plist
  - Android: Add `CAMERA` permission to AndroidManifest.xml
- [ ] Build `BarcodeScannerScreen` using `mobile_scanner`
- [ ] Handle scan result → extract barcode string
- [ ] Add scan button to `AddItemScreen` and home screen
- [ ] Handle permission denied gracefully (show settings link)

**Sunday (4-6 hrs): Open Food Facts Integration**
- [ ] Create `OpenFoodFactsApi` service class
  - Endpoint: `https://world.openfoodfacts.org/api/v2/product/{barcode}.json`
  - No API key needed
  - Parse response: product name, brand, image URL, categories
- [ ] On successful scan: auto-fill product name, brand, image
- [ ] Show confirmation screen: "Is this your product?" with edit option
- [ ] Handle product not found: fall back to manual entry with barcode saved
- [ ] Add loading state and error handling
- [ ] Use `BarcodeCache` table from Phase 1 — check cache before API call, cache results after successful lookup (TTL: 30 days)

**Weekday Evenings (~3 hrs total):**
- [ ] Test with 10+ real products (various barcodes)
- [ ] Handle edge cases: camera not available, dim lighting tip, barcode not readable
- [ ] Add "flashlight" toggle to scanner screen
- [ ] Polish scanner UI (viewfinder overlay, instructions text)

### Week 4 — OCR Expiry Date Scanning

**Saturday (4-6 hrs): OCR Integration**
- [ ] Build `OcrScannerScreen` using `google_mlkit_text_recognition`
- [ ] Capture image from camera → process with MLKit
- [ ] Build `DateParser` utility — THIS IS THE HARDEST PART:
  ```dart
  // Must handle ALL of these formats:
  // "EXP 01/26", "BB 2026-03-15", "Best Before: Mar 2026"
  // "USE BY 03/15/26", "15.03.2026", "2026.03.15"
  // "EXP: JAN 2027", "03/2026", "Best if used by March 15, 2026"
  // "Sell by 03-15-2026", "BB: 15/03/26"
  //
  // HARD EDGE CASES TO HANDLE:
  // - Manufacturing date vs expiry date on same label (OCR picks up BOTH)
  // - Batch codes that look like dates: "L2603" is NOT March 2026
  // - "Packed on" vs "Best by" — need to identify which is which
  // - Products with month/year only (no day) — default to last day of month
  // - Labels with NO expiry date at all — detect and prompt manual entry
  ```
- [ ] Strategy: First filter for keywords (EXP, BB, BEST, USE BY, SELL BY) near date-like patterns. If keyword found, prioritize that date. If no keyword, present ALL candidates to user.
- [ ] **IMPORTANT:** Never auto-select a date silently. Always confirm with user. Wrong expiry date = worse than no app.
- [ ] Show detected text with highlighted date regions

**Sunday (4-6 hrs): OCR Polish + Integration**
- [ ] Connect OCR result to `AddItemScreen` (auto-fill expiry date)
- [ ] Add "Scan Expiry Date" button alongside manual date picker
- [ ] User confirmation step: "We detected [date]. Is this correct?"
- [ ] Handle: no date found, multiple dates found, ambiguous format (01/02/26 — is it Jan 2 or Feb 1?)
  - Default: assume MM/DD/YY for US locale, DD/MM/YY for others
  - Let user toggle with a button
- [ ] Test with 10+ real product labels (take photos)

**Weekday Evenings (~3 hrs total):**
- [ ] Write unit tests for `DateParser` with 20+ format variations
- [ ] Optimize camera preview performance
- [ ] Add option to pick image from gallery (for items already in the pantry)
- [ ] Combined flow: scan barcode → auto-fill name → scan expiry → auto-fill date → confirm → save

**Phase 2 Testing Gate — DO NOT PROCEED TO PHASE 3 UNTIL ALL PASS:**
- [ ] `flutter analyze` returns no errors
- [ ] `flutter test` — all tests pass (add tests for DateParser: minimum 20 format variations)
- [ ] Manual test: scan barcode → product name auto-fills from Open Food Facts
- [ ] Manual test: scan barcode for unknown product → graceful fallback to manual entry
- [ ] Manual test: OCR scan picks up expiry date from a real product label
- [ ] Manual test: date parser handles at least MM/DD/YY, DD/MM/YYYY, "Month YYYY", "EXP MM/YY"
- [ ] Manual test: combined flow (barcode → OCR → confirm → save) works end-to-end
- [ ] Manual test: camera permission denied → user sees helpful message
- [ ] Manual test: no internet → barcode lookup fails gracefully, rest of app works
- [ ] All decisions documented in `DEVLOG.md`
- [ ] `PROJECT_PLAN.md` checkboxes updated

**Phase 2 Checkpoint:** User can scan a barcode to auto-fill product info AND scan expiry dates. The "magic" flow works: point camera at barcode, point camera at date, confirm, done. Manual entry still available as fallback.

---

## Phase 3: Notifications & Polish (Weeks 5-6)

### Week 5 — Push Notifications

**Saturday (4-6 hrs): Notification System**
- [ ] Initialize `flutter_local_notifications` in main.dart
- [ ] Configure platform-specific setup:
  - Android: Create notification channel
  - Do NOT request permission upfront (see just-in-time strategy below)
- [ ] **Just-in-time permission strategy:** Request notification permission when user saves their FIRST item. Show contextual prompt: "Want to be reminded when [item name] expires?" → then request OS permission. This converts 2-3x better than asking during onboarding. Store permission-requested state in shared_preferences so we only ask once.
- [ ] Build `NotificationService`:
  - `scheduleExpiryNotification(FoodItem item, int daysBefore)`
  - `cancelNotification(int itemId)`
  - `cancelAllNotifications()`
  - `rescheduleAllNotifications()` (for when items are edited)
- [ ] Default notification schedule:
  - 3 days before expiry (morning, 9 AM)
  - 1 day before expiry (morning, 9 AM)
  - Day of expiry (morning, 9 AM)

**Sunday (4-6 hrs): Notification Settings + Scheduling Logic**
- [ ] Build notification preferences in Settings screen:
  - Toggle notifications on/off
  - Customize days before (checkboxes: 7, 3, 1, 0 days)
  - Set notification time (time picker)
- [ ] Schedule notifications automatically when item is added/edited
- [ ] Cancel notifications when item is deleted/consumed
- [ ] Handle app restart: reschedule all pending notifications on app launch
- [ ] Test notifications on both platforms (use short delay for testing)

**Weekday Evenings (~3 hrs total):**
- [ ] **Notification deep linking:** Pass item ID in notification payload → configure go_router to handle `/item/:id` deep link → on notification tap, parse payload and navigate. Test with app in foreground, background, AND killed state (all three behave differently).
- [ ] Badge count on app icon (iOS)
- [ ] Handle notification permission denied gracefully (show in-app banner with settings link, not a blocking dialog)
- [ ] Group notifications (if multiple items expiring same day, batch into one)
- [ ] **Start Google Play closed testing track** — upload current build, recruit 20 testers (friends, family, classmates, coworkers). Google requires 20+ testers for 14 continuous days before production access is granted. THIS IS A HARD BLOCKER. Start NOW, not in Week 8.

### Week 6 — UI Polish & Onboarding

**Saturday (4-6 hrs): Visual Polish**
- [ ] Finalize color scheme and typography
- [ ] Add subtle animations:
  - List item appear/disappear
  - Category tab switch
  - FAB expand for scan options
- [ ] Design and implement proper empty states:
  - No items yet: "Scan your first item to get started!"
  - All items fresh: "Everything's fresh! Great job."
  - Category empty: "Nothing in the freezer"
- [ ] Dark mode support
- [ ] App icon design (use Figma or a generator — green/fresh themed)
- [ ] Splash screen

**Sunday (4-6 hrs): Onboarding + Settings**
- [ ] Build 2-screen onboarding flow (keep it SHORT — users skip long onboarding):
  1. "Never waste food again" — value prop + food waste stat
  2. "Scan or type" — show how to add items with quick visual
  - **NO notification permission request here** — that happens just-in-time when they add their first item (see Week 5)
- [ ] Complete Settings screen:
  - Notification preferences (from Week 5)
  - Default category
  - Date format preference (MM/DD vs DD/MM)
  - Dark mode toggle
  - About / Privacy Policy link
  - Rate the app link
  - Contact / feedback link
- [ ] Save onboarding completion to shared_preferences

**Weekday Evenings (~3 hrs total):**
- [ ] Accessibility audit: font scaling, screen reader labels, contrast ratios
- [ ] Add haptic feedback on key actions
- [ ] Performance check: list with 50+ items should scroll smoothly
- [ ] Fix any accumulated bugs from testing

**Phase 3 Testing Gate — DO NOT PROCEED TO PHASE 4 UNTIL ALL PASS:**
- [ ] `flutter analyze` returns no errors
- [ ] `flutter test` — all tests pass
- [ ] Manual test: add item → notification fires at correct time (test with short delay)
- [ ] Manual test: delete item → scheduled notification is cancelled
- [ ] Manual test: notification tap → app opens to correct item (test from killed state too)
- [ ] Manual test: notification permission denied → app still works, settings prompt shown
- [ ] Manual test: just-in-time permission flow → first item save triggers permission request
- [ ] Manual test: onboarding flow completes → not shown again on relaunch
- [ ] Manual test: dark mode → all screens render correctly
- [ ] Manual test: settings changes persist after app restart
- [ ] Accessibility spot check: VoiceOver/TalkBack reads key elements correctly
- [ ] Performance check: list with 30+ items scrolls smoothly (no jank)
- [ ] Google Play closed testing track has 20+ testers and has been running for at least 7 days (14 days needed total — check timeline)
- [ ] Crashlytics is receiving test crash reports
- [ ] All decisions documented in `DEVLOG.md`
- [ ] `PROJECT_PLAN.md` checkboxes updated

**Phase 3 Checkpoint:** App is feature-complete for MVP. Notifications work. UI is polished and feels professional. Onboarding guides new users.

---

## Phase 4: App Store Launch (Weeks 7-8)

### Week 7 — Store Preparation

**Saturday (4-6 hrs): Legal + Metadata**
- [ ] Generate privacy policy (use freeprivacypolicy.com or similar):
  - Camera usage: to scan barcodes and expiry dates
  - Local storage: food item data stored on-device only
  - Network: Open Food Facts API calls (no personal data sent)
  - Analytics: Firebase Analytics collects anonymous usage data (feature usage, crash reports). No personally identifiable information is collected. Disclose this clearly.
  - No account/login required
  - No third-party advertising
- [ ] Host privacy policy (GitHub Pages is free and easy)
- [ ] Write App Store description (see template below)
- [ ] Write Google Play description
- [ ] Prepare keyword list for ASO:
  - Primary: food expiration tracker, expiry date reminder, food waste
  - Secondary: grocery tracker, fridge organizer, best before date, pantry manager
- [ ] Choose app category: Food & Drink (iOS), Food & Drink (Android)

**Sunday (4-6 hrs): Screenshots + Assets**
- [ ] Create 6 App Store screenshots (6.7" iPhone 15 Pro Max):
  1. Home screen with items (color-coded urgency)
  2. Barcode scanning in action
  3. OCR date scanning
  4. Notification preview
  5. Add item form
  6. Categories/filtering
- [ ] Create Android screenshots (same content, Android frame)
- [ ] App Store preview video (optional but helps — 15-30 sec screen recording)
- [ ] Finalize app icon (1024x1024 for iOS, 512x512 for Android)
- [ ] Set up Apple Developer account ($99/year) if not already done
- [ ] Set up Google Play Console ($25 one-time) if not already done

**Weekday Evenings (~3 hrs total):**
- [ ] Prepare promotional text (short description, 30 chars max for iOS)
- [ ] Create feature graphic for Google Play (1024x500)
- [ ] Set up support email address
- [ ] Review Apple's App Store Review Guidelines sections:
  - 2.3: Accurate metadata
  - 4.0: Design (minimum functionality)
  - 5.1: Privacy (data collection disclosure)

### Week 8 — Testing + Submission

**Saturday (4-6 hrs): Beta Testing**
- [ ] iOS: Set up TestFlight
  - Build archive: `flutter build ipa`
  - Upload to App Store Connect
  - Add 3-5 beta testers (friends/family)
- [ ] Android: Verify Google Play closed testing track (started in Week 5) has completed 14-day requirement with 20+ testers. If not, this BLOCKS public launch — extend timeline as needed.
  - Upload latest build: `flutter build appbundle`
- [ ] Create a testing checklist:
  - [ ] Add item manually → verify saved
  - [ ] Scan barcode → verify product found
  - [ ] Scan expiry date → verify parsed correctly
  - [ ] Receive notification → verify timing
  - [ ] Delete item → verify removed
  - [ ] Mark consumed → verify moved
  - [ ] Filter by category → verify works
  - [ ] Dark mode → verify no UI issues
  - [ ] Kill app and reopen → verify data persists
  - [ ] No internet → verify app works offline (barcode lookup gracefully fails)

**Sunday (4-6 hrs): Fix + Submit**
- [ ] Fix critical bugs from beta testing
- [ ] Final build with version 1.0.0+1
- [ ] Submit to Apple App Store (review takes 1-3 days typically)
- [ ] Submit to Google Play Store (review takes 1-3 days, sometimes hours)
- [ ] Add App Store Connect metadata:
  - Privacy nutrition labels (camera usage, no tracking)
  - Content rating questionnaire
  - Pricing: Free with In-App Purchases
- [ ] Add Google Play metadata:
  - Content rating questionnaire
  - Data safety section
  - Pricing: Free

**Weekday Evenings (~3 hrs total):**
- [ ] Monitor review status
- [ ] Respond to any App Store rejection issues (common: missing privacy description, metadata mismatch)
- [ ] Prepare launch social media posts
- [ ] Post on Reddit: r/sideproject, r/apps, r/zerowaste, r/flutterdev

**Phase 4 Testing Gate — DO NOT SUBMIT TO STORES UNTIL ALL PASS:**
- [ ] Full testing checklist from Week 8 Saturday — ALL items green
- [ ] TestFlight build installs and runs on a real iPhone
- [ ] Android internal test build installs and runs on a real Android device (or emulator)
- [ ] Privacy policy URL loads correctly in a browser
- [ ] Food safety disclaimer visible in Settings > About
- [ ] App icon displays correctly on home screen (no clipping, correct rounded corners)
- [ ] All screenshots accurately represent the current app
- [ ] No hardcoded test data or debug prints in release build
- [ ] `flutter build ipa` and `flutter build appbundle` succeed without errors
- [ ] All decisions documented in `DEVLOG.md`
- [ ] Final session summary written in `DEVLOG.md`

**Phase 4 Checkpoint:** App is LIVE on both stores. You have a real product in the world.

---

## Phase 5: Post-Launch & Monetization (Weeks 9+)

### Immediate Post-Launch (Week 9-10)
- [ ] Monitor crash reports via Firebase Crashlytics dashboard (already integrated since Phase 1)
- [ ] Review Firebase Analytics data: scan success rates, items per user, notification engagement (already integrated since Phase 1)
- [ ] Respond to user reviews promptly
- [ ] Fix critical bugs ASAP (submit patch releases)
- [ ] Implement in-app review prompt (after user has tracked 5+ items)
- [ ] Add "Share your food waste savings" social card (first viral/referral mechanism)

### Monetization Implementation (Week 11-12)
- [ ] Implement RevenueCat or in-app purchases directly
- [ ] Define premium tier — **FreshCheck Pro** ($1.99/month or $14.99/year):
  - Unlimited items (free tier: 20 items)
  - Multiple storage locations (custom categories)
  - Household sharing via iCloud/Google sync
  - Home screen widget
  - Custom notification schedules
  - Priority support
- [ ] Build paywall screen (show after user hits 20-item limit or after 30 days)
- [ ] Set up App Store Connect / Google Play subscriptions

### Growth Features (Weeks 13+)
- [ ] Home screen widgets (iOS WidgetKit, Android Widgets)
- [ ] iCloud / Google Drive backup
- [ ] Household sharing (share a fridge with roommates/family)
- [ ] "Weekly Waste Report" — gamify reducing waste
- [ ] Recipe suggestions for items expiring soon (link to external recipe APIs)
- [ ] Shopping list integration
- [ ] Apple Watch complication (items expiring today)
- [ ] Siri Shortcuts / Google Assistant integration

---

## App Store Description Template

### iOS App Store

**App Name:** FreshCheck — Expiry Tracker
**Subtitle:** Never waste food again

**Description:**
Stop throwing away expired food. FreshCheck helps you track expiration dates so you always know what to use first.

SCAN & TRACK
- Scan barcodes to instantly identify products
- Scan expiry dates with your camera — no typing needed
- Or quickly add items manually

SMART REMINDERS
- Get notified before food expires
- Color-coded urgency so you see what needs attention
- Customize when and how you get reminded

ORGANIZE YOUR KITCHEN
- Sort by Fridge, Freezer, and Pantry
- See everything at a glance
- Mark items as consumed to track your waste

The average household throws away $1,500 of food per year. FreshCheck helps you save money and reduce waste.

**Keywords:** food expiration tracker, expiry date reminder, food waste, grocery tracker, fridge organizer, best before, pantry manager, food saver, expiration date, fresh food

---

## Monetization Pricing Strategy

| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | 20 items, barcode scan, OCR scan, basic notifications, 3 categories |
| Pro Monthly | $1.99/mo | Unlimited items, custom categories, widgets, household sharing, cloud backup |
| Pro Annual | $14.99/yr | Same as monthly (save 37%) |

**Why this pricing:**
- $1.99/mo is an impulse buy — low friction for conversion
- Annual at $14.99 incentivizes commitment (effective $1.25/mo)
- Free tier is generous enough to be useful (20 items covers most people's fridge)
- Paywall triggers when users hit the limit (they're already invested)

---

## User Acquisition Strategy

### Week of Launch
1. **Reddit:** Post on r/sideproject, r/sideprojects, r/apps, r/zerowaste, r/frugal, r/mealprep, r/flutterdev
2. **Product Hunt:** Submit as a launch (prepare 1 week before)
3. **Hacker News:** "Show HN: I built an app to stop throwing away food"

### Ongoing
4. **TikTok/Reels content ideas:**
   - "The average family wastes $1,500/year in food. I built an app to fix that."
   - "I scanned everything in my fridge in 2 minutes" (satisfying scan montage)
   - "POV: Your app tells you the milk expires tomorrow" (relatable humor)
5. **ASO Optimization:** Monitor keyword rankings, iterate on title/subtitle/keywords
6. **Food waste awareness days:** World Food Day (Oct 16), Stop Food Waste Day (April)

---

## Critical Gaps & Risks

### Technical Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| OCR can't parse all date formats | High — users give up | Ship with 15+ regex patterns. Add "manual fallback" button prominently. Iterate based on user reports. |
| Open Food Facts missing products | Medium — UX friction | Graceful fallback to manual entry. Barcode cache table avoids repeated failed lookups. |
| Notifications not firing reliably | High — core feature broken | Test extensively. Handle Doze mode (Android), background refresh (iOS). Reschedule on app open. |
| Camera performance on old devices | Medium | Set minimum iOS 15 / Android API 26. Test on low-end devices. |
| App binary size > 80MB | Medium — users skip download | google_mlkit bundles ML models. Check size early (Phase 1). If too large, investigate thin models or on-demand download. |
| Database migration breaks on update | High — data loss | Set schemaVersion from Day 1. Never modify existing columns — only add new ones. Test migrations before every release. |
| Local images bloat storage | Medium — users complain | Store images in app documents dir, compress to max 500KB. Add "Storage used" indicator in settings. Clear images for deleted items. |
| OCR picks up wrong date (mfg instead of exp) | High — wrong expiry stored | Use keyword proximity scoring: prioritize dates near "EXP", "BB", "USE BY". Never auto-select — always confirm with user. |
| Google Play 20-tester/14-day gate | High — blocks launch | Start closed testing in Week 5, not Week 8. Recruit testers from friends, family, classmates, coworkers early. |

### Business Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| Users don't hit 20-item limit | Low conversion | Track with analytics. Consider lowering to 15 or feature-gating OCR scan behind premium instead. |
| Competitor launches same thing | Medium | Speed to market matters. Build community around food waste. |
| App Store rejection | Medium — delays launch | Follow guidelines strictly. Privacy policy required. Camera permission description must be specific. |
| Low retention after install | High | Summary header on home screen for daily glance value. Just-in-time notifications. Quick-add for grocery hauls. |
| No viral/referral loop | Medium — slow organic growth | Add "Share your waste stats" social card. "Invite household member" in free tier. Build sharing into v1.1 if not MVP. |
| App name already taken | High — rework branding | Validate name in Phase 0 BEFORE writing any code. Have 2-3 backup names ready. |
| Flying blind without analytics | High — can't optimize | Firebase Analytics from Day 1. Track: items added/session, scan success rate, notification tap rate, paywall views. |

### Success Metrics (Define what "winning" looks like)
| Metric | 30-Day Target | 90-Day Target |
|--------|---------------|---------------|
| Downloads | 500 | 2,000 |
| Day-7 retention | 30% | 35% |
| Items added per active user | 5+ | 10+ |
| Scan success rate (barcode) | 70%+ | 80%+ |
| Premium conversion | — | 3-5% of active users |
| App Store rating | 4.0+ | 4.3+ |

### Legal Requirements
- [ ] **Privacy Policy** — REQUIRED by both stores. Must describe camera, storage, and API usage.
- [ ] **Food Safety Disclaimer** — "FreshCheck provides reminders based on dates you enter. Always use your judgment regarding food safety. We are not responsible for food safety decisions." Include in app + privacy policy.
- [ ] **No health claims** — Don't say "prevents food poisoning." Say "helps track expiry dates."

---

## Development Environment Setup

### First-Time Setup Commands
```bash
# Install Flutter (if not already)
# Visit https://docs.flutter.dev/get-started/install

# Create project
cd /Users/volannnanpalle/Desktop/Side\ Projects/expiry_reminder
flutter create --org com.freshcheck --project-name freshcheck .

# Verify setup
flutter doctor

# Install dependencies
flutter pub get

# Run code generation (after defining Drift tables + Riverpod providers)
dart run build_runner build --delete-conflicting-outputs

# Run app
flutter run

# Run tests
flutter test
```

### Required Accounts
- [ ] Apple Developer Program ($99/year) — developer.apple.com
- [ ] Google Play Console ($25 one-time) — play.google.com/console
- [ ] Open Food Facts — no account needed (free API)
- [ ] GitHub repo for version control

---

## Definition of Done (MVP)

The app is ready to submit when ALL of these are true:

**Core Features:**
- [ ] User can add items via manual entry (including "Save & Add Another" for batch adding)
- [ ] User can quick-select common produce items with preset shelf lives
- [ ] User can scan barcodes to auto-fill product info (with barcode cache for speed)
- [ ] User can scan expiry dates via OCR (with user confirmation, never auto-select)
- [ ] Items display in a sorted, color-coded list with summary header ("X items expiring this week")
- [ ] Items can be filtered by category
- [ ] Items can be edited and deleted (swipe-to-delete with undo SnackBar)
- [ ] Items can be marked as consumed
- [ ] Expired items are grouped in a collapsible section with "Clear all" option
- [ ] Notifications fire correctly (3 days, 1 day, day-of) with deep linking to specific item
- [ ] Notification permission is requested just-in-time (on first item save), NOT during onboarding
- [ ] App works fully offline (except barcode lookup)

**Quality & Operations:**
- [ ] Firebase Crashlytics is active and receiving reports
- [ ] Firebase Analytics is tracking key events (item_added, barcode_scanned, ocr_scanned, notification_tapped)
- [ ] Database schema version is set to 1 with migration strategy in place
- [ ] App binary size is under 100MB (check both platforms)
- [ ] No crashes on iPhone 13+ and Pixel 5+
- [ ] Dark mode works correctly
- [ ] Onboarding flow guides new users (2 screens, no permission requests)
- [ ] Settings screen is functional

**Store Readiness:**
- [ ] App name validated as available on both stores
- [ ] Privacy policy is hosted and linked
- [ ] Food safety disclaimer is visible in Settings > About
- [ ] App icon and splash screen look professional
- [ ] App Store screenshots are ready
- [ ] App Store description and keywords are written
- [ ] Google Play closed testing has completed 14-day / 20-tester requirement
- [ ] No hardcoded test data or debug prints in release build
- [ ] All implementation decisions documented in `DEVLOG.md`
