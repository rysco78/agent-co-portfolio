# Information Architecture — Auto Leads Platform

> **Agent:** UX Designer
> **Project:** auto-leads-platform
> **Status:** Complete
> **Date:** 2026-04-11
> **Depends on:** PRD §3–§4, api-contract.md

---

## Contents

1. [Site Map](#1-site-map)
2. [URL Structure](#2-url-structure)
3. [Navigation Model](#3-navigation-model)
4. [Page Inventory Table](#4-page-inventory-table)
5. [Next.js Route Architecture](#5-nextjs-route-architecture)
6. [State Routing Model](#6-state-routing-model)
7. [Error Page Inventory](#7-error-page-inventory)

---

## 1. Site Map

### Consumer-Facing Pages

```
/ (Landing)
├── /apply (Multi-step application shell — single route, step managed by server state)
│   ├── Step 1: Account Registration / Auth0 Login
│   │     [REDIRECT OUT → Auth0 Universal Login — auth0.com domain]
│   │     [REDIRECT BACK → /api/auth/callback → /apply]
│   ├── Step 2: Personal Information
│   ├── Step 3: Vehicle Selection
│   ├── Step 4: Dealer Selection
│   ├── Step 5: Income Entry
│   ├── Step 6: OTP Phone Verification       [LOCKED-FORWARD after completion]
│   ├── Step 7: KBA Identity Verification    [LOCKED-FORWARD after completion]
│   ├── Step 8: Review
│   ├── Step 9: FCRA Disclosure & Consent    [LOCKED-FORWARD after acceptance]
│   └── Step 10: Decisioning (loading state — no navigation)
├── /result (Result display — PREQUALIFIED or DECLINED)
│   ├── /result/prequalified
│   └── /result/declined
└── /resume (Resume DRAFT application entry point)
```

### Auth0 External Flow

```
[Auth0 Universal Login — hosted at Auth0 domain, NOT auto-leads.ryanrscott.com]
├── Registration (new consumer)
└── Login (returning consumer)
    └── POST-AUTH REDIRECT → /api/auth/callback
```

### Error Pages

```
/error
├── /error/session-expired
├── /error/otp-lockout
├── /error/system
├── /error/not-found
└── /error/unauthorized
```

### Next.js API Routes

```
/api/
├── /api/auth/
│   ├── /api/auth/login         — initiates Auth0 redirect
│   ├── /api/auth/callback      — handles Auth0 post-auth redirect; exchanges code for token
│   ├── /api/auth/logout        — clears session; redirects to landing
│   └── /api/auth/session       — returns sanitized session state for client use
├── /api/vehicles/
│   ├── /api/vehicles/years     — GET; NHTSA years (brand-aware)
│   ├── /api/vehicles/makes     — GET ?year=&brand=; NHTSA makes filtered to active brand (server-enforced)
│   ├── /api/vehicles/models    — GET ?year=&makeCode=; NHTSA models
│   └── /api/vehicles/trims     — GET ?year=&makeCode=&modelCode=; NHTSA trims
├── /api/dealers/
│   └── /api/dealers            — GET ?brand=&zip=; dealer list filtered by brand + proximity
└── /api/otp/
    ├── /api/otp/send           — POST; dispatches SMS via Amazon SNS
    └── /api/otp/verify         — POST; validates entered code; rotates session token on success
```

### Backend Application Service Routes (Node.js — separate from Next.js)

```
/v1/
├── POST   /v1/applications                        — create application
├── GET    /v1/applications/:id                    — read application state
├── PATCH  /v1/applications/:id/vehicle            — update vehicle selection
├── PATCH  /v1/applications/:id/dealer             — update dealer selection
└── POST   /v1/applications/:id/consent            — accept FCRA disclosure
```

### VPC-Internal Routes (not public-facing)

```
POST /internal/kba/callback     — KBA provider server-to-server callback (VPC-internal only)
```

---

## 2. URL Structure

### 2.1 Brand Encoding in URL

**Recommendation: brand is NOT encoded in the URL path at MVP; it is stored in server-side session state.**

**Rationale:**

The PRD distinguishes between MVP (toggle-controlled brand) and production (URL-driven brand). At MVP, the brand is set by a toggle on the landing page and must persist through the multi-step flow without appearing in the URL. A URL path pattern like `/toyota/apply` or `/apply?brand=toyota` would:

1. Allow a consumer to manually edit the URL and set a brand value that conflicts with the immutable `brand` field on the `applications` record (which is written at `POST /v1/applications` and cannot be changed)
2. Require per-step URL brand validation and matching against the stored application record, adding complexity and a mismatch attack surface
3. Be inconsistent with the MVP toggle mechanism where brand is a session state, not a URL parameter

**MVP approach:** The active brand is set at session initiation (from toggle interaction on landing page) and stored in the server-side session. The `/apply` route reads brand from session at render time, not from URL parameters. The `brand` field on the `applications` record is immutable after creation, eliminating any drift risk between URL state and stored state.

**Production approach (post-MVP):** When URL-driven brand selection is implemented (PRD §4.0 BRAND-03), the platform can adopt subdomains (`toyota.auto-leads.ryanrscott.com`, `lexus.auto-leads.ryanrscott.com`) or path prefixes (`/toyota/`, `/lexus/`). The IA is designed to accommodate this without structural change — the brand session variable is already the single source of truth for all routing and filtering decisions.

### 2.2 Multi-Step Form URL Pattern

**Recommendation: Single route `/apply` with step managed entirely by server-side session state. No `/apply/step/1`, `/apply/step/2`, etc.**

**Rationale:**

The back-navigation locking requirement (OTP, KBA, FCRA disclosure are locked-forward) is incompatible with step numbers in the URL. If step URLs were used:

- A consumer could type `/apply/step/6` directly and reach the OTP screen regardless of whether they completed prior steps
- Browser back/forward buttons would interact with the history stack in ways that circumvent the locking requirement without elaborate `history.replaceState` management on every single transition
- A consumer who completes Step 9 (FCRA disclosure acceptance) and then uses the browser forward button could re-render the consent screen from history

With a single `/apply` route:
- The server controls which step renders based on session state and application status
- `history.replaceState` is called on each step transition so the browser back button always resolves to `/apply`, which the server re-renders at the current authoritative step
- Direct navigation to `/apply` when a DRAFT application exists triggers the resume logic — the consumer sees the step appropriate to their application state
- There is no URL to share, bookmark, or manually edit that bypasses step sequencing

**Step identification for analytics and debugging** uses a non-routable `step` value in server-side session state and audit log events — not in the URL.

### 2.3 Confirmation Page URL

**URLs:** `/result/prequalified` and `/result/declined`

**Bookmarkability mitigation:**

The result pages are server-rendered. Navigation to `/result/prequalified` or `/result/declined` without a valid completed application in the server-side session produces a redirect:

- No session present: redirect to `/`
- Session present but `status` is not `PREQUALIFIED` or `DECLINED` respectively: redirect to `/apply` (for active applications) or `/` (for terminal/expired applications)
- Session present and `status` matches the route: renders result correctly

The confirmation number (`PQ-YYYYMMDD-[8 chars]`) is NOT encoded in the URL. It is read from the server-side session and the `applications` record at SSR render time. Sharing or bookmarking `/result/prequalified` does not re-trigger any application logic.

### 2.4 Auth0 Callback URL

**URL:** `/api/auth/callback`

This is the redirect URI registered in Auth0. After successful authentication, Auth0 redirects to this route. The route handler exchanges the authorization code for tokens, establishes the server-side session, and redirects to `/apply`.

### 2.5 Full URL Inventory

| Page / Route | URL |
|---|---|
| Landing | `/` |
| Application shell (all steps) | `/apply` |
| Resume application | `/resume` |
| Result — PREQUALIFIED | `/result/prequalified` |
| Result — DECLINED | `/result/declined` |
| Auth0 initiate login | `/api/auth/login` |
| Auth0 callback | `/api/auth/callback` |
| Auth0 logout | `/api/auth/logout` |
| Auth0 session state | `/api/auth/session` |
| Vehicle years | `/api/vehicles/years` |
| Vehicle makes | `/api/vehicles/makes` |
| Vehicle models | `/api/vehicles/models` |
| Vehicle trims | `/api/vehicles/trims` |
| Dealer list | `/api/dealers` |
| OTP send | `/api/otp/send` |
| OTP verify | `/api/otp/verify` |
| Error — session expired | `/error/session-expired` |
| Error — OTP lockout | `/error/otp-lockout` |
| Error — system error | `/error/system` |
| Error — not found | `/error/not-found` |
| Error — unauthorized | `/error/unauthorized` |

---

## 3. Navigation Model

### 3.1 Navigation Elements by Zone

| Element | Present on | Notes |
|---|---|---|
| Brand logo (linked) | Landing page only | Links to `/`; not a navigation affordance inside the application flow |
| Brand toggle (MVP) | Landing page only | Switches Toyota/Lexus skin; updates server-side session brand; not present inside the `/apply` shell |
| Branded header (non-linked) | `/apply` shell (all steps), `/result` pages | Logo present as trust signal; not a navigation link; no click action |
| Progress bar (grouped, 4 stages) | `/apply` steps 2–9 | Not shown on Step 1 (redirected to Auth0) or Step 10 (decisioning loading state); stages: Your info / Your vehicle / Your dealer / Verify & submit |
| Step back button | `/apply` steps 2–5, Step 8 only | Enabled on pre-verification steps and the Review screen; absent on OTP (Step 6), KBA (Step 7), FCRA Disclosure (Step 9), and Decisioning (Step 10) |
| In-flow "Change" edit links | Step 8 (Review screen) | Vehicle and dealer only; identity fields (name, address, DOB, SSN) have no edit link and cannot be changed |
| Footer (minimal) | Landing, error pages | Privacy policy link, legal disclaimer; not present inside the `/apply` application shell |
| Session timeout warning modal | All authenticated pages | Triggered at 50-minute idle mark (10 minutes before 60-min idle timeout); blocks all interaction until consumer responds |

### 3.2 Pages with No Navigation (Locked or Terminal States)

| Page / Step | Navigation state | Reason |
|---|---|---|
| Step 6 — OTP (after code submission) | No back button; progress bar visible but not interactive | Locked-forward; OTP completion is a one-way gate |
| Step 7 — KBA | No back button; progress bar visible but not interactive | Locked-forward; KBA result (pass or fail) is terminal for this step |
| Step 9 — FCRA Disclosure | No back button; "Accept and Submit" is the only forward action | Locked-forward per FCRA-06; routing block at frontend layer plus 409 enforcement at server layer |
| Step 10 — Decisioning | No navigation elements at all; no progress bar; no header links | Consumer must wait; no user action is appropriate or possible during decisioning chain execution |
| Result — DECLINED | No back button; no "Try again" CTA at MVP | Terminal application state; adverse action email already dispatched |

### 3.3 Brand Toggle Positioning and Persistence

**MVP (toggle):**
- Toggle is positioned in the landing page header (`/`)
- Toggle is NOT present inside the `/apply` shell — brand is fixed at the moment the consumer initiates the application (when the `/apply` route is first loaded)
- Active brand is stored in the server-side session when the toggle is engaged, not deferred until form submission
- If a consumer returns to `/` while mid-application (e.g., navigates back via the address bar), the toggle reflects the current session brand; changing it at that point does NOT affect the in-progress `applications` record (the `brand` field is immutable after `POST /v1/applications`)
- The toggle must fire a server-side session write — a client-side state change only creates a race condition where the `/apply` shell reads a stale brand from the session

**Production (URL-driven — post-MVP):**
- No toggle rendered; brand derived from entry point URL and written to session at session initiation
- Toggle component is feature-flagged and removed in production configuration

### 3.4 Session Timeout Warning Interaction

- Warning modal triggered at 50 minutes of idle time (10 minutes before the 60-minute idle timeout)
- Modal is a non-dismissable overlay — it blocks all page interaction until the consumer acts
- "Stay logged in" action pings `/api/auth/session` to refresh the idle timer; modal dismissed on success
- "Log out" action calls `/api/auth/logout`; session terminated; consumer redirected to landing
- If the consumer takes no action within 10 minutes of the modal appearing: session expires; consumer redirected to `/error/session-expired` on the next interaction

---

## 4. Page Inventory Table

| Page | URL | Auth required | Back-nav | Notes |
|---|---|---|---|---|
| Landing | `/` | No | N/A | Brand toggle (MVP); "Start application" CTA; Toyota or Lexus skin active |
| Application shell — Step 1 (transition) | `/apply` (pre-auth) | No (initiates auth) | N/A — redirects to Auth0 | Next.js renders a brief transition state before issuing the Auth0 redirect |
| Auth0 Universal Login | Auth0 domain | N/A | Auth0-managed | Not a Next.js page; Auth0 hosted; customizable via Auth0 dashboard branding only |
| Auth0 callback | `/api/auth/callback` | N/A | N/A | API route only; no UI; exchanges code, establishes session, redirects to `/apply` |
| Application shell — Step 2 | `/apply` | Yes | Yes — back to Step 1 landing | Personal info form; PII encrypted server-side; no localStorage/sessionStorage for any field |
| Application shell — Step 3 | `/apply` | Yes | Yes — back to Step 2 | Vehicle selection; make dropdown filtered to active brand (API-enforced, not UI-only) |
| Application shell — Step 4 | `/apply` | Yes | Yes — back to Step 3 | Dealer selection; filtered to active brand and proximity |
| Application shell — Step 5 | `/apply` | Yes | Yes — back to Step 4 | Income entry; stated monthly gross income in cents internally |
| Application shell — Step 6 | `/apply` | Yes | LOCKED after OTP success | OTP phone verification; phone entry then 6-digit code entry; resend link; attempt counter; lockout state |
| Application shell — Step 7 | `/apply` | Yes | LOCKED after KBA result | KBA questions with timer; KBA fail terminates application and routes to DECLINED path |
| Application shell — Step 8 | `/apply` | Yes | Yes — back to Step 5 | Review screen; "Change" links for vehicle and dealer; identity fields read-only with no edit affordance |
| Application shell — Step 9 | `/apply` | Yes | LOCKED after FCRA acceptance | FCRA disclosure; dynamic SSR only; `no-store` cache headers; dealer name resolved at render time from DB |
| Application shell — Step 10 | `/apply` | Yes | None | Decisioning loading state; no user action possible |
| Result — PREQUALIFIED | `/result/prequalified` | Yes | None | Confirmation number, dealer name, vehicle scope, APR range (if consent given), max finance (if consent given), expiration date; SSR from application record |
| Result — DECLINED | `/result/declined` | Yes | None | "Unable to complete your pre-qualification" message only; reason codes not surfaced on-screen; adverse action email dispatched separately |
| Resume application | `/resume` | Yes | N/A | Entry point for returning consumers with a DRAFT application; validates application state and redirects to correct step in `/apply` |
| Error — session expired | `/error/session-expired` | No | None | Shown on idle or absolute timeout; application state preserved for DRAFT; "Start a new application" CTA |
| Error — OTP lockout | `/error/otp-lockout` | Yes | None | 30-minute lockout; countdown timer displayed; application state preserved; "Try again in X minutes" |
| Error — system error | `/error/system` | No | None | Generic unhandled error or 503; "Please try again" CTA |
| Error — not found | `/error/not-found` | No | None | 404 and application-not-found cases (identical response per IDOR spec) |
| Error — unauthorized | `/error/unauthorized` | No | None | 401 cases; "Log in to continue" CTA linking to `/api/auth/login` |

---

## 5. Next.js Route Architecture

### 5.1 Router Recommendation

**Recommendation: App Router throughout.**

The App Router (Next.js 13+) is the correct choice for this platform for the following reasons:

1. **SSR control is explicit per-route.** `export const dynamic = 'force-dynamic'` on the FCRA disclosure step enforces the no-cache SSR requirement (FCRA-02) declaratively. The Pages Router equivalent (`getServerSideProps`) is less explicit and more prone to being omitted by mistake.
2. **Server Components keep PII server-side.** Server Components can access the session and pass non-PII display data to Client Components without a client-side fetch round-trip. PII never needs to transit to the client layer for display (first name in a greeting is the only consumer-visible PII field and can be decrypted server-side and passed as a prop).
3. **Route Groups support layout separation.** The `(application)` route group shares a branded header + progress bar layout without that layout bleeding into the landing page or error pages.
4. **Next.js Middleware enforces auth at the edge.** Middleware can redirect unauthenticated requests before the route renders — preferable to a server-side redirect inside a page component.

Pages Router is not recommended. It does not support Route Groups, its data-fetching conventions are less explicit, and mixing both routers increases cognitive overhead with no compensating benefit at this project scale.

### 5.2 SSR, SSG, and ISR Assignments

| Route | Rendering mode | Rationale |
|---|---|---|
| `/` (Landing) | SSG with ISR (revalidate: 3600) | Static content; brand toggle state is client-side interaction; no per-request server data dependencies |
| `/apply` (all steps) | SSR — `force-dynamic` | Step determination requires session read on every request; form state is server-side session-bound |
| `/result/prequalified` | SSR — `force-dynamic` | Reads application record at render time; confirmation number, dealer name, result fields |
| `/result/declined` | SSR — `force-dynamic` | Reads application record to confirm status before rendering; session validation required |
| `/resume` | SSR — `force-dynamic` | Reads application state to determine redirect target |
| `/error/*` (all error pages) | SSG | Static content; no per-request data |
| `/api/auth/*` | Route handlers — no rendering | Session and token management only |
| `/api/vehicles/*` | Route handlers — short-lived server cache (5 min TTL) | NHTSA API proxy; brand filter is per-request from session so ISR is not applicable; short TTL is acceptable for vehicle catalog data |
| `/api/dealers` | Route handler — no caching | DB query; dealer availability is live |
| `/api/otp/*` | Route handlers — no caching | Transactional; no caching permissible |

**FCRA Disclosure — explicit `no-store` header requirement:**

The FCRA disclosure renders within the `/apply` shell at Step 9. The route is `force-dynamic`. Additionally, the Step 9 server component must explicitly set the following response headers at the HTTP layer:

```
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
```

Both the Next.js rendering mode and the HTTP headers are required. A `force-dynamic` route still reaches a CDN or reverse proxy response cache if `no-store` is not set explicitly. FCRA-02 is satisfied only when both layers are enforced.

### 5.3 Route Group and File Structure

```
app/
├── (marketing)/
│   ├── layout.tsx                        — Landing layout: brand toggle, minimal header, footer
│   └── page.tsx                          — Landing page (/)
├── (application)/
│   ├── layout.tsx                        — Application layout: branded header (no nav), progress bar shell
│   ├── apply/
│   │   └── page.tsx                      — Application shell — all 10 steps render here
│   ├── result/
│   │   ├── prequalified/
│   │   │   └── page.tsx
│   │   └── declined/
│   │       └── page.tsx
│   └── resume/
│       └── page.tsx
├── error/
│   ├── session-expired/
│   │   └── page.tsx
│   ├── otp-lockout/
│   │   └── page.tsx
│   ├── system/
│   │   └── page.tsx
│   ├── not-found/
│   │   └── page.tsx
│   └── unauthorized/
│       └── page.tsx
├── api/
│   ├── auth/
│   │   ├── login/
│   │   │   └── route.ts
│   │   ├── callback/
│   │   │   └── route.ts
│   │   ├── logout/
│   │   │   └── route.ts
│   │   └── session/
│   │       └── route.ts
│   ├── vehicles/
│   │   ├── years/
│   │   │   └── route.ts
│   │   ├── makes/
│   │   │   └── route.ts
│   │   ├── models/
│   │   │   └── route.ts
│   │   └── trims/
│   │       └── route.ts
│   ├── dealers/
│   │   └── route.ts
│   └── otp/
│       ├── send/
│       │   └── route.ts
│       └── verify/
│           └── route.ts
└── middleware.ts                          — Auth guard for /apply, /result/*, /resume
```

### 5.4 Next.js Middleware (Authentication Guard)

`middleware.ts` matches the following path patterns and enforces authentication before any route renders:

```typescript
export const config = {
  matcher: ['/apply', '/result/:path*', '/resume'],
}
```

Unauthenticated requests to these paths are redirected to `/api/auth/login`, which initiates the Auth0 redirect. The middleware checks only for a valid session cookie — it does not read application state or verify step eligibility. Step-level authorization (e.g., confirming OTP completion before rendering the KBA step) is handled within the `/apply` page's server component using the step determination matrix (§6.1).

---

## 6. State Routing Model

### 6.1 Step Determination Matrix

The active step is determined server-side on every request to `/apply`. The determination uses two inputs:

1. **Application status** — from the `applications` record via `GET /v1/applications/:id`
2. **Verification claims** — from the server-side session cache (Redis `verification:{session_id}`)

The step rendered is the lowest incomplete step the consumer is eligible to access, subject to locked-forward rules. There is no client-side step counter. The client renders whatever step the server determines.

| Application status | OTP verified | KBA verified | Disclosure status | Step rendered |
|---|---|---|---|---|
| No application record | — | — | — | Step 2 (personal info; application created on submit of this step) |
| `DRAFT` | false | false | `not_yet_presented` | Lowest incomplete step among Steps 2–5 |
| `DRAFT` | false | false | `not_yet_presented` | Step 6 (OTP) — if Steps 2–5 all complete |
| `DRAFT` | true | false | `not_yet_presented` | Step 7 (KBA) |
| `IDENTITY_VERIFIED` | true | true | `not_yet_presented` | Step 8 (Review) |
| `IDENTITY_VERIFIED` | true | true | `not_yet_presented` | Step 9 (FCRA disclosure) — after Review confirmation action |
| `DISCLOSURE_ACCEPTED` | true | true | `accepted` | Step 10 (Decisioning loading state) |
| `DECISIONING` | true | true | `accepted` | Step 10 (Decisioning loading state) |
| `PREQUALIFIED` | — | — | — | Redirect to `/result/prequalified` |
| `DECLINED` | — | — | — | Redirect to `/result/declined` |
| `ABANDONED` | — | — | — | Redirect to `/` with "previous application expired" messaging |
| `EXPIRED` | — | — | — | Redirect to `/` with "previous application expired" messaging |
| `IDENTITY_VERIFIED` + KBA claim expired (> 60 min) | true | true (expired) | `not_yet_presented` | Redirect to `/` — must start new application (KBA-07) |

### 6.2 Direct Navigation to a Later Step URL

Because all steps render under the single `/apply` route, there is no step-specific URL to navigate to. A consumer typing `/apply` in the browser address bar at any point will see the step the server determines they are on — based on session and application state — regardless of what they intended.

This eliminates the class of vulnerability or UX failure where a consumer navigates directly to a step URL that bypasses prior steps or circumvents a locked-forward gate.

### 6.3 Back Navigation from Non-Locked Steps

For steps where back-navigation is permitted (Steps 2–5, Step 8):

- The consumer activates the back button within the UI
- The Next.js server action or API route processes the navigation, reads the prior step from session state, and re-renders `/apply` at the previous step
- `history.replaceState` is called on each forward step transition, so the browser back button at any step returns the consumer to `/apply`, which the server resolves to the correct previous step

The browser history stack never contains step-specific entries. The only URL in the consumer's history stack is `/apply`.

### 6.4 Resume Flow for DRAFT Applications

The `/resume` route handles returning-consumer entry:

1. Consumer arrives at `/resume` (e.g., from a re-engagement email link, or by returning to the platform)
2. Next.js Middleware confirms the consumer has an active session; if not, redirects to `/api/auth/login`
3. Server reads the consumer's active application (status `DRAFT` or `IDENTITY_VERIFIED`) via `GET /v1/applications/:id`
4. Server applies the step determination matrix (§6.1) to identify the resume step
5. Server writes the resume step indicator to the session and redirects to `/apply`
6. `/apply` renders the resume step

**Resume is blocked when:**
- Application status is `IDENTITY_VERIFIED` and the KBA claim has expired (> 60 minutes since `kba_verified_at`) — redirect to `/` with a message to start a new application (KBA-07; the 60-minute KBA window will have elapsed)
- Application is in a terminal state (`PREQUALIFIED`, `DECLINED`, `ABANDONED`, `EXPIRED`) — redirect to the appropriate result page or to `/`

**DRAFT application resume after session expiry:**
A `DRAFT` application (Steps 2–5) is resumable on re-authentication. The consumer logs in, `/resume` resolves the application, and the step determination matrix routes them to the lowest incomplete step. No re-entry of prior completed steps is required.

### 6.5 Browser Back Button on Locked-Forward Screens

On Steps 6 (OTP — post-completion), 7 (KBA), and 9 (FCRA Disclosure — post-acceptance):

1. On step completion, `history.replaceState` replaces the current history entry with `/apply` at the new step state. There is no "OTP completed" entry in the history stack for the consumer to navigate back to.
2. If the consumer presses the browser back button from any subsequent step, they navigate to `/apply`. The server's step determination matrix renders the current authoritative step — it will not render the locked step.
3. The server enforces the lock independently of any client-side routing state: a request to `/apply` when the session indicates a locked-forward step has been completed will always render the current step.

**FCRA disclosure hard-block (Option A):**
After `disclosure_status = 'accepted'`, the server returns `409 conflict` on any write attempt (`PATCH /v1/applications/:id/vehicle`, `PATCH /v1/applications/:id/dealer`). The step determination matrix will not render Step 9 again once `disclosure_status = 'accepted'` — it renders Step 10 (Decisioning). A consumer who starts over after acceptance has their application set to `ABANDONED` server-side and must begin a new application.

---

## 7. Error Page Inventory

### 7.1 Dedicated Error Pages

| Error state | URL | Trigger | User-facing message | Primary action |
|---|---|---|---|---|
| Session expired | `/error/session-expired` | Idle timeout (60 min) or absolute timeout (4 hours) elapsed; session cookie expired or revoked | "Your session has expired for your security. Any progress you made has been saved." | "Start a new application" (links to `/`) |
| OTP lockout | `/error/otp-lockout` | 5 failed OTP attempts or 3 resends exhausted; 30-minute lockout triggered (OTP-06) | "You've exceeded the verification limit. You can try again in [countdown timer]." | Countdown timer displayed; "Try again" button enabled when lockout expires; application state preserved throughout |
| System error | `/error/system` | Unhandled server error (500); upstream 503 (Redis + Management API both unavailable; bureau or KBA timeout at decisioning) | "Something went wrong on our end. Please try again." | "Try again" (returns consumer to `/apply`); secondary "Contact support" link |
| Application not found | `/error/not-found` | `GET /v1/applications/:id` returns 404 (application does not exist or not owned by authenticated consumer — identical response per IDOR spec) | "We couldn't find your application." | "Start a new application" (links to `/`) |
| Unauthorized | `/error/unauthorized` | 401 returned from any authenticated route; consumer reaches an authenticated route without a valid session and middleware redirect did not fire | "Please log in to continue." | "Log in" (links to `/api/auth/login`) |

### 7.2 Inline Error States (Not Dedicated Pages)

The following error conditions are handled inline within the step where they occur and do not route to a dedicated error page:

| Error condition | Handling location | Handling approach |
|---|---|---|
| Form field validation errors (Steps 2–5) | Below the affected field | Inline error messages; form does not submit; focus moves to first invalid field |
| OTP code incorrect, attempts remaining | OTP code entry field | Inline: "Incorrect code. X attempts remaining." |
| OTP code expired | OTP code entry field | Inline: "Code expired. Please request a new code." |
| Resend cooldown active | "Resend" link | Link disabled with countdown timer displayed inline; re-enables after 60 seconds |
| KBA fail (terminal outcome) | `/apply` Step 7 → transition to `/result/declined` | "Unable to complete" message shown; application terminates; redirect to `/result/declined`; Path A adverse action email dispatched |
| FCRA consent — dealer change after acceptance | `/apply` Step 9 re-render | Inline notice: disclosure re-presented with new dealer name; prior acceptance voided; no error page |
| Rate limit exceeded (429) | Current step | Inline: "Too many requests. Please wait X seconds before trying again." `Retry-After` header drives countdown |
| Vehicle brand mismatch (400 `invalid_brand`) | Should not surface in a correctly built UI (make dropdown is API-filtered before submission); if it occurs via API manipulation, treated as a system error and routes to `/error/system` | — |
| Decisioning system unavailable during Step 10 | `/apply` Step 10 loading state | After 10-second timeout threshold (KBA timeout threshold per PRD §5.3), inline: "Unable to complete your pre-qualification"; application terminates; routes to `/result/declined` |

### 7.3 Modal Overlays

| Overlay | Trigger | Actions |
|---|---|---|
| Session timeout warning | 50 minutes of idle time (10 min before 60-min idle timeout) | "Stay logged in" — pings `/api/auth/session`, refreshes idle timer, dismisses modal; "Log out" — calls `/api/auth/logout` |
| Start over confirmation | Consumer attempts to navigate away from `/apply` mid-flow (address bar change, refresh, tab close) | Browser `beforeunload` event; native browser dialog only — custom `beforeunload` dialogs are restricted by browsers and should not be relied upon |

---

## Design Notes for Lead Developer Handoff

The following IA decisions carry direct implementation implications that must be respected:

1. **Single `/apply` route — step is a server-side session variable, not a URL param or client-side React state.** The page component receives `currentStep` as a prop from the server. Client React state reflects this for rendering; it does not own it.

2. **`history.replaceState` on every forward step transition.** Call `replaceState` (not `pushState`) on each step advance. The browser back button stack must never accumulate step-specific entries.

3. **FCRA disclosure route must set both `dynamic = 'force-dynamic'` AND explicit `Cache-Control: no-store` response headers.** SSR rendering mode alone is insufficient if a CDN or reverse proxy is in the deployment path.

4. **`/api/vehicles/makes` must enforce brand filtering server-side using the session brand.** The API route must not accept a client-supplied brand parameter as authoritative — it must read brand from the server-side session and validate any supplied parameter against it (BRAND-05).

5. **`/resume` must check KBA claim age before routing.** A consumer resuming an `IDENTITY_VERIFIED` application where `kba_verified_at` is > 60 minutes ago must be redirected to start a new application, not to Step 8 (Review).

6. **Brand toggle must write to server-side session synchronously before the consumer can enter `/apply`.** A client-state-only toggle creates a race condition where the `/apply` shell reads a stale brand from the session on first render.

7. **`/api/auth/callback` is the only Auth0 redirect URI.** Do not register step-specific callback URLs. Post-auth, all consumers land at `/apply` via the callback handler.

---

*← [Auto Leads Platform — Progress](../progress.md)*
