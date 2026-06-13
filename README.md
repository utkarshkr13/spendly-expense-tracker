# Spendly — AI Expense Tracker

A Fold-style, Gmail-powered expense tracker. Minimal UI, heavy analytics, AI categorization, and multi-account internal-transfer dedup so moving money between your own accounts is never double-counted as spend.

**Live:** deployed on Vercel (static, zero-build).

## Features
- 📉 **Real vs naive spend** — self-transfers between your own accounts are auto-detected (equal amount, 2-day window) and excluded. Demo: ₹1,05,628 naive → ₹55,628 real.
- ✨ **AI categorization + UPI auto-tagging** — tag a UPI ID once, the rule applies to every future payment from it.
- 📊 Analytics: daily-spend area chart, category donut, spend-by-account bars, KPI cards.
- 🔍 Search, account filter, inline re-categorization.
- ⬆️ Excel import, 🛡️ 2FA-ready, ✉️ Gmail connect.

## Run locally
It's a single static file — just open `index.html` in a browser, or serve it:
```bash
npx serve .
```

## Architecture / backend plan
See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the Gmail API parsing approach, Supabase schema, transfer-dedup algorithm, Excel format, 2FA (TOTP), and Vercel deploy steps.

> Current build uses mock data emulating parsed Gmail bank alerts. Wire up Google OAuth (gmail.readonly) + Supabase to go live.
