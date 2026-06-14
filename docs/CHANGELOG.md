# Nivo — Changelog

Append-only. Each AI Tech Manager run appends a dated entry.

---

## 2026-06-14 — Hotfix: Vercel Speed Insights

### Summary
Added Vercel Speed Insights instrumentation via the auto-injected CDN script. No npm, no build step required.

### Why the npm package doesn't apply here
`@vercel/speed-insights/next` and `@vercel/speed-insights/react` are Node.js ESM packages that require a bundler (Next.js / Vite / CRA). Nivo is a single `index.html` with React 18 UMD + Babel standalone — there is no build pipeline. The correct integration for any Vercel-deployed static site is:

```html
<!-- placed just before </body> -->
<script defer src="/_vercel/speed-insights/script.js"></script>
```

Vercel injects and serves this file at that path for every deployment automatically. In local development the 404 is silent (defer + no error handler). No API key or config needed.

### What this tracks
- Core Web Vitals: LCP, FID/INP, CLS, TTFB, FCP
- Visible in the Vercel project dashboard under Analytics → Speed Insights
- Zero runtime overhead — `defer` ensures it loads after the React app

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
