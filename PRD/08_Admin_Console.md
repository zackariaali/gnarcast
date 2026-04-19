# Gnarcast PRD — Admin Console
> Sub-spec 08 of 08 | Status: **LOCKED (with open questions)**
> Related specs: `04_Weather_Data.md` (scraper pipeline), `05_Mountain_Status.md` (resort ops dashboard), `06_Tech_Stack.md` (Celery, Supabase roles)

---

## Overview

The Gnarcast Admin Console is an internal web interface for monitoring and managing the platform — scheduled jobs, resort configuration, system health, and user accounts. It lives within the main Next.js application at `/admin` and is protected by role-based authentication.

The admin console is the primary operational tool for keeping the platform healthy: checking that scrapers are running, debugging bad data, toggling resort availability, and understanding how the system is behaving.

[WHY] Building the admin console inside the main Next.js app (rather than as a separate deployment) means the same auth session, same codebase, same deployment pipeline. A separate subdomain can be added later if security isolation becomes a requirement, but the `/admin` route approach is simpler and sufficient for a small team.

---

## Access Control

**Authentication:** Supabase role-based access control. Admin users have an `admin` role assigned in the Supabase `user_roles` table. Any route under `/admin/*` checks for this role server-side — unauthenticated or non-admin requests are redirected to the main app.

**Access methods:**
1. **Admin button in main nav** — visible only to users with the `admin` role when logged in. Appears in the top navigation bar of the main app. Clicking it navigates to `/admin`.
2. **Direct URL** — `/admin` (and all sub-routes) are directly navigable. Supabase session cookie carries over from the main app — no second login required.

**Multi-admin support:**
The role system supports multiple admin users from day one. An `admin` can grant or revoke the admin role for other users. In the future, granular permissions can be layered on top (e.g., a "resort manager" role with access only to resort configuration, no access to user data).

[WHY] Starting with a simple binary admin role keeps implementation straightforward. The role infrastructure is already in Supabase — adding granularity later is a database migration and middleware check, not an architectural change.

---

## Navigation Structure

```
/admin
├── /admin                    → Dashboard (system health overview)
├── /admin/jobs               → Scheduled Jobs
│   └── /admin/jobs/:job-id   → Job detail + logs
├── /admin/resorts            → Resort Management
│   └── /admin/resorts/:id    → Resort detail + signal health
├── /admin/alerts             → Alert History
├── /admin/users              → User Management
└── /admin/settings           → Admin Settings
```

---

## Section 1: Dashboard (Overview)

The landing page of the admin console. A high-level health summary — the first thing to check when something feels off.

**Panels:**
- **System Health** — green/yellow/red status indicators for each external dependency: Tomorrow.io API, Twilio, State DOT APIs (per region), Supabase, Redis
- **Job Health** — summary of scheduled job status across all types: last run time, success/failure count in last 24 hours, any currently running jobs
- **Resort Health** — count of resorts by data confidence: X resorts fully healthy, Y with degraded signals, Z with missing data
- **Recent Alerts Fired** — last 24 hours of Scout alerts sent: count, success rate, any failures
- **Active Users** — count of users with at least one active Scout (proxy for platform engagement)

**Quick actions from dashboard:**
- "Run all scrapers now" — triggers a full one-off refresh cycle across all active resorts
- "View failed jobs" — deep link to Jobs filtered to failures

---

## Section 2: Scheduled Jobs

The primary operational view. Shows the status of every Celery job type — the thing Zack will check most often.

### Job Types Monitored

| Job | Schedule | Description |
|-----|----------|-------------|
| `scrape_resort_status` | Every 2–4 hrs (in season) | Lift count, terrain %, grooming, open/closed per resort |
| `fetch_weather` | Every 1–3 hrs | Tomorrow.io forecast + SNOTEL snowpack |
| `fetch_avalanche` | Daily 6am | avalanche.org danger levels |
| `fetch_road_conditions` | Every 15–30 min | State DOT API chain control + closures |
| `compute_scout_scores` | After each data refresh | Re-score all active Scouts |
| `send_alerts` | After score computation | Fire SMS for Scouts crossing threshold |

### Job List View (`/admin/jobs`)

Each job type displayed as a row with:
- **Job name** and description
- **Status badge**: Idle / Running / Scheduled / Failed
- **Last run**: timestamp + duration + success/failure
- **Next scheduled run**: countdown
- **24-hour history**: sparkline of success/failure over last 24 runs
- **Actions**: "Run now" button to trigger a one-off execution

Filterable by: job type, status (failed only), date range.

### Job Detail View (`/admin/jobs/:job-id`)

Clicking into a specific job run shows:
- **Full execution log** — raw stdout/stderr output from the job, timestamped line by line
- **Input parameters** — which resorts, which date range, which API was called
- **Output summary** — records processed, records updated, records failed
- **Error detail** — stack trace if the job failed, with the specific resort or API call that caused it
- **Retry button** — re-run the exact same job with the same parameters

[WHY] The log output is the most important debugging tool. When a scraper fails on one resort but not others, the per-job log isolates exactly which resort, which page, and which field caused the failure. This saves hours of blind debugging.

### Celery Flower Integration

**Flower** is Celery's open-source real-time monitoring UI. It runs as an additional service in Docker Compose and provides low-level Celery worker visibility (worker status, task queue depth, broker health) that complements the custom admin console job views.

Flower is linked from the Jobs section as "Advanced: Celery Monitor" — it's a power-user tool, not the primary interface.

```yaml
# docker-compose.yml addition
flower:
  image: mher/flower
  ports:
    - "5555:5555"
  environment:
    - CELERY_BROKER_URL=redis://redis:6379/0
  depends_on:
    - redis
```

Flower is admin-only (basic auth in front of it) and not exposed publicly.

---

## Section 3: Resort Management

The resort configuration center. Replaces the manual spreadsheet (`data/gnarcast-resort-directory.xlsx`) as the operational source of truth once the admin console is built.

### Resort List View (`/admin/resorts`)

All 163 resorts displayed in a searchable, sortable table:

| Column | Description |
|--------|-------------|
| Resort name | With region tag |
| Phase | Launch phase (1–4) |
| Tier | Tier 1 / Tier 2 |
| Available | Toggle (on/off) — controls user visibility |
| Data health | Green / Yellow / Red aggregate signal status |
| Last scraped | Timestamp of most recent successful scrape |
| Actions | Edit / View detail |

**Filters:** Region, Phase, Tier, Availability (on/off), Health status (degraded/missing only)

**Bulk actions:** Enable/disable multiple resorts at once (e.g., "Enable all Phase 1 resorts")

### Resort Detail View (`/admin/resorts/:id`)

Full resort configuration and health for a single resort.

**Configuration panel:**
- Resort name, region, phase, tier
- **Availability toggle** — on/off switch. Off = hidden from all users, no scraping, no Scout alerts. On = active in the platform.
- Resort website URL (source for scraping)
- Primary scraper type (Playwright / HTTPX)
- Pre-mapped highway routes (for road conditions DOT API)
- Chain control checkpoint locations
- Data source URLs (snow report page, grooming report page, lift status page)

**Signal health panel:**

Each data signal shown with its current confidence level:

| Signal | Status | Last Updated | Raw Value |
|--------|--------|-------------|-----------|
| Open/Closed | 🟢 High | 2h ago | Open |
| Lift count | 🟢 High | 2h ago | 14/22 |
| Terrain open % | 🟡 Medium | 6h ago | 62% |
| Grooming report | 🟢 High | 5h ago | 8 runs groomed |
| New snow (24hr) | 🟢 High | 1h ago | 8" |
| Road conditions | 🟢 High | 20min ago | Chain control SR-410 |
| Avalanche danger | 🟢 High | 6h ago | Considerable |
| Crowd prediction | 🟢 High | N/A | Algorithmic |

Signal status indicators:
- 🟢 **Green** — data current and confidence High
- 🟡 **Yellow** — data stale or confidence Medium (scrape succeeded but data may be incomplete)
- 🔴 **Red** — scrape failed or confidence Low/Missing

**Scraper debug panel:**
- "Run scraper now" — triggers a one-off `scrape_resort_status` job for this resort only
- "View last scrape log" — full log output from the most recent scrape
- "View raw HTML" — the last fetched page source (useful for diagnosing layout changes that break the scraper)
- Scrape history table — last 10 scrape attempts with status and duration

[WHY] Resort websites change their layouts without notice. The raw HTML view and scrape log are the two tools needed to diagnose and fix a broken scraper quickly — without them, debugging requires SSH-ing into the server and reading log files manually.

---

## Section 4: Alert History

A log of every Scout alert sent through the platform.

**List view:**
- Timestamp, user (anonymized or by ID), Scout name, resort, score that triggered, alert type (SMS)
- Delivery status: Delivered / Failed / Pending
- Filter by: date range, resort, delivery status, score range

**Detail view (per alert):**
- Full Scout configuration snapshot at time of alert
- Conditions data that produced the score (breakdown of each signal's contribution)
- Raw SMS message sent
- Twilio delivery receipt

[WHY] Alert history serves two purposes: debugging (why did this alert fire / why didn't it fire?) and future product development (which Scouts fire most often, which conditions drive alerts — this feeds algorithm improvement work).

---

## Section 5: User Management

Basic user oversight. Intentionally limited at launch — this is an ops tool, not a CRM.

**List view:**
- User ID, phone number (last 4 digits only for privacy), signup date, number of active Scouts, last active
- Account status: Active / Suspended
- Filter by: signup date, account status, Scout count

**User detail:**
- Account info (masked phone, signup date, TCPA consent timestamp)
- Scout list (name, mountains, active/paused)
- Alert history (count + last alert)
- Actions: Suspend account, Delete account (with confirmation)

**Admin role management:**
- Grant / revoke admin role for any user
- Admin user list — who currently has admin access

[WHY] User data is kept minimal in the admin console on purpose — phone numbers are masked, no browsing history, no Scout condition details. Admin access is for operational management, not surveillance of user behavior.

---

## Section 6: Admin Settings

Platform-wide configuration:

- **Season mode** — toggle between in-season (full scrape cadence) and off-season (reduced/paused scraping). Off-season reduces API costs when mountains are closed.
- **Alert throttle** — global rate limit on outbound SMS (e.g., max X alerts per minute to protect Twilio throughput)
- **Maintenance mode** — disable Scout alerts platform-wide (e.g., during a deployment or data issue)
- **Admin users** — manage who has admin access
- **API key status** — display (masked) current keys for Tomorrow.io, Twilio, DOT APIs with last-validated timestamp

---

## Technical Implementation

**Route protection:**
All `/admin/*` routes use a Next.js middleware that checks the Supabase session for the `admin` role. Non-admin users are redirected to `/` with no error message (security through obscurity — don't advertise the admin console exists).

```typescript
// middleware.ts
if (pathname.startsWith('/admin')) {
  const { data: { session } } = await supabase.auth.getSession()
  const isAdmin = session?.user?.app_metadata?.role === 'admin'
  if (!isAdmin) return NextResponse.redirect(new URL('/', request.url))
}
```

**Admin nav button:**
Shown in the main app top navigation only when `user.app_metadata.role === 'admin'`. Renders as a subtle "Admin" link — not a prominent CTA.

**Job triggering:**
"Run now" buttons call a Supabase Edge Function that enqueues the job in Celery via the Redis broker. This keeps the Next.js app stateless — it never talks directly to Celery, only through the shared Redis queue.

**Log streaming:**
Job logs are written to a `job_logs` table in PostgreSQL by the Celery worker. The admin console reads logs via Supabase Realtime subscription — logs stream in live as a job runs, no polling required.

---

## Open Questions

- `[OPEN]` Should the admin console be at `/admin` (same domain) or `admin.gnarcast.com` (subdomain)? Currently `/admin` — subdomain adds security isolation but more deployment complexity.
- `[OPEN]` Granular role permissions (e.g., "resort manager" role) — what specific actions should be permissioned separately when a second admin is added?
- `[OPEN]` Raw HTML storage for scraper debugging — how long to retain? Storage cost vs. debugging value tradeoff.

---

## Backlog

- `[BACKLOG]` Granular admin roles — "resort manager" (resort config only), "support" (user management only), "super admin" (everything)
- `[BACKLOG]` Email/SMS alerts to admin when a scraper fails or a resort goes red — proactive ops notifications
- `[BACKLOG]` Data diff view — show what changed between two scrape runs for a resort (useful for catching scraper regressions)
- `[BACKLOG]` A/B testing controls — toggle experimental Scout scoring models for a subset of users
- `[BACKLOG]` Revenue / subscription dashboard — when monetization is added
