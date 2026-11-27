# FinSmart — System Architecture & Engineering Blueprint (ARCHITECTURE.md)

> **Document Version:** 1.0.0  
> **Status:** Approved Source of Truth  
> **Topology:** Cloud-Native 3-Container Docker Bridge with Multi-Tenant Row-Level Security (RLS)

---

## 1. High-Level Architecture Topology

FinSmart is architected as an isolated, containerized 3-tier system running via Docker Compose:

```
                            ┌─────────────────────────────────┐
                            │   Client Browser / Mobile PWA   │
                            │   (localhost:3000 / Subdomain)  │
                            └────────────────┬────────────────┘
                                             │ HTTP / HTTPS
                                             ▼
  ┌─────────────────────────────────────────────────────────────────────────────────┐
  │                        Docker Compose Bridge Network                            │
  │                                                                                 │
  │   ┌───────────────────────┐   Internal API    ┌─────────────────────────────┐   │
  │   │   finsmart-frontend   │──────────────────▶│   finsmart-backend          │   │
  │   │   Next.js 14+ (App)   │   HTTP :8000/v1   │   FastAPI (Python 3.12)     │   │
  │   │   Port 3000           │                   │   Port 8000 (OpenAPI /docs) │   │
  │   └───────────────────────┘                   └──────────────┬──────────────┘   │
  │                                                              │ Async Connection │
  │                                                              ▼                  │
  │                                                ┌────────────────────────────┐   │
  │                                                │   finsmart-database        │   │
  │                                                │   PostgreSQL 16 + pgvector │   │
  │                                                │   Port 5432 / 5433         │   │
  │                                                │   Row-Level Security (RLS) │   │
  │                                                └────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Folder Structure

```
finsmart/
├── .env.example                  # Canonical environment configuration template
├── .gitignore                    # Root gitignore (ignoring secrets, caches, scratch)
├── docker-compose.yml            # 3-service orchestration (frontend, backend, database)
├── README.md                     # High-level overview & quickstart
├── DATABASE_ARCHITECTURE.md      # Deep dive on RLS & schema
├── docs/                         # Source of truth specifications
│   ├── PRD.md                    # Product requirements & user journeys
│   ├── DESIGN.md                 # Design system, tokens, Indic typography
│   ├── ARCHITECTURE.md           # This engineering architecture blueprint
│   ├── DATABASE.md               # Data dictionary, constraints, indexes
│   ├── API.md                    # REST API specification & conventions
│   └── AGENTS.md                 # Engineering guidelines for future agents
├── docker/
│   ├── 01-roles.sh               # Database role initialization script
│   └── 02-schema.sql             # Canonical PostgreSQL DDL & seed data
├── backend/                      # FastAPI Application
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── main.py               # FastAPI entrypoint, middleware & lifecycle
│   │   ├── core/
│   │   │   ├── config.py         # Pydantic BaseSettings environment loader
│   │   │   ├── database.py       # SQLAlchemy 2.0 async engine & sessionmaker
│   │   │   ├── security.py       # Argon2id/Bcrypt password hashing & JWT
│   │   │   ├── tenant.py         # X-Tenant-ID RLS session binding
│   │   │   ├── money.py          # Minor-unit integer paise calculations
│   │   │   └── pii.py            # Regex scrubber for Aadhaar/PAN/Phone
│   │   ├── api/v1/
│   │   │   ├── router.py         # Central v1 endpoint aggregation
│   │   │   └── endpoints/        # Modular API routes (auth, txns, budgets...)
│   │   ├── schemas/              # Typed Pydantic request/response models
│   │   └── services/             # Domain logic (analytics, RAG AI, rule DSL)
│   └── tests/                    # Unit & regression test suite (unittest)
└── frontend/                     # Next.js 14 Web Application
    ├── Dockerfile
    ├── package.json
    ├── tsconfig.json
    ├── next.config.js
    ├── components.json           # shadcn / React Bits configuration
    ├── app/
    │   ├── layout.tsx            # Root layout with meta tags & security headers
    │   ├── page.tsx              # Primary responsive SPA dashboard
    │   ├── not-found.tsx         # Custom 404 page with return link
    │   ├── privacy/page.tsx      # DPDP compliant privacy policy
    │   ├── terms/page.tsx        # Terms of service & financial advisory disclaimer
    │   └── globals.css           # Design tokens, CSS variables, keyframe animations
    ├── components/
    │   ├── react-bits/           # Verified animation suite (BlurText, CountUp...)
    │   └── ui/                   # Reusable UI primitives (Button, Card, Banners...)
    ├── lib/
    │   ├── api.ts                # Typed client fetch wrapper & minor-unit helpers
    │   ├── i18n.ts               # Dynamic translation dictionary (en, kn, hi)
    │   ├── motion.ts             # Shared motion tokens & easing configs
    │   └── tokens.ts             # TypeScript design token declarations
    └── public/                   # Static assets (favicons, sitemaps, robots.txt)
```

---

## 3. Data Flow & Multi-Tenant Security Boundaries

```
[Client Request]
    │ Headers:
    │   Authorization: Bearer <JWT>
    │   X-Tenant-ID: <slug> (e.g. 'public', 'navkis')
    ▼
[FastAPI Middleware & Dependencies]
    │ 1. get_tenant_db(): Resolves tenant slug -> validates is_active -> queries tenant_id
    │ 2. Sets transaction-local context:
    │    SELECT set_config('app.current_tenant_id', tenant_id, true);
    │ 3. get_current_user(): Decodes JWT -> verifies token.tid === header.tenant_id
    ▼
[PostgreSQL Database (RLS Enforced)]
    │ Current Role: 'finsmart_app' (NOSUPERUSER, NOBYPASSRLS)
    │ Table Policy: USING (tenant_id = current_setting('app.current_tenant_id')::uuid)
    ▼
[Data Returned (Strictly Isolated to Tenant & User)]
```

### Invariants
1. **No Superuser at Runtime**: The FastAPI backend connects strictly as the non-superuser role `finsmart_app`. Because it does not possess `BYPASSRLS`, PostgreSQL enforces tenant boundaries directly at the kernel storage level.
2. **Double Tenant Validation**: A valid JWT issued for Tenant A (`navkis`) cannot be reused to access Tenant B (`rvce`) even if the user attempts header spoofing; `get_current_user()` strictly rejects token-header mismatches.

---

## 4. Money Architecture: Minor-Unit Precision

To prevent IEEE-754 floating-point rounding bugs common in web financial apps:
- **Storage**: All currency values are stored as 64-bit integers (`BIGINT`) representing **paise** (1 Rupee = 100 Paise).
- **Exchange Rule**: Values are only divided by 100 at the final presentation layer (`formatINR()` / `toRupees()`).
- **Math**: All interest and SIP compounding formulas in `app/core/money.py` utilize Python's `decimal.Decimal` with explicit `ROUND_HALF_UP` banking rounding before converting to integer paise.

---

## 5. Security Architecture & Threat Model

| Threat | Mitigation Strategy | Implementation |
| :--- | :--- | :--- |
| **Cross-Tenant Data Leak** | PostgreSQL Row-Level Security | `tenant_isolation_policy` on all private tables. |
| **SQL Injection** | Parameterized Queries | SQLAlchemy `text()` with bound parameters `:u`, `:t`. |
| **XSS & Clickjacking** | HTTP Security Headers | `Content-Security-Policy`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`. |
| **PII Data Harvest** | In-Memory Regex Redaction | [app/core/pii.py](file:///c:/Users/VIVEK%20I%20H/Downloads/finsmart/backend/app/core/pii.py) scrubs Aadhaar, PAN, and Phone prior to logging or RAG evaluation. |
| **Credential Stuffing** | Modern Password Hashes | Argon2id / Bcrypt (`passlib`) with per-user salt. |
| **Replay & Offline Jitter** | Idempotency Keys | Client generates unique `client_id` UUID per transaction. |

---

## 6. MVP vs. Future Scale Architecture

### MVP (Current Clean Architecture)
- Single PostgreSQL database instance with Docker volume persistence.
- Next.js client-side SPA state with typed `api()` client and responsive tab navigation.
- In-memory rate limiting and RAG keyword/vector simulation in PostgreSQL.

### Future Scaling Roadmap
1. **Redis Cache Layer**: Offload rate limiting and session tokens to a distributed Redis cluster.
2. **Account Aggregator (AA) Gateway**: Integrate RBI-approved Financial Information Providers (FIPs) via secure consent handles.
3. **Dedicated Background Workers**: Celery / Redis queue for generating monthly PDF statements and asynchronous SMS/WhatsApp welfare alerts.
