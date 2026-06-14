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
- `activeDays` uses only days ≤ today to avoid inflating the denominator with future days
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

## (d) Anomaly Detection ✅ SHIPPED (client)

**Status**: v3 — median-based heuristic live.

**What's live (Transactions page):**
- `detectAnomalies(txns)` groups debits by merchant, computes median amount per merchant
- Transactions where `amount > 2 × median` (and merchant has ≥2 txns) are flagged
- Returns a `Set<id>` of anomalous transaction IDs
- Transactions list shows `⚠️ high` orange badge on anomalous rows
- `anomalies` computed with `useMemo` in Shell, passed through `ctx`

**Production upgrade path:**
```
POST /functions/v1/anomaly
{ txnId: string, merchantHistory: Transaction[] }
→ { isAnomaly: boolean, zScore: number, alert: string }
```
Use rolling IQR (interquartile range) over 90-day window. Send browser Push Notification + in-app toast for real-time alerting. Store acknowledged anomalies in `nivo_anomaly_ack` localStorage key.

---

## (e) Recurring Subscription Detection ✅ SHIPPED (client)

**Status**: v3 — ~30-day interval heuristic live.

**What's live (Dashboard):**
- `detectRecurring(txns)` groups non-transfer debits by merchant
- Detects pairs of transactions with: same merchant, ±12% amount difference, 25–37 day interval
- Returns array of `{merchant, amount, category, lastDate, count, monthlyTotal}`
- Dashboard shows "🔁 Subscriptions detected" card with monthly total and per-subscription breakdown
- Seed data includes May + June NETFLIX/SWIGGY entries for demo visibility

**Production upgrade path:**
```
POST /functions/v1/subscriptions
{ txns: Transaction[], lookbackDays: 90 }
→ { subscriptions: [{merchant, amount, cycle: "monthly"|"annual", nextExpected: date, priceChanged: boolean}] }
```
Use sliding-window pattern matcher across 90-day history. Send push notification when price changes (e.g., Netflix raises from ₹1299 to ₹1499). Store subscription registry in `nivo_subscriptions` key for user review/dismissal.

---

## (b) Monthly Narrative Summary — PLANNED (next)

**Target:** On the 1st of each month, generate a 3-sentence paragraph:
*"You spent ₹47,200 in May — ₹3,000 under budget. Food was your biggest category at 28%. Compared to April, Travel jumped by ₹8,500 due to your MakeMyTrip booking."*

**Implementation plan (next run):**
1. Client-side template engine: fill `{top_category}`, `{savings_rate}`, `{vs_budget}` tokens
2. Show in Dashboard AI insight card when it's the 1st–5th of the month
3. Production: `POST /functions/v1/narrative` → GPT-4o / claude-haiku-4-5 generates paragraph

---

## (f) Savings-Goal Coaching — PLANNED

Set a goal (e.g., "Save ₹50,000 for Goa trip by August"). Track monthly delta. Client-side projection: "At current savings rate you'll reach your goal in 3.2 months." Show progress ring.

---

## (g) Natural-Language Query — PLANNED

Parse "how much on food last week?" via regex intent extraction:
- Extract time range ("last week", "in May", "today")
- Extract category or merchant keyword
- Run filter on txns, return total + list

No LLM needed for basic intents. Ship NL query box in Insights page.

---

## Summary Table

| Feature | Client heuristic | LLM/edge version |
|---|---|---|
| (a) Categorisation | ✅ shipped v2 | Planned — embeddings |
| (b) Narrative | ❌ | Planned next run |
| (c) Forecast | ✅ shipped v2 | Planned — Holt-Winters |
| (d) Anomaly | ✅ shipped v3 | Planned — IQR + push |
| (e) Recurring | ✅ shipped v3 | Planned — 90-day window + push |
| (f) Goal coaching | ❌ | Planned |
| (g) NL query | ❌ | Planned |
| (h) Email parsing | ❌ | Planned |
| (i) Budget recs | ❌ | Planned |
