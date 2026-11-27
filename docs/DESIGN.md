# FinSmart — Design System & Visual Guidelines (DESIGN.md)

> **Philosophy:** "Civic Ledger" — Trustworthy, calm, legible, and institutional.  
> **Anti-Pattern Guardrail:** Strictly avoids the generic "AI SaaS" look (no loud purple-to-pink gradients, no neon glow buttons, no floating 3D blobs, no robotic emoji prefixes, no all-caps micro-tracking).

---

## 1. Visual Direction & Aesthetic Identity

FinSmart is designed for the **general public, rural households, first-time earners, and students**. The visual identity draws inspiration from modern passbooks, sovereign banking portals, and clear financial ledgers.

- **Primary Metaphor**: The physical civic ledger — crisp white cards on a glare-free slate canvas, hairline dividing borders, clear tabular numbers, and dignified emerald accents.
- **Lighting & Depth**: Clean flat elevation using soft ambient box-shadows (`rgba(0, 0, 0, 0.05)`) with crisp `1px` borders. No exaggerated drop shadows or glassmorphism blurs that degrade performance on low-cost Android phones.

---

## 2. Color Palette & Accessibility Tokens

All color combinations strictly comply with **WCAG 2.1 AA / AAA** requirements (minimum contrast ratio `4.5:1` for normal text, `3:1` for large elements):

| Token Name | Hex Code | Contrast on Surface | Purpose / Semantic Role |
| :--- | :--- | :--- | :--- |
| `canvas` | `#F8FAFC` | Background | Glare-free off-white screen canvas (Slate 50). |
| `surface` | `#FFFFFF` | — | Pure white card surfaces, dialogs, and table rows. |
| `surfaceSubtle` | `#F1F5F9` | — | Table header backgrounds and chip buttons. |
| `surfaceMuted` | `#E2E8F0` | — | Subtle dividing lines and input borders. |
| `textPrimary` | `#0F172A` | **15.8:1** | High-readability deep ink for headlines and numbers. |
| `textSecondary` | `#475569` | **7.1:1** | Subtitles, labels, and form field helper text. |
| `textMuted` | `#64748B` | **4.6:1** | Secondary timestamps and table column headers. |
| `brandPrimary` | `#047857` | **4.8:1** | Forest Emerald (Civic authority, primary CTAs, active pills). |
| `brandHover` | `#065F46` | **6.1:1** | Primary CTA pressed/hover state. |
| `brandSoft` | `#ECFDF5` | Background | Light emerald tint for active navigation tabs and badges. |
| `debitRed` | `#B91C1C` | **6.5:1** | Expenses, debit transactions, overspend alerts. |
| `creditGreen` | `#047857` | **4.8:1** | Income, savings gains, milestone achievements. |
| `warningAmber` | `#B45309` | **5.2:1** | Budget threshold warnings (80%–99%), cyber fraud alerts. |

---

## 3. Typography & Vernacular Script System

The typography system is engineered to handle English, Kannada, and Hindi with equal visual hierarchy:

```css
/* English & System Fallbacks */
font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, sans-serif;

/* Kannada Fallback Stack */
font-family: "Noto Sans Kannada", "Tunga", "Kedage", Inter, sans-serif;

/* Hindi / Devanagari Fallback Stack */
font-family: "Noto Sans Devanagari", "Mangal", "Nirmala UI", Inter, sans-serif;

/* Tabular Financial Numbers */
font-variant-numeric: tabular-nums;
```

### Script Rules
1. **Zero Character-Level Splitting**: Text splitting animations (`BlurText` / split text) must strictly split by **words** (`animateBy="words"`), never by character (`splitBy="letters"`). Character splitting fragments Kannada (e.g. `ಮತ್ತೆ ಸುಸ್ವಾಗತ`) and Hindi (e.g. `वापसी`) ligatures, breaking viramas and detached matras.
2. **Tabular Numerals**: Every monetary figure uses `tabular-nums` so numbers align cleanly across ledger rows and don't induce horizontal jitter during `CountUp` animations.
3. **Hierarchy**:
   - `H1` (Screen Hero): `1.5rem` (24px), weight `700`, line-height `1.3`
   - `H2` (Card Title): `1.125rem` (18px), weight `700`, line-height `1.4`
   - `Body`: `0.9375rem` (15px), weight `400`, line-height `1.6`
   - `Caption / Meta`: `0.8125rem` (13px), weight `500`

---

## 4. Spacing, Grid & Layout Rules

FinSmart follows an 8-point geometric spatial scale:
- `xs: 4px` | `sm: 8px` | `md: 12px` | `base: 16px` | `lg: 20px` | `xl: 24px` | `2xl: 32px` | `3xl: 48px`

### Layout Principles
- **Container**: Max width `1180px`, centered with `16px` mobile gutter padding.
- **Single Section Per Screen**: Dense information is grouped into clean, self-contained cards.
- **Strict Zero Horizontal Scroll**: All layout grids use `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))` with `min-width: 0` on flex items and `overflow-x: hidden` on viewport boundaries.

---

## 5. Reusable Component Standards

### 5.1 Buttons (`Button.tsx`)
- **Minimum Touch Target**: `48px` height on mobile (`min-height: 48px; min-width: 48px`).
- **Variants**:
  - `btn-primary`: Solid forest emerald `#047857`, text white, subtle `0.2s` scale on active.
  - `btn-secondary`: White surface with `#E2E8F0` border and `#0F172A` text.
  - `btn-danger`: Red text with `#FEF2F2` background for DPDP account erasure.

### 5.2 Cards (`Card.tsx` / `SpotlightCard.tsx`)
- **Structure**: Clean header (Title + Subtitle + Action Badge), body content, optional footer divider.
- **Border Radius**: `12px` (`var(--radius-lg)`).
- **Hairline Border**: `1.5px solid var(--border-light)`.

### 5.3 Metric Cards (`StatCard.tsx`)
- 4-card responsive overview grid on Dashboard.
- Shows Label, Large Tabular Value, and Contextual Trend Subtext.

### 5.4 Form Controls & Inline Validation
- Labels always explicitly visible with `htmlFor` association (no floating placeholders).
- Inputs feature `min-height: 48px`, `border: 1.5px solid var(--border-light)`, and emerald focus rings (`outline: 2px solid #047857; outline-offset: 2px`).
- **No Native `alert()`**: Form feedback renders via `<div className="alert-banner alert-success">` or `alert-danger` with auto-dismiss timers.

---

## 6. Motion & Animation Guidelines (React Bits)

All animations adhere to the **"Calm Civic Ledger"** principle:
1. **Respect `prefers-reduced-motion`**: When enabled by the user's OS, all transitions instantly complete without movement or delay.
2. **Animate Once**: All scroll reveal elements use `viewport={{ once: true }}` to eliminate repetitive distractions.
3. **Low-End Mobile Fallbacks**:
   - Canvas `DotGrid` and cursor-following `SpotlightCard` automatically deactivate on viewports `< 640px` and touchscreen devices (`pointer: coarse`).
   - Replaced by lightweight CSS background patterns.
4. **Restrained Easing**:
   - `duration`: Fast `0.2s`, Medium `0.35s`, Slow `0.5s`.
   - `easing`: Custom civic cubic bezier `[0.16, 1, 0.3, 1]`.
5. **Budget per Screen**: Maximum of 1–2 animated elements per viewport section.
