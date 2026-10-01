---
name: bills-and-invoices
description: "Enter and check vendor bills, create and issue invoices, and record payments against bank transactions."
---

# Bills and invoices

Bills (money the company owes) and invoices (money customers owe) post on the accrual basis: a bill posts to Accounts payable when it is approved, and an invoice posts to Accounts receivable when it is issued. Payments are recorded by matching the bank or card transaction that moved the money. Accountable never moves money.

## Bills
1. `search {kind: "bills", filters: {tab: "review"}}`: bills read from email or uploads. `get {kind: "bill", id}` shows `problem` when the lines do not add up, and any field the reader was unsure of.
2. `run {action: "update_bill"}` to fix the vendor, dates, amounts or line accounts. Use the company's usual expense accounts (load `categorize-for-this-company`).
3. `run {action: "submit_bill"}` sends it for approval.
4. When it is paid, `get {kind: "bill"}` lists `payment_candidates` (outgoing transactions of the right amount). `run {action: "record_bill_payment", input: {bill_id, transaction_id}}`.

## Invoices
1. `run {action: "create_invoice"}` drafts it: customer, issue date, terms, and lines (`quantity`, `unit_price_cents`, revenue `account`). `search {kind: "items"}` is the company's catalog: pass a line's `item_id` to reuse an item, and `save_item` to add one.
2. `run {action: "issue_invoice"}` posts it to receivables. A person sends it to the customer from Accountable.
3. `search {kind: "invoices"}` shows `payment_suggestions`: incoming payments that match an open invoice. `run {action: "record_invoice_payment"}` confirms one.
4. A customer who will never pay: `run {action: "write_off_invoice"}` with the reason.

## Mistakes
- A payment matched to the wrong bill or invoice: `run {action: "remove_payment"}` with the payment id from the bill or invoice. Its entries reverse and the bank transaction goes back to how it was. Then record the right one.

## Checks
- `report {name: "aging", params: {kind: "receivable"}}` (or `"payable"`): open amounts by party and days past due. `ties_to_ledger: false` means an entry was posted to Accounts receivable or payable by hand; find it with `search {kind: "entries"}`.

## Worked example
```json
[
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "create_bill", "input": { "vendor": "Cooley LLP", "bill_date": "{{today}}", "total_cents": 120000, "lines": [ { "description": "September legal", "amount_cents": 120000, "account": "6300" } ] } }, "expect": { "path": "preview_id" } },
  { "tool": "run", "args": { "company": "Northwind Labs", "action": "create_bill", "input": { "vendor": "Cooley LLP", "bill_date": "{{today}}", "total_cents": 120000, "lines": [ { "description": "September legal", "amount_cents": 120000, "account": "6300" } ] }, "preview_id": "{{step1.preview_id}}", "idempotency_key": "{{uuid}}" }, "expect": { "path": "status", "equals": "applied" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "bills", "filters": { "tab": "review" } }, "expect": { "path": "bills.0.id" } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "create_invoice", "input": { "customer": "Acme Corp", "issue_date": "{{today}}", "lines": [ { "description": "Implementation", "quantity": 10, "unit_price_cents": 20000, "account": "4100" } ] } }, "expect": { "path": "policy.decision", "equals": "apply" } },
  { "tool": "search", "args": { "company": "Northwind Labs", "kind": "items" }, "expect": { "path": "items" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "aging", "params": { "kind": "payable" } }, "expect": { "path": "totals.total.currency", "equals": "USD" } }
]
```
