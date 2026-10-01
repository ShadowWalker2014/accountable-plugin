---
name: answer-review-questions
description: "Work the review queue: one answer per vendor question categorizes all its transactions and teaches a rule."
---

# Answer review questions

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

The review queue groups uncategorized transactions into one question per vendor and direction ("Is AWS hosting or staging?"). Answering a question categorizes every transaction in it and, by default, saves a rule so the vendor never asks again.

## Steps
1. `list_questions`: each question has `vendor`, `total`, `transaction_ids`, and the AI's `suggested_account` with `confidence` and `reason`.
2. Check the suggestion against the company's history: `list_transactions {search: "<vendor>"}`.
3. `answer_question {transaction_ids, account}`. Pass `always: false` for a one-off.
4. When you cannot tell (a transfer, a personal charge, a vendor with two uses), leave the question and tell the person what you need to know.
