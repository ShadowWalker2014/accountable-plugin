---
name: post-a-journal-entry
description: "Accruals, reclassifications and corrections as balanced entries, and which ones wait for an owner."
---

# Post a journal entry

Use a journal entry for accruals, reclassifications and corrections that no bank transaction covers. For a miscategorized payment, recategorize the transaction instead (`categorize_transactions`).

## Steps
1. `search {kind: "accounts", filters: {search}}` finds each account. Pass an account's id, code ("6300") or exact name ("Legal Fees").
2. `run {action: "create_journal_entry", input: {date, memo, lines}}`: at least two lines, each with `debit_cents` or `credit_cents`. Debits must equal credits to the cent. For an accrual that should reverse, add `reverse_on`.
3. The result is `applied` with the `change_id` and the posted entry, or `pending_approval`.
4. `get {kind: "entry", id}` reads it back with its lines.

## What waits
- Entries at or over the owner's limit ($5,000 unless the owner changed it) wait for an owner: you get `pending_approval` with an `approval_url`. Tell the person and check `get {kind: "proposal", id}` later.
- Smaller entries apply at once under the default rules. The owner can require approval in Settings › Approval rules.
- A closed month refuses the entry. Date it in an open month, or load `change-a-closed-month`.
- Posted entries are never edited in place. To correct one, `run {action: "reverse_entry"}` and post the right entry.

## Common errors
- `UNBALANCED`: the message names the difference.
- `ACCOUNT_INACTIVE`: pick another account or ask a person to reactivate it.
- `PERIOD_CLOSED` or `POLICY_FORBIDS`: the month is closed; see above.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "accounts", "filters": { "search": "Accrued" } }, "expect": { "path": "accounts.0.code", "equals": "2300" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: small accrual", "lines": [ { "account": "6310", "debit_cents": 12000 }, { "account": "Accrued Expenses", "credit_cents": 12000 } ] }, "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "entry", "id": "{{step2.record.entry_id}}" }, "expect": { "path": "entry.lines.0.debit.amount", "equals": "$120.00" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: year-end legal accrual", "lines": [ { "account": "6300", "debit_cents": 650000 }, { "account": "Accrued Expenses", "credit_cents": 650000 } ] }, "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "pending_approval" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "proposal", "id": "{{step4.proposal_id}}" }, "expect": { "path": "proposal.status", "equals": "waiting" } }
]
```
