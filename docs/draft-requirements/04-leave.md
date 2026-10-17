# 04 — Leave Management

## 1. Leave types & policies

Leave **types** (Annual, Casual, Sick, Maternity, Paternity, Hajj/Umrah, Iddat, Bereavement, Marriage,
Unpaid, Compensatory off, Study, custom…) are configured as data. A **policy** binds types to employee
eligibility rules (entity, location, grade, employment type, gender, religion, tenure, nationality).
Pakistan and UAE country packs ship default templates; tenants adjust.

| ID | Requirement | Ver |
|---|---|---|
| LV-001 | Per leave type: paid/unpaid, unit (day / half-day / hours), entitlement, accrual method (upfront yearly, monthly, per-days-worked, none), leave year (calendar, fiscal, joining anniversary), proration for joiners/leavers, probation restrictions, max consecutive days, min notice, documents required after N days (e.g. medical certificate). | V1 |
| LV-002 | Weekends and holidays inside a leave span: count or exclude per type ("sandwich rule" configurable). | V1 |
| LV-003 | Carry-forward rules: cap, expiry date of carried balance, encashment of excess. | V1 |
| LV-004 | Negative balance allowance per type with cap; auto-convert excess to unpaid. | V1 |
| LV-005 | Once-in-service leaves (e.g. Hajj) and lifetime caps. | V1 |
| LV-006 | Compensatory-off credit from approved overtime/holiday work with expiry. | V1 |
| LV-007 | Balance is a ledger (grant, accrual, consumption, adjustment, encashment, expiry, carry-forward) — never a mutable number. Every balance is explainable line by line. | V1 |
| LV-008 | Manual adjustments by HR with reason (audited). Opening balances via import. | V1 |

## 2. Requests

| ID | Requirement | Ver |
|---|---|---|
| LV-010 | Apply for leave (web/mobile/copilot): type, dates, half-day/hours, reason, attachment, handover person; live balance-after preview and team-conflict warning. | V1 |
| LV-011 | Approval through workflow engine; manager sees team calendar + overlap + remaining balance in the approval card. | V1 |
| LV-012 | Cancel (before start) / withdraw (approval to cancel after start) / modify (shorten/extend) with re-approval. | V1 |
| LV-013 | Leave on behalf: HR/manager applies for an employee (audited). | V1 |
| LV-014 | Blackout periods (e.g. month-end close for finance team) block or require extra approval. | V2 |
| LV-015 | Minimum staffing rules per team ("at most 2 people off at once") — warn or block. | V2 |
| LV-016 | Leave and attendance integrate: approved leave sets attendance day status; changes recompute. | V1 |
| LV-017 | Leave feeds payroll: unpaid leave → LOP deduction; encashment → earning; final settlement uses balance. | V1 |

## 3. Views

- Employee: balances with ledger, upcoming leave, request history, team calendar (privacy-aware: shows "Out", not the type, unless permitted).
- Manager: team calendar, pending approvals, leave liability summary.
- HR: company calendar, balance reports, liability (days × daily rate) for finance, year-end processing (carry-forward/expiry run with preview).

## 4. AI

- Copilot: "How many annual leaves do I have?", "Apply 2 days casual leave next Thursday" → draft request for confirmation.
- Approval context card: overlaps, team coverage, unusual patterns (e.g. repeated Monday sick leaves) shown as neutral facts, not accusations.
- Policy Q&A grounded in the tenant's leave policy text with citations.
