# 06 — Requests, Letters, Expenses, Loans

All request kinds below run on the shared request framework + workflow engine (`01` §6–7). Each kind
has its own typed schema (no shared "god table").

## 1. Request catalog (employee self-service)

| Kind | Notes | Ver |
|---|---|---|
| Leave | `04` | V1 |
| Attendance regularization, overtime, WFH, on-duty | `03` | V1 |
| Letter request | §2 | V1 |
| Expense claim | §3 | V1 |
| Loan / salary advance | §4 | V1 |
| Profile/bank change | EMP-003 | V1 |
| Asset request | AST-002 | V1 |
| Resignation | OFF-001 | V1 |
| Shift swap | ATT-004 | V2 |
| Training request | `07` | V3 |
| Custom kinds (form designer) | FRM-002 | V2 |

Common behaviour: draft/save, attachments, comments, status tracking, cancel/withdraw, request on behalf,
SLA, notifications, mobile parity, copilot creation ("I need a salary certificate for my bank").

## 2. Letters

| ID | Requirement | Ver |
|---|---|---|
| LTR-001 | Letter types configurable: Salary certificate, Employment/experience certificate, NOC (travel, visa, bank), Salary transfer letter, Embassy letter, Relieving letter, Warning, Increment, Promotion, Custom. | V1 |
| LTR-002 | Per type: template(s) per language/entity, required inputs (addressee, purpose, bank name), whether salary is shown, approval workflow (or auto-approve), signatory rules (digital signature image + stamp). | V1 |
| LTR-003 | Generated PDF with QR verification; stored in employee documents; downloadable on web/mobile. | V1 |
| LTR-004 | HR can issue letters proactively (single or bulk, e.g. increment letters). | V1 |
| LTR-005 | AI: draft custom letters from intent; tone/grammar check; translate to Urdu/Arabic for review. | V1 (translate V2) |

## 3. Expense claims

| ID | Requirement | Ver |
|---|---|---|
| EXP-001 | Expense categories with policy limits per grade/location (per item, per day, per month), receipt required thresholds, mileage rates, per-diem rules. | V1 |
| EXP-002 | Claim with multiple line items, receipts (camera upload), multi-currency with FX, project/cost-centre tagging. | V1 |
| EXP-003 | AI receipt scanning: amount, date, merchant, category, currency auto-filled; duplicate receipt detection; policy breach flags. | V1 |
| EXP-004 | Approval by manager → finance; partial approval per line; send back. | V1 |
| EXP-005 | Reimbursement via payroll (next run) or separate payout batch with bank file. | V1 |
| EXP-006 | Cash advances for travel and settlement against claims. | V2 |
| EXP-007 | Travel requests (trip, dates, estimated cost, bookings) before the claim. | V2 |

## 4. Loans & salary advances

| ID | Requirement | Ver |
|---|---|---|
| LN-001 | Loan types with eligibility (tenure, grade), max amount (e.g. N × basic), max installments, interest-free or markup, one active loan rule. | V1 |
| LN-002 | Request → approval → disbursement (via payroll or separate) → installment schedule → payroll deduction; skip/reschedule installments by HR. | V1 |
| LN-003 | Early settlement; recovery in final settlement; loan ledger and statement. | V1 |
| LN-004 | Salary advance (single deduction next month). | V1 |
| LN-005 | Loan against provident fund (PK) rules. | V2 |
