---
name: rebalance-plan
description: Plans buys, and sells when the member asks, toward their own OtterBudget allocation targets, in whole units at the latest prices, and checks whether a planned buy crosses a target or a cap. Use for new money to invest, a rebalance or a planned buy's effect. Not for recording a trade (log-trade) or concentration alone (allocation-checkup).
---

# Rebalance plan

Plan whole-unit trades that move the connected OtterBudget portfolio toward its own allocation targets, or check what one planned buy does to a target or a cap. The plan is arithmetic on the member's targets. It saves nothing, sets no target and gives no advice.

## When it applies

- The ask is for new money to place, a rebalance, or where money should go. Ask first, every time, "Plan with new money only, or a full rebalance with sells?" and plan after the answer. A message that already says new money only, or a full rebalance with sells, answers it.
- The ask is a planned buy's effect on a target or a cap, such as buying 2,000 EUR of ASML. Ask nothing first and run "A planned buy" below.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.
- `list_allocation_targets` says it is waiting for exchange rates, or `get_allocation` refuses for missing rates. Say the plan waits for FX rates and give no figure.

## Inputs

1. Targets, bands, shares, drift, `capApplies` and the two caps: `list_allocation_targets`. A target's `amount` is its drift in money, negative below target.
2. Values per line and the portfolio total: `get_allocation`, with `by` set to the level (type, sector, geography, currency or holding). Its `portfolioTotal` is the whole portfolio, cash and deposits included.
3. Latest prices, units and the lines' assets: `list_holdings`. Each asset has `kind`, `sector`, `geography`, `currency` (its trade currency), `price`, `stale`, `quote.ts`, `rate` (one unit of its trade currency in the reporting currency), `weight` and its broker parts.
4. Cash: `list_cash_balances`, per broker and currency.

## The level

- Use the targets of the level the member names: type, sector, geography, currency or holding. Without a named level, use the type level.
- A level with no targets gets no plan. Say so, and say that a target is set in the app.
- The holdings of a line are the assets that belong to it: for a type, the assets of that `kind`; for a sector, geography or currency, the assets with that sector, geography or `currency`; for a holding, that asset. Plan no trade for a bond, a deposit or cash: name its gap and stop there.
- A line's gap is its target % of (the portfolio total plus the new money) minus its value. A line is below target when its gap is above 0 and above target when it is below 0.

## The amount

- Use the amount the member gave. Without one, give the cash per broker and currency from `list_cash_balances` and ask how much. When the mode is still open, ask both in one message. For a full rebalance, "none" is an answer.
- An amount in another currency than the reporting currency is converted at today's rate: take the `rate` of a `list_holdings` row in that currency, or, when none is held, call `list_holdings` with `reportingCurrency` set to that currency and divide 1 by the `rate` of a row in the reporting currency. Say the rate and the converted amount.

## New money only

1. Buy only for lines below target. When several are below target, split the money between the holdings of those lines by the holdings' own targets, where they have them, else in proportion to their current values. Name the split, so the member can change it.
2. No line gets more than its gap. Give the rest to the other lines by the same split.
3. Buy whole units: units = the money for the holding divided by (`price` x `rate`), rounded down. The cost is units x `price` x `rate`.
4. Give the cash left over, which is the amount minus all costs.

## A full rebalance with sells

1. Sell from lines whose `flags` hold `above_target`, down to their target. Split a line's sell between its holdings in proportion to their current values, and name the split. Sell whole units, rounded down, and no more than one broker part holds. When the asset sits at several brokers, name the parts and ask which one.
2. Buy for the lines below target as above, with the new money plus the sells' proceeds.
3. Under each sell, write "For this sell's tax, use sell-tax-preview." and give no tax figure.

## Buys that cross another level

Check every target and cap against all the plan's buys together, never one buy at a time. Two buys in the same sector can each stay under its cap and cross it together. For each line that holds any bought asset (the holding, its sector, its type, its geography and its trade currency), add the cost of all the plan's buys in that line. The buys raise only those lines.

- The line's share after the plan is (its value plus the cost of all the plan's buys in it) divided by (the portfolio total plus the new money). The total includes every buy, because the plan's money is new money. Keep the total unchanged only when the member says the money comes from a sale or from cash that the total already counts.
- A target is crossed when that share is above the target plus its band (`targetPct` plus `bandPts`).
- A cap is crossed when that share is above `holdingCapPct` for a holding or `sectorCapPct` for a sector, unless the line's target has `capApplies` false.
- Name each crossing, whatever level it is on: the line, its share after the plan, and the target and band or the cap it crosses.

## A planned buy

Run the crossing check above for the buys the member names, all together, and ask nothing about sells. Give each line's share before and after, and say which targets and caps the buys cross and which they do not. For a buy by amount, use the amount; for a buy by units, use units x `price` x `rate`.

## The answer

1. State the mode (new money only, or a full rebalance with sells), the level, and the amount with its conversion.
2. Give one row per trade: buy or sell, the asset, the whole units, the price, the cost or proceeds, each amount with its currency code. List sells first. Name no broker for a buy; log-trade asks for it.
3. Name the split between lines, as above.
4. Give the cash left over.
5. Give the crossings of other levels.
6. Say "Fees come when the trade is recorded."
7. End with "Record it" and say that each trade then goes to log-trade, which previews and confirms it.
8. Give one "Prices as of" line: the time of the oldest current price used.
9. Give the reasons for the marks.

## Marks

Put the mark on the row and give each reason once under the table.

- Stale: the row's price is stale. Give that price's time.
- No price: the asset has no price. Plan no units for it and say why.
- Estimate: the line is valued at cost, as a bond is.
- Incomplete: a figure a step needs is missing. Name it and give no figure that depends on it.

## Hand-offs

- Concentration, caps or drift without a plan: `allocation-checkup`.
- A currency move's effect on the portfolio: `allocation-checkup`.
- Setting or clearing a target: the server's guidance.
- Recording a trade the member made: `log-trade`.
- Which lots to sell, or a sell's tax: `sell-tax-preview`.
