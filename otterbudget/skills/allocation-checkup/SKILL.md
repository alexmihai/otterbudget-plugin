---
name: allocation-checkup
description: "Reads where the member's OtterBudget portfolio is concentrated or off its targets: caps, drift, the largest holdings and sectors, CFD exposure and currency shares. Use for concentration, targets, caps, CFD exposure or a currency move. Not for what to buy or sell (rebalance-plan)."
---

# Allocation checkup

Read where the connected OtterBudget portfolio is concentrated or off its targets, from the tools below. This skill only reads. Give no figure that these steps do not produce. Shares are percent of the whole portfolio, cash and deposits included.

## When it applies

- The ask is for concentration, caps, targets, drift, CFD exposure, currency shares or a currency move. Answer it.
- The ask is for what to buy or sell, or a planned buy's effect. See the first hand-off below.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.
- A tool says exchange rates are missing. Say so and give no share, drift or currency figure.

## Inputs

1. Caps, targets, drift and signals: `list_allocation_targets`. It gives the two caps with each line above them, every target with its drift in points (`driftPts`) and its `amount`, and the concentration signals (largest holding, top five's share, largest sector).
2. Shares by currency: `get_allocation` with type `all` and `by` `currency`.
3. CFD lots, quotes and prices: `list_holdings`. Each broker part of a holding has a `cfd` block (notional, value, unrealized) and lots with their margin. A CFD lot has instrument `cfd`.
4. Themes: `list_themes`, only when the ask names a theme. Give that theme's value and share.
5. Cash: `list_cash_balances`, only when the ask is about cash.

## Figures

- Targets out of band: the targets whose flag is `above_target` or `below_target`. Keep the drift and the amount as the tool gives them, with their signs.
- CFD exposure, per holding and per broker: notional, margin (the sum of the margins of its CFD lots) and unrealized, each in the holding's trade currency. Add nothing across currencies.
- A currency move, only when the ask names one: effect = share of that currency x the move x the portfolio total, in the reporting currency, at constant prices. A fall is negative. The reporting currency's own share has no effect. Give the effect and say that prices do not change in the scenario.
- Without a named move, give no scenario.

## Marks

Put the mark on the row and give each reason once under the answer.

- Stale: the row's price is stale. Give that price's time.
- No price: the asset has no current price. Say why, and leave its figure out of the CFD exposure.
- Incomplete: a part that the tools did not give, such as a CFD lot without a margin. Leave it out of the figure and name it.
- A bond line is valued at cost. Say so once.

## The answer

Give these in this order. Leave out a part with no lines, and say that it has none.

1. The lines above a cap, largest excess first: name, share, cap, points above and amount.
2. The targets out of band, largest drift first: name, target and band, share, drift in points and amount.
3. The concentration signals: the largest holding, the top five's share and the largest sector.
4. CFD exposure per holding and broker.
5. The shares by currency, largest first.
6. When the ask names a currency move, its effect.
7. One "Prices as of" line: the time of the oldest current price.
8. The reasons for the marks.
9. End with: "ETFs are not looked through, and themes can overlap, so a share can count the same exposure twice."

Every amount carries its currency code, and every change its sign.

## Hand-offs

- New money to place, a rebalance or a planned buy's effect: `rebalance-plan`.
- Setting or clearing a target: the server's guidance.
