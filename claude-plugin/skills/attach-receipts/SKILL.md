---
name: attach-receipts
description: "Match receipts and invoices in Documents to the payments they prove, and ask for the missing ones."
---

# Attach receipts

A payment with its receipt attached is proven; the close checklist and auditors look for them. People forward receipts by email or upload them, and Accountable reads the amount, date and vendor.

## Steps
1. `search {kind: "documents", filters: {unlinked_only: true}}`: receipts not yet linked, with what was read from each (`extracted`: vendor, date, amount). A file the person gives you: `upload_file`, then `read_file`.
2. For each, `search {kind: "transactions", filters: {search: "<vendor>", from, to}}` with a few days around the receipt date. A match has the same amount (payments are negative, receipts positive).
3. `run {action: "attach_document", input: {document_id, transaction_id}}` (or `entry_id` for a journal entry).
4. For a large payment with no receipt, `run {action: "add_comment", input: {object_type: "transaction", object_id, body}}` asking the person to forward it.
5. The reader got a field wrong (vendor, date, amount, or a statement's balances): `update_document` with the corrected fields; the rest are kept. A receipt on the wrong payment: `detach_document`, then `attach_document` to the right one.

## Worked example
```json
[
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "documents", "filters": { "unlinked_only": true, "limit": 5 } }, "expect": { "path": "documents" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "transactions", "filters": { "limit": 1 } }, "expect": { "path": "transactions.0.id" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "add_comment", "input": { "object_type": "transaction", "object_id": "{{step2.transactions.0.id}}", "body": "Could you forward the receipt for this payment?" } }, "expect": { "path": "preview_id" } }
]
```
