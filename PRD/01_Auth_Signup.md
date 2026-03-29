# Gnarcast — Auth & Signup Spec

> **Status:** In Progress | **Last Updated:** 2026-03-29

---

## Overview

The signup and authentication flow is the entry point to Gnarcast's personalized experience. The design goal is to simultaneously verify identity, enable SMS notifications, and capture the minimum information needed to deliver value — all in under 2 minutes.

---

## Authentication

### Primary Auth: Phone/SMS OTP `[LOCKED]`

Users enter their phone number and receive a 6-digit SMS verification code to log in. No password required.

**[WHY]** Phone verification solves three problems at once: identity verification, bot protection, and SMS notification opt-in. For a time-sensitive alert product targeting mobile-first skiers, SMS is the highest-value notification channel (98% open rate vs ~20% for email). Passwordless removes friction and eliminates a common drop-off point at signup.

### Secondary Auth: Google + Apple OAuth `[LOCKED — deferred to mobile launch]`

Google and Apple Sign In will be added when the mobile app ships.

**[WHY]** Apple Sign In is a mandatory App Store requirement if any social auth is offered on iOS. Both providers give verified email addresses for free. Deferring to mobile launch avoids unnecessary complexity on web while keeping the path clear for future implementation.

**[WHY deferred]** Adding OAuth before a mobile app exists creates configuration and maintenance overhead without user benefit. Using Clerk or Supabase Auth from day one means OAuth can be added with minimal engineering effort when the time comes.

### Account Linking `[LOCKED]`

User profiles will include a "Connected Accounts" section from day one (initially empty). When mobile launches, users who signed up on web via phone can link their Google or Apple accounts for easier mobile sign-in. If a social account's email matches the one on file, the system prompts to link automatically.

**[WHY]** Without account linking, a web user who tries Apple Sign In on the app with a different email could create a duplicate account. Building the "connected accounts" hook early prevents this at low cost.

### Auth Provider `[OPEN]`

Decision deferred to Tech Stack discussion. Finalists:

- **Clerk** — dedicated auth service, best-in-class DX, polished pre-built UI, web + mobile SDKs, 50K MAU free tier, ~$20/mo Pro. Requires a JWT bridge if Supabase is used as the database.
- **Supabase Auth** — auth included in the Supabase platform (database + storage + functions), free, deeply integrated with Row Level Security. Less polished UI components but fully capable. Best choice if Supabase is the backend.

---

## Bot Protection `[LOCKED]`

Phone verification is the primary bot protection mechanism. No separate CAPTCHA needed.

**[WHY]** CAPTCHA adds friction for legitimate users while phone OTP simultaneously serves as verification, notification setup, and spam prevention. The cost of an SMS per signup (~$0.008 via Twilio) is an acceptable anti-bot tax.

---

## User Identity `[LOCKED]`

Users create a **username** during onboarding. This becomes their public identity for future social/find-a-friend features. Auth credentials (phone, email) remain private.

**[WHY]** Keeping phone numbers private while enabling social discovery requires a separate public identifier. Building the username field at signup avoids a disruptive retrofit later and ensures every user has a social identity before social features ship.

---

## SMS Compliance (TCPA) `[LOCKED]`

**Required consent language at signup (cannot be pre-checked):**
> "By entering your phone number you agree to receive SMS alerts from Gnarcast. Message frequency varies. Message & data rates may apply. Reply STOP to unsubscribe, HELP for help."

**Required confirmation SMS on opt-in:**
> "You're in! Gnarcast alerts are on. Reply STOP anytime to unsubscribe."

**Additional requirements:**
- STOP must immediately halt all SMS to that number
- HELP must return a support contact
- Consent must be logged: timestamp, IP address, exact consent language shown
- Transactional vs. marketing classification: treat all SMS as marketing-compliant to be safe

**[WHY]** TCPA violations carry fines of $500–$1,500 per message. Compliance is non-negotiable and must be built in from day one, not retrofitted. Phone verification as the primary auth method means this consent is collected at the most natural moment in the user journey.

**Recommended SMS provider: Twilio** — handles opt-out management (STOP/HELP) natively, provides delivery receipts, ~$0.0079/SMS in the US.

---

## Email `[LOCKED]`

Email is collected at signup (either via OAuth or manually entered). Used for:
- Account verification (email verification link required before account activation)
- Secondary notification channel (longer-form content, weekly forecasts, account management)
- Account recovery

**Recommended email provider: Resend or Postmark** for transactional email. Both have excellent deliverability and free tiers. Avoid SendGrid for transactional email (deliverability has degraded).

**[WHY]** Even in an SMS-forward product, email is necessary for regulatory compliance (CAN-SPAM), account recovery, and users who prefer email notifications. Collecting it at signup avoids a secondary ask later.

---

## Onboarding Flow `[LOCKED — detail pass pending]`

The goal: user experiences core product value before leaving the signup screen. All steps completable in under 2 minutes.

| Step | Required | Notes |
|------|----------|-------|
| 1. Phone number entry + OTP | Yes | TCPA consent captured here |
| 2. Email entry + verification | Yes | Verification link sent; user can proceed and verify async |
| 3. Username creation | Yes | Public identity for future social features |
| 4. Mountain selection | Yes | Searchable list, 1–5 mountains to start. Sorted by proximity using browser geolocation (with zip code fallback if denied). This is the core value hook — required to proceed. |
| 5. Ski pass selection | No (skip allowed) | Ikon, Epic, Indy, none, other. Filters mountains by access. |
| 6. Condition preferences | No (skip allowed) | Lightweight version here — full customization in profile. Options: fresh snow, low wind, low crowds, visibility, etc. |
| 7. Notification preferences | No (skip allowed) | SMS, email, or both. Lookahead window (1-day, 3-day, 7-day). |

**[WHY mountain selection is required]** Without at least one mountain selected, Gnarcast cannot deliver any value. Every other step can be defaulted or skipped — mountain selection cannot.

**[WHY geolocation for sorting]** Web geolocation provides sufficient accuracy (city/region level) to sort mountains by proximity. Graceful fallback to zip code entry or popularity sort if user denies location permission.

**`[BACKLOG]`** Deep-dive pass needed on each onboarding step: exact copy, field validation, error states, empty states, and loading states.

---

## Named Alert Profiles `[BACKLOG — queued for User Preferences spec]`

Rather than a single set of notification preferences, users can create multiple named **Alert Profiles** — each with its own mountain set, condition thresholds, and notification settings.

**Default profile created at onboarding:** "Home Mountain"

**Example use cases:**
- *Home Mountain* — local resort, aggressive thresholds, SMS + email, 3-day lookahead
- *Tahoe Trip* — multiple Tahoe resorts, exceptional conditions only (worth the drive), 7-day lookahead
- *Powder Emergency* — any monitored mountain, 12"+ overnight only, SMS, day-of alert only

This feature shapes the data model for user preferences. Full spec to be written in `02_User_Preferences.md`.

**[WHY]** A single profile forces users to choose between monitoring different mountain sets at different thresholds. Named profiles accommodate the reality of how skiers actually plan — home mountain logic is different from destination trip logic.

---

## Open Questions

- `[OPEN]` Auth provider final decision: Clerk vs. Supabase Auth (deferred to tech stack)
- `[OPEN]` Should email verification be blocking (must verify before proceeding) or async (can use app, nudged to verify)?
- `[OPEN]` Exact lookahead window options for notifications — fixed options or fully user-configurable?
