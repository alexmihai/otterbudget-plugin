---
name: confirm-payments
description: "Walks the member through the bank payments waiting in their OtterBudget spending profile: shows each one's amount, date, category and account, then confirms, matches, moves or deletes it after a yes in the chat. Use when the member asks what is waiting to confirm, or to confirm, file, move or delete bank payments. Not for an expense the bank did not send (log-expense) or budgets (spending-check)."
---

# Confirm bank payments

Show the bank payments that wait in the connected OtterBudget spending profile, and confirm, match, move or delete them as the member says, as To confirm does in the app. Save nothing until the member says yes to a summary of that write.

## When it applies

- The member asks what is waiting, or asks to confirm, file, match, move or delete a bank payment. Do it.
- The ask is for an expense the bank did not send, a budget or an account's balance. Use the hand-offs below.
- The OtterBudget tools are not available to you. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and do nothing else.
- You know the connection lacks write in Spending or `spending:write`, or a tool refuses the connection as a reader. Answer "This connection cannot confirm payments. Reconnect and approve spending:write, or ask a household admin for Spending write access." and call no tool.

## Tools

- Reads: `list_payments_to_confirm`, `list_spending_categories`.
- Writes: `confirm_payment`, `delete_payment`.

## List the payments

1. Call `list_payments_to_confirm` once. When `truncated` is true, say that only the newest are listed and give `total`.
2. For each payment give the amount with its currency, the date, the note, the account, and why it waits (its `reason`):
   - `preselected`: the `suggestion` category, marked "suggested".
   - `pick`: the `choices`, as three category names.
   - `may_not_be_expense`: "May not be an expense", with the `choices`.
   - `deleted_last_time`: "Deleted last time", with the `choices`.
   - `possible_duplicate`: the expense in `duplicateOf` with its date, amount, category and account.
   - `move` and `same_move`: the account in `moveTo`, or the `moveChoices` when `moveTo` is null, and for `same_move` the move in `duplicateMove`.
   - `other_currency`: the amount in the bank's currency and, in `originalAmount`, what the card paid.
   - `category_not_suggested` and `no_categories`: no category is offered. Read the household's categories with `list_spending_categories` when the member names one.
3. Call no write while you list.

## Confirm one payment

1. Match the member's word to a payment by its merchant, amount or date. When more than one fits, list them and ask which one.
2. Match the member's category to one of the household's open categories with `list_spending_categories`, by name ignoring case or by clear meaning. Show the matched name. When none fits, list the categories and ask which one. Create no category.
3. Show the amount with its currency, date, category and account, for example "RON 86.40, 17 Oct 2026, Groceries, ING RON", and end with "Save it?". Call no write.
4. Call `confirm_payment` once, with `paymentId` and `category`, only when the member's next message is a plain yes, such as "yes", "save" or "ok". When the next message changes a field, show a new summary and end with "Save it?" again. After any other reply, save nothing, answer it, then show the summary and "Save it?" again before a yes counts.
5. After the save, name the expense the answer gives. When the answer says the payment was already confirmed, say so and change nothing else.

## Confirm all the suggested ones

1. List each payment whose `reason` is `preselected`, one line each with its amount and currency, date, category and account, and end with "Confirm these `<n>`?". Call no write.
2. On a plain yes, make one `confirm_payment` call per listed payment, in its suggested category. Confirm no payment that you did not list. A payment of any other reason gets its own question.
3. When a call fails, stop, say which payments were saved and which were not, and ask again before you call for the rest. Give the server's message.
4. After any reply but a plain yes, save nothing.

## A possible duplicate

Show the payment and the expense in `duplicateOf`, and ask whether it is the same expense or another one.

- Same: show "Same as the expense of `<date>`, `<amount>`, `<category>`. No expense is added." and end with "Save it?". After a plain yes, call `confirm_payment` with `sameExpense` true.
- Another one: it needs a category. Confirm it as above.

## Another currency

A payment with the reason `other_currency` needs its amount in RON or EUR before you show its summary. Ask "Give the amount in RON or EUR.", show the summary with that amount and currency, and after a plain yes call `confirm_payment` with the category, `amount` and `currency`. Calling it without them is refused.

## A move

A payment with the reason `move` is money moved between the household's accounts, not an expense.

1. Show the two accounts and the amount, such as "RON 500.00 from ING RON to Physical cash", and ask "Move it?". Offer only the accounts in `moveChoices` as the target; a `moveTo` is the suggested one. Call no write.
2. After a plain yes, call `confirm_payment` with `moveTo`.
3. For `same_move`, show the move in `duplicateMove` and ask whether it is the same move. After a plain yes call `confirm_payment` with `sameMove` true. For another one, treat it as a `move`.

## Delete a payment

When the member says a payment is not an expense, or asks to delete it, show it, add "Later payments to `<merchant>` will show 'Deleted last time'.", and end with "Delete it?". Leave that line out for a payment with neither a merchant nor a creditor name. Call `delete_payment` once only after a plain yes. No expense is added.

## Limits

- Change no payment date or account. The bank gives both.
- Add, edit or delete no expense. Use the hand-offs below.
- A server refusal is given as it is, and nothing is saved.

## The answer

After a save, give the saved fields in the same words as the summary, and say it is saved. When a write fails, give the server's message and say nothing was saved.

## Hand-offs

- Adding, scheduling, changing, deleting or stopping an expense: `log-expense`.
- Budgets or spending totals: `spending-check`.
- An account's balance: `cash-check`.
