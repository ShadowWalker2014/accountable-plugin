---
name: read-reports
description: "Pull P&L, balance sheet and trial balance for whole months or exact days, respect the readiness warnings, and drill into a line."
---

# Read reports

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

`get_report {report, from, to, basis}` runs any statement: `pl`, `bs`, `tb`, `cf` or `gl`, with `to` and for ranges `from`, and `basis` (cash or accrual). Each bound is a month (YYYY-MM, the whole month) or a day (YYYY-MM-DD, that exact day; both days are included, and a range past today stops at today). `period` takes a token instead: 2026-09, 2026-Q3, 2026, 2026-07-08..2026-08-20, 2026-07-08 or all. Balance sheet and trial balance are as of the last day of `to`. `get_close_status` says which months are closed.

## Read the warnings first
`warnings[]` lists reasons a number may be wrong or change: a stale feed (named, with its last sync date), months after the closed-through month, a cash account below zero (missing opening balances), and the share of revenue and expenses still uncategorized. Say them to the person before quoting figures.

## Drill down
Every amount is `{amount_cents, amount, currency}`. To see what makes up a line: `query_ledger {account, from, to}`, or `list_transactions {category}`. To say why it moved: `explain_change`.
