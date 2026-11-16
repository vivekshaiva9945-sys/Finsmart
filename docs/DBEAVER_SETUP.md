# DBeaver Database Connection & Verification Guide

This guide provides step-by-step instructions to connect **DBeaver** to the **FinSmart PostgreSQL 16 (pgvector)** database running in Docker.

---

## 1. Connection Parameters (from `.env`)

The PostgreSQL container exposes port `5432` on `localhost`.

### Profile A: Database Administrator (Full Access / Superuser)
*Use this connection to inspect all schemas, browse cross-tenant data, modify tables, and manage extensions.*

| Setting | Value | Notes |
|---|---|---|
| **Database Type** | `PostgreSQL` | Standard PostgreSQL driver |
| **Host** | `localhost` or `127.0.0.1` | Host machine port forwarding |
| **Port** | `5433` (or `5432` if set in `.env`) | Host machine port forwarding (`DB_PORT` in `.env`) |
| **Database** | `finsmart_db` | From `POSTGRES_DB` in `.env` |
| **Username** | `finsmart_admin` | From `POSTGRES_USER` in `.env` |
| **Password** | `change_me_admin` | From `POSTGRES_PASSWORD` in `.env` |

---

### Profile B: Application Role (RLS Enforced / Non-Superuser)
*Use this connection to test and verify Row-Level Security (RLS) tenant isolation.*

| Setting | Value | Notes |
|---|---|---|
| **Host** | `localhost` | |
| **Port** | `5433` (or `5432`) | From `DB_PORT` in `.env` |
| **Database** | `finsmart_db` | |
| **Username** | `finsmart_app` | From `APP_DB_USER` in `.env` |
| **Password** | `change_me_app` | From `APP_DB_PASSWORD` in `.env` |

---

## 2. Step-by-Step DBeaver Setup

1. Open **DBeaver Community** or **DBeaver PRO**.
2. Click **Database** → **New Database Connection** (or the plug icon in the top left toolbar).
3. In the "Connect to a database" dialog, select **PostgreSQL** and click **Next**.
4. In the **Main** tab, enter the connection details:
   - **Host:** `localhost`
   - **Port:** `5432`
   - **Database:** `finsmart_db`
   - **Authentication:** `Database Native`
   - **Username:** `finsmart_admin`
   - **Password:** `change_me_admin`
5. Click **Test Connection ...**:
   - *Note:* If this is your first time connecting to PostgreSQL in DBeaver, DBeaver will prompt you to download the **PostgreSQL JDBC Driver**. Click **Download**.
   - You should see: `Connected! PostgreSQL 16.x (pgvector)`.
6. Click **Finish**.
7. In the Database Navigator on the left, expand:
   `finsmart_db` → `Databases` → `finsmart_db` → `Schemas` → `public` → `Tables`.

---

## 3. Ready-to-Run DBeaver SQL Verification Queries

Open a SQL Editor in DBeaver (**SQL Editor** → **New SQL Script**, or press `Ctrl + ]`) and run the following queries:

### Query 1: Verify All Initialized Tables
```sql
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public' 
ORDER BY table_name;
```
*Expected Tables:*
- `accounts`
- `budgets`
- `consents`
- `goals`
- `knowledge_docs`
- `learning_modules`
- `schemes`
- `tenants`
- `transactions`
- `user_profiles`
- `users`

---

### Query 2: Inspect Pre-Seeded Tenants
```sql
SELECT id, slug, name, domain, is_active, created_at 
FROM tenants;
```
*Expected Rows:*
- `navkis`: Navkis College of Engineering, Hassan
- `rvce`: RV College of Engineering, Bengaluru
- `public`: FinSmart Public Financial Literacy Portal

---

### Query 3: Inspect Curated Government Schemes & JSON Rules
```sql
SELECT code, title_en, agency, category, source_url, rule 
FROM schemes 
ORDER BY code;
```
*Notice:* The `rule` column contains structured JSON evaluated by FinSmart's pure declarative rules engine:
```json
{
  "all": [
    {"field": "is_student", "op": "eq", "value": true, "label": "Currently an enrolled student"},
    {"field": "annual_family_income", "op": "lte", "value": 250000, "label": "Family income ceiling"}
  ]
}
```

---

### Query 4: Inspect Multilingual Learning Modules & Quizzes
```sql
SELECT id, lang, category, title, quiz 
FROM learning_modules 
ORDER BY lang, id;
```

---

### Query 5: Verify Row-Level Security (RLS) Isolation
To verify that the application user cannot read across tenant boundaries:

1. Connect as `finsmart_app` (or switch connection).
2. Run a query without setting the tenant context:
```sql
-- Returns 0 rows because app.current_tenant_id is not set:
SELECT count(*) FROM transactions;
```

3. Now simulate an authenticated request for the **Navkis College** tenant:
```sql
BEGIN;
-- Set transaction-local tenant context to Navkis tenant UUID:
SELECT set_config('app.current_tenant_id', (SELECT id::text FROM tenants WHERE slug = 'navkis'), true);

-- Now records belonging only to Navkis are visible:
SELECT id, amount_minor, kind, category, occurred_on FROM transactions;
COMMIT;
```

---

## 4. Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| **Connection Refused (`localhost:5432`)** | Docker container is not running | Run `docker compose up -d database` and wait until healthy. |
| **Password authentication failed for user** | Mismatched credentials in `.env` | Ensure `.env` has `POSTGRES_USER=finsmart_admin` and `POSTGRES_PASSWORD=change_me_admin`. |
| **Driver download prompt fails** | No internet connection in DBeaver | Ensure host machine has internet access to allow DBeaver to fetch `postgresql.jar`. |
| **Permission denied on table** | Connected as `finsmart_app` without table grants | Connect as superuser `finsmart_admin` or re-run `docker/02-schema.sql`. |

