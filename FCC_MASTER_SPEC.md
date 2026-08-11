# FINANCIAL COMMAND CENTER — MASTER SPECIFICATION

## 1. PURPOSE

FCC is a long-term personal Financial Command Center.

It is designed to provide a complete view of:

* Personal finances
* Liquidity
* Total net worth
* Capital allocation
* Investments
* Trading
* Crypto
* Gambling
* Snooker
* Credit cards
* EMIs
* Loans / debtors / creditors
* Goals
* Financial performance
* Analytics
* Reconciliation

Priority order:

1. Financial correctness
2. Data integrity
3. Security
4. Maintainability
5. Analytics
6. UX
7. Visual polish

Do not add complexity merely for the sake of adding features.

---

# 2. PRIMARY FINANCIAL METRICS

## 2.1 TOTAL NET WORTH — DASHBOARD

The Dashboard's primary financial figure is:

**TOTAL NET WORTH**

It represents the user's complete economic position.

Conceptually:

**TOTAL ASSETS − TOTAL LIABILITIES**

Assets may include:

* Cash
* Bank accounts
* FD
* Investments
* Trading capital/value
* Crypto
* Gambling capital
* Snooker capital
* Debtors
* Other assets

Liabilities may include:

* Credit cards
* Creditors
* EMI/loan liabilities
* Other liabilities

The system must avoid double-counting capital and performance.

---

## 2.2 LIQUIDITY — ACCOUNTS PAGE

The Accounts page's primary financial figure is:

**LIQUIDITY**

Liquidity means readily accessible personal money.

Primary liquidity consists of:

**Cash + Bank Account Balances**

Liquidity should NOT automatically include:

* Investments
* FD
* Trading
* Crypto
* Gambling
* Snooker
* Debtors

Credit-card credit limits are not liquidity.

The Accounts page should clearly distinguish liquidity from Total Net Worth.

A user may have high Net Worth but low Liquidity.

---

# 3. FINANCIAL LAYERS

## LAYER 1 — PERSONAL FINANCE

Includes:

* Bank accounts
* Cash
* Credit cards
* FD
* Personal income
* Personal expenses
* Debtors
* Creditors
* Personal liabilities

---

## LAYER 2 — CAPITAL ALLOCATION

Includes:

* Trading
* Crypto
* Gambling
* Snooker
* Investments

Capital is allocated from personal finances into these activities/assets.

Capital allocation is NOT automatically a personal expense.

---

## LAYER 3 — PERFORMANCE

Each activity maintains its own performance.

Includes:

* Trading P&L
* Crypto P&L
* Gambling P&L
* Snooker P&L
* Investment realized P&L
* Investment unrealized P&L where supported

Performance must not be double-counted as personal income/expense.

---

# 4. TRADING

Trading is treated as a separately tracked business/activity.

Money can be funded from personal finances into Trading.

Example:

**Bank → Trading ₹5,000**

Result:

* Bank decreases ₹5,000
* Trading capital increases ₹5,000

This is NOT a personal expense.

Trading P&L is tracked separately.

Trading must track:

* Capital funded
* Capital withdrawn
* Trading activity
* Trading P&L
* Performance metrics

Trading P&L should not automatically become ordinary personal income/expense.

---

# 5. CRYPTO

Crypto follows exactly the same conceptual model as Trading.

Crypto is treated as a separately tracked business/activity.

Example:

**Bank → Crypto ₹5,000**

This is capital allocation, not personal expense.

Crypto must track:

* Capital funded
* Capital withdrawn
* Crypto activity
* Crypto P&L
* Performance metrics

Do not create a fundamentally different accounting philosophy for Crypto.

---

# 6. GAMBLING

Gambling follows the same capital-allocation model as Trading and Crypto.

Example:

**Bank → Gambling ₹5,000**

This is:

**Capital Funded = ₹5,000**

NOT:

**Personal Expense = ₹5,000**

Gambling maintains its own performance ledger.

It should track:

* Capital funded
* Capital withdrawn
* Bets
* Gambling P&L
* Performance metrics
* Current activity capital where applicable

Gambling funding and withdrawals must not automatically appear as personal income/expense.

---

# 7. SNOOKER

Snooker follows the same capital-allocation model.

Example:

**Bank → Snooker ₹5,000**

This is capital allocation.

Snooker must track:

* Capital funded
* Capital withdrawn
* Snooker activity
* Snooker P&L
* Performance metrics
* Current activity capital where applicable

Snooker funding and withdrawals must not automatically appear as personal income/expense.

---

# 8. COMMON ACTIVITY FORMULA

Trading, Crypto, Gambling and Snooker use:

**Opening Capital

* Capital Funded
  − Capital Withdrawn
* Activity P&L
  = Closing Activity Capital**

Where applicable, additional adjustments must be explicitly identified.

Capital flow and performance must remain separate concepts.

---

# 9. CAPITAL MOVEMENT

Money moving between personal finances and an activity is a capital movement.

Examples:

* Bank → Trading
* Bank → Crypto
* Bank → Gambling
* Bank → Snooker
* Trading → Bank
* Crypto → Bank
* Gambling → Bank
* Snooker → Bank

These are NOT ordinary personal income/expense transactions.

---

# 10. INVESTMENTS

Investments are personal holdings.

Investment purchases move money from a personal account into an investment holding.

Investment purchases are NOT ordinary expenses.

The system should preserve:

* Security name
* Symbol
* Quantity
* Purchase price
* Purchase date
* Cost basis
* Purchase history

Cost basis must remain historically accurate.

Current market value may be obtained through market-data APIs where appropriate.

Investment sales must:

* Record sale proceeds
* Reduce the appropriate cost basis
* Calculate realized gain/loss
* Credit the receiving personal account

Sale proceeds themselves are not automatically ordinary income.

Realized investment P&L is performance.

Partial sales must be supported.

---

# 11. TRANSACTIONS

The Transactions module must remain simple.

It should represent actual financial movements without requiring the user to understand accounting complexity.

Core transaction concepts include:

* Income
* Expense
* Transfer
* Investment Buy
* Investment Sell
* EMI-related movements
* Other necessary financial movements

Transfers are part of Transactions.

Do not unnecessarily complicate the transaction entry interface.

---

# 12. TRANSFERS

Transfers move money between accounts.

Example:

**Bank A → Bank B ₹10,000**

Result:

* Bank A −₹10,000
* Bank B +₹10,000

Transfers are neither income nor expense.

Transfers must not change Total Net Worth.

---

# 13. CREDIT CARDS

Credit cards are personal liabilities.

A credit-card purchase increases the amount owed.

A credit-card payment decreases the amount owed.

Credit-card liabilities contribute negatively to Total Net Worth.

Credit-card credit limits are not liquidity.

Credit-card accounting must not blindly use normal asset-account balance logic.

---

# 14. FD

An FD is a separate personal asset.

If an FD secures a credit card, the FD and credit card remain separate records.

Example:

FD +₹50,000

Credit Card −₹20,000

Net effect:

**+₹30,000**

Never merge the two into one account.

---

# 15. EMI

EMIs represent repayment of an existing liability.

Principal repayment is NOT a new expense.

Interest, GST and applicable fees are expenses.

The EMI system should support:

* EMI amount
* Principal
* Interest
* GST
* Fees
* Tenure
* Schedule
* Due dates
* Paid/unpaid status
* Outstanding principal
* Outstanding interest
* Outstanding charges

The system must prevent double-counting of the original purchase and subsequent principal payments.

---

# 16. LOANS / DEBTORS / CREDITORS

Money lent:

Bank −₹10,000
Debtor +₹10,000

This does not change Net Worth.

Money borrowed:

Bank +₹10,000
Creditor +₹10,000

This does not create ordinary income.

Principal repayments are capital movements.

Interest should be handled separately where applicable.

---

# 17. NET WORTH INVARIANT

Whenever money simply moves between assets:

**Net Worth should not change.**

Examples:

* Bank → Investment
* Bank → Trading
* Bank → Crypto
* Bank → Gambling
* Bank → Snooker
* Bank A → Bank B

These are movements of existing wealth.

Net Worth changes when economic value actually changes.

Examples:

* Activity profit/loss
* Investment gain/loss
* New income
* Genuine expense
* New liability
* Liability repayment

Never double-count the same economic value.

---

# 18. LIQUIDITY INVARIANT

Liquidity is:

**Cash + Bank Accounts**

The primary Accounts-page liquidity figure must not silently include illiquid assets or activity capital.

Liquidity and Net Worth are intentionally different metrics.

---

# 19. PERSONAL INCOME

Personal income should remain separate from activity capital.

Trading funding is not income.

Crypto funding is not income.

Gambling funding is not income.

Snooker funding is not income.

Investment sale proceeds are not automatically ordinary income.

Transfers are not income.

---

# 20. PERSONAL EXPENSES

Personal expenses represent genuine consumption/spending.

Trading funding is not an expense.

Crypto funding is not an expense.

Gambling funding is not an expense.

Snooker funding is not an expense.

Investment purchases are not expenses.

Transfers are not expenses.

EMI principal is not a second expense.

EMI interest/GST/fees are expenses.

---

# 21. ANALYTICS

Analytics must separate:

### Personal Cash Flow

* Income
* Expenses
* Net cash flow
* Savings rate

### Capital Allocation

* Trading funding
* Crypto funding
* Gambling funding
* Snooker funding
* Investment capital

### Performance

* Trading P&L
* Crypto P&L
* Gambling P&L
* Snooker P&L
* Investment realized P&L
* Investment unrealized P&L

Never combine these categories without clearly identifying the relationship.

---

# 22. DATABASE INTEGRITY

Financial operations should be atomic where multiple records are affected.

Examples:

* Transfers
* Investment purchases
* Investment sales
* EMI payments
* Loan movements
* Activity funding
* Activity withdrawals

Prefer:

**Everything succeeds OR nothing changes.**

---

# 23. SECURITY

RLS must protect every user-owned record.

Frontend restrictions are NOT security.

Foreign keys must not permit one user to reference another user's records.

Ownership must be enforced at the database level.

Relevant references include:

* account_id
* to_account_id
* investment_id
* loan_id
* EMI references
* activity references

---

# 24. BACKUP

FCC should have a free backup mechanism.

Backup must include all financial tables and preserve relationships required for restoration.

Whenever a new financial table is added, backup functionality must be considered.

---

# 25. RECONCILIATION

FCC should eventually support reconciliation between:

**Recorded Balance**

and

**Actual Balance**

Reconciliation should identify discrepancies without silently destroying historical transactions.

---

# 26. DEVELOPMENT PRINCIPLES

Prefer:

**Simple UI + sophisticated financial logic**

over:

**Complex UI + complicated user workflow**

Do not expose unnecessary accounting complexity to the user.

The Transactions module in particular should remain simple.

---

# 27. AI DEVELOPMENT RULES

Any AI working on FCC must:

1. Read this document first.
2. Inspect the actual repository.
3. Inspect the actual database/schema when relevant.
4. Never assume architecture.
5. Never silently change financial semantics.
6. Never modify unrelated modules.
7. Check for double-counting.
8. Trace account balance effects.
9. Trace Net Worth effects.
10. Trace liquidity effects.
11. Trace income/expense effects.
12. Check RLS/security implications.
13. Test create/edit/delete behaviour where relevant.
14. Report exactly what changed.

---

# 28. ARCHITECTURAL CHANGES

For major architectural changes:

1. Inspect current implementation.
2. Explain the problem.
3. Propose the design.
4. Explain financial consequences.
5. Identify database changes.
6. Identify migration risks.
7. Wait for approval.
8. Implement.
9. Test.
10. Document the change.

Never silently redesign the financial architecture.

---

# 29. GOLDEN RULE

Before implementing any financial feature, answer:

**Where did the money come from?**

**Where did it go?**

**Is this income, expense, transfer, capital allocation or performance?**

**What happens to account balances?**

**What happens to liquidity?**

**What happens to Total Net Worth?**

**Could this be double-counted?**

**Can the operation be reversed safely?**

**Can the database become inconsistent?**

If these questions cannot be answered clearly, stop and clarify the design first.

