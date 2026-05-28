# CLAUDE.md — Workout App

## 1. What this project is

A multi-user, freemium indoor muscle workout tracker.
Distributed as a **Progressive Web App** — responsive on mobile and desktop, installable from the browser.
No native app stores for now.
Full specification in `SPEC.md`.
Living build plan in `PLAN.md`.
Backlog in `BACKLOG.md`.

---

## 2. Repository Layout

```
frontend/           # Next.js 14 + React + Tailwind CSS (PWA)
backend/            # FastAPI (Python 3.11+)
SPEC.md             # Product specification — source of truth
PLAN.md             # Living progress tracker — updated every session
BACKLOG.md          # Feature ideas to implement later
```

---

## 3. Tech Stack

| Layer        | Technology                                   |
|--------------|----------------------------------------------|
| Frontend     | Next.js 14 (App Router), React, Tailwind CSS |
| PWA          | next-pwa (service worker, installable)       |
| Backend      | FastAPI (Python 3.11+)                       |
| Auth         | Supabase Auth (JWT, email/password)          |
| Database     | Supabase (PostgreSQL + Row-Level Security)   |
| AI           | Claude API via `anthropic` Python SDK        |
| Payments     | Stripe                                       |

---

## 4. Key Conventions

### Backend (FastAPI)
- Python 3.11+
- One router per domain: `auth`, `exercises`, `sessions`, `sets`, `progress`,
  `advice`, `profile`, `subscription`
- Pydantic models for all request/response schemas — validate everything server-side
- No business logic in route handlers — extract to service functions
- `async def` for all route handlers
- All routes except `/health`, `/auth/login`, `/auth/register` require a valid JWT
- Every DB query scoped to `user_id` from the JWT — never from query params

### Frontend (Next.js)
- App Router only — not Pages Router
- One component per file
- Tailwind only — no external CSS frameworks
- All API calls go through `lib/api.ts`
- All routes except `/login`, `/register`, `/privacy`, `/subscription` require auth
- Redirect unauthenticated users to `/login`

### Premium Feature Gates
- Premium status is **always validated server-side** via the `subscription` table
  (synced by Stripe webhooks). Never trust a `isPremium` flag from the client.
- Backend: a `require_premium` FastAPI dependency raises HTTP 402 if user is not premium.
  Apply it to all premium endpoints (`/progress`, `/advice`, `/sessions` full history).
- Frontend: premium-only UI sections show an upsell card (not just a disabled button).
  The upsell card explains what the feature does and links to `/subscription`.
- Free history gate: `GET /sessions` returns only the last 30 days for free users,
  unlimited for premium. Enforced in the service layer, not just the route.

### Subscription
- Stripe is the single source of truth for subscription status.
- `POST /subscription/webhook/stripe` syncs status to the `subscription` table.
- Never manually activate premium without a verified Stripe webhook event.

### Security
- No secrets in code — all in `.env`. Document every variable in `.env.example`.
- CORS: allowed origins = web domain only.
- Rate limiting on `/auth/login` and `/auth/register`.
- Subscription status checked on every premium endpoint, not cached in the JWT.

### GDPR
- No PII in logs — user IDs only.
- All user data queries use `user_id` from JWT.
- Account deletion hard-deletes all rows in all user-scoped tables.
- Data export (`GET /profile/export`) is free and cannot be gated behind premium.

### Ergonomy
- Every async action shows a loading state.
- Error messages are human-readable and actionable.
- Confirmation dialog before any destructive action.
- Touch targets minimum 44×44px.
- All interactive elements have ARIA labels.
- Smooth scrolling and touch-friendly interactions on mobile browsers.

### Reliability
- Frontend API calls retry up to 3 times with exponential backoff.
- Optimistic UI for set logging — roll back on failure.
- AI endpoint unavailable → show last cached advice with a warning banner.

### Idempotency
- `POST /sessions` accepts `idempotency_key`.
- `POST /sets` deduplicates on (session_exercise_id, order).
- Stripe webhook handler is idempotent — replaying the same event is safe.

### Pedagogy
- Every exercise component: tooltip with muscles worked, common mistakes, beginner tip.
- Every AI recommendation: plain-language explanation of the reasoning.
- Training jargon: inline definition on first use, "?" tooltip after.
- Premium upsells: explain the value of the feature, not just that it costs money.

### Data
- All weights in kg
- All durations in seconds
- Dates in ISO 8601 (YYYY-MM-DD)
- Timestamps in UTC

---

## 5. Build Order (from SPEC.md)

1.  [ ] Project scaffold
2.  [ ] Auth & User profiles (GDPR)
3.  [ ] Exercise library (tooltips)
4.  [ ] Session logging (free)
5.  [ ] Session live page + timer + auto-save
6.  [ ] History & calendar (free 30d / premium gate)
7.  [ ] Progress charts (premium)
8.  [ ] Motivational alerts (premium)
9.  [ ] AI advice page (premium)
10. [ ] Freemium: subscription model + Stripe + premium gates
11. [ ] Offline mode (premium — PWA service worker)
12. [ ] Nutrition advice (premium)