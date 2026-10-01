---
name: close-a-month
description: "Work the month's checklist (feeds, categories, reconciliations, report) and close it."
---

# Close a month

Closing locks a month: nothing posts to it again except through one logged reopen, change and relock. `propose_close` closes it; it fails at preview with `CLOSE_BLOCKED` until the checklist is done, and the error lists what is still open.

## Steps
1. `report {name: "close_status"}`: `closed_through` and `next_month_to_close`. Close months in order.
2. `report {name: "close_checklist", params: {month}}`: `blocking` names every open item: uncategorized transactions, transactions to review, accounts that do not tie to their statement, missing statements, and the month-end report.
3. `report {name: "feed_health"}`: a stale or broken feed means missing transactions. Stop and tell the person.
4. Work each blocking item:
   - Uncategorized or to review: `search {kind: "transactions", filters: {from, to, uncategorized_only: true}}`, then load `categorize-for-this-company` or `answer-review-questions`.
   - Reconciliation: `report {name: "reconciliation"}`, then load `reconcile-an-account`.
   - Statements and the month-end report need a person in Accountable: say which ones.
5. `report {name: "bs", params: {to: month}}`: `totals.difference` must be $0.00. Read `warnings[]`.
6. `preview {action: "propose_close", input: {month}}`. If it returns `CLOSE_BLOCKED`, go back to step 2. Otherwise `run` it: closing applies at once under the default rules (an owner can make it wait), and the result is `applied`, or `pending_approval` with an `approval_url`.
7. For a waiting close, `get {kind: "proposal", id}` later shows `applied` (closed) or `rejected` with the reason.

## Common errors
- `CLOSE_BLOCKED`: the message and `details.blocking` list what is still open.
- `PERIOD_ALREADY_CLOSED`: nothing to do.
- `FORBIDDEN`: members and viewers cannot close months; an owner, admin or accountant must connect.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "close_status" }, "expect": { "path": "current_month" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "close_checklist", "params": { "month": "{{this_month}}" } }, "expect": { "path": "blocking" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "reconciliation", "params": { "month": "{{this_month}}" } }, "expect": { "path": "accounts.0.account.name" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "bs", "params": { "to": "{{last_month}}" } }, "expect": { "path": "data.totals.difference.amount_cents", "equals": 0 } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "propose_close", "input": { "month": "{{this_month}}" } }, "expect": { "error": "CLOSE_BLOCKED" } }
]
```
