---
name: bookkeeping-reconciliation
description:
  Perform month-end bank reconciliation for a rental property portfolio — match bank transactions to
  the general ledger, flag unmatched items, suggest adjusting journal entries, and verify the
  accounting equation.
trigger:
  explicit — invoked by command ("reconcile", "month-end close", "close the books") or intent
  ("reconcile accounts", "match transactions")
inputs:
  period: "object — {start: YYYY-MM-DD, end: YYYY-MM-DD} (default: previous full calendar month)"
  bank_statement_files: "array — paths to bank statement CSVs for the period"
  property_id: "string (optional) — scope to single property; omit for portfolio-wide"
  prior_reconciliation:
    "object (optional) — previous reconciliation state to carry forward unmatched items"
  gl_data:
    "object (optional) — current general ledger state; if not provided, build from staged journal
    entries and property records"
outputs:
  reconciliation_summary:
    "object — {total_bank_transactions, matched_count, unmatched_count, match_rate_pct,
    beginning_balance, ending_balance, gl_balance, difference}"
  matched_items: "array — [{bank_transaction, gl_entry, match_confidence, match_method}]"
  unmatched_bank: "array — [{date, description, amount, type}] — bank transactions with no GL match"
  unmatched_gl:
    "array — [{date, description, debit, credit, account}] — GL entries with no bank match"
  suggested_entries:
    "array — [{type: adjusting|correcting, journal_entry}] — staged entries to resolve differences"
  accounting_equation_check:
    "object — {assets, liabilities, equity, balanced: true|false, difference}"
  flags: "array — [{severity, type, description}]"
tools:
  - parse_financial_file
  - categorize_transaction
  - stage_journal_entry
  - run_calculation
  - manage_property
  - generate_report
  - save_progress
composes_with:
  - cash-flow-forecasting
  - rent-roll-revenue-analytics
---

# Bookkeeping Reconciliation

## Procedure

### Step 1: Load Period Data

1. Parse all bank statement CSVs for the period via `parse_financial_file` (file_type:
   `bank_statement`).
2. Parse any rent rolls for the period via `parse_financial_file` (file_type: `rent_roll`) to
   cross-reference expected vs. collected rent.
3. Load prior reconciliation state if available (carry forward unmatched items).
4. Load property records via `manage_property` (action: `query`) for all properties in scope.
5. Build a current-period GL from:
   - Previously staged and approved journal entries for the period
   - Depreciation schedule for each property
   - Mortgage amortization schedules (interest vs. principal split)

### Step 2: Match Transactions to GL

For each bank transaction, attempt to match to a GL entry using these methods in priority order:

**Method 1 — Exact match (amount + date ± 2 days):**

```
If bank.amount == gl_entry.amount AND abs(bank.date - gl_entry.date) <= 2:
    match_confidence = 0.99, match_method = "exact"
```

Use `run_calculation` for matching logic.

**Method 2 — Description match (amount + keyword):**

```
If bank.amount == gl_entry.amount AND (bank.description contains tenant name OR vendor name from gl_entry.description):
    match_confidence = 0.90, match_method = "description"
```

**Method 3 — Fuzzy match (amount within 5% + date within 7 days):**

```
If abs(bank.amount - gl_entry.amount) / max(bank.amount, gl_entry.amount) <= 0.05 AND abs(bank.date - gl_entry.date) <= 7:
    match_confidence = 0.70, match_method = "fuzzy"
```

Flag fuzzy matches for human review.

**Unmatched:** Any transaction not matched by method 1-3 goes to `unmatched_bank` or `unmatched_gl`.

### Step 3: Categorize Unmatched Items

For each unmatched bank transaction:

1. Use `categorize_transaction` to determine the likely account.
2. Identify the reason for the mismatch:
   - **Timing difference:** Bank transaction hasn't been journaled yet (outstanding checks, deposits
     in transit)
   - **Missing entry:** Transaction was never recorded in GL — needs a journal entry
   - **Bank error:** Duplicate charge, incorrect amount — flag for dispute
   - **GL error:** Entry recorded with wrong amount or date — needs correction

For each unmatched GL entry:

1. Determine if it's:
   - **Outstanding check:** Payment recorded in GL but not yet cleared bank
   - **Deposit in transit:** Rent recorded but not yet deposited
   - **Duplicate entry:** Same transaction recorded twice — needs reversal
   - **Erroneous entry:** Wrong amount, wrong account, or shouldn't exist

### Step 4: Generate Adjusting Entries

For each unmatched item requiring a journal entry, create a `stage_journal_entry`:

**Adjusting entries** (missing transactions):

- Rent received but not recorded → debit cash (1000), credit rental income (4000)
- Expense paid but not recorded → debit expense account (50xx), credit cash (1000)
- Bank fee not recorded → debit bank fees (5140), credit cash (1000)

**Correcting entries** (wrong amounts/dates/accounts):

- Wrong amount → reverse incorrect entry + post correct entry
- Wrong account → reclassify to correct account
- Duplicate → reverse the duplicate

**Recurring entries not yet recorded:**

- Monthly depreciation: debit depreciation expense (5130), credit accumulated depreciation (1220)
- Accrued interest: debit mortgage interest (5000), credit accrued expenses (2300)

### Step 5: Verify Accounting Equation

Use `run_calculation` to verify the accounting equation after all adjusting entries:

```
assets = sum of all asset account balances after adjustments
liabilities = sum of all liability account balances after adjustments
equity = sum of all equity + revenue - expense account balances after adjustments

balanced = abs(assets - (liabilities + equity)) < 0.01
```

If NOT balanced:

1. List all accounts with their ending balances.
2. Compute the difference: `assets - (liabilities + equity)`.
3. Flag as **critical** — the books are out of balance. Do NOT proceed with month-end close until
   resolved.
4. Attempt to identify the likely cause: missing offset in a journal entry, incorrect sign on an
   account, uncategorized transaction posted to wrong balance type.

### Step 6: Generate Reconciliation Report

Use `generate_report` (report_type: `reconciliation`) with:

1. **Header:** Period, properties, bank accounts, generated timestamp.
2. **Balance Summary:** Beginning balance (bank), ending balance (bank), GL balance, difference,
   match rate.
3. **Matched Items:** Count by match method. Highlight fuzzy matches for review.
4. **Outstanding Items:**
   - Outstanding checks: date, payee, amount, days outstanding
   - Deposits in transit: date, source, amount, days since deposit
5. **Unmatched Bank Transactions:** Full list with suggested categorization.
6. **Unmatched GL Entries:** Full list with suggested resolution.
7. **Suggested Journal Entries:** All staged entries with amounts, accounts, and rationale.
8. **Accounting Equation Verification:** Pass/fail with balances.
9. **Carry-Forward Items:** Unmatched items that persist to next period.

### Step 7: Present and Request Approval

1. Present the reconciliation report.
2. For each suggested journal entry: "I've staged {count} journal entries. Please review and
   approve."
3. If accounting equation is not balanced: "⚠️ CRITICAL: Books are out of balance by ${difference}.
   Cannot proceed with close until resolved."
4. If all is balanced and all entries are staged: "Month-end reconciliation complete. {match_rate}%
   of transactions matched. {unmatched_count} items require attention. All journal entries are
   staged for your review."
5. Save progress checkpoint via `save_progress` with checkpoint name `reconciliation-YYYY-MM`.

## Edge Cases

- **First reconciliation (no prior state):** No carry-forward items. Beginning balance comes from
  first bank statement of the period. Flag "Initial reconciliation — establishing baseline."
- **Zero unmatched items:** Perfect reconciliation. Flag as "Clean reconciliation — 100% match rate.
  Well done." Still verify accounting equation.
- **Large outstanding check (> 60 days):** Flag as stale — the payee may have lost the check.
  Recommend contacting payee or issuing a replacement.
- **Bank error (unauthorized charge, wrong amount):** Flag for immediate dispute. Do NOT create an
  adjusting journal entry — the bank must correct this. Create a memo entry tracking the disputed
  amount.
- **NSF / bounced rent check:** Debit accounts receivable (1100), credit cash (1000) to reverse the
  deposit. Flag tenant as delinquent. Add NSF fee from bank as expense (5140).
- **Inter-account transfer:** Transfer between operating (1000) and security deposit (1010)
  accounts. Both sides appear as bank transactions — match them to each other, not to GL
  income/expense entries. Flag as internal transfer.
- **Mortgage escrow disbursement:** Bank shows payment to tax assessor or insurance company from
  escrow. This is NOT an expense — the expense was already recorded monthly as part of the mortgage
  payment. No journal entry needed; flag as informational only.
- **Security deposit refund:** Tenant vacated, deposit returned. Debit security deposits held
  (2000), credit cash (1000). This is NOT an expense. Verify that any deductions (damages) are
  properly reclassified to income or offset against repairs expense.
- **Reconciliation difference is exactly a round number:** If the difference is $100, $1000, etc.,
  suspect a transposition error or a missing entry of that exact amount. Flag for human
  investigation.

## Success Criteria

- 100% of bank transactions matched or explicitly classified as unmatched with a documented reason
- Accounting equation verified as balanced (difference < $0.01) after all adjusting entries
- All suggested journal entries are balanced (debits = credits) before staging
- Fuzzy matches (confidence < 0.90) are explicitly flagged for human review
- Outstanding checks > 60 days are flagged as stale
- Security deposit refunds are correctly posted to liability reduction, not expense
- Inter-account transfers are excluded from income/expense matching
- Prior reconciliation unmatched items are carried forward and re-attempted
- Report includes source attribution for all figures
