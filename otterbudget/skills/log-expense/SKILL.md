---
name: log-expense
description: Adds, schedules, corrects, deletes or stops expenses in the member's OtterBudget spending profile, in RON or EUR, from one of their accounts, after a yes in the chat. Use when the member tells you an expense or a repeating one, or asks to change one this client added. Not for budget questions (spending-check), an account's balance (cash-check) or a bank payment waiting to be confirmed (confirm-payments).
---

# Log an expense

Add or schedule an expense in the connected OtterBudget spending profile, or correct, delete or stop one that this client added. Save nothing until the member says yes to a summary of that write.

## When it applies

- The member tells you an expense to add, a repeating one to schedule, or asks to change, delete or stop one. Do it.
- The ask is about a budget or spending totals, or a bank payment waiting to be confirmed. Use the hand-offs below.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and do nothing else.
- A tool says the connection cannot read a spending profile. Answer "This connection cannot read a spending profile. Reconnect and approve spending:read, or ask a household admin for Spending read access." and call no tool.
- You know the connection lacks `spending:write`, or a write tool says so. Answer "This connection cannot save expenses. Reconnect and approve spending:write, or ask a household admin for Spending write access." and call no write.

## Tools

- Reads: `list_spending_categories`, `list_spending_accounts`, `list_scheduled_expenses`, `list_spending_transactions`.
- Writes: `add_spending_transaction`, `add_scheduled_expense`, `update_spending_transaction`, `delete_spending_transaction`, `stop_scheduled_expense`.

## The fields

- Amount: above zero, with at most two decimals. The currency is RON or EUR; "lei" is RON. For any other currency answer "Expenses are in RON or EUR. Give the amount in one of them." and go no further.
- Category: read the household's categories with `list_spending_categories`. Match the member's word to one by name, ignoring case, or by clear meaning ("groceries" is Food). Show the matched name. When none fits, list the categories and ask which one. Create no category.
- Account: read the household's open accounts with `list_spending_accounts`. Match the member's word to one by name, ignoring case, or by clear meaning ("cash" is Physical cash, "my ING card" is ING RON). Show the matched name. When none fits, list the open accounts and ask which one. Create no account. When the member names none, leave the account out of the write and show the default account in the summary, so they can change it before the yes.
- Date: today in Europe/Bucharest unless the member gives one. Resolve "yesterday" and similar words to a calendar date, and show it as "22 Oct 2026". Send it as YYYY-MM-DD.
- Note: the shop or the reason, as the member says it. Leave it empty when they give none.
- Repeat: "every week", "every month" or "every year" means a schedule. Its first date is today unless the member gives one. When the first date is before today, answer "The first date is before today. Pick today or a later date." and offer the next date that falls on or after today in the same cycle.

## Add or schedule

1. Resolve the amount, currency, category, account, date and note, with the repeat and first date for a schedule.
2. Show them as a summary, for example "RON 230.00, Food, Physical cash, 22 Oct 2026, note Kaufland", and end with "Save it?". Call no write.
3. When the member's next message is a plain yes, such as "yes", "save" or "ok", call `add_spending_transaction`, or `add_scheduled_expense` for a schedule, once.
4. When the next message changes a field ("make it 250", "make it EUR 12.50"), show a new summary and end with "Save it?" again. Only a plain yes to that summary saves.
5. Only the member's next message answers a summary. After any other reply, including an unrelated question, save nothing. Answer it, then show the summary and "Save it?" again before a yes counts.

## Change, delete or stop

Only an expense or a schedule that this client added can be changed. The server decides which.

1. Find it. For an expense, use `list_spending_transactions` with its month, and with its category when the ask names one. For a schedule, use `list_scheduled_expenses`. Keep its `id`. When more than one row fits, list them and ask which one.
2. Show the summary and end with "Save it?". Call no write.
   - Change: every field the write will save, the changed ones and the ones that stay, such as "RON 250.00, Food, Physical cash, 22 Oct 2026, note Kaufland". An account the member did not change stays out of the write.
   - Account: when the member says a recorded expense was paid from another account ("that was from ING"), match it as in the fields, show the expense with the new account, and on a yes call `update_spending_transaction` with that account.
   - Delete: the expense's amount, currency, category, account, date and note.
   - Stop: the schedule's amount, currency, category, account, repeat and next date. Say that the expenses it already added stay.
3. On a plain yes, call `update_spending_transaction`, `delete_spending_transaction` or `stop_scheduled_expense` once, with that `id`. Only the member's next message answers the summary. When it changes a field, show a new summary and "Save it?". After any other reply, save nothing, answer it, then show the summary and "Save it?" again before a yes counts.
4. When the server refuses because another client added it, or because the bank added it, give the server's refusal sentence as it is and say that the app edits it. Change nothing else.

## Several expenses

A receipt photo or a pasted list, where you can read it: list the expenses you found. Give each one its own summary, its own "Save it?" and its own `idempotencyKey`. A yes saves only the expense it answers.

## Keys

Make a new `idempotencyKey` for each expense or schedule when you first call its write. Send the same key again only to retry that same call after a failure or no answer. A different expense never reuses a key.

## The answer

After a save, give the saved fields in the same words as the summary, and say it is saved. When a write fails, give the server's message and say nothing was saved.

## Hand-offs

- Budgets or spending totals: `spending-check`.
- An account's balance: `cash-check`.
- Setting an account's or a broker's balance: no skill; the tool's own description says how.
- A bank payment waiting to be confirmed: `confirm-payments`.
- Moving money between accounts: the app's Move money on Cash; no skill.
