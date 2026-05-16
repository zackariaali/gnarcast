# Gnarcast — Weather & Conditions Data Spec

> Sub-spec 04 | Status: **LOCKED (with open questions)** | Last Updated: 2026-05-16
> Related specs: `05_Mountain_Status.md`, `02_User_Preferences.md`, `data/gnarcast-resort-directory.xlsx`

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
| Temperature (high/low/feels-like) | Open-Meteo + Tomorrow.io | API | Hourly |
| Wind speed & direction | Open-Meteo + Tomorrow.io | API | Hourly |
| Visibility & cloud cover | Open-Meteo + Tomorrow.io | API | Hourly |
| Precipitation forecast (snow/rain) | Open-Meteo + Tomorrow.io | API | Hourly |
| New snow last 24hrs (estimated) | SNOTEL + Open-Meteo | API | Hourly |
| New snow last 48–72hrs (estimated) | SNOTEL + Open-Meteo | API | Hourly |
| Base depth | Resort snow reports | Scrape | Every 2–4 hrs in-season |
| Mountain open % (runs & lifts) | Resort websites | Scrape | Every 2–4 hrs in-season |
| Avalanche danger level | avalanche.org API | API | Daily |
| Ski pass compatibility | Static internal database | Static | On resort onboard |
| Day of week & holiday proximity | Internal calendar logic | Calculated | Real-time |

### Tier 2 — Required at Launch `[LOCKED]`

| Signal | Source | Method | Freshness |
|--------|--------|---------|-----------|
| Grooming reports | Resort websites | Scrape | Every 2–4 hrs in-season (updates ~once daily) |
| Crowd estimates | Day-of-week + holiday + post-storm model | Calculated | Daily |
| Specific lift status | Resort websites | Scrape | Every 2–4 hrs in-season |
| Snow water equivalent (SWE) | SNOTEL network | API | Daily |
| Newly opened terrain | Resort snow reports | Scrape + NLP | Every 2–4 hrs in-season (updates ~once daily) |
| Active storm vs. post-storm | Weather API + pattern logic | Calculated | Hourly |
| Days since last significant snowfall | Internal derived from history | Calculated | Daily |

### Tier 3 — Post-Launch `[BACKLOG]`

Slope aspect & wind loading, social signals (crowdsourced ratings), lift line wait times, resort events/competitions, night skiing availability, lift ticket pricing.

---

## Data Sources

### Weather APIs `[LOCKED]`

**Open-Meteo (primary)** — free, no API key, excellent global coverage, historical data to 1940. Exposes raw output from multiple high-resolution regional NWP models (ICON-D2 at 2km for Europe, ICON-US, AROME for the Alps) with elevation-aware forecasts and dedicated snow-specific parameters (snowfall amount, snow depth, freezing level height). Best for: temperature, wind, precipitation, cloud cover, snow accumulation forecasts.

**Tomorrow.io (parallel signal + fallback)** — commercial provider with generous free tier, polished real-time signals (precipitation type, weather codes), friendlier developer experience. Used in parallel with Open-Meteo for confidence comparison, and as automatic fallback if Open-Meteo is unavailable.

**[WHY Open-Meteo primary]** Both providers pull from the same underlying NWP models (GFS, ECMWF) — the differentiator is post-processing. Open-Meteo's raw multi-model exposure lets us do ensemble-style comparison and supports the data-confidence system's "honest about uncertainty" principle. It's also free with no API key and no surprise pricing changes, removing a class of operational risk.

**[WHY run both]** Weather APIs go down, change pricing, or degrade accuracy. Running both in parallel gives us automatic fallback resilience, and the ability to A/B compare prediction accuracy against actual observed conditions over time. Neither provider beats SNOTEL sensors on the ground for snowpack truth — both are model-driven forecasts.

**Provider abstraction:** Weather providers are abstracted behind a `WeatherProvider` interface (see `06_Tech_Stack.md`). The "primary" designation is a configuration flag, not hardcoded — switching primary is a config change, not a migration. This keeps the architecture provider-agnostic and supports future provider additions without schema changes.

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
| Mountain status (lifts, runs, grooming, terrain) | Every 2–4 hrs in-season | One unified scrape pass per resort pulls all signals together |
| Avalanche danger | Daily | Forecasters update each morning |
| SNOTEL readings | Hourly | Sensor network updates frequently |
| Scout score evaluation | After each data refresh | Alerts fire when score threshold crossed |
| Historical data archival | Continuous | All data retained — see retention backlog |

**[WHY one unified scrape pass per resort]** Most resorts publish lifts, terrain, grooming, and base depth on a single snow-report page. Polling that page every 2–4 hours pulls all signals together in one HTTP request — splitting into per-signal jobs would multiply fetches without changing the data. Grooming and terrain typically only update once per morning, so most polls just confirm "no change" — that's expected and cheap.

**[WHY 2–4 hrs and not hourly]** Lift status genuinely changes during the day (e.g., a wind hold on the gondola at noon), but resorts don't update their public snow-report pages more often than every couple hours in practice. Hourly polling would waste resources and risk rate-limiting without producing fresher data.

**[WHY per-resort jobs, not one monolithic job]** Each resort has its own scraper job with custom logic, its own scheduled run, and its own log stream. Failure of one resort's scraper (a site layout change, a timeout, a captcha) is contained to that resort — the other 162 keep running. This also makes the admin console's per-resort debug tools (see `08_Admin_Console.md`) work directly against the same per-resort job execution.

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

## Resort Data Profile `[LOCKED — operational details in 08_Admin_Console.md]`

Each resort in the directory has an internal **data profile** — a structured record of which condition signals are available (by tier), last successful scrape timestamps per signal, scraper health status (green/yellow/red), and whether the resort is user-visible.

Scout configuration UI shows only conditions available for a given resort. Users never see the underlying data profile. When a resort's profile gains new signals, users with that resort in a Scout are notified via the prompt-based setup flow.

The admin tooling that manages resort data profiles is fully specced in `08_Admin_Console.md` (see "Resort Management" → "Resort Detail View").

---

## Internal Resort Operations Dashboard `[LOCKED — fully specced in 08_Admin_Console.md]`

Admin-only tool, never exposed to users. Provides per-resort availability toggle, data quality status per signal, last-scraped timestamps, raw scraped data inspector, minimum data threshold gate, and scraper failure alerts.

**[WHY hidden from users]** Data quality management is an internal operations problem. Users should never see a resort with partial or broken data — they should either see it working properly or not see it at all. The dashboard is for the team, not the product.

Full operational spec: see `08_Admin_Console.md`.

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

- ✅ ~~Final weather API selection~~ — **Open-Meteo primary + Tomorrow.io parallel/fallback, behind `WeatherProvider` abstraction** (locked above)
- `[OPEN]` Exact minimum data threshold for resort graduation to user-visible
- `[OPEN]` Canadian avalanche API integration (Avalanche Canada) — structure differs from US avalanche.org
- `[OPEN]` Scraping rate limits and legal review per resort — some ToS explicitly prohibit scraping
- `[OPEN]` Confidence score weighting — how much does a Low-confidence signal affect the overall Scout score?
