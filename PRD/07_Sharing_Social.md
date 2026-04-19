# Gnarcast PRD — Sharing & Popular Setups
> Sub-spec 07 of 08 | Status: **LOCKED (with open questions)**
> Related specs: `02_User_Preferences.md` (Scout creation flow), `03_Look_Feel.md` (card design)

---

## Overview

Gnarcast's sharing features are intentionally focused — this is not a social platform. There is no feed, no followers, no public profiles. Sharing exists for two specific purposes:

1. **Sharing a conditions alert** — telling a friend the mountain is firing and inviting them to come
2. **Sharing a Scout setup** — sending someone your exact Scout configuration so they can use it as a starting point

A third related feature — **Popular Scout Setups** — is not a social feature at all but a Scout creation assist: when a user creates a Scout for a known mountain, they can see the most common configurations from other users and start from one rather than from scratch.

[WHY] A social feed creates moderation overhead, trust & safety requirements, and product complexity that isn't core to the value proposition. The value Gnarcast delivers is personalized conditions intelligence — sharing amplifies that value without building a social network.

---

## Feature 1: Share a Conditions Alert

### What it is

When a Scout fires and sends an alert, the user can share a **conditions card** with anyone — via iMessage, WhatsApp, or any share target — that communicates why the day is great without requiring the recipient to have a Gnarcast account.

### Trigger points

Sharing can be initiated from:
- The SMS alert itself — a short link included at the bottom of every alert: "Conditions looking epic → share: gnar.st/c/[token]"
- The Scout card in the app — a share button on any Scout card that has fired recently
- The Scout detail modal — share button in the header

### What the conditions card contains

The shared card is a rich-preview URL (Open Graph tags) that renders a visual snapshot when shared to iMessage, WhatsApp, Slack, etc.:

- **Resort name + date**
- **Score tier** (Epic / Great / Good) with the color-coded badge
- **Top 3 conditions that drove the score** — e.g., "12" new snow · Bluebird · 95% terrain open"
- **Avalanche danger level** (if relevant)
- **Road conditions** — "SR-410: Clear" or "Chain control in effect"
- **Gnarcast branding** — logo, tagline

The card is read-only and requires no login to view. At the bottom of the card page: "Want alerts like this for your mountains? → gnarcast.com" — passive acquisition, not aggressive.

### Link structure

```
gnar.st/c/[token]
```

- Short link redirects to `gnarcast.com/conditions/[token]`
- Token is a unique, time-limited identifier (valid for 7 days from alert time)
- The underlying conditions snapshot is stored at alert time — the card shows the conditions as they were when the alert fired, not the current conditions

[WHY] Storing the snapshot means the card is accurate even if conditions change after sharing. "This is what it looked like when the alert fired" is more honest than a live view that may have already changed.

### Privacy

- Shared cards do not reveal the sender's identity, their Scout name, or their Scout configuration
- The recipient sees only: resort, date, conditions snapshot, score tier
- No Gnarcast account required to view

---

## Feature 2: Share a Scout Setup

### What it is

A user can share their Scout configuration as a link. The recipient opens the link and sees the Scout's conditions, weights, and mountain — they can import it as a starting point for their own Scout and customize from there.

### How it works

1. User opens a Scout and taps "Share this setup" (in the Scout detail modal)
2. A shareable link is generated: `gnar.st/s/[token]`
3. User shares the link via any channel (iMessage, WhatsApp, email, etc.)
4. Recipient opens the link — sees a read-only preview of the Scout configuration:
   - Mountain(s) included
   - Conditions and their weights (Dealbreaker / Send It / Nice to Have / etc.)
   - Sensitivity dial setting (Epic / Great / Good)
   - LLM plain-English summary ("Alerts you on powder days with light crowds at Crystal")
5. Recipient taps "Use this setup" — if they have a Gnarcast account, the Scout is imported into their account as a new Scout (editable, not linked to the original). If they don't have an account, they're taken to signup with the Scout pre-queued to import after signup.

### What is shared vs. private

| Element | Shared | Private |
|---------|--------|---------|
| Mountain(s) | ✅ | |
| Conditions + weights | ✅ | |
| Sensitivity setting | ✅ | |
| Scout name | ✅ (can be changed on import) | |
| Alert history | | ✅ Never shared |
| Sender identity | | ✅ Never shared |
| Sender phone number | | ✅ Never shared |

### Link structure

```
gnar.st/s/[token]
```

- Persistent link (does not expire — the Scout configuration doesn't change unless the owner changes it)
- If the owner deletes the Scout or makes it private, the link returns a "Scout no longer available" page
- Owner can revoke a shared link at any time from the Scout detail modal

[OPEN] Should Scout sharing be on by default (any Scout can be shared) or opt-in (user must explicitly enable sharing per Scout)? Default-on is simpler; opt-in is more privacy-forward.

---

## Feature 3: Popular Scout Setups

### What it is

When a user creates a new Scout for a specific mountain, Gnarcast surfaces the most common configurations used by other users at that same mountain — as optional starting points. This is a **Scout creation assist**, not a social feature.

[WHY] The scoring system is powerful but has a learning curve. A new user staring at 30 conditions and 6 weight tiers may not know where to start. Popular setups from real users who ski the same mountain turn the configuration step from intimidating to intuitive.

### Where it appears

During Scout creation, after the user selects a mountain (or set of mountains), a section appears above the blank Scout builder:

> **Popular setups at Crystal Mountain**
> Based on how other riders have configured their Scouts
>
> [Pow Day]  [Groomers + Blue Bird]  [Park Day]
> Start from scratch →

Tapping a popular setup pre-fills the Scout builder with that configuration. The user can adjust any condition weight before saving — the popular setup is a starting point, never locked in.

### How popular setups are computed

- Aggregate Scout configurations for a given mountain across all users with active Scouts
- Group by similarity (clustering on condition weights) to identify the most common "archetypes"
- Surface the top 3 archetypes, each given an auto-generated name via LLM (same LLM summary mechanism from `02_User_Preferences.md`)
- Minimum threshold: a mountain needs at least X active Scouts before popular setups are shown (prevents a single user from defining "popular")

[OPEN] Minimum Scout count threshold before popular setups activate — 10? 25? 50? Calibrate at launch based on actual user volumes.

### Privacy guarantees

- Popular setups are **fully anonymized and aggregated** — no individual user's Scout is ever shown or attributed
- The recipient sees condition weights and patterns, not any personal data
- "Popular at Crystal" means "the cluster centroid of configurations at Crystal" — not any specific person's Scout

### Auto-generated names

Each archetype gets a short name generated by the LLM based on its dominant conditions:
- High new snow weight + low crowd weight → "Storm Chaser"
- High grooming weight + bluebird weight → "Groomers & Blue Bird"
- Park-specific conditions weighted high → "Park Day"
- High terrain open % + weekend scheduling → "Full Mountain"

Names are suggestions — the user can rename their Scout after importing.

---

## Acquisition Hook

Both sharing features (conditions card + Scout setup) include a passive acquisition path for non-Gnarcast users:

- **Conditions card page** — "Get alerts like this for your mountains" CTA at the bottom, links to signup
- **Scout setup preview page** — "Use this setup" requires an account; tapping it routes to signup with the Scout pre-queued to import

Neither CTA is aggressive — the shared content is the product, not a gate. The goal is that a non-user who sees a compelling conditions card is curious enough to sign up, not pressured to.

---

## What This Is Not

To be explicit about scope:

- ❌ No social feed or public activity stream
- ❌ No follower/following relationships
- ❌ No public profiles
- ❌ No comments or reactions on conditions
- ❌ No leaderboards or gamification
- ❌ No group Scouts or shared alert subscriptions (backlog)

[WHY] Each of these features adds trust & safety requirements, moderation overhead, and product complexity. Gnarcast's core value is personalized conditions intelligence — social features that don't serve that goal are deferred until the core is proven.

---

## Open Questions

- `[OPEN]` Scout sharing opt-in vs. default-on — privacy tradeoff
- `[OPEN]` Minimum active Scout count before popular setups activate for a mountain
- `[OPEN]` Should the conditions card include a live "current conditions" view in addition to the snapshot? Two panels: "When the alert fired" + "Right now"
- `[OPEN]` Group alerts / shared Scouts — multiple people subscribed to the same Scout — is this worth building? Useful for ski groups and couples. Captured in backlog.

---

## Backlog

- `[BACKLOG]` Group Scouts — multiple users subscribed to a single shared Scout, all get the same alert
- `[BACKLOG]` "Who's going" — lightweight opt-in feature where users can indicate they're going to a mountain on a specific day, visible only to people they've shared their Scout with
- `[BACKLOG]` Conditions card as an image (PNG/JPG) — shareable to Instagram Stories, not just as a link preview
- `[BACKLOG]` "Invite a friend" referral program — share link that gives both users a benefit (free premium month, etc.)
- `[BACKLOG]` Popular setups for multi-mountain Scouts — harder to cluster, deferred
