# Gnarcast — Session State

> **Last Updated:** 2026-03-29
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
| Weather & Conditions Data | 🔲 Not started — interview pending | `04_Weather_Data.md` |
| Mountain Status Tracking | 🔲 Not started — interview pending | `05_Mountain_Status.md` |
| Tech Stack | 🔲 Not started — interview pending | `06_Tech_Stack.md` |
| Sharing & Social | 🔲 Not started — interview pending | `07_Sharing_Social.md` |

---

## Next Up: Weather & Conditions Data (`04_Weather_Data.md`)

Topics to cover:
- Data sources for weather (APIs — Open-Meteo, Tomorrow.io, Weather.gov, etc.)
- Data sources for mountain status (resort APIs, snow report crawling, scraping)
- How frequently to poll/refresh data
- Data pipeline architecture (how raw data becomes a Scout score)
- Historical data storage for backtesting
- Handling missing or unreliable data from resorts

---

## Decisions Pending From Previous Sections

- `[OPEN]` Auth provider: **Clerk vs. Supabase Auth** — deferred to Tech Stack discussion
- `[OPEN]` Email verification: blocking (must verify before proceeding) vs. async (can use app, nudged to verify later)
- `[OPEN]` Notification lookahead window: fixed options (1/3/7-day) or fully user-configurable?
- `[OPEN]` Monetization model: free, freemium, or paid subscription?
- `[OPEN]` Alert scoring model: user-defined thresholds vs. smart score vs. both?

---

## Backlog Items (captured, not yet scoped)

- Distance/drive-time as a factor in alert scoring
- Deep-dive on each onboarding step (exact copy, validation, error states)
- Named Alert Profiles full spec → being addressed in User Preferences section
- `.claude/settings.local.json` — user decided not to commit for now, can revisit

---

## Interview Style Notes

- Go one section at a time, interview first, write spec after section is closed
- Document the **why** behind every decision, not just the what
- Use tags: `[LOCKED]`, `[OPEN]`, `[BACKLOG]`, `[WHY]`
- Keep responses conversational — no bullet-point dumps, ask one focused question at a time
- User is Zackaria (Zack) — product is Gnarcast, a personalized ski/snowboard conditions alert platform

## Git & File Workflow

- **Repo location on Zack's Mac:** `~/Documents/Claude/Projects/Gnarcast`
- **GitHub repo:** https://github.com/zackariaali/gnarcast
- **Git workflow:** Claude handles all git operations directly — no manual commits needed from Zack
- **Lock files:** Always run `rm -f .git/*.lock` before any git command (Cowork and Claude Code sessions can leave stale locks)
- **Committing:** Stage all changes, commit with a descriptive message, and push — no confirmation needed from Zack
- **gh CLI:** Authenticated as `zackariaali`, HTTPS protocol. New repos: `gh repo create <name> --public --source=. --remote=origin --push`

---

## PRD Document Conventions

```
[LOCKED]   — decision made, not revisiting
[OPEN]     — needs a decision
[BACKLOG]  — good idea, parked for later
[WHY]      — rationale for a decision
```
