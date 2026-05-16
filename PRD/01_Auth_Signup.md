# Gnarcast — Auth & Signup Spec

> Sub-spec 01 | Status: **LOCKED (with open questions)** | Last Updated: 2026-05-16

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

**[WHY deferred]** Adding OAuth before a mobile app exists creates configuration and maintenance overhead without user benefit. Using Supabase Auth from day one means OAuth can be added with minimal engineering effort when the time comes.

### Account Linking `[LOCKED]`

User profiles will include a "Connected Accounts" section from day one (initially empty). When mobile launches, users who signed up on web via phone can link their Google or Apple accounts for easier mobile sign-in. If a social account's email matches the one on file, the system prompts to link automatically.

**[WHY]** Without account linking, a web user who tries Apple Sign In on the app with a different email could create a duplicate account. Building the "connected accounts" hook early prevents this at low cost.

### Auth Provider `[LOCKED]`

**Supabase Auth** (decided in `06_Tech_Stack.md`).

Rationale recap: Gnarcast needs both auth and a database, and Supabase delivers both in one integrated platform. Clerk was the runner-up — better polished UI components and DX, but auth-only, which would have meant a separate database vendor, a JWT bridge, and two pricing models to manage. Supabase Auth supports phone/SMS OTP via Twilio, OAuth providers, row-level security, and session management — everything specced above. Migrating from local Supabase to Supabase Cloud requires zero code changes. See `06_Tech_Stack.md` for the full decision record.

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

## Scouts (formerly "Named Alert Profiles") `[LOCKED — see 02_User_Preferences.md]`

The "Named Alert Profiles" concept originally captured here evolved into **Scouts** — Gnarcast's first-class personalization primitive. A Scout is a named, self-contained configuration consisting of a mountain set, condition preferences with weights and thresholds, notification settings, a schedule, and a status (active/paused/scheduled/archived).

The default Scout created during onboarding is named "Home Mountain." Users can create unlimited additional Scouts (e.g., a "Tahoe Trip" Scout with exceptional-conditions-only thresholds across multiple Tahoe resorts, or a "Powder Emergency" Scout watching every monitored mountain for 12"+ overnight events).

Full spec: see `02_User_Preferences.md`.

---

## Open Questions

- ✅ ~~Auth provider final decision~~ — **Supabase Auth** (locked in `06_Tech_Stack.md`)
- `[OPEN]` Should email verification be blocking (must verify before proceeding) or async (can use app, nudged to verify)? — resolve in `10_User_Profile.md` or `14_Security_Compliance.md`
- `[OPEN]` Exact lookahead window options for notifications — fixed options or fully user-configurable? — resolve in `09_Notifications.md`
