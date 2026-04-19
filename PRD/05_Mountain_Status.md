# Gnarcast PRD — Mountain Status Tracking
> Sub-spec 05 of 07 | Status: **LOCKED (with open questions)**
> Related specs: `04_Weather_Data.md` (conditions layer), `02_User_Preferences.md` (Scout scoring)

---

## Overview

Mountain Status is the **operational layer** — what the resort itself is doing, as distinct from what nature is doing (weather/snowpack). While `04_Weather_Data.md` covers conditions signals (snow depth, temperature, avalanche danger), this spec covers resort operations: is it open, can you get there, what terrain is accessible, and how crowded will it be.

Both layers feed the same Scout scoring engine but come from different data sources and have different update cadences.

[WHY] Keeping these separate makes the data pipeline cleaner — resort operational data comes from scraping and DOT APIs, while conditions data comes from weather/sensor APIs. They also have different failure modes: a resort being closed is a hard gate, while a bad snow report is just a low score.

---

## Signals Overview

| Signal | Category | Data Source | Update Cadence | Scout Scoring |
|--------|----------|-------------|----------------|---------------|
| Resort open/closed | Operations | Resort website scrape | Daily (+ change detection) | Dealbreaker eligible |
| Lift status (# open, which lifts) | Operations | Resort website scrape | Every 2–4 hrs in season | Configurable |
| Terrain open % | Operations | Resort website scrape | Daily | Configurable |
| Grooming report | Operations | Resort website scrape | Daily (6–8am) | Configurable |
| Newly opened terrain | Operations | Resort website scrape | Daily | Configurable |
| Parking availability | Operations | Resort API / scrape (where available) | Hourly | Informational only |
| Predicted crowd level | Crowd | Algorithmic (see below) | Daily | Configurable (Avoid tier) |
| Road open/closed | Access | State DOT API | 15–30 min | Hard override (see below) |
| Chain control / traction advisory | Access | State DOT API | 15–30 min | Informational in alert |
| Drive time (from home) | Access | Google Maps / Mapbox API | On-demand at alert time | Informational in alert |

---

## Signal Definitions

### 1. Resort Open / Closed

**What it is:** Whether the resort is operating on a given day — covers seasonal open, seasonal close, weather closures (e.g., avalanche control), and unplanned closures.

**Data source:** Resort website scrape + resort-published status page where available.

**Scoring:** Eligible as a Dealbreaker. If a user has this set and the resort is closed, Scout does not fire — no point alerting someone to drive 3 hours to a closed mountain.

[WHY] This is the single most important operational signal. Everything else is moot if the mountain isn't open.

---

### 2. Lift Status

**What it is:** How many lifts are operating and, where possible, which named lifts.

**Two levels of granularity:**
- **Aggregate:** "14 of 22 lifts open" — always available, scraped from resort snow report page
- **Named lift status:** "High Campbell Chair: Open / Gondola: Closed" — available at some resorts, best-effort

**Scoring:** Configurable per Scout. A user who needs a specific lift (e.g., the gondola to access the summit) could set that as a condition. More commonly, users will set a minimum lift count or % threshold.

[OPEN] Do we expose named lift selection in Scout config at launch, or only aggregate lift count? Named lift tracking requires per-resort field mapping that may not be feasible at Phase 1 scale.

---

### 3. Terrain Open %

**What it is:** The percentage of the mountain's total skiable terrain that is currently open.

**Example:** "65 of 138 trails open (47%)" or "2,100 of 3,600 skiable acres open"

**Data source:** Resort snow report / trail report page scrape.

**Scoring:** Configurable per Scout. Users can set a minimum terrain open % threshold — "I only want to go if at least 70% is open."

[WHY] A mountain can be technically open with 5 trails running. Terrain open % is a better proxy for whether the day is worth it than open/closed alone.

---

### 4. Grooming Report

**What it is:** Which runs were groomed overnight and when the report was published.

**Data source:** Resort website scrape, typically posted by 7–8am day-of.

**Scoring:** Configurable per Scout. "Send It" if grooming is listed / "Nice to Have" if certain named runs are groomed. Users who prioritize groomers can weight this heavily.

**Freshness flag:** Grooming reports have a published timestamp. If we can't verify a report was posted today (scrape failure, site change), we surface a Low confidence indicator rather than showing stale data.

---

### 5. Newly Opened Terrain

**What it is:** Terrain (runs, bowls, zones) that opened since the last data refresh — e.g., a backcountry gate or upper mountain that just became accessible.

**Data source:** Delta comparison between consecutive resort scrapes.

**Scoring:** Configurable per Scout as a positive signal. Particularly relevant in early/mid season when terrain is progressively opening.

**Proactive notification (backlog):** Even if a Scout didn't fire, newly opened terrain at a saved resort could trigger a push notification. Captured in backlog.

---

### 6. Parking Availability

**What it is:** Live or near-live parking lot fill level at the resort.

**Data source:** Resort-published parking status (Crystal Mountain, Whistler, and a few others publish this). Not universally available.

**Scoring:** Informational only at launch — displayed in alert context but not used in Scout scoring.

[WHY] Parking data is too inconsistently available across resorts to be a reliable scoring signal at launch. Include as informational where available; revisit when data coverage improves.

[OPEN] Could webcam analysis eventually provide parking lot fill estimates for resorts that don't publish live data?

---

### 7. Predicted Crowd Level

**What it is:** An estimated busyness score (Low / Moderate / High / Very High) for the resort on a given day, derived algorithmically.

**Scoring signals used:**
- Day of week (Saturday/Sunday = higher baseline)
- Holiday calendar proximity (Presidents' Day, MLK, school breaks)
- Storm timing: large snowfall events reliably bring crowds 1–3 days later (the "storm chaser" effect)
- Ikon / Epic pass blackout dates at the target resort (proxy for expected volume)
- Historical patterns per resort (modeled over time as we accumulate data)

**Scout scoring:** Configurable as an **Avoid** signal — users can de-prioritize or suppress alerts when predicted crowd level is High or Very High.

[WHY] Real-time crowd data (lift line wait times, etc.) is not consistently available. A prediction model based on known crowd drivers is achievable at launch and genuinely useful — most experienced skiers already do this mental math anyway.

**Output:** A single `crowd_score` (0–100) mapped to Low / Moderate / High / Very High, with the primary drivers listed in the alert (e.g., "High — post-storm Saturday + Presidents' Day weekend").

---

## Access Layer: Road Conditions

### Overview

Access data answers the question: **"Can I actually get there today?"** This is surfaced separately from Scout scoring and treated as a safety-critical layer.

### User Home Address

Users set a home/starting address once in their profile. This is used for:
1. Drive time estimates (Google Maps / Mapbox API, calculated at alert time with live traffic)
2. Route identification — mapping which highway segments to monitor per resort
3. Distance as a potential future scoring signal (backlog)

[WHY] Personalizing road conditions to the user's actual route makes this far more useful than a generic "roads may be icy" disclaimer.

### Pre-Mapped Resort Routes

Each resort in the directory has 1–3 primary highway routes pre-mapped by the Gnarcast ops team. Example:

| Resort | Primary Route | Highway Segments | DOT API Region |
|--------|--------------|------------------|----------------|
| Crystal Mountain | SR-410 via Enumclaw | WA SR-410 (Greenwater to Crystal) | WSDOT |
| Stevens Pass | US-2 via Monroe or Everett | WA US-2 (Index to Stevens) | WSDOT |
| Mt. Bachelor | US-97 + Century Dr | OR US-97, OR-372 | ODOT |
| Sun Valley | US-75 (Titus Dr) | ID US-75 (Hailey to Ketchum) | ITD |

Route pre-mapping is part of the resort onboarding checklist in the ops dashboard.

### Road Status Signals

| Signal | Source | Update Cadence | Display |
|--------|--------|----------------|---------|
| Road open / closed | State DOT API | 15–30 min | Hard override banner |
| Chain control / traction advisory | State DOT API | 15–30 min | Shown in alert context |
| Road conditions (dry/wet/snow/ice) | State DOT API | 15–30 min | Shown in alert context |
| Estimated drive time | Google Maps / Mapbox | At alert generation | Shown in alert context |

**State DOT APIs:**
- Washington: WSDOT Traveler Information API (public, no key required for basic endpoints)
- Oregon: ODOT TripCheck API
- Idaho: ITD 511 API
- Utah: UDOT 511 API
- Colorado: CDOT Road Conditions API
- California: Caltrans QuickMap API
- Additional DOT APIs mapped per region during phased rollout

---

## Road Closure Override

Road closures are treated as a **hard override** — distinct from normal Scout scoring.

**Behavior:**
- If a Scout fires (conditions score meets threshold) but the primary route to that resort has an active road closure, the alert still sends.
- A prominent warning is prepended to the alert: **"⚠️ Road Closed — SR-410 to Crystal Mountain is currently closed. Verify conditions before heading out."**
- The Scout score and conditions summary display normally below the warning.

[WHY] We don't suppress the alert entirely — road closures can lift quickly and the user may want to monitor. But we never let an alert go out without surfacing a known closure. Safety-critical information always shows.

**Chain control** is not a closure — it's informational context shown in the alert body. Users are adults who know how to put on chains.

---

## Scout Scoring Integration

Mountain Status signals integrate into the Scout scoring engine defined in `02_User_Preferences.md`. Users can configure each signal using the same tier system:

| Tier | Points | Example use |
|------|--------|-------------|
| Dealbreaker | Alert suppressed | Resort closed, road closed |
| Send It | +7 | 80%+ terrain open |
| Nice to Have | +3 | Grooming report posted |
| Yolo | +1 | New terrain opened |
| Ignore | 0 | Not relevant to this Scout |
| Avoid | −4 | Predicted crowd: High |

**Road closure** is the only signal that bypasses the scoring system entirely — it triggers an override warning regardless of score.

---

## Data Availability by Resort Tier

Resort tiers (defined in `04_Weather_Data.md`) affect which Mountain Status signals are available in Scout configuration:

| Signal | Tier 1 Resort | Tier 2 Resort |
|--------|--------------|--------------|
| Open/Closed | ✅ Required | ✅ Required |
| Lift count (aggregate) | ✅ Required | ✅ Required |
| Terrain open % | ✅ Required | ⚠️ Best effort |
| Grooming report | ✅ Required | ⚠️ Best effort |
| Named lift status | ⚠️ Best effort | ❌ Not required |
| Parking availability | ⚠️ Where published | ❌ Not required |
| Crowd prediction | ✅ Algorithmic (all resorts) | ✅ Algorithmic |
| Road conditions | ✅ Required (pre-mapped) | ✅ Required (pre-mapped) |
| Drive time | ✅ All resorts | ✅ All resorts |

---

## Open Questions

- `[OPEN]` Named lift selection in Scout config — launch feature or post-launch? Requires per-resort lift field mapping.
- `[OPEN]` Parking webcam analysis — viable long-term signal for resorts without published parking data?
- `[OPEN]` Road conditions for international resorts (Whistler, Blackcomb) — Canadian DOT equivalents (DriveBC API exists).
- `[OPEN]` Should drive time affect Scout scoring (e.g., "only alert me if drive is under 2 hours")? Currently informational only — captured as backlog.

---

## Backlog

- `[BACKLOG]` Drive time as a Scout scoring signal ("only alert if under X hours from home")
- `[BACKLOG]` Named lift status for Tier 1 resorts — requires per-resort field mapping in ops dashboard
- `[BACKLOG]` Webcam analysis for parking lot fill estimation
- `[BACKLOG]` Proactive terrain unlock notifications — alert when a new zone opens at a saved resort even if Scout didn't fire
- `[BACKLOG]` Real-time lift wait times — if any resort APIs ever expose this
- `[BACKLOG]` DriveBC API integration for Canadian resorts (Whistler, Big White, etc.)
