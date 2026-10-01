---
name: clean-up-vendors
description: "Find every transaction from a vendor by its bank text, give it one clean name and category, and teach a rule that also fixes the past."
---

# Clean up vendors

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Bank text is messy: "AMAZON WEB SERVICES AWS.AMAZON.CO WA", "AWS EMEA" and "Amazon Web Services" are one vendor. Clean it once and the books, the reports and the 1099s read right.

## Steps
1. `list_transactions {search: "<vendor text>"}` finds every row whose bank text or name contains the words. Add `from` and `to` for a date range, and `min_amount_cents` or `max_amount_cents` for amounts. Page with `cursor`.
2. `list_rules`: a rule for this vendor may already exist. Fix it (`update_rule`) instead of adding a second one.
3. Teach a rule: `create_rule {match_value, account, set_description, apply_to_existing: true}`.
   - `set_description` renames every matching row to the clean name.
   - `apply_to_existing: true` also recategorizes past rows in open months. The preview lists every row it changes and counts the closed-month rows it leaves alone.
   - A rule that changes past rows applies at once under the default rules; the owner can make it wait in Settings › Approval rules.
4. Rows the rule does not cover (one-offs, rows categorized by hand): put the rename and the category with one call each:
   `set_transaction_fields {transaction_ids, description}`, then `categorize_transactions {transaction_ids, account}`.
5. Closed-month rows: load `change-a-closed-month`.
6. Say what changed: the rule, how many rows, and the `change_id`s. `undo_change` reverses any of them.

## Rules
- Categories a person set by hand are theirs: rules skip them unless you pass `include_manual: true` to `apply_rule`.
- Rename with the vendor's plain name ("Amazon Web Services"), never the bank's text.
