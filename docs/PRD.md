# FinSmart — Product Requirements Document (PRD)

> **Document Version:** 1.0.0  
> **Status:** Approved Source of Truth  
> **Target Audience:** General Public, Rural Households, First-Time Earners, Students (VTU Team LYNX)  
> **Core Mission:** Empower ordinary people with household budgeting, savings milestone tracking, vernacular financial education, fraud avoidance, and automated welfare scheme discovery with zero technical jargon.

---

## 1. Problem Statement

Across India, and particularly in non-metro and rural communities:
1. **Low Financial Literacy & Predatory Risks**: Millions of students and first-time earners fall victim to illegal instant lending apps, phishing UPI requests, and unregulated financial instruments.
2. **Untracked Cash & UPI Micro-Leaks**: Daily digital micro-payments via UPI make money invisible. Families struggle to maintain a basic monthly surplus or calculate daily safe-to-spend limits.
3. **Information Asymmetry in Welfare Schemes**: Substantial central and state benefits (scholarships, Jan Dhan accounts, Atal Pension, Sukanya Samriddhi, MUDRA loans) remain unclaimed due to complex eligibility circulars, lack of local language guidance, and opaque documentation requirements.
4. **Data Privacy Vulnerabilities**: Low-income users frequently have their sensitive identifiers (Aadhaar, PAN, phone numbers) logged or harvested without informed consent.

FinSmart solves this by providing a unified, calm, privacy-first civic fintech planner built specifically for ordinary smartphones in local Indian languages.

---

## 2. Target Users & Personas

| Persona | Demographics | Tech Profile | Primary Needs |
| :--- | :--- | :--- | :--- |
| **Ramesh (Rural Farmer / Daily Earner)** | Age 42, Hassan District, Kannada native | Low-end Android phone, patchy 4G | Simple expense entry, vernacular interface, understanding government pension (APY) and PMJDY benefits. |
| **Ananya (First-Generation Engineering Student)** | Age 19, RVCE Bengaluru, Hindi/English | Mid-range phone, active UPI user | Hostel budget management, tracking tuition savings goals, checking Karnataka SSP scholarship eligibility. |
| **Sunita (Homemaker / Micro-Entrepreneur)** | Age 35, Tier-2 Karnataka, Kannada/Hindi | Shared family mobile phone | Household monthly budget limits, 50/30/20 guidance, avoiding fake loan apps, privacy rights. |
| **Prof. Kulkarni (College Administrator / Faculty)** | Age 50, VTU affiliated institution | Desktop browser | Overseeing student financial literacy adoption without viewing any private personal financial data. |

---

## 3. User Journeys

### Journey 1: Ledger & Daily Allowance
1. User logs in (or continues persistent session).
2. Sets a monthly category budget (e.g., Food: ₹4,000).
3. The app dynamically calculates the **Daily Safe Allowance** (e.g., `₹133/day remaining`).
4. User logs an expense (e.g., ₹80 for groceries via UPI).
5. The daily allowance updates immediately; progress meter shifts to green, amber, or red.

### Journey 2: Welfare Scheme Discovery & Evaluation
1. User navigates to the *Schemes* tab.
2. Fills in basic demographic criteria (Age, Student status, Annual family income, Social category, State domicile, Bank account status).
3. Clicks **"Evaluate Eligibility"**.
4. The deterministic JSON Rule Engine evaluates criteria locally and instantly displays:
   - Green badge: **Likely Eligible** with explicit clause: *"Criteria satisfied: Annual income ≤ ₹2,50,000"*.
   - Amber badge: **Criteria Not Met** with reason: *"Criteria not met: Age between 18 and 40 years"*.
5. User expands the scheme to view required documents checklist and official government portal link.

### Journey 3: Vernacular Learning & Cyber Fraud Shield
1. User selects **ಕನ್ನಡ (KN)** or **हिन्दी (HI)**.
2. Reads an interactive module on UPI safety and fake loan app red flags.
3. Completes an interactive 1-question knowledge check.
4. Receives immediate feedback with educational explanation.

### Journey 4: AI Financial Mentor (RAG-Grounded)
1. User asks: *"How do I start a savings buffer on a ₹3,000 pocket money budget?"*
2. System scrubs any typed Aadhaar/PAN/phone numbers before processing.
3. System blocks speculative trading prompts (stocks/crypto/F&O).
4. System retrieves verified advisory chunks from the `knowledge_docs` database and replies with citations.

### Journey 5: DPDP Act 2023 Data Rights
1. User visits the *Profile* tab.
2. Inspects all active consent purposes granted.
3. Downloads a statutory Data Subject Access Request (DSAR) JSON export in one click.
4. Optionally triggers the **Right to be Forgotten** to permanently erase all user ledger records.

---

## 4. Feature Specifications

### 4.1 Household Ledger & Cashflow Tracker
- **Fields**: Date, Amount (in ₹), Kind (`DEBIT` / `CREDIT`), Category (`Food`, `Education`, `Books`, `Rent`, `Travel`, `Bills`, `Other`), Payment Mode (`UPI`, `Cash`, `Card`, `Bank`, `Wallet`), Description.
- **Precision**: Stored as exact integer paise minor units (`100` paise = ₹1.00). Zero floating-point rounding errors.
- **Idempotency**: Client generates a `client_id` UUID per transaction to prevent double-charging over slow mobile connections.

### 4.2 Budgets & Overspend Monitoring
- **Thresholds**:
  - Normal: `< 80%` of budget spent (Green).
  - Amber Warning: `80% - 99%` of budget spent.
  - Overspend Alert: `≥ 100%` of budget spent (Red).
- **Daily Safe Allowance**: `(Limit - Spent) / Days Remaining in Month`.

### 4.3 Savings Goals & Milestone Countdown
- **Fields**: Name, Target Amount, Target Date, Category.
- **Auto-Allocation**: Recommends monthly savings required based on days remaining until the target date.
- **Deposit**: Instant contribution recording with visual progress percentage.

### 4.4 Declarative Schemes Engine
- **Supported Programs**: PM Jan Dhan Yojana (PMJDY), Atal Pension Yojana (APY), NSP Post-Matric Scholarship, Karnataka SSP Post-Matric Scholarship, PM MUDRA Loan, Sukanya Samriddhi Yojana (SSY).
- **Execution**: Evaluated client/server side through deterministic JSON Rule DSL (`all`, `eq`, `gte`, `lte`, `between`, `in`).
- **Transparency**: Every evaluation outputs explainable human-readable clauses.

### 4.5 Financial Calculators
- **Loan EMI**: Computes monthly installment, total interest paid, and principal/interest ratio using bankers rounding.
- **SIP Wealth Accumulator**: Computes invested capital vs. compound wealth gain over monthly tenures.
- **Emergency Fund Planner**: Sizes mandatory 3-to-6 month liquid survival buffers based on essential monthly living expenses.

### 4.6 AI Financial Mentor
- **Guardrails**: Hardcoded refusal patterns against cryptocurrency speculation, stock tips, get-rich schemes, gambling, and F&O derivatives.
- **PII Redaction**: Regex scrubbing of 12-digit Aadhaar numbers, 10-character PAN cards, and 10-digit Indian phone numbers.
- **Citations**: All responses must return verified source citations (e.g. RBI, SEBI, MoF).

### 4.7 Privacy & DPDP Compliance
- **Consent Ledger**: Explicit purpose tracking with timestamped audit records.
- **DSAR Export**: Single-click statutory JSON download of profile, accounts, transactions, and budgets.
- **Account Erasure**: Permanent database cascading deletion of user profile and financial transactions.

### 4.8 Multi-Tenant Institution Architecture
- **Isolation**: PostgreSQL Row-Level Security (`FORCE ROW LEVEL SECURITY`) with session-bound `app.current_tenant_id`.
- **Admin Analytics**: Anonymized participation rollups (total enrolled users, volume tracked, active goals) with zero access to student transactions.

---

## 5. Non-Functional Requirements (NFRs)

1. **Performance**:
   - First Contentful Paint (FCP) `< 1.0s` on 4G mobile emulation (Pixel 7).
   - Total mobile page weight `< 1.5MB`.
   - Zero layout shifts (CLS `< 0.05`) through `tabular-nums` on financial figures.
2. **Accessibility & Usability**:
   - Full WCAG AA color contrast (`≥ 4.5:1` for normal text).
   - Large touch targets (`≥ 48px` minimum height on buttons and inputs).
   - Zero horizontal overflow (`scrollWidth === clientWidth`) across viewports from 320px to 4K.
   - Respects `prefers-reduced-motion` at all times.
3. **Indic Script Typography**:
   - Flawless rendering of Kannada (`ಕನ್ನಡ`) and Hindi (`हिन्दी`) without broken viramas or split conjuncts.
4. **Security**:
   - Zero secrets exposed in client bundles.
   - HTTPS enforcement with HSTS (`max-age=63072000`).
   - Content Security Policy (CSP), `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`.
   - Argon2id / Bcrypt password hashing and HMAC-SHA256 JWT tokens.

---

## 6. MVP Scope vs. Out-of-Scope

### In MVP Scope (Current Release)
- Single-page responsive web application with Tab navigation.
- Full offline-first design token system in English, Kannada, and Hindi.
- Multi-tenant database isolation (Public, Navkis, RVCE).
- Complete Ledger, Budgets, Goals, Schemes, Calculators, Literacy Modules, AI Mentor, and Privacy Center.
- Docker Compose 3-container topology.

### Out-of-Scope for Current MVP (Future Roadmap)
- Automated SMS bank transaction scraping (requires native Android APK permissions).
- Direct Account Aggregator (AA) live bank API integration (requires RBI NBFC-AA certification).
- UPI payment initiation (FinSmart is an educational planner, not a payment gateway).
- Native iOS Swift app (PWA covers mobile web requirements).

---

## 7. Success Criteria
1. Zero cross-tenant data leakage in multi-tenant RLS penetration tests.
2. 100% correct calculations for currency conversions (minor units) and loan amortization schedules.
3. Green rating in Google Lighthouse / Core Web Vitals on mobile emulation.
4. Total compliance with India Digital Personal Data Protection (DPDP) Act 2023.
