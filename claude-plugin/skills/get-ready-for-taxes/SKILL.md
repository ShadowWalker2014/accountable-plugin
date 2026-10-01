---
name: get-ready-for-taxes
description: "Check the year's 1099 list and W-9s, map accounts to the return's lines, draft notes and answers for the CPA, check the CPA package is ready, and read the Delaware franchise tax. Everything is prepared for a CPA; nothing is filed."
---

# Get ready for taxes

Accountable prepares the year for a CPA: the 1099s, the return mapped to lines, the book-to-tax flags, the CPA's questions and the Delaware franchise tax. It files nothing and gives no tax advice. **You never see a TIN**, never email vendors, and never download the IRIS file or the CPA package: a person does those on the Tax page.

## Steps
1. `report {name: "1099", params: {year}}`. For each vendor with `included: true`, check `w9_status` and `w9_request` (when it was sent, reminders, when it came back). Tell the person which W-9s are missing; they request them from the Tax page at `w9_requests_url`. A vendor with no email: add it with `update_1099_vendor` so the request is one click.
2. A vendor that should be left off (for example, paid only by card) or has the wrong classification: `run {action: "exclude_1099_vendor"}` with a `reason`, or `update_1099_vendor` with the classification from its W-9.
3. `report {name: "tax_return", params: {year}}`. `ties.ties_to_books` must be true. For each account in `unmapped` or `needs_review`, pick the line (the `suggestions` are a good start) and save them with `map_tax_lines`: they stay suggestions until a person confirms them.
4. For each `flags` item (meals, penalties, depreciation), draft a note for the CPA with `note_tax_flag`, and draft the `questionnaire` answers you can support from the books with `answer_tax_questions`.
5. `read {action: "get_tax_package", input: {year}}`: `ready`, the `blockers` (December not closed, accounts without a confirmed line, lines that don't tie) and `warnings` (open flags, unanswered questions, missing W-9s) left before a person downloads it at `download_url`.
6. `report {name: "franchise_tax", params: {year}}`: for a Delaware corporation, say which method is lower and the due date. Without `inputs`, ask the person for the share counts (or read them from the cap table) and save them with `set_franchise_inputs`.

What you tell the CPA and the IRS are the company's statements. `preview` each one, and tell the person what you saved so they can check it.

## Worked example
```json
[
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "1099", "params": { "year": "{{this_year}}" } }, "expect": { "path": "due_dates.file_with_irs" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "tax_return", "params": { "year": "{{this_year}}" } }, "expect": { "path": "form", "equals": "1120" } },
  { "tool": "read", "args": { "company": "Northwind Labs", "action": "get_tax_package", "input": { "year": "{{this_year}}" } }, "expect": { "path": "download_url" } },
  { "tool": "report", "args": { "company": "Northwind Labs", "name": "franchise_tax", "params": { "year": "{{this_year}}" } }, "expect": { "path": "applies", "equals": true } },
  { "tool": "preview", "args": { "company": "Northwind Labs", "action": "note_tax_flag", "input": { "year": "{{this_year}}", "kind": "meals", "status": "seen", "note": "Client meals only; 50% deductible." } }, "expect": { "path": "preview_id" } }
]
```
