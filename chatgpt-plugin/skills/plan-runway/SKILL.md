---
name: plan-runway
description: "Answer \"what if we hire, cut or raise\" from the live books, and save the scenario the founder wants to keep."
---

# Plan runway

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Runway questions start from the real books: cash today and net burn over recent complete months. Planning never posts entries.

## Steps
1. `get_metrics`: cash, net burn, runway, and how each was computed. Read `warnings[]` first: open months or a stale feed change the answer.
2. `list_scenarios`: runway on current burn and on each saved scenario.
3. `what_if {drivers}` answers a question without saving anything:
   - hire: `{id, kind: "hire", title, start: "YYYY-MM-DD", salaryCents}` (annual salary; `loadedFactor` defaults to 1.25 for taxes and benefits)
   - cost: `{id, kind: "cost", label, monthlyCents, from: "YYYY-MM"}` (negative cuts spend)
   - revenue: `{id, kind: "revenue", from, growthPct}` (monthly growth)
   - one_off: `{id, kind: "one_off", label, month, amountCents}`
   - raise: `{id, kind: "raise", amountCents, closeMonth}`
4. Say the answer as months of runway and the month cash runs out, before and after.
5. When the founder wants to keep it, `save_scenario`. Locking a scenario as the company plan is an owner or admin decision in Accountable.
6. `get_plan_vs_actual` compares the locked plan with each complete month: net burn, revenue, spend and cash, and `why` names the line behind most of the gap.
