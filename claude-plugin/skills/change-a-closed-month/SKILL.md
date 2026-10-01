---
name: change-a-closed-month
description: "Fix something in a month that is already closed, as one logged reopen, change and relock with a reason, and undo it if needed."
---

# Change a closed month

A closed month is locked: its statements were reported. It can still change, but only inside one logged change that reopens it with a reason, applies the fix and locks it again. The month is never left open.

## Steps
1. `report {name: "close_status"}`: `closed_through` is the last closed month.
2. Find the rows: `search {kind: "transactions", filters: {from, to, search}}` with that month's dates.
3. `preview {closed_month: {month, reason}, batch: [...]}` with every fix as a batch item, for example `{action: "categorize_transactions", input: {transaction_ids, account}}`. The preview shows each row before and after and the month reopening and relocking.
4. `run` with the same `closed_month`, `batch` and a new `idempotency_key`.
   - Under the default guardrails a closed-month change waits for an owner: `status: "pending_approval"` with an `approval_url`. Tell the person why it matters.
   - Under Full control it applies at once as one change with one `change_id`.
5. `undo {change_id}` reverses it inside the same reopen and relock.

For a single recategorization, `run {action: "propose_closed_month_recategorization", input: {transaction_ids, account}, reason}` sends the owner one change to approve.

## Rules
- Always give the real reason ("Bank fee booked as software in August"); it is logged on the month and shown to the approver.
- Say which reported figures move: a closed month's P&L and balance sheet change.
- Reopening a month on its own (`reopen_period`) always needs an owner. Prefer a closed-month batch: the month is never left open.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "close_status" }, "expect": { "path": "closed_through" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "transactions", "filters": { "from": "{{step1.closed_through}}-01", "to": "{{step1.closed_through}}-28", "limit": 1 } }, "expect": { "path": "transactions.0.id" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "propose_closed_month_recategorization", "input": { "transaction_ids": ["{{step2.transactions.0.id}}"], "account": "6900" }, "reason": "Skill test: bank fee booked in the wrong category", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "pending_approval" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "proposal", "id": "{{step3.proposal_id}}" }, "expect": { "path": "proposal.status", "equals": "waiting" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "closed_month": { "month": "{{step1.closed_through}}", "reason": "Skill test: bank fee booked in the wrong category" }, "batch": [ { "action": "categorize_transactions", "input": { "transaction_ids": ["{{step2.transactions.0.id}}"], "account": "6900" } } ] }, "requires": "apply_batch", "expect": { "path": "preview_id" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "closed_month": { "month": "{{step1.closed_through}}", "reason": "Skill test: bank fee booked in the wrong category" }, "batch": [ { "action": "categorize_transactions", "input": { "transaction_ids": ["{{step2.transactions.0.id}}"], "account": "6900" } } ], "idempotency_key": "{{uuid}}" }, "requires": "apply_batch", "expect": { "path": "status", "equals": "pending_approval" } }
]
```
