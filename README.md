# OtterBudget plugin

The OtterBudget plugin connects Claude to your OtterBudget portfolio and spending. It adds the connection to the OtterBudget MCP server and fourteen skills: `ytd-gains`, `log-trade`, `allocation-checkup`, `rebalance-plan`, `watchlist-review`, `spending-check`, `upcoming-payouts`, `portfolio-briefing`, `log-expense`, `sell-tax-preview`, `cash-check`, `confirm-payments`, `whats-new` and `earnings-preview`. You need an OtterBudget account. The owner must invite you first.

## Install

### claude.ai, Claude Desktop and Cowork

1. Open Customize, then Plugins. Choose Add, then Add marketplace.
2. Enter `alexmihai/otterbudget-plugin`.
3. Add the OtterBudget plugin.
4. On the plugin's Connectors tab, choose Connect. Sign in and approve the access you want to grant.

If Add marketplace does not work, see [If Add marketplace fails](#if-add-marketplace-fails).

### Claude Code

If the plugin is already on your claude.ai account, there is nothing to do. Claude Code gets the plugin when you sign in to that account.

Only if you do not have the plugin on your claude.ai account:

1. Run `claude plugin install otterbudget --marketplace alexmihai/otterbudget-plugin`.
2. Run `/mcp` in Claude Code and choose Authenticate.

### ChatGPT and Codex

These clients get the connection only. The skills do not install there.

1. Add a custom MCP server at `https://otterbudget.vercel.app/api/mcp`.
2. Sign in and approve the access you want to grant.

### If Add marketplace fails

1. Download `https://github.com/alexmihai/otterbudget-plugin/releases/latest/download/otterbudget.zip`.
2. In claude.ai, Claude Desktop or Cowork, go to Customize, then Plugins. Choose Add, then Upload plugin.
3. Upload the zip file. Then connect the plugin on its Connectors tab, as in the steps above.

An uploaded plugin does not update itself. See [Updates](#updates).

## Pre-earnings notes

The `earnings-preview` skill writes a note for each coming report of a stock you hold. To get the notes without asking, create a scheduled task in your own Claude client:

1. Use a client that has this plugin and the OtterBudget connection. Approve the connection with `portfolio:read` and `research:write`.
2. Create a Cowork scheduled task, or a Claude Code routine.
3. Set it to run daily at 08:30 Bucharest time.
4. Set its prompt to `Write the pre-earnings notes.`

A connection works in one portfolio, so create one task for each portfolio. The app does not schedule the task. The task saves each note without asking for a yes. You can hide a note on the research page.

## Requirements

- For Claude, a paid Claude plan. Plugins need one.
- In claude.ai, the skills need Code execution and file creation. Turn both on under Settings, then Capabilities.
- An invited OtterBudget account.

## Updates

- **claude.ai, Claude Desktop and Cowork:** turn on Sync automatically for the marketplace. Each update then arrives on its own.
- **Claude Code:** to get each update, run `/plugin`, open Marketplaces, choose otterbudget-plugin and turn on auto-update.
- **Zip install:** download the latest zip and upload it again. An uploaded plugin does not update itself.
- **ChatGPT:** refresh the connection.
- **Codex:** none needed.

## Source

The source of this plugin lives in the OtterBudget app's repository. This repository is published from it. Do not make changes here.
