# FinSmart — Master Implementation Plan (PLAN.md)

> **Document Version:** 1.0.0  
> **Status:** Approved Blueprint for Execution  
> **Execution Strategy:** Ordered phases. Each phase is small, focused, verifiable, and runnable at all times.

---

## Overview of Execution Roadmap

```
Phase 4: Foundation (Config, tokens, structure, i18n, Docker verification)
   │
   ▼
Phase 5: Backend & Database Hygiene (Declarative models, Alembic, connection pool, indices)
   │
   ▼
Phase 6: Authentication & Authorization (Persistent mobile auth, tenant selector, secure sessions)
   │
   ▼
Phase 7: Features Execution & Hardening (Ledger, Budgets, Goals, Schemes, Calculators, AI)
   │
   ▼
Phase 8: UI Polish & Indic Verification (Aesthetics, anti-slop, responsive layouts, ligature checks)
   │
   ▼
Phase 9: Launch Hygiene (Legal, Privacy Policy, Terms, SEO, security headers, analytics gate)
   │
   ▼
Phase 10: Testing & QA (Unit, integration, Playwright mobile E2E journeys)
   │
   ▼
Phase 11: Security & Compliance Audit (RLS penetration test, OWASP ASVS, DPDP Act 2023 check)
   │
   ▼
Phase 12: Architectural Code Review (Dead code elimination, bundle size, abstractions)
   │
   ▼
Phase 13: Deployment & Monitoring (Docker healthchecks, production env, backup scripts)
   │
   ▼
Phase 14: Final Production Audit (Comprehensive report, verification checklist, verdict)
```

---

## Detailed Phase Breakdown

### Phase 4: Foundation
- **Goal**: Consolidate repository cleanliness, enforce strict TypeScript/Python linting standards, ensure clean clone bootstrapping from `.env.example`, and finalize shared design tokens.
- **Tasks**:
  1. Verify `.env.example` has all needed environment keys for frontend, backend, and postgres.
  2. Unify design tokens in `frontend/lib/tokens.ts` and ensure direct synchronization with `frontend/app/globals.css`.
  3. Verify clean clone Docker boot sequence (`docker compose up --build`).
  4. Ensure `frontend/` and `backend/` scripts run cleanly in isolation.
- **Files Affected**:
  - `docker-compose.yml`, `.env.example`
  - `frontend/lib/tokens.ts`, `frontend/app/globals.css`
  - `backend/app/core/config.py`
- **Dependencies**: Phase 2 source-of-truth docs.
- **DB/API Changes**: None.
- **Tests**: `npm run build` in `frontend/`, `python -m unittest discover -s tests` in `backend/`.
- **Completion Criteria**: Clean clone builds and boots with zero runtime warnings or missing environment variables.

---

### Phase 5: Backend & Database Hygiene
- **Goal**: Establish formal database migrations via Alembic, provide declarative SQLAlchemy models as a programmatic contract, and optimize database indexing.
- **Tasks**:
  1. Initialize Alembic inside `backend/` with async SQLAlchemy configuration matching `app.core.database.engine`.
  2. Implement declarative SQLAlchemy ORM models in `backend/app/models/` matching `docker/02-schema.sql`.
  3. Generate initial revision migration baseline (`alembic/versions/001_initial_schema.py`).
  4. Verify PostgreSQL composite indices (`idx_transactions_user_date`, `idx_transactions_tenant_user`).
- **Files Affected**:
  - `backend/alembic.ini`, `backend/alembic/env.py`
  - `backend/app/models/__init__.py`, `backend/app/models/ledger.py`, `backend/app/models/tenant.py`
  - `backend/requirements.txt`
- **Dependencies**: Phase 4 Foundation.
- **DB/API Changes**: Introduces versioned migration tracking table (`alembic_version`).
- **Tests**: `alembic upgrade head`, backend unit tests.
- **Completion Criteria**: Clean database migration upgrade/downgrade executes without errors; models accurately mirror `02-schema.sql`.

---

### Phase 6: Authentication & Authorization
- **Goal**: Harden authentication, enable persistent mobile sessions for rural and low-end phone users, and expose an institutional tenant selector.
- **Tasks**:
  1. Add optional persistent session storage on mobile (30-day token support with secure local storage fallback).
  2. Add an explicit "Select Institution / General Public" dropdown on the login/registration card.
  3. Enforce strict role-based access control (RBAC) preventing non-admin users from invoking `/admin/stats`.
  4. Ensure token expiry cleanly triggers session cleanup without unhandled client exceptions.
- **Files Affected**:
  - `frontend/app/page.tsx`, `frontend/lib/api.ts`
  - `backend/app/api/v1/endpoints/auth.py`, `backend/app/core/tenant.py`
- **Dependencies**: Phase 5 Database models.
- **DB/API Changes**: None.
- **Tests**: Test token expiry handling, tenant isolation verification, and invalid credential rejections.
- **Completion Criteria**: Users can select their institution, log in, remain logged in across mobile browser restarts, and unauthorized cross-tenant requests fail with HTTP 401/403.

---

### Phase 7: Features Execution & Hardening
- **Goal**: Verify and harden each of the 9 core modules to ensure all loading, empty, success, and error states are robust.
- **Tasks**:
  1. **Ledger & Cashflow**: Verify idempotency of transaction logging via `client_id`; verify integer paise accuracy.
  2. **Budgets**: Verify dynamic calculation of daily safe allowances; verify threshold badge levels (`ok`, `warning`, `over`).
  3. **Savings Goals**: Verify milestone progress percentages and monthly recommended contribution calculations.
  4. **Schemes Matcher**: Verify deterministic evaluation of all 6 central and state welfare programs with explainable reasons.
  5. **Calculators**: Verify EMI, SIP, and Emergency Fund math using bankers rounding.
  6. **AI Mentor**: Verify PII scrubbing on typed prompts and ensure financial safety guardrails reject speculative queries.
  7. **DPDP Center**: Verify statutory DSAR JSON download and account erasure cascade.
- **Files Affected**:
  - `frontend/app/page.tsx`, `backend/app/api/v1/endpoints/*.py`
  - `backend/app/services/rules_engine.py`, `backend/app/services/ai_service.py`
- **Dependencies**: Phase 6 Auth.
- **DB/API Changes**: None.
- **Tests**: Unit tests for rules engine, money calculations, and RAG guardrails.
- **Completion Criteria**: Every feature handles empty states (e.g. no transactions), loading states, and error responses with accessible inline feedback.

---

### Phase 8: UI Polish & Indic Typography Verification
- **Goal**: Guarantee zero AI-slop visual tells, verify Indic typography in Kannada and Hindi, and enforce mobile touch ergonomics.
- **Tasks**:
  1. Audit all screens against `docs/DESIGN.md` guidelines.
  2. Verify that word-level splitting (`animateBy="words"`) operates without breaking Kannada and Hindi ligatures.
  3. Ensure all interactive touch targets meet or exceed `48px` height.
  4. Confirm zero horizontal scroll across mobile viewports (320px to 412px).
- **Files Affected**:
  - `frontend/app/globals.css`, `frontend/app/page.tsx`
  - `frontend/components/react-bits/*`
- **Dependencies**: Phase 7 Features.
- **DB/API Changes**: None.
- **Tests**: Playwright mobile emulation (Pixel 7).
- **Completion Criteria**: Design conforms to Civic Ledger aesthetic; mobile overflow check reports `hasOverflow: false`; Kannada/Hindi text displays cleanly.

---

### Phase 9: Launch Hygiene
- **Goal**: Establish comprehensive SEO, security headers, legal compliance pages, and analytics consent gating.
- **Tasks**:
  1. Verify Privacy Policy (`/privacy`) and Terms & Conditions (`/terms`) with financial educational disclaimer.
  2. Verify cookie consent banner blocks tracking until explicit user acceptance.
  3. Ensure security headers (HSTS, CSP, X-Frame-Options, Permissions-Policy) are present on all responses.
  4. Validate `sitemap.xml`, `robots.txt`, `favicon.svg`, and OpenGraph preview images.
- **Files Affected**:
  - `frontend/app/layout.tsx`, `frontend/app/privacy/page.tsx`, `frontend/app/terms/page.tsx`
  - `frontend/components/ui/CookieConsent.tsx`, `frontend/public/*`
- **Dependencies**: Phase 8 UI polish.
- **DB/API Changes**: None.
- **Tests**: HTTP response header inspection, metadata audit.
- **Completion Criteria**: 100% security headers active; legal pages accessible; cookie consent functional.

---

### Phase 10: Testing & QA
- **Goal**: Run comprehensive automated end-to-end user journeys and regression tests.
- **Tasks**:
  1. Expand backend test suite to cover all endpoint routers.
  2. Execute automated Playwright E2E tests for the core student journey: Registration -> Set Budget -> Record Transaction -> Check Scheme Eligibility.
  3. Validate negative test cases (invalid login, expired token, negative money amounts).
- **Files Affected**:
  - `backend/tests/test_api_endpoints.py`
  - `frontend/scratch/` or automated test scripts
- **Dependencies**: Phase 9 Launch hygiene.
- **DB/API Changes**: None.
- **Tests**: `python -m unittest discover -s tests`, Playwright E2E runner.
- **Completion Criteria**: All automated test suites exit with code 0.

---

### Phase 11: Security & Compliance Audit
- **Goal**: Verify multi-tenant RLS boundaries and validate OWASP ASVS and DPDP Act 2023 compliance.
- **Tasks**:
  1. Perform cross-tenant penetration test: attempt to read Tenant A's transactions using Tenant B's JWT token.
  2. Verify automated PII redaction on simulated Aadhaar, PAN, and phone submissions.
  3. Verify Right to be Forgotten permanently wipes user records from database tables.
- **Files Affected**:
  - `backend/tests/test_security.py`
- **Dependencies**: Phase 10 Testing.
- **DB/API Changes**: None.
- **Tests**: Automated RLS leak penetration tests.
- **Completion Criteria**: Zero data leak detected; RLS kernel enforcement confirmed.

---

### Phase 12: Architectural Code Review
- **Goal**: Eliminate redundant code, optimize frontend bundle size, and verify documentation alignment.
- **Tasks**:
  1. Scan for dead code, unused CSS classes, or orphaned imports.
  2. Verify dependency trees in `frontend/package.json` and `backend/requirements.txt`.
  3. Confirm exact alignment with `/docs` specifications.
- **Files Affected**:
  - `frontend/app/page.tsx`, `backend/app/main.py`
- **Dependencies**: Phase 11 Security audit.
- **DB/API Changes**: None.
- **Tests**: `npx tsc --noEmit`, Python `flake8` / `py_compile`.
- **Completion Criteria**: Codebase is clean, modular, and minimal.

---

### Phase 13: Deployment & Monitoring
- **Goal**: Finalize production build scripts, Docker healthchecks, and operational backup procedures.
- **Tasks**:
  1. Verify multi-stage Docker build caching and minimal container image footprints.
  2. Configure Docker container healthchecks and graceful restart policies.
  3. Document production deployment runbooks and automated backup procedures.
- **Files Affected**:
  - `docker-compose.yml`, `backend/Dockerfile`, `frontend/Dockerfile`
- **Dependencies**: Phase 12 Code review.
- **DB/API Changes**: None.
- **Tests**: `docker compose up --build` from clean clone.
- **Completion Criteria**: All 3 containers report `healthy` status and application is accessible on ports 3000 and 8000.

---

### Phase 14: Final Production Audit
- **Goal**: Deliver a comprehensive audit report with final verdict.
- **Tasks**:
  1. Inspect every requirement from `docs/PRD.md` against live runtime behavior.
  2. Issue formal verdict: **READY**, **READY WITH WARNINGS**, or **NOT READY**.
  3. Document all production placeholder credentials needing user customization.
- **Files Affected**:
  - Final Audit Report artifact.
- **Dependencies**: Phases 4 through 13.
- **DB/API Changes**: None.
- **Tests**: Full system verification test run.
- **Completion Criteria**: Final audit deliverable presented to user with verified operational status.

---

## Recommended First Task (To Start Phase 4 upon Approval)

**Task 4.1**: Audit and standardize environment configuration between `.env.example`, `.env`, `docker-compose.yml`, and `frontend/lib/tokens.ts`, verifying that a clean clone boots with zero missing variables or broken asset paths.
