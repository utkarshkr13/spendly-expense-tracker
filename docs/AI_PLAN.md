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

## (b) Monthly Narrative Summary ✅ SHIPPED (client) — Run 5

**Status**: v5 — client-side template engine live.

**What's live (Dashboard — AI insight card):**
- `generateMonthlyNarrative(enriched, budgets, byCat, income, totalSpend, pairs, dedupSaved)`
- **3-sentence structure**:
  1. Spend total + top category % + dedup note (transfers excluded)
  2. Budget health: over-budget categories named, or within-budget praise
  3. Savings rate coaching: celebrate ≥20%, nudge 1–19%, alert if negative
- Badge updated: `step b · narrative` sub-label on Dashboard AI card
- Replaces previous static one-liner

**Template tokens filled:**
- `{top_category}` → `byCat[0].name` + `topCatPct`
- `{savings_rate}` → `Math.round((income - totalSpend) / income * 100)`
- `{vs_budget}` → `overBudget.map(([c]) => c).join(' & ')` or praise
- `{dedup_note}` → `pairs.length` internal transfers excluded

**Production upgrade path:**
```
POST /functions/v1/narrative
{ history: MonthlySnapshot[], budgets: Budget[], currentMonth: MonthSummary }
→ { narrative: string }  // Claude Haiku generates personalised paragraph
```
With 3-month rolling history, peer-benchmark percentiles, and merchant-level insight ("your Swiggy spend jumped 40% vs last month").

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
- `SavingsGoalCard`: SVG circular progress ring, inline-editable goal, context-aware tips
- `isNaN(dash)` guard on `strokeDashoffset`

**Production upgrade path:**
```
POST /functions/v1/savings-coach
{ history: MonthlySnapshot[], goal: number, currentMonth: MonthSummary }
→ { tip: string, projectedGoalDate: date, suggestedCuts: {category, amount}[] }
```

---

## (g) Natural-Language Query ✅ SHIPPED (client) — Run 4

**Status**: v4 — client-side heuristic parser live.

**What's live (Insights page — Ask Nivo bar):**
- `parseNLQuery(q, enriched)` — 5 intents × 4 periods × all categories
- Enter key submits; result in violet card

**Production upgrade path:**
```
POST /functions/v1/nlquery
{ q: string, txns: Transaction[], context: UserContext }
→ { answer: string, txnIds: string[], chartData?: ChartPoint[] }
```
Claude Haiku with structured tool calling.

---

## (h) Bank-Email Parsing Templates — PLANNED (next)

**Target:** Parse common Indian bank SMS / email formats (HDFC, ICICI, SBI, Axis) into transactions.

**Implementation plan (next run):**
1. Define regex templates per bank for debit/credit SMS patterns
2. `parseHDFCSMS(text)`, `parseICICISMS(text)` → `{amount, merchant, date, type}`
3. Surface in Settings as "SMS import" paste-box
4. Production: Gmail API read-only → extract bank emails → parse → import

---

## (i) Budget Recommendations — PLANNED

**Target:** Suggest optimal budget limits based on historical spend patterns.

**Implementation plan:**
1. Client-side: compute 3-month rolling avg per category → suggest `avg * 1.1` as limit
2. Show "Suggested limit" chip in Budgets page with one-click accept
3. Production: Claude Haiku with full history → personalized recommendations with rationale

---

## Summary Table

| Feature | Client heuristic | LLM/edge version |
|---|---|---|
| (a) Categorisation | ✅ shipped v2 | Planned — embeddings |
| (b) Narrative | ✅ shipped v5 | Planned — Haiku + 3-month history |
| (c) Forecast | ✅ shipped v2 | Planned — Holt-Winters |
| (d) Anomaly | ✅ shipped v3 | Planned — IQR + push |
| (e) Recurring | ✅ shipped v3 | Planned — 90-day window + push |
| (f) Goal coaching | ✅ shipped v4 | Planned — Haiku + push |
| (g) NL query | ✅ shipped v4 | Planned — Haiku + tool calling |
| (h) Email parsing | ❌ planned next | Planned — Gmail API + templates |
| (i) Budget recs | ❌ planned | Planned — Haiku + 3-month avg |
