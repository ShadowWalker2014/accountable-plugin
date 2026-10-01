---
name: plan-runway
description: "Answer \"what if we hire, cut or raise\" from the live books, and save the scenario the founder wants to keep."
---

# Plan runway

Runway questions start from the real books: cash today and net burn over recent complete months. Planning never posts entries.

## Steps
1. `report {name: "metrics"}`: cash, net burn, runway, and how each was computed. Read `warnings[]` first: open months or a stale feed change the answer.
2. `search {kind: "scenarios"}`: runway on current burn and on each saved scenario.
3. `read {action: "what_if", input: {drivers}}` answers a question without saving anything:
   - hire: `{id, kind: "hire", title, start: "YYYY-MM-DD", salaryCents}` (annual salary; `loadedFactor` defaults to 1.25 for taxes and benefits)
   - cost: `{id, kind: "cost", label, monthlyCents, from: "YYYY-MM"}` (negative cuts spend)
   - revenue: `{id, kind: "revenue", from, growthPct}` (monthly growth)
   - one_off: `{id, kind: "one_off", label, month, amountCents}`
   - raise: `{id, kind: "raise", amountCents, closeMonth}`
4. Say the answer as months of runway and the month cash runs out, before and after.
5. When the founder wants to keep it, `run {action: "save_scenario"}`. Locking a scenario as the company plan is an owner or admin decision in Accountable.
6. `report {name: "plan_vs_actual"}` compares the locked plan with each complete month: net burn, revenue, spend and cash, and `why` names the line behind most of the gap.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "metrics" }, "expect": { "path": "metrics" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "scenarios" }, "expect": { "path": "current_burn.runway_months" } },
  { "tool": "read", "args": { "company": "Northwind Labs", "action": "what_if", "input": { "drivers": [ { "id": "d1", "kind": "hire", "title": "Senior engineer", "start": "{{today}}", "salaryCents": 20000000 } ] } }, "expect": { "path": "with_changes.runway_months" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "plan_vs_actual" }, "expect": { "path": "months" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "save_scenario", "input": { "name": "Hire a senior engineer", "drivers": [ { "id": "d1", "kind": "hire", "title": "Senior engineer", "start": "{{today}}", "salaryCents": 20000000 } ] } }, "expect": { "path": "preview_id" } }
]
```
