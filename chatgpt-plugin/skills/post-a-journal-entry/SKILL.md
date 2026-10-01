---
name: post-a-journal-entry
description: "Accruals, reclassifications and corrections as balanced entries, and which ones wait for an owner."
---

# Post a journal entry

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Use a journal entry for accruals, reclassifications and corrections that no bank transaction covers. For a miscategorized payment, recategorize the transaction instead (`categorize_transactions`).

## Steps
1. `list_accounts {search}` finds each account. Pass an account's id, code ("6300") or exact name ("Legal Fees").
2. `create_journal_entry {date, memo, lines}`: at least two lines, each with `debit_cents` or `credit_cents`. Debits must equal credits to the cent. For an accrual that should reverse, add `reverse_on`.
3. The result is `applied` with the `change_id` and the posted entry, or `pending_approval`.
4. `get_entry {entry_id}` reads it back with its lines.

## What waits
- Entries at or over the owner's limit ($5,000 unless the owner changed it) wait for an owner: you get `pending_approval` with an `approval_url`. Tell the person and check `get_proposal {proposal_id}` later.
- Smaller entries apply at once under the default rules. The owner can require approval in Settings › Approval rules.
- A closed month refuses the entry. Date it in an open month, or load `change-a-closed-month`.
- Posted entries are never edited in place. To correct one, `reverse_entry` and post the right entry.

## Common errors
- `UNBALANCED`: the message names the difference.
- `ACCOUNT_INACTIVE`: pick another account or ask a person to reactivate it.
- `PERIOD_CLOSED` or `POLICY_FORBIDS`: the month is closed; see above.
