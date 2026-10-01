---
name: undo-a-change
description: "Find a change in the history, see exactly what undoing it restores, and undo it."
---

# Undo a change

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Any change can be undone: an entry, a categorization, a rule and everything it changed, a batch, a close. Undo never deletes history. It records the reversing change, links the two, and names you as its actor.

## Steps
1. Find the `change_id`: from a change's result, or `get_audit_log {actor: "this_connection"}` for your own changes.
2. `undo_change {change_id}` lists every restoration: ledger lines, balances, rows affected, and whether it applies or waits.
3. `undo_change {change_id, mode: "apply", preview_id, idempotency_key}` applies it. Undo follows the company's approval rules like any change: undoing an entry posts its reversal, which applies at once under the default rules; undoing a close reopens the month, which waits for an owner.
4. `get_change {change_id}` on the original shows `status: "undone"` and `undone_by`. When the undo is `pending_approval`, tell the person and check `get_proposal {proposal_id}`.

## Common errors
- `ALREADY_UNDONE`: someone undid it already; `get_change {change_id}` names the undo.
- `PERIOD_CLOSED`: the undo would post into a closed month. Load `change-a-closed-month`.
