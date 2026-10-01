---
name: start-here
description: "The 13 tools, how they reach every action, and the ids to resolve first."
---

# Start here

Accountable keeps double-entry books for one or more companies. You run every part of them through 13 tools. Every change is logged with your client's name and the person whose connection you use, and every change can be undone.

## The tools
- `list_companies`: the companies you can open, your role in each, the month each is closed through, and what this connection may do.
- `find_actions`: searches everything Accountable can do ("vendor rule", "closed month", "send invoice"). Each result has the action's name, whether it reads or writes, and its `input_schema`.
- Reading: `search {kind}` lists records (transactions, entries, accounts, rules, bills...), `get {kind, id}` returns one record with its history, `report {name}` runs a statement or metric, `explain {what}` says why a number or row is what it is, and `read {action, input}` runs any other read action.
- Changing: `run {action, input}` applies a write action in one call; `preview` shows its exact effect first without changing anything; `undo {change_id}` reverses a change.
- Files: `upload_file` adds a receipt or statement, `read_file` reads one.
- `load_skill` with a name loads a guide; with no name it lists them.

## First calls
1. `list_companies`. Pass `company` everywhere else as the id or the exact name ("Northwind Labs" or "northwind labs, inc."). Every response echoes `company {id, name}`: check it.
2. `load_skill {name: "run-the-books"}` before your first change.
3. An outside accountant with several clients: `read {action: "list_clients"}` shows each client's closed-through month, open questions, changes waiting and feed health.

## Money and size
- Money is `{amount_cents, amount: "−$480.00", currency: "USD"}`. Inputs take cents (`debit_cents: 125000` is $1,250.00).
- Lists return a page and a `next_cursor`. Narrow with filters or `fields` when a response is large.

## Worked example
```json
[
  { "tool": "list_companies", "args": {}, "expect": { "path": "companies.0.name" } },
  { "tool": "get", "args": { "company": "northwind labs, inc.", "kind": "company" }, "expect": { "path": "company.name", "equals": "Northwind Labs" } },
  { "tool": "find_actions", "args": { "query": "categorize transactions" }, "expect": { "path": "actions.0.input_schema" } },
  { "tool": "load_skill", "args": {}, "expect": { "path": "skills.0.name" } }
]
```
