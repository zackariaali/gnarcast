# Gnarcast PRD — Coherence & Gap Review

> **Review Date:** 2026-04-19
> **Scope:** Full read of `00_Master_PRD.md` through `08_Admin_Console.md`
> **Purpose:** Identify contradictions, gaps, missing specs, and reorganization opportunities before implementation begins.

---

## Top-Level Assessment

The PRD is in strong shape. The product vision is clear, the key primitive (the Scout) is well-defined and used consistently, and the major engineering decisions (Supabase, Next.js, Python scrapers, Twilio, Tomorrow.io) are locked with good rationale. A solo developer could sit down with this and start scaffolding.

That said, there are **three classes of polish work** to do before coding:

1. **Direct contradictions between specs** — a small number, but they'd cause real bugs if coded against. Must fix.
2. **Content gaps within existing specs** — concepts referenced but never defined (home address onboarding, time zones, testing strategy, audit logs, etc.).
3. **Missing specs entirely** — the queued 09/10/11 are the biggest three, but there are at least 4–5 more that should exist before implementation.

I'd estimate 2–4 interview sessions and one tidy-up pass closes all of this.

---

## 1. Direct Contradictions Between Specs

These are the highest-priority fixes — they're not ambiguity, they're conflicting statements.

### 1.1 Weather API primary vs. fallback (CONFLICT)

- `04_Weather_Data.md` says: *"Open-Meteo — ... Strongly preferred for cost and reliability."*
- `06_Tech_Stack.md` says: *"Tomorrow.io (primary) + Open-Meteo (fallback)"*

These directly contradict each other. The tech stack spec is more recent and was locked as part of the stack decision, so 04 needs to be rewritten to match — Tomorrow.io primary, Open-Meteo fallback. (Session state already flagged this as part of Step 1 housekeeping.)

### 1.2 Resort scrape cadence (CONFLICT)

- `04_Weather_Data.md` table: Mountain open %, lift status, grooming → "Daily 6–7am"
- `05_Mountain_Status.md` table: Lift status → "Every 2–4 hrs in season"; terrain/grooming → "Daily"
- `06_Tech_Stack.md`: `scrape_resort_status` → "Every 2–4 hrs (in season)"
- `08_Admin_Console.md`: same as 06

The consistent answer in the three newer specs is "every 2–4 hours in-season." 04 needs to be updated to match. Also, inside 05 itself, lift status is listed at 2–4 hrs but terrain % and grooming are listed at daily — this is plausible but should be stated explicitly as "lift status polls more often than grooming because grooming only changes once a day."

### 1.3 Scout templates vs. Popular Setups — conceptual overlap, unclear relationship

- `02_User_Preferences.md` describes "Pre-Defined Scout Templates" (Powder Hound, Groomer Morning, etc.) as `[BACKLOG]`.
- `07_Sharing_Social.md` describes "Popular Scout Setups" (Pow Day, Groomers + Blue Bird, Park Day, etc.) as `[LOCKED]`.
- `03_Look_Feel.md` uses a third vocabulary (FIRST TRACKS, STORM CHASER, POW HUNT, BLUEBIRD) for the logged-out landing page.

These are three different answers to "what should a new user see as starting points?" They can all coexist, but the relationship and vocabulary need a single owner. Suggested resolution: pick canonical archetype names, use them everywhere, and make explicit:
- *Templates* = hand-curated seed data that exists from Day 1, shown to users when Popular Setups doesn't have enough data.
- *Popular Setups* = data-derived archetypes, shown once a mountain has ≥N active Scouts.
- *Logged-out examples* = illustrative only; should use the same names as Templates to avoid vocabulary sprawl.

### 1.4 Alert fire condition — road closure override missing from core formula

- `02_User_Preferences.md`: *"Alert fires when: Score ≥ trigger threshold AND no Dealbreakers are failed"*
- `05_Mountain_Status.md`: *"Road closure is the only signal that bypasses the scoring system entirely — it triggers an override warning regardless of score"*

These are compatible but 02's core algorithm statement is incomplete. The authoritative fire condition should include the override case explicitly: "Alert fires when: Score ≥ trigger threshold AND no Dealbreakers are failed. If fired, a road-closure override warning is prepended when applicable."

### 1.5 Status header format inconsistency

- 01, 02: "In Progress" (stale — should be Locked per session state)
- 03, 04: "Locked" (title case)
- 05–08: "LOCKED (with open questions)" (all caps)

Purely cosmetic, but worth picking one format for the whole PRD. The all-caps "LOCKED (with open questions)" is more useful because it distinguishes fully-resolved from mostly-resolved specs.

### 1.6 Sub-spec header counts

- 05 header: "Sub-spec 05 of 07"
- 06 header: "Sub-spec 06 of 07"
- 07 header: "Sub-spec 07 of 08"
- 08 header: "Sub-spec 08 of 08"

These were accurate when each spec was written but are now stale. Fix to "of 11" once 09/10/11 exist (or higher, given the missing-specs list below).

---

## 2. Content Gaps Within Existing Specs

Things mentioned in passing that have no authoritative spec anywhere.

### `00_Master_PRD.md`
- **No success metrics / KPIs.** What defines Gnarcast as working? Number of alerts? Alert-to-trip conversion? User retention week-2? No target metrics are written anywhere in the PRD. Without these, "we're ready to code" has no definition.
- **No timeline or phasing.** The resort-directory rollout has phases, but there's no launch date, beta target, or v1 scope line.
- **No risks/assumptions register.** E.g., the entire product assumes resort websites can be scraped reliably enough to deliver accurate alerts — that's a load-bearing assumption and a real risk (ToS, site changes, legal) that should be captured.

### `01_Auth_Signup.md`
- **Home address onboarding step is missing.** 05 says users "set a home address once in their profile," but 01's onboarding flow has no step for it. Session state flags this — must resolve in 10_User_Profile.
- **Phone loss / recovery flow is unspecced.** Passwordless auth + SMS OTP means "I lost my phone and my number changed" is a critical path that doesn't exist in the spec.
- **Phone number change flow unspecced** — a user moves carriers and their number changes. What happens to TCPA consent records?
- **Email verification: blocking vs. async** is still `[OPEN]` from day 1.
- **Rate limiting on OTP requests** — not mentioned. Attackers can flood a phone number with OTP requests; need a per-number send rate limit.

### `02_User_Preferences.md`
- **Scout upper bound.** Says "unlimited" but a user with 500 Scouts would generate an alert-throttling and compute-cost nightmare. Probably fine as a soft cap (e.g., 50 Scouts) with a visible count but no hard block initially.
- **Mountain-in-multiple-Scouts evaluation rule.** If 5 Scouts include Mammoth with different thresholds, all 5 are evaluated independently and all 5 can fire — is that intended? The spec implies yes, but it should say so explicitly because it affects the notification frequency discussion in 09.
- **Intensity multiplier formula** — still `[OPEN]`. Linear vs. exponential changes user experience materially.
- **"Forecast-aware Dealbreakers"** example is good but there's no general rule for how every condition handles forecast vs. historical. Each condition should have a defined "evaluation window."

### `03_Look_Feel.md`
- **No dark mode behavior spec.** Cards have dark variant but no system-level dark mode rule.
- **No responsive breakpoints specified.** Just "mobile web stacks vertically" — what width triggers the shift?
- **No accessibility commitment.** WCAG 2.1 AA as target? Keyboard navigation for card grid? Reduced-motion preference for the video background loop?
- **Video background performance on slow connections** — fallback poster image? Does it autoplay on cellular?
- **Empty state for zero-Scout user.** What does a freshly-onboarded dashboard look like before any scout has data?

### `04_Weather_Data.md`
- **Scraper change-detection cadence.** Spec says scrapers will break and we must detect failures, but no cadence is defined for "how long is too long without an update before we mark this resort degraded?"
- **Legal review of scraping per resort.** Spec says "if a resort blocks us, we explore partnerships" — but doesn't address the case where a resort's ToS explicitly forbids scraping from day one. This is a real legal risk for 163 resorts.
- **Historical data format / queryability.** "All data retained" doesn't say in what form. A 5-year-old weather reading stored as raw JSON is cheap; queryable analytics on it is a different system.

### `05_Mountain_Status.md`
- **Time zone handling is not defined.** Crowd prediction cares about "Saturday" from the user's perspective, grooming reports are local to the resort, alert send times (see 09) are local to the user. Which TZ is canonical per record?
- **Avalanche danger is not in the scoring table.** It's listed as a signal but the 05 tier/weight table doesn't show how it integrates into Scout scoring. Is it a Dealbreaker-eligible signal? Is "extreme" danger an auto-veto?
- **Named-lift configuration UI** is `[OPEN]` — this has implications for the Scout config modal design in 03.
- **Grooming report granularity** — is "grooming report posted" enough to satisfy the condition, or do users want to match on specific groomed runs?

### `06_Tech_Stack.md`
- **No testing strategy.** Unit tests? Integration tests for scrapers? E2E against staging? This is the single biggest gap for an engineering-ready spec.
- **No CI/CD pipeline.** GitHub Actions? Pre-commit hooks? Deploy on green-main?
- **No secret management story.** `.env` files? Doppler? GitHub Secrets? This matters for a multi-API product.
- **No security practices beyond RLS.** Rate limiting on public endpoints, CORS policy, CSP headers, auth session expiry, logging PII considerations.
- **No observability beyond Sentry + PostHog.** Structured logging format? Tracing for scraper-to-score flow?
- **No database schema / ERD.** Tables mentioned (users, scouts, conditions snapshots, user_roles, job_logs) but no canonical schema exists. This is needed before writing the first migration.
- **No cost projections.** Free tiers are cited but an actual monthly budget at 1K users / 10K users / 100K users doesn't exist.

### `07_Sharing_Social.md`
- **Abuse prevention.** A user could blast shared links or Scout setups to spam targets. No rate-limits on sharing.
- **Link analytics.** Are shared links tracked for viral coefficient / conversion? Currently no.
- **Attribution on signup.** When a non-user signs up via a shared Scout, does the sharer get any feedback or credit? Possibly a referral system later, but the data capture for it should be designed now.
- **Link revocation UX.** Spec says owner can revoke — where's that UI live?

### `08_Admin_Console.md`
- **Audit log is missing.** No record of "who granted admin role to whom at what time," "who disabled this resort," "who deleted this user account." Critical for a multi-admin system.
- **No 2FA requirement for destructive admin actions.** Role grants and account deletions probably warrant it.
- **Data export for end-users (GDPR/CCPA).** Admin can delete accounts but there's no spec for exporting a user's data — a legal requirement in many jurisdictions.
- **Customer support tooling.** Where does Zack reply to user support emails / SMS HELP messages? Nothing specced.

---

## 3. Missing Specs (Beyond 09/10/11)

The session state has three missing specs queued. I'd add at minimum these:

### High priority (should exist before coding starts)

**`12_Data_Model.md`** — Concrete database schema. Tables, columns, relationships, indexes, RLS policies. Everything in the stack depends on this being right; retrofitting schema is painful. This should be derivable from the existing specs, so the interview is short.

**`13_API_Design.md`** — REST/Edge-Function contract between frontend and backend. Twilio webhook endpoints. Sharing link (gnar.st) endpoint contracts. SMS reply handler (GOING/SKIP/PAUSE/MORE). This is the handshake layer and it's currently undefined.

**`14_Security_Compliance.md`** — Consolidated home for TCPA, CAN-SPAM, GDPR/CCPA, scraping legality, rate limiting, secret management, threat model. Currently TCPA lives in 01, CAN-SPAM is implied, GDPR is not mentioned, scraping legality is a sentence in 04. Legal risk warrants its own spec.

### Medium priority (can exist alongside or shortly after initial coding)

**`15_Testing_QA.md`** — Test strategy for Next.js app, Celery scrapers, Scout scoring engine, SMS delivery. Manual QA checklist for beta phase. Scraper regression detection (since resort sites change).

**`16_LLM_Strategy.md`** — The product uses LLMs in at least four places (Scout naming, LLM summary, bidirectional summary editing, popular setup naming, newly-opened-terrain parsing). No single spec names the model, prompts, cost ceiling, fallback behavior, or hallucination mitigation.

**`17_Analytics_Metrics.md`** — What PostHog tracks, what dashboards exist, definitions for KPIs referenced in marketing (alert-to-trip conversion, alert accuracy, Scout engagement). Tied to the KPI gap in 00.

### Lower priority (useful but not blocking)

**`18_Launch_Rollout.md`** — Phase 1 → Phase 3 resort gating, beta invite strategy, public launch plan, marketing. Some of this is embedded in 04 and 00 but deserves its own home.

**`19_Accessibility_i18n.md`** — WCAG target, reduced-motion handling, keyboard navigation, future language support (Canadian French at minimum given Whistler/Blackcomb references).

**`20_Resort_Onboarding_Playbook.md`** — Ops runbook for adding a new resort: scraper config, route pre-mapping, signal validation, minimum-threshold gate. Hybrid PRD/internal-doc.

---

## 4. Reorganization Opportunities

Content that exists but would be easier to maintain if restructured.

### 4.1 Create `0X_Concepts_Glossary.md` (or fold into 00)

Scout, Alert, Condition, Signal, Resort, Phase, Tier, Region, Archetype, Template, Setup are used throughout but never defined in one place. A glossary lets future specs reference the canonical term instead of re-explaining. Particularly important because several of these terms have overlapping definitions (Template vs. Setup, Condition vs. Signal).

### 4.2 Consolidate the full conditions catalog

The full conditions list appears in 02 (the canonical list) and is partially repeated in 04 (weather-source table) and 05 (mountain-status table). Three places to keep in sync. Suggested: 02 keeps the canonical list as an appendix or table, and 04/05 reference it by name only — not by re-listing the same signals.

### 4.3 Consolidate the data-refresh cadence table

The same cadence information appears in 04, 05, 06, and 08. Pick one home (I'd say 04, since it's the data spec) and make the others reference it.

### 4.4 Consolidate the backlog

Currently every spec has its own backlog section AND 00_Master_PRD has a master backlog. Items are duplicated (e.g., "tiered conditions rollout" lives in both 00 and 02). Pick one model:
- *Model A:* One master backlog in 00; sub-specs link upward.
- *Model B:* Each sub-spec owns its own backlog; 00 just summarizes with links.

Either is fine — current state (both) is the problem.

### 4.5 Move resort-ops content from 04 and 05 into 08

Both 04 and 05 have "Internal Resort Operations Dashboard" sections marked `[BACKLOG]`, but that backlog is now 08's actual spec. The sections in 04 and 05 should shrink to one-line pointers: *"Admin-only; fully specced in 08_Admin_Console.md."*

### 4.6 Promote "tiered conditions" from backlog to its own spec

Conditions tiering is referenced in 00, 02, 04, and 05. It's load-bearing for the onboarding UX ("don't overwhelm users with 30 options") and for the resort-data-profile system ("show only conditions we can track for this resort"). Currently parked in backlog across multiple specs. This deserves a real spec before 02 and 04 can be fully locked — maybe `0X_Conditions_Tiering.md`.

### 4.7 Move the onboarding flow detail from 01 into 10_User_Profile

The 7-step onboarding flow in 01 is really two concepts: (a) authentication, and (b) first-run profile setup. Home address, condition preferences, and notification preferences all belong in the user-profile domain. Propose shrinking 01's onboarding section to steps 1–3 (phone, email, username) and letting 10 own steps 4–7 (mountain selection, pass, conditions, notifications).

---

## 5. Recommended Order of Operations

Here's how I'd sequence the polish work before coding starts:

1. **Step 1 housekeeping** (already queued in SESSION_STATE.md). ~30 min, mechanical.
2. **Fix the five direct contradictions above** (§1.1–§1.5). ~30 min, mechanical.
3. **Interview → write `09_Notifications.md`** (queued). Resolves lookahead-window `[OPEN]`, fills the biggest product gap.
4. **Interview → write `10_User_Profile.md`** (queued). Resolves home-address gap, phone-recovery gap, account deletion gap.
5. **Short decision → write `11_Monetization.md`** (queued). Affects data model.
6. **Write `12_Data_Model.md`.** Mostly mechanical once 09–11 are done — this is where the PRD becomes machine-actionable.
7. **Write `13_API_Design.md` and `14_Security_Compliance.md`.** Can be parallel.
8. **Promote tiered conditions out of backlog** (§4.6) — short spec, unblocks Scout-config UI work.
9. **Reorganization pass** (§4.1–§4.5). One commit, purely structural.
10. **Final review + `15_Testing_QA.md` + `16_LLM_Strategy.md`** before first feature branch.

After step 9, this PRD is implementation-ready. Steps 10+ can happen in parallel with early coding.

---

## 6. Things That Are Already Good

Worth noting so the polish pass doesn't break them:

- The Scout concept is consistent and well-used across every spec.
- Weight tiers (Dealbreaker / Send It / Nice to Have / Yolo / Ignore / Avoid) match between 02 and 05 — same values, same vocabulary.
- Twilio / Supabase / Tomorrow.io / Next.js tech choices are traceable back to stated constraints.
- The data-confidence system in 04 (High/Medium/Low/Missing) is a genuinely strong design and is the right level of honest about uncertainty.
- The "no social feed" scope discipline in 07 is a strong product call and is well-defended.
- The `[WHY]` annotations throughout are excellent — they'll pay for themselves when someone (including future-you) asks "why did we do it this way?" six months from now.

---

*End of review.*
