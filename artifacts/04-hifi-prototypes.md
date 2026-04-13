# Hi-Fi Prototype Specification — Auto Leads Platform

> **Agent:** UX Designer
> **Project:** auto-leads-platform
> **Status:** Complete
> **Date:** 2026-04-11
> **Depends on:** PRD §3–§4, fcra-consent-routing-spec.md, adverse-action-spec.md, prequalified-email-spec.md

---

## Figma Links

| Skin | Link |
|------|------|
| **Toyota Prototype** | [View in Figma →](https://www.figma.com/design/qKcFHFpUKAtM4ivFYitvlC/Auto-Leads-Platform-%E2%80%94-Hi-Fi-Prototype?node-id=0-1&t=GVCLSPcG693flX6O-1) |
| **Lexus Prototype** | [View in Figma →](https://www.figma.com/design/qKcFHFpUKAtM4ivFYitvlC/Auto-Leads-Platform-%E2%80%94-Hi-Fi-Prototype?node-id=95-62&t=GVCLSPcG693flX6O-1) |
| **Full File** | [View in Figma →](https://www.figma.com/design/qKcFHFpUKAtM4ivFYitvlC/Auto-Leads-Platform-%E2%80%94-Hi-Fi-Prototype?node-id=4-81&t=5t94Zz1eK8HLiU38-1) |

---

## How to read this document

This is a screen-by-screen interaction specification. It is the source of truth for what appears on each screen, what states each screen renders, what interactions are available, and how the screen behaves on mobile vs desktop. The Lead Developer builds from this document. The QA Analyst writes test cases from this document.

**Out of scope here:**
- Adverse action email templates — see `adverse-action-spec.md`
- PREQUALIFIED confirmation email templates — see `prequalified-email-spec.md`
- ADF payload construction — see `adf-payload-spec.md`
- Credit policy thresholds — see `credit-policy-spec.md`

**Display conventions used throughout:**
- Monetary amounts display in dollars, converted from cents at the display layer. The integer `4500000` displays as `$45,000`.
- Rate ranges display as `X.XX% – X.XX% APR`, converted from basis points at the display layer. `590` basis points displays as `5.90%`. Always two decimal places.
- Confirmation numbers display as `PQ-YYYYMMDD-[8 chars]`.

---

## Global elements

### Progress bar

The 4-stage grouped progress bar appears on all application steps. It is absent from the landing screen, the Auth0 login screen, decisioning result screens, session-state screens (timeout warning, timeout expired, OTP lockout, error screen), and the Auth0 hosted login page.

**Stage names and step mapping:**

| Stage | Steps included |
|-------|---------------|
| Your info | Personal info, Income |
| Your vehicle | Vehicle selection |
| Your dealer | Dealer selection |
| Verify & submit | OTP, KBA, Review, FCRA disclosure |

**Visual states per stage:**
- **Completed:** filled bar segment, checkmark or solid fill; non-interactive
- **Active:** highlighted bar segment, stage label bold
- **Upcoming:** unfilled or muted bar segment; non-interactive

**Accessibility:** `role="progressbar"`, `aria-valuemin="1"`, `aria-valuemax="4"`, `aria-valuenow="[current stage number]"`, `aria-label="Application progress: [Stage name], step [N] of 4"`. The current stage label is announced to screen readers on navigation. Do not rely on color alone to communicate progress state — use fill and label together.

### Brand header

The brand logo and, on the MVP demo build only, the brand toggle switch appear in the page header on all consumer-facing screens. The toggle is not present in production builds. On production builds the brand is determined by the entry point URL.

**Brand toggle (MVP demo only):**
- Toggle label: "Toyota / Lexus"
- Switching the toggle immediately applies the new brand skin (colors, logo, typography) and resets the vehicle make dropdown filter to the newly active brand's make set.
- If the consumer has already selected a vehicle from the prior brand, the vehicle selection is cleared and the consumer is returned to the vehicle selection step with the new brand's make filter active.
- `role="switch"`, `aria-checked="[toyota|lexus]"`, `aria-label="Brand: [Toyota|Lexus]"`.

### Back button

Back navigation behaves differently by screen. The back button label is "Back." Where back navigation is permitted within the flow, the button appears in the page footer or below the form, left-aligned.

On OTP (once code entry begins), KBA, and FCRA disclosure screens, back navigation is locked-forward. The browser back button is suppressed via History API `pushState` manipulation. A back press on the FCRA disclosure screen after acceptance triggers the server-side 409 response; display the locked state message rather than navigating.

---

## Screen 1 — Landing / Entry Point

**Route:** `/`
**Progress bar stage:** N/A
**Back-navigation:** N/A

### Content and components

- Brand logo (Toyota or Lexus per active skin)
- MVP demo only: brand toggle switch (see Global elements)
- Headline: "See if you prequalify for a new [Toyota / Lexus]"
- Subheadline: "Get an instant estimate in minutes — no impact to your credit score."
- Supporting copy (3 bullets, one sentence each):
  - "Tell us about yourself and the vehicle you're interested in."
  - "Verify your identity with a quick phone and identity check."
  - "Receive an instant prequalification estimate — then visit your dealer."
- Primary CTA button: "Get started"
- Secondary link below CTA: "Already started? Sign in" — navigates to Auth0 Universal Login
- Disclosure line (small text below CTA): "Submitting this form initiates a soft credit inquiry, which does not affect your credit score. A hard credit inquiry will be conducted by the dealer and lender at the time of full financing application."
- Legal footer: links to Privacy Policy and Terms of Use (placeholder links at MVP)

### States

- **Default:** Page renders with active brand skin; brand toggle in correct position for active brand.
- **Loading:** N/A — static marketing screen; no server data required on initial render.
- **Error:** N/A on this screen.

### Interactions

- "Get started" button: navigates to `/api/auth/login`. The Auth0 Universal Login page loads in the same browser window.
- "Already started? Sign in" link: same destination as "Get started."
- Brand toggle (MVP only): switches brand skin and updates all brand-filtered content. No navigation occurs. Toggle state persisted to a cookie scoped to the domain.

### Mobile vs desktop

- Mobile: single-column layout; headline and subheadline above bullets; CTA full-width below bullets.
- Desktop: two-column layout acceptable — headline + CTA on left, a brand-appropriate vehicle image on right. Image is decorative (`alt=""`).
- Bullet points stack vertically on both breakpoints.

### Accessibility notes

- "Get started" is a `<button>` or `<a>` with meaningful label. Do not use a div with an `onClick`.
- Brand toggle uses `role="switch"` pattern.
- Soft pull disclosure text must meet 4.5:1 color contrast ratio against the background even at small size.
- Decorative vehicle image uses `alt=""`.
- Page title: "[Toyota / Lexus] Prequalification — Get Started"

---

## Screen 2 — Auth0 Universal Login (Registration / Sign In)

**Route:** Auth0-hosted (e.g., `your-tenant.auth0.com/u/login`); returns to `/apply/personal-info` on success
**Progress bar stage:** N/A (Auth0-hosted page; platform controls surrounding context only)
**Back-navigation:** Auth0-managed; user may navigate back to landing from Auth0 page

### Content and components

The Auth0 Universal Login page is hosted and rendered by Auth0. The platform controls:
1. **Branding applied via Auth0 Universal Login customization:** logo, primary color, button color, font family, background color. These must match the active brand skin.
2. **Page title in the browser tab:** "[Toyota / Lexus] — Sign In / Create Account" (set in Auth0 tenant branding).
3. **The post-login redirect URI:** set to `/apply/personal-info` on the active domain.
4. **The application name** shown in the Auth0 login widget: "Toyota Prequalification" or "Lexus Prequalification" per active brand.

Auth0 renders the following UI components (not platform-controlled content, listed for QA awareness):
- Email address field
- Password field
- "Continue" / "Sign In" button
- "Sign up" tab / toggle for new account creation
- Registration: email, password, confirm password (Auth0 default signup form)
- HIBP breach detection warning (shown inline by Auth0 when a compromised password is entered)

**Platform responsibilities on the surrounding page (if a custom login wrapper is used):**
- Show the brand logo and brand-appropriate header.
- No custom credential fields may be placed inside the Next.js application layer.
- "Persistent session" / "Remember me" must not be displayed or offered.

### States

- **Default:** Auth0 renders the login form; platform brand styling applied.
- **Loading:** Auth0-managed; submission shows Auth0's loading state.
- **Error:** Auth0-managed inline errors for incorrect credentials, locked accounts, HIBP-blocked passwords.
- **Registration success:** Auth0 redirects to `/apply/personal-info` with session token issued.

### Interactions

All credential input and authentication is handled by Auth0. On successful authentication: Auth0 redirects to the configured callback URL; the platform's `/api/auth/callback` handler processes the token and redirects to `/apply/personal-info`.

### Mobile vs desktop

Auth0 Universal Login renders responsively per Auth0's default behavior. Platform brand customization via Auth0 CSS variables handles this.

### Accessibility notes

- Auth0 Universal Login is WCAG 2.1 AA compliant by default. Confirm that any custom CSS overrides applied for brand theming do not reduce color contrast below 4.5:1 for text or 3:1 for UI components.
- Font sizes applied via Auth0 branding settings must be at minimum 16px on mobile.
- Do not suppress Auth0's default focus ring styles in brand CSS overrides.

---

## Screen 3 — Step 1: Personal Information

**Route:** `/apply/personal-info`
**Progress bar stage:** Your info (Stage 1 active)
**Back-navigation:** Allowed — back navigates to landing page (`/`). Application record remains in `DRAFT` state; navigating back does not delete the draft.

### Content and components

- Page heading (H1): "Tell us about yourself"
- Subheading: "Your information is encrypted and used only for your prequalification."
- Form: see fields table below
- Primary CTA button: "Continue"
- Back link: "Back" (returns to `/`)

### Form fields

| Field | Type | Validation | Error message |
|-------|------|-----------|--------------|
| First name | Text input | Required; 1–50 characters; letters, hyphens, apostrophes, spaces only | "Please enter your first name." |
| Last name | Text input | Required; 1–50 characters; letters, hyphens, apostrophes, spaces only | "Please enter your last name." |
| Street address | Text input | Required; max 100 characters | "Please enter your street address." |
| Apt / unit (optional) | Text input | Optional; max 50 characters | N/A |
| City | Text input | Required; max 60 characters | "Please enter your city." |
| State | Select dropdown | Required; 50 US states + DC | "Please select your state." |
| ZIP code | Text input | Required; 5 digits; US ZIP format | "Please enter a valid 5-digit ZIP code." |
| Date of birth | Date input (MM/DD/YYYY) | Required; consumer must be 18+; date must be in the past | "Please enter a valid date of birth." / "You must be at least 18 years old to apply." |
| Last 4 of Social Security Number | Numeric text input (4 digits) | Required; exactly 4 digits; numeric only | "Please enter the last 4 digits of your Social Security Number." |

**SSN last 4 field notes:**
- Input type: `type="text"` with `inputmode="numeric"` and `pattern="[0-9]{4}"`.
- Mask display: show as bullets (`••••`) after entry; unmask icon (eye icon) allows temporary reveal.
- Do not use `type="password"` — it triggers password manager prompts.
- The field must be labeled: "Last 4 digits of Social Security Number."
- `autocomplete="off"`.

### States

- **Default:** All fields empty; CTA disabled until all required fields pass client-side validation.
- **Loading:** On "Continue" click, CTA shows loading state (spinner replaces button label; button disabled); no field changes permitted during submission.
- **Inline validation errors:** Errors appear below the relevant field, associated via `aria-describedby`. Errors appear on blur (field exit) or on submit attempt. Red border on field; error icon adjacent to error message text.
- **Server error:** If the server returns a non-validation error (e.g., 500), display an inline error banner at the top of the form: "Something went wrong. Please try again." CTA re-enabled.
- **Success:** Navigation to `/apply/vehicle`.

### Interactions

- "Continue" button: client-side validation runs first; if valid, form submitted via `POST /api/apply/personal-info`; on `200`, navigate to `/apply/vehicle`; on `422`, display inline field errors returned by server; on `5xx`, display inline error banner.
- Back link: navigates to `/`; application draft preserved server-side.
- SSN unmask icon: toggles between masked and unmasked display for the SSN field. `aria-pressed` toggles accordingly. `aria-label="Show SSN"` / `aria-label="Hide SSN"`.

### Mobile vs desktop

- Mobile: single-column layout; all fields stacked vertically; address fields (street, apt, city, state, ZIP) full-width single-column.
- Desktop: two-column layout for name row (first name left, last name right); city/state/ZIP in a two-up or three-up row; DOB and SSN last 4 in a two-column row.
- CTA button: full-width on mobile; right-aligned fixed-width on desktop.

### Accessibility notes

- All form fields have visible `<label>` elements, not placeholder-only labels.
- Required fields indicated with an asterisk (*) in the label; screen reader reads "required" via `aria-required="true"`.
- Date of birth: if using three separate select dropdowns (Month/Day/Year), each has its own label. If using a single date input, include helper text `MM/DD/YYYY` as `aria-describedby` hint text (not placeholder).
- SSN last 4: label explicitly reads "Last 4 digits of Social Security Number" (not "SSN last 4"). `aria-label="Last 4 digits of SSN, masked"` on the masked display state.
- Error summary at top of form on submit-attempt with multiple errors: a focus-managed `role="alert"` listing all errors and linking to each field.
- Page title: "[Brand] Prequalification — Personal Information"

---

## Screen 4 — Step 2: Vehicle of Interest

**Route:** `/apply/vehicle`
**Progress bar stage:** Your vehicle (Stage 2 active)
**Back-navigation:** Allowed — back navigates to `/apply/personal-info`.

### Content and components

- Page heading (H1): "What vehicle are you interested in?"
- Subheading: "Select the new [Toyota / Lexus] you'd like to finance."
- Brand filter note (informational, small text below subheading): "Showing [Toyota / Lexus] vehicles only." (Reflects active brand skin.)
- Cascading dropdown form: Year → Make → Model → Trim
- Helper text below the dropdowns: "Don't see your vehicle? Only new [Toyota / Lexus] models are eligible."
- Primary CTA button: "Continue"
- Back link: "Back"

### Form fields

| Field | Type | Validation | Error message |
|-------|------|-----------|--------------|
| Year | Select dropdown | Required; populated from NHTSA API; years in descending order; current model year + 1 (if available) at top | "Please select a model year." |
| Make | Select dropdown | Required; populated from NHTSA API filtered to active brand; disabled until Year is selected | "Please select a make." |
| Model | Select dropdown | Required; populated from NHTSA API filtered by Year + Make; disabled until Make is selected | "Please select a model." |
| Trim | Select dropdown | Required; populated from NHTSA API filtered by Year + Make + Model; disabled until Model is selected | "Please select a trim level." |

**Cascading behavior:**
- Selecting Year: enables Make dropdown; fetches NHTSA makes filtered to active brand.
- Selecting Make: enables Model dropdown; fetches NHTSA models for selected Year + Make.
- Selecting Model: enables Trim dropdown; fetches NHTSA trims for selected Year + Make + Model.
- Changing a parent selection clears all downstream selections and resets those dropdowns to their disabled/placeholder state.

**Brand filter enforcement:**
- The Make dropdown only presents makes matching the active brand. This is enforced at the API layer. The dropdown cannot show or submit non-brand makes.
- If the consumer switches brands after selecting a vehicle, vehicle selection is cleared and all dropdowns reset.

### States

- **Default:** Year dropdown active; Make, Model, Trim dropdowns disabled with placeholder text ("Select year first", "Select make first", "Select model first"). CTA disabled.
- **Loading — dropdown fetch:** Each dropdown shows a loading spinner or "Loading..." option while NHTSA data is being fetched. Parent dropdown disabled during the child fetch.
- **Loading — CTA submit:** CTA shows spinner, button disabled, dropdowns disabled.
- **NHTSA API error:** If a dropdown fetch fails, display inline error below the dropdown: "Unable to load vehicles. Please try again." with a retry link. CTA remains disabled.
- **Success:** Navigation to `/apply/dealer`.

### Interactions

- Each dropdown change triggers an API fetch for the next level.
- "Continue" button: submits selected vehicle via `POST /api/apply/vehicle`; on `200`, navigate to `/apply/dealer`; on `422`, display inline error; on `5xx`, display banner error.
- Back link: navigates to `/apply/personal-info`; vehicle selection state cleared server-side.

### Mobile vs desktop

- Mobile: dropdowns full-width, stacked vertically.
- Desktop: single-column vertical stack is recommended for clarity — the cascade dependency is clearer in a vertical list. A 2x2 grid (Year + Make / Model + Trim) is acceptable.

### Accessibility notes

- Each dropdown has a distinct visible `<label>`.
- Disabled dropdowns have `aria-disabled="true"` and the `disabled` attribute. The reason is conveyed via `aria-describedby` or helper text ("Select a year first to enable this field").
- Loading state in dropdown: `aria-busy="true"` on the dropdown wrapper.
- On cascade reset (parent changed): focus moves to the newly enabled child dropdown.
- NHTSA API error: `role="alert"` on the error message so screen readers announce it immediately.
- Page title: "[Brand] Prequalification — Vehicle Selection"

---

## Screen 5 — Step 3: Dealer Selection

**Route:** `/apply/dealer`
**Progress bar stage:** Your dealer (Stage 3 active)
**Back-navigation:** Allowed — back navigates to `/apply/vehicle`.

### Content and components

- Page heading (H1): "Choose your preferred dealer"
- Subheading: "Select a [Toyota / Lexus] dealer near you. This dealer will receive your prequalification lead if you're approved."
- ZIP code field: pre-populated from personal info (Step 1); editable; "Update" button or auto-refresh on change.
- Dealer list: scrollable list of nearby dealers, brand-filtered, ordered by proximity.
- Each dealer list item:
  - Dealer name (bold)
  - Address (street, city, state, ZIP)
  - Estimated distance from consumer ZIP (e.g., "3.2 miles")
  - Selection: radio button per row; selected dealer shows a checkmark or filled visual indicator
- If returning from a vehicle change (Review → vehicle changed → re-routed through dealer selection): the previously selected dealer appears at the top of the list with a "Previously selected" label and is pre-selected. Consumer may change selection.
- Primary CTA button: "Continue"
- Back link: "Back"

### States

- **Default:** Dealer list populated; no dealer selected (first visit). CTA disabled until a dealer is selected.
- **Pre-populated (vehicle change return path):** Previously selected dealer at top of list, pre-selected. CTA enabled. Consumer may scroll and select a different dealer.
- **Loading — initial list fetch:** Skeleton list items (3–5 skeleton rows) while dealer data loads.
- **Loading — CTA submit:** CTA spinner, button disabled.
- **Empty state:** If no dealers found for the consumer's ZIP: "We couldn't find any [Toyota / Lexus] dealers near you. Try updating your location." with ZIP update field active.
- **Error state:** If dealer list fetch fails: "Unable to load dealers. Please try again." with retry action. CTA disabled.
- **Success:** Navigation to `/apply/income`.

### Interactions

- Selecting a dealer row: highlights the selected dealer and enables the CTA. Selection is pending until "Continue" is submitted.
- "Continue" button: submits selected dealer via `POST /api/apply/dealer`; on `200`, navigate to `/apply/income`; on error, display inline error.
- ZIP update: consumer updates ZIP; dealer list refreshes. Filtered to brand-matching dealers only.
- Back link: navigates to `/apply/vehicle`.

### Mobile vs desktop

- Mobile: list items are full-width cards; the full row is the tap target (minimum 44px height). Radio button visible but the full row is interactive.
- Desktop: single-column list preferred. A two-column grid of dealer cards is acceptable if card width allows all content without truncation.

### Accessibility notes

- Dealer list implemented as a group of radio inputs (`<fieldset>` + `<legend>` "Select a dealer"). Each radio button `<label>` contains the full dealer name and address.
- Selected state: `aria-checked` on the radio; selected row has a visual indicator beyond color (checkmark icon, bold border, or both).
- Skeleton loading: `aria-busy="true"` on the list container; skeleton items are `aria-hidden="true"`.
- "Previously selected" label: associated with the dealer row via `aria-describedby`.
- Page title: "[Brand] Prequalification — Dealer Selection"

---

## Screen 6 — Step 4: Income Entry

**Route:** `/apply/income`
**Progress bar stage:** Your info (Stage 1; income completes this stage)
**Back-navigation:** Allowed — back navigates to `/apply/dealer`.

### Content and components

- Page heading (H1): "What is your monthly gross income?"
- Subheading: "Enter your monthly income before taxes. This information is not shared with the dealer."
- Income input field: single currency input
- Helper text below field: "Enter your total monthly income from all sources, before taxes."
- Disclosure text (small, below helper): "Your income is used only to estimate your prequalification eligibility. It is not verified against third-party sources and is not included in the information shared with your dealer."
- Primary CTA button: "Continue"
- Back link: "Back"

### Form fields

| Field | Type | Validation | Error message |
|-------|------|-----------|--------------|
| Monthly gross income | Currency input (dollars) | Required; numeric; > $0; stored as integer cents internally | "Please enter your monthly gross income." / "Please enter a valid income amount." |

**Currency input formatting:**
- Display: `$` prefix; comma-formatted on blur (e.g., `$5,000`).
- `inputmode="numeric"`; strip commas and `$` before validation and submission.
- Do not enforce the $2,000 minimum on the client — the server applies the eligibility check. Displaying a client-side minimum creates a misleading signal about eligibility.

### States

- **Default:** Field empty; CTA disabled until a valid amount is entered.
- **Loading — submit:** CTA spinner, button disabled.
- **Error:** Inline error below field.
- **Success:** Navigation to `/apply/otp`.

### Interactions

- "Continue": submits income via `POST /api/apply/income`; on `200`, navigate to `/apply/otp`.
- Back: navigates to `/apply/dealer`.

### Mobile vs desktop

- Single-column layout on both breakpoints.
- Income field: full-width on mobile; fixed width (approximately 280px) on desktop, left-aligned.

### Accessibility notes

- Field label: "Monthly gross income" as a visible `<label>` (not placeholder-only).
- Currency prefix `$` must be presented in context — either as a visible prefix character in the field styling, or via `aria-label="Monthly gross income in US dollars"`.
- Helper and disclosure text associated via `aria-describedby`.
- Page title: "[Brand] Prequalification — Income"

---

## Screen 7 — Step 5: OTP Phone Verification

**Route:** `/apply/otp`
**Progress bar stage:** Verify & submit (Stage 4 active)
**Back-navigation:** Back is allowed from the phone entry sub-state (back navigates to `/apply/income`). LOCKED-FORWARD once OTP is successfully verified — browser back button is blocked on this screen and all subsequent screens after successful verification.

This screen has two sequential sub-states rendered on the same route: Phone entry and Code entry.

---

### Sub-state A: Phone entry

#### Content and components

- Page heading (H1): "Verify your phone number"
- Subheading: "We'll send a 6-digit verification code to confirm your identity."
- Phone number input field
- Disclosure text: "Your phone number is encrypted and used only for identity verification. Standard messaging rates may apply."
- Primary CTA button: "Send code"
- Back link: "Back" (navigates to `/apply/income`)

#### Form fields

| Field | Type | Validation | Error message |
|-------|------|-----------|--------------|
| Phone number | Tel input | Required; valid US phone number (10 digits after stripping formatting) | "Please enter a valid US phone number." |

Phone input: `type="tel"`, `inputmode="numeric"`, `autocomplete="tel"`. Apply `(NXX) NXX-XXXX` mask on blur.

#### States

- **Default:** Field empty; CTA disabled until valid phone entered.
- **Loading:** CTA spinner, button disabled during SMS dispatch.
- **Error — invalid phone:** Inline field error.
- **Error — SMS dispatch failure:** Banner: "We couldn't send a code to this number. Please check the number and try again."
- **Success:** Transition to Code entry sub-state.

---

### Sub-state B: Code entry

#### Content and components

- Page heading (H1): "Enter your verification code"
- Subheading: "We sent a 6-digit code to [masked phone number, e.g., (•••) •••-1234]. It expires in 10 minutes."
- OTP code input: 6-digit numeric entry. May be rendered as 6 individual single-digit inputs or a single 6-character input field. Either implementation must support SMS autofill (`autocomplete="one-time-code"`).
- Attempt count indicator: "Attempts remaining: [N]" — displayed after the first failed attempt; not shown on initial load.
- Resend link: "Resend code" — active after 60 seconds; countdown shown while inactive ("Resend code in [N] seconds").
- Resends remaining indicator: shown after 1st resend is used; hidden on initial state.
- Primary CTA button: "Verify"
- Optional: "Change phone number" link that returns consumer to phone entry sub-state and clears the current code without counting as a resend.
- No back link from code entry sub-state.

#### States

- **Default:** Code input empty; CTA disabled until 6 digits entered.
- **Loading — verify:** CTA spinner, button disabled.
- **Error — incorrect code:** Inline error below input: "The code you entered is incorrect. [N] attempts remaining." Red border on input; input cleared on error.
- **Error — code expired:** Inline error: "Your code has expired. Please request a new one." Resend link activated.
- **Error — max attempts reached (5 failures on one code):** Inline message: "You've used all your attempts for this code. Please request a new code." CTA disabled; resend link shown.
- **Error — resend cooling down:** Resend link shows countdown: "Resend code in [N] seconds." Not clickable during cooldown.
- **Loading — resend:** Resend link shows "Sending..." text; not clickable.
- **Resend success:** Inline confirmation: "A new code has been sent to [masked phone]." Attempt count resets; cooldown restarts.
- **Lockout (3 resends exhausted or server lockout triggered):** Navigate to Screen 16 — OTP Lockout.
- **Success:** Navigate to `/apply/kba`. Session token rotated server-side before KBA begins; consumer may see a brief loading state.

#### Interactions

- OTP code input: accept numeric input only; auto-advance focus between individual digit boxes if individual-input design is used; `autocomplete="one-time-code"` on the full input or the first digit box.
- "Verify": submits code via `POST /api/apply/otp/verify`.
- "Resend code": triggers `POST /api/apply/otp/resend`; disabled during 60-second cooldown.
- "Change phone number": returns to phone entry sub-state; does not consume a resend.

### Mobile vs desktop

- Phone entry: single-column; phone field full-width on mobile, fixed-width on desktop.
- Code entry: center the 6-digit input block. On mobile, individual digit boxes must be minimum 44×44px tap targets. Numeric keyboard opens automatically on focus via `inputmode="numeric"`.

### Accessibility notes

- Phone entry: `autocomplete="tel"`, `type="tel"`.
- Code entry (6 individual inputs): wrap in a `<fieldset>` with `<legend>` "Verification code". Each input has `aria-label="Digit [N] of 6"`. Auto-advance focus is implemented via JavaScript after a valid digit is entered.
- Code entry (single input): `autocomplete="one-time-code"`, `aria-label="6-digit verification code"`, `inputmode="numeric"`.
- Attempt counter: `role="status"` so it is announced to screen readers without requiring focus.
- Resend countdown: `aria-live="polite"` on the countdown display; announce only on significant state changes (link becoming active, new code sent) — not every second.
- Lockout transition: focus moves to the heading of the lockout screen.
- Page title: "[Brand] Prequalification — Phone Verification"

---

## Screen 8 — Step 6: KBA Identity Verification

**Route:** `/apply/kba`
**Progress bar stage:** Verify & submit (Stage 4, continuing)
**Back-navigation:** LOCKED-FORWARD. Back navigation is blocked. Consumer cannot return to OTP screen from KBA.

### Content and components

- Page heading (H1): "Confirm your identity"
- Subheading: "Answer the following questions to verify your identity. These are based on your personal records."
- Response timer: visible countdown in MM:SS format showing remaining time for the question set. Timer is display-only; the server enforces the time window.
- 3–5 multiple-choice questions. Recommended design: one question per screen (especially on mobile). All-on-one-page is acceptable on desktop.
  - Question text
  - 4–5 answer options as radio buttons
  - "None of the above" is always the final option
- "Next" button (one-at-a-time design) or "Submit answers" button (all-at-once design)
- Question progress indicator (one-at-a-time): "Question [N] of [total]"
- No back link.

### States

- **Default:** First question displayed; no answer selected; CTA disabled.
- **In-progress:** Consumer answering questions; CTA enabled once an answer is selected.
- **Timer warning:** At 30 seconds remaining, a `role="alert"` announces and displays: "30 seconds remaining to complete your identity verification."
- **Loading — submit:** CTA spinner; all inputs disabled.
- **KBA Pass:** Navigation to `/apply/review`. No pass confirmation screen — flow continues directly.
- **KBA Fail:** Navigation to Screen 17 — Variant A (KBA failure). Path A adverse action email dispatched server-side.
- **Timer expired:** Treated as KBA failure; navigate to Screen 17 — Variant A.

### Interactions

- Radio button selection: enables "Next" or contributes to enabling "Submit answers."
- "Submit answers" / "Next": submits answer selections via `POST /api/apply/kba/submit`.
- Timer: client-side countdown display only. Server enforces the time window.

### Mobile vs desktop

- Mobile: one question per screen (strongly recommended).
- Desktop: all questions on one screen is acceptable; one-at-a-time is also acceptable for consistency.

### Accessibility notes

- Each question rendered as a `<fieldset>` with `<legend>` = question text. Each answer is a radio `<input>` + `<label>`.
- Timer: `aria-live="off"` for the ongoing countdown. Threshold warnings (30 seconds) use `role="alert"`.
- On "Next" between questions: focus moves to the heading or `<legend>` of the new question.
- On submission loading: all inputs get `disabled`; announce "Verifying your answers, please wait" via `aria-live="polite"`.
- Page title: "[Brand] Prequalification — Identity Verification"

---

## Screen 9 — Step 7: Review Screen

**Route:** `/apply/review`
**Progress bar stage:** Verify & submit (Stage 4, continuing)
**Back-navigation:** LOCKED-FORWARD for OTP and KBA (cannot go back to them). Edits to vehicle and dealer are made via "Change" links on this screen, not the browser back button.

### Content and components

- Page heading (H1): "Review your information"
- Subheading: "Check that everything looks right before we run your prequalification."

**Your information section (H2):**
- Full name (first + last)
- Address (street, city, state, ZIP)
- Date of birth (formatted as Month DD, YYYY)
- Last 4 of SSN (displayed as "SSN: ••••[last 4]", e.g., "SSN: ••••7890")
- Monthly gross income (displayed as "$[amount]/month", e.g., "$5,000/month")
- No edit/change link for identity fields — locked after initial submission.
- Small note: "To update your personal information, you'll need to start a new application."

**Your vehicle section (H2):**
- Year Make Model Trim (e.g., "2026 Toyota Camry XSE")
- "Change" link (see Interactions)

**Your dealer section (H2):**
- Dealer name
- Dealer address
- "Change" link (see Interactions)

- Primary CTA button: "Continue to disclosure"
- No back link.

### States

- **Default:** All sections populated from server-side application state.
- **Loading — initial page load:** Skeleton cards for each section while application data is fetched.
- **Loading — continue:** CTA spinner; button disabled.
- **Success:** Navigation to `/apply/disclosure`.

### Interactions

- "Change" on vehicle: navigates to `/apply/vehicle` with current vehicle selections pre-populated. After vehicle re-selection, flow routes through `/apply/dealer` (prior dealer pre-populated at top of list) before returning to `/apply/review`.
- "Change" on dealer: navigates to `/apply/dealer` with prior dealer at top of list and pre-selected. After re-selection, returns to `/apply/review`.
- Both change paths are pre-disclosure; no FCRA consent routing action is required (Rule 1 from fcra-consent-routing-spec.md). The consumer does not re-do OTP or KBA.
- Vehicle change routing enforces mandatory dealer re-confirmation — the consumer cannot skip dealer selection after a vehicle change, even if they want the same dealer. This ensures the dealer selection is current before disclosure renders.
- "Continue to disclosure": submits confirmation via `POST /api/apply/review-confirm`; on `200`, navigates to `/apply/disclosure`.

### Mobile vs desktop

- Mobile: each review section is a full-width card, stacked vertically. "Change" links must be inline with their section heading row — not floating right-aligned elements that are difficult to tap.
- Desktop: two-column layout acceptable (personal info on left, vehicle + dealer stacked on right). "Change" links remain inline with section headings.

### Accessibility notes

- Review sections: each section uses an `<h2>` heading.
- "Change" links: `aria-label="Change vehicle selection"` and `aria-label="Change dealer selection"` to distinguish them (two "Change" links on the same page).
- SSN masked display: `aria-label="Social Security Number last 4 digits, masked"`.
- Skeleton loading: `aria-busy="true"` on the container; skeleton elements `aria-hidden="true"`.
- Page title: "[Brand] Prequalification — Review Your Information"

---

## Screen 10 — Step 8: FCRA Disclosure and Consent

**Route:** `/apply/disclosure`
**Progress bar stage:** Verify & submit (Stage 4, final step before decisioning)
**Back-navigation:** LOCKED-FORWARD. Browser back button blocked. Once accepted, any back attempt results in the server returning 409; prior application set to `ABANDONED` if consumer starts over.

**Rendering requirement:** This page must be rendered via dynamic SSR on every request. Static generation, ISR, and all forms of caching are prohibited for this route. Response headers must include `Cache-Control: no-store`. Dealer name must be resolved from `applications.dealer_id` at render time — never from a cached value. This is a compliance requirement, not an optimization preference.

### Content and components

- Page heading (H1): "Prequalification Disclosure and Authorization"
- Disclosure body — rendered dynamically with dealer name injected at SSR time. The disclosure is a versioned legal template managed by Legal SME. Required structural elements in the disclosure body:
  1. Introduction paragraph: what the consumer is authorizing
  2. Named dealer: "[Dealer Name]" — must appear in the disclosure body as the recipient of the lead and the party for whose use the credit report is obtained
  3. Soft pull description: explanation that a soft credit inquiry will be conducted that does not affect the consumer's credit score
  4. KBA / identity pull description (combined conditional disclosure): description that identity verification involved accessing consumer reporting information
  5. Consumer rights statement: right to request a free copy of the credit report from the CRA within 60 days of the pull
  6. CRA name and contact information: name and address of the consumer reporting agency used for the soft pull
- Conditional consent checkbox for rate/finance sharing (see note below)
- Primary CTA button: "Accept and submit" (or "I accept and authorize" — Legal SME to confirm final button label). Not pre-checked or pre-filled. Requires explicit click.
- "Review my application" link: visible only before acceptance; returns consumer to `/apply/review`.
- Do not offer a "Decline" button. The consumer's only active choice is to accept. Leaving the page abandons the application.

**Conditional consent for rate / max finance sharing:**
- If the platform collects separate consent to share rate range and max finance amount with the dealer, implement as an optional checkbox immediately before the "Accept and submit" button.
- Checkbox label: "I consent to [Dealer Name] receiving my estimated APR range and estimated maximum finance amount as part of my prequalification lead."
- Checkbox is optional and independent of base FCRA consent. Unchecked means the lead delivers without rate range and max finance amount.
- The base "Accept and submit" action does not depend on this checkbox.

### States

- **Default (not yet accepted):** Full disclosure rendered with dealer name; "Accept and submit" enabled (subject to scroll-gate pattern on mobile — see Mobile vs desktop); "Review my application" link visible.
- **Loading — accept:** "Accept and submit" shows spinner; button disabled; page locked.
- **Error — consent recording failure:** Inline error banner: "Something went wrong recording your consent. Please try again." CTA re-enabled. The bureau pull must not proceed if consent was not recorded.
- **Success:** Navigation to `/apply/processing`.
- **Locked (already accepted, consumer revisits via direct URL):** Server returns `409`; display: "You've already completed this step. Your application is being processed." Link: "Return to your application status."

### Interactions

- "Accept and submit": `POST /api/apply/disclosure/accept`. On `200`: navigate to `/apply/processing`. On `409`: show locked state. On `5xx`: show error banner, re-enable CTA.
- Conditional consent checkbox: local state; value submitted as part of the `POST /api/apply/disclosure/accept` body.
- "Review my application" link: navigate to `/apply/review` (available only before acceptance).
- Browser back button: suppressed via History API. Post-acceptance back attempt triggers 409 from server; display locked state message.

### Mobile vs desktop

- **Mobile:** Disclosure body is full-width, scrollable within the page. The "Accept and submit" button is sticky at the bottom of the viewport on mobile, remaining visible as the consumer scrolls the disclosure text. Apply a scroll-to-enable pattern: the button transitions from `aria-disabled="true"` (visually disabled, but tabbable) to active once the consumer has scrolled to within a defined threshold of the disclosure bottom. This ensures the consumer has seen the full disclosure. Do not use the `disabled` attribute for this gate — use `aria-disabled` and prevent click handling, so keyboard users can still tab to the button and read its description.
- **Desktop:** Disclosure body in a fixed-height scrollable container or full page scroll. CTA button below the disclosure text; sticky behavior is not required on desktop.

### Accessibility notes

- Disclosure text: `<section>` with `aria-labelledby` pointing to an internal heading ("Authorization and Consent Details").
- Dealer name injected in disclosure: if rendered as a distinct element, wrap in a `<strong>` or `<span>` with the dealer name as visible text — do not inject only as `aria-label`.
- "Accept and submit" button (scroll-gate state): `aria-disabled="true"`, `aria-describedby` pointing to helper text: "Scroll to the end of the disclosure to enable this button."
- Conditional consent checkbox: `<label>` wrapping `<input type="checkbox">` and full label text. Associate with disclosure via `aria-describedby`.
- On page load: focus moves to the page H1 so screen reader users hear the page title and can begin reading.
- Page title: "[Brand] Prequalification — Authorization and Disclosure"

---

## Screen 11 — Decisioning / Processing

**Route:** `/apply/processing`
**Progress bar stage:** Verify & submit (Stage 4, application locked during decisioning)
**Back-navigation:** N/A — application is locked during decisioning.

### Content and components

- Page heading (H1): "We're reviewing your information"
- Subheading: "This usually takes less than a minute. Please don't close this window."
- Indeterminate progress animation (spinner or animated bar — not a percentage complete indicator).
- Supporting copy: "We're checking your prequalification eligibility. You'll see your result shortly."
- No CTAs or navigation elements while processing.

### States

- **Default (processing):** Loading animation active.
- **KBA claim expired during decisioning (Check 1 failure):** If the server returns a session error, display: "Your session has timed out. Please start a new application." with a "Start over" button. Application is `DECLINED`; Path B adverse action email dispatched server-side.
- **Bureau API timeout (> 10 seconds + 1 retry):** Navigate to Screen 17 — Variant B (system error). Application is not decisioned; consumer prompted to try again later.
- **Decisioning complete — PREQUALIFIED:** Navigate to Screen 12.
- **Decisioning complete — DECLINED:** Navigate to Screen 13.

### Interactions

- No interactive elements during processing.
- The page polls or awaits a server-sent event for decisioning completion. On result: navigate to the appropriate result screen.
- If the consumer navigates away during processing and returns in the same session: check application state. If `PREQUALIFIED` or `DECLINED`, navigate to the result screen. If still `DECISIONING`, show the processing screen again.

### Mobile vs desktop

- Centered layout on both breakpoints. Spinner/animation centered horizontally and vertically.

### Accessibility notes

- Loading state: `role="status"` or `aria-live="polite"` on the processing message.
- `aria-busy="true"` on the page container during processing.
- Progress animation: must respect `prefers-reduced-motion`. Provide a non-animated fallback (static icon + text) for consumers with reduced motion preference enabled.
- Page title: "[Brand] Prequalification — Processing Your Application"

---

## Screen 12 — PREQUALIFIED Result Screen

**Route:** `/apply/result/prequalified`
**Progress bar:** N/A — result screen.
**Back-navigation:** N/A — terminal state.

### Content and components

- Brand logo in header (no progress bar).
- Success visual: confirmation check icon or brand-appropriate success mark (not relying on color alone).
- Page heading (H1): "You're prequalified!"
- Subheading: "Here's your prequalification estimate for a new [Toyota / Lexus] at [Dealer Name]."

**Confirmation number block:**
- Label: "Confirmation Number"
- Value: `PQ-YYYYMMDD-[8 chars]` — prominent, large font. Optional copy-to-clipboard button.
- Helper text: "Save this number. You'll need it when you visit the dealer."

**Selected dealer block:**
- Label: "Your Dealer"
- Value: dealer name and address.

**Expiration block:**
- Label: "Prequal valid through"
- Value: `Month DD, YYYY`
- Required qualifying text (proximate — not footer-only): "Your prequalification estimate is valid through [date]. Visiting the dealer before this date does not guarantee loan approval or the estimated rate range — final terms are subject to a full credit review at the time of financing."

**Rate range block (Version A — shown only when `consent_to_share_rate_range = true`):**
- Label: "Estimated APR Range"
- Value: `X.XX% – X.XX% APR` — always two decimal places; "APR" label on same line.
- Required qualifying text (proximate — not footer-only): "This APR range is an estimate based on your credit profile at the time of prequalification. It is not a guaranteed rate. Your actual APR will be determined by the lender at the time of financing and may differ based on your final credit application, vehicle selected, loan term, and other factors."
- This block is omitted entirely (no label, no placeholder) if consent was not given.

**Max finance amount block (Version A — shown only when `consent_to_share_max_finance = true`):**
- Label: "Estimated Maximum Finance Amount"
- Value: "Up to $[amount]" — dollar-formatted; always preceded with "Up to".
- Required qualifying text (proximate — not footer-only): "This amount is an estimate based on your credit profile and is not a loan approval. Your final financed amount will depend on the vehicle you select, its purchase price, your down payment, applicable taxes and fees, and lender approval at the time of financing."
- This block is omitted entirely if consent was not given.

**Version B baseline qualifying statement (shown when either rate or max finance block is absent):**
"This prequalification result is not a final credit approval. Financing is subject to dealer and lender review at the time of your visit."

**Next steps section (H2: "What to do next"):**
1. "Visit [Dealer Name] before [expiration date] and bring your confirmation number."
2. "The dealer's finance team will help you select a vehicle and complete a full credit application. At that time, a hard credit inquiry will be conducted."
3. "Your final loan rate, term, and monthly payment will be determined at that point — not now."
- Required proximate qualifying statement: "Your prequalification estimate is not a loan approval and does not lock in any rate or financing terms."

**Confirmation email notice:**
- Small text: "A confirmation email with these details has been sent to [masked email, e.g., j***@email.com]."

**Prohibited language on this screen:**
- "You're approved" — not permitted
- "You qualify for" — not permitted
- "Pre-approved" — not permitted
- Any language implying a credit commitment has been made
- Monthly payment estimate
- Number of payments or loan term
- Finance charge in dollars
- Down payment amount
- Single APR (not a range)
- ECOA anti-discrimination tagline (not shown on prequalified screen — only in adverse action notices)

### States

- **Default:** Result rendered from server-side application state.
- **Copy confirmation number (if button present):** Button label changes to "Copied!" with checkmark for 2 seconds.

### Interactions

- Copy-to-clipboard button (if present): copies confirmation number string to clipboard.
- No "Apply again" or "Continue" action. Any return navigation goes to the landing page.

### Mobile vs desktop

- Mobile: single-column; sections stacked vertically. Confirmation number at top in a prominent block. Rate range and max finance amount (if shown) displayed prominently with qualifying text immediately below each value — not in a footer.
- Desktop: two-column layout acceptable (confirmation number + next steps left; dealer + rate + finance amount right). Qualifying text must remain proximate to its associated value — not collapsed into a footer section.

### Accessibility notes

- Copy-to-clipboard button: `aria-label="Copy confirmation number [full confirmation number] to clipboard"`. On copy: announce via `aria-live="polite"` "Confirmation number copied."
- Qualifying text for rate range and max finance amount: these are `<p>` elements with normal readable text size — not `<small>` or footer text. Do not visually or semantically demote them.
- Rate range value: `aria-label="Estimated APR range: [X.XX] to [X.XX] percent APR"`.
- Max finance value: `aria-label="Estimated maximum finance amount: up to [formatted dollar amount]"`.
- Success icon: `aria-hidden="true"` if purely decorative; include a screen-reader-only `<span>` with "Application prequalified" if the icon is the primary success indicator.
- Page title: "[Brand] Prequalification — You're Prequalified"

---

## Screen 13 — DECLINED Result Screen

**Route:** `/apply/result/declined`
**Progress bar:** N/A
**Back-navigation:** N/A

### Content and components

- Page heading (H1): "We're unable to complete your prequalification"
- Body copy: "We weren't able to prequalify you at this time. You'll receive an email with more information."
- Notice: "An adverse action notice with information about your rights has been sent to your email address on file. You may request specific reasons for this decision within 60 days." (This sentence references the consumer's right under ECOA Reg B §1002.9(c) and must appear on this screen.)
- Support guidance: "If you have questions, please contact us at [support contact — placeholder at MVP]."
- "Return to home" button: navigates to `/`.

**Prohibited on this screen:**
- Specific decline reason codes (e.g., credit score, derogatory indicators, income level)
- Financial details of any kind
- Any language suggesting the consumer "almost qualified" or providing score-improvement guidance
- Adverse action reason codes — these appear only in the adverse action notice email

### States

- **Default:** Static message. No dynamic data required beyond confirming `DECLINED` status.

### Interactions

- "Return to home" navigates to `/`.

### Mobile vs desktop

- Centered, single-column layout on both breakpoints.

### Accessibility notes

- Neutral visual treatment. Avoid iconography suggesting a hard stop (no large red X). A calm, neutral design is appropriate.
- Body copy color contrast: 4.5:1 minimum.
- Page title: "[Brand] Prequalification — Application Status"

---

## Screen 14 — Session Timeout Warning Modal

**Route:** Overlay on the currently active application screen (not a full-page navigation)
**Progress bar:** Visible behind the modal overlay
**Back-navigation:** N/A (modal; background navigation suppressed while open)

### Content and components

This is a modal overlay that appears on any active application screen when the session is approaching expiration. It does not navigate away from the current screen.

**Trigger conditions:**
- Idle timeout warning: displayed with 5 minutes remaining before the 60-minute idle timeout (fires at 55 minutes idle).
- Absolute timeout warning: displayed with 5 minutes remaining before the 4-hour absolute timeout (fires at 3 hours 55 minutes).

**Modal content:**
- Modal heading: "Your session is about to expire"
- Body copy: "For your security, your session will expire in [MM:SS countdown]. Do you want to stay signed in?"
- Countdown timer in the body copy: live countdown updated every second. Format: "Your session will expire in 4:59."
- Primary button: "Stay signed in"
- Secondary button / link: "Sign out"

### States

- **Default (warning modal visible):** Countdown active; background page dimmed; focus trapped in modal.
- **"Stay signed in" clicked:** Modal closes; idle timeout reset; absolute timeout cannot be extended; page interaction resumes; focus returns to prior element.
- **"Sign out" clicked:** Session terminated; navigate to landing page.
- **Countdown reaches 00:00 before action:** Navigate to Screen 15 — Session Expired.
- **Consumer does not interact:** Countdown fires; navigate to Screen 15.

### Interactions

- "Stay signed in": `POST /api/auth/extend-session`; on `200`, dismiss modal and reset idle timer; on error, navigate to Screen 15.
- "Sign out": `POST /api/auth/logout`; navigate to `/`.
- Focus trap: keyboard Tab and Shift+Tab cycle only between the modal's interactive elements while the modal is open.
- Background page: `aria-hidden="true"` and `inert` while modal is open.

### Mobile vs desktop

- Mobile: bottom sheet (slides up from bottom of viewport) or centered overlay.
- Desktop: centered overlay modal.
- Modal width: minimum 320px, does not exceed 480px on either breakpoint.

### Accessibility notes

- `role="dialog"`, `aria-modal="true"`, `aria-labelledby="[modal-heading-id]"`.
- On modal open: focus moves to the modal heading or the primary button.
- On modal close ("Stay signed in"): focus returns to the element that was focused when the modal opened.
- Countdown: `aria-live="off"` for continuous count. At 1 minute remaining: `role="alert"` announces "One minute remaining before your session expires."
- Countdown display: do not rely on color alone to convey urgency — supplement with text and icon.

---

## Screen 15 — Session Expired

**Route:** `/session-expired`
**Progress bar:** N/A
**Back-navigation:** N/A

### Content and components

- Page heading (H1): "Your session has expired"
- Body copy: "For your security, your session ended after a period of inactivity. Any unsaved progress has been ended."
- Additional context: "If you were in the middle of your application, you'll need to sign in again. If you had already completed identity verification, you'll need to start a new application — your identity verification cannot be carried forward after a session expires."
- Primary CTA button: "Sign in again" — navigates to `/api/auth/login`
- Secondary link: "Return to home" — navigates to `/`

**On return after sign-in:**
If the consumer has an existing `DRAFT` application, the platform may offer to resume or prompt to start over (product decision). If the application was in `IDENTITY_VERIFIED` state, it cannot be resumed — a new application must be started (KBA claim has elapsed).

### States

- **Default:** Static screen.

### Interactions

- "Sign in again": navigates to `/api/auth/login`.
- "Return to home": navigates to `/`.

### Mobile vs desktop

- Centered, single-column layout on both breakpoints.

### Accessibility notes

- Page title: "[Brand] Prequalification — Session Expired"
- Body copy color contrast: 4.5:1 minimum.

---

## Screen 16 — OTP Lockout

**Route:** `/apply/otp/lockout`
**Progress bar stage:** Verify & submit (Stage 4, locked)
**Back-navigation:** N/A

### Content and components

- Page heading (H1): "Too many attempts"
- Body copy: "You've exceeded the number of allowed verification attempts. For your security, phone verification is temporarily locked."
- Lockout duration: "Please try again in 30 minutes."
- Countdown timer (recommended, not required for MVP): "You can try again in [MM:SS]." — counts down from 30 minutes.
- Application state note: "Your application information has been saved. Sign out and try again after the lockout period."
- Lockout scope note (optional, for consumer clarity): "Only phone verification is locked — your account is not affected."
- Primary CTA: "Sign out"
- Secondary link: "Return to home"

**Lockout scope:** The lockout is scoped to the phone number + application session. The consumer's Auth0 account is not locked. The same phone number in a new session within the lockout window is server-side blocked.

### States

- **Default:** Lockout active; countdown running (if implemented).
- **Lockout expired (countdown reaches 00:00):** Message updates to "You can now try again. Sign in to continue your application." CTA changes to "Continue application" (navigates to Auth0 login, then back to `/apply/otp`).

### Interactions

- "Sign out": `POST /api/auth/logout`; navigate to `/`.
- "Return to home": navigate to `/`.
- "Continue application" (post-lockout): navigates to `/api/auth/login`.

### Mobile vs desktop

- Centered, single-column on both breakpoints.

### Accessibility notes

- Countdown (if present): `aria-live="off"` for continuous count; announce at 5 minutes and 1 minute remaining via `role="alert"`.
- Page title: "[Brand] Prequalification — Verification Locked"

---

## Screen 17 — Error / Unable to Complete

**Route:** `/apply/error`
**Progress bar:** N/A
**Back-navigation:** N/A

This screen covers system errors distinct from a decisioning decline. A decisioning decline (checks 2–6 fail) routes to Screen 13. This screen covers KBA failure (Variant A) and technical system errors (Variant B).

---

### Variant A — KBA Failure (Path A adverse action)

- Page heading (H1): "We're unable to complete your prequalification"
- Body copy: "We were unable to verify your identity at this time."
- Notice: "An adverse action notice with information about your rights has been sent to your email address on file. You may request specific reasons for this decision within 60 days."
- Primary CTA: "Return to home"

**Do not show:** KBA failure reason, question details, or any information about how identity verification works. The reason codes and notice content appear only in the adverse action email (Path A), not on screen.

---

### Variant B — System Error (technical failure)

- Page heading (H1): "Something went wrong"
- Body copy: "We encountered a technical issue and couldn't complete your application. Your information has been saved."
- Guidance: "Please try again later. If the problem continues, contact us at [support contact — placeholder at MVP]."
- Primary CTA: "Try again" — attempts to re-enter the flow at the last successful step or returns to landing page on failure.
- Secondary link: "Return to home"

**Triggers for Variant B:**
- Bureau soft pull API timeout (> 10 seconds + 1 retry) during decisioning
- Unexpected server error during decisioning chain
- Any unrecoverable error in Steps 1–8 that cannot be handled inline on the step screen

---

### Interactions

- "Return to home": navigate to `/`.
- "Try again" (Variant B): navigates to the last viable resume point; on failure, navigates to `/`.

### Mobile vs desktop

- Centered, single-column on both breakpoints.

### Accessibility notes

- KBA failure variant (A): neutral visual treatment; do not use alarming iconography.
- System error variant (B): neutral icon is appropriate (not a bright red X).
- Body copy color contrast: 4.5:1 minimum.
- Page title (KBA variant): "[Brand] Prequalification — Application Status"
- Page title (system error variant): "[Brand] Prequalification — Unable to Complete"

---

## Screen cross-reference index

| Screen | Route | Trigger |
|--------|-------|---------|
| 1. Landing | `/` | Direct navigation; entry point URL |
| 2. Auth0 Login | Auth0-hosted | "Get started" or "Sign in" from landing |
| 3. Personal info | `/apply/personal-info` | Post-Auth0 callback |
| 4. Vehicle selection | `/apply/vehicle` | Step 1 complete; or "Change vehicle" from Review |
| 5. Dealer selection | `/apply/dealer` | Step 2 complete; or "Change dealer" from Review; or vehicle change return path |
| 6. Income | `/apply/income` | Step 3 (dealer) complete |
| 7. OTP verification | `/apply/otp` | Step 4 (income) complete |
| 8. KBA verification | `/apply/kba` | OTP success |
| 9. Review | `/apply/review` | KBA pass |
| 10. FCRA disclosure | `/apply/disclosure` | "Continue to disclosure" from Review |
| 11. Processing | `/apply/processing` | FCRA disclosure acceptance |
| 12. Prequalified result | `/apply/result/prequalified` | Decisioning: all 6 checks pass |
| 13. Declined result | `/apply/result/declined` | Decisioning: any check 2–6 fails |
| 14. Session timeout warning | Overlay on current screen | 55-min idle or 3h 55m absolute |
| 15. Session expired | `/session-expired` | Timeout fires; modal countdown reaches 00:00 |
| 16. OTP lockout | `/apply/otp/lockout` | 5 failed attempts on one code or 3 resends exhausted |
| 17. Error — KBA (Variant A) | `/apply/error` | KBA failure |
| 17. Error — System (Variant B) | `/apply/error` | Bureau timeout; unrecoverable system error |

---

## WCAG 2.1 AA — global checklist

The following requirements apply to all screens. Screen-specific notes appear within each screen's accessibility section.

| Criterion | Requirement |
|-----------|-------------|
| 1.1.1 Non-text content | All images have descriptive `alt` text; decorative images use `alt=""` |
| 1.3.1 Info and relationships | Form relationships conveyed via `<label>`, `<fieldset>`, `<legend>` — not visual proximity alone |
| 1.3.3 Sensory characteristics | Instructions do not rely on color, shape, or position alone |
| 1.4.1 Use of color | Color is not the sole means of conveying information (errors use icon + text + color) |
| 1.4.3 Contrast (minimum) | Normal text: 4.5:1. Large text and UI components: 3:1 |
| 1.4.4 Resize text | Content remains readable and functional at 200% zoom |
| 1.4.10 Reflow | No horizontal scrolling required at 320px viewport width |
| 1.4.11 Non-text contrast | UI components (inputs, buttons, progress bar segments) meet 3:1 contrast against adjacent colors |
| 1.4.12 Text spacing | No content or functionality lost with: line height 1.5×, letter spacing 0.12em, word spacing 0.16em |
| 2.1.1 Keyboard | All functionality operable via keyboard; no keyboard traps except intentional modal traps with documented exit |
| 2.1.2 No keyboard trap | Modal focus traps have a defined exit mechanism (Escape key or button) |
| 2.4.1 Bypass blocks | Skip navigation link present at top of page |
| 2.4.2 Page titled | Every page has a meaningful, descriptive `<title>` |
| 2.4.3 Focus order | Focus order is logical and matches visual reading order |
| 2.4.4 Link purpose | All links have descriptive text or `aria-label` |
| 2.4.6 Headings and labels | Headings describe page sections; form labels describe inputs |
| 2.4.7 Focus visible | Focus ring is visible on all interactive elements; brand CSS must not suppress `:focus-visible` |
| 3.1.1 Language of page | `lang="en"` on `<html>` element |
| 3.2.1 On focus | No unexpected context changes on focus |
| 3.2.2 On input | No unexpected context changes on input (cascade dropdown fetches trigger data load but do not navigate) |
| 3.3.1 Error identification | Errors identified in text and associated with the relevant field via `aria-describedby` |
| 3.3.2 Labels or instructions | Instructions provided before input fields where format is required (DOB, phone) |
| 4.1.2 Name, role, value | All UI components have accessible name, role, and value exposed to assistive technology |
| 4.1.3 Status messages | Status messages (success, loading, error) conveyed via `role="status"`, `role="alert"`, or `aria-live` without requiring focus |

---

*← [Auto Leads Platform — Progress](../progress.md)*
