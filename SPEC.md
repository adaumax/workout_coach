# Workout App — Product Specification

## 1. What it is

A multi-user indoor muscle workout tracker, accessible as a Progressive Web App (PWA)
on both laptop and phone. No app store required — installs from the browser.
Designed to be pedagogical: the app teaches the user about training, exercises, and health
through tooltips, explanations, and contextual guidance at every step.

---

## 2. Tech Stack

| Layer        | Technology                        |
|--------------|-----------------------------------|
| Frontend     | Next.js 14 + React + Tailwind CSS |
| Backend      | FastAPI (Python 3.11+)            |
| Database     | Supabase (PostgreSQL)             |
| Auth         | Supabase Auth (JWT, email/password)|
| AI           | Claude API (anthropic Python SDK) |
| Phone access | PWA (installable from browser)    |

---

## 3. Non-Functional Requirements

These apply to every feature, not just one step. Every piece of code must satisfy them.

### 3.1 GDPR Compliance

- **Explicit consent**: Users must accept a clear privacy notice on signup. Consent is
  timestamped and stored (`gdpr_consent_at`).
- **Data minimization**: Only collect data that is strictly necessary for the feature.
  No tracking pixels, no analytics by default.
- **Right to erasure**: Users can permanently delete their account and all associated
  data from their profile page. This triggers a hard delete cascade.
- **Data portability**: Users can export all their data (sessions, sets, body weight)
  as a JSON or CSV file from their profile page.
- **No PII in logs**: Server logs must never contain email addresses, names, or
  health data. Use user IDs only.
- **Privacy policy page**: A `/privacy` page explains what data is collected, why,
  how long it is kept, and how to exercise rights.

### 3.2 Security

- All routes (except `/login`, `/register`, `/privacy`) require a valid JWT token.
- Tokens are issued by Supabase Auth and validated on every backend request.
- Every database query is scoped to the authenticated `user_id` — a user can never
  read or write another user's data.
- All inputs are validated server-side with Pydantic. Never trust the frontend.
- Passwords are never stored — Supabase Auth handles hashing.
- Rate limiting on auth endpoints (login, register) to prevent brute force.
- CORS restricted to the frontend origin only.
- No secrets or credentials committed to git. All configuration via `.env` files.
  A `.env.example` file documents all required variables.
- HTTPS enforced in all non-local environments.

### 3.3 Ergonomy & Accessibility

- Fully responsive: all pages usable on a 375px mobile screen and a 1440px desktop.
- WCAG 2.1 AA compliance: sufficient color contrast, keyboard navigation, ARIA labels
  on all interactive elements, focus indicators visible.
- Touch targets minimum 44×44px on mobile.
- Loading states on every async action (spinner or skeleton screen) — never leave
  the user wondering if something is happening.
- Error messages must be human-readable and actionable ("Your session could not be
  saved. Please check your connection and try again.") — never show raw error codes.
- Confirmation dialogs before any destructive action (delete session, delete account).

### 3.4 Reliability & Failovers

- **Optimistic UI**: when logging a set, the UI updates immediately without waiting
  for the server. If the server call fails, the UI rolls back and shows an error.
- **Offline support**: the session live page works offline. Sets logged while offline
  are queued and synced when the connection is restored (PWA service worker).
- **Retry logic**: all API calls from the frontend retry up to 3 times with
  exponential backoff before showing an error to the user.
- **Graceful degradation**: if the AI advice endpoint is unavailable, the advice page
  shows the last cached advice with a warning, not a blank page.
- **Health endpoint**: `GET /health` returns backend and database status. Used for
  uptime monitoring.

### 3.5 Idempotency

- `POST /sessions` accepts an optional `idempotency_key` (UUID generated client-side).
  Replaying the same key returns the existing session instead of creating a duplicate.
- `POST /sets` is idempotent: submitting the same set twice (same session, exercise,
  order, reps, weight) does not create a duplicate.
- All write operations are safe to retry after a network failure.

### 3.6 Pedagogy

Every feature must include contextual education for the user:
- **Exercise tooltips**: each exercise shows the muscles worked, why it is useful,
  common mistakes to avoid, and a beginner tip.
- **AI explanations**: every AI recommendation includes a plain-language explanation
  of the reasoning ("We suggest a deload this week because your volume over the last
  7 days is 40% above your average").
- **Training concepts**: terms like "RPE", "deload", "progressive overload" are
  always accompanied by a short inline definition on first use, plus a "?" tooltip
  for subsequent uses.
- **Onboarding**: a guided onboarding flow for new users explains the app's
  philosophy and key features before the first session.
- **Progress context**: charts include a plain-language summary ("You increased your
  bench press by 5kg over the last 4 weeks — that's excellent progress").

---

## 4. User Stories

### Auth & Profile
- As a new user, I can register with email and password.
- I must explicitly accept the privacy policy before my account is created.
- As a returning user, I can log in and my data is waiting for me.
- I can update my profile: username, goal (muscle gain / fat loss / maintenance),
  avatar.
- I can export all my data at any time.
- I can permanently delete my account and all my data.

### Workout Session
- I can start a new workout session at any time.
- I pick exercises freely from a pre-loaded library.
- For each exercise I log: sets × reps × weight, rest time between sets.
- I can add free-text notes per session.
- I can log my body weight on any given day.

### Session Page (Live Workout)
- A guided page walks me through my session in real time.
- One exercise is highlighted at a time (step-by-step mode).
- A linear recap of the full session is always visible below.
- A rest timer starts automatically between sets.
- I tap "Done" to advance to the next set or exercise.
- The session is auto-saved continuously — closing the browser and returning
  resumes the session exactly where I left off.

### Exercise Library
- The app ships with a curated library of common gym exercises.
- Each exercise shows: name, muscle group(s), image, description, muscles worked
  diagram, common mistakes, and a beginner tip.
- I can search and filter by muscle group or difficulty.

### Calendar & History
- A visual calendar shows which days I trained.
- Each trained day is color-coded by muscle group(s) worked.
- Progress charts show weight/reps evolution over time per exercise, with a
  plain-language summary.
- AI suggests rest days based on recent training volume.
- Motivational alerts appear on non-trained days.
- Alerts flag overtraining patterns.
- Before each session, the app highlights exercises where progression is possible.

### AI Advice
- Recovery tips after hard sessions (stretching, sleep, nutrition).
- Progression advice with reasoning explained.
- Weekly program suggestions based on history and goals.
- Injury prevention alerts with explanations.
- Basic nutrition advice tailored to goals.
- All advice includes a plain-language explanation of the reasoning.

---

## 5. Pages

| Page            | Description                                                  |
|-----------------|--------------------------------------------------------------|
| `/`             | Dashboard — today's summary, motivational alert, quick start |
| `/login`        | Login form                                                   |
| `/register`     | Registration form with GDPR consent checkbox                 |
| `/session`      | Live workout page — step-by-step guide with timer            |
| `/history`      | Calendar view + progress charts                              |
| `/exercises`    | Browse the exercise library with tooltips                    |
| `/advice`       | AI-generated advice with explanations                        |
| `/profile`      | Body weight log, goals, data export, account deletion        |
| `/privacy`      | Privacy policy (GDPR)                                        |

---

## 6. Data Model (Draft)

```
user  (managed by Supabase Auth — extended with a profile table)
  id, email, username, goal, avatar_url, gdpr_consent_at, created_at

exercise  (shared across all users — not editable by users)
  id, name, muscle_groups[], difficulty, image_url, description,
  muscles_worked_diagram_url, common_mistakes, beginner_tip

session  (per user)
  id, user_id, date, notes, duration_seconds, body_weight_kg,
  idempotency_key, completed_at

session_exercise
  id, session_id, exercise_id, order

set
  id, session_exercise_id, reps, weight_kg, rest_seconds, logged_at

body_weight  (per user)
  id, user_id, date, weight_kg
```

Row-Level Security (RLS) is enabled on all user-scoped tables in Supabase.
This ensures the database itself enforces that users can only access their own rows.

---

## 7. Out of Scope (for now)

- Social or sharing features
- Video recording
- Wearable / smartwatch integration
- Paid tiers
- OAuth login (Google, Apple) — email/password only for now

---

## 8. Build Order

1.  Project scaffold (Next.js + FastAPI + Supabase connection + env setup)
2.  Auth & User profiles (register, login, GDPR consent, profile page)
3.  Exercise library (pre-loaded data, browse page, pedagogical tooltips)
4.  Session logging (start session, add exercises, log sets)
5.  Session live page (step-by-step + timer + auto-save + offline)
6.  History & calendar view
7.  Progress charts (with plain-language summaries)
8.  Motivational alerts & progression suggestions
9.  AI advice page (Claude integration with explanations)
10. PWA configuration (installable on phone)
11. Nutrition advice (Claude)