---
name: read-reports
description: "Pull P&L, balance sheet and trial balance for whole months or exact days, respect the readiness warnings, and drill into a line."
---

# Read reports

`report {name, params}` runs any statement: `pl`, `bs`, `tb`, `cf` or `gl`, with `to` and for ranges `from`, and `basis` (cash or accrual). Each bound is a month (YYYY-MM, the whole month) or a day (YYYY-MM-DD, that exact day; both days are included, and a range past today stops at today). `period` takes a token instead: 2026-09, 2026-Q3, 2026, 2026-07-08..2026-08-20, 2026-07-08 or all. Balance sheet and trial balance are as of the last day of `to`. `report {name: "close_status"}` says which months are closed.

## Read the warnings first
`warnings[]` lists reasons a number may be wrong or change: a stale feed (named, with its last sync date), months after the closed-through month, a cash account below zero (missing opening balances), and the share of revenue and expenses still uncategorized. Say them to the person before quoting figures.

## Drill down
Every amount is `{amount_cents, amount, currency}`. To see what makes up a line: `search {kind: "entries", filters: {account, from, to}}`, or `search {kind: "transactions", filters: {category}}`. To say why it moved: `explain {what: "change"}`.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "pl", "params": { "from": "{{last_month}}", "to": "{{last_month}}" } }, "expect": { "path": "data.totals.net_income.amount" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "tb", "params": { "to": "{{last_month}}" } }, "expect": { "path": "data.totals.difference.amount_cents", "equals": 0 } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "entries", "filters": { "account": "6100", "limit": 5 } }, "expect": { "path": "entries.0.id" } }
]
```
