---
name: undo-a-change
description: "Find a change in the history, see exactly what undoing it restores, and undo it."
---

# Undo a change

Any change can be undone: an entry, a categorization, a rule and everything it changed, a batch, a close. Undo never deletes history. It records the reversing change, links the two, and names you as its actor.

## Steps
1. Find the `change_id`: from the `run` result, or `search {kind: "changes", filters: {actor: "this_connection"}}` for your own changes.
2. `undo {change_id, preview_only: true}` lists every restoration: ledger lines, balances, rows affected, and whether it applies or waits.
3. `undo {change_id, idempotency_key}` applies it. Undo follows the company's approval rules like any change: undoing an entry posts its reversal, which applies at once under the default rules; undoing a close reopens the month, which waits for an owner.
4. `get {kind: "change", id}` on the original shows `status: "undone"` and `undone_by`. When the undo is `pending_approval`, tell the person and check `get {kind: "proposal", id}`.

## Common errors
- `ALREADY_UNDONE`: someone undid it already; `get {kind: "change"}` names the undo.
- `PERIOD_CLOSED`: the undo would post into a closed month. Load `change-a-closed-month`.

## Worked example
```json
[
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: to undo", "lines": [ { "account": "6600", "debit_cents": 2500 }, { "account": "1000", "credit_cents": 2500 } ] }, "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "changes", "filters": { "actor": "this_connection", "limit": 1 } }, "expect": { "path": "changes.0.change_id", "equals": "{{step1.change_id}}" } },
  { "tool": "undo", "args": { "company": "Northwind Labs", "change_id": "{{step1.change_id}}", "preview_only": true }, "expect": { "path": "diff.rows_affected", "equals": 2 } },
  { "tool": "undo", "args": { "company": "Northwind Labs", "change_id": "{{step1.change_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "change", "id": "{{step1.change_id}}" }, "expect": { "path": "change.status", "equals": "undone" } }
]
```
