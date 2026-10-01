---
name: clean-up-vendors
description: "Find every transaction from a vendor by its bank text, give it one clean name and category, and teach a rule that also fixes the past."
---

# Clean up vendors

Bank text is messy: "AMAZON WEB SERVICES AWS.AMAZON.CO WA", "AWS EMEA" and "Amazon Web Services" are one vendor. Clean it once and the books, the reports and the 1099s read right.

## Steps
1. `search {kind: "transactions", filters: {search: "<vendor text>"}}` finds every row whose bank text or name contains the words. Add `from` and `to` for a date range, and `min_amount_cents` or `max_amount_cents` for amounts. Page with `cursor`.
2. `search {kind: "rules"}`: a rule for this vendor may already exist. Fix it (`update_rule`) instead of adding a second one.
3. Teach a rule: `run {action: "create_rule", input: {match_value, account, set_description, apply_to_existing: true}}`.
   - `set_description` renames every matching row to the clean name.
   - `apply_to_existing: true` also recategorizes past rows in open months. The preview lists every row it changes and counts the closed-month rows it leaves alone.
   - A rule that changes past rows applies at once under the default rules; the owner can make it wait in Settings › Approval rules.
4. Rows the rule does not cover (one-offs, rows categorized by hand): put the rename and the category in one `batch`, so they apply and undo as one change:
   `run {batch: [{action: "set_transaction_fields", input: {transaction_ids, description}}, {action: "categorize_transactions", input: {transaction_ids, account}}]}`.
5. Closed-month rows: load `change-a-closed-month`.
6. Say what changed: the rule, how many rows, and the `change_id`s. `undo` reverses any of them.

## Rules
- Categories a person set by hand are theirs: rules skip them unless you pass `include_manual: true` to `apply_rule`.
- Rename with the vendor's plain name ("Amazon Web Services"), never the bank's text.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "transactions", "filters": { "uncategorized_only": true, "limit": 1 } }, "expect": { "path": "transactions.0.description" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "transactions", "filters": { "search": "{{step1.transactions.0.description}}", "limit": 5 } }, "expect": { "path": "transactions.0.id" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "rules" }, "expect": { "path": "rules" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "create_rule", "input": { "match_value": "{{step1.transactions.0.description}}", "account": "6100", "set_description": "Skill test vendor", "apply_to_existing": true } }, "expect": { "path": "policy.decision", "equals": "apply" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_rule", "input": { "match_value": "{{step1.transactions.0.description}}", "account": "6100", "set_description": "Skill test vendor", "apply_to_existing": true }, "preview_id": "{{step4.preview_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "set_transaction_fields", "input": { "transaction_ids": ["{{step1.transactions.0.id}}"], "memo": "Cleaned up with the vendor rule" }, "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "batch": [ { "action": "set_transaction_fields", "input": { "transaction_ids": ["{{step1.transactions.0.id}}"], "description": "Skill test vendor" } }, { "action": "categorize_transactions", "input": { "transaction_ids": ["{{step1.transactions.0.id}}"], "account": "6100" } } ], "idempotency_key": "{{uuid}}" }, "requires": "apply_batch", "expect": { "path": "status", "equals": "applied" } },
  { "tool": "undo", "args": { "company": "Northwind Labs", "change_id": "{{step5.change_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } }
]
```
