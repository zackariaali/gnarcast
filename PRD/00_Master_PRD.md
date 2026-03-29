# Gnarcast — Master PRD

> **Status:** In Progress | **Last Updated:** 2026-03-29

---

## Vision & Mission

**Vision:** Never miss an epic day on the mountain. `[LOCKED]`

**Mission:** Your personal mountain scout — personalized conditions, forecasts, and alerts so you always know when your mountain is calling. `[LOCKED]`

---

## Product Overview

Gnarcast is a web-first (with future mobile app) platform that tracks snow conditions and mountain status across ski resorts, and delivers personalized alerts to users when their ideal conditions are met. The product is built for passionate skiers and snowboarders — not the casual family-of-four, but the rider who actually cares about conditions.

The core value proposition: the further in advance a user knows about their perfect day, the better they can plan to actually be there. Gnarcast closes the gap between "I wish I knew" and "I'll be on the hill Friday."

---

## Target Audience

- Passionate skiers and snowboarders (intermediate to expert)
- Condition-aware riders — not just bluebird chasers, but storm chasers, pow hounds, and anyone with specific conditions they care about
- Multi-mountain riders with ski pass awareness (Ikon, Epic, Indy, etc.)
- Busy adults who need advance notice to make arrangements

**Not the primary audience:** Casual/beginner skiers, families planning annual trips, ski resort operators

---

## Feature Areas & Sub-Specs

| Area | Sub-Spec | Status |
|------|----------|--------|
| Auth & Signup Flow | [01_Auth_Signup.md](./01_Auth_Signup.md) | In Progress |
| User Preferences | [02_User_Preferences.md](./02_User_Preferences.md) | Not Started |
| Look & Feel | [03_Look_Feel.md](./03_Look_Feel.md) | Not Started |
| Weather & Conditions Data | [04_Weather_Data.md](./04_Weather_Data.md) | Not Started |
| Mountain Status Tracking | [05_Mountain_Status.md](./05_Mountain_Status.md) | Not Started |
| Tech Stack | [06_Tech_Stack.md](./06_Tech_Stack.md) | Not Started |
| Sharing & Social | [07_Sharing_Social.md](./07_Sharing_Social.md) | Not Started |

---

## Key Principles

- **Personalization first** — no two users should have the same alert thresholds or mountain lists
- **Timeliness over volume** — one well-timed, relevant alert beats ten noisy ones
- **Mobile-aware from day one** — web first, but every decision should account for the future app
- **Why we decide things matters** — rationale is documented alongside every decision

---

## Backlog (Parking Lot)

> Items captured during interviews that are good ideas but not yet scoped.

- `[BACKLOG]` Distance and traffic/drive-time as a factor in alert scoring — e.g. "Is it worth the 4-hour drive given current conditions?" Not core to initial mission but a natural evolution.
- `[BACKLOG]` Deep-dive into each onboarding step to define exact copy, field validation, and error states.
- `[BACKLOG]` Named Alert Profiles with configurable mountain sets, condition thresholds, and notification preferences (see 01_Auth_Signup.md for initial capture).

---

## Open Questions

- `[OPEN]` Final auth provider decision (Clerk vs. Supabase Auth) — deferred to tech stack discussion.
- `[OPEN]` Monetization model — free, freemium, or paid subscription? To be discussed.
- `[OPEN]` Alert scoring model — user-defined thresholds vs. Gnarcast-calculated smart score vs. both?
- `[OPEN]` Notification lookahead window — fixed (3-day, 7-day) or user-configurable?

---

*This document is the index and overview. All detailed decisions live in the sub-specs linked above.*
