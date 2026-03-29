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
| User Preferences | 🔄 In progress — conditions list locked, weighting & profiles next | `02_User_Preferences.md` |
| Look & Feel | 🔲 Not started — interview pending | `03_Look_Feel.md` |
| Weather & Conditions Data | 🔲 Not started — interview pending | `04_Weather_Data.md` |
| Mountain Status Tracking | 🔲 Not started — interview pending | `05_Mountain_Status.md` |
| Tech Stack | 🔲 Not started — interview pending | `06_Tech_Stack.md` |
| Sharing & Social | 🔲 Not started — interview pending | `07_Sharing_Social.md` |

---

## Next Up: User Preferences (`02_User_Preferences.md`)

This is the next interview section. Topics to cover:

- Mountain selection (how users manage their list, add/remove mountains)
- Ski pass awareness (Ikon, Epic, Indy, etc.) and how it filters mountains
- Condition preferences — what data points matter (fresh snow, wind, visibility, temperature, crowds, avalanche danger, etc.) and how users configure thresholds
- Named Alert Profiles — full spec (introduced in Auth spec as a backlog item, needs full treatment here)
- Profile defaults vs. customization depth
- How preferences map to the alert/scoring system

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
- **Git workflow:** Claude writes files, Zack commits and pushes from his Terminal
- **When to commit:** Claude will call out "ready to commit" after each meaningful update — Zack runs `git add . && git push` from the Gnarcast folder
- **Why:** The mounted filesystem prevents Claude from running git commits directly; writing files works fine

---

## PRD Document Conventions

```
[LOCKED]   — decision made, not revisiting
[OPEN]     — needs a decision
[BACKLOG]  — good idea, parked for later
[WHY]      — rationale for a decision
```
