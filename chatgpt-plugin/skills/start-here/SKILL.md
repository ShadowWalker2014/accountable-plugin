---
name: start-here
description: "The tools Accountable offers in ChatGPT, how a change previews before it applies, and the first calls to make."
---

# Start here

Accountable keeps double-entry books for one or more companies. In ChatGPT each thing it can do is its own tool. Every change is logged with this client's name and the person whose connection you use, and every change can be undone.

## First calls
1. `list_companies`: the companies you can open, your role in each, the month each is closed through, and whether this connection may change the books. Pass `company` to every other tool as the id or the exact name ("Northwind Labs").
2. Read before you change anything: `get_report`, `get_metrics` and `list_transactions` answer most questions.

## Changes preview first
- A tool that changes the books previews by default: it returns exactly what would change and a `preview_id`, and changes nothing. Show the person the preview. Then call the same tool with the same input, `mode: "apply"` and the `preview_id`.
- Pass an `idempotency_key` on apply, so a retry never applies twice.
- The result is `applied` with a `change_id`, or `pending_approval` when the company's rules need a person; say so and share the `approval_url`.
- `undo_change` reverses a change by its `change_id`, the same way: preview, then apply.
- A role that can't make a change gets `FORBIDDEN`, naming the roles that can. Nothing here moves money: payments are recorded, never sent.

## The tools

### Companies
- `list_companies`: List companies.
- `get_company`: Get a company.

### Reports and analysis
- `get_report`: Get a financial report.
- `get_metrics`: Get the founder metrics.
- `get_spend_or_revenue`: Analyze spend or revenue.
- `explain_change`: Explain why a number changed.
- `get_lineage`: Where a number comes from.
- `compare_reports`: Compare what the books said then and now.
- `what_if`: What if (runway).
- `list_scenarios`: List planning scenarios.
- `get_plan_vs_actual`: Plan vs actual.
- `list_alerts`: List alerts and anomaly questions.

### Transactions, entries and accounts
- `list_transactions`: List transactions.
- `get_transaction`: Get a transaction.
- `list_accounts`: List accounts.
- `query_ledger`: Query the ledger.
- `get_entry`: Get a journal entry.
- `list_rules`: List categorization rules.
- `list_questions`: List review questions.

### The close
- `get_close_status`: Get close status.
- `get_close_checklist`: Get a month's close checklist.

### Bills and invoices
- `list_bills`: List bills.
- `get_bill`: Get a bill.
- `list_invoices`: List invoices.
- `get_invoice`: Get an invoice.
- `get_aging`: Get receivables or payables aging.

### History and approvals
- `get_audit_log`: Get the audit log.
- `get_change`: Get a change.
- `get_proposal`: Get a proposal.

### Changes
- `categorize_transactions`: Categorize transactions (previews first).
- `answer_question`: Answer a review question (previews first).
- `set_transaction_fields`: Rename a vendor or set a memo (previews first).
- `create_rule`: Create a categorization rule (previews first).
- `create_journal_entry`: Create a journal entry (previews first).
- `save_journal_draft`: Save a journal entry draft (previews first).
- `reverse_entry`: Reverse a journal entry (previews first).
- `label_alert`: Label an alert (previews first).
- `add_comment`: Add a comment (previews first).
- `save_scenario`: Save a planning scenario (previews first).
- `undo_change`: Undo a change (previews first).

## Money and size
- Money is `{amount_cents, amount, currency}`. Inputs take cents (`debit_cents: 125000` is $1,250.00).
- Lists return a page and a `next_cursor`.
