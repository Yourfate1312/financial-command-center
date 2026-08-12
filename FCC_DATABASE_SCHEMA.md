# FCC DATABASE SCHEMA

**Last updated:** 2026-08-12
**Source:** Inspected from `index.html` JS queries, all SQL migrations run during build, and verified `pg_tables`/`pg_policies` query results provided by user.
**Supabase project ID:** adcgzaiovsmztipkshws
**Region:** Mumbai/Singapore

This document describes the FCC database **as it exists today**.
It does NOT describe future intended state.

Anything not directly verified is explicitly labelled **[UNVERIFIED]**.

---

## ENUMS

```sql
account_type: 'bank' | 'cash' | 'credit_card' | 'trading' | 'crypto' | 'fd'
update_frequency: 'realtime' | 'monthly'
transaction_type: 'income' | 'expense' | 'transfer' | 'investment_buy' | 'investment_sell' | 'emi_conversion' | 'emi_principal'
bet_outcome: 'pending' | 'win' | 'loss' | 'push'
loan_type: 'debtor' | 'creditor'
reward_type: 'shareholder' | 'credit_card' | 'airline_hotel' | 'app'
```

---

## TABLE: profiles

**Purpose:** 1:1 extension of auth.users. Stores per-user app settings.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK, FK → auth.users ON DELETE CASCADE | equals auth.users.id |
| full_name | text | nullable | |
| timezone | text | NOT NULL, default 'Asia/Kolkata' | |
| created_at | timestamptz | NOT NULL, default now() | |
| updated_at | timestamptz | NOT NULL, default now() | |

**Triggers:** `on_auth_user_created` (AFTER INSERT on auth.users → auto-inserts profile), `set_updated_at`.
**RLS policies:** SELECT (`auth.uid() = id`), UPDATE (`auth.uid() = id`). No INSERT (trigger only). No DELETE.

---

## TABLE: accounts

**Purpose:** All financial accounts — bank, cash, credit card, trading, crypto, FD.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK, default gen_random_uuid() | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| name | text | NOT NULL | e.g. 'Kotak Savings' |
| type | account_type | NOT NULL | enum |
| balance | numeric(14,2) | NOT NULL, default 0 | for credit_card: amount owed (positive). Trigger direction inverted. |
| include_in_net_worth | boolean | NOT NULL, default true | false for trading, crypto |
| update_frequency | update_frequency | NOT NULL, default 'realtime' | 'monthly' for trading, crypto, CC, FD |
| credit_limit | numeric(12,2) | nullable | credit card only |
| statement_date | date | nullable | credit card only |
| due_date | date | nullable | credit card only |
| maturity_date | date | nullable | FD only |
| maturity_amount | numeric(14,2) | nullable | FD only |
| lien_marked | boolean | NOT NULL, default false | FD: true if securing a CC |
| lien_against_account_id | uuid | nullable, FK → accounts | self-referential |
| renewal_instruction | text | nullable | FD: e.g. 'Renew upon maturity' |
| created_at | timestamptz | NOT NULL, default now() | |
| updated_at | timestamptz | NOT NULL, default now() | |

**Triggers:** `set_updated_at`.
**RLS policies:** SELECT, INSERT (with_check), UPDATE (qual only — with_check: null), DELETE — all: `auth.uid() = user_id`.
**Known gap:** UPDATE with_check is null — row values not validated after update.

**Current data (verified from user query):**

| Name | Type | Balance | include_in_net_worth |
|------|------|---------|----------------------|
| Kotak Savings | bank | ₹0 | true |
| CSB Savings | bank | ₹250 | true |
| Cash in Hand | cash | ₹0 | true |
| Kotak FD | fd | ₹21,577 | true |
| Kotak Credit Card | credit_card | ₹18,900 (owed) | true |
| Groww Trading | trading | ₹7,800 | false |
| Crypto | crypto | ₹0 | false |

---

## TABLE: transactions

**Purpose:** All financial movements. Balance changes are trigger-managed, not application-managed.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| account_id | uuid | NOT NULL, FK → accounts ON DELETE CASCADE | source account |
| to_account_id | uuid | nullable, FK → accounts | destination (transfer type only) |
| type | transaction_type | NOT NULL | enum |
| category | text | NOT NULL | from CATEGORIES constant in JS |
| amount | numeric(12,2) | NOT NULL, check > 0 | |
| txn_date | date | NOT NULL, default current_date | |
| note | text | nullable | |
| investment_id | uuid | nullable, FK → investments | links buy/sell to holding |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**Balance trigger logic (all 3 functions are SECURITY DEFINER):**

| type | account_id effect | to_account_id effect |
|------|-------------------|----------------------|
| income | +amount (bank/cash) / -amount (CC) | n/a |
| expense | -amount (bank/cash) / +amount (CC) | n/a |
| transfer | -amount (bank/cash) / +amount (CC) | +amount (bank/cash) / -amount (CC) |
| investment_buy | -amount (bank/cash) / +amount (CC) | n/a |
| investment_sell | +amount (bank/cash) / -amount (CC) | n/a |
| emi_conversion | -amount (always — reduces CC owed balance) | n/a |
| emi_principal | -amount (always — reduces bank balance) | n/a |

**Triggers:** `on_transaction_insert` (AFTER INSERT → apply_transaction_to_balance), `on_transaction_update` (AFTER UPDATE → adjust_transaction_on_update — reverses old then applies new), `on_transaction_delete` (AFTER DELETE → reverse_transaction_from_balance), `set_updated_at` (BEFORE UPDATE).

**RLS policies:** All 4 operations: `auth.uid() = user_id`.
**Known gap:** account_id ownership not verified at DB level. UPDATE with_check null.

**Income categories (JS CATEGORIES constant):**
`Pocket Money`, `Gambling Winnings`, `Snooker Winnings`, `Windfall Gains`, `Stipend`, `Other Income`

**Expense categories (JS CATEGORIES constant):**
`Food & Dining`, `Groceries`, `Transport`, `Fuel`, `Shopping`, `Clothing`, `Personal Care`, `Medical`, `Entertainment`, `Subscriptions`, `Education`, `Mobile & Internet`, `Gifts & Donations`, `Miscellaneous`, `Gambling`, `Snooker`, `Windfall Loss`

---

## TABLE: bets

**Purpose:** Gambling/betting performance ledger. Not connected to accounts or transactions.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| bet_date | date | NOT NULL | |
| sport | text | NOT NULL | 'Cricket', 'Football', 'Tennis', 'Snooker (Bets)', 'F1', 'NBA', 'MLB', 'Other' |
| match | text | nullable | |
| stake | numeric(12,2) | NOT NULL | |
| odds | numeric(6,2) | NOT NULL | |
| outcome | bet_outcome | NOT NULL, default 'pending' | enum |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**No account balance effects.** P&L computed client-side as (stake × odds) - stake for wins.
**34 real historical bets imported** (2026-07-17 to 2026-08-06).
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: trades

**Purpose:** Trading performance ledger (closed trades only).
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| symbol | text | NOT NULL | e.g. 'RELIANCE', 'NIFTY' |
| quantity | numeric(12,4) | NOT NULL | |
| buy_price | numeric(12,2) | NOT NULL | |
| sell_price | numeric(12,2) | NOT NULL | |
| buy_date | date | NOT NULL | |
| sell_date | date | NOT NULL | |
| notes | text | nullable | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**P&L:** (sell_price - buy_price) × quantity — computed client-side, never stored.
**No account balance effects.** Capital movement tracked via transfer transactions.
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: crypto_trades

**Purpose:** Crypto performance ledger. Mirrors trades with higher quantity precision.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| symbol | text | NOT NULL | e.g. 'BTC', 'ETH' |
| quantity | numeric(18,8) | NOT NULL | higher precision for fractional crypto |
| buy_price | numeric(12,2) | NOT NULL | |
| sell_price | numeric(12,2) | NOT NULL | |
| buy_date | date | NOT NULL | |
| sell_date | date | NOT NULL | |
| notes | text | nullable | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: investments

**Purpose:** Investment holding master records (lot-tracked only; legacy mode removed).
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| name | text | NOT NULL | e.g. 'RELIANCE' (case-insensitive match on buy) |
| category | text | NOT NULL, default 'Stocks' | |
| cost_price | numeric(12,2) | NOT NULL | always 0 for lot-tracked holdings |
| purchase_date | date | NOT NULL | date of first lot |
| is_sold | boolean | NOT NULL, default false | true when remaining_qty ≈ 0 |
| sold_price | numeric(12,2) | nullable | [LEGACY FIELD — not used for lot-tracked] |
| sold_date | date | nullable | [LEGACY FIELD — not used for lot-tracked] |
| notes | text | nullable | |
| uses_lot_tracking | boolean | NOT NULL, default true | false = legacy (no records should exist) |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**Actual cost basis lives in investment_lots, not here.**
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: investment_lots

**Purpose:** Individual purchase lots. Enables multiple buys of same security with weighted-average cost tracking.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| investment_id | uuid | NOT NULL, FK → investments ON DELETE CASCADE | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| quantity | numeric(18,6) | NOT NULL | |
| price_per_unit | numeric(14,4) | NOT NULL | |
| purchase_date | date | NOT NULL | |
| account_id | uuid | nullable, FK → accounts | funding account |
| transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | linked investment_buy transaction |
| created_at | timestamptz | NOT NULL | |

**Remaining cost basis** = sum(quantity × price_per_unit) from lots - sum(cost_basis_removed) from sales.
**RLS policies:** All 4 operations: `auth.uid() = user_id`.
**⚠ NOT BACKED UP by the current backup function.**

---

## TABLE: investment_sales

**Purpose:** Individual sale events. Supports partial and multiple sales.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| investment_id | uuid | NOT NULL, FK → investments ON DELETE CASCADE | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| quantity_sold | numeric(18,6) | NOT NULL | |
| sale_proceeds | numeric(14,2) | NOT NULL | full cash received |
| cost_basis_removed | numeric(14,2) | NOT NULL | avg_cost × qty_sold |
| realized_pl | numeric(14,2) | NOT NULL | proceeds - cost_basis_removed (stored, not computed) |
| sale_date | date | NOT NULL | |
| receiving_account_id | uuid | nullable, FK → accounts | bank account credited |
| transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | linked investment_sell transaction |
| created_at | timestamptz | NOT NULL | |

**Cost method:** Weighted average (pooled across all lots for the holding).
**RLS policies:** All 4 operations: `auth.uid() = user_id`.
**⚠ NOT BACKED UP by the current backup function.**

---

## TABLE: loans

**Purpose:** Debtors (money owed to user) and creditors (money user owes). Net Worth impact.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| type | loan_type | NOT NULL | 'debtor' adds to NW, 'creditor' subtracts |
| counterparty | text | NOT NULL | name of person/entity |
| balance | numeric(12,2) | NOT NULL, default 0 | outstanding amount |
| notes | text | nullable | |
| is_closed | boolean | NOT NULL, default false | settled records excluded from Net Worth |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**Net Worth effect:** +debtors_total - creditors_total (open only).
**No automatic connection to transactions or account balances.**
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: emi_plans

**Purpose:** EMI plan master records.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| description | text | NOT NULL | |
| merchant | text | nullable | |
| credit_card_account_id | uuid | NOT NULL, FK → accounts | must be credit_card type |
| original_amount | numeric(12,2) | NOT NULL | |
| conversion_date | date | NOT NULL | |
| tenure_months | int | NOT NULL | |
| interest_rate | numeric(6,3) | NOT NULL, default 0 | annual % |
| monthly_emi | numeric(12,2) | NOT NULL | |
| processing_fee | numeric(12,2) | NOT NULL, default 0 | |
| gst_rate | numeric(5,2) | NOT NULL, default 18 | |
| first_emi_date | date | NOT NULL | |
| status | text | NOT NULL, default 'active' | 'active' / 'completed' / 'foreclosed' |
| notes | text | nullable | |
| conversion_transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | |
| processing_fee_transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**RLS policies:** All 4 operations: `auth.uid() = user_id`.
**⚠ NOT BACKED UP by the current backup function.**

---

## TABLE: emi_schedule

**Purpose:** Installment-level schedule. Auto-generated on plan creation via reducing-balance amortization.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| emi_plan_id | uuid | NOT NULL, FK → emi_plans ON DELETE CASCADE | |
| installment_number | int | NOT NULL | 1-based |
| due_date | date | NOT NULL | |
| opening_principal | numeric(12,2) | NOT NULL | |
| emi_amount | numeric(12,2) | NOT NULL | |
| principal_component | numeric(12,2) | NOT NULL | |
| interest_component | numeric(12,2) | NOT NULL | |
| gst_component | numeric(12,2) | NOT NULL | 18% of interest |
| closing_principal | numeric(12,2) | NOT NULL | |
| payment_status | text | NOT NULL, default 'pending' | 'pending' / 'paid' |
| paid_date | date | nullable | |
| principal_transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | emi_principal type |
| interest_transaction_id | uuid | nullable, FK → transactions ON DELETE SET NULL | expense type |
| created_at | timestamptz | NOT NULL | |

**RLS policies:** All 4 operations — ownership verified via emi_plans parent (existence subquery).
**⚠ NOT BACKED UP by the current backup function.**

---

## TABLE: rewards

**Purpose:** Loyalty points, credit card rewards, airline miles, shareholder perks tracking.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| brand | text | NOT NULL | e.g. 'HDFC Credit Card', 'IndiGo BluChip' |
| type | reward_type | NOT NULL | enum |
| balance | numeric(12,2) | NOT NULL, default 0 | points/miles count |
| expiry_date | date | nullable | |
| notes | text | nullable | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**No financial effect.** Expiry alerts computed client-side (<30 days = amber, past = red).
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: goals

**Purpose:** Manually tracked financial targets with progress bar.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| name | text | NOT NULL | |
| target_amount | numeric(12,2) | NOT NULL | |
| current_amount | numeric(12,2) | NOT NULL, default 0 | manually entered — no auto-link to accounts |
| target_date | date | nullable | |
| notes | text | nullable | |
| is_achieved | boolean | NOT NULL, default false | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**No financial effect. No account links.**
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: documents

**Purpose:** Document reference tracker. Stores name/category/link only — no file storage.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| name | text | NOT NULL | e.g. 'Health Insurance Policy' |
| category | text | NOT NULL, default 'Other' | Insurance / Investment Certificate / Tax / Identity / Other |
| url | text | nullable | Google Drive link etc. |
| notes | text | nullable | |
| created_at | timestamptz | NOT NULL | |
| updated_at | timestamptz | NOT NULL | |

**No financial effect.**
**RLS policies:** All 4 operations: `auth.uid() = user_id`.

---

## TABLE: account_reconciliations

**Purpose:** Manual reconciliation events. Comparison record only — never modifies financial data.
**RLS:** Enabled.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | uuid | PK | |
| user_id | uuid | NOT NULL, FK → auth.users ON DELETE CASCADE | |
| account_id | uuid | NOT NULL, FK → accounts ON DELETE CASCADE | |
| reconciliation_date | date | NOT NULL | |
| actual_statement_balance | numeric(14,2) | NOT NULL | user-entered from real statement |
| system_balance_at_time | numeric(14,2) | NOT NULL | snapshot of accounts.balance at time of entry |
| difference | numeric(14,2) | NOT NULL | system - actual (stored, not recomputed) |
| note | text | nullable | |
| created_at | timestamptz | NOT NULL | |

**No financial effect. Read-only history.**
**Expected balance reconstruction:** most recent reconciliation's actual_statement_balance + sum of txnDeltaForAccount() for all transactions after reconciliation_date.
**RLS policies:** All 4 operations: `auth.uid() = user_id`.
**⚠ NOT BACKED UP by the current backup function.**

---

## DATABASE FUNCTIONS

### handle_new_user()
- Type: trigger function
- Fires: AFTER INSERT on auth.users
- Action: inserts row into profiles with matching id

### handle_updated_at()
- Type: trigger function
- Fires: BEFORE UPDATE on all tables with updated_at
- Action: sets updated_at = now()
- Shared across all tables — not duplicated

### apply_transaction_to_balance()
- Type: SECURITY DEFINER trigger function
- Fires: AFTER INSERT on transactions
- Action: credit-card-direction-aware balance update on account_id (and to_account_id for transfers)
- Bypasses RLS (security definer)

### reverse_transaction_from_balance()
- Type: SECURITY DEFINER trigger function
- Fires: AFTER DELETE on transactions
- Action: exact mirror of apply — reverses the original effect

### adjust_transaction_on_update()
- Type: SECURITY DEFINER trigger function
- Fires: AFTER UPDATE on transactions
- Action: Step 1 — reverse old row's effect on old accounts. Step 2 — apply new row's effect on new accounts.

---

## NET WORTH FORMULA (verified from code)

```
Net Worth = 
  Σ signedAccountBalance(a) for accounts WHERE include_in_net_worth = true
  + Σ balance for loans WHERE type = 'debtor' AND is_closed = false
  - Σ balance for loans WHERE type = 'creditor' AND is_closed = false
  + (Σ quantity × price_per_unit from investment_lots)
  - (Σ cost_basis_removed from investment_sales)

where signedAccountBalance(a) = 
  credit_card → -balance
  all others → +balance
```

---

## LIQUIDITY FORMULA (verified from code)

```
Liquidity (Accounts page) = 
  Σ balance for accounts WHERE type IN ('bank', 'cash')
```

Note: labelled "Accounts Total" in UI. Matches Master Spec definition of Liquidity = Cash + Bank Accounts.

---

## BACKUP COVERAGE

| Table | Backed Up? |
|-------|-----------|
| accounts | ✅ |
| transactions | ✅ |
| bets | ✅ |
| trades | ✅ |
| loans | ✅ |
| investments | ✅ |
| rewards | ✅ |
| goals | ✅ |
| documents | ✅ |
| profiles | ❌ |
| crypto_trades | ❌ |
| investment_lots | ❌ ⚠ CRITICAL |
| investment_sales | ❌ ⚠ CRITICAL |
| emi_plans | ❌ |
| emi_schedule | ❌ |
| account_reconciliations | ❌ |

---

## KNOWN OPEN ISSUES

1. **Gambling/Snooker not modelled as capital allocation** — expenses appear in personal expense analytics incorrectly.
2. **investment_lots and investment_sales not backed up** — loss of these tables means permanent loss of actual cost basis history.
3. **transactions.account_id ownership not DB-enforced** — crafted API calls could affect other users' balances.
4. **UPDATE with_check null on all tables** — post-update row values not DB-validated.
5. **Multi-step operations not atomic** — investment buy, sell, EMI creation involve separate Supabase calls; partial failure leaves DB in inconsistent state.
6. **No Snooker capital-allocation module** — Snooker follows different model than Master Spec §7 describes.
