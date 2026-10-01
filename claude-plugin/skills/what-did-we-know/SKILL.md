---
name: what-did-we-know
description: "Show a report as the books were recorded at a moment (or as a closed month was frozen), compare it with now, and name the changes in between."
---

# Answer "what did we know on date X"

Investors, auditors and boards ask what the numbers said when a decision was made: "what did the September P&L show when we sent the board deck on October 12?" The books keep every change with its time, so any report can be read as it was recorded at a moment.

## Steps
1. `report {name: "pl", params: {to, known_at}}` (any statement). `known_at` is a date, meaning the end of that day in UTC, or a date and time with its offset such as `2026-10-12T17:00-07:00`. The response says how many changes were recorded since, and how many of those posted into this period.
2. For a closed month, pass `version` (1 is the first close) instead: the statement exactly as frozen at that close; `from` and `to` are the same month.
3. `report {name: "compare", params: {report, to, a_known_at}}` (or `a_version`), with `b` defaulting to now: only the lines that differ, each with its change ids.
4. `get {kind: "change", id}` on those ids says who changed what, when and why (a late invoice, a recategorization, a reopened month).
5. `explain {what: "number", params: {accounts, from, to}}` traces any number today to its entries, transactions, documents and the changes that posted them.

Say the answer as: "On Oct 12 the September P&L showed a net loss of $X. Since then N changes moved it to $Y: …".

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "pl", "params": { "to": "{{last_month}}", "known_at": "{{today}}" } }, "expect": { "path": "as_of.known_at" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "compare", "params": { "report": "pl", "to": "{{last_month}}", "a_known_at": "{{today}}" } }, "expect": { "path": "lines_differing", "equals": 0 } },
  { "tool": "explain", "args": { "company": "Northwind Labs", "what": "number", "params": { "accounts": ["6100"], "from": "{{last_month}}-01", "to": "{{today}}" } }, "expect": { "path": "total.currency", "equals": "USD" } }
]
```
