# Workout App — Product Specification

## 1. What it is

A personal indoor muscle workout tracker, accessible as a Progressive Web App (PWA)
on both laptop and phone. No app store required — installs from the browser.

---

## 2. Tech Stack

| Layer        | Technology                        |
|--------------|-----------------------------------|
| Frontend     | Next.js 14 + React + Tailwind CSS |
| Backend      | FastAPI (Python 3.11+)            |
| Database     | Supabase (PostgreSQL)             |
| AI           | Claude API (anthropic Python SDK) |
| Phone access | PWA (installable from browser)    |

---

## 3. User Stories

### Workout Session
- As a user, I can start a new workout session at any time.
- I pick exercises freely from a pre-loaded library (no fixed program required).
- For each exercise I can log: sets × reps × weight, rest time between sets.
- I can add free-text notes per session (how I felt, energy level, comments).
- I can log my body weight on any given day.

### Session Page (Live Workout)
- A dedicated page guides me through my session in real time.
- One exercise is highlighted at a time (step-by-step mode).
- A linear recap of the full session is always visible below (what I've done, what's next).
- A rest timer starts automatically between sets.
- I tap "Done" to advance to the next set or exercise.

### Exercise Library
- The app ships with a curated library of common gym exercises.
- Each exercise has: name, muscle group(s), image, and a short description.
- I can add custom exercises if needed (future iteration).

### Calendar & History
- A visual calendar shows which days I trained.
- Each trained day is color-coded by muscle group(s) worked.
- Progress charts show weight/reps evolution over time per exercise.
- AI suggests rest days based on recent training volume.
- Motivational alerts appear on non-trained days (encourage consistency).
- Alerts flag overtraining (too many consecutive sessions on the same muscle group).
- Before each session, the app highlights exercises where I could push:
  one more rep, a heavier weight, or a more difficult variation.

### AI Advice
- Recovery tips after hard sessions (stretching, sleep, nutrition).
- Progression advice (when to increase weight, switch exercises, deload).
- Weekly program suggestions based on training history and goals.
- Injury prevention alerts based on volume and frequency patterns.
- Basic food/nutrition advice tailored to workout goals.

---

## 4. Pages

| Page            | Description                                             |
|-----------------|---------------------------------------------------------|
| `/`             | Dashboard — today's summary, motivational alert, quick start |
| `/session`      | Live workout page — step-by-step guide with timer       |
| `/history`      | Calendar view + progress charts                         |
| `/exercises`    | Browse the exercise library                             |
| `/advice`       | AI-generated advice and program suggestions             |
| `/profile`      | Body weight log, personal goals                         |

---

## 5. Data Model (Draft)

```
exercise
  id, name, muscle_groups[], difficulty, image_url, description

session
  id, date, notes, duration_minutes, body_weight_kg

session_exercise
  id, session_id, exercise_id, order

set
  id, session_exercise_id, reps, weight_kg, rest_seconds

body_weight
  id, date, weight_kg
```

---

## 6. Out of Scope (for now)

- User authentication / multi-user support
- Social or sharing features
- Video recording
- Wearable / smartwatch integration
- Paid tiers

---

## 7. Build Order

1. Project scaffold (Next.js + FastAPI + Supabase connection)
2. Exercise library (pre-load data, browse page)
3. Session logging (start session, add exercises, log sets)
4. Session live page (step-by-step + timer)
5. History & calendar view
6. Progress charts
7. Motivational alerts & progression suggestions
8. AI advice page (Claude integration)
9. PWA configuration (installable on phone)
10. Food / nutrition advice (Claude)