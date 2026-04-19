# Gnarcast — Master PRD

> **Status:** First Pass Complete | **Last Updated:** 2026-04-19

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
| Auth & Signup Flow | [01_Auth_Signup.md](./01_Auth_Signup.md) | ✅ Locked |
| User Preferences | [02_User_Preferences.md](./02_User_Preferences.md) | ✅ Locked |
| Look & Feel | [03_Look_Feel.md](./03_Look_Feel.md) | ✅ Locked |
| Weather & Conditions Data | [04_Weather_Data.md](./04_Weather_Data.md) | ✅ Locked |
| Mountain Status Tracking | [05_Mountain_Status.md](./05_Mountain_Status.md) | ✅ Locked |
| Tech Stack | [06_Tech_Stack.md](./06_Tech_Stack.md) | ✅ Locked |
| Sharing & Popular Setups | [07_Sharing_Social.md](./07_Sharing_Social.md) | ✅ Locked |
| Admin Console | [08_Admin_Console.md](./08_Admin_Console.md) | ✅ Locked |

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
- `[BACKLOG]` **Algorithm feedback & calibration system** — track when alerts were wrong in both directions: (1) false positives — alerted the user but conditions weren't actually worth it, (2) false negatives — a great day happened and Gnarcast missed it. Use LLM-assisted analysis to understand why the algorithm got it wrong and suggest threshold adjustments. Needs a backtesting framework: feed historical conditions data through the algorithm and see what would have been triggered differently. This is key to making the scoring system self-improving over time.
- `[BACKLOG]` **Tiered conditions rollout** — the full conditions list (~30 signals) should not all be presented to users at once. Conditions are tiered by complexity and relevance. Core tier presented at onboarding; advanced tier (e.g. SWE, aspect/wind loading, specific lift status) surfaced progressively or on request. Need to define tier 1 vs. tier 2 vs. tier 3 conditions and the logic for when/how to prompt users to configure them.
- `[BACKLOG]` **Three preference setup flow versions** — (1) **Onboarding flow**: minimal, gets the user to a working first profile fast; (2) **Ad hoc flow**: user creates a new named Alert Profile from scratch at any time; (3) **Prompt-based flow**: Gnarcast proactively asks users about specific conditions at the right moment (e.g. "A big event is scheduled at Mammoth this weekend — do you want us to factor resort events into your alerts?"). Each flow needs its own UX spec.
- `[BACKLOG]` **User feedback loop for missing conditions** — a free-text input where users can describe a condition Gnarcast isn't currently factoring in. System behavior: (1) LLM checks if the condition already exists and routes the user to the right setting with an explanation; (2) if it doesn't exist, log it as a feature request, add to the product backlog, and notify the user when it's been built. This creates a direct channel between user frustration and product improvement.
- `[BACKLOG]` **Resort data profile schema** — every resort has an internal record of which condition signals we have reliable data for (e.g. new snow ✓, grooming ✗, specific lift status ✗). Scout configuration UI dynamically shows only conditions we can track for a given resort. Profile updates automatically as scraping coverage improves.
- `[BACKLOG]` **Proactive condition unlock notifications** — when a resort's data profile gains a new trackable signal, proactively notify users who have that resort in a Scout and offer to add the new condition. Ties into the prompt-based setup flow.
- `[BACKLOG]` **Internal resort operations dashboard** — admin-only tool (never exposed to users) for managing resort data quality and availability. Features: per-resort availability toggle (on/off for user-facing product), data quality status indicator per resort (which signals are green/yellow/red), last-scraped timestamps, debug tools to inspect raw scraped data, and a minimum data threshold gate that controls when a resort graduates to user-visible. This is essential operational infrastructure for managing 163 resorts across 16 regions.
- `[BACKLOG]` **Historical data retention policy** — keep all data for now, revisit retention strategy once usage patterns and storage costs are better understood. Do not delete anything until there's a clear reason to.

---

## Open Questions

- `[LOCKED]` ~~Auth provider~~ — **Supabase Auth** (decided in 06_Tech_Stack.md)
- `[OPEN]` Monetization model — free, freemium, or paid subscription? To be discussed.
- `[LOCKED]` ~~Alert scoring model~~ — **user-configured tiered weights + sensitivity dial** (decided in 02_User_Preferences.md)
- `[OPEN]` Notification lookahead window — fixed (3-day, 7-day) or user-configurable?
- `[OPEN]` Scout sharing opt-in vs. default-on — see 07_Sharing_Social.md
- `[OPEN]` Named lift selection in Scout config — launch feature or post-launch? See 05_Mountain_Status.md
- `[OPEN]` Minimum Scout count threshold for Popular Setups feature — calibrate at launch

---

*This document is the index and overview. All detailed decisions live in the sub-specs linked above.*
