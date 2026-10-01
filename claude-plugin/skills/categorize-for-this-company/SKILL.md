---
name: categorize-for-this-company
description: "Learn how this company categorizes (its rules and past choices), then categorize, split or rename transactions the same way."
---

# Categorize for this company

Categorize the way this company already does. Its rules and the categories people chose before are the answer key; your own guess comes last.

## Steps
1. `search {kind: "rules"}`: the company's rules. A rule that matches a vendor is the answer for that vendor.
2. `search {kind: "accounts", filters: {type: "expense"}}` (and `revenue`, `cost_of_revenue`): the categories you may use. Use the account's code or exact name.
3. `search {kind: "transactions", filters: {uncategorized_only: true}}`: the work. For a vendor you are unsure about, search with `search: "<vendor>"` to see how its earlier payments were categorized.
4. `run {action: "categorize_transactions", input: {transaction_ids, account}}`: group the same vendor into one call. `preview` first shows every row before and after.
5. A payment covering several things (an AWS bill split between hosting and staging, a Gusto run): `split_transaction` with `parts` that add up to the amount.
6. Messy bank text: `set_transaction_fields` with a clean `description` ("Figma" instead of "FIGMA* SUBSCRIPTION 4821"). For a whole vendor, load `clean-up-vendors`.
7. A vendor that will repeat: pass `always: true`, or `create_rule`. A rule with `apply_to_existing: true` also recategorizes past rows in open months.
8. Rules exist but rows are still uncategorized (a new rule, a late feed): `run_rules` runs every active rule over them. `apply_rule` runs one rule over past rows in open months; it keeps categories people set by hand unless `include_manual: true`.
9. A rule that keeps choosing wrong: `update_rule` to fix its match or account, `active: false` to pause it, or `delete_rule`. Rows it already categorized keep their category.

## Rules
- Rows in closed months change only through `change-a-closed-month`.
- Categories a person set by hand are theirs; rules never override them.
- When unsure, leave it for the review queue and say why instead of guessing.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "rules" }, "expect": { "path": "rules" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "transactions", "filters": { "uncategorized_only": true, "limit": 1 } }, "expect": { "path": "transactions.0.id" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "categorize_transactions", "input": { "transaction_ids": ["{{step2.transactions.0.id}}"], "account": "6600" } }, "expect": { "path": "diff.transactions.0.id", "equals": "{{step2.transactions.0.id}}" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "categorize_transactions", "input": { "transaction_ids": ["{{step2.transactions.0.id}}"], "account": "6600" }, "preview_id": "{{step3.preview_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "transaction", "id": "{{step2.transactions.0.id}}" }, "expect": { "path": "transaction.id" } }
]
```
