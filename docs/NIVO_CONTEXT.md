# Nivo — Living Architecture & Context

> Auto-maintained by the AI Tech Manager. Updated every run. Last: 2026-06-14.

## What Is Nivo?

Single-file React expense tracker for Indian / UPI users. Deployed on Vercel, auto-deploys on push to `main`. Live at https://spendly-expense-tracker-six.vercel.app/

## Tech Stack

| Layer | Choice | Notes |
|---|---|---|
| Framework | React 18 UMD + Babel standalone | CDN, no build step |
| Styling | Tailwind CDN (JIT) | All utilities available |
| Charts | Hand-rolled inline SVG | `AreaSpark`, `Donut`, `Bars` — NO Recharts (causes blank screens) |
| Persistence | `localStorage` under `nivo_` prefix | version-migrated; quota-guarded |
| Auth | Login → 6-digit 2FA (demo: any 6 digits) | Production: Supabase Auth TOTP |
| AI | Client-side heuristics | Production: Supabase Edge Functions |

## File Structure

```
index.html          ← entire app (single file)
docs/
  NIVO_CONTEXT.md   ← this file
  ROADMAP.md        ← prioritised backlog
  AI_PLAN.md        ← AI feature roadmap
  CHANGELOG.md      ← dated change log
  DESIGN.md         ← design system tokens
```

## Core Concepts

### HDFC Salary → Spends Dedup
When salary lands in `hdfc_sal` and the user self-transfers to `hdfc_spend`, Nivo detects it as an internal transfer via `detectTransfers()` — matching same-amount debit/credit pairs within 2 days across different accounts. User confirms in the Transactions page. Confirmed transfers are excluded from spend calculations.

### AI Categorisation
`aiCategorizeFull(merchant, vpa, rules)` returns `{category, confidence, method}`.
- `method: 'rule'` — user-defined UPI tag rule (confidence 99)
- `method: 'pattern'` — keyword regex (confidence 78–96 depending on specificity)
- `method: 'fallback'` — unknown merchant (confidence 30)

User corrections via the category dropdown set `catConf=100, catMethod='user'` and teach the rules store.

### State
```
nivo__v         schema version (2)
nivo_txns       Transaction[]
nivo_rules      {[vpa]: category}
nivo_confirmed  number[]  (transfer-pair IDs)
nivo_budgets    {[category]: number}
nivo_user       {name, email}
nivo_twofa      boolean
nivo_gmail      boolean
```

## Pages

| Page | Key component | Purpose |
|---|---|---|
| Dashboard | `Dashboard` | Overview: stats, area chart, donut, budget bars, recent txns |
| Transactions | `Transactions` | Full list with search/filter, transfer review, manual add |
| Accounts | `Accounts` | Balance cards per account, HDFC dedup explainer |
| Budgets | `Budgets` | Per-category limit editing + progress bars |
| Insights | `Insights` | Top merchants, category bars, month-end forecast, stats |
| Settings | `Settings` | Profile, 2FA toggle, Gmail sync, UPI rules, data export |

## Known Issues / Constraints

- Live site title shows "Spendly" (old cache); code title is correct "Nivo". Vercel CDN will invalidate eventually.
- Charts are SVG-only — no animation yet (roadmap P2).
- Account balances are hardcoded seed data — no real bank API.
- 2FA demo accepts any 6 digits; production requires Supabase Auth MFA wiring.
- Month-end forecast uses simple daily-average linear model, not seasonality-aware.

## Conventions

- Never introduce external chart libraries (Recharts caused white screens).
- Keep everything in one `index.html`. No separate JS/CSS files.
- Use `₹` currency, Indian number formatting via `toLocaleString("en-IN")`.
- Every txn must have: `id, date, account, type, amount, merchant, vpa, source, category, catConf, catMethod`.
- `amount` is always guarded with `Math.max(0, Number(amount)||0)`.
- `INR(n)` safely handles `undefined`/`NaN` via `Math.round(n||0)`.
