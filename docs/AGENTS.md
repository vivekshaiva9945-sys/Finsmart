# FinSmart — Engineering & Coding Discipline for Agents (AGENTS.md)

> **Audience:** Human developers and Autonomous AI Agents collaborating on the FinSmart codebase.  
> **Mandate:** Preserve simplicity, security, accessibility, Indic language safety, and containerized stability.

---

## 1. Prime Invariants

1. **Never Compromise Multi-Tenant RLS**:
   - Every user query touching `users`, `accounts`, `transactions`, `budgets`, `goals`, `user_profiles`, or `consents` must run within a session bound to `app.current_tenant_id`.
   - Never use the PostgreSQL superuser (`finsmart_admin`) for application API routes. The application connection string must always use `finsmart_app`.
2. **Never Break Money Math**:
   - Currency stored in the database must always be integer **paise** (`BIGINT`).
   - Never store floats or perform floating-point divisions directly on currency values in the database or backend. Use `app/core/money.py` with `Decimal` and `ROUND_HALF_UP`.
3. **Never Break Indic Script Typography**:
   - Text animation components (`BlurText`, split text) must strictly split by **words** (`animateBy="words"`).
   - Never split Kannada or Hindi strings by character or letter (`splitBy="letters"`). Doing so fractures complex Brahmic grapheme clusters (consonants + viramas + matras) into illegible, detached glyphs.
4. **Never Hardcode or Leak Secrets**:
   - Client-side code (`frontend/`) must never reference database passwords, secret keys, or private API tokens.
   - All server configuration must resolve via `backend/app/core/config.py` using `BaseSettings`.
5. **No Disruptive AI Tells**:
   - Avoid generic AI-generated SaaS aesthetics: no neon glow boxes, no loud purple gradients, no robot emojis, and no browser-native `alert()` popups. Use inline accessible banners (`<div className="alert-banner">`).

---

## 2. Code Quality & Architecture Rules

### 2.1 Backend (FastAPI & Python 3.12)
- **Typing**: Use modern Python 3.12 syntax (`list[str]`, `dict[str, Any]`, `X | None`).
- **Validation**: All endpoint payloads must be validated using strict Pydantic v2 schemas in `app/schemas/`.
- **SQL Execution**: Use parameterized queries via SQLAlchemy `text()` with named parameters (`:u`, `:t`, `:id`) to guarantee 100% SQL injection prevention.
- **Dependencies**: Keep external dependencies minimal. Do not add heavy ORMs or unneeded packages without prior architectural justification.

### 2.2 Frontend (Next.js 14 App Router & TypeScript)
- **Component Reuse**: Check `components/ui/` and `components/react-bits/` before creating a new component.
- **Mobile First & Zero Overflow**: Ensure every view is verified for mobile responsiveness. Flex items must include `min-width: 0` to prevent horizontal overflow on long strings.
- **Accessibility (a11y)**:
  - Form inputs must have visible, associated `<label>` tags.
  - Interactive icons must have `aria-hidden="true"` or accessible button labels.
  - Maintain WCAG AA contrast ratio (`≥ 4.5:1`).
  - Always respect `prefers-reduced-motion`.

---

## 3. Error Handling & Validation Discipline

1. **Client-Side**:
   - Catch errors gracefully and render user-friendly inline error alerts.
   - Do not use `alert()` or `confirm()` for normal form validation.
2. **Server-Side**:
   - Raise `HTTPException` with explicit status codes and informative `detail` strings.
   - Catch database errors and return meaningful 400/404/422 responses rather than unhandled 500 crashes.

---

## 4. Git & Verification Discipline

1. **Small, Atomic Commits**: Commit one logical change at a time with a descriptive conventional commit title (`feat(ui): ...`, `fix(api): ...`, `chore: ...`).
2. **Verification Before Push**:
   - Run backend test suite: `python -m unittest discover -s tests` (must pass 100%).
   - Run frontend TypeScript typecheck: `npx tsc --noEmit` (must exit code 0).
   - Test mobile viewport layout (`412px` width) for 0px horizontal overflow.
3. **Never Bypass Checks**: Never weaken validation or delete tests to force passing builds.

---

## 5. Architectural Discrepancies & Flags in Existing Codebase

Where existing implementation differs from target ideals:
1. **Empty ORM Models Folder**: `backend/app/models/__init__.py` is currently empty because endpoints query via parameterized SQL `text(...)`. Future work in Phase 5 may introduce declarative models or keep this explicit raw SQL pattern if approved.
2. **Session Storage vs Local Storage**: Currently, JWT tokens reside in `sessionStorage` (`lib/api.ts`). In Phase 6, we will align this with the user-approved decision regarding persistent mobile logins.
3. **Alembic Versioning**: Currently, schema changes are defined in `docker/02-schema.sql`. In Phase 5, Alembic will be introduced for migration tracking.
