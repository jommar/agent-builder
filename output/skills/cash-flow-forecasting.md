---
name: cash-flow-forecasting
description: Generate 7/30/90-day cash flow forecasts for a rental property portfolio using historical transaction data, recurring patterns, and known future obligations.
trigger: intent — triggered when the user asks about "cash flow", "runway", "forecast", "how long can we", or "when will we run out"
inputs:
  period_start: "string (YYYY-MM-DD) — start of historical window to analyze (default: 90 days ago)"
  period_end: "string (YYYY-MM-DD) — end of historical window (default: today)"
  forecast_horizon_days: "number — how many days to forecast (7, 30, or 90; default: 30)"
  property_id: "string (optional) — scope to single property; omit for portfolio-wide"
  scenario: "enum (optional) — best_case | base_case | worst_case (default: base_case)"
  known_future_items: "array (optional) — known future inflows/outflows: [{date, description, amount, type}] e.g., upcoming tax payment, expected rent increase"
outputs:
  forecast_table: "array — [{date, projected_balance, inflows, outflows, net_change}]"
  runway_days: "number — days until cash reaches zero (null if positive through horizon)"
  risk_flags: "array — [{severity: high|medium|low, date, description}] e.g., 'Mortgage payment exceeds projected balance on 2026-08-01'"
  summary: "object — {starting_balance, ending_balance, total_inflows, total_outflows, net_change, avg_monthly_burn}"
  comparison_to_prior: "object (optional) — variance from previous forecast for same period"
tools:
  - parse_financial_file
  - run_calculation
  - generate_report
  - save_progress
composes_with:
  - rent-roll-revenue-analytics
  - bookkeeping-reconciliation
---

# Cash Flow Forecasting

## Procedure

### Step 1: Gather Historical Data
1. Parse all available bank statement CSVs for the historical window via `parse_financial_file` (file_type: `bank_statement`).
2. Parse rent rolls for the same period via `parse_financial_file` (file_type: `rent_roll`) to identify expected rent vs. actual collections.
3. If prior forecasts exist, load them for variance comparison.
4. Collect known future items from the user: upcoming mortgage payments, property tax deadlines, insurance premiums, expected vacancies, planned capital improvements, known rent increases.

### Step 2: Identify Patterns
Use `run_calculation` to:
1. Group transactions by description to identify recurring items (monthly rent, mortgage, utilities, insurance).
2. Calculate average monthly inflows and outflows by category over the historical window.
3. Compute seasonal adjustments: compare month-over-month patterns. Flag Q4 heating costs, Q2 HVAC maintenance, vacancy seasonality.
4. Calculate daily burn rate: total outflows / days in period.
5. Identify one-time/non-recurring items and exclude from baseline projection.

### Step 3: Build Projection Model
Use `run_calculation` to:
1. Start with current bank balance (from most recent bank statement).
2. For each day in the forecast horizon:
   - Add projected inflows (recurring rent on known due dates, adjusted for vacancy risk)
   - Subtract projected outflows (recurring expenses, known future obligations)
   - Apply scenario multiplier:
     - `best_case`: 100% rent collection, lowest expense estimates
     - `base_case`: 95% rent collection, average expenses + 5% contingency
     - `worst_case`: 85% rent collection, highest expenses, add vacancy for 1 unit
3. Track running balance day by day.
4. Flag any day where projected balance drops below:
   - Mortgage payment due amount → **high risk**
   - 1 month of operating expenses → **medium risk**
   - 3 months of operating expenses → **low risk** (healthy, but monitor)

### Step 4: Run Scenarios
If `scenario` is `base_case` (default), also run `worst_case` and compare:
1. Identify the gap between base and worst case at the end of the forecast horizon.
2. Calculate "stress buffer" — how many months of expenses does each scenario leave?
3. Flag properties where worst-case runway < 3 months.

### Step 5: Generate Forecast Report
Use `generate_report` (report_type: `cash_flow`) with:
1. **Header:** Period, properties, scenario(s), generated timestamp.
2. **Summary KPIs:** Starting balance, projected ending balance, net change, monthly burn rate, runway in days/months.
3. **Forecast table:** Daily or weekly projected balances with inflow/outflow breakdown. For 90-day forecasts, aggregate to weekly; for 7/30-day, show daily.
4. **Risk flags:** Chronological list of dates where balance drops below thresholds.
5. **Scenario comparison:** Side-by-side base vs. worst case (if generated).
6. **Prior forecast comparison:** Variance from last forecast for the same period, if available.

### Step 6: Save and Present
1. Save checkpoint via `save_progress` with checkpoint name like `cash-flow-forecast-YYYY-MM-DD-{horizon}d`.
2. Present the report to the user. Highlight the top 3 risk flags.
3. If runway < 90 days in base case, explicitly recommend actions: "Consider reducing discretionary expenses, accelerating rent collection, or securing a line of credit."

## Edge Cases

- **No historical data (new portfolio):** Use known future items + manual estimates. Flag forecast as "low confidence — insufficient historical data." Recommend uploading at least 3 months of bank statements.
- **Single property with 1 tenant:** Amplified vacancy risk — if that tenant leaves, revenue drops to zero. Model this explicitly in worst-case scenario and flag prominently.
- **Seasonal rental (vacation property):** Adjust for high/low season. Do not project peak-season revenue across off-season months. Identify seasonal patterns and apply monthly multipliers.
- **Large one-time expense in forecast window** (roof replacement, HVAC upgrade): Model as single-day outflow on the expected date. Do not average it across months. Flag for cash reserve planning.
- **Mortgage escrow changes:** If the mortgage includes escrow for taxes/insurance, the payment may adjust annually. Ask user if an escrow analysis is expected soon and adjust projections.
- **Security deposit inflows:** Do NOT count security deposits as forecasted income — they are liabilities offset by future outflows. Only include non-refundable portions (e.g., deducted for damages).
- **Rent increase on lease renewal:** If a lease renews within the forecast window, apply the new rent amount from the renewal date forward. Ask the user for the new rate if unknown.

## Success Criteria

- Forecast correctly identifies all recurring transactions (rent, mortgage, utilities) with > 90% pattern recognition on a test set of 6+ months of rental data
- Runway calculation matches manual calculation within 2 days
- Worst-case scenario correctly models 1-unit vacancy and reduced rent collection rate
- Risk flags fire when projected balance drops below mortgage payment amount
- Report includes source attribution for all figures (which file, which calculation)
- Prior forecast variance is computed and presented when historical forecast data exists
