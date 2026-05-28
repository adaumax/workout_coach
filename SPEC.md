# Workout App — Product Specification

## 1. What it is

A multi-user indoor muscle workout tracker available on three surfaces:
- **Web** (Progressive Web App, runs in any browser)
- **iOS** (App Store, packaged with Capacitor)
- **Android** (Play Store, packaged with Capacitor)

The app follows a **freemium model**: core features are free, advanced features
(AI, full history, charts, offline) require a paid subscription.
Pedagogical by design: every feature teaches the user something about training and health.

---

## 2. Tech Stack

| Layer              | Technology                                      |
|--------------------|-------------------------------------------------|
| Frontend / Web     | Next.js 14 + React + Tailwind CSS               |
| Mobile packaging   | Capacitor (wraps the web app for iOS + Android) |
| Backend            | FastAPI (Python 3.11+)                          |
| Database           | Supabase (PostgreSQL + Row-Level Security)      |
| Auth               | Supabase Auth (JWT, email/password)             |
| AI                 | Claude API (anthropic Python SDK)               |
| In-App Purchases   | RevenueCat (normalizes Apple + Google IAP)      |
| Web payments       | Stripe (for subscriptions via the web)          |

### Why Capacitor?
Capacitor wraps the existing Next.js web app into a native iOS and Android shell.
There is **one codebase** — not two separate mobile apps. The same UI, logic, and
components run everywhere. Capacitor also provides plugins for native APIs
(push notifications, haptics, camera) when needed later.

### Why RevenueCat?
Apple App Store and Google Play each have completely different In-App Purchase APIs.
RevenueCat provides a single SDK and backend that normalizes both, handles receipt
validation, manages entitlements (free vs premium), and integrates with Stripe
for web payments. It is the industry standard for freemium mobile apps.

---

## 3. Freemium Model

### Principle
The free tier must be genuinely useful — a user can track all their workouts for free.
Premium features are for users who want deeper insights, AI guidance, and a fully
offline mobile experience.

### Feature Split

| Feature | Free | Premium |
|---------|------|---------|
| Exercise library (full, with tooltips) | ✅ | ✅ |
| Log unlimited workout sessions | ✅ | ✅ |
| Live session page with rest timer | ✅ | ✅ |
| Session notes and body weight log | ✅ | ✅ |
| Data export (JSON / CSV) | ✅ | ✅ |
| History & calendar (last 30 days) | ✅ | ✅ |
| History & calendar (full, unlimited) | ❌ | ✅ |
| Progress charts + trend summaries | ❌ | ✅ |
| Motivational alerts & progression suggestions | ❌ | ✅ |
| AI recovery advice | ❌ | ✅ |
| AI progression and program advice | ❌ | ✅ |
| AI injury prevention tips | ❌ | ✅ |
| AI nutrition advice | ❌ | ✅ |
| Offline session page (mobile) | ❌ | ✅ |

> **GDPR note:** Data export (Article 20 — right to data portability) is a legal
> obligation and must remain free regardless of subscription status.

### Pricing (indicative — to be confirmed before launch)
- **Free**: always free
- **Premium monthly**: ~4.99 €/month
- **Premium yearly**: ~39.99 €/year (~33% discount)

### Store rules
- Apple App Store takes **30%** of IAP revenue (15% after year 1 for small developers).
- Google Play takes **30%** (15% after year 1).
- Apple **requires** IAP for any digital goods sold inside an iOS app. You cannot
  link to an external payment page from inside the iOS app.
- Apple **requires** "Sign in with Apple" if any third-party OAuth is offered.
  Since we use email/password only, this rule does not apply for now.
- A **Privacy Policy URL** is mandatory for App Store and Play Store submission.
  Our `/privacy` page satisfies this requirement.

---

## 4. Non-Functional Requirements

These apply to every feature. Every piece of code must satisfy them.

### 4.1 GDPR Compliance

- **Explicit consent**: Users must accept a clear privacy notice on signup.
  Consent is timestamped (`gdpr_consent_at`).
- **Data minimization**: Collect only what is strictly necessary.
- **Right to erasure**: Users can permanently delete their account and all data.
  This triggers a hard-delete cascade. Available to all users (free and premium).
- **Data portability**: Users can export all their data as JSON or CSV.
  Available to all users regardless of subscription (legal obligation).
- **No PII in logs**: Logs use user IDs only. No emails, names, or health data.
- **Privacy policy page**: `/privacy` explains data collected, retention, and rights.

### 4.2 Security

- All routes except `/health`, `/auth/login`, `/auth/register`, `/privacy` require a valid JWT.
- JWT issued by Supabase Auth, validated on every backend request.
- Every DB query scoped to authenticated `user_id` — enforced by Supabase RLS.
- All inputs validated server-side with Pydantic. Never trust the frontend.
- Passwords never stored — Supabase Auth handles hashing.
- Rate limiting on auth endpoints (login, register).
- CORS restricted to known origins (web domain + Capacitor app origin).
- No secrets in code — all configuration via `.env` files. `.env.example` documents all variables.
- Subscription status validated server-side via RevenueCat webhook — never trust client claims.
- IAP receipts validated by RevenueCat backend, not by the app itself.

### 4.3 Ergonomy & Accessibility

- Fully responsive: usable on 375px mobile and 1440px desktop.
- WCAG 2.1 AA: sufficient contrast, keyboard navigation, ARIA labels, visible focus.
- Touch targets minimum 44×44px on mobile.
- Loading states on every async action (spinner or skeleton).
- Human-readable, actionable error messages — never raw error codes.
- Confirmation dialogs before any destructive action.
- Native mobile feel on iOS and Android via Capacitor (smooth scrolling, haptic feedback on key actions).

### 4.4 Reliability & Failovers

- **Optimistic UI**: set logging updates immediately, rolls back on failure.
- **Offline support** (premium): session page works offline, sets queued and synced on reconnect.
- **Retry logic**: all frontend API calls retry up to 3 times with exponential backoff.
- **Graceful degradation**: AI unavailable → show last cached advice with a warning.
- **Health endpoint**: `GET /health` returns backend and database status.

### 4.5 Idempotency

- `POST /sessions` accepts an `idempotency_key` — replaying returns the existing session.
- `POST /sets` deduplicates on (session_exercise_id, order).
- `POST /subscription/verify` is idempotent — replaying the same receipt is safe.

### 4.6 Pedagogy

- Tooltips on every exercise: muscles worked, common mistakes, beginner tip.
- Every AI recommendation includes a plain-language explanation of the reasoning.
- Training jargon always has an inline definition or "?" tooltip.
- Onboarding flow for new users.
- Progress charts include plain-language summaries.
- Premium upsell explains **why** a feature requires premium (not just a paywall).

---

## 5. User Stories

### Auth & Profile
- Register with email + password + GDPR consent.
- Log in; data is available across web, iOS, and Android.
- Update profile: username, goal, avatar.
- Export all personal data (free).
- Delete account and all data permanently (free).
- Manage subscription: upgrade, cancel, view current plan.

### Subscription
- See which features are free and which are premium, with explanations.
- Subscribe via the web (Stripe) or in the app (Apple/Google IAP).
- Premium unlocks instantly after payment on all devices.
- Cancellation takes effect at end of billing period; data is never deleted.

### Workout Session (free)
- Start a session, pick exercises from the library.
- Log sets × reps × weight, rest time, notes.
- Session live page with step-by-step guide and rest timer.

### Exercise Library (free)
- Browse exercises with muscle group and difficulty filters.
- Each exercise shows: name, image, muscles worked, common mistakes, beginner tip.

### History (free: 30 days / premium: unlimited)
- Calendar view of trained days, color-coded by muscle group.
- Session detail on tap.

### Progress Charts (premium)
- Weight and volume progression per exercise.
- Plain-language trend summary.

### Motivational Alerts & Progression Suggestions (premium)
- Dashboard alerts for inactivity and overtraining.
- Pre-session progression suggestions with explanations.

### AI Advice (premium)
- Recovery, progression, program, injury prevention, and nutrition advice.
- Every recommendation includes plain-language reasoning.

### Offline (premium — mobile only)
- Session page works without internet.
- Sets queued and synced when connection returns.

---

## 6. Pages

| Page              | Free/Premium | Description |
|-------------------|-------------|-------------|
| `/`               | Free        | Dashboard — summary, alerts (premium), quick start |
| `/login`          | Free        | Login form |
| `/register`       | Free        | Registration + GDPR consent |
| `/session`        | Free        | Live workout — timer, step-by-step |
| `/history`        | Free/Premium| Calendar (30d free, unlimited premium) |
| `/exercises`      | Free        | Exercise library with tooltips |
| `/progress`       | Premium     | Progress charts + summaries |
| `/advice`         | Premium     | AI advice page |
| `/subscription`   | Free        | Pricing, plan management |
| `/profile`        | Free        | Goals, data export, account deletion |
| `/privacy`        | Free        | Privacy policy (required for stores) |

---

## 7. Data Model

```
profile  (extends Supabase Auth user)
  id (= auth.user.id), username, goal, avatar_url,
  gdpr_consent_at, created_at

subscription
  id, user_id, plan (free | premium),
  provider (apple | google | stripe | manual),
  provider_subscription_id, status (active | cancelled | expired),
  current_period_end, created_at, updated_at

exercise  (shared — not user-scoped)
  id, name, muscle_groups[], difficulty, image_url, description,
  muscles_worked_diagram_url, common_mistakes, beginner_tip

session  (user-scoped)
  id, user_id, date, notes, duration_seconds, body_weight_kg,
  idempotency_key, completed_at

session_exercise
  id, session_id, exercise_id, order

set
  id, session_exercise_id, reps, weight_kg, rest_seconds, logged_at

body_weight  (user-scoped)
  id, user_id, date, weight_kg
```

Row-Level Security enabled on all user-scoped tables.
`subscription` status is the source of truth for feature access — always validated
server-side via RevenueCat, never trusted from the client.

---

## 8. API Endpoints (planned)

### Auth
```
POST /auth/register
POST /auth/login
POST /auth/logout
```

### Profile
```
GET    /profile
PATCH  /profile
GET    /profile/export          (free — GDPR)
DELETE /profile                 (free — GDPR)
```

### Subscription
```
GET    /subscription/status                  → current plan + expiry
POST   /subscription/verify                 → validate Apple/Google IAP receipt
POST   /subscription/webhook/revenuecat     → RevenueCat webhook (server-side sync)
POST   /subscription/webhook/stripe         → Stripe webhook (web payments)
```

### Exercises
```
GET /exercises
GET /exercises/{id}
```

### Sessions
```
POST   /sessions                            (idempotency_key)
GET    /sessions                            (paginated; premium = unlimited, free = 30d)
GET    /sessions/{id}
PATCH  /sessions/{id}
DELETE /sessions/{id}
POST   /sessions/{id}/exercises
POST   /sessions/{id}/exercises/{ex_id}/sets
```

### Progress (premium)
```
GET /progress/exercises/{id}    → weight/volume over time
GET /progress/summary           → overall trend summary
```

### Advice (premium)
```
POST /advice                    → generate AI advice (cached)
POST /advice/nutrition          → generate nutrition advice (cached)
```

### Health
```
GET /health
```

---

## 9. Out of Scope (for now)

- Social or sharing features
- Video recording
- Wearable / smartwatch integration
- OAuth login (Google, Apple) — email/password only for now
- Web3 / crypto payments
- Team / coach accounts

---

## 10. Build Order

1.  Project scaffold (Next.js + FastAPI + Supabase + env setup)
2.  Auth & User profiles (register, login, GDPR consent, profile, privacy page)
3.  Exercise library (pre-loaded data, browse page, pedagogical tooltips)
4.  Session logging (start session, add exercises, log sets — all free)
5.  Session live page (step-by-step + timer + auto-save)
6.  History & calendar (free: 30 days / premium gate: unlimited)
7.  Progress charts (premium — with plain-language summaries)
8.  Motivational alerts & progression suggestions (premium)
9.  AI advice page (premium — Claude integration with explanations)
10. Freemium: subscription model, RevenueCat setup, premium gates, `/subscription` page
11. Stripe integration (web payments)
12. Offline mode (premium — PWA service worker + sync queue)
13. Capacitor: package as iOS + Android app
14. App Store & Play Store submission
15. Nutrition advice (premium — Claude)