# FCC AI DEVELOPMENT INSTRUCTIONS

You are an AI coding agent working on an existing Financial Command Center.

Before modifying anything:

1. Read `FCC_MASTER_SPEC.md`.
2. Read `FCC_DATABASE_SCHEMA.md`.
3. Read the relevant existing code.
4. Understand the current implementation rather than assuming it.

## CORE RULES

* Do not treat FCC as a blank-slate project.
* Do not rewrite working systems unnecessarily.
* Do not modify unrelated modules.
* Do not silently change financial logic.
* Do not introduce duplicate financial concepts.
* Do not create unnecessary database tables.
* Do not simplify financial logic merely to make implementation easier.
* Do not make the UI unnecessarily complicated.

## FINANCIAL MODEL

Trading, Crypto, Gambling and Snooker operate as capital-allocation activities/businesses.

The general flow is:

**Personal Finance → Capital Funded → Activity → Activity P&L → Capital Withdrawn → Personal Finance**

Funding an activity is NOT automatically a personal expense.

Activity P&L is NOT automatically personal income/expense.

The Dashboard shows:

**TOTAL NET WORTH**

The Accounts page shows:

**LIQUIDITY = CASH + BANK ACCOUNTS**

Do not double-count activity capital or performance.

## FINANCIAL VERIFICATION

For every financial change, determine:

* Source of money
* Destination of money
* Account balance effect
* Liquidity effect
* Net Worth effect
* Income/expense effect
* Capital allocation effect
* Performance effect
* Reversal behaviour
* Double-counting risk
* RLS implications

## BEFORE IMPLEMENTATION

For significant changes, provide:

1. Current implementation
2. Problem
3. Proposed solution
4. Files affected
5. Database changes
6. Financial consequences
7. Risks

Do not implement major architectural changes until explicitly approved.

## AFTER IMPLEMENTATION

Report:

* Files changed
* Database migrations
* Features implemented
* Tests performed
* Edge cases tested
* Remaining issues
* Assumptions made

If the existing code conflicts with `FCC_MASTER_SPEC.md`, stop and report the conflict instead of silently choosing an interpretation.

## MAINTAINABILITY

The FCC may be worked on by multiple AI systems.

Therefore:

* Keep documentation current.
* Do not delete architectural history.
* Make changes understandable to another developer/AI.
* Use reusable logic where appropriate.
* Avoid unnecessary duplication.
* Avoid hidden assumptions.

## SECURITY

Never rely solely on frontend restrictions.

Check:

* RLS
* Ownership
* Foreign keys
* Cross-user references
* Database functions
* Security-definer functions
* API-accessible operations

## CHANGE DISCIPLINE

If a requested change affects financial architecture, stop before implementation and explain the design.

Do not make a major architectural decision simply because it is easier to code.

