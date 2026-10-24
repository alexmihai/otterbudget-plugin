---
name: watchlist-review
description: "Reviews the member's OtterBudget watchlist: items near their target price, large 30-day moves, stale quotes and missing or old theses. Use for the watchlist. Not for the holdings' moves (portfolio-briefing)."
---

# Watchlist review

Flag the items of the connected OtterBudget watchlist that need attention, from the tool below. Give no figure that these steps do not produce. This skill reads only.

## When it applies

- The ask is for the watchlist: items near their target, watchlist moves, missing or old theses. Answer it.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.

## Inputs

1. The items: `list_watchlist`, with its label or sector filter when the ask names one. Each item has a `target`, a `quote` (null when it has no usable price), `change30d` (null, or `pct` in percent with `since` and `estimated`), `range52w`, `thesis` and `thesisChangedAt`. Use today's date for every age.
2. One flagged item the member asks about: `get_research_dossier` with its ticker and exchange. Summarise the latest session. For the full text of a session, `get_research_entry` with its id.

## Flags

An item gets every flag it meets, in this order:

1. Near target: the item has a `target` and a `quote`, and the price is within 5% of the target, above or below it. The distance is |price - target| ÷ target, in the item's currency.
2. 30-day move: `change30d.pct` is above +10 or below -10. Give the move with its sign. When `change30d` is null, the item has no 30-day move: do not flag it, and say under the answer that its 30-day move is missing.
3. Stale quote: `quote.stale` is true. Give the quote's time (`quote.ts`).
4. Thesis: the item has no `thesis`, or `thesisChangedAt` is more than 180 days before today. Give the date. When the thesis has no change date, say so.

An item without a `target` cannot be near target. An item with no `quote` gets the mark No price and only the thesis flag.

## Marks

Put the mark on the row and give each reason once under the answer.

- Stale: the price is stale, as flag 3 says.
- No price: the item has no usable price. Say why.
- Estimate: `change30d.estimated` or `range52w.estimated` is true. Put the mark on that figure.
- Incomplete: `range52w` is `partial`, or the item has none. Say the range is missing or partial.

## The answer

1. Give one row per flagged item: its ticker and name, then its flags in the order above, each with its figure. Sort the rows by their first flag in that order, then by ticker.
2. Give each flagged item its place in its 52-week range: where the price sits between `range52w.low` and `range52w.high`, in percent, with the two prices. The range covers closes, not intraday prices.
3. Say that a target crossing is not a buy instruction.
4. Say how many items have no flag, without listing them.
5. Every amount carries its currency code; a change carries its sign.
6. Give one "Prices as of" line: the time of the oldest quote of the flagged items.
7. Give the reasons for the marks.
8. When the member asks about one flagged item, read its dossier and summarise its latest session: the date, the summary and the open questions. Do not judge whether its thesis still holds.

## Hand-offs

- The holdings' moves today or this week: `portfolio-briefing`.
- Whether a thesis still holds: the server's guidance.
