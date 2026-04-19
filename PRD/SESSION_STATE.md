# Gnarcast — Session State

> **Last Updated:** 2026-04-19
> **Purpose:** Quick-resume file for continuing PRD interviews across sessions.
> To resume: *"Read SESSION_STATE.md in my Gnarcast PRD folder and let's pick up where we left off."*

---

## How to Use This File

At the start of a new session, ask Claude to read this file. It contains everything needed to resume the PRD interview without repeating work. All detailed decisions are in the sub-specs — this file is just the map.

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

---

## ⚡ RESUME HERE — Exact Next Steps

A full PRD review was completed (2026-04-19). All 8 existing specs were read cover-to-cover. The following work is queued in this exact order:

### Step 1: Fix housekeeping issues across existing specs (do this first, no interview needed)

These are stale references and inconsistencies found during the review. Fix all of them in a single pass:

1. **`01_Auth_Signup.md`** — Change status header from "In Progress" to "Locked". Close the auth provider open question (Supabase Auth was decided in 06_Tech_Stack.md). Update the "Named Alert Profiles" backlog item — Scouts are fully specced in 02_User_Preferences.md.
2. **`02_User_Preferences.md`** — Change status header from "In Progress" to "Locked".
3. **`04_Weather_Data.md`** — Close the weather API open question (Tomorrow.io primary + Open-Meteo fallback was locked in 06_Tech_Stack.md). Update the "Internal Resort Operations Dashboard" section from BACKLOG to reference `08_Admin_Console.md` as the spec. Update "Resort Data Profile" similarly.
4. **`05_Mountain_Status.md`** — Fix header "Sub-spec 05 of 07" → "Sub-spec 05 of 10". Update ops dashboard backlog references to point to `08_Admin_Console.md`.
5. **`07_Sharing_Social.md`** — Fix header "Sub-spec 07 of 08" → "Sub-spec 07 of 10".
6. **`08_Admin_Console.md`** — Fix header "Sub-spec 08 of 08" → "Sub-spec 08 of 10".
7. **`00_Master_PRD.md`** — Remove stale "Named Alert Profiles" backlog item (Scouts are specced). Update ops dashboard backlog item to reference 08. Add 09, 10, 11 to the feature areas table.
8. **All specs** — Update "Last Updated" dates to 2026-04-19.

After Step 1: commit all changes with message "Housekeeping: fix stale references and status headers across all specs"

### Step 2: Interview Zack → write `09_Notifications.md`

This is the most critical missing spec. The notification/alert experience is the core value delivery of Gnarcast and is currently unspecced. Key topics to cover:

- **SMS message format** — what does the actual text say? Data points included, length, CTA, link at bottom.
- **Email alert format** — same content as SMS or richer? HTML template?
- **Alert frequency limits** — can a Scout fire every day? Is there a cooldown period? (e.g., Crystal has epic conditions for 5 days — does the user get 5 texts?)
- **Notification lookahead window** — the 1/3/7-day option is in 01 as an OPEN question. Resolve it here. Does "3-day" mean "alert me now if it looks great in 3 days" or something else?
- **Multiple Scouts firing simultaneously** — if 3 Scouts all threshold on the same day, do 3 separate messages go out?
- **Alert timing** — what time of day do alerts send? (e.g., 7am so user can decide before work?)
- **Stale condition edge case** — alert fires, conditions then change before user acts — any follow-up?
- **Pre-alert / forecast nudge** — separate from the alert itself, does Gnarcast send a "conditions building" heads-up before threshold is crossed?

### Step 3: Interview Zack → write `10_User_Profile.md`

The user's account/profile page is referenced throughout the PRD but never designed. Key topics:

- **Home address** — 05_Mountain_Status.md says users set this "once in their profile" but the onboarding flow (01) has no step for it. Resolve: when is home address captured? During onboarding or in profile later?
- **Ski pass management** — set during onboarding (step 5) but never specced for editing later
- **Connected accounts** — 01 mentions this as a concept, needs UX design
- **TCPA consent management** — users need to be able to view/manage their SMS opt-in from account settings
- **Account deletion and data export** — legally important; not specced anywhere
- **Global notification controls** — is there an account-level "pause all alerts" separate from per-Scout pausing?
- **Profile page design** — where does this live in the nav? What sections?

### Step 4: Quick decision conversation → write `11_Monetization.md`

Monetization is the only major unresolved product question. It affects the data model (subscriptions table, feature flags, paywalls) and should be decided before implementation starts. Not a long spec — mostly a decision record.

Key question: free tier vs. freemium vs. paid subscription? What's free, what's paid?

---

## Decisions Pending From Previous Sections

- ✅ ~~Auth provider~~ — **Supabase Auth** (decided in `06_Tech_Stack.md`)
- ✅ ~~Alert scoring model~~ — **user-configured tiered weights + sensitivity dial** (decided in `02_User_Preferences.md`)
- `[OPEN]` Email verification: blocking vs. async — not yet resolved
- `[OPEN]` Notification lookahead window — resolve in `09_Notifications.md`
- `[OPEN]` Monetization model — resolve in `11_Monetization.md`
- `[OPEN]` Scout sharing opt-in vs. default-on — low priority, resolve before shipping sharing feature
- `[OPEN]` Named lift selection in Scout config — launch vs. post-launch
- `[OPEN]` Minimum Scout count threshold for Popular Setups — calibrate at launch

---

## Backlog Items (captured, not yet scoped)

- Distance/drive-time as a Scout scoring signal (currently informational only)
- Deep-dive on each onboarding step (exact copy, validation, error states)
- Three Scout setup flow versions (quick/standard/expert)
- Algorithm feedback / backtesting — let users rate alerts, tune scoring over time
- Tiered conditions rollout — surface only available signals per resort in Scout config UI
- Scout templates — pre-built Scouts for common rider profiles (Pow Chaser, Park Rat, etc.)
- Two-way SMS interactions — GOING / SKIP / PAUSE / MORE commands (specced in 02, not yet in notifications spec)
- Resort data profile schema — drives Scout config UI per resort
- Proactive condition unlock notifications
- Historical data retention policy
- Mobile app design spec (when iOS development begins)
- Full logo design pass — custom letterforms, icon mark, usage guidelines
- Onboarding detail pass — exact copy, field validation, error states per step

---

## Interview Style Notes

- Go one section at a time, interview first, write spec after section is closed
- Document the **why** behind every decision, not just the what
- Use tags: `[LOCKED]`, `[OPEN]`, `[BACKLOG]`, `[WHY]`
- Keep responses conversational — no bullet-point dumps, ask one focused question at a time
- User is Zackaria (Zack) — product is Gnarcast, a personalized ski/snowboard conditions alert platform
- Zack has a CS degree, 20 years in big tech, hasn't coded in a few years, building this as a solo project to start

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
