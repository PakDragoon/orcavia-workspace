# 02 — Core HR

## 1. Employee record

Single source of truth for a person's employment. Tabs/sections, each with its own permission and sensitivity:

| Section | Contents | Sensitivity |
|---|---|---|
| Profile | Legal name (+ Urdu/Arabic name), preferred name, photo, pronouns (optional), employee code, work email/phone, status | Public/Internal |
| Personal | DOB, gender, marital status, nationality(ies), religion (optional, PK leave/holiday rules), blood group, personal email/phone, addresses (current/permanent) | Personal |
| Identity | CNIC (PK) with expiry, passport, Emirates ID / visa / labour card (UAE), NTN, driving licence; scans attached | Sensitive |
| Family & dependents | Spouse, children, parents, for benefits/insurance/medical | Personal |
| Emergency contacts | Name, relation, phone | Personal |
| Job (effective-dated) | Entity, location, org unit, title, grade, employment type (permanent, contract, probation, intern, consultant, daily-wage), manager(s), cost-centre split, work schedule, probation end, contract end, notice period | Internal |
| Compensation (effective-dated) | Pay group, basic + allowances, currency, pay frequency, change reason, approvals | Sensitive |
| Bank | Account(s) + IBAN, salary split, wallet (e.g. JazzCash/Easypaisa ⚠️ confirm need) | Sensitive |
| Tax & statutory | Tax status, previous-employer YTD income/tax (PK), EOBI number, social-security number, provident-fund membership, UAE GPSSA for nationals | Sensitive |
| Education, experience, skills, certifications, languages | With attachments + verification status | Personal |
| Documents | All employee files with type, expiry, versioning | varies |
| Assets | Issued company assets | Internal |
| Timeline | Every lifecycle event (from audit + effective-dated tables) | varies |
| Notes | HR-only private notes, disciplinary records | Sensitive |

| ID | Requirement | Ver |
|---|---|---|
| EMP-001 | Create employee manually, by import, from an accepted offer (ATS) [V2], or by self-onboarding invitation. | V1 |
| EMP-002 | Employee code auto-generation per entity with configurable pattern. | V1 |
| EMP-003 | Employees can edit allowed fields themselves; changes to sensitive fields (bank, name, marital status) go through an approval workflow with evidence upload. | V1 |
| EMP-004 | Profile completeness score + nudges. | V1 |
| EMP-005 | Duplicate detection on CNIC/passport/email at creation. | V1 |
| EMP-006 | Rehire: link new employment to previous record, carry history. | V1 |
| EMP-007 | Multiple concurrent employments (e.g. part-time in two entities) [V3]. | V3 |

## 2. Lifecycle events

All are effective-dated "actions" that go through the workflow engine and update the record on approval:

Hire · Probation confirmation / extension · Transfer (location/org unit/entity) · Promotion / demotion ·
Compensation change · Manager change · Employment-type change · Contract renewal · Suspension ·
Leave of absence (long-term) · Resignation · Termination · Retirement · Death in service · Rehire.

| ID | Requirement | Ver |
|---|---|---|
| LC-001 | Each action has a configurable form, workflow, generated letter and downstream effects (payroll proration, leave policy change, access changes). | V1 |
| LC-002 | Mass actions (annual increment, bulk transfer) with preview and approval. | V2 |
| LC-003 | Probation management: reminders to manager N days before end, evaluation form, confirm/extend/terminate. | V1 |
| LC-004 | Contract expiry tracking + renewal workflow. | V1 |

## 3. Org chart & directory

| ID | Requirement | Ver |
|---|---|---|
| ORGC-001 | Interactive org chart (zoom, search, expand/collapse, by legal entity or org unit, photo cards, headcount per node, vacancy placeholders [V2]). Export PNG/PDF. | V1 |
| ORGC-002 | People directory with filters (department, location, skills), privacy-aware contact info, "who's out today". | V1 |
| ORGC-003 | What-if org design (drag people, see cost impact) [V3]. | V3 |

## 4. Documents & compliance tracking

| ID | Requirement | Ver |
|---|---|---|
| DOC-001 | Document types configurable (mandatory per employment type/nationality, expiry required, who can view, retention). | V1 |
| DOC-002 | Expiry dashboard + reminders (employee, HR) at configurable offsets (e.g. 60/30/7 days). | V1 |
| DOC-003 | Missing-document checklist per employee. | V1 |
| DOC-004 | AI document intake: upload CNIC/passport/visa/degree → extract fields → pre-fill → human confirms (see `11` AI-DOC). | V1 |
| DOC-005 | UAE: visa, Emirates ID, labour card, medical insurance, permit lifecycle tracking with renewal workflows [V2]. | V2 |
| DOC-006 | Company documents & policies library with versioning and employee acknowledgement tracking ("read & accept"). | V1 |

## 5. Onboarding

| ID | Requirement | Ver |
|---|---|---|
| ONB-001 | Pre-boarding portal for new hire before Day 1: personal info, documents, bank, tax declaration, policy acknowledgements, e-sign offer/contract. | V1 |
| ONB-002 | Onboarding templates per entity/role: task checklist across HR, IT, Admin, Manager, Buddy with due dates relative to joining date. | V1 |
| ONB-003 | Welcome experience: team intro, first-week schedule, buddy assignment. | V2 |
| ONB-004 | AI onboarding plan: generates a 30/60/90-day plan from job description for manager to edit. | V2 |

## 6. Offboarding

| ID | Requirement | Ver |
|---|---|---|
| OFF-001 | Resignation submission by employee (with notice period calculation) or termination by HR/manager; approval workflow. | V1 |
| OFF-002 | Clearance checklist across departments (IT, Finance, Admin, Assets, Manager) with sign-off. | V1 |
| OFF-003 | Exit interview form; AI theme summary across exits [V2]. | V1 |
| OFF-004 | Final settlement calculation hand-off to payroll: last salary proration, leave encashment, gratuity/PF, loan recovery, notice-period pay/deduction. Explainable statement. | V1 |
| OFF-005 | Experience / relieving letter generation; access revocation on last working day. | V1 |
| OFF-006 | Alumni record and rehire eligibility flag. | V2 |

## 7. Assets

| ID | Requirement | Ver |
|---|---|---|
| AST-001 | Asset catalog (type, serial, value, purchase date, warranty, location), assign/return to employee with acknowledgement, condition notes. | V1 |
| AST-002 | Asset requests through workflow; assets in offboarding clearance. | V1 |
| AST-003 | Depreciation, maintenance schedules, QR tags [V3]. | V3 |

## 8. Disciplinary & grievances (⚠️ confirm scope)

| ID | Requirement | Ver |
|---|---|---|
| DSC-001 | Warnings, show-cause notices, investigation notes, outcomes; strictly permissioned; letters from templates. | V2 |
| DSC-002 | Confidential grievance/whistle-blower channel with optional anonymity. | V2 |
