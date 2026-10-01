---
name: answer-review-questions
description: "Work the review queue: one answer per vendor question categorizes all its transactions and teaches a rule."
---

# Answer review questions

The review queue groups uncategorized transactions into one question per vendor and direction ("Is AWS hosting or staging?"). Answering a question categorizes every transaction in it and, by default, saves a rule so the vendor never asks again.

## Steps
1. `search {kind: "questions"}`: each question has `vendor`, `total`, `transaction_ids`, and the AI's `suggested_account` with `confidence` and `reason`.
2. Check the suggestion against the company's history: `search {kind: "transactions", filters: {search: "<vendor>"}}`.
3. `run {action: "answer_question", input: {transaction_ids, account}}`. Pass `always: false` for a one-off.
4. When you cannot tell (a transfer, a personal charge, a vendor with two uses), leave the question and tell the person what you need to know.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "questions", "filters": { "limit": 1 } }, "expect": { "path": "questions.0.transaction_ids" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "answer_question", "input": { "transaction_ids": "{{step1.questions.0.transaction_ids}}", "account": "6600", "always": false }, "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } }
]
```
