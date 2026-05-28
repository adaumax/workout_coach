# Workout App — Living Plan

## How to use this file

At the start of every session, tell Claude:
> "Check PLAN.md and continue from where we left off."

Claude will read this file, understand the current state, and pick up exactly where we stopped.
Update this file at the end of every session before closing.

---

## Current Status

**Active step:** None — not started yet
**Last session:** Spec updated with GDPR, security, multi-user, and pedagogy constraints
**Next action:** Verify Node.js and Python versions, then scaffold the project (Step 1)

---

## Build Steps

### Step 1 — Project Scaffold
**Status:** ⬜ Todo

**Goal:** Create the folder structure, install dependencies, configure environment
variables, and verify that frontend and backend can run locally and communicate.

**Definition of done:**
- `frontend/` runs on http://localhost:3000 and shows a placeholder page
- `backend/` runs on http://localhost:8000 and responds to `GET /health`
- `.env.example` documents all required variables (Supabase URL, keys, Anthropic key)
- Frontend can successfully call the backend health endpoint
- Supabase project created, connection tested from backend

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 2 — Auth & User Profiles
**Status:** ⬜ Todo

**Goal:** Implement multi-user authentication with GDPR-compliant registration,
login, profile management, data export, and account deletion.

**Definition of done:**
- Supabase Auth configured (email/password)
- Row-Level Security (RLS) enabled on all user-scoped tables
- `POST /auth/register` — creates account, records `gdpr_consent_at`
- `POST /auth/login` — returns JWT token
- `GET /profile` / `PATCH /profile` — read and update username, goal, avatar
- `GET /profile/export` — returns all user data as JSON
- `DELETE /profile` — hard-deletes account and all associated data
- `/register` page with email, password, and GDPR consent checkbox (required)
- `/login` page
- `/profile` page with goal selector, data export button, delete account button
- `/privacy` page with full privacy policy
- Unauthenticated users are redirected to `/login`
- Rate limiting active on login and register endpoints

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 3 — Exercise Library
**Status:** ⬜ Todo

**Goal:** Pre-load a curated set of gym exercises and build a browsable,
pedagogically rich library page.

**Definition of done:**
- `exercise` table created in Supabase (shared, not user-scoped)
- ~40 exercises seeded: name, muscle groups, difficulty, image, description,
  muscles worked, common mistakes, beginner tip
- `GET /exercises` endpoint (public, no auth needed)
- `GET /exercises/{id}` endpoint
- `/exercises` page with search, muscle group filter, difficulty filter
- Each exercise card shows a tooltip with muscles worked, common mistakes, beginner tip
- Muscle groups color-coded consistently (same colors used in calendar later)

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 4 — Session Logging
**Status:** ⬜ Todo

**Goal:** Allow authenticated users to start a workout session, add exercises,
and log sets. Idempotent and safe to retry.

**Definition of done:**
- Tables `session`, `session_exercise`, `set` created with RLS enabled
- `POST /sessions` — create session (accepts `idempotency_key`)
- `GET /sessions` — paginated session history for current user
- `GET /sessions/{id}` — session detail with all exercises and sets
- `POST /sessions/{id}/exercises` — add an exercise to a session
- `POST /sessions/{id}/exercises/{ex_id}/sets` — log a set (idempotent on order)
- `PATCH /sessions/{id}` — update notes, body weight
- `DELETE /sessions/{id}` — delete a session (with confirmation)
- Basic UI: create session, add exercises from library, log sets
- Human-readable error messages on all failure states

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 5 — Session Live Page + Timer
**Status:** ⬜ Todo

**Goal:** Build the guided workout page for use during a live session, with
auto-save and offline support.

**Definition of done:**
- Step-by-step view: one exercise highlighted at a time with its tooltip info
- Linear recap always visible below (done, in progress, upcoming)
- Rest timer starts automatically between sets, with skip and adjust buttons
- "Done" advances to the next set or exercise
- Session state is persisted to localStorage — closing and returning resumes exactly
- Sets logged optimistically (UI updates before server confirms)
- Offline queue: sets logged offline are synced when connection is restored
- Exercise tooltip accessible during the session without leaving the page

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 6 — History & Calendar
**Status:** ⬜ Todo

**Goal:** Show past sessions in a calendar with muscle group color-coding and
session detail view.

**Definition of done:**
- Calendar page shows all trained days highlighted
- Each day color-coded by muscle group(s) worked (using the same palette as the library)
- Clicking a day opens a session summary (exercises, sets, notes)
- Empty days show a subtle motivational message
- Fully paginated — can scroll back months

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 7 — Progress Charts
**Status:** ⬜ Todo

**Goal:** Visualize performance trends per exercise and overall volume, with
plain-language summaries.

**Definition of done:**
- Chart: max weight over time per exercise
- Chart: total volume (sets × reps × weight) per session
- Each chart includes a plain-language summary generated from the data
  (e.g. "You increased your squat by 10kg in 6 weeks — great progress")
- Accessible from exercise detail page and from history

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 8 — Motivational Alerts & Progression Suggestions
**Status:** ⬜ Todo

**Goal:** Surface smart, pedagogical nudges before and after sessions.

**Definition of done:**
- Dashboard alert if no session in the last 3 days (configurable threshold)
- Dashboard alert if same muscle group trained 3+ days in a row, with an explanation
  of why rest matters for that muscle group
- Before starting a session: list of exercises with possible progressions
  (e.g. "Last time: 3×10 at 60kg → try 3×10 at 62.5kg or 3×11 at 60kg")
- Each suggestion includes a short pedagogical note explaining progressive overload

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 9 — AI Advice Page
**Status:** ⬜ Todo

**Goal:** Integrate Claude API to generate personalized advice with explained reasoning.

**Definition of done:**
- Anthropic API key configured in `.env`
- `POST /advice` sends recent session data + user goal to Claude, returns structured advice
- Advice categories: recovery, progression, program suggestion, injury prevention
- Each advice item includes a plain-language explanation of the reasoning
- Advice is cached (avoid calling Claude on every page visit)
- If Claude is unavailable, show last cached advice with a warning banner
- `/advice` page with clear sections and "Refresh advice" button

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 10 — PWA Configuration
**Status:** ⬜ Todo

**Goal:** Make the app installable on phone from the browser with offline support.

**Definition of done:**
- `manifest.json` configured (name, icons, theme color, display: standalone)
- Service worker handles offline caching for the session live page
- App installable on iOS Safari and Android Chrome
- Tested on a real phone — session page fully usable without internet

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 11 — Nutrition Advice
**Status:** ⬜ Todo

**Goal:** Add Claude-powered nutrition advice tailored to the user's goal and
recent training load.

**Definition of done:**
- User's goal (muscle gain / fat loss / maintenance) set in profile
- `POST /advice/nutrition` returns meal timing and macro suggestions with explanations
- Advice shown on the `/advice` page below training advice
- Pedagogical: macros explained in plain language (what protein does, why timing matters)

**Key decisions:** —
**Files created:** —
**Notes:** —

---

## Decision Log

| Date | Decision | Reason |
|------|----------|--------|
| 2026-05-28 | Next.js 14 App Router for frontend | Most modern approach, best for job market |
| 2026-05-28 | FastAPI (Python) for backend | User preference + great for AI integration |
| 2026-05-28 | Supabase for database + auth | Managed Postgres, built-in auth, RLS, free tier |
| 2026-05-28 | Supabase Auth (email/password only) | Simplest GDPR-compliant auth, OAuth deferred |
| 2026-05-28 | RLS enabled on all user-scoped tables | Database enforces user isolation, not just the API |
| 2026-05-28 | Hard delete on account removal | GDPR right to erasure — no soft delete for account data |
| 2026-05-28 | All weights in kg, durations in seconds | Consistency across the data model |
| 2026-05-28 | Optimistic UI for set logging | Session page must feel instant, especially on mobile |

---

## Known Blockers

- Need to confirm Node.js 18+ and Python 3.11+ are installed
- Need a Supabase account (free tier is sufficient) — https://supabase.com
- Need an Anthropic API key for Steps 9 and 11 (can be skipped and added later)