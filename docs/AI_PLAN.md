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
- `confidence` ranges 30–99 based on match specificity:
  - 99 = user UPI rule (exact)
  - 93 = exact well-known brand name match
  - 78–88 = broad keyword pattern
  - 30 = unknown ("Other")
- `method`: `'rule'` | `'pattern'` | `'fallback'`
- `ConfBadge` component shows amber/red badge on uncertain predictions (<80%)
- User corrections via dropdown set `catConf=100, catMethod='user'` → teaches rules store
- New UPI rules persist and get confidence 99 on future matches

**Production upgrade path:**
```
POST /functions/v1/categorize
{ merchant, vpa, history: last5SameMerchant[] }
→ { category, confidence, reasoning }
```
Use `text-embedding-3-small` (OpenAI) to embed merchant name, find nearest centroid in a pre-labeled UPI dataset. Fine-tune on user's correction history stored in Supabase `corrections` table.

**Learn-from-corrections (planned):**
Persist corrections in `nivo_learned_rules` (merchant → category map). After 2+ corrections to same category for same merchant, promote to permanent rule with confidence 95.

---

## (c) Spend Forecasting ✅ SHIPPED (client)

**Status**: v2 — linear projection live, ML upgrade planned.

**What's live (Insights page):**
- Linear model: `projectedMonthEnd = (totalSpend / activeDays) * daysInMonth`
- Shows: projected total, progress bar (% of month elapsed), projected remaining spend
- Labeled as heuristic in UI; TODO comment marks edge-function hookup point

**Production upgrade path:**
```
POST /functions/v1/forecast
{ txns: Transaction[], month: "2026-06", budgets: Budget[] }
→ { projected: number, confidence: "high"|"medium"|"low", breakdown: {[cat]: number} }
```
Use exponential smoothing on weekly spend velocity (Holt-Winters if sufficient history). Apply day-of-week seasonality (weekends typically spike for Food/Entertainment).

---

## (b) Monthly Narrative Summary — PLANNED

**Target:** On the 1st of each month, generate a 3-sentence paragraph:  
*"You spent ₹47,200 in May — ₹3,000 under budget. Food was your biggest category at 28%. Compared to April, Travel jumped by ₹8,500 due to your MakeMyTrip booking."*

**Implementation plan:**
1. Client-side template engine (this run or next) — fill in {top_category}, {vs_last_month}, {savings_rate}
2. Production: `POST /functions/v1/narrative` with monthly summary data → GPT-4o or claude-haiku-4-5 generates paragraph

---

## (d) Anomaly Detection — PLANNED

Flag transactions >2× the median for that merchant in the last 90 days. Show orange warning badge in transaction list. Client-side median calculation is feasible without LLM.

---

## (e) Recurring Subscription Detection — PLANNED

Group transactions by `merchant + approximate_amount` within ±5%. If pattern repeats with interval 28–31 days, mark as `isRecurring=true`. Show subscription summary card in Dashboard. Pure client-side heuristic.

---

## Summary Table

| Feature | Client heuristic | LLM/edge version |
|---|---|---|
| (a) Categorisation | ✅ shipped v2 | Planned — embeddings |
| (b) Narrative | ❌ | Planned next |
| (c) Forecast | ✅ shipped v2 | Planned — Holt-Winters |
| (d) Anomaly | ❌ | Planned (client first) |
| (e) Recurring | ❌ | Planned (client first) |
| (f) Goal coaching | ❌ | Planned |
| (g) NL query | ❌ | Planned |
| (h) Email parsing | ❌ | Planned |
| (i) Budget recs | ❌ | Planned |
