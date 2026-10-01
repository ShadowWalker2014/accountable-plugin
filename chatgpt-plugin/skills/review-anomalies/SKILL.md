---
name: review-anomalies
description: "Work through what the detectors flagged, label each alert, and explain why a number moved between two months with its transactions."
---

# Review anomalies and explain changes

> **Using this guide in ChatGPT.** Each Accountable action is its own tool here, named in the steps below.
> - A tool that changes the books previews by default and changes nothing: call it, show the person the preview, then call it again with the same input, `mode: "apply"` and the `preview_id`. Pass an `idempotency_key` so a retry never applies twice.
> - Where a step says `run` or `preview` an action, call that action's tool. A batch is one call per action here.
> - `undo_change` reverses any change by its `change_id`, previewing first in the same way.
> - A step that names a tool or guide this plugin doesn't include is done in Accountable itself at https://accountable.im.

Detectors watch every posted transaction: duplicate payments, unusual amounts, large first payments to a vendor, lookalike payees, a vendor paid to a new bank account, foreign card charges, assets below zero, transfers posted to equity, refunds counted as revenue. An alert never changes the books.

## Alerts
1. `list_alerts`: open alerts, each with its evidence and transactions.
2. `get_transaction {transaction_id}` on its transactions to check the vendor, the category and who categorized it.
3. Decide, and `label_alert`:
   - A normal pattern (an annual renewal, a known one-off): `label: "expected"` and a short `note`. The same vendor and amount stay quiet next time.
   - A real problem (a double charge, a payment to a lookalike vendor): `label: "real_problem"`, then tell the person. For a double charge, `add_comment` on the transaction asking whether to request a refund. Never fix a real problem silently.
   - Noise: `label: "dismissed"`.

## Why did a number change?
- `explain_change` with `scope: "account"` (an account id, code or name) or `scope: "metric"` (`revenue`, `spend`, `net_burn`, `gross_margin`), `from` and `to` months. The drivers (new vendor, price change, one-off, timing, recategorization...) add up to the change exactly; quote the `sentence` of the largest drivers and name their transactions.
