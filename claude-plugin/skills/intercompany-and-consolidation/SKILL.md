---
name: intercompany-and-consolidation
description: "Post entries between the founder's companies on both sides at once, and read consolidated statements with intercompany eliminated."
---

# Intercompany and consolidation

When one company pays for another, lends to it, or charges it a fee, the entry belongs in both companies' books: a receivable or loan on one side, a payable or loan on the other. `post_intercompany` writes both sides at once through each company's own approval rules; if either side cannot post, neither does.

## Post an intercompany entry
1. `list_companies`: both companies must be reachable by this connection, and you need write access to both.
2. `preview {company, action: "post_intercompany", input}`: `company` is the side the money or service comes from; the input has `to_company`, `kind` (`expense_on_behalf`, `cost_share`, `loan_advance`, `loan_repayment`, `interest`, `management_fee`, `capital`, `other`), `date`, `amount_cents`, and for most kinds the other account on each side (`from_account`, `to_account`, e.g. the cash account that paid).
3. Read both sides' lines and each company's `policy`, then `run` with the `preview_id` and an `idempotency_key`.
4. `pending_approval` means at least one side waits for a person; both post together once approved. `get {kind: "intercompany", id}` shows the state.

## Read the group
Each company is its own tax entity: companies are never added up just because one person runs them. Consolidated statements exist only for a group someone created, a parent company and the companies it owns.
- `search {kind: "groups"}`: the groups (parent, subsidiaries, ownership, reporting currency). An empty list means there is nothing to consolidate: report on each company on its own.
- `report {name: "consolidated", params: {group_id, report, period}}` with `report` `pl`, `bs`, `cf` or `tb`. Amounts are in the group's `currency` (a subsidiary in another currency is translated). Intercompany balances are eliminated. Read `warnings` and `differences`: a difference means the two sides of an intercompany balance do not match, usually because one side was posted outside `post_intercompany`. Consolidated statements need the Holding plan.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "groups" }, "expect": { "path": "groups.0.currency", "equals": "USD" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "intercompany" }, "expect": { "path": "entries" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "post_intercompany", "input": { "to_company": "Halcyon Holdings", "kind": "expense_on_behalf", "date": "{{today}}", "amount_cents": 25000, "memo": "Skill test: shared software", "from_account": "1000", "to_account": "6100" } }, "expect": { "path": "preview_id" } }
]
```
