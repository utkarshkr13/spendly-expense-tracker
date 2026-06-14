# Nivo — Living Architecture & Context

> Auto-maintained by the AI Tech Manager. Updated every run. Last: 2026-06-14 (Run 5).

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

## Core Functions

### HDFC Salary → Spends Dedup
`detectTransfers(txns)` — matches same-amount debit/credit pairs within 2 days across different accounts. User confirms in Transactions page. Confirmed transfers excluded from spend calculations.

### AI Categorisation
`aiCategorizeFull(merchant, vpa, rules)` returns `{category, confidence, method}`.
- `method: 'rule'` — user-defined UPI tag rule (confidence 99)
- `method: 'pattern'` — keyword regex (confidence 78–96 depending on specificity)
- `method: 'fallback'` — unknown merchant (confidence 30)

User corrections via category dropdown set `catConf=100, catMethod='user'`.

### Recurring Subscription Detection (v3)
`detectRecurring(enriched)` — groups non-transfer debits by merchant; flags pairs with ±12% amount and 25–37 day gap as recurring subscriptions.

### Anomaly Detection (v3)
`detectAnomalies(txns)` — per merchant, computes median spend; flags txns >2× median as anomalous. Returns `Set<id>`. Shown as `⚠️ high` badge in Transactions list.

### Monthly Narrative (v5)
`generateMonthlyNarrative(enriched, budgets, byCat, income, totalSpend, pairs, dedupSaved)` — 3-sentence template: spend summary, budget health, savings rate coaching. Shown in Dashboard AI card.

### Spend Forecasting
In `Insights` page — linear model: `projectedMonthEnd = (totalSpend / activeDays) * daysInMonth`.

### Budget Alert Notifications (v5)
In `Shell`: `useEffect` watches `[budgets, byCat, notifPerm]`. When `notifPerm === 'granted'`, fires `Notification` at 90%/100% budget threshold. Deduped per category per month via `nivo_budgetAlert_{cat}_{YYYY-MM}` in localStorage.

## State

```
nivo__v                schema version (2)
nivo_txns              Transaction[]
nivo_rules             {[vpa]: category}
nivo_confirmed         number[]  (transfer-pair IDs)
nivo_budgets           {[category]: number}
nivo_user              {name, email}
nivo_twofa             boolean
nivo_gmail             boolean
nivo_savingsGoal       number  (monthly savings target, default 20000)
nivo_budgetAlert_*     boolean (per category per month, prevents repeat alerts)
```

## Transaction Schema

```ts
{
  id: number,
  date: string,        // "YYYY-MM-DD"
  account: string,     // ACCOUNTS[].id
  type: "debit"|"credit",
  amount: number,      // always ≥ 0, guarded Math.max(0, ...)
  merchant: string,
  vpa: string,         // UPI ID or ""
  source: "gmail"|"excel"|"manual",
  category: string,    // from CATS
  catConf: number,     // 0–100 AI confidence
  catMethod: "rule"|"pattern"|"fallback"|"user",
  isTransfer: boolean, // set in enriched (confirmed transfer)
  suggestedTransfer: boolean // matched but not yet confirmed
}
```

## Pages

| Page | Key component | Purpose |
|---|---|---|
| Dashboard | `Dashboard` | Stats, area chart, donut, budget bars, recent txns, savings ring, subscriptions card, AI narrative |
| Transactions | `Transactions` | SegmentedControl type filter + search/account/category selects, transfer review, anomaly badges, manual add |
| Accounts | `Accounts` | Balance cards per account, HDFC dedup explainer |
| Budgets | `Budgets` | Over-budget alert banner, per-category limit editing + progress bars |
| Insights | `Insights` | NL query bar, month-end forecast, top merchants, category bars, stats |
| Settings | `Settings` | Profile, 2FA toggle, budget alert notifications, Gmail sync, UPI rules, data export / reset |

## Known Issues / Constraints

- Live site title shows "Spendly" (old Vercel CDN cache); code title is correct "Nivo — Smart Expense Tracker". Will resolve after next Vercel re-deploy clears cache.
- Charts are SVG-only — no animation yet (roadmap P2 R-09).
- Account balances are hardcoded seed data — no real bank API.
- 2FA demo accepts any 6 digits; production requires Supabase Auth MFA wiring.
- Seed data has May 2026 entries; existing users with localStorage v2 won't see them until "Reset demo data" in Settings.
- Recurring detection requires ≥2 transactions with ~30-day gap.
- Budget notifications require the app tab to be open (foreground only). Background alerts need Web Push API (VAPID).

## Conventions

- Never introduce external chart libraries (Recharts caused white screens).
- Keep everything in one `index.html`. No separate JS/CSS files.
- Use `₹` currency, Indian number formatting via `toLocaleString("en-IN")`.
- Every txn must have: `id, date, account, type, amount, merchant, vpa, source, category, catConf, catMethod`.
- `amount` is always guarded with `Math.max(0, Number(amount)||0)`.
- `INR(n)` safely handles `undefined`/`NaN` via `Math.round(n||0)`.
- `useMemo` for all derived data (byCat, byDay, recurring, anomalies, enriched).
- One `useMemo` call per derived value — keep Shell's computation section readable.
- Use `SegmentedControl` for 2–4 mutually exclusive toggles; prefer over `<select>` when options fit in one row.
