---
name: whats-new
description: "Tells the member what changed in the OtterBudget app, version by version, newest first, with each version's day and the link to its commit on GitHub. Use when the member asks what is new or what changed in OtterBudget, or what a version brought. Not for changes in their own portfolio, watchlist or spending (portfolio-briefing, watchlist-review, spending-check)."
---

# What's new

Tell the member what changed in the OtterBudget app, from one read. Read only: this skill saves nothing. Give no change, detail or link that `list_whats_new` did not return.

## When it applies

- The ask is for what is new or what changed in OtterBudget or the app, or what a version brought, such as "What's new in OtterBudget?", "What changed in the app this week?" or "What did version 2.60.0 change?". Answer it.
- The OtterBudget tools are not available to you, or the connection is not valid. Answer "OtterBudget is not connected. In claude.ai, Desktop or Cowork, open the plugin's Connectors tab and choose Connect. In Claude Code, run /mcp and choose Authenticate." and give no entry.
- This skill has no scope refusal: `list_whats_new` answers every valid connection, whatever its scopes.

## Inputs

One call to `list_whats_new`. It returns `appVersion`, `entries` (newest first, each with `version`, `date` as YYYY-MM-DD, `lines` and `commitUrl`) and `more`.

- The ask names no version and no period: call it with no argument. It returns the 10 newest entries.
- The ask names a period ("this week", "since Monday", "in September"): pass the period's first day as `since`, its last day as `until` (today for a period that has not ended), both as YYYY-MM-DD in Bucharest time, and `limit` 100. A week starts on Monday.
- The ask names a version: pass it as `version`.

## The answer

Give the entries in the order returned. Each entry gives:

1. Its version.
2. Its day, written as "28 Oct 2026".
3. Its lines, as the tool returned them.
4. Its `commitUrl` as the commit link. When `commitUrl` is null, give no link.

When `more` is true, end the answer with "Older versions are on the What's new page." and the page's address, https://otterbudget.vercel.app/whats-new.

When no entry matches, answer in one sentence and nothing else: "No version 9.9.9 is listed." for a version, with the asked version, or "No version between 26 Oct 2026 and 28 Oct 2026." for a period, with the period's first and last day.
