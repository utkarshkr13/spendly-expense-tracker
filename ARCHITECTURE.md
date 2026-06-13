# Spendly — Architecture & Build Plan

A Fold-style, Gmail-powered expense tracker. Minimal UI, heavy analytics, AI categorization, multi-account transfer dedup. Frontend in React, backend on Supabase, hosted on Vercel.

## Data flow
Gmail bank/UPI alert emails → Gmail API (read-only) + push notifications → ingest worker (Vercel cron / Supabase Edge Function) parses → normalizes → AI categorizes → dedups → Supabase Postgres (RLS per user) → React app on Vercel. Manual Excel import feeds the same pipeline.

## Gmail integration
- OAuth 2.0 scope `gmail.readonly` (never send/delete). Tokens encrypted in Supabase.
- `users.messages.list` with query like `from:(alerts@hdfcbank.net OR alerts@icicibank.com) newer_than:35d`.
- Use `historyId` + Gmail push (Pub/Sub) so each sync only pulls new mail.
- Parse with `users.messages.get?format=full`, decode base64 body, extract via per-bank regex: amount, debit/credit, account last-4, merchant/VPA, date, ref.

## Multi-account transfer dedup (core feature)
Model your own accounts (salary, spends, ICICI). Money moved between accounts you own is an internal transfer — not spend or income.
Algorithm: for each debit in account A, find a credit in a different owned account B where amount is equal (within tolerance) and dates are within a 2-day window. Match → tag both legs as Transfer and exclude from totals → surface for one-tap confirm.
Demo: naive spend ₹1,05,628 → true spend ₹55,628 (₹50,000 of self-transfers netted out).

## Excel import
Upload .xlsx/.csv with columns: date, account, type, amount, merchant, upi_id, ref. The `ref` column prevents re-importing a transaction already captured via Gmail. Same categorize + dedup pipeline.

## AI categorization + UPI auto-tagging
1. Rule memory: `upi_rules` table maps VPA → category. Tag `zomato@hdfcbank` as Food once, every future payment to it auto-marks Food.
2. AI fallback: an LLM classifies unknown merchants and proposes a new rule; also powers natural-language insights.

## Supabase schema
```sql
accounts(id, user_id, name, bank, last4, role)
transactions(id, user_id, account_id, date, type, amount, merchant, vpa, category, source, ref, is_transfer, transfer_group_id)
upi_rules(id, user_id, vpa, category)
gmail_tokens(user_id, access_token, refresh_token, history_id)
```
Row-level security so each user sees only their rows. Edge Functions run Gmail sync + dedup on a schedule.

## Two-factor authentication
Supabase Auth MFA: sign in with Google or magic-link, enroll TOTP via `supabase.auth.mfa.enroll()` (QR code), require 6-digit code on login (`mfa.challenge` + `verify`). Optional WebAuthn/passkeys later.

## Deploy to Vercel
1. Static build (this repo) deploys as-is, or `create-next-app` for the full version.
2. Add env vars: SUPABASE_URL, SUPABASE_ANON_KEY, Google OAuth creds.
3. Gmail sync as a Vercel Cron Job / Supabase Edge Function every 15 min.
4. `git push` → Vercel auto-builds and deploys.
