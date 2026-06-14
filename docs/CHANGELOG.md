# Nivo — Changelog

Append-only. Each AI Tech Manager run appends a dated entry.

---

## 2026-06-14 — Run 1 (initial docs + v2 polish)

### Summary
First documented run. Created entire `docs/` vault from scratch. Delivered four improvements to `index.html`.

### Audit
- Live site (Vercel) returns title "Spendly — AI Expense Tracker" — old CDN cache, no code fix needed. Current `index.html` correctly brands as "Nivo — Smart Expense Tracker".
- `docs/` directory did not exist — created all five docs this run.
- Figma MCP: not available this run — proceeded from Apple HIG.

### Design (Task 2) — Frosted-Glass Nav
- **iOS 26 spirit**: sidebar and header now use `.glass-nav` CSS class: `background: rgba(255,255,255,0.82); backdrop-filter: saturate(180%) blur(20px)`.
- Border opacity reduced to `border-slate-200/60` for layered depth.
- Stat card value font bumped from `text-2xl` to `text-3xl` for better hierarchy.
- Active nav item uses `rounded-xl` (up from `rounded-lg`) + `shadow-sm` for pill-style iOS feel.
- Recorded in `DESIGN.md`.

### Feature (Task 3) — Month-end Spend Forecast
- New card at top of **Insights** page: "Month-end Forecast".
- Computes `projectedMonthEnd = (totalSpend / activeDays) * daysInMonth`.
- Shows: projected total (large), progress bar (% of month elapsed), projected remaining, daily average.
- Labeled as heuristic in UI with TODO comment for Supabase Edge Function upgrade.
- Gradient violet card with ✨ AI badge.

### AI Advancement (Task 4) — Confidence Scoring
- Added `aiCategorizeFull()` returning `{category, confidence, method}`.
- Confidence scale: 99 (user rule), 93 (exact brand), 78–96 (pattern), 30 (fallback).
- `ConfBadge` component shows amber/red pill next to category dropdown only when confidence <80%.
- `seedTxns()` now stores `catConf` and `catMethod` on every transaction.
- `reCat()` sets `catConf=100, catMethod='user'` when user corrects — teaches the system.
- `addTxn()` uses `aiCategorizeFull` and stores confidence on new transactions.

### Hardening (Task 5) — localStorage + a11y
- `LS.set()` catches `QuotaExceededError` (code 22) with `console.warn` instead of silent fail.
- `LS_VERSION=2` with migration: stale v1 `txns`/`confirmed` wiped on first v2 load.
- `INR(n)` guards `n||0` to prevent NaN display.
- `totalSpend` / `naiveSpend` / `income` use `(t.amount||0)` guard.
- `addTxn` validates `amt > 0` before inserting; `amount: Math.max(0, parseFloat(...)||0)`.
- Budget input uses `Math.max(0, ...)` guard.
- Accessibility: `aria-label` on all inputs, `aria-current="page"` on active nav, `role="navigation"`, `role="dialog"` + `aria-modal` on modal, `role="table"` + `scope="col"` on table, `aria-pressed` + `aria-label` on 2FA toggle, `role="alert"` on error messages, `aria-hidden` on decorative QR grid.
- `URL.revokeObjectURL(u)` after data export (memory leak fix).
- `Math.max(0, ...)` on "Biggest expense" Stat to avoid `-Infinity` when no transactions.

### Docs Created
- `docs/NIVO_CONTEXT.md` — architecture, conventions, known issues
- `docs/ROADMAP.md` — P0/P1/P2 backlog
- `docs/AI_PLAN.md` — AI roadmap with implementation plans
- `docs/CHANGELOG.md` — this file
- `docs/DESIGN.md` — design system tokens

---

## 2026-06-14 — Run 2 (recurring detection + anomaly + card depth + Donut hardening)

### Summary
Four improvements shipped in one commit: refined card shadow depth system (design), recurring subscription detection (R-01 feature), anomaly detection flag in transaction list (AI step d+e), Donut chart hardened for empty/single-category edge cases.

### Audit
- Live site still returns old "Spendly" title (Vercel cache) — no fix needed, code is correct "Nivo — Smart Expense Tracker".
- All 5 docs present and up to date.
- Figma MCP: not available this run — proceeded from Apple HIG.
- No regressions detected. `index.html` compiles cleanly (Babel standalone JSX).

### Design (Task 2) — Refined Card Shadow System (Apple HIG Depth Tokens)
- Replaced Tailwind `shadow-sm` on `Card` with `.card-shadow` CSS class.
- Hover state with `transition: box-shadow 0.18s ease`.
- Added `.btn-press` CSS class: iOS tactile press feedback (`scale(0.97)` on active).
- Applied `btn-press` to all buttons and clickable cards.
- Updated Bars chart bars to use `transition-all duration-500`.
- Recorded in `DESIGN.md`.

### Feature (Task 3) — Recurring Subscription Detection (R-01 ✅)
- New `detectRecurring(txns)` function: groups debits by merchant, detects any two payments within ±12% amount and 25–37 day interval → marks as recurring.
- New "🔁 Subscriptions detected" card on Dashboard.
- Seed data extended with May 2026 entries for demo.

### AI Advancement (Task 4) — Anomaly Detection (AI step d) + Recurring (AI step e)
- New `detectAnomalies(txns)`: median-based outlier detection, returns `Set<id>`.
- Anomalous transactions show `⚠️ high` orange badge in Transactions list.
- Both computed with `useMemo` in Shell, passed via `ctx`.

### Hardening (Task 5) — Donut Chart
- Empty data shows placeholder instead of broken SVG.
- Single-category full-circle fix: sweep uses `frac * 1.9999 * π`.
- `Math.max(0.001, frac)` guard on each slice.
- Budget over-limit alert banner added to Budgets page.

---

## 2026-06-14 — Run 4 (NL query + savings goal coach + date guard + Enter-key auth)

### Summary
Shipped AI steps (f) and (g): savings goal coaching with SVG progress ring on Dashboard, and natural-language query bar in Insights. Hardened `detectTransfers` with date guard and auth forms with Enter-key submission.

### Audit
- Live site Vercel title still "Spendly — AI Expense Tracker" (Vercel cache lag) — repo `index.html` is correct, no action needed.
- All 5 docs present. No regressions.
- Figma MCP: not available this run — proceeded from Apple HIG.

### Design (Task 2) — Savings Goal Card (iOS Progress Ring)
- New `SavingsGoalCard` component on Dashboard: SVG circular progress ring (Apple Watch–style).
- Ring color: emerald when on-track, indigo when behind.
- Inline-editable goal amount (number input with dashed underline, indigo text — iOS-style).
- AI Coach badge (emerald). Context-aware coaching tips: 2 lines, different for on-track vs behind.
- Persisted to `nivo_savingsGoal` in localStorage.
- Default goal: ₹20,000/month.
- Recorded in `DESIGN.md` as new component token.

### Feature (Task 3) — Natural Language Query Bar (R-04 ✅) + Savings Goal Coach (R-06 ✅)
- **NL Query**: `NLQueryBar` component at top of Insights page.
  - Text input + "Ask" button; Enter key submits.
  - Parses: period (`this month` / `last 7 days` / `last month` / `today` / `all time`), category (any CATS item), intent (`how much` / `how many` / `biggest` / `average` / `income`).
  - Shows result inline in violet card beneath input.
  - Example queries surfaced in placeholder text.
- **Savings Goal**: `SavingsGoalCard` on Dashboard (described above under Design).

### AI Advancement (Task 4) — Steps (f) and (g)
- **Step (f) Savings Goal Coaching**: Client-side. `actual = income - spend`. Progress ring. Tips change based on on-track vs behind. Production path: Supabase Edge Function with 3-month history + browser Push Notification on goal achieved.
- **Step (g) Natural Language Query**: Client-side heuristic parser `parseNLQuery(q, enriched)`. Handles 5 intent types × 4 time periods × all categories = broad coverage. Production path: POST to Supabase Edge Function → Claude Haiku for open-ended queries.

### Hardening (Task 5) — Date Guards + Enter-key Auth
- `detectTransfers`: Added `isNaN(dDate.getTime())` and `isNaN(cDate.getTime())` guards — silently skips transactions with invalid/missing date strings, preventing `NaN` propagation in the `days` calculation.
- `AuthScreens`: All three form inputs (name, email, password) now fire `submit()` on Enter key via `onKeyDown={e=>{if(e.key==="Enter")submit();}}`. 2FA input fires `verify()` on Enter.
- Settings: UPI rule input also fires add-rule on Enter key for consistency.
