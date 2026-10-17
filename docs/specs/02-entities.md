# Spec 02 — Entities, Structure, Roles & Permissions

> Status: **LOCKED** 2026-10-18 (grilling rounds 1–3, all recommendations accepted; Q3 revised by user).
> Every feature spec builds on this file. Changing anything here requires a new decision entry.

---

## 1. Tenant

| # | Decision |
|---|---|
| E-1.1 | A **tenant = one customer account**: one subscription, one bill, one fully isolated data space. |
| E-1.2 | A business group with several companies is **one tenant with many companies**. Two unrelated businesses are always two tenants; nothing is shared between tenants. |
| E-1.3 | Tenant settings: name, logo, brand colour, default country, default language, default timezone, plan, owner(s). |
| E-1.4 | Each tenant gets its **own subdomain** (`acme.orcavia.app`), chosen at signup, changeable by an Owner. Custom domains (`hr.acme.com`) in V3. |
| E-1.5 | The person who signs up becomes **Owner** (full rights + billing). Owners can add other admins; ownership can be transferred; **there is always at least one Owner**. An Owner does not need an employee record. |

## 2. Structure levels

```
Tenant (customer account)
├── Company (legal entity; payroll runs per company)
│   └── Department tree (per company)
├── Location / Branch (linked to one or more companies of the tenant)
└── Master lists: Designations · Grades · Worker types · Cost centres
```

| # | Decision |
|---|---|
| E-2.1 | UI word is **"Company"**; internally the legal entity. |
| E-2.2 | **Departments belong to one company.** Each company has its own department tree (Textiles Finance ≠ Logistics Finance). *(Revised by user 2026-10-18.)* |
| E-2.3 | **Visibility is controlled by role scope, never by structure.** A company's HR sees only that company's employees unless given group-wide scope. |
| E-2.4 | **Shared services** (e.g. one group HR team): those people are employed by one company; their role is scoped to several companies. |
| E-2.5 | **Group reporting**: tenant-wide roles can view combined data grouped by department *name* across companies, without merging departments. |
| E-2.6 | A **location can be linked to one or more companies** in the tenant (shared office); holidays, geofence, timezone defined once. Each employee is still in exactly one company and one of that company's departments. |
| E-2.7 | Moving an employee between companies requires choosing a department in the new company's tree. |
| E-2.8 | People-based structure in V1 (person reports to person). **Positions / vacancy / headcount planning in V3.** |

## 3. Company

| Level | Fields |
|---|---|
| **Required to create** | Legal name, display name, country, base currency (default PKR), fiscal year start (default July), registered address, employee-code pattern |
| **Required before first payroll** | NTN; EOBI registration no. (if EOBI-registered); provincial social-security registration no. for **each province where it has staff**; salary disbursement bank account(s); authorised signatories |
| **Optional** | SECP incorporation no., STRN, letterhead, stamp image, website, logo |

| # | Decision |
|---|---|
| E-3.1 | Two-level requirement (create vs. payroll-ready) with a **setup checklist** showing what is missing before payroll. |
| E-3.2 | A company can be **archived only when it has no active employees**. |
| E-3.3 | **Merger** = bulk transfer of all employees into the absorbing company (rules in §8), with preview first. |

## 4. Location

| # | Decision |
|---|---|
| E-4.1 | Fields: name, code, type (**Head office, Branch, Factory, Warehouse, Site, Remote**), address, city, province, country, timezone, work week, holiday calendar, geofence(s), linked companies, contact person. |
| E-4.2 | **Remote** location type for fully remote staff: no geofence; province for social security taken from the **employee's home address**. |
| E-4.3 | Provincial rules (e.g. PESSI/SESSI) come from the location's province (or home address for Remote). |

## 5. Department

| # | Decision |
|---|---|
| E-5.1 | Fields: name, code, parent department, department head, optional default cost centre, status. Unlimited depth. |
| E-5.2 | **Department head can be any active employee in the tenant**, including from another group company (e.g. group CFO heads Textiles Finance); the UI shows a note when the head is from another company. |
| E-5.3 | Department head gets **view access** to everyone in their department tree (profile basics, attendance, leave, KPIs) — **not salary or sensitive fields**. Approval rights only where a workflow names "department head" as a step. |
| E-5.4 | If a department has no head, approvals route to the next resolver and the gap is flagged. |

## 6. Designations, grades, worker types, cost centres

| # | Decision |
|---|---|
| E-6.1 | **Designation**: title, job family (Sales, Engineering…), job description (also used by AI for KPI suggestions), default grade. Tenant-wide, optionally limited to certain companies. |
| E-6.2 | **Grade**: name + level number (ordering and rules like "grade 5+ gets fuel allowance"). **Tenant-wide.** |
| E-6.3 | **Salary bands** (min / mid / max) are set **per company per grade**. |
| E-6.4 | **Worker types** (all seven): Permanent · Fixed-term contract · Part-time · Intern/Trainee · Daily-wage · Consultant/Freelancer · Outsourced/third-party. Each company can rename or disable types. |
| E-6.5 | **Probation is a status, not a worker type.** |
| E-6.6 | Consultants and outsourced staff use **limited modules** (attendance, leave, assets; no statutory payroll). |
| E-6.7 | **Cost centres in V1**, optional per company; independent of departments; used to split payroll cost (e.g. 60% Project A / 40% Project B). Default: one cost centre per department. |

## 7. User vs. Employee

| # | Decision |
|---|---|
| E-7.1 | **User** = login identity (email/phone, password, MFA). **Employee** = employment record (job, salary, leave…). Separate objects. |
| E-7.2 | One user can have an employee record in **several tenants** and switches between them. |
| E-7.3 | A user can exist **without** an employee record (external payroll consultant, auditor, non-payroll owner) with limited roles. |
| E-7.4 | **One active employment per person per tenant in V1.** Shared staff handled by cost split. Concurrent employments in V3. |

## 8. Employee

### 8.1 Required data — three levels

| Level | Fields |
|---|---|
| **To create** | First name, last name, company, location, department, designation, worker type, joining date, manager (except top person), work email **or** mobile number |
| **Payroll-ready** | CNIC (Pakistani nationals) or passport (foreign nationals), date of birth, gender, salary, bank account **or** cash/cheque flag, tax status |
| **Complete profile** | Everything else (family, education, emergency contacts, documents…) with completeness score + reminders to the employee |

Payroll **refuses** employees who aren't payroll-ready and lists exactly what is missing.

### 8.2 Names & identity

| # | Decision |
|---|---|
| E-8.1 | Store first, middle, last name; **full name exactly as on CNIC**; **father's/husband's name with relation** (S/O, D/O, W/O); optional **Urdu name** (used in Urdu letters, V2). |
| E-8.2 | CNIC: 13 digits validated as `XXXXX-XXXXXXX-X`, with expiry. **NICOP** supported for overseas Pakistanis. **Passport** mandatory for foreign nationals, with nationality. |
| E-8.3 | CNIC and passport numbers **unique among active employees in the tenant**. Rehire matches the old record by CNIC. |
| E-8.4 | Identity numbers **stored encrypted** and **shown masked** (`35202-*******-1`) without permission. |

### 8.3 Employee code

| # | Decision |
|---|---|
| E-8.5 | Unique **within the tenant**; pattern configurable **per company** (e.g. `ATX-0001`, `ALG-0001`). |
| E-8.6 | Codes are **never reused**, even after separation. |
| E-8.7 | On transfer between companies: **keep code by default**; setting allows issuing a new code. |

### 8.4 Statuses

Pre-boarding · Active – On probation · Active – Confirmed · On notice · On long leave / sabbatical · Suspended ·
Separated – Resigned / Terminated / Retired / Deceased / Contract ended / **Absconded**.

| # | Decision |
|---|---|
| E-8.8 | Separated employees keep full history; **rehire links to the old record**. |

### 8.5 History (effective-dated)

| # | Decision |
|---|---|
| E-8.9 | Effective-dated with "effective from" (back-dated and future-dated changes allowed **with approval**): **Job** (company, location, department, designation, grade, manager, worker type, cost centre) · **Salary** · **Bank account** · **Status**. |
| E-8.10 | All other fields (phone, address…) keep an **audit trail**. |

### 8.6 Login identifiers

| # | Decision |
|---|---|
| E-8.11 | Work email **unique per tenant**. |
| E-8.12 | Mobile number used for OTP login **unique among active employees in the tenant**. |
| E-8.13 | Employees with **no login** (e.g. daily-wage staff managed by a supervisor) may share a family number or have none. |
| E-8.14 | Employees without email log in with **employee code + phone OTP**. |

### 8.7 Profile privacy defaults

| Visible to | Fields |
|---|---|
| All colleagues | Photo, name, designation, department, location, work email, work phone, birthday as **day + month only** (employee can opt out) |
| Never to colleagues | Personal phone, address, family, CNIC, salary |
| Managers (team) | Personal contact details **only if the company enables it** |

## 9. Manager

| # | Decision |
|---|---|
| E-9.1 | A manager is **not an account type**; it is any employee someone reports to. |
| E-9.2 | **One direct manager per employee** (person-to-person). The top person (e.g. CEO) has none. |
| E-9.3 | Manager rights (approvals, team views) come **automatically** from the reporting line. |
| E-9.4 | Department head is a separate setting on the department (§5). |
| E-9.5 | Dotted-line / project managers in **V2**. |

**Manager edge cases (all locked):**

1. Circular reporting (A → B → A) is blocked.
2. Manager must be an active employee.
3. Manager leaves → their reports auto-move to **the manager's manager**; HR gets a task to confirm or pick new managers.
4. Manager on leave → approvals delegated via the approvals engine.
5. Manager may be in a different group company.
6. An employee cannot be their own manager.
7. Top person (CEO) leaves → HR must assign a replacement before the exit is finalised.

## 10. Transfer between group companies

| # | Decision |
|---|---|
| E-10.1 | Effective-dated transfer: **old company pays up to the day before; new company from the effective date.** |
| E-10.2 | **Service continuity** is a per-transfer setting, **default: continuous** (drives gratuity, leave accrual, probation). |
| E-10.3 | **Leave balance**: carried over (**default**) or settled in the old company. |
| E-10.4 | **Tax**: income and tax at the old company carry into the new company's calculation as previous-employer income for the same tax year. |
| E-10.5 | **Loans** move with the employee unless HR settles them. |
| E-10.6 | EOBI and social security switch to the new company's registrations. |
| E-10.7 | Employee code kept by default (E-8.7); a department in the new company must be chosen (E-2.7). |

## 11. Archive vs. delete

| # | Decision |
|---|---|
| E-11.1 | Companies, locations, departments, designations, grades **in use cannot be deleted — only archived**. Archived items leave dropdowns but remain on historical records. |
| E-11.2 | Archiving a department that still has people requires moving them first. |
| E-11.3 | Deleting is allowed only for **unused** items. |

## 12. Custom fields (V1)

Companies can add fields to **Employee, Company, Location, Department, Designation**, each with type,
validation, sensitivity level and visibility by role.

## 13. Initial setup & import

| # | Decision |
|---|---|
| E-13.1 | Setup wizard order: **Companies → Locations → Departments → Designations & grades → Employees** (spreadsheet import). |
| E-13.2 | Manager links in the import refer to **employee codes**, so managers and reports can appear in any order in the same file. |
| E-13.3 | Validation previews **every error before anything is saved**. |

---

## 14. Roles & permissions

### 14.1 Default roles (copy / rename / edit per tenant)

| Role | Meant for | Default scope |
|---|---|---|
| **Owner** | Signup person, directors | Whole tenant, everything incl. billing |
| **HR Admin** | Day-to-day HR operations | Assigned company(ies) |
| **HR Manager** | Policies, approvals, reports | Assigned company(ies) |
| **Payroll Officer** | Runs payroll | Assigned company(ies) |
| **Finance Approver** | Approves payroll, expenses, loans | Assigned company(ies) |
| **IT / Asset Admin** | Equipment, onboarding & exit IT tasks | Assigned location(s) or company |
| **Auditor** | Read-only review | Assigned company(ies), time-boxed |
| **Manager** *(automatic)* | Anyone with reports | Their reports |
| **Employee** *(automatic)* | Every active employee | Self |
| **Recruiter** *(V2)* | Hiring | Assigned company(ies) |

### 14.2 Access by status

| Status | Access |
|---|---|
| Pre-boarding | Onboarding portal only |
| Active | Employee role (+ any assigned roles) |
| Separated (normal) | Read-only own payslips, tax certificates, letters for **12 months** (configurable) |
| Absconded / terminated for cause | **Access removed immediately**; HR can send documents manually |

### 14.3 Scopes

Self · Direct reports · All reports (whole chain) · Department(s) incl. sub-departments · Location(s) ·
Company(ies) · Whole tenant. A scope can hold **multiple values** (e.g. Lahore + Karachi).

### 14.4 Permission granularity

- **Module × action**: view, create, edit, archive, approve, export, configure.
- **Sensitive field groups** with separate view/edit permissions: **Compensation, Bank, Identity, Medical, Disciplinary, Family**.
- Multiple roles combine (union of permissions, scopes merged).
- The UI hides what the user can't do; **the API is the only enforcement point**.

### 14.5 Guardrails

1. Only Owners or users with "Manage roles" can edit roles.
2. **Nobody can grant a permission or scope they don't have themselves.**
3. Nobody can change their own roles.
4. The last Owner cannot be removed.
5. Every role change is audited and the affected user is notified.

### 14.6 Separation of duties

| # | Decision |
|---|---|
| E-14.1 | Nobody approves their own requests (leave, expenses, salary change) — including HR and Owners; theirs go to their own manager / next approver. |
| E-14.2 | Payroll **maker-checker**: calculator ≠ approver. |
| E-14.3 | Salary change: requester ≠ approver. |
| E-14.4 | Only exception: a single-person HR team may allow an Owner to be maker and checker, **with a warning**. |

### 14.7 Specific rules

| # | Decision |
|---|---|
| E-14.5 | **Managers seeing team salary**: off by default; company can enable it (direct reports only, never the whole chain). |
| E-14.6 | **HR on their own record** sees it as an employee; own sensitive changes go through the same workflow as everyone. |
| E-14.7 | **"View as"**: Owners and users with the permission; **read-only**; cannot view as someone with more access; banner always visible; every session/page audited; viewed person sees "viewed by X on date" in their account activity; Orcavia support only with **time-limited Owner grant**. |
| E-14.8 | **Exports**: separate permission; sensitive fields need the field permission too; every export logged (who/what/when); files carry exporter name + time in the footer. |
| E-14.9 | **Role assignments** have a start date and optional end date and **expire automatically** (V1). Permission simulator in V2. |

---

## 15. Feature order (locked)

1. Employee profile & lifecycle (Core HR) → 2. Onboarding → 3. Attendance & shifts → 4. Leave →
5. WFH / remote work → 6. Requests & letters → 7. Assets / IT equipment → 8. Expenses & loans →
9. KPI evaluation → 10. Offboarding & final settlement → 11. Payroll →
12. Announcements & policies, HR helpdesk, reports & dashboards, AI copilot (cross-checked against all).
