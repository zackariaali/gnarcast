# Gnarcast PRD — Tech Stack
> Sub-spec 06 of 07 | Status: **LOCKED (with open questions)**
> Related specs: `01_Auth_Signup.md` (auth), `04_Weather_Data.md` (data pipeline), `05_Mountain_Status.md` (road/access APIs)

---

## Overview

Gnarcast is built as a containerized web application from day one — running locally on a Linux development machine via Docker Compose, with a clear and low-friction path to cloud deployment. The stack prioritizes developer velocity for a small team, sound architecture that won't need to be rewritten at scale, and performance characteristics appropriate for a real-time conditions platform.

[WHY] The project starts as a solo build with potential to bring on team members. The stack choices favor widely-understood tools with large hiring pools, strong documentation, and indie-friendly hosting that can migrate to enterprise infrastructure if needed.

---

## Decision: Auth Provider

**LOCKED: Supabase Auth (not Clerk)**

[WHY] Supabase Auth is the right call because Gnarcast needs both authentication *and* a database — and Supabase provides both in a single integrated platform. Clerk is auth-only, requiring a separate database vendor, additional integration surface area, and two pricing models to manage. Supabase Auth supports phone/SMS OTP (via Twilio, already locked in `01_Auth_Signup.md`), OAuth providers, row-level security, and session management — everything the auth spec requires. Migrating from local Supabase to Supabase Cloud requires zero code changes.

---

## Stack Summary

| Layer | Technology | Notes |
|-------|-----------|-------|
| Frontend | Next.js (TypeScript) | App Router, React Server Components |
| Styling | Tailwind CSS | Utility-first, consistent with design system |
| Backend / API | Supabase (PostgreSQL + Edge Functions) | Auth, DB, realtime, storage |
| Auth | Supabase Auth + Twilio | Phone/SMS OTP primary |
| SMS Notifications | Twilio | Already locked |
| Scraping | Python + Playwright + BeautifulSoup | Separate service |
| Job Queue | Celery + Redis | Scheduled scrape + score computation jobs |
| Weather API | Tomorrow.io (primary) + Open-Meteo (fallback) | |
| Road Conditions | State DOT APIs (WSDOT, ODOT, etc.) | No vendor dependency |
| Caching | Redis | Weather API responses, pre-computed scores |
| Video CDN | Cloudflare R2 or Bunny.net | Background video delivery |
| Error Tracking | Sentry | Frontend + backend |
| Analytics | PostHog | Product analytics, self-hostable |
| Local Dev | Docker Compose + Supabase CLI | Full stack runs locally |
| Frontend Deploy | Vercel | Edge network, automatic deploys from GitHub |
| Backend Deploy | Supabase Cloud | One migration command from local |
| Scraper Deploy | Railway or Render | Container deploy from GitHub |
| Cache Deploy | Upstash (serverless Redis) | Or Railway managed Redis |

---

## Frontend

**Next.js with TypeScript**

- App Router (Next.js 14+) with React Server Components — server-rendered pages for SEO on the logged-out landing page, client components for interactive Scout cards and real-time updates
- TypeScript throughout — catches type errors before runtime, essential for a data-heavy app with complex scoring models
- Tailwind CSS for styling — consistent with the design system in `03_Look_Feel.md`, utility-first keeps CSS manageable for a small team

**Performance design:**
- Scout scores and conditions data are **pre-computed and cached** — page load reads stored results, never runs scoring logic live
- Static/ISR pages served from Vercel's edge network — globally fast
- Background videos served from CDN (Cloudflare R2 or Bunny.net) — never from the app server
- No blocking external API calls on page load

[WHY] Next.js is the dominant React framework with the largest ecosystem, best documentation, and most straightforward path from local development to production. Server components reduce client-side JavaScript and improve initial load time — important for a product where the first impression (the video landing page) needs to feel instant.

---

## Backend & Database

**Supabase (PostgreSQL)**

Supabase provides the full backend layer:
- **PostgreSQL database** — structured relational data for resorts, users, Scouts, scores, conditions snapshots
- **Auth** — phone/SMS OTP + OAuth (see auth spec), session management, TCPA consent records
- **Realtime** — live subscriptions for condition updates (Scout score changes, road closure alerts)
- **Edge Functions** — lightweight serverless functions for webhook handling (Twilio SMS callbacks, etc.)
- **Storage** — user avatars, any binary assets not on CDN
- **Row-level security** — users can only read/write their own Scouts and preferences; enforced at the database layer

**Local development:**
Supabase CLI runs the entire Supabase stack locally via Docker Compose — PostgreSQL, Auth, Realtime, Edge Functions, and the Supabase Studio dashboard. No cloud account needed during development.

**Connection pooling:**
PgBouncer is built into Supabase and handles connection pooling automatically — important when scraper jobs and the web app are hitting the database concurrently.

[WHY] A managed PostgreSQL backend with built-in auth, realtime, and RLS removes a significant amount of infrastructure the team would otherwise need to build and maintain. Supabase's local-to-cloud migration story is seamless — `supabase db push` and `supabase functions deploy` are the only commands needed to go from local to production.

---

## Scraping Infrastructure

**Python + Playwright + Celery**

The scraping service is a **separate Python service** running alongside the Next.js app in Docker Compose. Two languages in the codebase (TypeScript for the app, Python for scrapers) is a common and manageable pattern — the services communicate through the shared Supabase database, not directly.

**Components:**
- **Playwright** — headless browser for JavaScript-heavy resort sites; handles sites that require JS execution to render snow reports
- **BeautifulSoup / HTTPX** — lighter-weight scraping for simpler resort pages and DOT APIs
- **Celery** — distributed task queue for managing scheduled scrape jobs across all resorts
- **Redis** — Celery broker + result backend; also used for caching weather API responses

**Job types:**
| Job | Cadence | Description |
|-----|---------|-------------|
| `scrape_resort_status` | Every 2–4 hrs (in season) | Lift count, terrain %, grooming, open/closed |
| `fetch_weather` | Every 1–3 hrs | Tomorrow.io forecast + SNOTEL snowpack |
| `fetch_avalanche` | Daily (6am) | avalanche.org danger levels |
| `fetch_road_conditions` | Every 15–30 min | State DOT API chain control + closures |
| `compute_scout_scores` | After each data refresh | Re-score all active Scouts with fresh data |
| `send_alerts` | After score computation | Fire SMS for Scouts that crossed their threshold |

**Scraper failure handling:**
- Failed scrapes logged with timestamp and error type
- Retry with exponential backoff (3 attempts before marking as failed)
- Confidence level downgraded to Low on stale data (see `04_Weather_Data.md`)
- Ops dashboard surfaces scraper health per resort (green/yellow/red)

[WHY] Python is the correct language for web scraping — the ecosystem (Playwright, BeautifulSoup, Scrapy, HTTPX) is far more mature than the Node.js equivalents. Celery is the industry standard for Python job queues and handles the complexity of running hundreds of concurrent resort scrapes on a schedule.

---

## Performance Architecture

**The core principle: pre-compute everything, read nothing live.**

```
[Scraper jobs] → [Raw data in PostgreSQL]
                         ↓
              [Score computation job]
                         ↓
              [Scores cached in Redis]
                         ↓
              [Next.js reads cached scores]
                         ↓
              [User sees instant UI]
```

**What this means in practice:**
- When a user opens the app, they're reading pre-computed Scout scores from Redis — no weather API calls, no scoring logic, no database joins on the critical path
- Scout score cache is invalidated and recomputed after every data refresh cycle
- Page load time is essentially a single Redis read + a Supabase query for the user's Scout list
- Real-time score updates use Supabase Realtime subscriptions — the page updates reactively when a new score is written, without polling

**Video delivery:**
- Background MP4s are not served from the app server — they live on Cloudflare R2 or Bunny.net (both ~$0.01/GB, edge-delivered globally)
- Videos are referenced by URL in the app; no video bytes ever touch the Next.js server

**Target performance:**
- Time to Interactive (logged-in, Scout dashboard): < 1.5 seconds
- Time to First Contentful Paint (logged-out landing): < 1 second (edge-cached static page)

---

## iOS App (Future)

The current stack is designed to support a native iOS app without architectural changes.

**Path to iOS:**
- Supabase has a first-class Swift SDK (`supabase-swift`) — the iOS app authenticates against the same Supabase Auth instance and reads from the same database
- The same Twilio integration handles SMS regardless of whether the alert was configured on web or mobile
- The Scout data model is platform-agnostic — iOS just becomes another client reading from the same backend

**React Native** is also worth evaluating when the time comes — shared TypeScript codebase between web and mobile reduces maintenance burden. Given Gnarcast's visual design (video backgrounds, animations), native UIKit/SwiftUI may ultimately feel better, but React Native (particularly with Expo) has significantly closed that gap. Decision deferred to when mobile development begins.

[WHY] Designing the API and data model with iOS in mind from day one avoids the common mistake of building a web-specific backend that requires a rewrite when mobile is added. Supabase's multi-platform SDK support makes this straightforward.

---

## Local Development Setup

Everything runs locally via Docker Compose. The developer experience:

```bash
# Clone repo
git clone https://github.com/zackariaali/gnarcast

# Start full local stack
docker compose up

# Services running locally:
# - Next.js app        → http://localhost:3000
# - Supabase Studio    → http://localhost:54323
# - PostgreSQL         → localhost:54322
# - Redis              → localhost:6379
# - Celery worker      → background
# - Celery beat        → scheduled jobs
```

Supabase CLI manages the local Supabase stack; `supabase start` / `supabase stop` are wrapped into the Docker Compose setup.

---

## Deployment Path

**Phase 1: Local Linux box**
Full stack on Docker Compose. No cloud costs. Good for development and internal testing.

**Phase 2: Cloud (when ready)**
| Service | Local | Cloud |
|---------|-------|-------|
| Next.js | Docker (port 3000) | Vercel (one command) |
| Supabase | Supabase CLI local | Supabase Cloud (`supabase db push`) |
| Scrapers + Celery | Docker Compose | Railway or Render (container deploy) |
| Redis | Docker | Upstash or Railway managed Redis |
| Videos | Local filesystem | Cloudflare R2 or Bunny.net |

No code changes required between local and cloud — environment variables swap out connection strings.

**Future cloud migration (if needed):**
Vercel → AWS (ECS/Fargate), Supabase Cloud → RDS PostgreSQL, Railway → ECS. All viable paths that don't require rewriting application code.

---

## Open Questions

- `[OPEN]` React Native vs. native Swift for iOS — deferred to when mobile development begins
- `[OPEN]` Cloudflare R2 vs. Bunny.net for video CDN — both viable, decision at implementation time
- `[OPEN]` Railway vs. Render for scraper deployment — evaluate pricing/DX at deployment time
- `[OPEN]` PostHog self-hosted vs. PostHog Cloud — self-hosted saves cost but adds ops overhead; probably Cloud to start

---

## Backlog

- `[BACKLOG]` Push notifications — Firebase FCM when iOS app is built
- `[BACKLOG]` GraphQL API layer — if the iOS app needs more flexible querying than REST
- `[BACKLOG]` Edge Functions for Scout score computation — move hot-path scoring to Supabase Edge Functions if latency becomes an issue
- `[BACKLOG]` Load testing — simulate concurrent Scout alerts firing (e.g., after a major storm when thousands of Scouts may cross threshold simultaneously)
- `[BACKLOG]` Database read replicas — if read load from many concurrent users exceeds Supabase single-instance limits
