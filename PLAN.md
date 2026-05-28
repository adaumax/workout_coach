# Workout App — Living Plan

## How to use this file

At the start of every session, tell Claude:
> "Check PLAN.md and continue from where we left off."

Claude will read this file, understand the current state, and pick up exactly where we stopped.
Update this file at the end of every session before closing.

---

## Current Status

**Active step:** None — not started yet
**Last session:** Spec updated for freemium model, mobile App Store/Play Store distribution, and Capacitor + RevenueCat integration
**Next action:** Verify Node.js and Python versions, then scaffold the project (Step 1)

---

## Build Steps

### Step 1 — Project Scaffold
**Status:** ⬜ Todo

**Goal:** Create folder structure, install dependencies, configure environment
variables, and verify frontend + backend can run locally and communicate.

**Definition of done:**
- `frontend/` runs on http://localhost:3000 and shows a placeholder page
- `backend/` runs on http://localhost:8000 and responds to `GET /health`
- `.env.example` documents all required variables
- Frontend calls backend health endpoint successfully
- Supabase project created and connection confirmed from backend

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 2 — Auth & User Profiles
**Status:** ⬜ Todo

**Goal:** Multi-user authentication with GDPR-compliant registration, login,
profile management, data export, and account deletion.

**Definition of done:**
- Supabase Auth configured (email/password)
- `profile` table created, RLS enabled on all user-scoped tables
- `subscription` table created (default plan = "free" on register)
- `POST /auth/register` — creates account, records `gdpr_consent_at`, sets plan = free
- `POST /auth/login` — returns JWT
- `GET /profile` / `PATCH /profile` — username, goal, avatar
- `GET /profile/export` — all user data as JSON (free, always)
- `DELETE /profile` — hard-deletes account + all data cascade
- `/register` page with GDPR consent checkbox (required field)
- `/login` page
- `/profile` page with goal, export, delete account
- `/privacy` page with full privacy policy text
- Unauthenticated users redirected to `/login`
- Rate limiting on login and register

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 3 — Exercise Library
**Status:** ⬜ Todo

**Goal:** Pre-load curated gym exercises, build a pedagogically rich browsable page.

**Definition of done:**
- `exercise` table created (shared, no RLS needed — public data)
- ~40 exercises seeded: name, muscle groups, difficulty, image, description,
  muscles worked, common mistakes, beginner tip
- `GET /exercises` and `GET /exercises/{id}` endpoints
- `/exercises` page with search, muscle group filter, difficulty filter
- Exercise card shows tooltip: muscles worked, common mistakes, beginner tip
- Muscle group colors consistent (same palette used in calendar)

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 4 — Session Logging
**Status:** ⬜ Todo

**Goal:** Authenticated users can start sessions, add exercises, log sets (all free).

**Definition of done:**
- Tables `session`, `session_exercise`, `set` created with RLS
- `POST /sessions` (idempotency_key)
- `GET /sessions` (paginated — 30 days for free users, unlimited for premium)
- `GET /sessions/{id}` — full session detail
- `POST /sessions/{id}/exercises`
- `POST /sessions/{id}/exercises/{ex_id}/sets` (idempotent on order)
- `PATCH /sessions/{id}` — notes, body weight
- `DELETE /sessions/{id}` — with confirmation
- Basic UI: create session, pick exercises, log sets
- Human-readable errors on all failure states

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 5 — Session Live Page + Timer
**Status:** ⬜ Todo

**Goal:** Guided workout page with auto-save. Offline capability added in Step 12.

**Definition of done:**
- Step-by-step view: one exercise at a time with tooltip info
- Linear recap always visible (done / in progress / upcoming)
- Rest timer with skip and adjust buttons
- "Done" advances to next set or exercise
- Session state persisted to localStorage — resume if browser closes
- Sets logged optimistically (UI updates before server confirms)
- Exercise tooltip accessible without leaving the page

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 6 — History & Calendar
**Status:** ⬜ Todo

**Goal:** Visual calendar of training history with free/premium history gate.

**Definition of done:**
- Calendar shows trained days color-coded by muscle group(s)
- Free users: last 30 days visible, older days blurred with premium upsell
- Premium users: full unlimited history
- Clicking a day shows session summary
- Empty days show a motivational nudge
- Premium upsell card explains the value of full history

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 7 — Progress Charts (Premium)
**Status:** ⬜ Todo

**Goal:** Visualize performance trends per exercise, gated behind premium.

**Definition of done:**
- `GET /progress/exercises/{id}` and `GET /progress/summary` (premium-only)
- Chart: max weight over time per exercise
- Chart: total volume per session
- Plain-language trend summary generated from the data
- Free users see a preview + premium upsell explaining the feature
- Accessible from exercise detail and history

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 8 — Motivational Alerts & Progression Suggestions (Premium)
**Status:** ⬜ Todo

**Goal:** Smart nudges before and after sessions, gated behind premium.

**Definition of done:**
- Alert: no session in the last 3 days (dashboard)
- Alert: same muscle group 3+ days in a row (with pedagogical explanation)
- Pre-session: list exercises where progression is possible, with explanation
- Free users see a sample alert + premium upsell
- All alerts include a plain-language "why this matters" note

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 9 — AI Advice Page (Premium)
**Status:** ⬜ Todo

**Goal:** Claude-powered personalized advice, gated behind premium.

**Definition of done:**
- Anthropic API key configured
- `POST /advice` sends session data + goal to Claude, returns structured advice
- Categories: recovery, progression, program, injury prevention
- Each item includes plain-language reasoning
- Advice cached — not regenerated on every visit
- Claude unavailable → show last cached advice with warning
- `/advice` page with sections and "Refresh" button
- Free users see a preview + upsell

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 10 — Freemium: Subscription Model + Premium Gates
**Status:** ⬜ Todo

**Goal:** Implement RevenueCat, connect subscription status to feature gates,
build the `/subscription` page.

**Definition of done:**
- RevenueCat account created, products defined (monthly, yearly)
- `POST /subscription/verify` validates Apple/Google receipt via RevenueCat
- `POST /subscription/webhook/revenuecat` syncs subscription status to DB
- `require_premium` FastAPI dependency created and applied to all premium endpoints
- All premium features return HTTP 402 (with a descriptive message) for free users
- `/subscription` page shows: current plan, pricing table, upgrade/cancel flow
- Premium activates instantly after payment across all devices
- Cancellation takes effect at period end; data is never deleted on cancel

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 11 — Stripe Web Payments
**Status:** ⬜ Todo

**Goal:** Allow users to subscribe via the web using Stripe.

**Definition of done:**
- Stripe account configured with monthly and yearly price IDs
- `POST /subscription/webhook/stripe` handles subscription lifecycle events
- Checkout flow on `/subscription` page (Stripe Checkout or embedded)
- Subscription status synced to `subscription` table via webhook
- RevenueCat notified of Stripe subscriptions for unified entitlement management

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 12 — Offline Mode (Premium)
**Status:** ⬜ Todo

**Goal:** PWA service worker enables offline session logging for premium users.

**Definition of done:**
- Service worker caches the session page and exercise library
- Sets logged offline are queued in IndexedDB
- Queue is synced automatically when connection is restored
- Offline mode only activates for premium users (checked before caching)
- Visual indicator when the app is offline

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 13 — Capacitor: iOS + Android Packaging
**Status:** ⬜ Todo

**Goal:** Wrap the Next.js app with Capacitor and build for iOS and Android.

**Definition of done:**
- Capacitor installed and configured (`capacitor.config.ts`)
- `ios/` and `android/` folders generated
- App builds and runs on iOS Simulator and Android Emulator
- RevenueCat iOS and Android SDKs integrated
- Haptic feedback on key actions (set logged, rest timer done)
- App icon and splash screen configured
- `capacitor://localhost` added to CORS allowed origins in backend

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 14 — App Store & Play Store Submission
**Status:** ⬜ Todo

**Goal:** Publish to Apple App Store and Google Play Store.

**Definition of done:**
- Apple Developer account active (99 USD/year)
- Google Play Developer account active (25 USD one-time)
- App signed for iOS (provisioning profile + distribution certificate)
- App signed for Android (release keystore)
- Privacy policy URL (`/privacy`) included in both store listings
- App Store screenshots prepared (at least iPhone + iPad)
- Play Store screenshots prepared
- Age rating completed (both stores)
- App submitted and approved

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 15 — Nutrition Advice (Premium)
**Status:** ⬜ Todo

**Goal:** Claude-powered nutrition advice tailored to goal and training load.

**Definition of done:**
- `POST /advice/nutrition` returns meal timing + macro suggestions with explanations
- User's goal (muscle gain / fat loss / maintenance) used as context
- Macros explained in plain language (what protein does, why timing matters)
- Shown on `/advice` page below training advice

**Key decisions:** —
**Files created:** —
**Notes:** —

---

## Decision Log

| Date | Decision | Reason |
|------|----------|--------|
| 2026-05-28 | Next.js 14 App Router | Most modern, best for job market |
| 2026-05-28 | FastAPI (Python) backend | User preference + great for AI |
| 2026-05-28 | Supabase for DB + Auth | Managed Postgres, RLS, built-in auth, free tier |
| 2026-05-28 | Supabase Auth email/password only | Simplest GDPR-compliant auth; OAuth deferred |
| 2026-05-28 | Capacitor for mobile | One codebase for web + iOS + Android |
| 2026-05-28 | RevenueCat for IAP | Normalizes Apple + Google APIs; industry standard |
| 2026-05-28 | Stripe for web payments | Lower fees than App Store/Play Store (2.9% vs 30%) |
| 2026-05-28 | Hard delete on account removal | GDPR right to erasure |
| 2026-05-28 | Data export always free | GDPR Article 20 — legal right, cannot be paywalled |
| 2026-05-28 | Core session logging free | Free tier must be genuinely useful to drive adoption |
| 2026-05-28 | All weights kg, durations seconds | Consistency across data model |
| 2026-05-28 | Optimistic UI for set logging | Session page must feel instant on mobile |

---

## Known Blockers

- Confirm Node.js 18+ and Python 3.11+ installed
- Supabase account needed (free tier) — https://supabase.com
- Anthropic API key needed for Steps 9 and 15 (can be skipped and added later)
- RevenueCat account needed for Step 10 — https://revenuecat.com (free tier available)
- Stripe account needed for Step 11 — https://stripe.com
- Apple Developer account (99 USD/year) and Google Play account (25 USD one-time) needed for Step 14