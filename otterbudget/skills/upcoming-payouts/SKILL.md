---
name: upcoming-payouts
description: Lists the coupons and maturities the member's OtterBudget bonds and deposits pay in the next 12 months, as the Dashboard's Upcoming payouts card does. Use for coupons, maturities and payouts. Not for gains (ytd-gains).
---

# Upcoming payouts

Show the coupons and maturities of the connected OtterBudget portfolio's bonds and deposits, as the Dashboard's Upcoming payouts card shows them. Read only. Give no figure that `list_upcoming_payouts` does not return.

## When it applies

- The ask is for coupons, maturities or payouts, such as "When is my next coupon?" or "What matures in the next 90 days?". Answer it.
- The ask names a maturity beyond the next 12 months, such as "When does my Fidelis 2031 bond mature?". Run "A maturity beyond 12 months" below.
- The ask is about dividends. Say that dividends are not tracked.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.

## Inputs

1. The rows: `list_upcoming_payouts`, called with no argument. It returns `currency` (the reporting currency), `from` (the day after today), `to` (31 December of next year, where the card ends), `total` (the sum of every row to `to`) and `waitingForFx`, and the rows in the card's order: by date, so the matured rows come first. Each row has `kind` (`coupon`, `bond_maturity` or `deposit_maturity`), `mark` (`coupon`, `matures` or `matured`), `date`, `name`, `isin`, `institutionName` (the broker or the bank), `units`, `amount` with `amountCurrency`, `amountReporting` and `estimate`.
2. A maturity beyond the rows, for a deposit: `list_holdings`, whose `deposits` give `maturityDate`. For a bond: `get_holding` with the ticker or ISIN, whose bond details give `maturityDate`.

## Which rows

The rows reach past 12 months, so cut them first. Count from today, which is the day before `from`.

- Default: every row dated up to 12 months after today (the same day of the month, or the last day of a shorter month), and every matured row. Leave out the rows dated later.
- A shorter window, such as "the next 90 days": the rows dated up to today plus that window, and every matured row.
- An ask about coupons alone or maturities alone: the rows of that kind (a bond or a deposit maturity), and the matured ones among them.
- Keep the card's order.
- An ask longer than 12 months, such as "the next 2 years": give the 12-month rows and say that this skill covers the next 12 months.
- A matured row does not count as a payout in the window. When no row other than a matured one is left, list the matured rows first, then say "No payouts in the next 12 months." or, for a window, that none falls in it.

## A maturity beyond 12 months

1. Find the asset or the deposit in `list_holdings` (a deposit) or call `get_holding` (a bond), and give its maturity date.
2. Give no amount.
3. Say that this answer covers 12 months, so this maturity is not in its list.

## The answer

1. Open with the rows, one per line, each with: the mark (Coupon, Matures or Matured), the date, the name, the broker or the bank, and the amount in the reporting currency. Add the amount in its own currency beside it when its `amountCurrency` differs from the reporting currency. A bond's row also gives its units.
2. A Matured row reads "Matured" and "awaiting your decision".
3. Give the total of the rows listed, in the reporting currency: the sum of their `amountReporting`, each already rounded to the cent. Do not take `total`, which also counts the rows after 12 months. It may differ from the card's "Total payout".
4. When `waitingForFx` is true, give the amounts in their own currency, give no reporting-currency amount and no total, and say that the payouts wait for FX rates.
5. Say: "The amounts are gross, deposit interest stays gross, no tax is computed and dividends are not tracked."
6. Mark an Estimate row on the row, and give its reason once under the answer: the amount of a bond maturity is estimated.
7. Every amount carries its currency code, and every change its sign. Add no amounts of different currencies yourself.
8. This skill uses no price, so it gives no "Prices as of" line.

## Hand-offs

- Interest or gains earned this year: `ytd-gains`.
- Months that savings would cover: `spending-check`.
- A repeating expense: `log-expense`.
