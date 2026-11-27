# FinSmart — RESTful API Specification (API.md)

> **API Version:** v1  
> **Base URL:** `http://localhost:8000/v1` (or `${NEXT_PUBLIC_API_URL}`)  
> **Authentication:** Bearer JWT in `Authorization` header  
> **Multi-Tenant Context:** `X-Tenant-ID` header (defaults to `'public'`)  
> **Currency Unit Convention:** All monetary amounts are integer **paise** (`100` paise = ₹1.00)

---

## 1. Global Request Headers & Conventions

Every API request from the frontend must include:

| Header Name | Type | Required? | Example / Description |
| :--- | :--- | :--- | :--- |
| `Content-Type` | string | For POST/PUT | `application/json` |
| `X-Tenant-ID` | string | **YES** | `public`, `navkis`, `rvce`. Identifies institutional tenant boundary. |
| `Authorization` | string | Protected APIs | `Bearer <JWT_TOKEN>` |

---

## 2. Standard Error Format

All error responses strictly return RFC 7807 compliant JSON:

```json
{
  "detail": "Descriptive human-readable error message or validation errors array."
}
```

### Standard Status Codes
- `200 OK`: Request succeeded, response body returned.
- `201 Created`: Resource successfully created.
- `204 No Content`: Resource successfully deleted / modified with empty body.
- `400 Bad Request`: Validation failure (e.g. invalid date, negative amount).
- `401 Unauthorized`: Token missing, expired, or tenant mismatch.
- `403 Forbidden`: Insufficient role permissions (e.g. student calling `/admin/stats`).
- `404 Not Found`: Tenant or resource does not exist.
- `429 Too Many Requests`: Rate limit threshold exceeded.

---

## 3. Endpoints Directory

### 3.1 Authentication (`/auth`)

#### `POST /auth/register`
Creates a user account within the active tenant.
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "SecurePassword123!",
    "full_name": "Ramesh Kumar",
    "role": "student"
  }
  ```
- **Response** (`201 Created`):
  ```json
  {
    "access_token": "eyJhbGciOi...",
    "token_type": "bearer",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "full_name": "Ramesh Kumar",
      "role": "student",
      "tenant_slug": "public"
    }
  }
  ```

#### `POST /auth/login`
Authenticates user credentials against the specified tenant.
- **Request Body**: `{"email": "...", "password": "..."}`
- **Response** (`200 OK`): Same token payload as register.

#### `GET /auth/me`
Returns current session profile. Requires Bearer JWT.

---

### 3.2 Household Ledger & Transactions (`/transactions`)

#### `GET /transactions/?limit=50&offset=0&kind=DEBIT`
Lists user transactions with pagination and optional filters (`kind`, `category`).

#### `POST /transactions/`
Records an income or expense transaction. Idempotent via `client_id`.
- **Request Body**:
  ```json
  {
    "client_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "amount_minor": 8000,
    "kind": "DEBIT",
    "category": "food",
    "payment_mode": "UPI",
    "description": "Vegetables & Milk",
    "occurred_on": "2026-11-27"
  }
  ```

#### `GET /transactions/summary`
Calculates automated monthly cashflow metrics:
- **Response** (`200 OK`):
  ```json
  {
    "total_income_minor": 2500000,
    "total_expense_minor": 1800000,
    "balance_minor": 700000,
    "savings_rate_pct": 28.0,
    "month_str": "November 2026"
  }
  ```

#### `DELETE /transactions/{id}`
Deletes a specific transaction from the ledger. Returns `204 No Content`.

---

### 3.3 Budgets (`/budgets`)

#### `PUT /budgets/`
Sets or updates a monthly category budget limit.
- **Request Body**: `{"category": "food", "limit_minor": 500000, "period": "MONTHLY"}`

#### `GET /budgets/status`
Retrieves all active budgets with spending progress and safe daily allowances:
- **Response** (`200 OK`):
  ```json
  [
    {
      "category": "food",
      "limit_minor": 500000,
      "spent_minor": 320000,
      "remaining_minor": 180000,
      "percent": 64,
      "level": "ok"
    }
  ]
  ```

---

### 3.4 Savings Goals (`/goals`)

#### `POST /goals/`
Creates a savings goal.
- **Request Body**: `{"name": "Laptop", "target_minor": 4500000, "target_date": "2027-06-30", "category": "Gadgets"}`

#### `POST /goals/{id}/contribute`
Contributes funds to a goal milestone.
- **Request Body**: `{"amount_minor": 100000}`

---

### 3.5 Welfare Schemes & Eligibility (`/schemes`)

#### `POST /schemes/eligibility`
Evaluates demographic profile against deterministic JSON Rule DSL.
- **Request Body**:
  ```json
  {
    "age": 20,
    "is_student": true,
    "annual_family_income": 180000,
    "social_category": "OBC",
    "state": "Karnataka",
    "has_bank_account": true,
    "has_disability": false
  }
  ```
- **Response** (`200 OK`): List of schemes with `eligible: boolean` and `reasons: string[]`.

---

### 3.6 AI Financial Mentor (`/ai/chat`)

#### `POST /ai/chat`
RAG-grounded advisory endpoint with automated PII scrubbing and safety guardrails.
- **Request Body**: `{"message": "How do I start an emergency fund?", "language": "kn"}`
- **Response** (`200 OK`):
  ```json
  {
    "reply": "ತುರ್ತು ನಿಧಿಯು...",
    "citations": [{"title": "RBI Guidelines", "source": "rbi.org.in"}],
    "pii_scrubbed": false
  }
  ```

---

### 3.7 Financial Precision Calculators (`/calc`)

- `POST /calc/emi`: Computes monthly EMI, interest, and total payable.
- `POST /calc/sip`: Computes invested principal, compound gain, and future value.
- `POST /calc/emergency-fund`: Sizes mandatory 3–6 month liquid survival buffers.

---

### 3.8 Multilingual Financial Literacy (`/learning`)

- `GET /learning/?lang=kn`: Retrieves curriculum modules in English, Kannada, or Hindi.
- `POST /learning/quiz/submit`: Validates interactive knowledge check answers with immediate educational feedback.

---

### 3.9 DPDP Act 2023 Privacy & Profile (`/profiles`)

- `GET /profiles/`: Retrieves demographic profile.
- `PUT /profiles/`: Updates demographic profile.
- `GET /profiles/dsar-export`: Generates downloadable Data Subject Access Request JSON file.
- `DELETE /profiles/erasure`: Triggers irreversible Right to be Forgotten account erasure.

---

### 3.10 Institution Admin Analytics (`/admin/stats`)
Accessible only to `college_admin` and `admin` roles. Returns aggregate, zero-PII counts (`total_users`, `total_transactions`, `total_volume_minor`, `total_goals`).
