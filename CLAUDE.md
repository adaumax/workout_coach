# CLAUDE.md — Workout App

## 1. What this project is

A personal indoor muscle workout tracker.
Full specification is in `SPEC.md` — read it before making any changes.

---

## 2. Repository Layout

```
frontend/        # Next.js 14 + React + Tailwind CSS (PWA)
backend/         # FastAPI (Python 3.11+)
SPEC.md          # Product specification — source of truth
BACKLOG.md       # Feature ideas to implement later
```

---

## 3. Tech Stack

- **Frontend**: Next.js 14, React, Tailwind CSS
- **Backend**: FastAPI (Python), Supabase Python client
- **Database**: Supabase (PostgreSQL)
- **AI**: Claude API via `anthropic` Python SDK
- **Phone**: PWA — no app store

---

## 4. Key Conventions

### Backend (FastAPI)
- Python 3.11+
- One router per domain (exercises, sessions, sets, advice)
- Pydantic models for all request/response schemas
- No business logic in route handlers — keep them thin
- Use `async def` for all route handlers

### Frontend (Next.js)
- App Router (not Pages Router)
- One component per file
- Tailwind only — no external CSS frameworks
- All API calls go through `lib/api.ts`

### Data
- All weights in kg
- All durations in seconds
- Dates in ISO 8601 format (YYYY-MM-DD)

---

## 5. Build Order (from SPEC.md)

1. [ ] Project scaffold
2. [ ] Exercise library
3. [ ] Session logging
4. [ ] Session live page + timer
5. [ ] History & calendar
6. [ ] Progress charts
7. [ ] Motivational alerts
8. [ ] AI advice page
9. [ ] PWA config
10. [ ] Nutrition advice