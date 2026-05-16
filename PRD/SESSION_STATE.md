# Gnarcast — Session State

> **Last Updated:** 2026-05-16 (after Step 1 housekeeping pass — all 5 contradictions resolved, stale references cleaned up)
> **Purpose:** Quick-resume file for continuing PRD interviews across sessions.
> To resume: *"Read SESSION_STATE.md in my Gnarcast PRD folder and let's pick up where we left off."*

---

## How to Use This File

At the start of a new session, ask Claude to read this file. It contains everything needed to resume PRD work without repeating context. All detailed decisions are in the sub-specs — this file is just the map.

**Key companion document:** `PRD_REVIEW_2026-04-19.md` — a full coherence/gap review of all 8 specs. The work queued below is derived from that review. If you're picking this up cold, read the review doc first, then this file.

---

## Current Status

| Section | Status | File |
|---------|--------|------|
| Vision & Mission | ✅ Locked | `00_Master_PRD.md` |
| Auth & Signup | ✅ Locked (with open questions) | `01_Auth_Signup.md` |
| User Preferences | ✅ Locked (with open questions) | `02_User_Preferences.md` |
| Look & Feel | ✅ Locked (with open questions) | `03_Look_Feel.md` |
| Weather & Conditions Data | ✅ Locked (with open questions) | `04_Weather_Data.md` |
| Mountain Status Tracking | ✅ Locked (with open questions) | `05_Mountain_Status.md` |
| Tech Stack | ✅ Locked (with open questions) | `06_Tech_Stack.md` |
| Sharing & Popular Setups | ✅ Locked (with open questions) | `07_Sharing_Social.md` |
| Admin Console | ✅ Locked (with open questions) | `08_Admin_Console.md` |
| Notifications | 🔲 Not started — interview pending | `09_Notifications.md` |
| User Profile & Account | 🔲 Not started — interview pending | `10_User_Profile.md` |
| Monetization | 🔲 Not started — decision pending | `11_Monetization.md` |
| Data Model | 🔲 Not started — mostly derivation from 09–11 | `12_Data_Model.md` |
| API Design | 🔲 Not started | `13_API_Design.md` |
| Security & Compliance | 🔲 Not started — consolidates TCPA/GDPR/scraping | `14_Security_Compliance.md` |
| Testing & QA | 🔲 Not started — medium priority | `15_Testing_QA.md` |
| LLM Strategy | 🔲 Not started — medium priority | `16_LLM_Strategy.md` |
| Analytics & Metrics | 🔲 Not started — medium priority | `17_Analytics_Metrics.md` |
| Conditions Tiering | 🔲 Not started — promote from backlog | `0X_Conditions_Tiering.md` (number TBD) |
| Launch / Rollout | 🔲 Lower priority | `18_Launch_Rollout.md` (optional) |
| Accessibility / i18n | 🔲 Lower priority | `19_Accessibility_i18n.md` (optional) |
| Resort Onboarding Playbook | 🔲 Lower priority | `20_Resort_Onboarding_Playbook.md` (optional) |

---

## ⚡ RESUME HERE — Exact Next Steps

A full coherence/gap review was completed on 2026-04-19. Full findings in `PRD_REVIEW_2026-04-19.md`. The work below is sequenced so the PRD reaches implementation-ready state.

### Step 1: Housekeeping pass ✅ DONE (2026-05-16)

All five contradictions resolved and all stale references cleaned up across `00`–`08`. Committed as a single batch. Specifically:

- **Weather API** flipped to Open-Meteo primary + Tomorrow.io parallel/fallback. `WeatherProvider` abstraction added to `06` as a locked architectural decision. `[OPEN]` weather-API question closed.
- **Scrape cadence** unified to "Every 2–4 hrs in-season" across `04`/`05`/`06`/`08`. Per-resort Celery job isolation made architecturally explicit.
- **Status headers** standardized across `00`–`08` to `Sub-spec NN | Status: **LOCKED (with open questions)** | Last Updated: 2026-05-16`. "of N" framing removed.
- **Alert fire formula** in `02` updated with road-closure override framing (override modifies alert content, not the fire condition; extensible for future safety overrides).
- **Scout Templates** locked in `02` with 6 canonical templates: FIRST TRACKS, STORM CHASER, POW HUNT, BLUEBIRD, PARK DAY, ROAD TRIP. Relationship to Popular Setups defined: Templates from Day 1 for cold-start; Popular Setups take precedence once a mountain has ≥N active Scouts. Vocabulary aligned in `03` and `07`.
- **Auth provider** `[OPEN]` in `01` closed (Supabase Auth). Stale Clerk reference removed.
- **Resort-ops pointer cleanup** in `04`: Resort Data Profile and Internal Resort Operations Dashboard sections now point to `08` as the authoritative spec.
- **00 Master PRD updates**: stale "Named Alert Profiles" backlog item replaced by Scouts reference; ops-dashboard backlog item removed (specced in 08); feature-areas table expanded to include 09–20+ planned/optional specs; open questions list refreshed with newly-locked decisions and newly-surfaced opens.

### Step 2: Interview Zack → write `09_Notifications.md`

Most critical missing spec. The alert experience is the core value delivery of Gnarcast and is currently unspecced. Interview topics:

- **SMS message format** — what does the text actually say? Data points included, length, CTA, share link at bottom (format already in 07: `gnar.st/c/[token]`).
- **Email alert format** — same content as SMS or richer? HTML template?
- **Alert frequency limits** — can a Scout fire every day? Cooldown period? (If Crystal has epic conditions for 5 days, does user get 5 texts?)
- **Notification lookahead window** — resolve the 1/3/7-day `[OPEN]` question from 01 and 00. Does "3-day" mean "alert me now if it looks great in 3 days"?
- **Multiple Scouts firing simultaneously** — if 3 Scouts threshold on the same day, do 3 separate messages go out or one consolidated message?
- **Alert send time** — what time of day do alerts send? (e.g., 7am so user can decide before work?)
- **Stale condition edge case** — alert fires, conditions change before user acts, any follow-up?
- **Pre-alert / forecast nudge** — separate from the alert itself, does Gnarcast send a "conditions building" heads-up before threshold is crossed?
- **Two-way SMS commands** (GOING/SKIP/PAUSE/MORE/STOP) — referenced in 02, needs to be formalized here with exact responses.

### Step 3: Interview Zack → write `10_User_Profile.md`

Profile page referenced throughout but never designed. Critical items from the review:

- **Home address** — 05 says users set this "once in their profile" but 01's onboarding flow has no step for it. Resolve: captured during onboarding (new step) or after onboarding in profile settings?
- **Ski pass management** — set at onboarding step 5 but no spec for editing later.
- **Connected accounts UX** — mentioned as concept in 01, needs actual design.
- **TCPA consent management** — users need to view/manage their SMS opt-in from settings.
- **Phone loss / phone number change flow** — unspecced; critical because auth is passwordless + SMS OTP.
- **Account deletion and data export (GDPR/CCPA)** — legally required, not specced anywhere.
- **Global notification controls** — "pause all alerts" separate from per-Scout pausing?
- **Profile page location in nav, sections, layout.**

Consider during this step: the review recommends moving onboarding steps 4–7 (mountain selection, pass, conditions, notifications) out of `01_Auth_Signup.md` and into `10_User_Profile.md`, leaving 01 to own only auth itself (steps 1–3). Decide with Zack whether to do that restructure now.

### Step 4: Quick decision → write `11_Monetization.md`

Short decision-record spec. Key question: free vs. freemium vs. paid subscription? What's free, what's paid? Affects the data model (subscriptions table, feature flags, paywalls) — must be decided before 12_Data_Model can be finalized.

### Step 5: Write `12_Data_Model.md`

Mostly derivation work once 09–11 are done. Tables, columns, relationships, indexes, RLS policies. No long interview needed — most of this can be reconstructed from existing specs and then reviewed with Zack.

### Step 6: Parallel tracks → `13_API_Design.md` and `14_Security_Compliance.md`

- 13 covers: REST/Edge-Function contract between frontend and backend, Twilio webhook endpoints, sharing link endpoints (gnar.st), SMS reply handler contracts.
- 14 consolidates: TCPA (currently in 01), CAN-SPAM, GDPR/CCPA (currently nowhere), scraping legality (one sentence in 04), rate limiting, secret management, threat model.

Can be written in either order or in parallel.

### Step 7: Promote "Conditions Tiering" out of backlog

Currently parked in 00, 02, 04, and 05 backlog sections but is load-bearing for the onboarding UX and the resort data profile. Short spec — should list Tier 1 / Tier 2 / Tier 3 conditions explicitly and the logic for when each tier is surfaced to users.

### Step 8: Reorganization pass (mechanical, no interviews)

One commit. From review §4:
- Create a concepts/glossary doc (or fold into 00) defining Scout, Alert, Condition, Signal, Resort, Phase, Tier, Region, Archetype, Template, Setup.
- Consolidate the full conditions catalog — make 02 the canonical list, have 04 and 05 reference by name instead of re-listing.
- Consolidate the data-refresh cadence table — make 04 canonical, have 05/06/08 reference.
- Consolidate backlog ownership — pick "master backlog in 00 with sub-specs linking up" OR "each sub-spec owns its backlog and 00 summarizes." Current duplication is the problem.

### Step 9: Medium-priority specs

`15_Testing_QA.md`, `16_LLM_Strategy.md`, `17_Analytics_Metrics.md`. Can happen in parallel with early coding. LLM strategy is the most urgent of these because the product uses LLMs in at least four places (Scout naming, plain-English summary, bidirectional summary editing, popular-setup naming) with no single home for model/cost/prompt/fallback decisions.

### Step 10: Lower-priority specs (optional pre-launch)

`18_Launch_Rollout.md`, `19_Accessibility_i18n.md`, `20_Resort_Onboarding_Playbook.md`. Nice-to-have; not blocking for a first version.

---

## Master PRD-Level Gaps (flagged in review, not yet addressed)

These belong in `00_Master_PRD.md` as additions, not as new specs:

- **Success metrics / KPIs** — what defines Gnarcast as "working"? Alert count? Alert-to-trip conversion? Week-2 retention? Currently nothing is measurable.
- **Timeline / launch target** — no dates or phasing committed.
- **Risks & assumptions register** — the entire product assumes 163 resort websites can be scraped reliably enough to deliver accurate alerts. That's a load-bearing assumption (ToS, site changes, legal) with no captured risk entry.

Consider adding these during Step 1 housekeeping or as a dedicated mini-interview before Step 2.

---

## Decisions Pending From Previous Sections

- ✅ ~~Auth provider~~ — **Supabase Auth** (decided in `06_Tech_Stack.md`)
- ✅ ~~Alert scoring model~~ — **user-configured tiered weights + sensitivity dial** (decided in `02_User_Preferences.md`)
- ✅ ~~Weather API~~ — **Tomorrow.io primary + Open-Meteo fallback** (decided in `06_Tech_Stack.md`) — still needs 04 updated to reflect this
- `[OPEN]` Email verification: blocking vs. async — not yet resolved (resolve in 10 or 14)
- `[OPEN]` Notification lookahead window — resolve in `09_Notifications.md`
- `[OPEN]` Monetization model — resolve in `11_Monetization.md`
- `[OPEN]` Scout sharing opt-in vs. default-on — low priority, resolve before shipping sharing feature
- `[OPEN]` Named lift selection in Scout config — launch vs. post-launch
- `[OPEN]` Minimum Scout count threshold for Popular Setups — calibrate at launch
- `[OPEN]` Intensity multiplier formula (02 scoring) — linear, exponential, or capped?
- `[OPEN]` Scout upper bound — unlimited or soft cap? (Affects alert throttling and compute cost.)

---

## Backlog Items (captured, not yet scoped)

Moved here so they don't get lost; see also individual spec backlog sections (to be consolidated in Step 8).

- Distance/drive-time as a Scout scoring signal (currently informational only)
- Deep-dive on each onboarding step (exact copy, validation, error states)
- Three Scout setup flow versions (quick/standard/expert)
- Algorithm feedback / backtesting — let users rate alerts, tune scoring over time
- Scout templates — pre-built Scouts for common rider profiles
- Two-way SMS interactions — specced in 02 conceptually; formalize in 09
- Resort data profile schema — drives Scout config UI per resort
- Proactive condition unlock notifications
- Historical data retention policy
- Mobile app design spec (when iOS development begins)
- Full logo design pass — custom letterforms, icon mark, usage guidelines

---

## Interview Style Notes

- Go one section at a time, interview first, write spec after section is closed.
- Document the **why** behind every decision, not just the what.
- Use tags: `[LOCKED]`, `[OPEN]`, `[BACKLOG]`, `[WHY]`.
- Keep responses conversational — no bullet-point dumps, ask one focused question at a time.
- User is Zackaria (Zack) — product is Gnarcast, a personalized ski/snowboard conditions alert platform.
- Zack has a CS degree, 20 years in big tech, hasn't coded in a few years, building this as a solo project to start.

---

## Git & File Workflow

- **Repo location on Zack's Mac:** `~/Documents/Claude/Projects/Gnarcast`
- **GitHub repo:** https://github.com/zackariaali/gnarcast
- **Sandbox git issue:** The mounted macOS filesystem blocks git lock file operations from the sandbox. Commits sometimes succeed with warnings. If `git commit` fails with HEAD.lock error, Zack runs this from Terminal: `cd ~/Documents/Claude/Projects/Gnarcast && rm -f .git/*.lock && git add . && git push`
- **gh CLI:** Authenticated as `zackariaali`, HTTPS protocol.

---

## PRD Document Conventions

```
[LOCKED]   — decision made, not revisiting
[OPEN]     — needs a decision
[BACKLOG]  — good idea, parked for later
[WHY]      — rationale for a decision
```

---

## Review Document Reference

Full coherence/gap review is at `PRD_REVIEW_2026-04-19.md`. If this session-state file becomes unclear, that doc is the authoritative source for **why** each of the steps above exists. Structure of that doc:

1. Top-level assessment
2. Direct contradictions (5 items)
3. Content gaps within existing specs (spec-by-spec)
4. Missing specs (12 through 20+)
5. Reorganization opportunities
6. Recommended sequence
7. What's already good (don't break these)
