# Gnarcast — Weather & Conditions Data Spec

> **Status:** Locked | **Last Updated:** 2026-03-29
> **Related:** `05_Mountain_Status.md`, `02_User_Preferences.md`, `data/gnarcast-resort-directory.xlsx`

---

## Overview

The data pipeline is the engine of Gnarcast. Everything users experience — Scout scores, alerts, confidence indicators, the LLM summaries — flows from how reliably and completely we can collect, normalize, and evaluate conditions data for each resort.

Two guiding principles:
1. **Honest over optimistic** — if we don't have data, we say so. We never inflate confidence or fill gaps with assumptions.
2. **Own the infrastructure** — we build and maintain our own scraping stack rather than depending on third-party aggregators. More upfront work, real moat over time.

---

## Data Tiers

### Tier 1 — Required at Launch `[LOCKED]`

| Signal | Source | Method | Freshness |
|--------|--------|---------|-----------|
| Temperature (high/low/feels-like) | Open-Meteo or Tomorrow.io | API | Hourly |
| Wind speed & direction | Open-Meteo or Tomorrow.io | API | Hourly |
| Visibility & cloud cover | Open-Meteo or Tomorrow.io | API | Hourly |
| Precipitation forecast (snow/rain) | Open-Meteo or Tomorrow.io | API | Hourly |
| New snow last 24hrs (estimated) | SNOTEL + weather API | API | Hourly |
| New snow last 48–72hrs (estimated) | SNOTEL + weather API | API | Hourly |
| Base depth | Resort snow reports | Scrape | Daily 6–7am |
| Mountain open % (runs & lifts) | Resort websites | Scrape | Daily 6–7am |
| Avalanche danger level | avalanche.org API | API | Daily |
| Ski pass compatibility | Static internal database | Static | On resort onboard |
| Day of week & holiday proximity | Internal calendar logic | Calculated | Real-time |

### Tier 2 — Required at Launch `[LOCKED]`

| Signal | Source | Method | Freshness |
|--------|--------|---------|-----------|
| Grooming reports | Resort websites | Scrape | Daily 6–7am |
| Crowd estimates | Day-of-week + holiday + post-storm model | Calculated | Daily |
| Specific lift status | Resort websites | Scrape | Daily 6–7am |
| Snow water equivalent (SWE) | SNOTEL network | API | Daily |
| Newly opened terrain | Resort snow reports | Scrape + NLP | Daily 6–7am |
| Active storm vs. post-storm | Weather API + pattern logic | Calculated | Hourly |
| Days since last significant snowfall | Internal derived from history | Calculated | Daily |

### Tier 3 — Post-Launch `[BACKLOG]`

Slope aspect & wind loading, social signals (crowdsourced ratings), lift line wait times, resort events/competitions, night skiing availability, lift ticket pricing.

---

## Data Sources

### Weather APIs `[OPEN — final selection TBD at tech stack]`

**Open-Meteo** — free, no API key, excellent global coverage, historical data to 1940. Best for: temperature, wind, precipitation, cloud cover, snow accumulation forecasts. Strongly preferred for cost and reliability.

**Tomorrow.io** — generous free tier, more granular real-time data, better hyperlocal accuracy. Good fallback or supplement. May be worth for ski-specific forecast quality.

**[WHY not a single provider]** Weather APIs go down, change pricing, or degrade accuracy. Running Open-Meteo as primary with Tomorrow.io as fallback provides resilience at minimal cost.

### SNOTEL Network (USDA)

Free public sensor network with stations near most major western US ski resorts. Provides: snow depth, snow water equivalent (SWE), temperature, precipitation. Coverage is excellent for the West, limited in the Northeast and Southeast.

**[WHY important]** SNOTEL is ground truth for snow conditions — more reliable than weather model estimates for actual snowpack. When a resort's snow report hasn't updated yet, SNOTEL fills the gap.

### avalanche.org API

Public API from the American Avalanche Association. Covers most western US avalanche centers. Returns danger levels by elevation band and aspect. Free, reliable, updated daily by certified forecasters.

**Canada equivalent:** Parks Canada and regional avalanche centers (Avalanche Canada) — similar structure, different endpoints. To be integrated per-region during rollout.

### Resort Scraping Infrastructure `[LOCKED]`

Built and owned by Gnarcast. Key design decisions:

**Per-resort scraper configs** — each resort has a scraper configuration that defines: the URL(s) to hit, the HTML selectors or JSON paths to extract data from, the field mapping to our internal schema, and the expected update schedule. Configs stored in a structured registry, not hardcoded.

**Scraper resilience** — resorts change their websites. Scrapers will break. The system must: detect failures (no update in expected window), alert the ops dashboard, fall back to last known good data with a staleness flag, and queue for manual review.

**Epic/Vail resorts first** — Vail Resorts properties (Breckenridge, Vail, Whistler, etc.) tend to have consistent, machine-readable snow report structures. Building scrapers for these first establishes patterns that apply across ~20 resorts efficiently.

**Scraping ethics** — use reasonable rate limits, respect `robots.txt`, and identify the scraper in the User-Agent header. We are not adversarial — if a resort blocks us, we explore official data partnerships as an alternative.

---

## Data Refresh Cadence `[LOCKED]`

| Data Type | Frequency | Notes |
|-----------|-----------|-------|
| Weather forecast | Hourly | Conditions shift throughout the day |
| Mountain status (lifts, runs, grooming) | Daily at ~6am | Resorts publish morning reports |
| Avalanche danger | Daily | Forecasters update each morning |
| SNOTEL readings | Hourly | Sensor network updates frequently |
| Scout score evaluation | After each data refresh | Alerts fire when score threshold crossed |
| Historical data archival | Continuous | All data retained — see retention backlog |

**[WHY hourly weather but daily resort status]** Weather forecasts genuinely change hour by hour — an afternoon wind event or storm cell can flip a day from great to skip. Resort status (which lifts are open, grooming) only changes meaningfully once a day when the morning report drops. Polling resort sites hourly would waste resources and risk getting rate-limited.

---

## Data Confidence System `[LOCKED]`

Every condition signal in a Scout score carries a **confidence level** — not shown as a number to users, but expressed qualitatively when relevant.

### Confidence Levels

| Level | Description | User-facing |
|-------|-------------|-------------|
| High | Fresh data from reliable source, verified | Shown normally, no indicator |
| Medium | Data is slightly stale or from a secondary source | Subtle indicator, no alert |
| Low | Data is old, estimated, or inferred | Visible indicator + suggestion to verify |
| Missing | Signal not available for this resort | Excluded from score, noted if relevant |

### User-Facing Confidence UI `[LOCKED]`

When a Scout card or alert contains low-confidence data, a subtle indicator appears:

> *"We're not sure about grooming at Crystal today — our data is 14hrs old. Worth checking their site, or [let us know what you find!]"*

The "let us know" link triggers a quick 3-option crowdsource prompt: **Groomed ✓ / Not groomed ✗ / Not sure**. One tap. This feeds directly into the confidence model and seeds the social data layer.

**[WHY expose confidence to users]** Surfacing uncertainty builds trust. A product that quietly fires alerts based on stale data destroys credibility when users show up and conditions are wrong. A product that honestly says "we're not sure about this one — go verify" keeps users informed and engaged. The crowdsource prompt turns an honest limitation into a community feature.

### Missing Signal Behavior `[LOCKED]`

| Situation | Behavior |
|-----------|----------|
| Signal missing for resort | Exclude from score, do not assume positive or negative |
| Signal stale (>24hrs) | Include with Low confidence, flag in UI |
| Signal from fallback source (e.g. SNOTEL instead of resort) | Include with Medium confidence, no UI flag unless user inspects |
| No weather data at all | Block Scout evaluation, do not alert |
| No mountain status at all | Evaluate on weather signals only, flag as partial |

---

## Resort Data Profile `[BACKLOG — see 00_Master_PRD.md]`

Each resort in the directory has an internal **data profile** — a structured record of:
- Which condition signals are available (by tier)
- Last successful scrape timestamps per signal
- Scraper health status (green/yellow/red)
- Whether the resort is user-visible (availability toggle)

Scout configuration UI shows only conditions available for a given resort. Users never see the underlying data profile. When a resort's profile gains new signals, users with that resort in a Scout are notified via the prompt-based setup flow.

---

## Internal Resort Operations Dashboard `[BACKLOG — see 00_Master_PRD.md]`

Admin-only tool. Never exposed to users. Features:
- Per-resort availability toggle (controls user-facing visibility)
- Data quality status per signal (green/yellow/red)
- Last scraped timestamps
- Raw scraped data inspector for debugging
- Minimum data threshold gate (controls when a resort graduates to user-visible)
- Scraper failure alerts and queue management

**[WHY hidden from users]** Data quality management is an internal operations problem. Users should never see a resort with partial or broken data — they should either see it working properly or not see it at all. The dashboard is for the team, not the product.

---

## Resort Onboarding Gate `[LOCKED]`

A resort is not exposed to users until it meets a minimum data threshold. Exact threshold TBD but likely: Tier 1 weather signals ✓ + base depth ✓ + mountain open % ✓ + at least one season of historical data loaded.

Resorts that don't meet the threshold appear in the internal dashboard but not in user-facing search or Scout configuration. Completely invisible to users.

---

## Historical Data `[LOCKED]`

All collected data is retained indefinitely. No deletion policy until storage costs and usage patterns are better understood. Historical data enables:
- Backtesting Scout configurations against past seasons
- Algorithm calibration and false positive/negative analysis
- Trend analysis (is this mountain getting less snow over time?)
- Training future ML models on conditions + user behavior

**[WHY keep everything]** You can't go back and collect data you didn't store. Storage is cheap. The cost of a retention policy mistake (deleting something useful) far exceeds the cost of keeping it all.

---

## Rollout Strategy `[LOCKED]`

Data pipeline is built region by region, following the resort directory launch plan (`data/gnarcast-resort-directory.xlsx`):

- **Phase 1 (PNW + Eastern WA + Idaho/Sun Valley):** Build and validate the full data pipeline on 21 resorts. Establish scraper patterns, confidence system, and ops dashboard.
- **Phase 2 (CA + CO + UT):** Scale to 56 more resorts. Pipeline is proven, focus on scraper coverage.
- **Phase 3+ :** Continued expansion. By Phase 3, scraping patterns are well-established and new regions move faster.

**[WHY phased rollout]** Attempting to scrape 163 resorts at once before the pipeline is proven is a recipe for noisy, unreliable data. Phase 1 is a controlled proving ground. Ship fewer resorts with excellent data rather than more resorts with mediocre data.

---

## Open Questions

- `[OPEN]` Final weather API selection: Open-Meteo vs. Tomorrow.io vs. combination — decide at tech stack
- `[OPEN]` Exact minimum data threshold for resort graduation to user-visible
- `[OPEN]` Canadian avalanche API integration (Avalanche Canada) — structure differs from US avalanche.org
- `[OPEN]` Scraping rate limits and legal review per resort — some ToS explicitly prohibit scraping
- `[OPEN]` Confidence score weighting — how much does a Low-confidence signal affect the overall Scout score?
