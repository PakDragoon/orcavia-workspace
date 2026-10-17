# 00 — Product Vision

## 1. One-liner

**Orcavia is the HR system that does the work, not just stores it**: a multi-tenant HRMS where
employees ask instead of searching, managers approve with context instead of guessing, and HR runs
payroll, compliance and people processes with an AI co-worker that drafts, checks and explains —
always with a human making the final call.

## 2. Problem

Mid-market companies in Pakistan (and later the UAE/GCC) run HR on a mix of spreadsheets, legacy
desktop payroll, WhatsApp approvals and a generic HRMS that:

- needs an HR person to answer every "how many leaves do I have / where is my salary certificate";
- calculates payroll but cannot explain *why* a number changed;
- has rigid approval chains that break when someone is on leave;
- is built for one country, so tax, social security and bank files are manual;
- has a poor mobile experience for the 60–80% of staff who are not at a desk.

## 3. Target customers (⚠️ ASSUMPTION — confirm in grilling)

| Segment | Size | Notes |
|---|---|---|
| Primary (V1) | 50–2,000 employees, Pakistan | Multi-branch, often multi-entity groups; mix of desk and field staff |
| Secondary (V2) | 50–5,000 employees, UAE | Multi-national workforce, WPS, visas |
| Later (V3) | Groups operating across PK + GCC, PEO/EOR providers, outsourcing firms | Multi-country payroll, client portals |

## 4. Personas

| Persona | Goals | Top jobs in Orcavia |
|---|---|---|
| **Employee** (desk + field) | Get answers and paperwork fast, get paid correctly | Check balance, apply leave, clock in, download payslip, request letter, ask the copilot |
| **Line Manager** | Keep team running, not drown in approvals | Approve with context, see who is out, run 1:1s and reviews |
| **HR Admin** | Keep records clean and compliant | Onboard, update records, manage documents and expiries, answer queries |
| **HR Manager / HRBP** | Policy, culture, talent | Configure policies, run reviews and surveys, read analytics |
| **Payroll Officer** | Correct, on-time pay | Collect inputs, run payroll, reconcile, file tax, produce bank files |
| **Finance** | Cost control, accounting | Approve payroll, post journals, see cost by cost-centre |
| **Recruiter** [V2] | Fill roles fast | Post jobs, screen, schedule, offer |
| **Company Owner / CXO** | Visibility | Headcount, cost, attrition, risk dashboards |
| **Platform Admin** (Orcavia staff) | Operate the SaaS | Provision tenants, support, billing, feature flags |

## 5. Product principles

1. **Self-service first.** Every employee question HR answers twice becomes a self-service flow or a copilot skill.
2. **AI proposes, humans decide.** AI drafts, summarizes, flags and explains; any change to pay, employment
   status, or a performance/hiring outcome needs an explicit human action. Every AI suggestion is logged.
3. **Explainable numbers.** Every computed value (salary line, tax, leave balance, overtime) can show its formula and inputs.
4. **Configuration over customization.** Policies, workflows, forms, letters, roles and fields are tenant data, not code.
5. **Country packs, not forks.** Statutory logic lives in versioned country packs (PK first, then UAE).
6. **History is never lost.** Effective-dated records + an append-only audit log.
7. **Mobile is a first-class citizen**, not a shrunken web app.
8. **Secure by default.** Tenant isolation in the database, least-privilege permissions, sensitive fields encrypted.

## 6. What we take from HR Direct, and what we change

| HR Direct had | Orcavia does |
|---|---|
| Boolean role flags (`isAdmin`, `isHR`…) checked against URL prefixes | Roles are tenant data → permissions + scope, enforced on the server |
| One `requests` table mixing leave/letter/salary fields | One request *framework* + typed request kinds with their own schemas |
| Process-flow builder (form → document → email → payment) | Generalized **workflow engine** with versioned definitions, SLA, delegation, escalation |
| Module-specific approval logic in private packages | One approval engine used by all modules |
| Manual `company_ID` filter on each query | Row-level security keyed on tenant |
| UAE-only payroll (WPS, gratuity, visa) | Country-pack payroll engine; Pakistan first, UAE second |
| Letter / email / SMS template designers | One template system (documents, email, SMS, WhatsApp, push) with merge fields and versioning |
| Face recognition attendance | Kept as optional [V2], with explicit consent and data-retention rules |
| Surveys, news, social, rewards, forums | Consolidated "Engagement" module |
| No AI | AI copilot + AI in every module (see `11`) |
| No recruitment, no learning | ATS [V2], Learning [V3] |
| Desktop-first UI | Responsive web + native mobile app |

**Dropped (not planned):** visa outsourcing / PRO services as a module for third parties, client
invoicing, "nathan10" gallery, Outlook calendar embed. (Visa/permit *tracking* for UAE employees
is kept inside Core HR documents [V2].) ⚠️ ASSUMPTION — confirm.

## 7. Release roadmap (⚠️ draft — to be locked in grilling)

| Release | Theme | Modules |
|---|---|---|
| **V1 — "Run HR in Pakistan"** | All HRMS basics + KPI evaluation | Platform foundations (company structure, roles & permissions, workflows, templates, notifications, imports), Core HR (employee database, documents & expiries, org chart, directory, lifecycle actions, assets), Onboarding & Offboarding with final settlement, Attendance & Shifts, Holidays, Leave, Requests & Letters, Expense claims, Loans/advances, **Payroll (Pakistan pack)**, **KPI & evaluation system (fully configurable)**, Announcements & policies, HR helpdesk, Standard reports + dashboards, Mobile ESS, **AI copilot + document AI + payroll anomaly checks + KPI assistant** |
| **V2 — "Grow talent, enter UAE"** | Talent + second country | Advanced performance (OKRs, 360, calibration, 1:1s, continuous feedback), Recruitment (ATS), Surveys & recognition, Report builder, **UAE pack** (WPS, gratuity, visas/permits, Arabic/RTL), SSO/SCIM, face-recognition attendance, advanced AI (NL reports, review assistant, CV parsing behind a feature flag) |
| **V3 — "Enterprise & scale"** | Bigger groups, more countries | Learning (LMS), Succession & workforce planning, compensation planning cycles, multi-country groups in one tenant, PEO/EOR mode + client portal, public API & marketplace, KSA pack, data warehouse export, advanced AI agents (workflow builder by description, attrition insights) |

## 8. Success metrics (⚠️ to confirm)

- Time to onboard a new tenant to first payroll: **≤ 10 working days** (V1), ≤ 3 days with import + AI mapping (V2).
- ≥ **60%** of employee HR questions answered by self-service/copilot without HR involvement.
- Payroll run (500 employees) computed in **≤ 2 minutes**; zero unexplained variances at approval.
- Approval median turnaround **≤ 24h**.
- Mobile weekly active rate **≥ 50%** of employees.
