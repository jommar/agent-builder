---
name: rent-roll-revenue-analytics
description: Track rent collection, occupancy rates, late payments, and per-property revenue trends — identify problem tenants and underperforming properties.
trigger: intent — triggered when the user asks about "rent roll", "occupancy", "vacancy", "revenue", "late rent", "tenant", "collection rate"
inputs:
  period_start: "string (YYYY-MM-DD) — start of analysis window"
  period_end: "string (YYYY-MM-DD) — end of analysis window (default: today)"
  property_id: "string (optional) — scope to single property; omit for portfolio-wide"
  include_tenant_details: "boolean (optional) — include per-tenant breakdowns (default: true)"
outputs:
  rent_roll_summary: "object — {total_expected_rent, total_collected, collection_rate_pct, outstanding, late_fees_assessed}"
  occupancy: "object — {total_units, occupied_units, vacant_units, occupancy_rate_pct, avg_vacancy_days}"
  per_property_breakdown: "array — [{property_id, name, units, occupied, rent_collected, rent_expected, collection_rate, flags}]"
  per_tenant_breakdown: "array — [{tenant_name, unit, property_id, monthly_rent, ytd_paid, ytd_owed, late_payments_count, days_outstanding, status: current|late|delinquent|vacated}]"
  revenue_trends: "array — [{month, total_revenue, rent_revenue, other_revenue, yoy_change_pct, mom_change_pct}]"
  flags: "array — [{severity: high|medium|low, type, description}] e.g., 'Unit B: 45 days past due — consider pay-or-quit notice'"
tools:
  - parse_financial_file
  - categorize_transaction
  - manage_property
  - run_calculation
  - generate_report
composes_with:
  - cash-flow-forecasting
  - bookkeeping-reconciliation
---

# Rent Roll & Revenue Analytics

## Procedure

### Step 1: Load Property & Tenant Data
1. Query all properties via `manage_property` (action: `query`) to get current tenant rosters, lease terms, and monthly rent amounts.
2. For each property, build an expected rent schedule: `{ tenant, unit, monthly_rent, due_date }`.
3. If properties don't exist yet in the system, ask the user to provide a rent roll spreadsheet and create properties via `manage_property` (action: `create`).

### Step 2: Parse Actual Collections
1. Parse bank statement CSVs for the analysis period via `parse_financial_file` (file_type: `bank_statement`).
2. Filter to credits (deposits).
3. Use `categorize_transaction` to identify which credits are rent payments and which tenant made them.
4. Match each rent payment to the expected rent schedule by tenant name and amount.

### Step 3: Calculate Collection Metrics
Use `run_calculation` to compute for each property and the portfolio:

**Per-property:**
```
expected_rent = sum of all tenant monthly rents for months in period
collected_rent = sum of matched rent payment transactions
collection_rate = (collected_rent / expected_rent) * 100
outstanding = expected_rent - collected_rent
```

**Per-tenant:**
```
ytd_paid = sum of payments matched to this tenant
ytd_owed = sum of monthly rents for months in period
days_outstanding = days since oldest unpaid rent was due
late_payments_count = number of months where payment was > 5 days after due date
status:
  - "current" if all rent paid within 5 days of due date
  - "late" if unpaid rent exists but < 30 days past due
  - "delinquent" if unpaid rent > 30 days past due
  - "vacated" if tenant moved out during period
```

**Portfolio occupancy:**
```
occupancy_rate = (occupied_units / total_units) * 100
avg_vacancy_days = average days vacant across all vacant units
revenue_per_occupied_unit = total_collected / occupied_units
```

### Step 4: Analyze Revenue Trends
Use `run_calculation` to:
1. Aggregate revenue by month (rent revenue + other revenue like late fees, pet fees, parking).
2. Compute month-over-month (MoM) and year-over-year (YoY) change percentages.
3. Identify seasonal patterns: compare same-month revenue across years.
4. Calculate trailing 12-month (TTM) revenue and growth rate.

### Step 5: Flag Issues
Identify and flag:
1. **Delinquent tenants:** > 30 days past due → **high severity**. Include recommendation: "Consider pay-or-quit notice per state law."
2. **Late tenants:** 5-30 days past due → **medium severity**. Include: "Send late payment reminder."
3. **Vacant units > 30 days:** → **medium severity**. Include: "Unit has been vacant {N} days. Consider rent reduction or increased marketing."
4. **Declining collection rate:** < 90% collection rate for 2+ consecutive months → **medium severity**.
5. **Revenue decline:** YoY revenue drop > 10% → **medium severity**. Investigate cause.
6. **Upcoming lease expirations:** Leases ending within 60 days → **low severity**. Include renewal reminder.
7. **Rent below market:** If a unit's rent is > 10% below portfolio average per unit → **low severity**. Flag for increase at renewal.

### Step 6: Generate Rent Roll Report
Use `generate_report` (report_type: `rent_roll`) with:
1. **Header:** Period, properties, generated timestamp.
2. **Portfolio Summary:** Total units, occupancy rate, collection rate, total collected, outstanding.
3. **Per-Property Table:** Property name, units, occupied, rent collected, collection rate, flags.
4. **Per-Tenant Table:** Tenant, unit, monthly rent, YTD paid, YTD owed, late payments, status.
5. **Revenue Trends:** MoM/YoY table with sparkline indicators.
6. **Flags & Recommendations:** Prioritized by severity.

### Step 7: Present
1. Present the report, highlighting the top 3 issues by severity.
2. If delinquent tenants exist, surface legal options (pay-or-quit, eviction) as informational — do NOT provide legal advice. Recommend consulting a landlord-tenant attorney.

## Edge Cases

- **No properties configured:** Return guidance on how to set up properties and tenants via `manage_property`. Offer to walk through setup step by step.
- **No rent roll uploaded:** Cannot compute collection rate or tenant status. Return revenue analysis from bank statements only. Flag "Collection rate unavailable — no rent roll data."
- **Tenant with same name across properties:** Disambiguate by property + unit. If a payment matches two possible tenants equally, flag for human review.
- **Partial payment:** Tenant paid less than full rent. Split: mark paid portion as collected, remaining as outstanding. Flag for follow-up.
- **Prepayment:** Tenant paid future months in advance. Credit to future periods, do not count as current month overpayment.
- **Security deposit used as last month's rent:** If tenant's last payment matches security deposit amount and tenant has vacated, do NOT count as rental income. Flag for security deposit reconciliation.
- **New tenant mid-month:** Prorate expected rent for partial month. Calculate collection rate based on prorated amount.
- **Owner-occupied unit:** Exclude from occupancy and collection calculations. Flag as "owner-occupied — excluded from metrics."
- **Vacation/short-term rental property:** Occupancy is measured in nights, not units. Adapt metrics accordingly if property_type is not standard residential.

## Success Criteria

- All rent payments correctly matched to tenants by name and amount with > 95% accuracy on test data
- Collection rate calculation matches manual calculation within 0.5%
- Occupancy rate correctly counts occupied vs. total units
- Delinquent tenants (30+ days) correctly identified and flagged as high severity
- Late tenants (5-30 days) correctly identified and flagged as medium severity
- Revenue trends compute correct MoM and YoY change percentages
- Report includes source attribution for all figures
- Upcoming lease expirations (within 60 days) are flagged
