# Lean Business Case: Auto Leads Platform

> **Project:** auto-leads-platform
> **Status:** Complete
> **Skill:** `/leanbizcase`
> **Last updated:** 2026-04-10

---

## 1. Executive Summary

Toyota Financial Services has an opportunity to meaningfully increase new vehicle sales for Toyota and Lexus dealers by building a proprietary digital prequalification and lead generation platform. Consumers complete a lightweight form, receive a real-time soft-pull prequalification decision, and are matched to a local dealer — who receives a structured lead in ADF format. At steady state, the platform is projected to generate approximately $8.6M in annual incremental loan margin at a sustained marketing investment of $1M/year, with build costs recovering within the first year of operation. This document requests green light approval to proceed, formalize the build plan, and assemble the team.

---

## 2. Problem Statement

**Who feels it:** Toyota and Lexus franchise dealers across the US, and the captive lender's business development and dealer relations teams.

**The pain:** Dealers lack a scalable, captive-lender-sourced pipeline of prequalified buyers — particularly for customers with lower FICO scores who are underserved by traditional acquisition channels. Without a proprietary lead source, dealers rely on third-party lead aggregators and general marketing channels that are expensive, generic, and do not leverage Toyota Financial's unique customer data advantage.

**Current cost:** Lost loan capture, lost vehicle sales, and dealer dependency on third-party lead providers whose data and lead quality Toyota Financial cannot control or optimize.

**If nothing changes:** Competitors and third-party platforms continue to intermediate the customer relationship, eroding Toyota Financial's influence over the purchase funnel and limiting their ability to deploy strategic financing tools (rate incentives, trim-level offers, loyalty rewards) at the point of customer intent.

---

## 3. Proposed Solution & MVP Scope

**What we're building:** A consumer-facing digital prequalification platform that collects basic contact information, vehicle of interest, income, and last 4 of SSN, executes a soft credit pull, renders a prequalification decision with a max finance amount and rate range, and delivers a structured lead in ADF format to the matched dealer — while the consumer receives a branded email summary and can register to track their application history.

**MVP includes:**
- Consumer prequalification form (contact info, vehicle of interest, income, last 4 SSN)
- Soft bureau pull with real-time decisioning (approved / declined / referred)
- Max finance amount and rate range presented to consumer
- ADF-format lead delivery to matched dealer
- Consumer email notification with application summary
- Guest and registered user flows (with history claim post-registration)
- Unique confirmation number per application
- New Toyota and Lexus vehicles only

**MVP explicitly excludes:**
- Hard credit pull or full loan origination
- Dealer CRM deep integration (ADF delivery only at launch)
- Fraud / AML scoring layer
- Reporting and analytics service (direct DB query at MVP)
- SMS or push notifications
- BFF layer / separate API gateway per channel
- Non-Toyota / non-Lexus vehicles
- Used vehicle prequal

**Phased roadmap:**

| Phase | Title | Key Capability |
|---|---|---|
| Phase 1 | MVP Launch | Core prequal flow, soft pull, ADF lead delivery, email notification |
| Phase 2 | Dealer Depth | CRM integrations, dealer portal, lead response tracking |
| Phase 3 | Data & Optimization | Reporting service, conversion analytics, decisioning model tuning |
| Phase 4 | Product Expansion | Used vehicles, loyalty rate offers, trim-level incentive targeting |

---

## 4. Target Audience & Stakeholders

**Consumer (Front-End User):** Toyota and Lexus vehicle shoppers in the US — particularly buyers with lower FICO scores seeking financing clarity before visiting a dealership. Job-to-be-done: understand whether they can finance a vehicle and for how much, without committing to a hard pull or a dealer visit.

**Primary Beneficiary:** Toyota and Lexus franchise dealers — receive qualified, prequalified leads in their expected format with consumer vehicle intent and financing pre-assessed.

**Internal Stakeholders:**

| Role | Team | Relationship to Project |
|---|---|---|
| Sponsor | Digital / Product | Funds and champions the initiative |
| Approver | Executive Leadership | Required green light to proceed |
| Dealer Network | Dealer Relations | Lead recipients; onboarding and adoption dependency |
| Sales | Sales / BD | Business case validation; dealer rollout support |
| Legal / Compliance / Privacy | Legal | FCRA, GLBA, privacy policy sign-off; mandatory gate |
| Marketing | Marketing | Consumer acquisition spend; campaign support |

---

## 5. Strategic Alignment

**Company Strategic Pillars this supports:**
1. Drive vehicle sales — direct pipeline of prequalified buyers to franchise dealers
2. Build scalable digital capabilities — proprietary platform vs. third-party dependency
3. Support dealer partners — tangible customer acquisition tool for the dealer network

**Why build, why now:** The internal strategic driver is Toyota Financial's need to provide dealers with measurable, captive-lender-sourced customer acquisition support. The timing aligns with increasing dealer demand for digital lead sources and Toyota Financial's broader push to own more of the customer journey digitally.

**Urgency driver:** Internal mandate to deepen dealer support and build scalable digital capabilities. No external regulatory pressure, but FCRA and GLBA compliance requirements must be baked in from the start — retrofitting is materially more expensive.

---

## 6. Build vs. Buy Analysis

| Option | Description | Cost | Time-to-Market | Control | Strategic Fit |
|---|---|---|---|---|---|
| **Build** | Proprietary AWS microservices platform | $1.1M–$1.3M build + $1M/yr marketing | 4 months to MVP | Full | High |
| **Buy** | Third-party prequal / lead gen SaaS | Lower upfront, high recurring per-transaction fees at scale | 1–3 months | Low | Low |
| **Partner** | White-label or API-based prequal wrapper | Mid upfront, ongoing licensing | 2–3 months | Medium | Medium |

**Recommendation: Build.** Toyota Financial's competitive advantage is its proprietary view of customer behavior across the full vehicle lifecycle. No SaaS vendor models that data the way Toyota Financial needs it, and at their loan volume, SaaS per-transaction economics flip unfavorably at scale. Regulatory and compliance control, deep dealer network specificity, and the strategic importance of owning the customer relationship all reinforce the build decision. The captive lender's ability to deploy rate incentives tied to specific inventory, trim levels, and customer loyalty is impossible to replicate on a generic platform.

---

## 7. Financial Model

**Estimated Build Cost:** $1.12M – $1.26M (4-month MVP build)

| Category | Low | High |
|---|---|---|
| Team (engineers, PM, architect, design) | $984,000 | $1,032,000 |
| AWS infrastructure (build period) | $8,400 | $14,800 |
| Third-party services (bureau setup, auth, email) | $6,000 | $18,000 |
| Other (tooling, pen test, legal, security audit) | $21,000 | $47,000 |
| Contingency (10–15%) | $100,000 | $150,000 |
| **Total** | **$1,119,400** | **$1,261,800** |

**Ongoing costs:** ~$3,700/month infrastructure + $1,000,000/year marketing

**Revenue model assumptions:**
- 20,000 prequalifications/month at steady state (end of Year 1)
- 36% 90-day purchase rate → 7,200 purchases/month
- 20% incremental volume → 1,440 incremental sales/month
- 50% captive capture rate → 720 funded loans/month
- $1,000 lifetime loan margin → $720,000/month incremental revenue at steady state
- Year 1 ramps linearly to steady state (dealer opt-in dependency)

| Year | Build / Marketing Cost | Projected Incremental Revenue | Net |
|---|---|---|---|
| Year 1 | ~$2.2M (build + marketing + infra) | ~$4.3M (ramp avg ~$360K/mo) | **~$2.1M** |
| Year 2 | ~$1.05M (marketing + infra) | ~$8.6M (steady state) | **~$7.55M** |
| Year 3 | ~$1.05M (marketing + infra) | ~$8.6M (steady state) | **~$7.55M** |

**Break-even on build cost:** Approximately Month 13–14 from project start (within the first 2 months of steady-state operation).

---

## 8. Risk Register

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | **Microservices complexity at MVP** — 7 services means 7 pipelines, 7 DBs, 7 monitoring surfaces; operational overhead can dominate early sprints | Medium | High | Standardize service scaffolding and CI/CD templates before sprint 1; assign a dedicated DevOps / platform engineer; consider starting with 3–4 core services and extracting the rest post-MVP |
| 2 | **FCRA compliance gaps** — permissible purpose, adverse action notices, and soft pull disclosure requirements must be correct at launch | Medium | Critical | Legal SME in Discovery (not just Validate); FCRA requirements baked into UX spec before design is finalized; legal gate mandatory before any consumer-facing copy is approved |
| 3 | **GLBA and data privacy** — financial services data handling, privacy notices, and SSN handling fall under GLBA; non-compliance risk is significant | Low | Critical | Legal and privacy review before launch; SSN never logged; PII encrypted at column level; privacy policy and terms live before first consumer interaction |
| 4 | **Dealer adoption** — a technically correct lead in the wrong format or ignored by the dealer CRM is worthless; ADF compliance and dealer onboarding are underestimated | High | High | Engage 3–5 dealer partners during build to validate lead format and delivery; establish dealer success SLAs; measure lead response rate from week 1 |
| 5 | **Timeline** — 4 months for 7 microservices, bureau integration, consumer portal, and compliance readiness is aggressive; one integration delay or compliance question cascades | High | Medium | Build compliance requirements into sprint 1 (not sprint 4); identify bureau integration as the longest lead-time dependency and start vendor conversations immediately; maintain 10–15% schedule contingency |

---

## 9. Success Criteria & KPIs

**Launch Gate (ship / no-ship criteria):**

*Functional*
- [ ] Consumer can complete prequal form end-to-end
- [ ] Soft bureau pull executes successfully in production
- [ ] Decisioning returns approved / declined / referred
- [ ] Customer receives email summary within 5 minutes
- [ ] Lead delivered to dealer in correct ADF format
- [ ] Guest applications can be claimed post-registration
- [ ] Confirmation number generated for every application

*Compliance*
- [ ] FCRA permissible purpose language on form
- [ ] Soft pull disclosure presented and accepted before pull executes
- [ ] Adverse action notice generated for all declines
- [ ] Privacy policy and terms of service live
- [ ] SSN never appears in any application log
- [ ] PII encrypted at rest and in transit
- [ ] Legal sign-off on all consumer-facing copy

*Technical*
- [ ] All 7 services deployed independently
- [ ] End-to-end prequalification completes in under 5 seconds
- [ ] 99.9% uptime baseline established
- [ ] Monitoring and alerting live on all services
- [ ] Pen test completed with no critical findings
- [ ] Disaster recovery plan documented and tested

*Operational*
- [ ] Minimum 3 dealer partners onboarded and receiving leads
- [ ] Support runbook documented for common failure scenarios
- [ ] Bureau production credentials active and verified

**Success Metrics:**

| Metric | Baseline | 90-Day Target | 6-Month Target |
|---|---|---|---|
| Monthly prequalification volume | 0 | 500+ | 1,000+ |
| Form completion rate | — | >60% | >70% |
| Bureau pull success rate | — | >98% | >98% |
| Applications resulting in an offer | — | >40% | >40% |
| Lead delivery success rate | — | >99% | >99% |
| Dealer lead response rate | — | >50% | >50% |
| Prequal → dealership visit rate | — | >15% | >15% |
| **Prequal → funded deal conversion** | — | Baseline | >8% |
| Dealer partners active | 3 | 5+ | 10+ |
| Service uptime | — | >99.9% | >99.9% |
| P95 API response time | — | <3 seconds | <3 seconds |
| Net Promoter Score | — | — | >40 |

---

## 10. Decision Ask & Next Steps

**The Ask:** Green light to proceed with the Auto Leads Platform build.

**If approved, next steps:**

1. **Formalize the build plan** — finalize service architecture, sprint structure, and compliance requirements with Legal SME; identify bureau integration vendor and begin onboarding (longest lead-time dependency) — *Week 1–2*
2. **Assemble the team** — confirm headcount, engage contractors or agency partners as needed, assign product and engineering leads — *Week 2–3*
3. **Start development** — establish service scaffolding, CI/CD pipelines, and shared standards before sprint 1 business logic begins; engage 3 pilot dealer partners for lead format validation — *Week 3–4*

---

*← [Auto Leads Platform — Progress](../progress.md)*
