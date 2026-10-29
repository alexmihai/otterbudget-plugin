---
name: log-trade
description: Records, corrects or deletes a trade on an asset already in the member's OtterBudget portfolio, through the tool's preview and a yes in the chat. Use when the member tells you a buy or a sell they made, or asks to fix or delete a recorded one. Not for a sell's tax before selling (sell-tax-preview) or a plan (rebalance-plan).
---

# Log a trade

Record, correct or delete a trade on an asset the connected OtterBudget portfolio already holds or has held. Every write goes through its preview and digest. Save nothing before the member says yes.

## When it applies

- The member tells you a buy or a sell they made, or asks to fix or delete a recorded one. Answer it.
- The ask is a sell's tax before selling, or a plan. Use the Hand-offs below.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and save nothing.
- A tool says the connection lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and save nothing.
- A tool says the connection lacks portfolio:write, or the member's role cannot write. Answer "This connection cannot save trades. Reconnect and approve portfolio:write, or ask a household admin for Investments write access." and save nothing.
- The ticker is not in the portfolio. Answer "<ticker> is not in this portfolio yet. Add it in the app with Add position, then tell me the trade again." with the member's ticker, and save nothing.

## Tools

- Reads: `list_holdings` (open lots with their ids, dates and prices), `get_holding` (an asset's lots and sells), `list_transactions` (recorded lots, by ticker, broker, side and date), `list_cash_balances` (each broker's cash per currency).
- Writes: `add_position`, `update_position` and `delete_position`. Call each first without `reviewed`: that call saves nothing and returns the preview and its digest.

## A new trade

Resolve every field from the message and the reads before you ask anything. Guess what the message leaves out. Ask once.

1. Resolve these from the message:
   - The asset: its ticker, its ISIN or a broker's symbol form, a market code joined to the ticker by a dot ("US.SIMO" is SIMO).
   - The broker and the settlement currency ("tradeville usd" is Tradeville, settled in USD). When the message names no broker and the portfolio holds the asset at one broker only, take that broker. The settlement currency defaults to the asset's trade currency.
   - The side, the quantity and the price ("cumparat 5 US.SIMO la 215.73" is a buy of 5 at 215.73).
   - The date: today unless the message gives one. Leave `fees` out, so the broker's fee rule fills them.
   - A part no field takes, such as the time or an order number, is not saved.
2. Guess these when the message does not state them, and do not ask first:
   - The instrument: the one the portfolio holds of that asset at that broker, else shares. Held there both ways, shares. Show the other in brackets: "Shares (or CFD)".
   - A sell's lots at a lot-selection broker: the oldest set of whole open lots of that instrument that totals the quantity. It takes the oldest lot that can be part of such a set, then the oldest next one, and so on. Name each lot by its date, units and price, and list the other open lots in brackets. When no set of whole open lots totals the quantity, list the open lots with their dates and prices, ask which ones the sell closed, and call no write until the member answers. Send the lots as `closedLotIds`.
   - The FX rate, when the settlement currency differs from the trade currency: leave `fxRateAtTrade` out. `add_position` fills it from the stored close of the trade date, or the closest earlier one. Show the preview's rate and the date of its close, with the broker's own rate as the alternative: "0.8650 (stored rate for 28 Oct 2026, or your broker's rate)". When the tool refuses with "Enter an FX rate at trade.", ask for that rate and for nothing else.
3. When a field has no guess and the message and the reads leave it open (the side, the quantity, the price, or the broker of an asset held at more than one), ask for those fields only, in one message. Call no write until the member answers.
4. Look for the same trade. Call `list_transactions` narrowed by the ticker, the broker, the side and the trade date, and compare the quantity and the price on its rows. On a match, the summary says "Looks already recorded" above "Save or change?". A plain yes still saves.
5. Call `add_position` without `reviewed`, then show the summary as set out in "The summary".
6. Several trades in one ask: preview them one at a time in date order. Take one yes for each trade and save it before you preview the next.

## A correction or a delete

1. Find the lot from `get_holding` and `list_transactions`.
2. When more than one lot matches, list them with their dates and prices, ask which one, and call no write.
3. Call `update_position` with the lot's `id` and only the fields to change, or `delete_position` with its `id`, each without `reviewed`. Show the summary as set out in "The summary". For a delete, show the lot and the cash it returns.

## The summary

Show one summary and end it with "Save or change?". It holds, in this order:

1. For a new trade: the asset, the broker, the side, the instrument, the quantity, the price, the date, the settlement currency, and the FX rate and the lots where they apply. Each guess shows its alternatives in brackets.
2. The preview's figures: gross, fees, swap, tax, net and the cash after. For a new trade, name the fee as the broker's rule gives it. Then list each recorded sell the write recomputes, by its date, with its new figures.
3. "Looks already recorded", where it applies.

## The yes

- Send the digest only when the member's next message is a plain yes, such as "yes", "save" or "ok". Send the same fields with `reviewed` set to the preview's digest. When the preview named recomputed sells, also send `confirmRecompute` true.
- When the next message changes a field or picks an alternative ("CFD", "close the June lot", "the rate was 0.8700", "make it 6"), call the write again without `reviewed` with that field changed, show the new summary and ask "Save or change?" again. Only a plain yes to that new summary sends its digest.
- When the member answers "change" and names no field, ask what to change and save nothing.
- Any other reply, including an unrelated one, saves nothing and sends no digest.
- When the tool refuses because the figures changed, ask for a new preview and show it.

## The answer

1. After a save, say what was saved. When the cash ends below 0, give the tool's line on the cash.
2. Every amount carries its currency code and every change its sign.
3. When the answer gives a current price, mark it as `ytd-gains` does: Stale with the price's time, Estimate, No price or Incomplete, each reason once under the answer, with one "Prices as of" line.

## Hand-offs

- A plan or a rebalance: `rebalance-plan`.
- A sell's tax before selling: `sell-tax-preview`.
