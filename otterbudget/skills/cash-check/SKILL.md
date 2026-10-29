---
name: cash-check
description: "Tells the member how much cash they hold in OtterBudget and where: each current account and each broker's cash, per currency, with a total per currency. Use for cash, an account's balance or the cash at a broker. Not for months of spending covered (spending-check) or the portfolio's value (portfolio-briefing)."
---

# Cash check

Tell the member how much cash they hold in the connected OtterBudget portfolio and where, from one read. Read only: this skill saves nothing. Give no figure that these steps do not produce.

## When it applies

- The ask is for cash, an account's balance or the cash at a broker. Answer it.
- The ask is to set a balance. Call no write; hand off with the last line below.
- The ask is for months of spending covered, the portfolio's value or today's change. Hand over as the hand-offs say and give no figure.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no figure.
- A tool says the connection has no portfolio or lacks portfolio:read. Answer "This connection cannot read a portfolio. Reconnect and approve portfolio:read, or ask a household admin for Investments read access." and give no figure. Account balances need the same access.

## Inputs

1. `list_cash_balances`, once. It gives two lists: `accounts`, the household's current accounts, and the broker rows, one per broker and currency. Each carries its amount, its last-updated stamp and whether it is archived. Each account also carries its name, its currency, whether it is the default, its bank, its owner (a member's name, or null for Shared) and whether it is linked; each broker row its broker's name.

## Figures

- Every typed balance shows the date of its last update, as in "updated 20 Oct 2026", because a typed balance can be old.
- A linked account's balance comes from its bank, not from a member's typing. Show it as "from `<bank>`" with the date and time of its last read (its `updatedAt`), as in "from ING, read 28 Oct 2026, 14:05", in place of the updated date.
- A balance below 0 keeps its sign.
- Never add amounts of different currencies and never convert one. Read no rate.
- A total is the sum of the balances in one currency: every account and every broker row listed, and every archived one with a balance other than 0.
- An archived account or broker with a balance of 0 is left out. One with another balance is listed and marked as archived, and the totals count it.

## The answer

The ask names no account and no broker. Give, in this order:

1. Each open account with its balance and date, and each account's bank and owner when `list_cash_balances` gives them (Shared for a null owner; leave out what it does not give), the default account first, then any archived account with a balance other than 0, marked as archived.
2. Each broker's cash, per currency, with its date, archived brokers with a balance other than 0 marked as archived.
3. One total per currency.
4. Under the totals: "For one total in your reporting currency, see the Cash page." and a note that deposits are not cash and are left out.

The ask names one account or one broker. Match the name to an account or a broker, ignoring case or by clear meaning ("my ING card" to ING RON). Give that one's balances with their dates, and an account's bank and owner, and nothing else: no totals, no Cash page sentence and no note on deposits. When the name fits none, list the accounts and the brokers and ask which one.

Do not add an expense, give a coverage figure or give a portfolio value.

## Hand-offs

- Months of spending that savings would cover: `spending-check`.
- The portfolio's value or today's change: `portfolio-briefing`.
- Adding, scheduling, changing, deleting or stopping an expense: `log-expense`.
- Setting an account's or a broker's balance: no skill; the tool's own description says how.
