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
