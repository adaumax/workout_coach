# CLAUDE.md — Workout App

## 1. What this project is

A multi-user indoor muscle workout tracker (PWA).
Full specification is in `SPEC.md` — read it before making any changes.
The living build plan is in `PLAN.md` — check it to know where we are.

---

## 2. Repository Layout

```
frontend/        # Next.js 14 + React + Tailwind CSS (PWA)
backend/         # FastAPI (Python 3.11+)
SPEC.md          # Product specification — source of truth
PLAN.md          # Living progress tracker — updated every session
BACKLOG.md       # Feature ideas to implement later
```

---

## 3. Tech Stack

- **Frontend**: Next.js 14 (App Router), React, Tailwind CSS
- **Backend**: FastAPI (Python 3.11+), Supabase Python client
- **Auth**: Supabase Auth (JWT, email/password)
- **Database**: Supabase (PostgreSQL with Row-Level Security)
- **AI**: Claude API via `anthropic` Python SDK
- **Phone**: PWA — no app store

---

## 4. Key Conventions

### Backend (FastAPI)
- Python 3.11+
- One router per domain: `auth`, `exercises`, `sessions`, `sets`, `advice`, `profile`
- Pydantic models for all request/response schemas — validate everything server-side
- No business logic in route handlers — keep them thin, extract to service functions
- Use `async def` for all route handlers
- All routes except `/health`, `/auth/login`, `/auth/register` require a valid JWT
- Every query must be scoped to the authenticated `user_id` — never query without it
- Never log email addresses, names, or health data — use user IDs only

### Frontend (Next.js)
- App Router (not Pages Router)
- One component per file
- Tailwind only — no external CSS frameworks
- All API calls go through `lib/api.ts` — never call fetch directly in components
- All routes except `/login`, `/register`, `/privacy` require authentication
- Redirect unauthenticated users to `/login`

### Security (apply everywhere)
- No secrets or credentials in code — always use `.env` variables
- Never trust frontend input — validate with Pydantic on every endpoint
- CORS restricted to frontend origin only
- Rate limiting on `/auth/login` and `/auth/register`

### GDPR (apply everywhere)
- No PII in logs
- All user data queries must use `user_id` from the JWT, never from query params
- Account deletion must hard-delete all rows in all user-scoped tables
- Data export must include all tables: sessions, sets, body_weight, profile

### Ergonomy (apply to every UI component)
- Every async action shows a loading state (spinner or skeleton)
- Error messages are human-readable and actionable — never show raw errors or codes
- Confirmation dialog before any destructive action
- Touch targets minimum 44×44px
- All interactive elements have ARIA labels

### Reliability
- Frontend API calls retry up to 3 times with exponential backoff
- Optimistic UI for set logging — update immediately, roll back on failure
- If AI endpoint is unavailable, show last cached advice with a warning

### Idempotency
- `POST /sessions` accepts an `idempotency_key` — replaying returns existing session
- `POST /sets` deduplicates on (session_exercise_id, order)

### Pedagogy
- Every exercise component includes a tooltip: muscles worked, common mistakes, tip
- Every AI recommendation includes a plain-language explanation of the reasoning
- Training jargon (RPE, deload, etc.) always has an inline definition or "?" tooltip

### Data
- All weights in kg
- All durations in seconds
- Dates in ISO 8601 format (YYYY-MM-DD)
- Timestamps in UTC

---

## 5. Build Order (from SPEC.md)

1.  [ ] Project scaffold
2.  [ ] Auth & User profiles (GDPR consent)
3.  [ ] Exercise library (tooltips)
4.  [ ] Session logging
5.  [ ] Session live page + timer + offline
6.  [ ] History & calendar
7.  [ ] Progress charts
8.  [ ] Motivational alerts
9.  [ ] AI advice page
10. [ ] PWA config
11. [ ] Nutrition advice