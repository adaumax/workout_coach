# Workout App — Living Plan

## How to use this file

At the start of every session, tell Claude:
> "Check PLAN.md and continue from where we left off."

Claude will read this file, understand the current state, and pick up exactly where we stopped.
Update this file at the end of every session before closing.

---

## Current Status

**Active step:** None — not started yet
**Last session:** Spec and plan created
**Next action:** Verify Node.js and Python versions, then scaffold the project (Step 1)

---

## Build Steps

### Step 1 — Project Scaffold
**Status:** ⬜ Todo

**Goal:** Create the folder structure, install dependencies, and make sure frontend
and backend can run locally and talk to each other.

**Definition of done:**
- `frontend/` runs on http://localhost:3000 and shows a placeholder page
- `backend/` runs on http://localhost:8000 and responds to GET /health
- Frontend can call the backend successfully
- Supabase project created and connection tested

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 2 — Exercise Library
**Status:** ⬜ Todo

**Goal:** Pre-load a set of common gym exercises into the database and display them
in a browsable page.

**Definition of done:**
- Database table `exercise` created in Supabase
- ~30 exercises seeded (name, muscle group, image, description)
- `GET /exercises` API endpoint works
- `/exercises` page in the frontend displays the library with filters by muscle group

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 3 — Session Logging
**Status:** ⬜ Todo

**Goal:** Allow the user to start a workout session, add exercises, and log sets.

**Definition of done:**
- Database tables `session`, `session_exercise`, `set` created
- `POST /sessions` — create a session
- `POST /sessions/{id}/exercises` — add an exercise to a session
- `POST /sessions/{id}/exercises/{ex_id}/sets` — log a set (reps, weight, rest)
- Basic UI to create a session and log sets (no live timer yet)

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 4 — Session Live Page + Timer
**Status:** ⬜ Todo

**Goal:** Build the guided workout page used during a live session.

**Definition of done:**
- Step-by-step view: one exercise highlighted at a time
- Linear recap always visible (what's done, what's next)
- Rest timer starts automatically between sets
- "Done" button advances to the next set or exercise
- Session auto-saves as you go

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 5 — History & Calendar
**Status:** ⬜ Todo

**Goal:** Show past sessions in a calendar view with muscle group color-coding.

**Definition of done:**
- `GET /sessions` returns paginated session history
- Calendar page shows trained days highlighted
- Each day is color-coded by muscle group(s) worked
- Clicking a day shows the session detail

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 6 — Progress Charts
**Status:** ⬜ Todo

**Goal:** Show evolution of weight and reps over time per exercise.

**Definition of done:**
- Chart page per exercise showing max weight over time
- Chart showing total volume (sets × reps × weight) per session
- Accessible from history and from exercise library

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 7 — Motivational Alerts & Progression Suggestions
**Status:** ⬜ Todo

**Goal:** Surface smart nudges before and after sessions.

**Definition of done:**
- Alert on dashboard if no session in the last 3 days
- Alert if same muscle group trained 3+ days in a row (overtraining)
- Before a session: highlight exercises where progression is possible
  (e.g. last time 3×10 at 60kg → suggest trying 3×10 at 62.5kg or 3×11 at 60kg)

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 8 — AI Advice Page
**Status:** ⬜ Todo

**Goal:** Integrate Claude API to generate personalized advice.

**Definition of done:**
- Anthropic API key configured
- `POST /advice` endpoint sends recent session context to Claude and returns advice
- `/advice` page displays: recovery tips, progression advice, program suggestion,
  injury prevention notes
- Advice is regenerated on demand (button), not on every page load

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 9 — PWA Configuration
**Status:** ⬜ Todo

**Goal:** Make the app installable on phone from the browser.

**Definition of done:**
- `manifest.json` configured (name, icons, theme color)
- Service worker enabled (offline support for the session page)
- App installable on iOS Safari and Android Chrome
- Tested on a real phone

**Key decisions:** —
**Files created:** —
**Notes:** —

---

### Step 10 — Nutrition Advice
**Status:** ⬜ Todo

**Goal:** Add food and nutrition advice tailored to workout goals via Claude.

**Definition of done:**
- User can set a goal (muscle gain, fat loss, maintenance) in their profile
- `POST /advice/nutrition` returns meal timing and macro suggestions
  based on the day's session and the user's goal
- Displayed on the advice page alongside training advice

**Key decisions:** —
**Files created:** —
**Notes:** —

---

## Decision Log

A record of important choices made during the project.

| Date | Decision | Reason |
|------|----------|--------|
| 2026-05-28 | Next.js 14 App Router for frontend | Most modern approach, best for job market |
| 2026-05-28 | FastAPI (Python) for backend | User preference + great for AI integration |
| 2026-05-28 | Supabase for database | Managed Postgres, easy setup, free tier |
| 2026-05-28 | No user auth for now | Keep scope focused, single user app |
| 2026-05-28 | All weights in kg, durations in seconds | Consistency across the data model |

---

## Known Blockers

- Need to confirm Node.js 18+ and Python 3.11+ are installed
- Need an Anthropic API key for Steps 8 and 10 (can be added later)
- Need a Supabase account (free tier is sufficient)