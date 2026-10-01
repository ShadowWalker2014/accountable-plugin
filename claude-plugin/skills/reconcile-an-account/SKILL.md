---
name: reconcile-an-account
description: "Tie a bank or card account to its statement for a month, find the difference, and mark it reconciled."
---

# Reconcile an account

Reconciling proves the books match the bank: the ledger balance at the statement date, minus entries the bank has not shown yet, equals the statement's ending balance to the cent.

## Steps
1. `report {name: "reconciliation", params: {month, account}}`. Each account shows `statement_balance`, `status` (`reconciled`, `ties`, `difference`, `statement_missing`) and `difference`.
2. `statement_missing`: ask the person for the statement, or `upload_file` it and import it.
3. `difference`: find it. `search {kind: "entries", filters: {account, from, to}}` shows every posting; `search {kind: "transactions", filters: {account}}` shows what the feed recorded. Common causes: a missing or duplicated transaction, a transfer posted to the wrong side, a payment dated across the month end.
4. Entries in the books that the bank shows next month (a check not cashed yet) are uncleared: pass their ids as `uncleared_entry_ids`.
5. `preview {action: "propose_reconciliation", input: {account, month, statement_balance_cents}}`. It fails with `RECONCILE_DIFFERENCE` and the exact amount when it does not tie. When it ties, `run` it.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "reconciliation", "params": { "month": "{{this_month}}", "account": "Mercury Checking" } }, "expect": { "path": "accounts.0.account.code", "equals": "1000" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "entries", "filters": { "account": "1000", "from": "{{this_month}}-01", "limit": 5 } }, "expect": { "path": "entries" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "propose_reconciliation", "input": { "account": "Mercury Checking", "month": "{{this_month}}", "statement_balance_cents": 1 } }, "expect": { "error": "RECONCILE_DIFFERENCE" } }
]
```
