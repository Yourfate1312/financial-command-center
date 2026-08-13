# FCC CHANGELOG

## 2026-08-13 — Complete Database Backup Coverage

**Change:** Extended the existing "Download Full Backup (JSON)" button in Settings to cover all 16 database tables (was previously 9/16).

**Tables added to backup:**
- `crypto_trades`
- `investment_lots` ⭐ (critical — contains actual investment cost-basis history)
- `investment_sales` ⭐ (critical — contains realized P&L records)
- `emi_plans`
- `emi_schedule`
- `account_reconciliations`
- `profiles`

**Backup method:** Client-side JSON export via browser download. No new infrastructure introduced. Extended existing button.

**Storage:** User's local device (downloaded file). No cloud storage, no automation.

**Coverage:** 16/16 tables.

**Cost:** ₹0.

**How to create a backup:**
1. Open FCC → Settings page
2. Click "Download Full Backup (JSON)"
3. Save the downloaded `.json` file somewhere safe (Google Drive, email to yourself, etc.)

**How to restore financial DATA:**
The JSON backup can be reimported into Supabase via:
1. Go to Supabase → SQL Editor
2. For each table, use INSERT statements built from the JSON arrays
3. Restore in dependency order: profiles → accounts → transactions → investments → investment_lots → investment_sales → loans → bets → trades → crypto_trades → emi_plans → emi_schedule → rewards → goals → documents → account_reconciliations
4. Note: foreign key ordering matters. Restore parent tables before child tables.

**How to restore DATABASE STRUCTURE:**
The database structure (tables, enums, triggers, functions, RLS policies) is NOT included in the JSON backup. To restore structure:
- Re-run all SQL migrations from this conversation history in order (001–022)
- The migration history in the conversation is the only source of truth for structure

**What cannot be restored automatically:**
- Database schema (tables, enums, types, triggers, functions, RLS policies) — requires re-running migrations manually
- auth.users entries (Supabase Auth) — must be recreated manually; the `profiles` table backup covers app data but not the auth credentials
- The `user_id` foreign keys in all tables reference `auth.users.id` — if the user is recreated with a different UUID, all records must be updated

**Limitations:**
- Manual only — no automation, no scheduling
- Stored locally only — depends on user remembering to run it
- No incremental backup — always a full export
- No encryption — the JSON file contains all financial data in plaintext; store securely
- Supabase free tier retains its own internal backups for 7 days (Point-in-Time Recovery not available on free tier)

**Files modified:** `index.html` (Settings page backup button + description text only)

## 2026-08-13 — Gambling & Snooker Capital Architecture

**Change:** Converted Gambling and Snooker from personal expense categories into capital-allocation activities, following the same architecture as Trading and Crypto.

**Database changes (run by user before this UI build):**
- `alter type account_type add value 'gambling'`
- `alter type account_type add value 'snooker'`
- Inserted 'Gambling Capital' account (type=gambling, include_in_net_worth=false, balance=0)
- Inserted 'Snooker Capital' account (type=snooker, include_in_net_worth=false, balance=0)

**UI changes:**
- Gambling page: added Capital & Funding section (Funding Allocated, Withdrawals, Net Capital, Balance) identical to Trading/Crypto
- New Snooker page added: Capital & Funding + Performance (P&L, Win Rate, Staked/Returned from bets table filtered to sport='Snooker (Bets)')
- Removed 'Gambling' and 'Snooker' from expense categories
- Removed 'Gambling Winnings' and 'Snooker Winnings' from income categories
- Added 'gambling' and 'snooker' to Account CRUD type dropdown
- Capital flow refreshes on transaction add/delete

**How it works:**
- Fund via Transfer (Transactions page): Bank → Gambling Capital or Bank → Snooker Capital
- Withdraw via Transfer: Gambling Capital → Bank or Snooker Capital → Bank
- Funding is NOT a personal expense; withdrawal is NOT personal income
- P&L remains display-only (bets table), never booked to transactions

**Historical data:**
- No historical transactions used Gambling/Snooker expense categories (confirmed by user)
- All 34 historical bets preserved untouched in bets table
- Snooker bets preserved; sport='Snooker (Bets)' still correctly separates them

**Net Worth impact:**
- Gambling Capital and Snooker Capital: include_in_net_worth=false (same as Trading/Crypto)
- NOTE: Transferring Bank → Gambling DOES reduce Net Worth by the funded amount (bank decreases, gambling account not counted). This matches Trading/Crypto behaviour but differs from the Master Spec requirement of NW-neutral transfers. Documented as a known limitation consistent with the established architecture.

**Liquidity impact:**
- Funding reduces liquidity (bank decreases) ✅
- Gambling/Snooker Capital not counted as liquidity ✅
- Withdrawal increases liquidity ✅

**Tests:** Logic verified via code inspection and architecture trace. No destructive tests run against production data.

## 2026-08-13 — Net Worth Capital Allocation Fix

**Bug:** Capital allocation to Trading, Crypto, Gambling, and Snooker incorrectly reduced Net Worth. When funding an activity account via transfer, the source bank balance decreased but the activity account was excluded from Net Worth (include_in_net_worth = false), causing a net reduction equal to the funded amount.

**Root cause:** Activity capital accounts had include_in_net_worth = false. This was architecturally incorrect — these accounts hold real capital (actual money in real accounts like Groww), not excluded speculative positions.

**Key architectural clarification confirmed during inspection:**
- accounts.balance for Trading/Crypto/Gambling/Snooker = real account balance (manually updated by user)
- Trade/crypto/gambling/snooker P&L = computed separately from trades/bets tables, display-only, never enters Net Worth formula
- No double-counting risk exists — account balance and P&L are completely independent data sources

**Fix:** Single SQL UPDATE — set include_in_net_worth = true for all four activity account types:
```sql
update public.accounts set include_in_net_worth = true
where type in ('trading', 'crypto', 'gambling', 'snooker');
```

**No code changes required** beyond updating the Dashboard Net Worth subtitle label.

**Accounting invariant verified (all 8 tests via simulation):**
- Fund Trading ₹10,000 → NW unchanged ✅
- Withdraw Trading ₹4,000 → NW unchanged ✅
- Fund Crypto ₹10,000 → NW unchanged ✅
- Fund Gambling ₹10,000 → NW unchanged ✅
- Fund Snooker ₹10,000 → NW unchanged ✅
- Activity loss ₹2,000 → NW decreases ₹2,000 ✅
- Activity gain ₹2,000 → NW increases ₹2,000 ✅
- Full lifecycle (fund + loss + withdraw) → NW = 48,000 ✅

**Liquidity:** Unchanged — liquidity calculation still filters to bank + cash only.

**Files modified:** index.html (subtitle label only), FCC_CHANGELOG.md
**Database change:** UPDATE accounts SET include_in_net_worth = true WHERE type IN ('trading','crypto','gambling','snooker')
