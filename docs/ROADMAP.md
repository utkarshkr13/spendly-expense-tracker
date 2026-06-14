# Nivo — Product Roadmap

> Maintained by AI Tech Manager. Last updated: 2026-06-14.

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

## Backlog

### P0 — Must ship soon

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-01 | Recurring subscription detection | Detect Netflix/Spotify/SaaS charges repeating monthly; alert if price changed | M |
| R-02 | Anomaly detection | Flag transactions >2× usual amount for that merchant | S |
| R-03 | Budget alert notifications | Browser Notification API when approaching/over budget | S |

### P1 — High value

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-04 | Natural-language query | "How much on food last week?" — parse intent + run filter | M |
| R-05 | Monthly narrative summary | "Here's your June in one paragraph" — LLM-generated digest | M |
| R-06 | Savings goal coaching | Set a goal (e.g., save ₹50k for trip), track progress, suggest cuts | L |
| R-07 | Dark mode | System-preference aware; `prefers-color-scheme` + CSS vars | M |
| R-08 | CSV / bank-statement import | Upload HDFC/ICICI CSV → parse → load txns | L |

### P2 — Nice to have

| ID | Feature | Rationale | Effort |
|---|---|---|---|
| R-09 | Chart animations | Smooth SVG transitions on load | S |
| R-10 | Split transactions | Mark ₹5000 Amazon order as ₹2k Electronics + ₹3k Household | M |
| R-11 | Net worth trend | Line chart of monthly net worth | S |
| R-12 | Peer benchmarking | "You spend X% less on food than similar earners" — anonymised | L |
| R-13 | PWA / install prompt | `manifest.json` + service worker for home-screen install | M |
| R-14 | Multi-currency | Track USD/EUR expenses, convert to INR | M |

## Completed This Run (2026-06-14)

- ✅ Month-end forecast (Insights page) — R-NEW → now part of shipped
- ✅ AI confidence scores (Transactions) — AI roadmap step (a)
- ✅ Frosted-glass UI (sidebar + header) — design system
- ✅ a11y hardening across auth, transactions, budgets, settings
