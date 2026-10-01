---
name: check-this-months-close
description: "Read the Bookkeeper's close run for a month, its 8 steps, the batch it drafted with the evidence behind each change, and answer its open questions."
---

# Check this month's close

The Bookkeeper closes each month on its own: it categorizes, gathers receipts from Gmail and Drive, answers the questions it has evidence for, reconciles, drafts schedules and accruals, writes the month-end report, and puts every change in one batch. Nothing in the batch posts until a person approves it in the app, and no tool approves it: you check the batch and answer what it could not.

## Steps
1. `get {kind: "close_run", options: {month}}`. With `run: null`, start one with `run {action: "run_close", input: {month}}` (owners, admins and accountants only).
2. Read `steps`: each of the 8 has a status and a one-line summary. A `failed` step has an `error`; say it plainly (for example "Gmail access expired") and stop.
3. `read {action: "list_close_batch", input: {month}}`: `items` are the drafted changes, each with `confidence` and `evidence` (the email, Drive file or prior answer it relied on). Check a few with low confidence or large amounts.
4. `open_questions` are what the Bookkeeper could not answer. Each says `how_to_answer`:
   - categorize: `answer_close_question` with `account` (a `review:` question needs none).
   - alert: `answer_close_question` with `label` expected or real_problem.
   - documents and reconcile: `attach_document` and `propose_reconciliation`.
   `preview` shows whether an answer applies at once or waits for a person; `run` applies it.
5. Tell the person: the run's status, how many changes the batch holds, what you answered, and the `batch_url` where they approve the batch.

## Worked example
```json
[
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "run_close", "input": { "month": "{{this_month}}" } }, "expect": { "path": "steps.7.step", "equals": "assemble" } },
  { "tool": "get", "args": { "company": "Northwind Labs", "kind": "close_run", "options": { "month": "{{this_month}}" } }, "expect": { "path": "steps.0.title", "equals": "Categorize" } },
  { "tool": "read", "args": { "company": "Northwind Labs", "action": "list_close_batch", "input": { "month": "{{this_month}}", "limit": 5 } }, "expect": { "path": "open_questions.0.how_to_answer" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "answer_close_question", "input": { "month": "{{this_month}}", "question_key": "{{step3.open_questions.0.key}}", "account": "Travel and Meals" } }, "expect": { "path": "policy.decision" } }
]
```
