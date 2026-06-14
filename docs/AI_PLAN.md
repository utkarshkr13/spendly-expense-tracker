# Nivo — AI Capabilities Roadmap

> Each run advances one step. "Shipped (client)" = live heuristic. "Planned" = LLM/edge-function version waiting.

## Target AI Surface Area

```
(a) Smart auto-categorisation with confidence + learn-from-corrections
(b) Natural-language insights & monthly narrative summaries
(c) Spend forecasting / projected month-end
(d) Anomaly & duplicate detection
(e) Recurring-subscription detection
(f) Savings-goal coaching
(g) Natural-language query ("how much on food last week?")
(h) Bank-email parsing templates
(i) Budget recommendations
```

---

## (a) Smart Auto-Categorisation ✅ SHIPPED (client)

**Status**: v2 — heuristic live, LLM upgrade planned.

**What's live:**
- `aiCategorizeFull(merchant, vpa, rules)` returns `{category, confidence, method}`
- `confidence` ranges 30–99 based on match specificity
- `method`: `'rule'` | `'pattern'` | `'fallback'`
- `ConfBadge` shows amber/red badge on uncertain predictions (<80%)
- User corrections via dropdown set `catConf=100, catMethod='user'`

**Production upgrade path:**
```
POST /functions/v1/categorize
{ merchant, vpa, history: last5SameMerchant[] }
→ { category, confidence, reasoning }
```
Use `text-embedding-3-small` to embed merchant name, find nearest centroid in pre-labeled UPI dataset. Fine-tune on user's correction history in Supabase `corrections` table.

---

## (c) Spend Forecasting ✅ SHIPPED (client)

**Status**: v2 — linear projection live.

**What's live (Insights page):**
- Linear model: `projectedMonthEnd = (totalSpend / activeDays) * daysInMonth`
- Shows: projected total, progress bar, projected remaining

**Production upgrade path:**
```
POST /functions/v1/forecast
{ txns: Transaction[], month: "2026-06", budgets: Budget[] }
→ { projected: number, confidence: "high"|"medium"|"low", breakdown: {[cat]: number} }
```
Use exponential smoothing (Holt-Winters) on weekly spend velocity.

---

## (d) Anomaly Detection ✅ SHIPPED (client)

**Status**: v3 — median-based heuristic live.

**What's live (Transactions page):**
- `detectAnomalies(txns)`: flags txns where `amount > 2× median` for that merchant
- `⚠️ high` orange badge in Transactions list

**Production upgrade path:**
```
POST /functions/v1/anomaly
{ txnId: string, merchantHistory: Transaction[] }
→ { isAnomaly: boolean, zScore: number, alert: string }
```
Rolling IQR over 90-day window. Browser Push Notification on detection.

---

## (e) Recurring Subscription Detection ✅ SHIPPED (client)

**Status**: v3 — ~30-day interval heuristic live.

**What's live (Dashboard):**
- `detectRecurring(txns)`: same merchant, ±12% amount, 25–37 day interval
- Dashboard "Subscriptions detected" card with monthly total

**Production upgrade path:**
```
POST /functions/v1/subscriptions
{ txns: Transaction[], lookbackDays: 90 }
→ { subscriptions: [{merchant, amount, cycle, nextExpected, priceChanged}] }
```
Push notification when price changes.

---

## (f) Savings Goal Coaching ✅ SHIPPED (client) — Run 4

**Status**: v4 — client-side goal ring + coaching tips live.

**What's live (Dashboard — Savings Goal card):**
- `SavingsGoalCard` component: SVG circular progress ring (Apple Watch style)
- `actual = max(0, income - totalSpend)` for current month
- `pct = min(100, round(actual / goal * 100))`
- Ring color: emerald (≥ goal) / indigo (< goal)
- Inline-editable goal (number input, dashed border, indigo text)
- **Context-aware tips**: 2 coaching lines that change based on on-track vs behind
  - On-track: celebrate + suggest FD/liquid fund
  - Behind: reduce discretionary spend + pay-yourself-first auto-transfer
- Persisted: `nivo_savingsGoal` in localStorage, default ₹20,000/month
- `isNaN(dash)` guard on `strokeDashoffset` prevents broken ring on invalid goal

**Production upgrade path:**
```
POST /functions/v1/savings-coach
{ history: MonthlySnapshot[], goal: number, currentMonth: MonthSummary }
→ { tip: string, projectedGoalDate: date, suggestedCuts: {category, amount}[] }
```
Claude Haiku generates personalized tips from 3-month spending history. Browser Push Notification when monthly goal is achieved for the first time.

---

## (g) Natural-Language Query ✅ SHIPPED (client) — Run 4

**Status**: v4 — client-side heuristic parser live.

**What's live (Insights page — Ask Nivo bar):**
- `parseNLQuery(q, enriched)` function
- **Period extraction**: `this month` / `last 7 days` / `last month` / `today` / `all time`
- **Category extraction**: matches any entry in CATS array (case-insensitive)
- **Intent detection** (5 types):
  - `how much / total / spent / spend` → sum of matching txns
  - `how many / count / number` → count of matching txns
  - `biggest / largest / most expensive / highest` → top single txn
  - `average / avg / mean` → mean amount
  - `income / earn / salary` → total credits categorized as Income
- Fallback: friendly error message with example queries
- Enter key submits query
- Result shown in violet card below input
- Example queries in placeholder text

**Production upgrade path:**
```
POST /functions/v1/nlquery
{ q: string, txns: Transaction[], context: UserContext }
→ { answer: string, txnIds: string[], chartData?: ChartPoint[] }
```
Claude Haiku with structured tool calling: `filter_transactions`, `compute_aggregate`, `format_answer`. Handles complex queries: "Compare my Food spend this month vs last month" or "Which day did I spend the most last week?"

---

## (b) Monthly Narrative Summary — PLANNED (next)

**Target:** On the 1st–5th of each month, generate a 3-sentence paragraph in the AI insight card:
*"You spent ₹47,200 in May — ₹3,000 under budget. Food was your biggest category at 28%. Compared to April, Travel jumped by ₹8,500 due to your MakeMyTrip booking."*

**Implementation plan (next run):**
1. Client-side template engine: fill `{top_category}`, `{savings_rate}`, `{vs_budget}`, `{biggest_change}` tokens
2. Show in Dashboard AI insight card when `dayOfMonth <= 5`
3. Store last month's snapshot in `nivo_last_month_snapshot` on month-end
4. Production: `POST /functions/v1/narrative` → Claude Haiku generates paragraph

---

## Summary Table

| Feature | Client heuristic | LLM/edge version |
|---|---|---|
| (a) Categorisation | ✅ shipped v2 | Planned — embeddings |
| (b) Narrative | ❌ | Planned next run |
| (c) Forecast | ✅ shipped v2 | Planned — Holt-Winters |
| (d) Anomaly | ✅ shipped v3 | Planned — IQR + push |
| (e) Recurring | ✅ shipped v3 | Planned — 90-day window + push |
| (f) Goal coaching | ✅ shipped v4 | Planned — Haiku + push |
| (g) NL query | ✅ shipped v4 | Planned — Haiku + tool calling |
| (h) Email parsing | ❌ | Planned |
| (i) Budget recs | ❌ | Planned |
