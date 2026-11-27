# FinSmart - Production Cloud Deployment & Integration Guide
**Supabase (Database) + Render (FastAPI Backend) + Vercel (Next.js Frontend)**

This document provides the canonical, step-by-step production deployment and integration guide for the FinSmart application across **Supabase**, **Render**, and **Vercel**.

---

## 1. Architecture & Network Topology

```
┌─────────────────────────────────┐
│       Vercel (Next.js 14)       │
│   https://finsmart.vercel.app   │
└────────────────┬────────────────┘
                 │
                 │ HTTPS REST (Bearer JWT + X-Tenant-ID Header)
                 ▼
┌─────────────────────────────────┐
│       Render (FastAPI API)      │
│  https://finsmart.onrender.com  │
│  Port: $PORT, Python 3.12       │
└────────────────┬────────────────┘
                 │
                 │ PostgreSQL SSL (asyncpg Connection Pooler)
                 ▼
┌─────────────────────────────────┐
│      Supabase (PostgreSQL 16)   │
│  Row-Level Security (RLS) Active│
│  Integer Paise (BIGINT)         │
└─────────────────────────────────┘
```

---

## 2. Step 1: Database Deployment on Supabase

### 2.1 Create Supabase Project
1. Log in to [Supabase Console](https://supabase.com).
2. Click **New project**, select your organization, and provide:
   - **Name**: `finsmart-production`
   - **Database Password**: Choose a strong password (save this securely).
   - **Region**: `ap-south-1` (Mumbai / India) for lowest latency to Indian users.
3. Once the database is provisioned, go to **Project Settings** → **Database**.

### 2.2 Schema & Seed Initialization (2 Options)

#### Option A: 1-Click Supabase SQL Editor (Recommended)
1. In your Supabase Dashboard, click on **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Copy the entire contents of [`backend/supabase_schema.sql`](file:///c:/Users/VIVEK%20I%20H/Downloads/finsmart/backend/supabase_schema.sql).
4. Paste into the SQL editor and click **Run**.
5. All 11 tables (`tenants`, `users`, `user_profiles`, `accounts`, `transactions`, `budgets`, `goals`, `consents`, `schemes`, `learning_modules`, `knowledge_docs`), Row-Level Security policies, and initial seeds (`navkis`, `rvce`, `public`, welfare schemes, multilingual literacy modules) will be created immediately.

#### Option B: Automatic Alembic Migration from CLI
If you have local access to the Supabase URI:
```bash
cd backend
export DATABASE_URL="postgresql+asyncpg://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?ssl=require"
alembic upgrade head
python seed.py
```

### 2.3 Supabase Connection String Format
In **Project Settings** → **Database** → **Connection String** → **URI**:
- Direct URI: `postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres`
- *Note:* FinSmart's `Settings` automatically converts `postgres://` or `postgresql://` into `postgresql+asyncpg://` and standardizes `sslmode=require` to `ssl=require`.

---

## 3. Step 2: Backend Deployment on Render

### 3.1 Deploy via Render Dashboard
1. Log in to [Render Dashboard](https://dashboard.render.com).
2. Click **New +** → **Web Service**.
3. Connect your backend GitHub repository: `https://github.com/vivekshaiva9945-sys/FinsmartBackend.git`.
4. Configure service settings:
   - **Name**: `finsmart-backend`
   - **Region**: `Singapore` (closest to India on Render free tier).
   - **Branch**: `main`
   - **Root Directory**: `.` (leave empty or root)
   - **Runtime**: `Python 3`
   - **Build Command**: `pip install -r requirements.txt && alembic upgrade head && python seed.py`
   - **Start Command**: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
   - **Plan**: `Free`

### 3.2 Environment Variables on Render
Under the **Environment** tab, add the following variables:

| Key | Value | Description |
| :--- | :--- | :--- |
| `PYTHON_VERSION` | `3.12.2` | Python runtime version |
| `ENVIRONMENT` | `production` | Enables HTTPS enforcement & security validators |
| `DATABASE_URL` | `postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres` | Your Supabase connection string |
| `JWT_SECRET` | Click *Generate* (min 32 characters) | HMAC-SHA256 signature key |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `120` | Session lifetime |
| `CORS_ORIGINS` | `http://localhost:3000,https://finsmart.vercel.app,https://[your-vercel-app].vercel.app` | Allowed frontend origins (comma-separated) |

### 3.3 Health Check Configuration
- Under **Advanced Settings** → **Health Check Path**, enter: `/v1/health`
- Render will ping `/v1/health` to confirm zero-downtime deployment before routing traffic.

---

## 4. Step 3: Frontend Deployment on Vercel

### 4.1 Deploy via Vercel Dashboard
1. Log in to [Vercel](https://vercel.com).
2. Click **Add New...** → **Project**.
3. Import your frontend repository: `https://github.com/vivekshaiva9945-sys/Finsmartfrontend.git`.
4. Configure project settings:
   - **Framework Preset**: `Next.js`
   - **Root Directory**: `./`
   - **Build Command**: `npm run build`
   - **Output Directory**: `.next`

### 4.2 Environment Variables on Vercel
Add the following production environment variables:

| Key | Value | Description |
| :--- | :--- | :--- |
| `NEXT_PUBLIC_API_URL` | `https://finsmart-backend.onrender.com/v1` | Your live Render backend URL (must end in `/v1`) |
| `NEXT_PUBLIC_TENANT` | `public` | Default tenant fallback |
| `NEXT_PUBLIC_ANALYTICS_ID` | `G-XXXXXXXXXX` *(optional)* | Google Analytics ID (only active if user accepts cookies) |

5. Click **Deploy**. Vercel will build and assign your production URL (e.g. `https://finsmart.vercel.app`).

### 4.3 Update Backend CORS on Render
Once you have your live Vercel URL:
1. Return to **Render** → `finsmart-backend` → **Environment**.
2. Update `CORS_ORIGINS` to include your exact Vercel URL:
   ```
   http://localhost:3000,https://finsmart.vercel.app,https://your-custom-domain.com
   ```
3. Save changes (Render will automatically redeploy with the new CORS policy).

---

## 5. Step 4: Verification & Integration Testing

Verify all layers with the following smoke tests:

### 1. Backend & Database Health Check
```bash
curl -i https://finsmart-backend.onrender.com/v1/health
```
**Expected Response (HTTP 200)**:
```json
{"status":"healthy","service":"FinSmart API","version":"1.0.0","environment":"production"}
```

### 2. Multi-Tenant User Registration Test
```bash
curl -X POST https://finsmart-backend.onrender.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: public" \
  -d '{
    "email": "testuser@finsmart.org",
    "password": "StrongPassword123!",
    "full_name": "Test User",
    "consent_data_processing": true
  }'
```
**Expected Response (HTTP 201)**:
Returns JWT `access_token` and user session info.

### 3. Scheme Matching & JSON Rule Engine Test
```bash
curl -X POST https://finsmart-backend.onrender.com/v1/schemes/eligibility \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: public" \
  -d '{
    "age": 20,
    "is_student": true,
    "annual_family_income": 150000,
    "social_category": "OBC",
    "has_bank_account": true,
    "state": "Karnataka"
  }'
```
**Expected Response (HTTP 200)**:
Returns deterministic eligibility results for `NSP-POSTMATRIC` and `KAR-SSP`.

### 4. Grounded AI Financial Assistant Test
```bash
curl -X POST https://finsmart-backend.onrender.com/v1/ai/chat \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: public" \
  -d '{
    "message": "What is the 50/30/20 budgeting rule for college students?",
    "language": "en"
  }'
```
**Expected Response (HTTP 200)**:
Returns grounded financial explanation with citations and zero speculative trading advice.

---

## 6. Step 5: Monitoring, Uptime & Maintenance

### 6.1 Render Free Tier Spin-Down Prevention
Render free web services spin down after 15 minutes of inactivity.
- **Solution**: Set up a free monitor on [UptimeRobot](https://uptimerobot.com) or [BetterStack](https://betterstack.com):
  - **Monitor Type**: `HTTP(s)`
  - **URL**: `https://finsmart-backend.onrender.com/v1/health`
  - **Monitoring Interval**: Every `10 minutes`
  - This keeps the Render backend warm and eliminates cold starts for users.

### 6.2 Supabase Backup & Recovery
- Supabase automatically maintains continuous WAL logs and daily automated snapshots.
- To export a manual offline backup at any time:
  ```bash
  pg_dump -h db.[PROJECT-REF].supabase.co -U postgres -d postgres -F c -b -v -f finsmart_backup.dump
  ```

### 6.3 Security & Privacy Guidelines for Logs
- Never log passwords, tokens, Aadhaar, PAN, or financial numbers in plaintext.
- The PII scrubber in `app/core/pii.py` automatically scrubs Indian identifiers before AI processing.
