---
name: schedules-and-accruals
description: "Spread prepaid contracts, fixed assets and deferred revenue over their months, accrue expenses before the bill arrives, and read either basis."
---

# Schedules and accruals

Cash books record a payment when money moves. Accrual books record it when it is earned or used: an annual software contract paid in January is an expense of $X a month all year; a laptop is depreciated over its life; revenue billed a year ahead is earned month by month. Accountable keeps both views from one ledger; schedules post the accrual side.

## Work the suggestions first
1. `search {kind: "schedule_suggestions"}`: payments the books think are prepaid, fixed assets, accruals or deferred revenue, each with the proposed months, accounts and a confidence.
2. Check one against its payment (`get {kind: "transaction"}`) and document. If it is right, `run {action: "accept_schedule_suggestion"}` (`preview` shows every entry first). If not (a one-off, already handled), `dismiss_schedule_suggestion` with a `reason`; `dont_ask_again` for a vendor that never needs one.

## Create one yourself
- `create_schedule` with `kind`, `description`, `total_cents`, `start_month`, `months`, and the accounts. Fixed assets take an `asset_class` for the account and useful life.
- `create_accrual` for an expense incurred but not yet billed; it reverses on the 1st of the next month.
- `create_recurring_accrual` for a cost billed late every month (payroll taxes, a contractor): each month end it books the accrual from the last payment, the 3-month average or a fixed amount.
- `skip_schedule_month` moves a planned month to the end (a paused contract).
- Past months catch up in one entry in the current month. Posting into each past closed month separately is an owner's choice in the app.

## Each month
- `run {action: "post_schedule_entries", input: {period}}` posts everything due (the close does this too; a second run posts nothing).
- `search {kind: "schedules", filters: {kind}}` per register shows what has posted and what is left.

## Reading both bases
- `report {name: "basis"}` says which basis the company reports on and whether accrual books are available.
- `report {name: "pl", params: {basis: "accrual"}}` (or `"cash"`) reads either view.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "basis" }, "expect": { "path": "status" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "schedule_suggestions" }, "expect": { "path": "suggestions" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "create_schedule", "input": { "kind": "prepaid", "description": "Skill test: annual design tool", "total_cents": 48000, "start_month": "{{this_month}}", "months": 12, "pl_account": "6100", "source_account": "6100" } }, "expect": { "path": "diff.rows_affected" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "schedules", "filters": { "kind": "prepaid" } }, "expect": { "path": "schedules" } }
]
```
