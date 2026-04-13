# Product Requirements Document (PRD)

> **Project:** auto-leads-platform
> **Status:** Complete
> **Last updated:** 2026-04-11

---

## Changelog

This document is a living record. Once finalized, scope changes, requirement updates, and constraint amendments are logged here rather than silently rewriting original content. References to the decisions log are included for full rationale.

| Date | Section affected | Change | Reason | Ref |
|------|-----------------|--------|--------|-----|
| 2026-04-11 | Document created | Initial stub; full PRD in progress | — | — |
| 2026-04-11 | All sections | Full PRD written; synthesizes all locked decisions from discovery, SME consultations, and design phase artifacts | Phase 2 Design & Definition complete | decisions-log.md, all artifacts in artifacts/ |
| 2026-04-11 | §1.5, §2.1, §3 Step 3, §4 (new §4.0 Brand / Skin) | Brand skin toggle incorporated: Toyota/Lexus dual-brand support, vehicle make filtering by active brand skin, shared credit policy across brands, MVP toggle vs production URL-driven brand selection | New product decision 2026-04-11 | decisions-log.md entries 2026-04-11 (brand skin toggle + NHTSA make filter) |

---

## 1. Product Overview

### 1.1 What This Platform Is

The Auto Leads Platform is a consumer-facing soft-pull prequalification product for new Toyota and Lexus vehicles. Consumers complete a guided multi-step application, pass identity verification (OTP phone verification and KBA identity confirmation), accept an FCRA soft pull disclosure naming their selected dealer, and receive an instant prequalification estimate — including an estimated APR range and estimated maximum finance amount — within a single session.

Prequalified consumers generate a structured ADF lead delivered to their selected dealer's DMS platform (DealerTrack and/or RouteOne). The dealer then contacts the consumer directly to complete the purchase and arrange financing.

The platform is a lead distribution system. It connects consumers with dealers. It does not originate, underwrite, fund, or service any loan.

### 1.2 What This Platform Is Not

The platform is explicitly not:
- A lender
- A loan originator
- A credit decision system for final underwriting
- A service provider to the consumer after lead delivery
- A dealer management portal (deferred)

The prequalification result is an estimate, not a credit approval. Final loan terms are determined by the dealer and indirect lender at the time of full credit application, after a hard pull.

### 1.3 Business Objective

Increase Toyota and Lexus franchise dealer lead quality by delivering pre-screened, credit-aware consumers who have demonstrated purchase intent and passed a soft-pull credit filter. Reduce dealer cost-per-funded-unit by improving the conversion rate from initial consumer inquiry to dealership visit and completed financing application.

### 1.4 Consumer Persona (MVP)

**Consumer borrower** — a retail consumer considering the purchase of a new Toyota or Lexus vehicle who wants to understand their financing eligibility before visiting a dealer. This persona represents 100% of the consumer-facing scope at MVP.

The following personas are out of scope at MVP:
- **Franchise dealer portal user** — dealer-facing lead management, application status tracking, and reporting interface. Deferred post-MVP.
- **Lease customer** — lease product eligibility and lease-specific prequal flow. Deferred post-MVP.
- **Internal operations user** — platform administration, dealer enrollment, and policy management. Deferred post-MVP.

### 1.5 Non-Goals

The following are explicitly not goals of the MVP:

| Non-goal | Reason deferred or excluded |
|----------|-----------------------------|
| Direct loan origination | Platform is a lead generator; lending is done by dealers and indirect lenders |
| Dealer portal UI | Deferred post-MVP |
| Lease prequalification | Separate product flow; deferred post-MVP |
| Used vehicle eligibility | Scope is new Toyota and Lexus vehicles only |
| Hard pull credit decisioning | Hard pull is performed by the dealer/lender at full application |
| Income verification against third-party data | Stated income only at prequal |
| Payment or finance charge calculation | No TILA triggering terms in the consumer-facing output |
| Real-time bureau integration | Stubbed for MVP |
| Live DealerTrack/RouteOne ADF delivery | Stubbed for MVP |
| Live KBA provider integration | Stubbed for MVP |
| Multi-brand expansion (beyond Toyota and Lexus) | Not in current scope |
| Brand-specific credit policy (separate thresholds per brand) | Credit policy is shared across Toyota and Lexus at MVP; brand-split max finance table evaluation is post-MVP (see §2.2) |

---

## 2. Scope

### 2.1 In Scope for MVP

| Area | Scope |
|------|-------|
| Vehicle eligibility | New Toyota and Lexus vehicles only |
| Brand skins | Toyota and Lexus visual skins; MVP brand controlled by toggle switch; production brand driven by entry point URL or dealer context; vehicle make filtering applied per active brand at API and UI layers; credit policy shared across brands |
| Consumer flow | Full 8-step prequalification flow (account creation through result display) |
| Authentication | Auth0 Universal Login; OAuth; mandatory account registration |
| OTP verification | 6-digit CSPRNG code via Amazon SNS |
| KBA identity verification | Stubbed (emulated pass/fail) |
| Bureau soft pull | Stubbed (randomized bureau file selection) |
| Credit policy decisioning | 6-check chain per `artifacts/specs/credit-policy-spec.md` (MVP stub values) |
| FCRA disclosure | Single combined conditional disclosure; dynamic SSR; 5-rule consent routing |
| Adverse action notices | Path A (KBA failure) and Path B (soft pull decline / income failure); email via Amazon SES |
| PREQUALIFIED confirmation | On-screen result + confirmation email (Version A and Version B) via Amazon SES |
| ADF lead delivery | Payload generation and audit logging; DealerTrack and RouteOne delivery stubbed |
| Data retention | 7-year retention for FCRA-covered records |
| PII protection | Column-level AES-256-GCM encryption on 10 PII fields |
| Audit log | 23 named events; FCRA_7YR events synchronous within parent transaction |
| Session security | 60-min idle / 4-hour absolute timeout; refresh token rotation; OTP lockout |
| Responsive design | Equal mobile and desktop priority |
| WCAG 2.1 AA | All consumer-facing surfaces |

### 2.2 Out of Scope / Deferred

| Item | Reason |
|------|--------|
| Dealer portal UI | Post-MVP phase; dealer receives leads via existing DMS (DealerTrack/RouteOne) |
| Live bureau soft pull (Experian/Equifax) | Integration complexity; MVP validates flow with stubbed bureau files |
| Live KBA provider integration | Server-to-server callback infrastructure deferred; MVP emulates correct/incorrect handling |
| Live DealerTrack/RouteOne ADF delivery | Vendor onboarding (platform registration, vendor ID acquisition) not started; deferred to production phase |
| Lease prequalification | Separate product and pricing model; not in MVP scope |
| Used vehicle prequal | New vehicles only per product and finance SME scope decision |
| Brand-split max finance tables (Toyota vs. Lexus) | Unified table for MVP; post-MVP evaluation required given Lexus ATPs |
| Conditional derogatory indicator logic | Full conditional logic requires underwriting rule engine; auto-disqualifier behavior at MVP |
| Email verification pre-submission | Triggered post-submission within 7-day window; does not block prequal flow |
| Re-application flow | Not built at MVP; "at this time" language in reason codes implies future eligibility |
| Pricing committee-approved credit policy values | All thresholds (score floor, rate ranges, max finance amounts, lookback periods) are MVP stubs; production values require pricing committee formal approval |
| Multi-state usury cap exclusion logic | Tier 4 APR range (19.9–24.9%) may exceed auto loan usury caps in some states; state exclusion logic deferred pending legal counsel and Credit & Risk SME analysis |
| DEROGATORY_TAX_LIEN activation | Major bureaus removed tax liens from files under NCAP 2017–2018; activation blocked until data source verified by legal counsel |
| DealerTrack and RouteOne vendor ID registration | Requires platform onboarding with each vendor; placeholder values used at MVP |
| Dealer enrollment agreement templates | Required before first dealer onboarded; Legal SME item |
| Reg B §1002.9(c) response process | Process for consumer requests for specific denial reasons within 60 days; operational SOP deferred |

---

## 3. Consumer Flow — Step by Step

### Progress Bar Grouping

The application displays a 4-stage grouped progress bar throughout the flow. All steps map to one of four named stages:

| Stage | Steps covered |
|-------|--------------|
| Your info | Account creation, personal info, income |
| Your vehicle | Vehicle selection |
| Your dealer | Dealer selection |
| Verify & submit | OTP, KBA, Review, FCRA disclosure, Decisioning |

---

### Step 1 — Account Registration / Auth0 Login

**Consumer sees:** Auth0 Universal Login (hosted page); registration form (email, password) or returning user login.

**Consumer does:** Creates an account or logs in.

**System does:**
- Auth0 Universal Login handles all credential entry — no credential handling in the Next.js layer
- Password policy enforced at registration: 12-character minimum; HIBP breach detection active
- On login: session issued; Auth0 access token returned with base claims
- Persistent session / "Remember me" is not offered

**Compliance note:** Account registration is mandatory. Guest flow is not supported.

---

### Step 2 — Personal Information

**Consumer sees:** Form with fields for first name, last name, address (street, city, state, ZIP), date of birth, and last 4 digits of SSN.

**Consumer does:** Enters personal information and submits.

**System does:**
- Validates all fields server-side
- PII fields encrypted at column level (AES-256-GCM) before writing to `applications` table
- `email_lookup_hash` (HMAC-SHA256 of normalized email) stored separately for consumer lookup
- Form state stored server-side only — no localStorage/sessionStorage for PII
- Application record created in state `DRAFT`; single active application per consumer enforced (prior `DRAFT` / `IDENTITY_VERIFIED` records set to `ABANDONED`)

---

### Step 3 — Vehicle Selection

**Consumer sees:** Cascading dropdowns for vehicle year, make, model, and trim, populated from the NHTSA API. The make dropdown is pre-filtered to the active brand — Toyota makes only when the Toyota skin is active; Lexus makes only when the Lexus skin is active.

**Consumer does:** Selects year, make, model, and trim of the new vehicle of interest.

**System does:**
- Queries NHTSA API for vehicle data at each dropdown level
- The NHTSA make filter is applied based on the active brand skin at the API layer and the UI layer — this is not a UI-only filter. When Toyota is active, only Toyota make codes are returned. When Lexus is active, only Lexus make codes are returned.
- Validates that selected vehicle is new Toyota or Lexus (Check 6 in decisioning chain)
- Stores NHTSA codes and display names in the `applications` record (point-in-time snapshot; not foreign keyed to a lookup table)

---

### Step 4 — Dealer Selection

**Consumer sees:** List of nearby franchise dealers filtered by brand (Toyota or Lexus per vehicle selection); ordered by proximity.

**Consumer does:** Selects preferred dealer from the list.

**System does:**
- Dealer list queried from `dealers` table; filtered to `is_active = true` and matching brand
- Selected `dealer_id` stored server-side; not held in client state until submission
- Dealer assignment preserved server-side to prevent race condition where session crash after dealer change routes lead to old dealer

---

### Step 5 — Income Entry

**Consumer sees:** Income input field (monthly gross income, consumer-stated).

**Consumer does:** Enters monthly gross income.

**System does:**
- Stores consumer-stated income (in cents) in the `applications` record
- Income is used for Check 5 (income below minimum) and income band determination for max finance lookup
- Income is never verified against third-party data at prequal
- Income is never included in the ADF payload to the dealer

---

### Step 6 — OTP Phone Verification

**Consumer sees:** Phone number entry field, then OTP entry field after SMS dispatch; resend link; attempt count indication.

**Consumer does:** Enters phone number; receives SMS with 6-digit code; enters code to verify phone ownership.

**System does:**
- Generates 6-digit CSPRNG code (`crypto.randomInt()`); zero-padded; never `Math.random()`
- Stores hashed code (bcrypt, cost factor 10); phone number encrypted (AES-256) in RDS
- Dispatches SMS via Amazon SNS (SMSType: Transactional)
- Enforces: 10-minute TTL per code; prior code invalidated immediately on resend; max 5 attempts per code; max 3 resends per session; 60-second minimum between resends (server-side, not UI-only)
- On 5 failed attempts or 3 resends exhausted: 30-minute lockout scoped to phone number + application session; OAuth account unaffected; application state preserved
- On OTP success: session token rotated before KBA step begins (prevents session fixation); `otp_verified: true`, `otp_verified_at: [timestamp]` claims set in Auth0 `app_metadata` via Auth0 Actions

---

### Step 7 — KBA Identity Verification

**Consumer sees:** 3–5 multiple-choice identity questions drawn from bureau data; response timer.

**Consumer does:** Answers questions within the time limit.

**System does:**
- MVP: emulated correct/incorrect handling from a bank of test questions; real bureau KBA API deferred post-MVP
- KBA integration model: server-to-server callback only (`POST /internal/kba/callback`, VPC-internal, authenticated via pre-shared secret + HMAC-SHA256); browser-mediated result posting not used
- On KBA pass: `kba_verified: true`, `kba_verified_at: [timestamp]`, `verification_context: "lead_submission_[applicationId]"` set in Auth0 `app_metadata`; application state advanced to `IDENTITY_VERIFIED`; KBA claim expires 60 minutes from `kba_verified_at` (independent of session timeout)
- On KBA fail: application terminated; Path A adverse action record created; consumer shown "Unable to complete" message; Path A adverse action notice email dispatched via Amazon SES
- KBA claim invalidation triggers: 60-min claim age, session timeout, device fingerprint change, `verification_context` mismatch; IP address change alone is not an invalidation trigger

---

### Step 8 — Review Screen

**Consumer sees:** Summary of all application data — name, address, vehicle, dealer — before disclosure. Vehicle and dealer shown with "Change" edit links. SSN last 4 displayed masked (not editable from this screen). Income displayed. Confirmation button to proceed to disclosure.

**Consumer does:** Reviews all entered information; optionally changes vehicle (re-routed through dealer selection; prior dealer pre-populated) or dealer; confirms accuracy.

**System does:**
- Vehicle or dealer changes at this step are pre-disclosure; no consent routing action required
- Vehicle change triggers automatic re-routing through dealer selection; prior dealer pre-populated at top of list if still available
- Dealer assignment updated atomically: `PATCH /applications/:id/dealer` writes `dealer_id` and `disclosure_status` in single DB transaction
- SSN last 4 and identity fields (name, address, DOB) are not editable; PII field modification blocked at API layer (not UI-only)
- Identity verification steps (OTP, KBA) are locked-forward; back-navigation to them is blocked

---

### Step 9 — FCRA Disclosure and Consent (Step 7 in the disclosure numbering)

**Consumer sees:** Full FCRA combined conditional disclosure rendered with the name of the currently selected dealer. Explicit "Accept and Submit" button (not pre-checked). Cannot proceed without affirmative action.

**Consumer does:** Reads disclosure; clicks "Accept and Submit" to consent to the soft pull and, if applicable, to share rate range and max finance amount with the named dealer.

**System does:**
- Disclosure rendered via dynamic SSR only — no static generation, ISR, or caching; `no-cache` headers enforced
- Dealer name populated from current `applications.dealer_id` at render time — never from a cached value
- On acceptance: inserts row in `fcra_consents` table (stores `dealer_id`, `disclosure_version`, `disclosure_status = 'accepted'`, `otp_verified_at`, `kba_verified_at`, `verification_context`, `session_id`, `ip_address`, `user_agent`, timestamp); updates `applications.disclosure_status = 'accepted'` atomically in single transaction
- Consent record stores `dealer_id` at time of acceptance for server-side submission guard (Rule 5)
- If separate consent to share rate range / max finance amount is obtained at this screen, stores `consent_to_share_rate_range`, `consent_to_share_max_finance`, and associated values in consent record
- Back-navigation past Step 9: Option A hard-block enforced — frontend routing block + server-side 409 on write attempts after `disclosure_status = accepted`; consumer starting over after Step 9 has prior application set to `status = ABANDONED` server-side (audit record preserved)
- Server-side submission guard (Rule 5): at submit, verifies `disclosure_status = 'accepted'`, `bureau_pull_authorization IS NOT NULL`, and `applications.dealer_id = fcra_consents.dealer_id`; any mismatch blocks bureau pull

**FCRA consent routing rules:**

| Rule | Trigger | Action |
|------|---------|--------|
| Rule 1 | Changes before disclosure | No action required |
| Rule 2 | Vehicle change after acceptance | Hard-block back-navigation past Step 9 (Option A) |
| Rule 3 | Dealer change after acceptance | `disclosure_status = 'invalidated'`; hard-block bureau pull; disclosure re-presented with new dealer name |
| Rule 4 | Disclosure render | Dealer name resolved from current `applications.dealer_id` at render time — never cached |
| Rule 5 | Submit | Server-side guard: `disclosure_status = 'accepted'` AND `dealer_id_at_consent = dealer_id_at_delivery` |

---

### Step 10 — Decisioning

**Consumer sees:** Loading state with progress indicator.

**System does (6-check decisioning chain):**

| Check | What is evaluated | Failure path |
|-------|-------------------|--------------|
| 1 | Session re-validation: server-side token assertion; KBA claim non-expired (≤60 min from `kba_verified_at`) | Hard-block; session error |
| 2 | SSN last 4 matches bureau record (identity confirmation) | Path B adverse action |
| 3 | Credit score ≥ 620 (FICO 8 / VantageScore 3.0) | Path B adverse action; `CREDIT_SCORE_BELOW_MINIMUM` |
| 4 | No disqualifying derogatory indicators (8 defined indicators; MVP: all automatic) | Path B adverse action; specific reason code(s) per precedence order |
| 5 | Stated income ≥ $2,000/month gross | Path B adverse action; `INCOME_BELOW_MINIMUM` |
| 6 | Selected vehicle is new Toyota or Lexus | Path B adverse action |

All 6 checks must pass. Decisioning executes sequentially. On any failure, the chain stops and the failure path executes. Bureau soft pull (checks 2–4) executes only after Check 1 (session re-validation) passes and after Rule 5 submission guard confirms valid consent.

**On any failure:** Application advances to `DECLINED`; `expires_at` set to `decided_at + 30 days`; Path B adverse action email dispatched via Amazon SES; consumer shown "Unable to complete your pre-qualification" message.

**On all checks pass:** Application advances to `PREQUALIFIED`; credit tier determined; max finance amount looked up from `max_finance_amounts` table; `rate_range_floor_bps` and `rate_range_ceiling_bps` stored; `expires_at` set to `decided_at + 30 days`; confirmation number generated (`PQ-YYYYMMDD-[8 Crockford Base32 chars]`); `prequalification.completed` event emitted to EventBridge.

**Application state machine:**

| State | Description |
|-------|-------------|
| `DRAFT` | Application created; personal info and preferences in progress |
| `IDENTITY_VERIFIED` | OTP and KBA passed; not resumable after session expiry |
| `DISCLOSURE_ACCEPTED` | FCRA consent recorded |
| `DECISIONING` | Decisioning chain in progress |
| `PREQUALIFIED` | All checks passed; lead delivery triggered |
| `DECLINED` | One or more checks failed; adverse action triggered |
| `ABANDONED` | Consumer started over or session expired after Step 9; prior application set server-side |
| `EXPIRED` | `expires_at` elapsed; application no longer active |

Terminal states: `PREQUALIFIED`, `DECLINED`, `ABANDONED`, `EXPIRED`.

---

### Step 11 — Result Display

**PREQUALIFIED path:**

**Consumer sees:** Decision screen with confirmation number, selected dealer name, estimated APR range (if consent given), estimated maximum finance amount (if consent given), expiration date, and next steps.

**System does:** Reads decision from DB; renders result. Does not re-execute decisioning logic on page render.

**DECLINED path:**

**Consumer sees:** "Unable to complete your pre-qualification" message. Consumer-facing text does not include reason codes.

**System does:** Path B adverse action email dispatched (if not already dispatched during decisioning). Consumer does not receive a lead delivery.

---

### Step 12 — Confirmation Email

**Trigger:** `prequalification.completed` EventBridge event consumed by Notification Service; conditioned on `applications.status = 'PREQUALIFIED'`.

**Two email versions:**

| Version | Condition | Contains |
|---------|-----------|----------|
| A — Full | `consent_to_share_rate_range = true` AND `consent_to_share_max_finance = true` AND `dealer_id_at_consent = dealer_id_at_delivery` | Confirmation number, consumer name, dealer name, vehicle scope, expiration date, estimated APR range, estimated max finance amount, required qualifying language, next steps |
| B — Confirmation only | Any consent condition not met | Confirmation number, consumer name, dealer name, vehicle scope, expiration date, next steps, baseline qualifying statement |

Conditional fields in Version B are omitted entirely — no placeholder text, no explanation.

Full spec: `artifacts/specs/prequalified-email-spec.md`

---

### Step 13 — Adverse Action Notice Emails

**Path A — KBA failure:**
Triggered at KBA fail event. Sent to consumer's registered email via Amazon SES. Required elements: adverse action notice per FCRA §1681m (if KBA product is classified as consumer report) and ECOA Reg B §1002.9.

**Path B — Soft pull decline / income failure:**
Triggered at DECLINED decisioning outcome. Sent to consumer's registered email via Amazon SES. Required elements: FCRA §1681m elements (CRA name and contact, right to free report, right to dispute, statement CRA did not make the decision) and ECOA Reg B §1002.9 elements (action taken, ECOA anti-discrimination tagline, creditor name/address, principal reason code(s), right to request specific reasons within 60 days).

On-screen message alone is non-compliant for either path.

Full spec: `artifacts/specs/adverse-action-spec.md`

---

### Step 14 — ADF Lead Delivery

**Trigger:** `prequalification.completed` EventBridge event consumed by Lead Distribution Service; delivery conditioned on `applications.status = 'PREQUALIFIED'`.

**System does:**
- Resolves delivery target(s) from `dealers` table: if `dealertrack_id` non-null, enqueue DealerTrack delivery; if `routeone_id` non-null, enqueue RouteOne delivery; if both non-null, dual delivery
- Assembles ADF 1.0 XML payload using XML builder library (not string interpolation)
- Conditional fields (rate range, max finance amount) included only if `consent_to_share_rate_range = true` AND `consent_to_share_max_finance = true` AND `dealer_id_at_consent = dealer_id_at_delivery`; omitted silently otherwise
- MVP: generates structurally valid ADF, logs payload hash with `delivery_status: STUBBED`, does not call live endpoints
- All delivery attempts written to audit log

Full spec: `artifacts/specs/adf-payload-spec.md`

---

## 4. Functional Requirements

### 4.0 Brand and Skin

| ID | Requirement |
|----|-------------|
| BRAND-01 | The platform must support two visual brand skins: Toyota and Lexus. Each skin applies a distinct brand theme including color scheme, typography, logo, and branded email templates. The underlying application logic, API behavior, credit policy, and decisioning engine are identical for both brands. |
| BRAND-02 | **MVP / Demo:** Brand is controlled by a toggle switch that allows the user to switch between Toyota and Lexus skins. Engaging the toggle simultaneously switches the visual skin and the active brand filter for vehicle selection and dealer selection. |
| BRAND-03 | **Production:** The active brand must be determined by the consumer's entry point — either the URL (e.g., toyota.auto-leads.io vs lexus.auto-leads.io) or referral context (OEM site source) or dealer context — not a manual toggle. The toggle mechanism is not acceptable for production. |
| BRAND-04 | When the Toyota skin is active, the vehicle make dropdown must be restricted to Toyota make codes from the NHTSA API. When the Lexus skin is active, the make dropdown must be restricted to Lexus make codes from the NHTSA API. Non-brand makes must not appear regardless of call origin. |
| BRAND-05 | The NHTSA make filter based on active brand must be enforced at the **API layer and the UI layer**. UI-only filtering is not acceptable. The vehicle service endpoint must accept a `brand` parameter and return only makes matching the active brand. The frontend must not be able to submit makes that do not match the active brand. |
| BRAND-06 | When the Toyota skin is active, the dealer selection list must be filtered to dealers with `dealers.brand = 'TOYOTA'`. When the Lexus skin is active, the dealer selection list must be filtered to `dealers.brand = 'LEXUS'`. |
| BRAND-07 | Credit policy is shared across both brands. The same minimum credit score threshold, income minimum, derogatory indicator rules, pricing tier table, and max finance amount lookup table apply to Toyota and Lexus applicants equally. No brand-specific credit policy variation is permitted at MVP. |
| BRAND-08 | Branded email templates (PREQUALIFIED confirmation and adverse action notices) must render with the branding of the brand active at the time of application submission — not the brand active at the time of email delivery. The brand context must be stored on the application record and used for all downstream communications. |

---

### 4.1 Authentication and Identity

| ID | Requirement |
|----|-------------|
| AUTH-01 | The system must use Auth0 Universal Login (hosted page) for all consumer authentication. Credential entry must not occur within the Next.js application layer. |
| AUTH-02 | Account registration is mandatory before prequalification submission. Guest flow is not supported. |
| AUTH-03 | Password policy: minimum 12 characters; no mandatory complexity rules per NIST SP 800-63B. Auth0's default 8-character minimum must be overridden. |
| AUTH-04 | Auth0 HIBP breach detection must be enabled. Compromised passwords must be blocked at registration and login with a notification to the consumer. |
| AUTH-05 | Session idle timeout: 60 minutes. Session absolute timeout: 4 hours. Both enforced at Auth0 tenant and application level. |
| AUTH-06 | Persistent sessions ("Remember me") must not be offered. |
| AUTH-07 | Refresh token rotation must be enabled with reuse detection. On reuse detection, the full token family must be revoked. |
| AUTH-08 | Session token must be rotated after OTP success before the KBA step begins. |
| AUTH-09 | Auth0 Actions (not deprecated Rules) must be used for all custom claim injection into the access token. |
| AUTH-10 | Consumer flow must run on a standard Auth0 Application, not an Auth0 Organization. |
| AUTH-11 | Email verification is deferred until after prequal submission. Submission must not be blocked on email verification. Verification link sent post-submission with a 7-day window. |

### 4.2 OTP Phone Verification

| ID | Requirement |
|----|-------------|
| OTP-01 | OTP code must be 6 digits, numeric, generated via `crypto.randomInt()`, zero-padded. `Math.random()` is prohibited. |
| OTP-02 | OTP code must be stored as a bcrypt hash (cost factor 10). Plaintext storage is prohibited. |
| OTP-03 | OTP TTL: 10 minutes from generation. Code is invalidated immediately on resend — a new code replaces the prior code; only one active OTP per phone number at any time. |
| OTP-04 | Maximum 5 failed verification attempts per code. Code invalidated after 5 failures; consumer directed to resend. |
| OTP-05 | Maximum 3 resends per session. Minimum 60 seconds between resends. Both limits enforced server-side, not UI-only. |
| OTP-06 | On exhaustion of resends or attempts: 30-minute lockout scoped to phone number + application session. OAuth account is unaffected. Application state is preserved during lockout. |
| OTP-07 | OTP SMS must be delivered via Amazon SNS, SMSType Transactional. Phone number must be encrypted (AES-256) in RDS before storage. |
| OTP-08 | On OTP success: `otp_verified: true` and `otp_verified_at: [Unix timestamp]` must be set in Auth0 `app_metadata` via a server-side Auth0 Action. |

### 4.3 KBA Identity Verification

| ID | Requirement |
|----|-------------|
| KBA-01 | KBA is stubbed for MVP. The stub must emulate the production server-to-server callback model internally — the MVP stub calls the internal function directly; it does not register an HTTP route. |
| KBA-02 | KBA integration model (production): server-to-server callback only. The KBA provider must POST to `POST /internal/kba/callback` (VPC-internal, not public-facing). Callback endpoint must require pre-shared secret authentication + HMAC-SHA256 payload signing. Browser-mediated result posting is prohibited. |
| KBA-03 | On KBA pass: `kba_verified: true`, `kba_verified_at: [Unix timestamp]`, and `verification_context: "lead_submission_[applicationId]"` must be set in Auth0 `app_metadata` server-side. Application state must advance to `IDENTITY_VERIFIED`. |
| KBA-04 | KBA claim expiry: 60 minutes from `kba_verified_at`. This is independent of session timeout. Server-side enforcement is mandatory on every protected endpoint. Auth0 token lifetime must not be used as the KBA re-use window. |
| KBA-05 | KBA claim invalidation triggers: claim age > 60 minutes; session idle or absolute timeout; device fingerprint change; `verification_context` mismatch with active application ID. IP address change alone must not trigger invalidation. |
| KBA-06 | Vehicle and dealer changes do not require KBA re-verification. PII field changes (name, address, DOB, SSN) require new KBA challenge. PII field modification must be blocked at the API layer on all PATCH endpoints, not UI-only. |
| KBA-07 | `IDENTITY_VERIFIED` applications are not resumable after session expiry. The 60-minute KBA claim window will have elapsed. Consumer must start a new application. |
| KBA-08 | On KBA fail: application must be terminated; Path A adverse action record created; Path A adverse action notice email dispatched; consumer shown "Unable to complete" message. |

### 4.4 FCRA Disclosure and Consent

| ID | Requirement |
|----|-------------|
| FCRA-01 | The FCRA disclosure must be a single combined conditional disclosure covering both the KBA bureau identity pull and the soft pull. |
| FCRA-02 | The FCRA disclosure page must be rendered via dynamic SSR only. Static generation, ISR, and all caching strategies are prohibited for this route. |
| FCRA-03 | The dealer name in the FCRA disclosure must be resolved from `applications.dealer_id` at SSR render time. Populating it from any cached or previously stored value is a compliance defect. |
| FCRA-04 | The "Accept and Submit" button must require explicit affirmative interaction. Pre-checking or pre-populating consent is prohibited. |
| FCRA-05 | On acceptance: an `fcra_consents` row must be inserted with `dealer_id`, `disclosure_version`, `disclosure_status`, `otp_verified_at`, `kba_verified_at`, `verification_context`, `session_id`, `ip_address`, `user_agent`, and timestamp. `applications.disclosure_status` must be updated to `'accepted'` in the same atomic DB transaction. |
| FCRA-06 | Back-navigation past the disclosure screen (Option A): the frontend must block back-navigation routing; the server must return 409 on any write attempts once `disclosure_status = 'accepted'`. Consumer starting over after acceptance has their prior application set to `status = ABANDONED` server-side. |
| FCRA-07 | Dealer change after disclosure acceptance must atomically set `disclosure_status = 'invalidated'` and clear `bureau_pull_authorization` in a single DB transaction. Bureau pull must be hard-blocked until consumer re-accepts the disclosure with the new dealer name rendered. |
| FCRA-08 | Server-side submission guard (Rule 5): before executing the bureau pull, the system must verify: `disclosure_status = 'accepted'`, `bureau_pull_authorization IS NOT NULL`, and `fcra_consents.dealer_id = applications.dealer_id`. Any mismatch must hard-block execution. |
| FCRA-09 | The `fcra_consents` table must retain records for 7 years (FCRA + ECOA retention requirement). Hard deletion is prohibited while records are within the retention window. |
| FCRA-10 | Conditional consent fields (`consent_to_share_rate_range`, `consent_to_share_max_finance`) must be stored in the consent record with values shown to consumer at consent time and dealer_id at consent. |

### 4.5 Credit Policy Decisioning

| ID | Requirement |
|----|-------------|
| DEC-01 | The decisioning engine must execute 6 checks in sequence. All 6 must pass for a `PREQUALIFIED` outcome. |
| DEC-02 | Check 1: Server-side session re-validation. KBA claim must be non-expired (`kba_verified_at` age ≤ 60 minutes) at submit time. This check must execute even if KBA was valid at disclosure acceptance. |
| DEC-03 | Check 2: SSN last 4 identity confirmation against bureau record. |
| DEC-04 | Check 3: Credit score ≥ 620 (FICO 8 / VantageScore 3.0). Failure reason code: `CREDIT_SCORE_BELOW_MINIMUM`. |
| DEC-05 | Check 4: Derogatory indicator evaluation. 8 defined indicators with lookback periods per `artifacts/specs/credit-policy-spec.md`. At MVP, all conditional indicators implemented as automatic disqualifiers. `DEROGATORY_COMBINED` must not be used as a standalone adverse action reason code. Multi-reason scenarios must apply the defined precedence order. |
| DEC-06 | Check 5: Stated income ≥ $2,000/month gross. Failure reason code: `INCOME_BELOW_MINIMUM`. |
| DEC-07 | Check 6: Selected vehicle is new Toyota or Lexus. |
| DEC-08 | Credit policy parameters (score threshold, income threshold, tier boundaries, rate ranges, max finance amounts, derogatory lookback periods) must be stored as seeded database rows in `credit_policy_versions`, `credit_tiers`, `income_bands`, `max_finance_amounts`, and `derogatory_indicator_rules` tables. Hard-coded constants are prohibited. |
| DEC-09 | Only one active credit policy version may exist at a time. Policy version updates must execute as an atomic 2-statement swap (deactivate current, activate new) within a single transaction. No code deploy is required for pricing committee value changes. |
| DEC-10 | The decisioning engine must execute a single lookup query joining `credit_tiers`, `income_bands`, and `max_finance_amounts` on the active policy version. It must not assemble results from multiple independent queries. |
| DEC-11 | On `PREQUALIFIED`: `expires_at` must be set to `decided_at + 30 calendar days`. Confirmation number must be generated via server-side CSPRNG, format `PQ-YYYYMMDD-[8 Crockford Base32 chars]`, stored as unique indexed secondary field alongside UUID primary key. |
| DEC-12 | On `DECLINED`: `expires_at` must be set to `decided_at + 30 calendar days`. Path B adverse action email must be dispatched. |
| DEC-13 | `DEROGATORY_TAX_LIEN` must be seeded as `is_active = FALSE` in `derogatory_indicator_rules`. Activation requires data source verification and legal counsel confirmation. |
| DEC-14 | Rate ranges must be stored as integers in basis points (`rate_range_floor_bps`, `rate_range_ceiling_bps`). 590 = 5.90% APR. Floating-point storage is prohibited. |
| DEC-15 | Adverse action reason code multi-reason precedence order: bankruptcy > repossession > charge-off > foreclosure > tax lien > auto collection > score below minimum > multiple lates > income below minimum > combined derogatory. |

### 4.6 Lead Distribution (ADF)

| ID | Requirement |
|----|-------------|
| ADF-01 | ADF lead delivery must be conditioned on `applications.status = 'PREQUALIFIED'`. The Lead Distribution Service must filter on this status independently — it must not rely solely on the upstream event filter. |
| ADF-02 | ADF payload must conform to ADF 1.0 XML schema. XML must be generated using an XML builder library (e.g., `xmlbuilder2`). String interpolation and template literals are prohibited for XML construction. |
| ADF-03 | Hard-excluded fields must never appear in any ADF payload on any code path: last 4 SSN, full SSN, date of birth, stated income, raw credit score. An automated test asserting these fields absent from every code path is required before production release. |
| ADF-04 | Conditional fields (rate range floor/ceiling, max finance amount) must be included only when: `consent_to_share_rate_range = true`, `consent_to_share_max_finance = true`, and `fcra_consents.dealer_id = applications.dealer_id` at delivery time. When any condition is unmet, all three fields must be omitted silently (no empty elements). |
| ADF-05 | Idempotency key for ADF delivery: the application confirmation number. Must be passed as appropriate header to each delivery target. |
| ADF-06 | Retry policy: retry on HTTP 5xx and network timeout; do not retry on HTTP 4xx. Maximum 3 attempts (initial + 2 retries). Backoff: 1s, 4s, 16s (base-4 exponential with ±20% jitter). After retry exhaustion: enqueue for manual ops review; CloudWatch alert. |
| ADF-07 | Dual delivery: if `dealer.dealertrack_id` is non-null, deliver to DealerTrack. If `dealer.routeone_id` is non-null, deliver to RouteOne. If both are non-null, deliver to both. Dual delivery is the correct behavior — do not suppress one. |
| ADF-08 | Every delivery attempt must be written to the audit log including: `application_id`, `confirmation_number`, `delivery_target`, `dealer_id`, platform-specific dealer ID used, `payload_hash` (SHA-256 of serialized XML), `conditional_fields_included`, `conditional_fields_omitted_reason`, `delivery_status`, `platform_assigned_lead_id`, `http_status_code`, `delivered_at`. |
| ADF-09 | Full ADF XML payload must not be logged in plaintext. Log the `payload_hash` only. If full payload capture is required for debugging, write to a separate encrypted log store with access controls. |
| ADF-10 | MVP stub: assemble complete, structurally valid ADF XML; execute all conditional field logic and phone/amount formatting; log with `delivery_status: STUBBED`; do not call live DealerTrack or RouteOne endpoints. |
| ADF-11 | Rate range format for ADF payload: decimal string representing percentage with one decimal place (`"5.9"`, `"8.9"`). No `%` symbol. Derived from basis-point integers at payload assembly time. |
| ADF-12 | Max finance amount format for ADF payload: whole dollar integer as string (`"45000"`). Derived from cents integer at payload assembly time (`floor(cents / 100)`). |
| ADF-13 | Phone number format for ADF payload: 10-digit national format `(NXX) NXX-XXXX`. Strip leading `+1` from E.164 stored format at assembly time. |

### 4.7 Adverse Action Notices

| ID | Requirement |
|----|-------------|
| AA-01 | Two adverse action notice variants are required: Path A (KBA failure) and Path B (soft pull decline, including income failure). On-screen message alone is non-compliant for either path. |
| AA-02 | Path A notice must include all FCRA §1681m required elements (if KBA product classified as consumer report) and all ECOA Reg B §1002.9 required elements. |
| AA-03 | Path B notice must include: FCRA §1681m elements (name, address, phone of CRA; statement CRA did not make the decision and cannot explain reasons; right to free report within 60 days; right to dispute) and ECOA Reg B §1002.9 elements (action taken; ECOA anti-discrimination tagline verbatim; creditor name and address; principal reason code(s); right to request specific reasons within 60 days). |
| AA-04 | ECOA anti-discrimination tagline must appear verbatim in all adverse action notices. See `artifacts/specs/adverse-action-spec.md` §2 for exact text. |
| AA-05 | Adverse action notices must be delivered via Amazon SES within 30 days of the completed application event. For instant decisioning, dispatch at or immediately following the decline event. |
| AA-06 | `DEROGATORY_COMBINED` must not be used as a standalone adverse action reason. Multi-derogatory scenarios must list specific reason codes per the defined precedence order. |
| AA-07 | `DEROGATORY_TAX_LIEN` reason code must not be activated until data source verification is complete and legal counsel confirms tax lien tradelines are present in soft pull data. |
| AA-08 | Adverse action reason codes may appear in the notice email and internal audit log only. The consumer-facing on-screen message must be "Unable to complete your pre-qualification." only. |

### 4.8 PREQUALIFIED Confirmation Email

| ID | Requirement |
|----|-------------|
| EMAIL-01 | PREQUALIFIED confirmation email must fire only when `applications.status = 'PREQUALIFIED'`. Status must be read from the database — it must not be inferred from other fields. |
| EMAIL-02 | Two email versions must be implemented: Version A (Full) and Version B (Confirmation only), determined by consent conditions identical to ADF conditional field logic (same Rule 5 check). |
| EMAIL-03 | Required fields in both versions: confirmation number (display as `PQ-YYYYMMDD-[8 chars]`), consumer first and last name (decrypted at application layer), selected dealer name, vehicle scope, expiration date (`applications.expires_at`), next steps, qualifying statement. |
| EMAIL-04 | Version A additional fields: estimated APR range (displayed as `X.XX% – X.XX% APR`; two decimal places always; "APR" label on same line) and estimated maximum finance amount (displayed as "Up to $X,XXX"). Required qualifying language must appear proximate to these fields — not in the footer only. |
| EMAIL-05 | Rate range must be derived from `rate_range_floor_bps` and `rate_range_ceiling_bps`; converted to percentage strings at the application layer (not the email template). Basis-point integers must not reach the template. |
| EMAIL-06 | Max finance amount must be derived from `max_finance_amount_cents`; converted to dollar amount at the application layer. Cents must not reach the template. |
| EMAIL-07 | The following fields are prohibited from the email under all circumstances: monthly payment estimate, number of payments, loan term, finance charge in dollars, down payment amount, single APR (not a range), full SSN, DOB, full address, stated income, adverse action reason codes, ECOA anti-discrimination tagline. |
| EMAIL-08 | "Up to" framing is mandatory for max finance amount. "You qualify for", "approved amount", or any language implying a committed amount is prohibited. |
| EMAIL-09 | Expiration date label must not use the word "offer." Use "Prequal valid through," "Estimate valid through," or "Result valid through." |
| EMAIL-10 | Qualifying language is static copy in the email template. Changes to qualifying language require a template deploy. Runtime flags must not suppress required qualifying language. |

### 4.9 Dealer Portal

The dealer portal is explicitly deferred to post-MVP. No dealer-facing UI is in scope for this release. Dealers receive leads via their existing DMS platforms (DealerTrack, RouteOne) through the ADF lead delivery mechanism.

---

## 5. Non-Functional Requirements

### 5.1 Responsive Design

The application must deliver a fully functional, usable experience on both mobile and desktop viewports. Equal priority — mobile is not a degraded experience. No specific breakpoint constraints are prescribed at PRD level; implementation per UX Designer specification.

### 5.2 Accessibility

All consumer-facing surfaces must meet WCAG 2.1 Level AA compliance. This is a non-negotiable global standard. Accessibility audit (VPAT) is a Phase 4 deliverable (`artifacts/14-accessibility-audit-vpat.md`) but accessibility must be built-in from the start, not retrofitted.

### 5.3 Performance

| Target | Metric |
|--------|--------|
| End-to-end prequal (form submit → decision displayed) | < 5 seconds P95 |
| KBA API response timeout threshold | 10 seconds (then "Unable to complete") |
| Bureau soft pull API response timeout threshold | 10 seconds with 1 retry (then "Unable to complete") |
| PREQUALIFIED email delivery after `prequalification.completed` event | Within 5 minutes |
| ADF lead delivery after `prequalification.completed` event | Within 60 seconds |

### 5.4 Browser and Device Back-Navigation

Identity verification steps (OTP, KBA, FCRA disclosure) are locked-forward once completed. Back-navigation to these steps is blocked at two layers:
1. Frontend routing block
2. Server-side 409 response on write attempts after `disclosure_status = accepted`

The Review screen (Step 8) is the designated correction point for vehicle and dealer changes. All other edits to identity fields (name, address, DOB, SSN) are blocked after initial submission.

### 5.5 Data Retention

FCRA-covered records (application records, consent records, adverse action records, audit log events marked `FCRA_7YR`) must be retained for a minimum of 7 years. ECOA Reg B requires 25 months minimum for most records; the more restrictive 7-year FCRA window governs this platform.

Hard deletion of records within the retention window is prohibited. Soft-delete only (status flags, `is_active` fields). The `dealers` table enforces `ON DELETE RESTRICT` on foreign keys referenced by `fcra_consents` within the 7-year window.

### 5.6 PII Security

| Requirement | Detail |
|-------------|--------|
| Column-level encryption | 10 PII fields encrypted at rest using AES-256-GCM. Fields: `first_name`, `last_name`, `date_of_birth`, `ssn_last_four`, `phone_number`, `email`, `address_line_1`, `address_line_2`, `city`, `state_code`, `zip_code` |
| Encryption algorithm | AES-256-GCM (authenticated encryption). CBC is prohibited. IV embedded in ciphertext blob: `[4-byte key_version][12-byte IV][16-byte auth tag][ciphertext]` as base64url |
| Key management | Encryption key stored in AWS Secrets Manager only. Retrieved via IAM role at application startup. If key unavailable after 5 retries: `process.exit(1)`. No PII write endpoint executes if key is unavailable. |
| In-request key outage | 503 with `Retry-After: 30`. Circuit breaker with 10-minute extended grace window for stale-but-valid cached key. |
| Email lookup hash | `email_lookup_hash` stored as HMAC-SHA256 of normalized email using a separate key in Secrets Manager. Allows consumer lookup without decryption. |
| Browser storage | localStorage and sessionStorage are prohibited for all PII fields. Form state restoration must be server-side and session-bound. |
| Crypto library | `node:crypto` only. Third-party crypto libraries are prohibited. |

### 5.7 Server-Side Form State

All form state is maintained server-side. The browser does not hold PII fields between page loads. Session-bound server-side storage is the only permissible mechanism for partial form state restoration.

### 5.8 Monetary Values

All monetary values (income, max finance amount, rate ranges) must be stored and calculated as integers (cents or basis points). Floating-point representation of monetary values is prohibited.

---

## 6. MVP Stubs

The following components are explicitly stubbed at MVP. Each stub must faithfully emulate the production interface contract so that no re-architecture is required at go-live.

| Component | MVP Stub Behavior | Production Replacement |
|-----------|-------------------|----------------------|
| Bureau soft pull (Experian/Equifax) | Random selection from ~50 pre-generated bureau response files. Returns a realistic bureau response object. | Real Experian or Equifax soft pull API via server-to-server integration |
| KBA identity verification | Bank of test questions with emulated correct/incorrect answer handling. Mirrors production server-to-server callback model via internal function call (no HTTP route registered at MVP). | Real bureau KBA API; KBA provider calls `POST /internal/kba/callback` (VPC-internal + VPN/Direct Connect); authenticated via pre-shared secret + HMAC-SHA256 |
| DealerTrack/RouteOne dealer ID lookup | Returns randomized IDs following the DealerTrack and RouteOne naming convention. | Live dealer ID lookup API for each platform |
| DealerTrack ADF delivery | Assembles structurally valid ADF 1.0 XML; exercises all conditional field logic; logs payload hash with `delivery_status: STUBBED`; does not call live DealerTrack endpoint. Synthetic lead ID returned: `STUB-DT-[confirmationNumber]` | Live HTTPS POST to DealerTrack Lead API endpoint with API key header |
| RouteOne ADF delivery | Same as DealerTrack stub but for RouteOne. Synthetic lead ID: `STUB-R1-[confirmationNumber]` | Live HTTPS POST to RouteOne Lead API endpoint with OAuth 2.0 client credentials |

**Stub fidelity requirements:**
- Bureau stub must return a response object structurally identical to what the production bureau API returns
- KBA stub must call the same internal handler function that the production callback will call
- ADF stubs must execute the full phone formatting, amount formatting, and conditional field logic
- No stub may use a different code path from the production integration for the business logic it exercises

---

## 7. Security and Compliance Requirements

### 7.1 Security Controls

The following controls are mandatory. These derive from the Security Gate review (threat model: `artifacts/08-threat-model.md`; gate verdict: PASS WITH CONDITIONS; three gate blockers closed via `artifacts/specs/column-encryption-key-spec.md`, `artifacts/specs/idor-enforcement-spec.md`, `artifacts/specs/audit-log-spec.md`).

| Control | Requirement |
|---------|-------------|
| Column encryption | AES-256-GCM on 10 PII fields; key from Secrets Manager; hard fail if key unavailable (see §5.6) |
| IDOR protection | All application endpoints must validate `applications.consumer_user_id = authenticated JWT sub` using single query `WHERE id = $1 AND consumer_user_id = $2`. 404 returned for both genuine not-found and ownership mismatch (identical status and body). Middleware chain: `jwtMiddleware → requireApplicationOwnership → handler`. TypeScript type extension makes skipping middleware a compile-time error. |
| Audit log immutability | Four-layer protection: PostgreSQL RULE (suppressor), trigger (visible error on violation), app role REVOKE on UPDATE/DELETE, pgAudit logging. |
| Audit log PII exclusion | `sanitizeEventData` throws on known PII key — silent stripping is prohibited. `event_data` contains IDs, status codes, reason codes, and counts only. |
| Structured audit log | 23 named events. FCRA_7YR events written synchronously within parent transaction (async queue rejected for FCRA records due to loss risk). Non-FCRA_7YR events use a separate transaction. Full event list in `artifacts/specs/audit-log-spec.md`. |
| Session fixation prevention | Session token rotation after OTP success. |
| API scope restriction | Field-scoped PATCH endpoints only. `PATCH /applications/:id/vehicle` and `PATCH /applications/:id/dealer` only. No general-purpose `PATCH /applications/:id`. Identity fields (name, address, DOB, SSN) are not patchable by consumer after creation. Schema-level rejection on requests containing identity fields. |
| FCRA submission guard | Server-side Rule 5 guard non-bypassable. See FCRA-08. |
| Dealer PATCH atomicity | `PATCH /applications/:id/dealer` must atomically write `dealer_id` and `disclosure_status` in a single DB transaction. Partial update is an FCRA risk. |
| KBA callback authentication | Production KBA callback endpoint must use pre-shared secret + HMAC-SHA256. Unauthenticated callback is a critical vulnerability. |
| Secrets management | Three Secrets Manager secrets: `auto-leads/{env}/auth0/m2m-credentials`, `/redis/auth-token`, `/kba/callback-preshared-secret`. 90-day rotation on M2M and Redis. IAM: `GetSecretValue` + `DescribeSecret` on named ARNs only. App runtime role must not have `PutSecretValue`. |
| Auth0 M2M scope restriction | Scopes: `read:users` + `update:users` only. Application-layer discipline: only write the 5 defined verification claim fields; never write auth-bearing fields. |
| Redis access | VPC private subnet only. Security group: inbound port 6379/6380 from backend service SG only. TLS enabled. AUTH token from Secrets Manager. Hard-fail 503 if both Redis and Auth0 `app_metadata` unavailable — no fail-open on JWT claim alone. |
| ADF payload exclusion test | Automated test required before production release: assert SSN-4 and stated income absent from every ADF payload on every code path. |
| GLBA vendor agreements | Auth0 DPA, AWS BAA, and dealer enrollment agreement template required before first consumer data in production. |
| Anomaly detection threshold | IDOR ownership mismatch: log at WARN as `application.ownership_mismatch`; SIEM threshold 5 mismatches from same `user_id` in 10 minutes. |

### 7.2 Regulatory Compliance Summary

| Regulation | How it applies | Key requirements |
|------------|----------------|-----------------|
| FCRA | Soft pull consumer report; KBA product (if classified as consumer report); adverse action notice obligation on decline | Single combined conditional disclosure; adverse action notices Path A and Path B; 7-year record retention; permissible purpose (consumer written instructions naming specific dealer) |
| ECOA / Reg B | Prequal flow is treated as a credit application | Principal reason code(s) in adverse action notice; ECOA tagline verbatim; right to request specific reasons within 60 days |
| GLBA / Reg P | Dealer classified as nonaffiliated third party; sharing of consumer NPI | Sharing basis: §1016.14 (consumer-directed) + §1016.15(a)(1) (explicit consent); separate consent for rate range / max finance sharing; last 4 SSN, DOB, stated income excluded from ADF; dealer enrollment agreements with data use restrictions |
| TILA / Reg Z | Email and on-screen content may constitute advertising | APR range alone is not a TILA 1026.24(c) triggering term; monthly payment, number of payments, finance charge in dollars, down payment must not appear in consumer communications; "APR" label required per 1026.24(b) |
| UDAAP | Consumer-facing messaging and adverse action notices | "Our records indicate" language removed from reason codes; "recent" temporal qualifiers removed; qualifying language on rate range and max finance amount estimates |

### 7.3 Mandatory Gates Before Production Release

The following gates must pass before any component is deployed to production. All are currently in progress or pending.

| Gate | Required sign-off | Status | Pre-condition |
|------|------------------|--------|---------------|
| Architecture Gate | Architect | Pending — Phase 2 artifact completion | Architecture document and all ADRs finalized |
| Security Gate | Cybersecurity SME | Partial — 3 gate blockers closed; full gate pending implementation review | All security controls built and tested |
| Legal Gate | Legal SME | Partial — core decisions locked; 5 open counsel items outstanding | Qualified legal counsel confirmation on all items in `artifacts/specs/adverse-action-spec.md` §5, `artifacts/specs/fcra-consent-routing-spec.md` §6, and `artifacts/specs/prequalified-email-spec.md` §7 |
| Finance Gate | Finance SME | Not started | Financial spec and ADF payload implementation complete |
| Credit Policy Gate | Credit & Risk SME | MVP spec complete | Pricing committee approval for all production threshold values |
| Release Gate | QA Analyst + Cybersecurity SME | Not started | All above gates cleared; full test cycle complete |

---

## 8. Open Items and Pre-Production Gates

The following items are explicitly deferred to legal counsel review, pricing committee approval, or external vendor onboarding. None of these block MVP development. All must be resolved before production launch.

### 8.1 Legal Counsel — Required Before Production

| # | Item | Source | Risk if unresolved |
|---|------|--------|--------------------|
| L-01 | FCRA §1681m applicability to Check 5 (income failure): confirm bureau data accessed as prerequisite in the same decisioning chain is sufficient to trigger adverse action notice obligation | `artifacts/specs/adverse-action-spec.md` §5 | Wrong notice framework applied |
| L-02 | Tax lien data source: confirm whether soft pull data includes tax lien tradelines (removed from major bureau files 2017–2018 per NCAP); if absent, `DEROGATORY_TAX_LIEN` must remain deactivated | `artifacts/specs/adverse-action-spec.md` §5 | Reason code applied without corresponding data = FCRA + UDAAP violation |
| L-03 | `DEROGATORY_COMBINED` disposition: confirm it must not function as standalone adverse action reason; document multi-indicator handling plan | `artifacts/specs/adverse-action-spec.md` §4 | Fails Reg B §1002.9(b)(2) specificity requirement |
| L-04 | State-specific adverse action variations: federal baseline confirmed; CA, NY, and applicable states may require additional content, timing, or delivery | `artifacts/specs/adverse-action-spec.md` §5 | State examination exposure |
| L-05 | Reg B pre-qualification characterization: confirm this flow is treated as a Reg B application; no marketing or disclosure language creates a conflicting characterization | `artifacts/specs/adverse-action-spec.md` §5 | Adverse action obligations may be incorrectly disclaimed |
| L-06 | States in scope: California and other states with FCRA analogues may require additional overlay analysis | `artifacts/specs/fcra-consent-routing-spec.md` §6 | State compliance gap |
| L-07 | Qualified legal counsel confirms consent naming Dealer A does not authorize pull/delivery to Dealer B (conservative interpretation) | `artifacts/specs/fcra-consent-routing-spec.md` §6 | Hard-block implementation rationale |
| L-08 | PREQUALIFIED email: confirm it is a transactional communication, not a Reg Z advertisement | `artifacts/specs/prequalified-email-spec.md` §7 | Affects whether 12 CFR 1026.24 formally applies |
| L-09 | ECOA Reg B application characterization: confirm qualifying language satisfies conditions-of-approval notice requirement under 1002.9(a)(1) | `artifacts/specs/prequalified-email-spec.md` §7 | Incomplete notice obligation |
| L-10 | California DFPI exposure: confirm qualifying language satisfies CCFPL requirements for prequalification communications | `artifacts/specs/prequalified-email-spec.md` §7 | DFPI enforcement risk |
| L-11 | Tier 4 usury cap state analysis: confirm consumers in states where 19.9–24.9% APR exceeds auto loan usury caps are excluded from Tier 4 results before those rate ranges appear in consumer communications | `artifacts/specs/prequalified-email-spec.md` §7 | Product eligibility + disclosure accuracy |
| L-12 | Dealer enrollment agreement templates must be drafted and executed before first dealer is onboarded | decisions-log 2026-04-11 | GLBA data use restriction requirement |

### 8.2 Pricing Committee — Required Before Production

| # | Item | Source |
|---|------|--------|
| P-01 | Minimum credit score floor (620): formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-02 | All four tier rate ranges (T1–T4): formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-03 | All 20 cells of max finance amount lookup table: formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-04 | Minimum income threshold ($2,000/month): formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-05 | All derogatory lookback periods: formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-06 | Non-auto charge-off dollar threshold ($1,000): formal approval | `artifacts/specs/credit-policy-spec.md` |
| P-07 | Post-MVP: evaluate brand-split (Toyota vs. Lexus) max finance amount table | `artifacts/specs/credit-policy-spec.md` |

### 8.3 Vendor Onboarding — Required Before Production

| # | Item | Owner |
|---|------|-------|
| V-01 | DealerTrack platform registration and vendor ID acquisition | Product Manager |
| V-02 | RouteOne platform registration and vendor/source code acquisition | Product Manager |
| V-03 | DealerTrack API endpoint URL and authentication header name | Product Manager / Lead Developer |
| V-04 | RouteOne API endpoint URL and authentication method (API key vs. OAuth 2.0 CC) | Product Manager / Lead Developer |
| V-05 | Auth0 DPA (Data Processing Agreement) executed | Product Manager |
| V-06 | AWS Business Associate Agreement (BAA) in place | Product Manager |

### 8.4 Engineering — Required Before Production

| # | Item | Source |
|---|------|--------|
| E-01 | ADR-001 through ADR-005 formal write-ups in `decisions/` directory | architecture.md ADR index |
| E-02 | HMAC lookup key rotation decision: Product Manager to decide whether miss window is acceptable or rotation requires maintenance window | `artifacts/specs/column-encryption-key-spec.md` |
| E-03 | ADF payload exclusion automated test (SSN-4 and stated income absent on every code path) | decisions-log 2026-04-11 |
| E-04 | Step 7 consent UI extension for rate range and max finance conditional consent (before conditional fields can appear in any production payload or email) | `artifacts/specs/adf-payload-spec.md` §7 |

---

## 9. Spec File Reference Index

The following specification files contain authoritative detail for their respective domains. This PRD does not duplicate that content — it references it.

| Spec file | Covers |
|-----------|--------|
| `artifacts/02-service-blueprint.md` | End-to-end service delivery map; failure paths; SLA requirements; integration points |
| `artifacts/specs/adverse-action-spec.md` | Path A and Path B notice elements; approved reason code consumer-facing text; multi-reason handling; legal counsel items |
| `artifacts/specs/fcra-consent-routing-spec.md` | 5 consent routing rules; hard-block vs. soft-warning analysis; implementation confirmations |
| `artifacts/specs/session-verification-spec.md` | JWT claim structure; KBA claim lifetime; invalidation conditions; Auth0 configuration |
| `artifacts/specs/adf-payload-spec.md` | Full ADF 1.0 XML field spec; conditional field consent logic; DealerTrack vs. RouteOne differences; XML envelope template; implementation notes |
| `artifacts/specs/prequalified-email-spec.md` | Email Version A and B content requirements; prohibited fields; regulatory basis; legal counsel items |
| `artifacts/specs/credit-policy-spec.md` | Derogatory indicator definitions; pricing tier table; max finance lookup table; reason code mapping; pricing committee items |
| `artifacts/07-data-dictionary-schema.md` | Full DDL for all tables; column-level encryption fields; seed data |
| `artifacts/08-threat-model.md` | STRIDE threat model; security gate verdict; gate blocker resolutions |
| `artifacts/specs/column-encryption-key-spec.md` | AES-256-GCM key management; startup behavior; rotation procedure |
| `artifacts/specs/idor-enforcement-spec.md` | IDOR middleware; ownership check query; error response spec |
| `artifacts/specs/audit-log-spec.md` | 23 named audit events; immutability controls; PII exclusion rules; retention classification |
| `architecture.md` | System components; infrastructure decisions; ADR index (ADR-001 through ADR-005) |

---

*← [Auto Leads Platform — Progress](../progress.md)*
