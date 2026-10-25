---
name: portfolio-briefing
description: "Briefs the member on their OtterBudget portfolio today or this week: value and change, the largest movers, stale prices, negative cash, lines off target and payouts due in 30 days. Use for today, this morning or this week. Never for this year (ytd-gains) or the watchlist (watchlist-review)."
---

# Portfolio briefing

Brief the member on the connected OtterBudget portfolio for today or for this week, from the tools below. This skill only reads. Give no figure that these steps do not produce, and none for this year.

## When it applies

- The ask is for today, this morning, this week, or what moved. Answer it. An ask for the week gets "The week" below; any other ask gets "Today".
- The ask is about this year or a past year. Hand over to `ytd-gains` and give no figure.
- The ask names another period, such as a month. Answer "This covers today and this week only." and give no figure.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.
- A tool says exchange rates are missing, or `list_holdings` gives no `total`. Say the briefing waits for FX rates and give no figure.

## Inputs

1. Value, units, prices and previous closes: `list_holdings`. Its `total` is the portfolio value in the reporting currency, cash and deposits included. Each holding has `units` (shares and CFD units together), `rate` (one unit of its trade currency in the reporting currency), `providerSymbol` and a `quote` with `price`, `previousClose`, `previousEstimated`, `stale` and `ts`. A `quote` of null means no price.
2. Cash: `list_cash_balances`, per broker and currency, archived brokers included.
3. Targets and caps: `list_allocation_targets`. Each target has `subject`, `level`, `sharePct`, `targetPct`, `bandPts`, `driftPts` and `flags`. `caps.above` lists the lines above a cap.
4. Payouts: `list_upcoming_payouts`, whose rows reach 31 December of next year (`to`). Keep the rows dated from today to 30 days after today.
5. For the week only: the closes of the last sessions. Call `get_prices` with the `providerSymbol`s of the holdings (50 at most per call), `latestOnly` false, from 21 days before today to today. Call it for no holding without a `providerSymbol`.

## Today

Compute per holding with a `quote` and a `previousClose`, in the reporting currency:

- Change in percent: (`quote.price` - `quote.previousClose`) ÷ `quote.previousClose`.
- Change in amount: `units` x (`quote.price` - `quote.previousClose`) x `rate`.
- The portfolio change since the previous close: the sum of those amounts. Its percent is the change ÷ (`total` - the change).

Rank the holdings by change in percent.

- Gainers: the three holdings with the largest change above 0, largest first.
- Losers: the three holdings with the largest fall below 0, largest fall first.
- A holding appears in one list only. With fewer than three, list the ones there are.

## The week

Use the closes from `get_prices`, in date order. A holding's current session is the date of its `quote.ts`, which is Friday's on a weekend. Its reference close is the fifth close dated before that session, counting back from it, so the reference is the close five sessions before the current session. Ignore a close dated on the current session or later.

- Change in percent: (`quote.price` - reference) ÷ reference, in the holding's trade currency.
- Change in amount: `units` x (`quote.price` - reference) x `rate`.
- The portfolio week change: the sum of those amounts, and its percent against (`total` - the change). Mark it Estimate, and give this reason: "It uses today's units and today's rates, so trades and currency moves during the week are not in it."
- Give every holding its row, ranked by change in percent. The gainers and the losers are chosen as for today, from the week changes. A holding with fewer than five closes, or no `providerSymbol`, gets the mark Incomplete and no week change.

## The other parts

The same for today and for the week.

- Stale prices: the holdings whose `quote.stale` is true, each with its `quote.ts`. Mark every row it feeds Stale: the holding's row and the portfolio change.
- Negative cash: every balance below 0, with its broker, currency and sign. Name an archived broker as archived. With none, say there is none.
- Lines off target: the targets whose `flags` hold `above_target` or `below_target`, and the lines in `caps.above`. Name each line by its `subject`, with its share, its target and band or its cap. A line in both lists is named once. With `waitingForFx` true, say the lines wait for FX rates.
- Payouts of the next 30 days: one row per payout, with its mark, date, name, broker or bank and amount in the reporting currency, with its own-currency amount beside it when it differs. Add the `amountReporting` of these rows for their total; do not take `total`, which counts every row to `to`. Leave out rows dated after 30 days from today and rows dated before today. Where `amountReporting` is null, give the own-currency amount, and say the total waits for FX rates. Amounts are gross.

## Marks

Put the mark on the row and give each reason once under the answer.

- Stale: the row's price is stale. Give that price's time.
- Estimate: `previousEstimated` is true, or a close from `get_prices` is marked stale, which means estimated; or a payout is marked `estimate`; or the portfolio week change, as above.
- No price: the holding has no `quote`. Leave it out of the change and the movers, and say so.
- Incomplete: a figure a step needs is missing, such as a previous close or a rate. Leave the row out of every figure that needs it and name it.
- A bond or a deposit has no day change. Say once that the change covers the holdings with a quote.

## The answer

1. Open with the total value in the reporting currency and the portfolio change since the previous close, or for the week, in amount and percent, each with its code and sign.
2. Give the gainers, then the losers: ticker, change in percent, and the amount beside it, ranked by percent.
3. Give the stale prices with their times.
4. Give the negative cash balances.
5. Give the lines off target or above a cap.
6. Give the payouts of the next 30 days.
7. Give one "Prices as of" line: the time of the oldest current price used.
8. Give the reasons for the marks.

Every amount carries its currency code, and every change its sign.

## Hand-offs

- Gains or performance this year or in a past year: `ytd-gains`.
- CFD exposure or concentration: `allocation-checkup`.
- The watchlist: `watchlist-review`.
- Coupons, maturities or payouts asked on their own: `upcoming-payouts`.
- One broker's cash balance: `cash-check`.
