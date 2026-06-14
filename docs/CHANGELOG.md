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
- Replaced Tailwind `shadow-sm` on `Card` with `.card-shadow` CSS class: `box-shadow: 0 1px 3px rgba(0,0,0,0.05), 0 4px 14px rgba(0,0,0,0.05)`.
- Hover state: `0 2px 6px rgba(0,0,0,0.08), 0 8px 22px rgba(0,0,0,0.07)` with `transition: box-shadow 0.18s ease`.
- Added `.btn-press` CSS class: `transition: transform 0.1s ease; :active { transform: scale(0.97) }` — iOS tactile press feedback on all buttons and clickable cards.
- Applied `btn-press` to: all `<button>` elements, clickable `Card` wrapper, nav items.
- Updated Bars chart bars to use `transition-all duration-500` for smooth width transitions.
- Recorded in `DESIGN.md`.

### Feature (Task 3) — Recurring Subscription Detection (R-01 ✅)
- New `detectRecurring(txns)` function: groups debits by merchant, detects any two payments within ±12% amount and 25–37 day interval → marks as recurring.
- New "Subscriptions detected" card on Dashboard (bottom): shows each recurring merchant with monthly amount, category, detection count.
- Shows total monthly subscription cost in header.
- Seed data extended with May 2026 entries (NETFLIX ₹1299, SWIGGY ₹3100) so recurring detection fires for new users. Existing users: click "Reset demo data" in Settings to see it.
- Production path documented: Supabase Edge Function with 90-day rolling window + push notification on price change.

### AI Advancement (Task 4) — Anomaly Detection (AI step d) + Recurring (AI step e)
- New `detectAnomalies(txns)` function: for each merchant with ≥2 transactions, computes median amount; flags any txn >2× median as anomalous. Returns a `Set<id>`.
- Anomalous transactions now show an `⚠️ high` orange badge in the Transactions list merchant cell.
- `recurring` and `anomalies` computed with `useMemo` in Shell, passed through `ctx` to all pages.
- Both functions documented in `AI_PLAN.md` with production upgrade paths.

### Hardening (Task 5) — Donut Chart
- Added `const nd = data.filter(d => d.value > 0)` guard — empty data shows "No spending data yet" placeholder instead of broken SVG.
- Single-category guard: when only one category, sweep uses `frac * 1.9999 * Math.PI` instead of full `2π` — prevents the SVG degenerate arc (start=end point) that rendered as blank.
- `Math.max(0.001, frac)` guard on each slice to prevent zero-width arcs.
- Budget page: added over-budget alert banner when any category exceeds limit (in-app notification, no browser API needed).
