# Nivo — Product Roadmap

> Maintained by AI Tech Manager. Last updated: 2026-06-14 (Run 4).

## Shipped ✅

| Feature | Shipped | Notes |
|---|---|---|
| Auth: login + 6-digit 2FA | v1 | Demo accepts any 6 digits |
| HDFC salary→spends dedup | v1 | `detectTransfers` with 2-day window |
| AI categorisation (heuristics) | v1 | keyword regex map |
| Dashboard: stats + charts | v1 | Area, Donut, Bars — pure SVG |
| Transactions: filter + manual add | v1 | |
| Budgets: per-category limits | v1 | |
| Insights: top merchants + category bars | v1 | |
| Settings: UPI rules + Gmail toggle + export | v1 | |
| AI confidence scores on category predictions | v2 | `catConf`, `catMethod` fields + `ConfBadge` |
| Month-end spend forecast | v2 | Linear daily-avg projection in Insights |
| Frosted-glass sidebar + header | v2 | `.glass-nav` CSS class, backdrop-blur-20 |
| localStorage quota guard + version migration | v2 | `LS_VERSION=2`, QuotaExceededError catch |
| Accessibility: aria labels, roles, aria-current | v2 | nav, tables, dialogs, toggles |
| **Recurring subscription detection** | **v3** | `detectRecurring()` — ~30-day same-amount grouping; Dashboard card |
| **Anomaly detection** | **v3** | `detectAnomalies()` — >2× median → ⚠️ badge in Transactions |
| **Refined card shadow system** | **v3** | `.card-shadow` + `.btn-press` iOS press state |
| **Budget over-limit alert banner** | **v3** | In-app banner on Budgets page when any category over limit |
| **Donut chart hardening** | **v3** | Empty data guard + single-category full-circle fix |
| **Natural-language query bar** | **v4** | `parseNLQuery()` — 5 intents × 4 periods × all categories; Insights page |
| **Savings goal coaching** | **v4** | `SavingsGoalCard` — SVG ring + tips + inline-editable goal; Dashboard |
| **detectTransfers date guard** | **v4** | `isNaN(date.getTime())` prevents NaN in days calc |
| **Enter-key auth forms** | **v4** | Login, signup, 2FA, UPI rule input all submit on Enter |

## Backlog

### P0 — Must ship soon

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-05 | Monthly narrative summary | Template-first: fill {top_category}, {savings_rate} tokens in AI insight card on 1st–5th of month | S |
| R-03 | Budget alert notifications (browser) | Browser Notification API when approaching/over budget; permission flow | S |

### P1 — High value

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-07 | Dark mode | System-preference aware; `prefers-color-scheme` + CSS vars | M |
| R-08 | CSV / bank-statement import | Upload HDFC/ICICI CSV → parse → load txns | L |
| R-02 | Anomaly summary card | Add anomaly count + top anomaly on Dashboard / Insights | S |

### P2 — Nice to have

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-09 | Chart animations | Smooth SVG transitions on load | S |
| R-10 | Split transactions | Mark ₹5000 Amazon order as ₹2k Electronics + ₹3k Household | M |
| R-11 | Net worth trend | Line chart of monthly net worth | S |
| R-12 | Peer benchmarking | "You spend X% less on food than similar earners" — anonymised | L |
| R-13 | PWA / install prompt | `manifest.json` + service worker for home-screen install | M |
| R-14 | Multi-currency | Track USD/EUR expenses, convert to INR | M |
