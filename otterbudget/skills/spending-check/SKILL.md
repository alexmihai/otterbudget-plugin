---
name: spending-check
description: Compares the member's OtterBudget spending with the budget for a month or the year to date, in their spending currency, and says how many months broker cash and deposits would cover. Use for budgets, spending and an emergency fund. Not for adding an expense (log-expense), investment gains (ytd-gains) or how much cash the member holds (cash-check).
---

# Spending check

Compare the connected spending profile's spending with its budget, in their spending currency, for a month or the year to date. Add the coverage line only when the ask is about a cushion, an emergency fund or months covered. Read only: this skill saves nothing. Give no figure that these steps do not produce.

## When it applies

- The ask is about budgets, spending by month or year, being over or under, an emergency fund or months covered. Answer it.
- The ask names no month. Use the current month in Europe/Bucharest. The ask names a month: use that month.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no spending profile or lacks spending:read. Answer "This connection cannot read a spending profile. Reconnect and approve spending:read, or ask a household admin for Spending read access." and give no figure.

## Inputs

Call the spending reads without `reportingCurrency`, so they answer in the member's spending currency. If the member names EUR, USD or RON for the answer, pass that one to every read of the answer that takes `reportingCurrency`.

1. The month: `get_spending_summary` with `month` as YYYY-MM. It gives `totalRon`, `yearToDateRon`, `yearlyBudgetRon` and, per category, `spent`, `budget`, `left` (null when the category has no budget), `yearToDate`, `yearlyBudget` and `lumpSum`. Each RON money field has a `…Reporting` twin: `totalReporting`, `yearToDateReporting`, `yearlyBudgetReporting` and, per category, `spentReporting`, `budgetReporting`, `leftReporting`, `yearToDateReporting`, `yearlyBudgetReporting` and `lumpSumReporting`. The answer gives `reportingCurrency`, `rate` (RON per unit), `rateAsOf` (the rate's fetch time) and `waitingForFx`. `lumpSum` is a category's yearly budget and `yearlyBudget` is its total budget (its twelve monthly amounts with its yearly budget).
2. The previous month, for the comparison: `get_spending_summary` for the month before.
3. The largest expenses of a category: `list_spending_transactions` with `month` and `category`. It lists 50 to a page; read every page up to `total`, then sort by `amountRon`. Each expense gives `amountReporting` and `reportingCurrency`; the answer also gives `rate`, `rateAsOf` and `waitingForFx`. When the ask names a category in its own words, match it to a name from `list_spending_categories`.
4. Repeating expenses, only when the ask is about them: `list_scheduled_expenses`.
5. The coverage line only: `get_allocation` with `type` all, `by` type and `reportingCurrency` from the summary. Its rows with key cash and key deposit give broker cash and deposits in that currency; a type with no row is 0. The read fails with `waiting_for_fx` when an exchange rate is missing. Also take `get_spending_summary` for each of the six months before the current month.

## Figures

All amounts are in the summary's `reportingCurrency`, from the `…Reporting` twins, with the currency code and, for a change, the sign.

- Class of a category, from the month's summary: monthly when `yearlyBudget` is above `lumpSum`; yearly-budget when `lumpSum` is above 0 and equals `yearlyBudget`; no budget when `yearlyBudget` is 0.
- Month spending: the sum of `spentReporting` over the monthly and no-budget categories. A yearly-budget category's spending is not in the month spending. Month budget: the sum of `budgetReporting` over the same categories.
- Left: budget minus spending. Below zero, give it as "<amount> over".
- A category over budget: its `leftReporting` is below 0. Its overage is the absolute value. A category whose `budget` is 0 is never over budget.
- Yearly-budget line: only while the year has a yearly-budget category. Give the yearly-budget categories' `spentReporting` this month, and the sum of their `yearToDateReporting` against the sum of their `lumpSumReporting` (the yearly budgets), with left or over, every time. When the year has no yearly-budget category, give no yearly-budget line.
- When the ask names a yearly-budget category, give its `yearToDateReporting` against its `lumpSumReporting`.
- This month against last month: this month's monthly-budget spending (the `spentReporting` of its monthly and no-budget categories, each month classed from its own summary) minus last month's, and that change over last month's. With no monthly-budget spending last month, give the amount and a dash for the percent. For the current month, say it is the month so far. The comparison with last month counts monthly-budget spending.
- With no budget for the month, give the amount spent and "No budget", with no left and no percent.
- Year to date: the profile's year to date is `yearToDateReporting`, with `yearlyBudgetReporting` beside it and the categories' `yearToDateReporting`. Give it only when the member asks for it.
- Coverage: (broker cash + deposits) divided by the average monthly spending of all categories (`totalReporting`) over the last six complete months, in months with one decimal. A complete month is a past calendar month; the current month is never one. Months before the first month with any spending do not exist yet. When fewer than six complete months exist, average the complete months there are and name how many. When none exists, give no coverage line and say that no complete month of spending exists yet.

## Marks

- If any read gives `waitingForFx` true, use the RON fields for the whole answer, call `get_allocation` with `reportingCurrency` RON, and say once under the answer: "No `<pair>` rate is stored yet, so amounts are in RON." Take `<pair>` from that read's `reportingCurrency`, such as USD/RON. Use `totalRon`, `yearToDateRon` and `yearlyBudgetRon` and the categories' `spent`, `budget`, `left`, `yearToDate`, `yearlyBudget` and `lumpSum` in place of their twins; for expenses use `amountRon`. Never mix RON fields and reporting twins in one answer.
- Stale: a `get_allocation` row is stale. Mark the coverage line Stale and give the read's `asOf` time. Give the reason once under the answer.
- Incomplete: `get_allocation` fails with `waiting_for_fx`, or a read of a month fails. Give no coverage line, and say which read failed.

## The answer

1. Open with the month against its budget: the month spending, the month budget, and left or over.
2. List the categories over budget, largest overage first, each with its overage and its three largest expenses (date, note, `amountReporting` with the answer's currency). Say so when none is over.
3. When the year has a yearly-budget category, give the yearly-budget line.
4. Give this month against last month, and say that the comparison counts monthly-budget spending.
5. Give the profile's year to date only when asked.
6. Give the coverage line only when the ask is about a cushion, an emergency fund or months covered. It gives broker cash, deposits, the average monthly spending with the number of months it averages, and the months covered. Say that current accounts and physical cash are not counted and that a deposit may be locked until it matures. Add one "Prices as of" line with the read's `asOf` time.
7. When the connection has a spending profile but lacks portfolio:read or has no portfolio, give the spending part and, in place of the coverage line, "The coverage line needs portfolio:read too."
8. Give the reasons for the marks once, under the answer.

## Hand-offs

- Adding, scheduling, changing, deleting or stopping an expense: `log-expense`.
- How much cash the member holds, or where: `cash-check`.
