---
name: run-the-books
description: "How every change works: find the action, run it in one call with an idempotency key, batch several as one change, change a closed month, and undo."
---

# Run the books

You are the person's bookkeeper and controller. Every capability is an action; `find_actions` finds it and `run` applies it.

## Steps
1. `list_companies`: pick the company, and read `write_policy` to know what applies at once for this connection.
2. `find_actions {query}` with plain words ("rename vendor", "reverse entry", "close month"). Use the action's `input_schema`: inputs take cents and accept an account's id, code ("6300") or exact name.
3. `run {company, action, input, idempotency_key}` applies it in one call. The person already confirmed it in your client's own prompt, so there is no second step.
   - `status: "applied"`: you get `change_id`, the `summary` and the re-read `record`.
   - `status: "pending_approval"`: a person must approve it at `approval_url`. No tool can approve. Tell the person, and check later with `get {kind: "proposal", id}`.
4. Not sure what a change will do? `preview` with the same arguments first: every row before and after, balances, closed months touched, and whether it applies or waits. Then `run` with its `preview_id` to apply exactly that.

## Idempotency keys
Pass a new `idempotency_key` (any unique string of 8+ characters) on every `run`. If a call times out, send it again with the same key: you get the first result with `replayed: true`, and nothing happens twice.

## Several changes as one
`run {batch: [{action, input}, ...]}` applies up to 200 actions as one change with one `change_id` and one undo. Each is checked on the books as the earlier ones leave them; if one fails, nothing applies. Use it for a cleanup that belongs together: rename a vendor and recategorize its rows.

## Closed months
A closed month changes only inside one logged reopen, change and relock. Pass `closed_month {month, reason}` to `run` with the changes in `batch`. Load `change-a-closed-month`.

## What waits for a person
Under the default guardrails, changes apply at once except:
- journal entries at or over the owner's limit ($5,000 unless the owner changed it);
- sending anything to people outside the company;
- changes in closed months, and reopening a month.

Under Full control everything the person's role allows applies at once. The owner's approval rules (Settings › Approval rules) bind agents like people.

## Undo
`undo {change_id}` reverses any change, including a batch or a closed-month change, and links the two. `undo {change_id, preview_only: true}` shows what it restores first.

## Worked example
```json
[
  { "tool": "list_companies", "args": {}, "expect": { "path": "companies.0.name" } },
  { "tool": "find_actions", "args": { "query": "journal entry" }, "expect": { "path": "actions.0.input_schema" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: run the books", "lines": [ { "account": "6310", "debit_cents": 4200 }, { "account": "Accrued Expenses", "credit_cents": 4200 } ] } }, "expect": { "path": "policy.decision", "equals": "apply" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: run the books", "lines": [ { "account": "6310", "debit_cents": 4200 }, { "account": "Accrued Expenses", "credit_cents": 4200 } ] }, "preview_id": "{{step3.preview_id}}", "idempotency_key": "{{run_key}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: run the books", "lines": [ { "account": "6310", "debit_cents": 4200 }, { "account": "Accrued Expenses", "credit_cents": 4200 } ] }, "idempotency_key": "{{run_key}}" }, "expect": { "path": "change_id", "equals": "{{step4.change_id}}" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "entry", "id": "{{step4.record.entry_id}}" }, "expect": { "path": "entry.memo", "equals": "Skill test: run the books" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "batch": [ { "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: batch one", "lines": [ { "account": "6310", "debit_cents": 1000 }, { "account": "1000", "credit_cents": 1000 } ] } }, { "action": "create_journal_entry", "input": { "date": "{{today}}", "memo": "Skill test: batch two", "lines": [ { "account": "6310", "debit_cents": 2000 }, { "account": "1000", "credit_cents": 2000 } ] } } ], "idempotency_key": "{{uuid}}" }, "requires": "apply_batch", "expect": { "path": "status", "equals": "applied" } },
  { "tool": "undo", "args": { "company": "Northwind Labs", "change_id": "{{step4.change_id}}", "preview_only": true }, "expect": { "path": "diff.rows_affected", "equals": 2 } },
  { "tool": "undo", "args": { "company": "Northwind Labs", "change_id": "{{step4.change_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } }
]
```
