# FinSmart: Smart Financial Literacy & Budget Planner for Public

> **A scalable, production-grade, industrial-standard mobile/web application designed to empower citizens, students, and low-to-middle-income households with household budgeting, savings milestone tracking, government welfare scheme discovery, and grounded AI financial literacy support.**

---

## 1. System Architecture & 3-Container Topology

FinSmart runs a strict, cloud-native **3-Container Docker Topology** with zero external dependencies:

```
                          ┌────────────────────────────────┐
                          │   Client Browser / Mobile PWA  │
                          │   (Subdomain / localhost:3000) │
                          └───────────────┬────────────────┘
                                          │ HTTP
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      Docker Compose Bridge Network                              │
│                                                                                 │
│   ┌──────────────────────┐    Internal API    ┌─────────────────────────────┐   │
│   │   frontend           │───────────────────▶│   backend                   │   │
│   │   Next.js 14+ (App)  │    HTTP :8000/v1   │   FastAPI (Python 3.12)     │   │
│   │   Port 3000          │                    │   Port 8000 (OpenAPI /docs) │   │
│   └──────────────────────┘                    └──────────────┬──────────────┘   │
│                                                              │ Async Connection │
│                                                              ▼                  │
│                                               ┌─────────────────────────────┐   │
│                                               │   database                  │   │
│                                               │   PostgreSQL 16 + pgvector  │   │
│                                               │   Port 5432 (Host Mapped)   │   │
│                                               │   Row-Level Security (RLS)  │   │
│                                               └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
                                          ▲
                                          │ Host Port 5432
                                          │
                          ┌───────────────┴────────────────┐
                          │   DBeaver Database GUI Client  │
                          │   (localhost:5432/finsmart_db) │
                          └────────────────────────────────┘
```

1. **Frontend (`finsmart-frontend`)**: Next.js 14+ (App Router, TypeScript, Responsive CSS Design System, Multi-Language EN/KN/HI support) on port `3000`.
2. **Backend (`finsmart-backend`)**: Modern FastAPI (Python 3.12, strict Pydantic v2 validation, SQLAlchemy 2.0 async engine, Argon2id/Bcrypt cryptography, RAG Knowledge Assistant, pure declarative rules engine) on port `8000`.
3. **Database (`finsmart-database`)**: PostgreSQL 16 with `pgvector` extension, multi-tenant Row-Level Security (RLS) enforcement, and persistent storage on port `5432`.

---

## 2. Industrial Standards & Invariants

- **Multi-Tenant College & Institution Isolation**: Partitioned at database layer using PostgreSQL **Row-Level Security (RLS)** with `FORCE ROW LEVEL SECURITY`. Application requests bind transaction-local context via `SET LOCAL app.current_tenant_id = :tenant_id`, guaranteeing zero cross-tenant data leakage.
- **Money Safety (Minor Units)**: Stored strictly as `bigint` paise (₹1.00 = `100` paise). Decimal calculators use `ROUND_HALF_UP` bankers rounding with zero floating-point imprecision.
- **India DPDP Act 2023 Compliance**: Mandatory explicit consent logging, granular purpose toggling, automated Data Subject Access Request (**DSAR**) JSON data export, and Right to be Forgotten account erasure.
- **Automated PII Scrubbing**: Real-time regex sanitization of Aadhaar numbers, PAN cards, and 10-digit Indian phone numbers prior to logging or AI prompt evaluation.
- **AI Financial Guardrails**: Grounded RAG assistant with verified citations that strictly refuses speculative stock trading, intraday calls, or get-rich schemes in accordance with SEBI/RBI investor education guidelines.

---

## 3. Quick Start with Docker

### Step 1: Clone & Configure Environment
```bash
cp .env.example .env
```
*(Review `.env` to verify your database credentials and secret keys).*

### Step 2: Boot the 3-Container Environment
```bash
docker compose up --build
```

### Step 3: Access the Applications
| Service | URL | Description |
|---|---|---|
| **Web Application** | [http://localhost:3000](http://localhost:3000) | Next.js Frontend (Mobile & Web responsive) |
| **API Documentation** | [http://localhost:8000/docs](http://localhost:8000/docs) | Interactive Swagger / OpenAPI Specification |
| **Alternative Docs** | [http://localhost:8000/redoc](http://localhost:8000/redoc) | ReDoc Architectural Documentation |
| **PostgreSQL Database** | `localhost:5432` | Accessible directly via **DBeaver** or `psql` |

---

## 4. DBeaver Connection Instructions

The PostgreSQL database is exposed on host port `5433` (configured via `DB_PORT` in `.env`).

### Admin Profile (Superuser Access)
- **Database Engine:** PostgreSQL
- **Host:** `localhost`
- **Port:** `5433` (or `5432`)
- **Database:** `finsmart_db`
- **Username:** `finsmart_admin` (from `POSTGRES_USER` in `.env`)
- **Password:** `change_me_admin` (from `POSTGRES_PASSWORD` in `.env`)

### Application Profile (Row-Level Security Verification)
- **Username:** `finsmart_app` (from `APP_DB_USER` in `.env`)
- **Password:** `change_me_app` (from `APP_DB_PASSWORD` in `.env`)

*For complete technical schema details, see the [Database Architecture Document](file:///c:/Users/VIVEK%20I%20H/Downloads/finsmart/DATABASE_ARCHITECTURE.md) and the [DBeaver Setup Guide](file:///c:/Users/VIVEK%20I%20H/Downloads/finsmart/docs/DBEAVER_SETUP.md).*

---

## 5. Core Features Breakdown

### 📊 1. Household Ledger & Budget Tracker
- Real-time tracking of cash, bank, and UPI transactions.
- Automated monthly cashflow summary: Total Income, Total Expenses, Net Balance, and Savings Rate %.
- Category breakdown with visual progress meters.
- Idempotent creation via `client_id` for network resilience and offline tolerance.

### 🎯 2. Budgets & Overspend Alerting
- Monthly category budget allocations with dynamic daily spending allowances.
- Real-time thresholds: Normal (`<80%`), Amber Warning (`80% - 99%`), and Red Alert (`≥100%`).

### 🏆 3. Savings Goals with Milestone Countdown
- Goal tracking with target dates and recommended monthly contribution calculations.
- Instant deposit contributions with celebratory progress bars.

### 🏛️ 4. Government Schemes & Scholarship Discovery (Flagship)
- Curated repository of Indian Central & Karnataka State welfare programs (PM Jan Dhan Yojana, Atal Pension Yojana, NSP Post-Matric Scholarship, Karnataka SSP Scholarship, PM MUDRA, Sukanya Samriddhi).
- Pure declarative JSON Rule DSL: evaluates demographic, income, and education criteria deterministically.
- Human-readable explainability clauses: "Why you are eligible" vs "Criteria not met".
- Interactive document checklist and step-by-step application walkthroughs with direct links to official government portals.

### 🤖 5. FinSmart AI Financial Mentor
- Grounded RAG assistant with pgvector and semantic document retrieval.
- Automated PII scrubbing for Aadhaar, PAN, and phone numbers.
- Statutory investor protection guardrails rejecting speculative trading advice.

### 🧮 6. Precision Financial Calculators
- **Loan EMI Calculator**: Monthly EMI, total interest, and principal amortization ratio.
- **SIP Wealth Accumulator**: Future corpus calculation with compound returns.
- **Emergency Fund Planner**: Sizing a liquid 3–6 month mandatory living buffer.
- **Compound Interest Comparison**: Long-term sovereign compounding.

### 📚 7. Vernacular Literacy & Cyber Fraud Shield
- Multilingual education in **English**, **ಕನ್ನಡ (Kannada)**, and **हिन्दी (Hindi)**.
- Topics covering 50/30/20 budgeting, UPI PIN safety, and avoiding predatory loan apps.
- Interactive knowledge checks with immediate answer feedback and explanations.

### 🛡️ 8. DPDP Act 2023 Privacy Center
- Granular consent ledger with purpose specification and withdrawal workflows.
- Data Subject Access Request (**DSAR**) single-click JSON export.
- Right to be Forgotten account erasure.

### 🏫 9. Institution Administrator Console
- Anonymized participation statistics for college administrators (enrolled students, volume tracked, active goals) with zero student personal financial records exposed.

---

## 6. Running Local Verification Tests

Run the backend unit test suite directly:
```bash
cd backend
python -m unittest discover -s tests
```
*Expected Result:*
```
Ran 19 tests in 0.588s - OK
```

To reset the database and re-seed from scratch:
```bash
docker compose down -v
docker compose up --build
```
