# Gnarcast — User Preferences Spec

> **Status:** In Progress | **Last Updated:** 2026-03-29

---

## Overview

User Preferences defines how Gnarcast personalizes the experience for each rider. The core unit of personalization is a **Scout** — a named, self-contained configuration that watches a set of mountains and alerts the user when conditions match their criteria. Users can have unlimited Scouts, each fully independent.

---

## Scouts

### What is a Scout? `[LOCKED]`

A Scout is Gnarcast's first-class personalization primitive. It is a saved configuration consisting of:

- A **name** (user-defined, default "Home Mountain")
- A **mountain list** (one or more resorts to monitor)
- **Condition preferences** (what to watch for, with thresholds and weights)
- **Notification settings** (SMS, email, or both; advance notice window)
- A **schedule** (active dates, days of week)
- A **status** (active, paused, or archived)

**[WHY]** A single global preference set forces users to choose one way to monitor all their mountains. Scouts accommodate the reality of how skiers plan — home mountain logic is completely different from a destination trip or a powder emergency watch.

### Scout Rules `[LOCKED]`

- A user can have **unlimited Scouts**
- The **same mountain can appear in multiple Scouts** with completely different conditions and thresholds
- Scouts are **fully independent** — no shared settings between them
- The first Scout is created during onboarding and defaults to the name "Home Mountain"

**[WHY unlimited]** Skiers' needs are diverse — a home mountain Scout, a Tahoe trip Scout, a powder emergency Scout covering 10 resorts are all legitimate simultaneous use cases. Capping Scouts would artificially constrain the product's value.

### Scout States `[LOCKED]`

| State | Description |
|-------|-------------|
| Active | Monitoring conditions, will send alerts when triggered |
| Paused | Temporarily suspended — no alerts sent, preferences preserved |
| Scheduled | Active only during configured date ranges and/or days of week |
| Archived | Inactive and hidden from main view, but preserved |

### Scout Scheduling `[LOCKED]`

Users can configure a Scout to activate automatically during specific windows:
- Date ranges (e.g. "December 1 through April 15")
- Days of week (e.g. "weekdays only" or "weekends only")

**[WHY]** Skiers already think in seasons. A Scout that auto-activates for the season and auto-sleeps in the off-season removes the overhead of manual management.

### Scout Interactions `[LOCKED]`

Scouts can be interacted with via two channels:

**SMS (quick actions):** When an alert is sent, users can reply with keywords:

| Keyword | Action |
|---------|--------|
| GOING | Marks user as attending — useful for future social features |
| SKIP | Snoozes this alert for the day |
| PAUSE | Pauses the Scout until manually reactivated |
| MORE | Returns a detailed conditions breakdown |
| STOP | Unsubscribes from all alerts (TCPA required) |

**App/Site (full management):** Editing thresholds, creating new Scouts, scheduling, viewing history — all require the web app or future mobile app.

**[WHY split]** SMS two-way interaction (via Twilio) handles the time-sensitive decisions a user makes the morning an alert arrives. Deep configuration requires a full UI. When the mobile app launches, SMS quick actions can migrate to push notification taps.

### Pre-Defined Scout Templates `[BACKLOG]`

To accelerate onboarding and help new users get value faster, Gnarcast will offer pre-built Scout templates:

- **Powder Hound** — optimized for deep snow, high new snow threshold, low crowd tolerance
- **Groomer Morning** — fresh corduroy, sunny and cold, early day conditions
- **Storm Chaser** — active snowfall, acceptable visibility, mid-storm vibes
- **Bluebird Day** — post-storm sunshine, settled snow, crowd-aware

Users pick a template, select their mountains, and are live in under 60 seconds.

### Scout Sharing `[BACKLOG]`

Future feature: users can share a Scout configuration with friends. Recipient gets a copy of the Scout (with their own mountains applied) rather than a live subscription to someone else's Scout.

---

## Conditions System

### Full Conditions List `[LOCKED]`

All trackable signals, organized by category:

**Snow Quality**
- New snow last 24hrs (inches)
- New snow last 48–72hrs (inches)
- Base depth (inches)
- Snow surface type (powder / packed powder / groomed / corn / crud / ice)
- Days since last significant snowfall (staleness)
- Snow water equivalent (SWE) — density of snow; differentiates light Utah powder from heavy Cascade concrete
- Newly opened terrain percentage — when a resort opens a new bowl, back country area, or significant run

**Weather**
- Temperature high and low (°F)
- Feels-like temperature
- Wind speed and direction
- Visibility and sunshine (cloud cover, clear sky percentage)
- Actively snowing during the day (storm day vs. bluebird)
- Overnight/incoming storm forecast (it's going to snow tonight)

**Mountain Status**
- Percentage of mountain open (acres and runs)
- Specific terrain open: bowls, trees, steeps, park, backcountry
- Number and percentage of lifts running
- Specific lift status (user can flag lifts they care about)
- Avalanche danger level
- Grooming reports (fresh grooming = corduroy morning)

**Crowds**
- Estimated crowd level
- How busy the day before (was it skied out?)
- Day of week (weekday vs. weekend)
- Proximity to holidays and school breaks
- Days since last powder day (post-storm surge predictor)
- Lift line wait time estimates
- Resort events (competitions, concerts, rail jams)

**Practical**
- Drive distance and time
- Traffic conditions
- Ski pass compatibility (Ikon, Epic, Indy, other)
- Lift ticket price (for non-pass days — dynamic pricing signal)

**Social Signals**
- Crowdsourced condition ratings from Gnarcast users at that mountain
- Recent trail reports and notes from other riders

**Aspect & Advanced** *(Tier 2 — surfaced progressively)*
- Slope aspect and wind loading (north-facing holds powder longer, south-facing sun-affected)
- Night skiing availability

**[WHY this list]** Hardcore skiers make decisions based on a complex mental model of conditions. The more signals Gnarcast can factor in, the more precisely it can match a user's ideal day — and the more differentiated it becomes from a basic weather app.

### Conditions Tiering `[BACKLOG]`

Not all ~30 conditions should be presented at once. Three tiers:
- **Tier 1 (onboarding):** New snow, visibility, temperature, crowds, drive time, pass compatibility
- **Tier 2 (progressive):** SWE, specific lift status, avalanche danger, resort events, social signals
- **Tier 3 (advanced/prompted):** Aspect, wind loading, night skiing, ticket pricing

Full tiering spec to be defined separately.

---

## Scoring & Weighting System

### Design Principle `[LOCKED]`

The scoring system is entirely internal — users never see points or math. They interact only through:
1. **Weight tiers** expressed in ski-culture terms
2. **Thresholds** expressed in natural units (inches, °F, %)
3. **Sensitivity** expressed as Epic / Great / Good

**[WHY hidden scoring]** Asking users to think in points creates friction and feels like a spreadsheet, not a mountain scout. The ski-culture vocabulary makes weight-setting intuitive while the algorithm does the heavy lifting underneath.

### Weight Tiers `[LOCKED]`

| Tier | Points | Description |
|------|--------|-------------|
| Dealbreaker | Veto | Cancels alert entirely if condition not met — regardless of score |
| Send It | +7 base | Heavily drives the decision |
| Nice to Have | +3 base | Adds meaningful weight; enough Nice to Haves can tip the scale |
| Yolo | +1 base | Barely registers, going regardless |
| Ignore | 0 | Not factored in for this Scout |
| Avoid | −4 base | Actively hurts the score — not a dealbreaker but drags it down |

### Scoring Formula `[LOCKED — algorithm TBD]`

**Score = Σ (base points × intensity multiplier) − penalties**

Where **intensity multiplier** rewards conditions that exceed the threshold. Example: 12" of new snow when the threshold is 6" scores higher than barely clearing 6". The multiplier is applied proportionally to how far above/below the threshold the condition lands.

**Alert fires when:** Score ≥ trigger threshold AND no Dealbreakers are failed

**Trigger threshold** maps to user-facing sensitivity:
- **Epic** — high bar, only genuinely special days
- **Great** — solid days worth the trip
- **Good** — lower bar, alert me more often

Exact point thresholds for Epic/Great/Good to be calibrated with real data and adjusted over time.

### Forecast-Aware Dealbreakers `[LOCKED]`

Dealbreakers must account for both historical and forecast data. Example: "no snow in 48hrs" is a dealbreaker — UNLESS it's actively snowing or a storm is forecast to arrive today. The system evaluates:

- **Actual conditions** (what has happened)
- **Forecast conditions** (what is coming)

Users set dealbreakers at the condition level; the system applies them intelligently across both timeframes. Question phrasing handles this transparently: *"Does a big storm coming count, even if it hasn't snowed yet?"*

### Threshold Input `[LOCKED]`

Thresholds support two input modes:
- **Buckets** (default): dusting / 3"+ / 6"+ / 12"+ for snow; mild / cold / freezing for temp, etc.
- **Precise** (advanced): raw number input for power users

**[WHY both]** Buckets cover 90% of users with minimal friction. Precise input satisfies the data-driven rider who knows they want exactly 8" before they'll drive 3 hours.

---

## LLM Plain-English Summary `[LOCKED]`

Each Scout's configuration is summarized in a human-readable paragraph generated by an LLM from the structured preference data.

**Example:**
> *"You're chasing powder days at Mammoth — you want at least 6" of new snow in the last 24 hours, temps staying below 30°F, and low winds. You prefer uncrowded weekdays and won't go if it's raining. You don't mind if it's snowing during the day but visibility needs to be decent. Your sensitivity is set to Great days."*

Users can **edit this paragraph directly** and the system parses it back into updated structured data.

**[WHY]** Natural language as the UI for preference editing is more intuitive than a settings panel for many users. It also serves as a legibility check — if the summary doesn't sound right, the user's settings probably aren't right either.

---

## Algorithm Feedback & Calibration `[BACKLOG]`

See `00_Master_PRD.md` backlog for full description. Key points:
- Track false positives (alerted but conditions weren't worth it) via post-trip prompts
- Track false negatives (great day missed) via automated post-hoc condition analysis
- Backtesting framework: run current preferences against historical data to show users what they would have been alerted to
- LLM-assisted analysis to explain misses and suggest threshold adjustments

---

## User Feedback Loop `[BACKLOG]`

See `00_Master_PRD.md` backlog for full description. Free-text input for conditions not currently tracked. System either routes to existing setting or logs as a feature request with user notification on completion.

---

## Open Questions

- `[OPEN]` Exact point thresholds for Epic / Great / Good sensitivity levels — to be calibrated with data
- `[OPEN]` Intensity multiplier formula — linear, exponential, or capped?
- `[OPEN]` Tier 1 / Tier 2 / Tier 3 conditions breakdown — full tiering spec needed
- `[OPEN]` Should Avoid conditions have a "soft avoid" (subtracts points) vs. "hard avoid" (dealbreaker)? Or is Dealbreaker sufficient for hard stops?
- `[OPEN]` LLM summary parsing — bidirectional editing needs careful scoping to avoid unexpected preference changes
