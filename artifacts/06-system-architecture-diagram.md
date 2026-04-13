# System Architecture Diagram

> **Project:** Auto Leads Platform
> **Status:** Complete
> **Last updated:** 2026-04-12

---

## 1. High-Level System Overview

The platform's components, external services, and data stores with their primary interaction paths.

```mermaid
flowchart TB
    subgraph Consumer["Consumer Layer"]
        BROWSER["Browser<br/>(Mobile + Desktop)"]
    end

    subgraph Frontend["Frontend — Next.js (SSR)"]
        NEXTJS["Next.js App<br/>SSR for regulated surfaces"]
    end

    subgraph Auth["Identity — Auth0"]
        UL["Universal Login<br/>(Hosted)"]
        ACTIONS["Auth0 Actions<br/>(Custom JWT Claims)"]
        MGMT["Management API<br/>(app_metadata)"]
    end

    subgraph Backend["Backend — Node.js / TypeScript"]
        API["API Service<br/>(Field-Scoped Endpoints)"]
        DECISIONING["Decisioning Engine"]
        LEAD_DIST["Lead Distribution<br/>Service"]
        EMAIL_SVC["Email Service<br/>(SES)"]
        ENCRYPT["Encryption Module<br/>(AES-256-GCM)"]
        AUDIT["Audit Logger<br/>(23 Named Events)"]
        IDOR["Ownership Middleware<br/>(IDOR Protection)"]
    end

    subgraph Data["Data Layer"]
        RDS[("PostgreSQL<br/>AWS RDS<br/>(Multi-AZ)")]
        REDIS[("ElastiCache Redis<br/>(Session Verification<br/>Cache)")]
    end

    subgraph Secrets["Secrets & Keys"]
        SM["AWS Secrets Manager"]
        KMS["AWS KMS<br/>(Customer-Managed Key)"]
    end

    subgraph External["External Services"]
        SNS["Amazon SNS<br/>(OTP SMS)"]
        NHTSA["NHTSA API<br/>(Vehicle Data)"]
        SES["Amazon SES<br/>(Email Delivery)"]
    end

    subgraph Stubbed["Stubbed for MVP"]
        BUREAU["Bureau Soft Pull<br/>(Experian / Equifax)"]
        KBA_PROV["KBA Provider"]
        DT["DealerTrack<br/>(ADF)"]
        RO["RouteOne<br/>(ADF)"]
    end

    BROWSER --> NEXTJS
    BROWSER --> UL
    UL --> ACTIONS
    NEXTJS --> API
    API --> IDOR --> ENCRYPT --> RDS
    API --> REDIS
    API --> MGMT
    API --> DECISIONING
    DECISIONING --> RDS
    API --> LEAD_DIST
    LEAD_DIST -.-> DT
    LEAD_DIST -.-> RO
    API --> EMAIL_SVC
    EMAIL_SVC --> SES
    API --> AUDIT --> RDS
    API --> SNS
    NEXTJS --> NHTSA
    ENCRYPT --> SM
    SM --> KMS
    API -.-> BUREAU
    API -.-> KBA_PROV

    style Stubbed fill:#f9f0e0,stroke:#d4a843
    style Consumer fill:#e8f4fd,stroke:#2196F3
    style Auth fill:#f3e5f5,stroke:#9C27B0
    style Secrets fill:#fce4ec,stroke:#E91E63
```

---

## 2. VPC Network Topology

Security group boundaries, subnet placement, and network isolation for all infrastructure components.

```mermaid
flowchart TB
    subgraph Internet["Public Internet"]
        CONSUMER["Consumer Browser"]
        AUTH0_EXT["Auth0 Tenant"]
    end

    subgraph VPC["AWS VPC"]
        subgraph PubSub["Public Subnet"]
            ALB["Application Load Balancer<br/>(Internet-Facing)"]
        end

        subgraph PrivSub["Private Subnet"]
            ECS["ECS Tasks<br/>(Next.js + API Service)"]
        end

        subgraph DataSub["Private Subnet — Data"]
            RDS_VPC[("PostgreSQL RDS<br/>Multi-AZ<br/>Port 5432")]
            REDIS_VPC[("ElastiCache Redis<br/>Port 6379/6380 TLS<br/>No IGW Route")]
        end

        subgraph IntSub["Private Subnet — Internal"]
            INT_ALB["Internal ALB<br/>(Non-Internet-Facing)<br/>Production KBA Only"]
        end
    end

    subgraph AWS_SVC["AWS Services (VPC Endpoints)"]
        SM_VPC["Secrets Manager"]
        SNS_VPC["SNS"]
        SES_VPC["SES"]
        KMS_VPC["KMS"]
    end

    CONSUMER -->|"HTTPS 443"| ALB
    CONSUMER -->|"OAuth Flow"| AUTH0_EXT
    ALB -->|"443"| ECS
    ECS -->|"5432"| RDS_VPC
    ECS -->|"6380 TLS"| REDIS_VPC
    ECS -->|"IAM Role"| SM_VPC
    ECS -->|"IAM Role"| SNS_VPC
    ECS -->|"IAM Role"| SES_VPC
    SM_VPC -->|"Decrypt"| KMS_VPC
    ECS -->|"M2M Token"| AUTH0_EXT
    INT_ALB -.->|"443 — KBA Callback<br/>(Production Only)"| ECS

    style PubSub fill:#e8f5e9,stroke:#4CAF50
    style PrivSub fill:#e3f2fd,stroke:#2196F3
    style DataSub fill:#fff3e0,stroke:#FF9800
    style IntSub fill:#fce4ec,stroke:#E91E63
    style Internet fill:#f5f5f5,stroke:#9E9E9E
```

**Security group rules summary:**

| Source | Target | Port | Purpose |
|--------|--------|------|---------|
| `0.0.0.0/0` | ALB | 443 | Consumer HTTPS |
| ALB SG | ECS SG | 443 | App traffic |
| ECS SG | RDS SG | 5432 | Database |
| ECS SG | Redis SG | 6380 | Session cache (TLS) |
| KBA Provider CIDR | Internal ALB | 443 | KBA callback (prod only) |
| Internal ALB SG | ECS SG | 443 | KBA callback forwarding |

Redis and RDS have **no inbound from `0.0.0.0/0`** on any port. Redis has **no route to IGW**.

---

## 3. Consumer Request Flow

The end-to-end prequalification flow from account creation through lead delivery.

```mermaid
sequenceDiagram
    participant C as Consumer
    participant NJ as Next.js (SSR)
    participant A0 as Auth0
    participant API as Backend API
    participant R as Redis
    participant DB as PostgreSQL
    participant SNS as Amazon SNS
    participant SE as Amazon SES
    participant SM as Secrets Manager

    Note over C,SM: Phase 1 — Authentication
    C->>A0: Universal Login (email + password)
    A0-->>C: JWT (access + ID tokens)
    C->>NJ: Authenticated session

    Note over C,SM: Phase 2 — Application Creation
    C->>NJ: Start prequalification
    NJ->>API: POST /applications
    API->>SM: Retrieve encryption key (startup)
    API->>DB: Insert application (PII encrypted AES-256-GCM)
    DB-->>API: Application created (DRAFT)
    API-->>NJ: 201 + application ID

    Note over C,SM: Phase 3 — OTP Verification
    C->>NJ: Enter phone number
    NJ->>API: POST /applications/:id/otp/send
    API->>SNS: Send 6-digit OTP (Transactional SMS)
    SNS-->>C: SMS delivered
    C->>NJ: Enter OTP code
    NJ->>API: POST /applications/:id/otp/verify
    API->>A0: Set otp_verified claims (M2M)
    API->>R: Write verification state (TTL 3600s)
    API-->>NJ: OTP verified

    Note over C,SM: Phase 4 — KBA Verification
    C->>NJ: Answer KBA questions
    NJ->>API: POST /applications/:id/kba/verify
    API->>API: Internal KBA stub (MVP)
    API->>A0: Set kba_verified claims (M2M)
    API->>R: Update verification state
    API->>DB: Update status → IDENTITY_VERIFIED
    API-->>NJ: KBA verified

    Note over C,SM: Phase 5 — Vehicle + Dealer Selection
    C->>NJ: Select vehicle (year/make/model)
    NJ->>API: PATCH /applications/:id/vehicle
    C->>NJ: Select dealer
    NJ->>API: PATCH /applications/:id/dealer

    Note over C,SM: Phase 6 — FCRA Consent + Decisioning
    C->>NJ: Review screen → Accept FCRA disclosure
    NJ->>API: POST /applications/:id/consent
    API->>R: Validate KBA + OTP non-expired
    API->>DB: Insert fcra_consents + set disclosure_status=accepted
    API->>DB: Update status → DISCLOSURE_ACCEPTED
    API->>API: Execute decisioning engine
    API->>DB: Read credit policy (tier + income band + max finance)
    API->>DB: Update status → PREQUALIFIED or DECLINED
    API-->>NJ: Decision result

    Note over C,SM: Phase 7 — Lead Delivery (PREQUALIFIED only)
    API->>DB: Generate ADF payload
    API->>API: Lead Distribution Service (stubbed)
    API->>SE: Send PREQUALIFIED email
    SE-->>C: Confirmation email delivered
```

---

## 4. Application State Machine

The 8 application states, 11 transitions, and 4 terminal states.

```mermaid
stateDiagram-v2
    [*] --> DRAFT: POST /applications

    DRAFT --> IDENTITY_VERIFIED: KBA verified
    DRAFT --> ABANDONED: New application created\nor session expired

    IDENTITY_VERIFIED --> DISCLOSURE_ACCEPTED: POST /consent
    IDENTITY_VERIFIED --> ABANDONED: Session expired\nor new application

    DISCLOSURE_ACCEPTED --> DECISIONING: Submit for decisioning
    DISCLOSURE_ACCEPTED --> ABANDONED: User starts over

    DECISIONING --> PREQUALIFIED: Score + income pass
    DECISIONING --> DECLINED: Score or income fail\nor derogatory hit

    PREQUALIFIED --> EXPIRED: decided_at + 30 days
    DECLINED --> EXPIRED: decided_at + 30 days

    PREQUALIFIED --> [*]
    DECLINED --> [*]
    ABANDONED --> [*]
    EXPIRED --> [*]

    note right of PREQUALIFIED: ADF lead delivered\nto dealer DMS
    note right of DECLINED: Adverse action\nemail sent (Path B)
    note left of ABANDONED: Prior application\npreserved in audit log
```

**State rules:**
- **Terminal states:** PREQUALIFIED, DECLINED, ABANDONED, EXPIRED
- **Single active application:** new DRAFT creation auto-abandons any existing DRAFT or IDENTITY_VERIFIED record for the same `consumer_user_id`
- **IDENTITY_VERIFIED not resumable** after session expiry (60-min KBA claim window elapsed)
- **DRAFT resumable** within a valid session

---

## 5. Decisioning Engine Flow

Credit policy evaluation from FCRA consent through outcome determination.

```mermaid
flowchart TB
    START["Consent Accepted<br/>(DISCLOSURE_ACCEPTED)"] --> GUARD

    GUARD{"Server-side guards<br/>KBA non-expired?<br/>OTP verified?<br/>Dealer matches consent?"} -->|"Any fail"| BLOCK["403 / 409<br/>Hard block"]
    GUARD -->|"All pass"| PULL

    PULL["Bureau Soft Pull<br/>(Stubbed MVP)"] --> CHECK1

    CHECK1{"Check 1<br/>Credit score >= 620?"} -->|"No"| DECLINE1["DECLINED<br/>SCORE_BELOW_MINIMUM"]
    CHECK1 -->|"Yes"| CHECK2

    CHECK2{"Check 2<br/>Derogatory indicators?<br/>(8 rules, automatic MVP)"} -->|"Hit"| DECLINE2["DECLINED<br/>Specific derogatory code(s)"]
    CHECK2 -->|"Clear"| CHECK3

    CHECK3{"Check 3<br/>Map to pricing tier<br/>(4 tiers)"} --> CHECK4

    CHECK4{"Check 4<br/>Map to income band<br/>(5 bands A-E)"} --> CHECK5

    CHECK5{"Check 5<br/>Stated income >= $2,000/mo?"} -->|"No"| DECLINE3["DECLINED<br/>INSUFFICIENT_INCOME"]
    CHECK5 -->|"Yes"| LOOKUP

    LOOKUP["Lookup max finance amount<br/>(tier x income band)<br/>+ rate range (tier)"] --> RESULT

    RESULT["PREQUALIFIED<br/>Rate range + max finance<br/>+ confirmation number"] --> DELIVER

    DELIVER["Lead Distribution<br/>(ADF to DealerTrack / RouteOne)<br/>+ PREQUALIFIED email"]

    DECLINE1 --> AA["Adverse Action Notice<br/>(Path B — email via SES)"]
    DECLINE2 --> AA
    DECLINE3 --> AA

    style BLOCK fill:#ffcdd2,stroke:#E53935
    style DECLINE1 fill:#ffcdd2,stroke:#E53935
    style DECLINE2 fill:#ffcdd2,stroke:#E53935
    style DECLINE3 fill:#ffcdd2,stroke:#E53935
    style RESULT fill:#c8e6c9,stroke:#43A047
    style DELIVER fill:#c8e6c9,stroke:#43A047
    style AA fill:#fff9c4,stroke:#F9A825
```

**Decisioning lookup query (single read):**
```sql
SELECT ct.tier_code, ct.rate_range_floor_bps, ct.rate_range_ceiling_bps,
       mfa.max_finance_amount_cents
FROM credit_tiers ct
JOIN income_bands ib ON ib.policy_version_id = ct.policy_version_id
JOIN max_finance_amounts mfa ON mfa.tier_code = ct.tier_code
                             AND mfa.band_code = ib.band_code
                             AND mfa.policy_version_id = ct.policy_version_id
WHERE ct.policy_version_id = (SELECT id FROM credit_policy_versions WHERE is_active = true)
  AND $credit_score BETWEEN ct.min_score AND ct.max_score
  AND $stated_income_cents BETWEEN ib.min_income_cents AND ib.max_income_cents;
```

---

## 6. Data Model — Entity Relationship

```mermaid
erDiagram
    credit_policy_versions ||--o{ credit_tiers : "has"
    credit_policy_versions ||--o{ income_bands : "has"
    credit_policy_versions ||--o{ max_finance_amounts : "has"
    credit_policy_versions ||--o{ derogatory_indicator_rules : "has"

    dealers ||--o{ applications : "receives leads"
    dealers ||--o{ fcra_consents : "named in consent"

    applications ||--o{ fcra_consents : "has consent records"

    applications {
        uuid id PK
        uuid consumer_user_id
        uuid dealer_id FK
        varchar brand "TOYOTA | LEXUS"
        varchar status "8-state machine"
        varchar disclosure_status "not_yet_presented | accepted | invalidated"
        text first_name "AES-256-GCM encrypted"
        text last_name "AES-256-GCM encrypted"
        text ssn_last_four "AES-256-GCM encrypted"
        text date_of_birth "AES-256-GCM encrypted"
        text phone_number "AES-256-GCM encrypted"
        text email "AES-256-GCM encrypted"
        varchar email_lookup_hash "HMAC-SHA256"
        int credit_score
        int stated_income_cents
        varchar credit_tier_code
        int rate_range_floor_bps
        int rate_range_ceiling_bps
        int max_finance_amount_cents
        varchar adverse_action_reason_1
        varchar adverse_action_reason_2
        varchar adverse_action_reason_3
        varchar adverse_action_reason_4
        varchar confirmation_number UK
        timestamptz decided_at
        timestamptz expires_at
        int key_version
    }

    dealers {
        uuid id PK
        varchar name
        varchar dba_name
        varchar brand "TOYOTA | LEXUS"
        varchar dealertrack_id UK
        varchar routeone_id UK
        boolean is_active
        timestamptz enrolled_at
    }

    fcra_consents {
        uuid id PK
        uuid application_id FK
        uuid dealer_id FK "immutable after insert"
        timestamptz accepted_at
        varchar disclosure_version
        varchar disclosure_status "accepted | invalidated"
        timestamptz invalidated_at
        varchar invalidated_reason
        timestamptz otp_verified_at "snapshot"
        timestamptz kba_verified_at "snapshot"
        varchar verification_context "snapshot"
        varchar session_id
        inet ip_address
        text user_agent
    }

    credit_policy_versions {
        uuid id PK
        varchar version_label
        boolean is_active "single active"
    }

    credit_tiers {
        uuid id PK
        uuid policy_version_id FK
        varchar tier_code "T1-T4"
        int min_score
        int max_score
        int rate_range_floor_bps
        int rate_range_ceiling_bps
    }

    income_bands {
        uuid id PK
        uuid policy_version_id FK
        varchar band_code "A-E"
        int min_income_cents
        int max_income_cents
    }

    max_finance_amounts {
        uuid id PK
        uuid policy_version_id FK
        varchar tier_code FK
        varchar band_code FK
        int max_finance_amount_cents
    }

    derogatory_indicator_rules {
        uuid id PK
        uuid policy_version_id FK
        varchar indicator_code
        varchar description
        int lookback_months
        boolean is_active
        boolean is_mvp_stub_automatic
    }
```

---

## 7. Encryption & Secrets Architecture

How encryption keys, secrets, and PII protection are layered across the system.

```mermaid
flowchart LR
    subgraph KMS_Layer["AWS KMS"]
        CMK["Customer-Managed Key<br/>(CloudTrail audit per call)"]
    end

    subgraph SM_Layer["AWS Secrets Manager"]
        AES_KEY["AES-256 Field<br/>Encryption Key"]
        HMAC_KEY["HMAC-SHA256<br/>Lookup Key"]
        M2M_CRED["Auth0 M2M<br/>Credentials"]
        REDIS_AUTH["Redis AUTH<br/>Token"]
        KBA_PSK["KBA Callback<br/>Pre-Shared Secret"]
        WEB_CRED["Auth0 Web App<br/>Credentials"]
        AUTH0_SEC["AUTH0_SECRET<br/>(Session Encryption)"]
    end

    subgraph App_Layer["Application Runtime"]
        ENCRYPT_MOD["Encryption Module<br/>(node:crypto only)"]
        HMAC_MOD["HMAC Lookup Module"]
    end

    subgraph DB_Layer["PostgreSQL RDS"]
        PII["10 PII Columns<br/>(AES-256-GCM encrypted)"]
        HASH["email_lookup_hash<br/>(HMAC-SHA256)"]
        AUDIT_TBL["audit_events<br/>(immutable — RULE + trigger + REVOKE + pgAudit)"]
    end

    CMK -->|"Decrypt"| SM_Layer
    AES_KEY --> ENCRYPT_MOD
    HMAC_KEY --> HMAC_MOD
    ENCRYPT_MOD -->|"Encrypt/Decrypt"| PII
    HMAC_MOD -->|"Hash"| HASH

    style KMS_Layer fill:#fce4ec,stroke:#E91E63
    style SM_Layer fill:#fff3e0,stroke:#FF9800
    style DB_Layer fill:#e3f2fd,stroke:#2196F3
```

**Ciphertext blob format:** `[4-byte key_version][12-byte IV][16-byte auth tag][ciphertext]` as base64url

**Key rules:**
- AES-256-GCM (not CBC) — authenticated encryption; tampered ciphertext fails explicitly
- Hard fail on startup if encryption key unavailable (5 retries then `process.exit(1)`)
- 503 with `Retry-After: 30` on in-request outage; 10-min stale-key grace window
- HMAC rotation requires re-hash sweep (one-way function; no dual-key window)
- No third-party crypto libraries — `node:crypto` only

---

## 8. Audit Log Architecture

23 named events across 3 retention categories with layered immutability enforcement.

| Category | Retention | Write Mode | Events |
|----------|-----------|------------|--------|
| **FCRA_7YR** | 7 years | Synchronous (in parent transaction) | consent_accepted, consent_invalidated, bureau_pull_executed, adverse_action_sent, lead_delivered |
| **SECURITY** | 1 year | Separate transaction | otp_sent, otp_verified, otp_failed, kba_verified, kba_failed, login_success, login_failed, ownership_mismatch |
| **OPERATIONAL** | 90 days | Separate transaction | application_created, application_submitted, application_decided, dealer_changed, vehicle_changed, email_sent, status_transition, policy_version_activated, encryption_key_rotated, application_abandoned |

**Immutability enforcement (4 layers):**
1. PostgreSQL RULE — silently suppresses UPDATE/DELETE (defense in depth)
2. PostgreSQL trigger — raises exception on UPDATE/DELETE (visible error in pgAudit)
3. Application role REVOKE — no UPDATE/DELETE privilege granted
4. pgAudit — logs any attempted modification

**PII exclusion:** `sanitizeEventData()` throws on known PII keys (does not silently strip). Event data contains only IDs, status codes, reason codes, and counts.

---

*← [Auto Leads Platform — Progress](../progress.md)*
