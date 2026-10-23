---
name: ytd-gains
description: Computes year-to-date gains on the stocks and ETFs in the member's OtterBudget portfolio, per asset in its trade currency, from the OtterBudget MCP tools. Use when the member asks for gains, returns or performance this year, YTD or year to date, for the portfolio, its stocks and ETFs, one ticker or one broker.
---

# Year-to-date gains

Compute Active YTD, Closed YTD and Total YTD for the stocks and ETFs of the connected OtterBudget portfolio, from 1 January to today, from the tools below. Give no figure that these steps do not produce.

## When it applies

- The ask is for gains, returns or performance this year, YTD or year to date, for the portfolio, its stocks and ETFs, one ticker or one broker. Answer it.
- The ask names another period. Answer "This computes the current calendar year only, from 1 January to today." and give no figure.
- The ask covers bonds or deposits. Give no figure for them and say "Bond and deposit gains are not computed here, because the OtterBudget tools do not give a bond's last coupon date or the deposits that matured and were archived this year; Gains YTD on the Dashboard includes them." An ask about the whole portfolio gets the stocks and ETFs answer with that sentence.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments access." and give no figure.

## Inputs

1. Held assets, their open lots and current prices: `list_holdings`, with its broker filter when the ask names one broker. Keep stocks and ETFs. Each asset has a trade currency and a `providerSymbol`, which is null when it has none.
2. Every sell of an asset and the units each one closed: `get_holding`, read in full. Count a sell by its own trade date, never by the day it was recorded. Leave out sells dated after today.
3. Sold-out assets with a sell this year: `list_holdings` does not list them. Read every sell once with `list_transactions` (side sell, no from bound, limit 500 on every read, one broker when the ask names one). Its from and to filter the recorded day, so a sell dated this year may have been saved earlier. Select this year's sells by each lot's tradeDate. Call `get_holding` for each asset a sell names that `list_holdings` did not give.
   - The read stops at 500 rows and sets `truncated`. When it does, read again once per broker.
   - When one broker's read is still truncated, split its from and to range, which filter by recorded day, until the read is complete or the range is one day.
   - When one broker on one day is still truncated, stop narrowing it and mark that broker Incomplete (below).
4. Year-end closes: `get_prices` with the `providerSymbol`s, `latestOnly` false, and a from and to range that covers the end of the previous year. Take the last close on or before 31 December. Call `get_prices` for no asset without a `providerSymbol`.

## Figures

Compute per asset in its trade currency. Add no currency effect, no bonds, no deposits and no YTD %.

- Reference price of a lot: the previous year-end close for a lot bought before 1 January; the buy price for a lot bought this year. A CFD uses its underlying's close.
- Active YTD: the sum over open lots of (current price - reference price) x units.
- Closed YTD: the sum over this year's sells of (sell price - reference price) x units sold. The units are the lots the sell closed, as `get_holding` gives them. Ignore the cost price override.
- Total YTD: Active YTD plus Closed YTD. Do not subtract fees or swap. Leave out dividends.
- A sold-out asset has an Active YTD of 0.
- CFD lots of an asset get a row of their own.
- One broker: count only that broker's lots and sells.

## Marks

Put the mark on the row and give each reason once under the table.

- Estimate: the asset has no close for the end of the previous year (no `providerSymbol`, or `get_prices` has none). Take the buy price as the reference of its lots open on 1 January. A close that `get_prices` marks estimated is still the reference; the row also reads Estimate.
- Stale: the row's current price is stale. Give that price's time.
- No price: the asset has no current price. Give its Closed YTD and no Active YTD, and say why.
- Incomplete, for a sell: `get_holding` reads a sell dated this year as unavailable. Leave it out of Closed YTD and name its date under the table.
- Incomplete, for sell discovery: for each broker whose read you stopped narrowing, say under the table that a sold-out asset with a sell this year at that broker may be missing. This is a line for the whole answer, not a row mark.

## The answer

1. Open with this sentence, and do not name the portfolio: "These figures cover the stocks and ETFs of this connection's portfolio, each in its trade currency, computed from your lots and closes, with no YTD % because the OtterBudget tools do not give the portfolio's value on 31 December."
2. Give one row per asset in scope, with Active YTD, Closed YTD and Total YTD, each amount with its currency code and its sign. A ticker ask gets that asset's rows only.
3. Group the rows by trade currency. Sort each group by Total YTD, largest first, and end it with its Total YTD subtotal. Add nothing across currencies.
4. For a whole-portfolio ask, add: "For one total in your reporting currency, with the currency effect, bonds and deposits, see Gains YTD on the Dashboard."
5. Give one "Prices as of" line: the time of the oldest current price and the session date of the year-end closes.
6. Give the reasons for the marks.
7. Show the working per lot only when the member asks for it.
