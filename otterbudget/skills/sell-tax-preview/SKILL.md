---
name: sell-tax-preview
description: Previews the tax and net of a sell the member is considering in OtterBudget, from the app's own tax model, and compares lot choices; it saves nothing. Use for a sell's tax before selling or which lots to sell. Not for a sell already made (log-trade).
---

# Sell tax preview

Preview the tax and net of a sell the member is considering, using the app's own tax model. Compare up to three lot choices. Save nothing and give no tax rate of your own.

## When it applies

- The member asks for a sell's tax before selling, or which lots to sell. Follow the steps below.
- The ask belongs to a topic under Hand-offs. Hand it over instead of running this preview.
- The OtterBudget tools are not available. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- The connection has no portfolio or lacks `portfolio:read`. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure.
- The connection lacks `portfolio:write` or the Investments write role, even for a preview. Answer "This connection cannot preview a sell's tax. Reconnect and approve portfolio:write, or ask a household admin for Investments write access." and give no figure.

## Inputs

1. Read `list_holdings`, with its broker filter when the member names a broker. Find the asset, its listing, trade currency, latest quote and open lots with their ids, dates, instruments and remaining units.
2. Read `get_holding` for that ticker and exchange in full. Use the chosen broker's `costMethod` and lots. Keep share and CFD lots separate. Ask for a missing or ambiguous asset, broker, instrument or quantity before previewing; do not guess which lots the member means.
3. Use the member's price when named. Otherwise use the latest quote's price and time. When there is no quote, mark No price and ask for a price; do not use the last trade as a current quote.
4. Use the sell date the member names, or today. Only lots opened on or before that date can close. Keep the date, quantity, price and other trade fields the same across choices.

## Lot choices

At a broker whose `costMethod` is `lot_selection`, form these choices in order:

1. The lots the member names, if any. Resolve a named purchase date to its actual open lot ids.
2. The oldest open lots first, in the trade order the tools give.
3. The lots held longest past 12 months on the sell date, oldest first.

Each choice must use whole open lots whose remaining units total exactly the sell quantity. Send their ids as `closedLotIds`; do not split a lot or mix instruments. If the quantity cannot be formed from whole lots, explain why and ask for a quantity or lot choice that fits. If the sell exceeds the units held past 12 months, omit that choice and say why.

Drop a choice when its set of lot ids repeats an earlier choice. Preview each remaining choice once, at most three. Thus, a named oldest lot can leave just one preview; a named newer lot and the oldest lot leave two when the third choice repeats the oldest.

At a broker whose `costMethod` is `weighted_average`, preview once without selecting lot ids. Say that the oldest units go first. Do not compare selected lots there.

## Preview only

For each distinct choice, call `add_position` with `ticker`, `exchange` when needed, `broker`, `side: "sell"`, the matching `instrument` (`share` or `cfd`), `date` in YYYY-MM-DD, `quantity` and `price`. Include `closedLotIds` only for lot selection.

- Omit `reviewed` on every call. The response has `ready: false`, a `reviewed` digest and `preview`; this is not a saved trade. Never send that digest, including after "yes", "save", "ok" or "sell B".
- Omit `fees` unless the member gives them, so the broker's stored fee rule supplies them. Send a named `swap` only for a CFD sell, signed in settlement currency; omitted swap is zero.
- `settlementCurrency` defaults to the asset's trade currency. If the member chooses a different settlement currency, ask for `fxRateAtTrade` from trade currency to settlement currency; do not invent a rate. Fees and swap are in settlement currency.
- Leave `costPriceOverride` out so each choice uses the broker's cost method. Do not add a tax rate or compute tax yourself.
- A changed input needs new previews without `reviewed`. A refused or incomplete preview supplies no tax or net figure; explain its tool error instead of estimating them.

When the member says to sell, hand over to `log-trade`, which previews again and asks "Save it?". This skill never confirms or saves a trade.

## The answer

1. Name the asset, broker, instrument, sell date, quantity and price. Give the latest quote's time when that price is used.
2. Give one row per preview, naming its lots by purchase date and units. Show gross, swap, fees, tax and net from `preview.sell`, in the asset's trade currency. Gross and net are profit figures, not sale proceeds. Compare the returned tax and net across the choices; give no rate of your own.
3. Show the returned `preview.cashChange` in settlement currency. If `preview.affected` names recorded sells, show their dates, old and new cost, tax and net, cash changes and cash after where returned. Do not invent a cash balance absent from the response.
4. Say which choices were duplicates or could not cover the quantity, and why. State that nothing was saved.
5. Every amount carries its currency code; every change carries its sign. Do not add amounts across currencies.
6. Mark Stale with the price's time, Estimate for an estimated price or figure, No price for a missing current quote, and Incomplete for unavailable lots or preview figures. Give each reason once under the answer. Never replace a missing figure with zero.
7. Give one "Prices as of" line with the time of the oldest current price used. If none was used, say so and identify the member's supplied price.

## Hand-offs

- Gains on a holding this year: `ytd-gains`.
- A sell already made, or a fix to a recorded trade: `log-trade`.
- Tax already paid this year: the server's guidance.
