# 05 — Payroll

> Statutory rates, slabs, ceilings and minimum wages **change every budget year** and differ by
> province. None are hard-coded: they live as versioned, effective-dated country-pack data published
> from the platform console (PLT-002). Figures named here are examples to verify with a
> Pakistan payroll/tax consultant before V1 go-live.

## 1. Engine (country-agnostic)

| ID | Requirement | Ver |
|---|---|---|
| PAY-001 | **Pay groups** per legal entity: frequency (monthly [V1]; semi-monthly/weekly/daily-wage [V2]), pay period, cut-off dates, pay date, currency, country pack. | V1 |
| PAY-002 | **Pay components** (earnings, deductions, employer contributions, reimbursements, informational) with: taxable/exempt treatment, proration rule, rounding, formula, GL account, payslip visibility/order, statutory flags. | V1 |
| PAY-003 | **Formula language** (safe expression DSL, no code eval): references to basic, gross, other components, attendance/leave inputs, employee attributes, tables (slabs), YTD values. Formula editor with test-run against a sample employee. | V1 |
| PAY-004 | **Salary structures/templates** per grade (e.g. basic 60%, house rent 30%, utilities 10%); CTC ↔ gross ↔ net calculator, including reverse (net-to-gross) [V2]. | V1 |
| PAY-005 | **Inputs** collected per period: attendance/LOP, overtime, leave encashment, expense reimbursements, loan installments, one-time earnings/deductions (bonus, arrears, fines), recurring items with start/end, salary changes with mid-period effective dates (auto proration + arrears for back-dated changes). | V1 |
| PAY-006 | **Payroll run lifecycle**: Draft → Inputs locked → Calculated → Reviewed → Approved (workflow, maker-checker) → Finalized (immutable) → Paid → Posted. Re-open only by permission with audit; off-cycle runs (bonus, final settlement, corrections). | V1 |
| PAY-007 | **Calculation** runs in a background worker; 500 employees ≤ 2 min; per-employee recalculation; deterministic and idempotent; every line stores formula, inputs and version of country pack used. | V1 |
| PAY-008 | **Variance review**: compare with previous period per employee/component; AI-flagged anomalies with explanations (see `11` AI-PAY); reviewer must acknowledge flagged items before approval. | V1 |
| PAY-009 | **Payslips**: PDF from template, bilingual option, password-protected (e.g. CNIC digits ⚠️ confirm), published to ESS/mobile on finalization, email delivery optional. "Explain my payslip" in copilot. | V1 |
| PAY-010 | **Bank disbursement files** per bank format (configurable file builder: CSV/fixed-width/XLSX); PK banks first (HBL, Meezan, UBL, MCB, Bank Alfalah, … ⚠️ which), split salaries, cash/cheque payees list. | V1 |
| PAY-011 | **Accounting**: journal voucher per run by GL account × cost centre × entity; export CSV/XLSX [V1]; push to accounting systems (QuickBooks, Xero, Odoo, SAP B1, Dynamics) [V2/V3]. | V1 |
| PAY-012 | **Loans & advances** deduction schedules (see `06`), auto-recovery in final settlement. | V1 |
| PAY-013 | **Final settlement** run per leaver (OFF-004): pro-rata salary, leave encashment, gratuity/PF payout, notice adjustments, loan recovery, tax; settlement statement document. | V1 |
| PAY-014 | **Reports**: payroll register, component summary, department cost, bank summary, statutory reports, YTD, variance, headcount-cost trends. | V1 |
| PAY-015 | **Payroll calendar** and checklist with reminders to input providers (attendance lock, expense cut-off). | V1 |
| PAY-016 | **Retro/arrears** calculation across closed periods without re-opening them. | V1 |
| PAY-017 | **Employee self-service**: payslips, tax certificate, YTD, salary history, tax projections, investment/deduction declarations. | V1 |
| PAY-018 | **Compensation planning cycles** (merit increase budgets, manager proposals, approvals) | V3 |
| PAY-019 | **Benefits administration** (medical insurance plans, dependants, enrollment windows, claims limits) | V2 |

## 2. Pakistan country pack [V1]

| ID | Requirement |
|---|---|
| PK-001 | **Income tax on salary** (Income Tax Ordinance 2001): annual slab table per tax year (July–June), monthly withholding = projected annual tax ÷ remaining months, adjusting for actual YTD; handles mid-year joiners with previous-employer income/tax declarations, arrears, bonuses, and tax-exempt components as defined in the pack. |
| PK-002 | **Tax credits/adjustments** supported by declarations (e.g. charitable donations, approved pension/Voluntary Pension System contributions, Zakat deduction) — exact list maintained in pack data. |
| PK-003 | **EOBI**: employee and employer contributions as a % of the applicable minimum-wage base, eligibility rules, monthly contribution report. Rates and base are pack data. |
| PK-004 | **Provincial employees' social security** (PESSI Punjab, SESSI Sindh, KP-ESSI, BESSI Balochistan, ICT): employer contribution for employees under the wage ceiling; province derived from the employee's location. |
| PK-005 | **Provident fund** (recognised/approved funds): employee + employer contribution %, vesting rules, loans against PF, PF ledger and annual statement; exempt limit handling. |
| PK-006 | **Gratuity**: scheme per entity (none / gratuity scheme / in lieu of PF): typically last drawn basic (or gross, configurable) × years of service with configurable rounding and minimum service; provision accrual report for finance. |
| PK-007 | **Minimum wage** check per province and year → warning on employees below. |
| PK-008 | **Statutory outputs**: monthly withholding statement data (for FBR IRIS upload format ⚠️ verify), annual salary tax certificate per employee, EOBI and social-security challans/reports, PF statements. |
| PK-009 | **Zakat** deduction on applicable funds with exemption declaration (affidavit) tracking. ⚠️ verify relevance for salary. |
| PK-010 | **Bank files**: bulk salary formats for major PK banks; IBFT/RAAST inter-bank considerations. |
| PK-011 | Pre-configured default components: Basic, House Rent, Utilities, Medical (exempt rules per pack), Conveyance, Overtime, Bonus, LOP, Loan, Advance, EOBI (ee/er), PF (ee/er), Income Tax, Social Security (er). |

## 3. UAE country pack [V2]

| ID | Requirement |
|---|---|
| AE-001 | **WPS**: SIF file generation (employer MOL id, routing codes, EDR/SCR records), per agent bank/exchange format. |
| AE-002 | **End-of-service gratuity** per UAE Labour Law (basic-salary-based: days per year by service band, caps, unpaid-leave exclusions), DIFC DEWS / ADGM variants. |
| AE-003 | **GPSSA** pension contributions for UAE/GCC nationals. |
| AE-004 | **ILOE** (unemployment insurance) tracking. |
| AE-005 | Leave salary/air-ticket allowance accruals, annual ticket entitlement. |
| AE-006 | No income tax; multi-currency home-country remittance info optional. |

## 4. Future country packs [V3]

KSA (GOSI, Mudad WPS, EOSB), Qatar, Oman, Bahrain, UK/US via partners (⚠️ decide build vs partner).

## 5. Explainability (non-negotiable)

Each payslip line is clickable → shows formula, inputs, source records (attendance days, leave ledger
lines, overtime approvals, loan schedule) and pack version. The copilot uses the same structure to answer
"why is my salary lower this month?".
