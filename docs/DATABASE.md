# FinSmart — Database Schema & Data Dictionary (DATABASE.md)

> **Document Version:** 1.0.0  
> **Engine:** PostgreSQL 16 with `pgvector`  
> **Isolation Model:** Multi-Tenant Row-Level Security (RLS)  
> **Standard Compliance:** DPDP Act 2023, RBI Guidelines, Minor-Unit Currency Accounting

---

## 1. Database Users & Roles

FinSmart strictly enforces security separation by maintaining two distinct database roles:

| Role Name | Superuser? | Bypass RLS? | Purpose |
| :--- | :--- | :--- | :--- |
| `finsmart_admin` | **YES** | **YES** | Bootstrapping, running DDL migrations, superuser maintenance, DBeaver admin inspection. |
| `finsmart_app` | **NO** | **NO** | Runtime application connection used by FastAPI. Strictly subject to Row-Level Security. |

---

## 2. Entity-Relationship Summary

```
tenants (1) ──────────< users (N)
   │                       │
   │ (1)                   │ (1)
   ▼                       ▼
user_profiles (N)       accounts (N)
   │                       │
   │ (1)                   │ (1)
   ▼                       ▼
budgets (N)             transactions (N)
   │
   │ (1)
   ▼
goals (N) ─── consents (N)
```

---

## 3. Data Dictionary

### 3.1 `tenants` (Colleges, Universities & Public Portal)
Partitioning boundary for multi-institutional data isolation.
- `id` (UUID, Primary Key, `gen_random_uuid()`)
- `slug` (TEXT, Unique, Not Null) — e.g. `'public'`, `'navkis'`, `'rvce'`.
- `name` (TEXT, Not Null) — Human-readable institution name.
- `domain` (TEXT, Nullable) — e.g. `'navkis.edu.in'`.
- `subscription_tier` (TEXT, Not Null, Default `'standard'`).
- `is_active` (BOOLEAN, Not Null, Default `true`).
- `created_at` / `updated_at` (TIMESTAMPTZ, Not Null, Default `now()`).

### 3.2 `users` (Authentication & Access)
- `id` (UUID, Primary Key, `gen_random_uuid()`)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`, ON DELETE CASCADE, Not Null)
- `email` (TEXT, Not Null)
- `password_hash` (TEXT, Not Null) — Argon2id or Bcrypt hash.
- `full_name` (TEXT, Not Null)
- `phone_number` (TEXT, Nullable)
- `role` (TEXT, Not Null, Default `'student'`) — Check: `IN ('student', 'faculty', 'college_admin', 'admin')`.
- `is_active` (BOOLEAN, Not Null, Default `true`).
- `created_at` / `updated_at` (TIMESTAMPTZ, Not Null, Default `now()`).
- **Constraint**: `UNIQUE (tenant_id, email)` — Allows same email across different institutional tenants if desired.

### 3.3 `user_profiles` (Demographic Profile for Welfare Matching)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`, Unique, Not Null)
- `age` (INTEGER) — Check: `0 <= age <= 120`.
- `gender` (TEXT) — Check: `IN ('male', 'female', 'other', 'prefer_not_to_say')`.
- `state` (TEXT, Default `'Karnataka'`).
- `district` (TEXT, Nullable).
- `is_student` (BOOLEAN, Default `true`).
- `academic_year` / `department` (TEXT, Nullable).
- `annual_family_income_minor` (BIGINT, Not Null, Default `0`) — Stored in integer paise.
- `social_category` (TEXT, Default `'GEN'`) — Check: `IN ('GEN', 'OBC', 'SC', 'ST', 'EWS')`.
- `has_bank_account` (BOOLEAN, Default `true`).
- `has_disability` (BOOLEAN, Default `false`).
- `ration_card_type` (TEXT, Default `'None'`) — Check: `IN ('BPL', 'APL', 'AAY', 'None')`.

### 3.4 `accounts` (Financial Accounts)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`)
- `name` (TEXT, Not Null) — e.g. `'State Bank of India'`, `'Cash Wallet'`.
- `account_type` (TEXT, Default `'BANK'`) — Check: `IN ('BANK', 'CASH', 'WALLET', 'SAVINGS')`.
- `balance_minor` (BIGINT, Not Null, Default `0`) — Integer paise.
- `currency` (CHAR(3), Default `'INR'`).
- `is_active` (BOOLEAN, Default `true`).

### 3.5 `transactions` (Ledger Entries)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`)
- `account_id` (UUID, Foreign Key -> `accounts.id`, ON DELETE SET NULL)
- `client_id` (UUID, Not Null) — Idempotency key generated on client.
- `amount_minor` (BIGINT, Not Null, Check `amount_minor > 0`) — Integer paise.
- `currency` (CHAR(3), Default `'INR'`).
- `kind` (TEXT, Not Null, Check `kind IN ('DEBIT', 'CREDIT')`).
- `category` (TEXT, Not Null) — Normalized lowercase string.
- `payment_mode` (TEXT, Default `'UPI'`) — Check: `IN ('CASH', 'UPI', 'CARD', 'BANK', 'WALLET')`.
- `description` (TEXT, Default `''`).
- `occurred_on` (DATE, Default `CURRENT_DATE`).
- `created_at` (TIMESTAMPTZ, Default `now()`).
- **Constraint**: `UNIQUE (user_id, client_id)` — Prevents duplicate inserts on retries.
- **Indexes**:
  - `idx_transactions_user_date` ON `transactions(user_id, occurred_on DESC)`.
  - `idx_transactions_tenant_user` ON `transactions(tenant_id, user_id)`.

### 3.6 `budgets` (Monthly Category Allocations)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`)
- `category` (TEXT, Not Null)
- `limit_minor` (BIGINT, Not Null, Check `limit_minor > 0`)
- `period` (TEXT, Default `'MONTHLY'`) — Check: `IN ('MONTHLY', 'SEMESTER', 'YEARLY')`.
- **Constraint**: `UNIQUE (user_id, category, period)`.

### 3.7 `goals` (Savings Milestones)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`)
- `name` (TEXT, Not Null)
- `target_minor` (BIGINT, Not Null, Check `target_minor > 0`)
- `saved_minor` (BIGINT, Not Null, Default `0`)
- `target_date` (DATE, Nullable)
- `category` (TEXT, Default `'General'`)
- `is_completed` (BOOLEAN, Default `false`)

### 3.8 `consents` (DPDP Act 2023 Consent Ledger)
- `id` (UUID, Primary Key)
- `tenant_id` (UUID, Foreign Key -> `tenants.id`)
- `user_id` (UUID, Foreign Key -> `users.id`)
- `purpose` (TEXT, Not Null)
- `granted` (BOOLEAN, Not Null)
- `at` (TIMESTAMPTZ, Default `now()`)

### 3.9 `schemes` (Government Welfare Catalog & Rule DSL)
- `code` (TEXT, Primary Key) — e.g. `'PMJDY'`, `'APY'`, `'NSP-POSTMATRIC'`.
- `title_en` / `title_kn` / `title_hi` (TEXT) — Multilingual titles.
- `agency` / `category` / `summary` / `benefits` / `source_url` (TEXT).
- `documents_required` / `application_steps` (TEXT[]).
- `last_verified_at` (DATE).
- `rule` (JSONB, Not Null) — Declarative evaluation DSL.

### 3.10 `learning_modules` (Multilingual Financial Education)
- `id` (SERIAL, Primary Key)
- `lang` (TEXT, Check `lang IN ('en', 'kn', 'hi')`)
- `category` / `title` / `summary` / `body` (TEXT)
- `quiz` (JSONB, Default `'[]'::jsonb`)

### 3.11 `knowledge_docs` (RAG AI Knowledge Chunks)
- `id` (SERIAL, Primary Key)
- `title` / `category` / `content` / `source_citation` (TEXT)
- `embedding` (vector(384)) — For semantic embedding similarity matching.

---

## 4. Row-Level Security (RLS) Policies

All user-specific tables (`users`, `user_profiles`, `accounts`, `transactions`, `budgets`, `goals`, `consents`) have RLS forced:

```sql
ALTER TABLE <table> ENABLE ROW LEVEL SECURITY;
ALTER TABLE <table> FORCE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON <table>
  FOR ALL
  USING (tenant_id = NULLIF(current_setting('app.current_tenant_id', true), '')::uuid)
  WITH CHECK (tenant_id = NULLIF(current_setting('app.current_tenant_id', true), '')::uuid);
```

---

## 5. Backup & Recovery Protocol

- **Automated Dump**: `pg_dump -U finsmart_admin -d finsmart_db -F c -b -v -f /backup/finsmart_$(date +%Y%m%d).dump`
- **Zero-Loss Point-In-Time Recovery**: Achieved through PostgreSQL Write-Ahead Logging (WAL) archiving configured on production hosts.
