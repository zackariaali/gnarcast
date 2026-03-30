# Gnarcast — Look & Feel Spec

> **Status:** Locked | **Last Updated:** 2026-03-29
> **Reference mockup:** `mockups/scout-card-mockup.html`

---

## Design Philosophy

Gnarcast should feel like it was built by someone who actually skis — not a generic SaaS dashboard. Every design decision should evoke the visceral pull of a great day on the mountain: cinematic, confident, and a little bit charged. The visual language borrows from ski culture brands (Burton, Salomon, Völkl) and high-end editorial design, filtered through a modern web sensibility.

**Core feeling:** You open the app and immediately want to go ride.

---

## Brand Identity

### Logo `[LOCKED]`

**GNAR/CAST** — all caps, Helvetica Now, split color treatment:
- **GNAR** — deep purple (`#8b2be2`)
- **/** — dim separator (`rgba(0,0,0,0.2)` on light, `rgba(255,255,255,0.25)` on dark)
- **CAST** — neon chartreuse (`#7ec800`)

The slash is structural — it echoes ski run notation, fraction bars, and the "send/receive" metaphor of scouting. Font weight 700, letter spacing 0.04em, uppercase. Logo always sits top-left, never recolored or rearranged.

**[WHY]** Split-color wordmarks are immediately distinctive and ownable. Purple/green is unexpected in ski branding (which skews blue/white/red) — it stands out while still feeling technical and energized.

**[BACKLOG]** Full logo design pass — custom letterforms, icon mark, usage guidelines, dark/light variants.

### Typography `[LOCKED]`

**Display font:** Helvetica Now Display — all major headlines, Scout names, hero text. Uppercase, tight tracking (−2px to −4px at large sizes), weight 700. No italic.

**Body font:** Helvetica Now Text — all supporting copy, labels, summaries, UI text. Mixed case, normal tracking, weights 400/500/600.

**Fallback stack:** `'Helvetica Now Display', 'Helvetica Now', 'Helvetica Neue', Helvetica, Arial, sans-serif`

**[WHY Helvetica Now]** Burton's brand font — immediately credible in ski culture without being derivative. Clean Swiss typography at display sizes conveys precision and confidence. The Display cut is optimized for large sizes, Text cut for readability at small sizes.

**Type scale (approximate):**
- Hero headline: `clamp(56px, 8vw, 108px)`, uppercase, tracking −4px
- Scout name on card: `clamp(30px, 3.2vw, 46px)`, uppercase, tracking −1.5px
- Section labels: `10–11px`, uppercase, tracking +0.2em, muted
- Body / summaries: `11.5–13px`, normal weight, normal tracking
- Nav links: `14px`, weight 500

### Color Palette `[LOCKED]`

| Token | Value | Usage |
|-------|-------|-------|
| Purple | `#8b2be2` / `#c084fc` | Logo GNAR, Epic score badge, accent |
| Chartreuse | `#7ec800` / `#b3ff00` | Logo CAST, active neon accents |
| Active green | `#22c55e` | Status LED dot (active Scout) |
| Alert amber | `#b45309` / `#fbbf24` | Conditions building signal |
| Sky blue | `#0369a1` / `#38bdf8` | Great score badge |
| Ink | `#0d0d0d` | Primary text on light backgrounds |
| White | `#ffffff` | Primary text on dark backgrounds |
| Glass light | `rgba(255,255,255,0.55)` | Frosted card backgrounds (light theme) |
| Glass dark | `rgba(10,12,18,0.78)` | Dark card variant |
| Overlay | `rgba(240,246,250,0.45)` | Video background wash |

---

## Video Background `[LOCKED]`

### Behavior
Full-viewport looping video plays behind all page content on the homepage (logged-out) and dashboard (logged-in). Key properties:
- **Autoplay, muted, no controls** — no sound ever, ever
- **Random start** — on each page load, a video is randomly selected from the library so the experience feels fresh
- **Crossfade on swap** — when a video ends, a 2-second opacity crossfade transitions to the next video. Two video layers (A/B) stacked with CSS transitions handle this invisibly
- **Preloading** — the next video is silently preloaded while the current one plays, eliminating buffering gaps at the crossfade

### Overlay
A light frosted overlay (`rgba(240,246,250,0.45)`) sits above the video, toning it down slightly and ensuring text and cards remain legible without obscuring the footage.

### Video Library
- **Source format:** MP4 (H.264), minimum 1080p, ideally 4K for retina displays
- **Duration:** 15–60 seconds per clip — long enough to feel cinematic, short enough to loop without fatigue
- **Content:** Epic ski and snowboard footage — powder runs, steep chutes, bluebird groomers, tree skiing, backcountry lines. Mix of disciplines and mountain types
- **Tone:** Action and beauty — rider-inspired, not resort marketing. Think ski film aesthetic, not tourist ad

### Video Sourcing Strategy `[OPEN]`
Options to evaluate during build:
- License clips from ski film companies (Warren Miller, Matchstick Productions, Poor Boyz)
- Stock footage platforms (Artgrid, Storyblocks) for affordable high-quality ski content
- User-submitted footage (future — community-sourced, moderated)
- Mountain-specific footage tied to user's selected resorts (future — when a user's Scout is for Mammoth, show Mammoth footage)

**[WHY no static images]** Video backgrounds create immediate emotional engagement. A looping powder run in the background makes the product feel alive in a way static imagery cannot. The crossfade ensures it never feels like a slideshow.

**[WHY video assets on CDN, not git]** Binary video files bloat git history permanently and make cloning slow. All production video assets are hosted on Cloudflare R2 or S3 and referenced by URL. Videos are `.gitignored` in this repo.

---

## Page Layout

### Navigation Bar `[LOCKED]`

Always fixed at top of viewport — never scrolls away, always accessible.

| Position | Element |
|----------|---------|
| Left | GNAR/CAST logo |
| Right | Conditions (text link) · Alerts (text link) · Account (pill button) |

Nav background: transparent over video, subtle frosted blur when content scrolls beneath it. Account button: rounded pill, semi-transparent white, border `rgba(0,0,0,0.2)`.

### Hero Section `[LOCKED]`

Logged-in dashboard and logged-out landing page share the same hero layout:
- Small uppercase label: `YOUR PERSONAL MOUNTAIN SCOUT`
- Giant headline: **THE MOUNTAIN / IS CALLING.** — "IS CALLING." in muted color
- CTA button: **"Send out your Scout →"** — pill style, frosted glass
- Supporting tagline: *"Personalized alerts so you never miss the days that matter."*

**Logged-out variant:** Same layout. Scout cards below show pre-populated example Scouts (see Scout Templates). A demo banner appears above the cards: *"These are example Scouts. Sign up free to build your own."*

### Scout Dashboard (Card Grid) `[LOCKED]`

Below the hero, a CSS grid of Scout cards:
- `grid-template-columns: repeat(auto-fill, minmax(340px, 1fr))`
- 14px gap between cards
- Cards wrap naturally as viewport narrows
- Last card is always the **+ New Scout** add card

**Hover spotlight effect:** When hovering any card, all sibling cards blur (`blur(2.5px)`) and fade (`opacity: 0.4`). The hovered card subtly lifts (`translateY(-4px) scale(1.01)`). Implemented via CSS `:has()` selector. Establishes a core design language used across the product.

---

## Scout Card Design `[LOCKED]`

### Card Anatomy

```
┌─────────────────────────────────────────┐
│ ● ACTIVE                    Mammoth     │  ← status dot + mountain list
│                             June Mtn    │
│                                         │
│ FIRST                                   │  ← Scout name (display type)
│ TRACKS                                  │
│                                         │
│ ⚡ Conditions building at Mammoth       │  ← alert signal
│                                         │
│ ─────────────────────────────────────── │
│ ● EPIC INCOMING                         │  ← score badge
│ Chasing corduroy mornings — cold over-  │  ← LLM summary (1-2 lines)
│ night temps, fresh grooming, clear sky. │
└─────────────────────────────────────────┘
```

### Card Elements

**Scout name** — hero display text, uppercase, Helvetica Now Display. User-defined name or LLM-generated suggestion (e.g. FIRST TRACKS, STORM CHASER, POW HUNT, TAHOE TRIP).

**Mountain list** — top right, small muted text, lists the mountains in this Scout. If more than 3, shows count: "8 Ikon resorts".

**Status dot** — top left, alongside status label:
- Active: green LED glow (`#22c55e` with `box-shadow: 0 0 5px #22c55e`)
- Paused: dim dot, muted label
- Scheduled: amber dot

**Alert signal** — below Scout name, one line:
- ⚡ Live signal (amber): *"Conditions building at Mammoth"*
- ◎ Watching (muted): *"Watching 7-day forecast"*
- ○ Inactive (muted): *"Activate to start watching"*

**Score badge** — small pill above summary text:
- `● EPIC INCOMING` — purple
- `● GREAT CONDITIONS` — blue
- `● WATCHING` — neutral/muted

**LLM summary** — 1–2 lines, generated from the Scout's structured preference data. Plain English description of what the Scout is watching for. Separated from badge by a hairline rule. Muted color, small size.

### Card Variants

| Variant | Background | Text | Use |
|---------|-----------|------|-----|
| Light glass | `rgba(255,255,255,0.55)` + blur | Dark ink | Default active Scout |
| Dark | `rgba(10,12,18,0.78)` + blur | White | Alternate, high contrast |
| Add card | Dashed outline, transparent | Muted | Always last in grid |

### LLM-Generated Scout Names `[LOCKED]`

On Scout creation, an LLM generates 3 suggested names based on the user's configured conditions and mountains. User can pick one or type their own. Names should be 1-2 words, uppercase-friendly, ski-culture resonant.

**[WHY]** A well-named Scout ("STORM CHASER") is more emotionally engaging than "My Scout 1". Names users identify with make the product feel personal and alive.

---

## Scout Modal (Card Detail View) `[LOCKED]`

### Layout

When a Scout card is clicked, a modal overlay opens covering ~85% of the screen. The fixed nav bar remains visible above the modal at all times.

**Modal structure:**
- Backdrop: frosted blur overlay (`backdrop-filter: blur(12px)`) with dark wash over the dashboard behind it
- Modal panel: large centered container, rounded corners, scrollable content
- Modal always opens with a smooth scale + fade-in animation (200ms)

### Modal Header (always visible, sticky within modal)

```
[ Scout Name ]                    [Share] [✕]
Mountains: Mammoth · June Mtn · Bear Mtn
● Active  |  Watching 3-day forecast
```

### Modal Content (scrollable)

Full Scout configuration in sections:
1. **Mountains** — search and manage the mountain list for this Scout
2. **Conditions** — full conditions preference panel (weight tiers, thresholds)
3. **Notification Settings** — SMS, email, advance notice window
4. **Schedule** — active dates, days of week
5. **LLM Summary** — editable plain-English paragraph of the Scout's intent

### Modal Actions `[LOCKED]`

| Action | Label | Style |
|--------|-------|-------|
| Save changes | **"Lock it in"** | Primary filled button |
| Discard changes | **"Bail"** | Ghost/text button |
| Share Scout | **"Share"** | Icon + text, top right of header |
| Close without saving | **✕** | Top right corner icon |

**[WHY "Lock it in" / "Bail"]** Ski culture vocabulary in UI copy makes the product feel native to the audience. It's a small touch with outsized brand impact. Every interaction point is an opportunity to reinforce the product's identity.

**Share button:** Opens a share sheet — copy link, send via SMS, or invite a friend to Gnarcast with this Scout pre-configured as a template. (Full sharing spec in `07_Sharing_Social.md`.)

---

## Logged-Out Landing Page `[LOCKED]`

Identical layout to the logged-in dashboard with three differences:
1. Example Scouts replace user's actual Scouts (pre-populated with ski culture names and realistic conditions)
2. Demo banner: *"👋 These are example Scouts. Sign up free to build your own."*
3. Add card CTA reads **"Send out your Scout →"** which triggers sign-up flow

**[WHY same layout]** New visitors immediately understand the product by seeing it in its natural state. No separate "marketing page" — the product is its own best marketing.

**Example Scout names for logged-out page:**
- FIRST TRACKS, STORM CHASER, POW HUNT, BLUEBIRD

---

## Mobile Considerations `[BACKLOG]`

Mobile app design will follow the same visual language — Helvetica Now, purple/green brand colors, card-based layout, video backgrounds. Key adaptations:
- Cards stack vertically (single column) on mobile web
- Modal becomes full-screen bottom sheet on mobile
- Video background optimized for mobile bandwidth (lower bitrate source)
- Touch interactions replace hover — tap to focus card, tap again to open

Full mobile spec to be written when app development begins.

---

## Open Questions

- `[OPEN]` Video sourcing strategy — license, stock, or community? Budget implications?
- `[OPEN]` Mountain-specific video (show Mammoth footage when a Mammoth Scout is active) — feasible? Requires a video library indexed by resort.
- `[OPEN]` Dark vs. light card balance — should the default be all light cards with user control over dark, or mix determined by Gnarcast based on conditions score?
- `[OPEN]` Exact animation spec for modal open/close — scale from card, or slide up from bottom?
