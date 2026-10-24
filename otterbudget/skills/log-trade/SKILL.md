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

1. Ask for each required field the member did not state: the broker, the instrument (shares or CFD), the quantity, the price, and the FX rate at trade when the settlement currency differs from the trade currency. Ask before any preview. Assume no instrument: an asset held as shares can get a CFD trade, and `add_position` has no default for it.
2. Default the date to today. Leave `fees` out, so the broker's fee rule fills them.
3. A sell at a lot-selection broker: list the open lots with their dates and prices and ask which ones the sell closed. Pick none yourself, and call no write until the member answers. Send the answer as `closedLotIds`.
4. Call `add_position` without `reviewed`, then show the preview as set out in "The preview".
5. Several trades in one ask: preview them one at a time in date order. Take one yes for each trade and save it before you preview the next.

## A correction or a delete

1. Find the lot from `get_holding` and `list_transactions`.
2. When more than one lot matches, list them with their dates and prices, ask which one, and call no write.
3. Call `update_position` with the lot's `id` and only the fields to change, or `delete_position` with its `id`, each without `reviewed`. Show the preview as set out in "The preview". For a delete, show the lot and the cash it returns.

## The preview

Show the preview's figures: gross, fees, swap, tax, net, the cash after, and the trade's date. For a new trade, name the fee as the broker's rule gives it. Then list each recorded sell the write recomputes, by its date, with its new figures. End with "Save it?".

## The yes

- Send the digest only when the member's next message is a plain yes, such as "yes", "save" or "ok". Send the same fields with `reviewed` set to the preview's digest. When the preview named recomputed sells, also send `confirmRecompute` true.
- When the next message changes a field ("make it 11", "the fee was 2.50"), call the write again without `reviewed`, show the new preview and ask "Save it?" again. Only a plain yes to that new preview sends its digest.
- Any other reply, including an unrelated one, saves nothing and sends no digest.
- When the tool refuses because the figures changed, ask for a new preview and show it.

## The answer

1. After a save, say what was saved. When the cash ends below 0, give the tool's line on the cash.
2. Every amount carries its currency code and every change its sign.
3. When the answer gives a current price, mark it as `ytd-gains` does: Stale with the price's time, Estimate, No price or Incomplete, each reason once under the answer, with one "Prices as of" line.

## Hand-offs

- A plan or a rebalance: `rebalance-plan`.
- A sell's tax before selling: `sell-tax-preview`.
