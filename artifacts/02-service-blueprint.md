# Service Blueprint: Auto Leads Platform

> **Project:** auto-leads-platform
> **Version:** 1.0
> **Last updated:** 2026-04-11
> **Status:** Approved

---

## Overview

This blueprint maps the end-to-end service delivery for the Auto Leads Platform — a consumer-facing digital prequalification product for Toyota and Lexus franchise dealers. It covers the full journey from consumer discovery through dealer lead receipt, including identity verification, soft bureau pull, credit policy decisioning, and ADF lead distribution. The service is fully automated at launch with no manual review queue.

---

## Journey Map

| # | Step | Description |
|---|------|-------------|
| 1 | Discovers platform | Consumer clicks prequal CTA on Toyota or Lexus OEM website |
| 2 | Lands on prequal entry | Consumer views branded landing page and initiates application |
| 3 | Creates account | Consumer registers with email and password (mandatory) |
| 4 | Completes form | Consumer enters contact info, income, last 4 SSN, vehicle of interest, and selects preferred dealer |
| 5 | OTP verification | Consumer confirms phone number ownership via SMS one-time code |
| 6 | KBA verification | Consumer confirms identity by answering bureau-sourced identity questions |
| 7 | Reviews & accepts disclosure | Consumer reads and explicitly accepts FCRA soft pull disclosure |
| 8 | Submits application | Application Service orchestrates soft pull and runs credit policy check chain |
| 9 | Receives decision | Consumer sees "Prequalified" (amount + rate range) or "Unable to complete your pre-qualification" |
| 10 | Receives email summary | Consumer receives branded email with full result, confirmation number, and selected dealer info |
| 11 | Dealer receives ADF lead | Lead Distribution Service delivers ADF-formatted lead to selected dealer's endpoint |
| 12 | Dealer follows up | Dealer contacts consumer directly via their preferred channel |

---

## Blueprint Layers

### Layer 1 — Physical Evidence
*What the customer sees, touches, or interacts with at each step.*

| Step | Physical Evidence |
|------|-----------------|
| 1. Discovers platform | Prequal CTA (banner, button, or interstitial) on Toyota.com / Lexus.com |
| 2. Lands on prequal entry | Branded Next.js landing page with product explainer and "Get Prequalified" CTA |
| 3. Creates account | Registration form (email, password); account confirmation message |
| 4. Completes form | Multi-step form: contact info, income, last 4 SSN; NHTSA-powered vehicle dropdowns (year/make/model/trim); dealer selector showing nearest locations |
| 5. OTP verification | OTP entry screen; SMS message with verification code |
| 6. KBA verification | Identity questions screen (multiple choice, bureau-sourced); response timer |
| 7. Reviews & accepts disclosure | Full FCRA disclosure rendered; explicit accept/consent button |
| 8. Submits application | Loading state with progress indicator |
| 9. Receives decision | Decision screen: "Prequalified — up to $[amount] at [rate range]%" or "Unable to complete your pre-qualification" |
| 10. Receives email summary | Branded transactional email: decision, max finance amount, rate range, confirmation number, selected dealer name and contact info |
| 11. Dealer receives ADF lead | Not visible to consumer |
| 12. Dealer follows up | Phone call or email from dealer (channel determined by dealer) |

---

### Layer 2 — Customer Actions
*What the customer actively does at each step.*

| Step | Customer Action |
|------|----------------|
| 1. Discovers platform | Clicks prequal CTA on Toyota.com or Lexus.com |
| 2. Lands on prequal entry | Reads product explainer; clicks "Get Prequalified" |
| 3. Creates account | Enters email and password; submits registration |
| 4. Completes form | Enters contact info, income, last 4 SSN; selects vehicle year/make/model/trim via NHTSA-powered dropdowns; selects preferred dealer from nearby list |
| 5. OTP verification | Receives SMS; enters OTP code on screen |
| 6. KBA verification | Answers 3–5 bureau-sourced identity questions within time limit |
| 7. Reviews & accepts disclosure | Reads FCRA disclosure; clicks accept to consent to soft pull |
| 8. Submits application | Clicks submit; waits for decision |
| 9. Receives decision | Reads decision; views max finance amount and rate range (if prequalified) |
| 10. Receives email summary | Opens email; reviews result details and dealer contact info |
| 11. Dealer receives ADF lead | No customer action |
| 12. Dealer follows up | Receives dealer outreach; decides whether to visit |

**— LINE OF INTERACTION —**

---

### Layer 3 — Frontstage Actions
*What the system does that the customer directly experiences.*

| Step | Frontstage Action | Owner |
|------|------------------|-------|
| 1. Discovers platform | OEM website renders prequal CTA | Toyota/Lexus OEM (external) |
| 2. Lands on prequal entry | Next.js app loads; landing page rendered via CloudFront | Frontend |
| 3. Creates account | Registration inputs validated in real-time; account creation confirmed | Identity & Auth Service |
| 4. Completes form | Real-time field validation; NHTSA vehicle dropdowns populate dynamically; dealer list rendered by geolocation | Frontend + Vehicle Service + Application Service |
| 5. OTP verification | OTP prompt displayed; SMS dispatched; code validated on submission | Identity & Auth Service |
| 6. KBA verification | Identity questions rendered with timer; pass/fail determined and recorded | Bureau Integration Service |
| 7. Reviews & accepts disclosure | FCRA disclosure rendered in full; accept button requires explicit interaction | Frontend |
| 8. Submits application | Loading state shown while decisioning runs | Frontend + Application Service |
| 9. Receives decision | Decision screen rendered: prequalified result with amount and rate range, or unable to complete message | Application Service → Frontend |
| 10. Receives email | Branded email delivered with full result details | Notification Service |
| 11. Dealer receives lead | Not visible to consumer | — |
| 12. Dealer follows up | Not visible to consumer | — |

**— LINE OF VISIBILITY —**

---

### Layer 4 — Backstage Actions
*What happens invisibly to deliver the frontstage experience.*

| Step | Backstage Action | Owner |
|------|-----------------|-------|
| 1. Discovers platform | — | — |
| 2. Lands on prequal entry | Session initialized; analytics event fired | Frontend + Analytics |
| 3. Creates account | Account record created in DB; session token issued | Identity & Auth Service + RDS |
| 4. Completes form | NHTSA API queried for vehicle data; dealer list queried by geolocation; form state persisted to DB | Vehicle Service + Application Service + RDS |
| 5. OTP verification | OTP generated and stored with TTL; SMS dispatched via provider; submitted code validated against stored value; expiry enforced; max resend attempts enforced | Identity & Auth Service |
| 6. KBA verification | KBA questions fetched from bureau; consumer responses submitted to bureau for scoring; pass/fail result returned and persisted; failure terminates application and triggers adverse action record | Bureau Integration Service + RDS |
| 7. Reviews & accepts disclosure | Disclosure acceptance timestamped and persisted with application record in DB | Application Service + RDS |
| 8. Submits application | Application Service orchestrates sequential policy check chain: (1) Identity verification confirmed — OTP + KBA both passed; (2) SSN last-4 matches bureau record; (3) Minimum credit score threshold met; (4) No disqualifying derogatory indicators (bankruptcy, charge-off, policy-defined exclusions); (5) Stated income meets minimum threshold; (6) Selected vehicle is new Toyota or Lexus. All checks must pass for approval. Result persisted; confirmation number generated; `prequalification.completed` event emitted to EventBridge | Application Service + Bureau Integration Service + RDS + EventBridge |
| 9. Receives decision | Decision result read from DB and passed to frontend; if unable to complete — adverse action record created and stored | Application Service + RDS |
| 10. Receives email | Notification Service consumes `prequalification.completed` event; renders email template with result data; dispatches via email provider | Notification Service + SES/SendGrid |
| 11. Dealer receives lead | Lead Distribution Service consumes `prequalification.completed` event; packages consumer data, vehicle intent, and prequal result into ADF format; delivers to selected dealer's endpoint only — no routing fallback | Lead Distribution Service |
| 12. Dealer follows up | — | — |

**— LINE OF INTERNAL INTERACTION —**

---

### Layer 5 — Support Processes
*Systems, tools, third-party services, and policies enabling backstage actions.*

| Step | Support Process | Type |
|------|----------------|------|
| 2. Lands on prequal entry | CloudFront + S3 (static asset delivery); CloudWatch (monitoring + alerting) | Infrastructure |
| 3. Creates account | Amazon Cognito or Auth0 (identity management); RDS (account storage) | Infrastructure |
| 4. Completes form | NHTSA API (vehicle year/make/model data — free, public); RDS (dealer list + geolocation); API Gateway | Third-party + Infrastructure |
| 5. OTP verification | SMS provider (Twilio or Amazon SNS); OTP rate limiting and lockout policy; OTP TTL per Cybersecurity SME requirements | Third-party + Policy |
| 6. KBA verification | Bureau KBA API (Experian or Equifax); FCRA permissible purpose policy for identity pull | Third-party + Policy |
| 7. Reviews & accepts disclosure | FCRA compliance policy; disclosure content Legal-SME-approved; consent audit log retained per FCRA requirements | Policy |
| 8. Submits application | Bureau soft pull API (Experian or Equifax); credit policy ruleset (policy-defined thresholds and exclusions); ECS/Fargate (Application + Decisioning services); EventBridge (event bus); SQS (event delivery); RDS (application + result storage); FCRA adverse action policy | Third-party + Infrastructure + Policy |
| 9. Receives decision | RDS (decision read); FCRA adverse action notice policy; CloudWatch | Infrastructure + Policy |
| 10. Receives email | SES or SendGrid; Notification Service on ECS/Fargate; Legal-SME-approved email template | Third-party + Infrastructure + Policy |
| 11. Dealer receives lead | Lead Distribution Service on ECS/Fargate; ADF format specification; dealer endpoint (email or webhook); SQS retry queue (2 retries); SQS dead letter queue after retry exhaustion | Infrastructure + Policy |

---

## Failure Paths

| Step | Failure Mode | Customer Experience | System Response | Owner |
|------|-------------|--------------------|-----------------|----|
| 2. Lands on prequal entry | App / CloudFront unavailable | Error page or blank screen | CloudWatch alarm; on-call notified | Platform / DevOps |
| 3. Creates account | Duplicate email | Inline error; prompted to log in | No duplicate account created | Identity & Auth Service |
| 3. Creates account | Validation error (weak password, invalid email) | Inline field error | User prompted to correct | Frontend |
| 4. Completes form | NHTSA API unavailable | Vehicle dropdowns fail to populate; error state shown | Cached vehicle data served if available; CloudWatch alarm if persistent | Vehicle Service |
| 4. Completes form | Dealer list fails to load | Dealer selector shows error; cannot proceed | 3 retries; CloudWatch alarm if persistent | Application Service |
| 5. OTP verification | SMS not delivered | Prompted to resend; resend limit enforced | OTP regenerated on resend; max 3 resend attempts; lockout after limit | Identity & Auth Service |
| 5. OTP verification | OTP expired | Expiry message shown; prompted to resend | OTP invalidated after TTL; new OTP issued on resend | Identity & Auth Service |
| 5. OTP verification | Max resend attempts exceeded | Lockout message shown | Session terminated; lockout logged | Identity & Auth Service |
| 6. KBA verification | Consumer fails KBA questions | "Unable to complete your pre-qualification" | KBA failure recorded; application terminated; adverse action record created | Bureau Integration Service |
| 6. KBA verification | Bureau KBA API timeout (>10 seconds) | "Unable to complete your pre-qualification" | Timeout recorded; CloudWatch alarm | Bureau Integration Service |
| 8. Submits application | Bureau soft pull timeout (>10 seconds) | "Unable to complete your pre-qualification" | 1 retry; timeout after 10 seconds; failure recorded; CloudWatch alarm | Bureau Integration Service |
| 8. Submits application | Bureau soft pull API error | "Unable to complete your pre-qualification" | Error recorded; adverse action record created; CloudWatch alarm | Bureau Integration Service |
| 8. Submits application | Credit policy check fails | "Unable to complete your pre-qualification" | Specific fail reason recorded internally (not surfaced to customer); adverse action record created | Application Service |
| 8. Submits application | Decisioning service unavailable | "Unable to complete your pre-qualification" | Circuit breaker triggered; CloudWatch alarm; on-call notified | Application Service |
| 8. Submits application | EventBridge event fails to publish | Customer sees decision (unaffected) | Email and ADF lead will not trigger; SQS dead letter queue captures event; CloudWatch alarm; manual recovery required | Application Service |
| 10. Receives email | Email delivery failure | Consumer does not receive summary | SES/SendGrid retry attempted; CloudWatch alarm if undelivered | Notification Service |
| 11. Dealer receives ADF lead | ADF delivery fails — dealer endpoint unavailable | No customer impact | SQS retry queue — 2 retries; dead letter queue after retry exhaustion; CloudWatch alarm; Dealer Relations team notified | Lead Distribution Service |
| 11. Dealer receives ADF lead | Dealer endpoint invalid or not enrolled | No customer impact | Lead rejected at delivery; dead letter queue; CloudWatch alarm; Dealer Relations team notified | Lead Distribution Service |

---

## Human-in-the-Loop Steps

This service is fully automated at launch. Human intervention is limited to:

| Trigger | Human Action Required | Owner | SLA |
|---------|----------------------|-------|-----|
| CloudWatch alarm (any service) | Incident triage and resolution | On-call engineer | Per incident response runbook |
| ADF lead in dead letter queue | Manual lead recovery or dealer outreach | Dealer Relations team | TBD — establish at launch |
| Dealer endpoint invalid | Dealer enrollment correction | Dealer Relations team | TBD — establish at launch |

---

## Timing & SLA Requirements

| Step | Requirement | Target | Failure Behavior |
|------|------------|--------|-----------------|
| 8. Submits application | End-to-end prequal (form submit → decision displayed) | <5 seconds (P95) | Bureau timeout triggers "unable to complete" at 10 seconds |
| 6. KBA verification | Bureau KBA API response | <10 seconds | Timeout → "unable to complete" |
| 8. Submits application | Bureau soft pull API response | <10 seconds | 1 retry; timeout → "unable to complete" |
| 10. Receives email | Email delivered after `prequalification.completed` event | Within 5 minutes | SES/SendGrid retry; CloudWatch alarm |
| 11. Dealer receives lead | ADF delivered after `prequalification.completed` event | Within 60 seconds | 2 SQS retries; dead letter queue; Dealer Relations notified |

---

## Regulatory Touchpoints

| Step | Regulation | Requirement | Implementation Note |
|------|-----------|-------------|---------------------|
| 6. KBA verification | FCRA | Permissible purpose required for bureau identity pull | Legal SME to confirm whether single FCRA disclosure covers both KBA pull and soft pull, or whether separate disclosure is required |
| 7. Reviews & accepts disclosure | FCRA | Full soft pull disclosure must be presented and explicitly accepted before any bureau pull executes | Acceptance timestamped and stored; accept button requires explicit interaction; cannot be pre-checked |
| 9. Receives decision | FCRA | Adverse action notice required when "unable to complete" is driven by bureau data | Must include: right to free credit report, right to dispute, CRA name and contact info; Legal SME to confirm "unable to complete" language satisfies FCRA §615 |
| 10. Receives email | GLBA | Privacy notice required for consumers whose NPI is collected | Privacy policy must be live before first consumer interaction; email content requires Legal SME sign-off |
| 11. Dealer receives ADF lead | GLBA | Data sharing rules apply to NPI included in ADF lead payload | Legal SME to confirm which consumer data fields are permissible to include in ADF lead under GLBA data sharing rules |

---

## Key Integration Points

| Integration | Purpose | Step(s) | Failure Impact |
|------------|---------|---------|---------------|
| Toyota.com / Lexus.com | Consumer entry point — hosts prequal CTA | 1 | No traffic to platform if OEM CTA is removed or broken |
| NHTSA API | Vehicle year/make/model/trim data for form dropdowns | 4 | Vehicle selector fails; form cannot be completed |
| SMS Provider (Twilio / Amazon SNS) | OTP delivery to consumer phone | 5 | Consumer cannot complete identity verification |
| Bureau KBA API (Experian / Equifax) | Identity verification via knowledge-based authentication | 6 | All applications fail identity check; platform non-operational |
| Bureau Soft Pull API (Experian / Equifax) | Credit data for prequal decisioning | 8 | All applications return "unable to complete"; platform non-operational |
| Amazon EventBridge | Event bus connecting Application Service to Notification and Lead Distribution services | 8, 10, 11 | Email and ADF lead delivery fail if event not published |
| SES / SendGrid | Consumer email delivery | 10 | Consumer does not receive result summary |
| Dealer ADF Endpoint | Lead delivery to selected dealer | 11 | Lead not received by dealer; dead letter queue and manual recovery |

---

## Open Questions

| # | Question | Owner | Needed By |
|---|---------|-------|-----------|
| 1 | Does a single FCRA disclosure cover both the KBA bureau pull and the soft pull, or are two separate disclosures required? | Legal SME | Before UX design of disclosure screen |
| 2 | Does "unable to complete your pre-qualification" satisfy FCRA §615 adverse action notice requirements, or is additional content/delivery required? | Legal SME | Before UX design of decision screen |
| 3 | Which consumer data fields are permissible to include in the ADF lead payload under GLBA data sharing rules? | Legal SME | Before Lead Distribution Service spec |
| 4 | What is the OTP TTL and maximum resend attempt policy? | Cybersecurity SME | Before Identity & Auth Service development |
| 5 | What is the SLA for Dealer Relations team response to leads in the dead letter queue? | Dealer Relations / Operations | Before launch |

---

## Changelog

This document is a living record. Scope changes, flow revisions, and integration decisions that affect a customer-facing step, a system lane, or a service interaction are logged here. Minor implementation details that do not alter the blueprint structure are captured in the decisions log only.

| Date | Section affected | Change | Reason | Ref |
|------|-----------------|--------|--------|-----|
| 2026-04-11 | Open Questions 1–3 | Q1, Q2, Q3 resolved via Legal SME consultation | FCRA disclosure, adverse action, and ADF payload questions answered | decisions-log 2026-04-11 |
| 2026-04-11 | Open Questions 4 | Q4 resolved via Cybersecurity SME consultation | OTP policy defined: 10-min TTL, max 5 attempts, max 3 resends, 30-min lockout | decisions-log 2026-04-11 |
| 2026-04-11 | Supporting Systems — SES/SendGrid | Email provider updated to Amazon SES | AWS-native; no additional GLBA data processor | decisions-log 2026-04-11 |
| 2026-04-11 | Consumer flow — Step 7 | FCRA disclosure confirmed as single combined conditional disclosure; disclosure screen names selected dealer | Legal SME: GLBA §1016.15(a)(1) consent requirement | decisions-log 2026-04-11 |
| 2026-04-11 | Consumer flow — Step 8 | Review screen added before FCRA disclosure; vehicle and dealer editable; identity fields locked | UX Designer + Legal SME: Review screen is designated correction point | decisions-log 2026-04-11 |
| 2026-04-11 | Lead Distribution Service | ADF delivery: PREQUALIFIED outcomes only; DealerTrack + RouteOne both stubbed for this project | Legal SME + Product Manager: live onboarding deferred | decisions-log 2026-04-11 |
| 2026-04-11 | Lead Distribution Service | Dual delivery confirmed: send to both DealerTrack and RouteOne if dealer enrolled in both | Product Manager | decisions-log 2026-04-11 |
| 2026-04-11 | Consumer flow — Decision screen | Prequal expiration: 30 days from decision date | Product Manager | decisions-log 2026-04-11 |

---

*← [Auto Leads Platform — Progress](../progress.md)*
