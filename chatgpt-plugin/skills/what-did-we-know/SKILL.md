---
name: what-did-we-know
description: "Show a report as the books were recorded at a moment (or as a closed month was frozen), compare it with now, and name the changes in between."
---

# Answer "what did we know on date X"

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Investors, auditors and boards ask what the numbers said when a decision was made: "what did the September P&L show when we sent the board deck on October 12?" The books keep every change with its time, so any report can be read as it was recorded at a moment.

## Steps
1. `get_report {report: "pl", to, known_at}` (any statement). `known_at` is a date, meaning the end of that day in UTC, or a date and time with its offset such as `2026-10-12T17:00-07:00`. The response says how many changes were recorded since, and how many of those posted into this period.
2. For a closed month, pass `version` (1 is the first close) instead: the statement exactly as frozen at that close; `from` and `to` are the same month.
3. `compare_reports {report, to, a_known_at}` (or `a_version`), with `b` defaulting to now: only the lines that differ, each with its change ids.
4. `get_change {change_id}` on those ids says who changed what, when and why (a late invoice, a recategorization, a reopened month).
5. `get_lineage {accounts, from, to}` traces any number today to its entries, transactions, documents and the changes that posted them.

Say the answer as: "On Oct 12 the September P&L showed a net loss of $X. Since then N changes moved it to $Y: …".
