---
name: review-anomalies
description: "Work through what the detectors flagged, label each alert, and explain why a number moved between two months with its transactions."
---

# Review anomalies and explain changes

Detectors watch every posted transaction: duplicate payments, unusual amounts, large first payments to a vendor, lookalike payees, a vendor paid to a new bank account, foreign card charges, assets below zero, transfers posted to equity, refunds counted as revenue. An alert never changes the books.

## Alerts
1. `search {kind: "alerts"}`: open alerts, each with its evidence and transactions.
2. `get {kind: "transaction", id}` on its transactions to check the vendor, the category and who categorized it.
3. Decide, and `run {action: "label_alert"}`:
   - A normal pattern (an annual renewal, a known one-off): `label: "expected"` and a short `note`. The same vendor and amount stay quiet next time.
   - A real problem (a double charge, a payment to a lookalike vendor): `label: "real_problem"`, then tell the person. For a double charge, `add_comment` on the transaction asking whether to request a refund. Never fix a real problem silently.
   - Noise: `label: "dismissed"`.

## Why did a number change?
- `explain {what: "change", params}` with `scope: "account"` (an account id, code or name) or `scope: "metric"` (`revenue`, `spend`, `net_burn`, `gross_margin`), `from` and `to` months. The drivers (new vendor, price change, one-off, timing, recategorization...) add up to the change exactly; quote the `sentence` of the largest drivers and name their transactions.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "alerts", "filters": { "state": "all" } }, "expect": { "path": "alerts" } },
  { "tool": "explain", "args": { "company": "Northwind Labs", "what": "change", "params": { "scope": "metric", "id": "spend", "from": "{{two_months_ago}}", "to": "{{last_month}}" } }, "expect": { "path": "change.currency", "equals": "USD" } },
  { "tool": "explain", "args": { "company": "Northwind Labs", "what": "change", "params": { "scope": "account", "id": "6100", "from": "{{two_months_ago}}", "to": "{{last_month}}" } }, "expect": { "path": "drivers" } }
]
```
