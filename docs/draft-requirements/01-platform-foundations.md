# 01 — Platform Foundations

Everything in this file is shared by all modules. Modules must not re-implement any of it.

---

## 1. Tenancy & organization structure

### 1.1 Hierarchy (⚠️ ASSUMPTION — explore after this doc is reviewed)

```
Platform (Orcavia)
└── Tenant (a customer account; billing + data isolation boundary)
    └── Legal Entity (company registered for tax/statutory purposes; payroll runs per entity)
        └── Location / Branch (address, timezone, holiday calendar, geofence, provincial rules)
    └── Org Units (tree: Division → Department → Team; independent of legal entity)
    └── Cost Centres (finance dimension, many-to-many with org units)
    └── Grades / Bands, Job Titles, Job Families
```

| ID | Requirement | Ver |
|---|---|---|
| ORG-001 | A tenant has 1..n legal entities. Each entity has its own tax registration (NTN for PK, TRN/MOL establishment id for UAE), statutory registrations (EOBI, provincial social security numbers), bank accounts, currency, fiscal year and country pack. | V1 |
| ORG-002 | Locations belong to one legal entity and carry timezone, address, province/emirate, holiday calendar, work-week definition and optional geofence(s). | V1 |
| ORG-003 | Org units form an unlimited-depth tree, effective-dated (re-orgs keep history). Each unit has a head (employee). | V1 |
| ORG-004 | An employee's primary assignment = legal entity + location + org unit + job title + grade + manager + cost centre split, all effective-dated. | V1 |
| ORG-005 | Secondary / matrix managers (dotted line) and project-based reporting. | V2 |
| ORG-006 | A tenant may span countries (PK + UAE entities in one tenant), with payroll per entity. | V3 |
| ORG-007 | PEO/EOR mode: one provider tenant manages many client companies with a client portal and per-client billing. | V3 |
| ORG-008 | Org changes can be scheduled for a future effective date and previewed (who moves, whose approver changes). | V2 |

### 1.2 Tenant isolation

| ID | Requirement | Ver |
|---|---|---|
| TEN-001 | Every tenant-owned row carries `tenantId`. Isolation is enforced in PostgreSQL with Row-Level Security set per request/transaction, not by remembering filters. | V1 |
| TEN-002 | Automated cross-tenant test: every endpoint called as tenant B with tenant A's ids must return 404/403. Runs in CI. | V1 |
| TEN-003 | Tenant-scoped storage prefixes for files; signed URLs only; no public buckets. | V1 |
| TEN-004 | Tenant data export (full, machine-readable) and tenant deletion with a retention grace period. | V1 |
| TEN-005 | Data residency: tenant chooses region at signup (PK customers: ⚠️ decide region; UAE: `me-central-1`). | V2 |

---

## 2. Identity & authentication

| ID | Requirement | Ver |
|---|---|---|
| IAM-001 | One `User` (login identity, UUID) can link to one `Employee` per tenant, and may belong to several tenants (consultants, group HR). Tenant switcher in the UI. | V1 |
| IAM-002 | Login methods: email + password, phone + OTP (SMS/WhatsApp), magic link. Passwords hashed with Argon2id/bcrypt; breach-list check; strength meter. | V1 |
| IAM-003 | MFA: TOTP app, SMS/WhatsApp OTP fallback; tenant can enforce MFA for roles (e.g. Payroll, HR Admin). | V1 |
| IAM-004 | Access tokens short-lived (≤15 min), refresh tokens rotated with reuse detection; **signature always verified** server-side. Session list + "sign out other devices". | V1 |
| IAM-005 | Account lockout + rate limits on login/OTP; CAPTCHA only after repeated failures. | V1 |
| IAM-006 | SSO via OIDC/SAML (Microsoft Entra ID, Google Workspace, Okta). JIT provisioning. | V2 |
| IAM-007 | SCIM 2.0 user provisioning/deprovisioning. | V2 |
| IAM-008 | Employees without email (field staff) can log in with employee code + phone OTP. | V1 |
| IAM-009 | Biometric unlock on mobile (Face ID / fingerprint) after first login. | V1 |
| IAM-010 | Invitation flow: HR invites by email/phone; first-login sets password, accepts policies. | V1 |
| IAM-011 | "View as" (impersonation) for support and HR: server-side session claim carrying both the real actor and the viewed user; read-only by default; every action audit-logged with both ids; visible banner. Platform support impersonation requires tenant consent toggle. | V1 |

---

## 3. Permissions & access control

| ID | Requirement | Ver |
|---|---|---|
| ACL-001 | Roles are tenant data: `Role → Permission[]`. Permission keys are code constants (e.g. `leave.request.approve`, `payroll.run.finalize`, `employee.compensation.read`). | V1 |
| ACL-002 | Every grant has a **scope**: Self · Direct reports · All reports (recursive) · Org unit(s) · Location(s) · Legal entity(ies) · Whole tenant. Check = `can(permission, resource)` evaluated server-side. | V1 |
| ACL-003 | Field-level sensitivity classes: Public (name, title, photo), Internal (phone, location), Personal (DOB, address, family), Sensitive (CNIC/Emirates ID, passport, bank, salary, medical, disciplinary). Sensitive fields need explicit permission and are masked by default in lists and exports. | V1 |
| ACL-004 | Default role templates shipped: Employee, Manager, HR Admin, HR Manager, Payroll Officer, Finance Approver, Recruiter, IT/Asset Admin, Company Owner, Auditor (read-only). Tenants clone and edit them. | V1 |
| ACL-005 | Users can hold multiple roles; effective permission = union, scopes merged. | V1 |
| ACL-006 | Manager rights derive automatically from the reporting line — no manual role assignment needed for managers. | V1 |
| ACL-007 | Permission simulator: "what can user X see/do?" for HR admins. | V2 |
| ACL-008 | Time-boxed access grants (e.g. external auditor for 30 days). | V2 |
| ACL-009 | The UI hides what the user can't do, but **the API is the only enforcement point**. | V1 |

---

## 4. Audit, history & data lifecycle

| ID | Requirement | Ver |
|---|---|---|
| AUD-001 | Append-only `AuditEvent` written in the same transaction as each change: actor, acting-as, tenant, entity, action, before/after diff (sensitive values hashed or redacted), IP, user agent, request id, AI-suggestion id if any. | V1 |
| AUD-002 | Effective-dated tables for job/assignment, compensation, org membership, bank account, address, tax info. Corrections vs. future-dated changes are distinguished. | V1 |
| AUD-003 | Soft delete for people records; hard delete only by retention policy jobs. | V1 |
| AUD-004 | Audit viewer per record ("History" tab) and tenant-wide audit search with export. | V1 |
| AUD-005 | Retention policies per data category (e.g. ex-employee records N years, attendance raw punches N months, biometric templates deleted at exit). Configurable within legal minimums. | V2 |
| AUD-006 | Employee data subject requests: export my data, request correction. | V2 |

---

## 5. Dates, time zones, currency, localization

| ID | Requirement | Ver |
|---|---|---|
| LOC-001 | Calendar days (leave dates, attendance date, holidays, pay-period bounds, DOB, joining date) are stored as SQL `date`. Instants (punches, approvals) as `timestamptz`. Timezone resolved tenant → location → employee, server-side. | V1 |
| LOC-002 | CI runs time-sensitive tests under at least two TZs (`Asia/Karachi`, `America/Chicago`). | V1 |
| LOC-003 | Money stored as integer minor units + ISO currency; per-entity base currency; FX rates table for multi-currency expenses. | V1 |
| LOC-004 | UI languages: English [V1], Urdu [V2 ⚠️ confirm], Arabic with full RTL [V2]. All strings externalized from day one; RTL-safe layout (logical CSS properties) from day one. | V1 |
| LOC-005 | Locale-aware number/date formats; Hijri date display option (UAE) [V2]; Pakistani number grouping (lakh/crore) option. | V2 |
| LOC-006 | Configurable work week per location (Mon–Fri, Mon–Sat, Sun–Thu, half-day Saturday). | V1 |

---

## 6. Workflow & approval engine

The single engine behind every approval in Orcavia (leave, attendance correction, overtime, expense, loan,
letter, salary change, payroll finalization, onboarding tasks, offboarding clearance, requisitions, offers…).

| ID | Requirement | Ver |
|---|---|---|
| WF-001 | Workflow definitions are versioned tenant data: trigger (request kind + conditions) → ordered/parallel steps → outcome actions. Running instances keep the version they started with. | V1 |
| WF-002 | Approver resolvers: direct manager, Nth-level manager, org-unit head, role within scope (e.g. "HR Admin of employee's location"), specific person, group (any-one / all / N-of-M). | V1 |
| WF-003 | Conditions on any request field and employee attribute (amount > X, leave days > N, grade, location, entity). | V1 |
| WF-004 | Auto-skip rules: requester is the approver, approver = previous approver, approver missing → escalate to next resolver (never silently auto-approve unless configured). | V1 |
| WF-005 | Delegation: approver sets out-of-office delegate for a date range; auto-delegate when approver is on approved leave. | V1 |
| WF-006 | SLA per step with reminders and escalation; business-hours aware. | V1 |
| WF-007 | Actions: approve, reject (reason required), send back for changes, reassign, comment with @mentions, attach files. Bulk approve where safe. | V1 |
| WF-008 | Approve from email, push notification and mobile with one tap (signed, single-use action links). | V1 |
| WF-009 | Unified **Inbox**: all pending items for me across modules, with AI context card (see `11`). | V1 |
| WF-010 | Outcome actions: update record, generate document, send notification, create task, call webhook, start another workflow. | V1 |
| WF-011 | Visual workflow designer (drag-drop) with simulation ("who would approve this request for employee X?"). | V2 |
| WF-012 | Task checklists (onboarding/offboarding/probation) as workflow steps assignable to roles (IT, Admin, Finance). | V1 |
| WF-013 | AI "describe your workflow" → draft definition for human review. | V3 |

---

## 7. Forms, custom fields & request kinds

| ID | Requirement | Ver |
|---|---|---|
| FRM-001 | Custom fields on Employee, Job, Request kinds, Assets: types text, number, date, select, multi-select, boolean, file, person, currency; validation; sensitivity class; visibility per role. | V1 |
| FRM-002 | Custom request kinds (e.g. "Uniform request", "Parking pass") built from a form designer + a workflow, no code. | V2 |
| FRM-003 | Form schemas are JSON Schema/zod generated so web and mobile render the same form. | V1 |

---

## 8. Templates & document generation

| ID | Requirement | Ver |
|---|---|---|
| TPL-001 | One template system for documents (PDF/DOCX), email, SMS, WhatsApp, push and in-app messages, with merge fields from a typed catalog (employee, job, compensation, company, request). | V1 |
| TPL-002 | Rich-text/HTML designer with letterhead, signature blocks, QR verification code; versioning; per-entity/per-language variants. | V1 |
| TPL-003 | Generated documents stored against the employee with verification URL (anyone can verify a letter's authenticity via QR). | V1 |
| TPL-004 | E-signature (internal click-to-sign with audit trail) [V1]; external providers (DocuSign/Adobe Sign) [V3]. | V1 |
| TPL-005 | AI drafting of a template or a one-off letter from a prompt (see `11`). | V1 |

---

## 9. Notifications

| ID | Requirement | Ver |
|---|---|---|
| NTF-001 | Channels: in-app (bell + realtime), email, mobile push, SMS, WhatsApp Business. | V1 (WhatsApp V2 ⚠️) |
| NTF-002 | Every domain event (see `13` §Events) can trigger notifications through tenant-configurable rules; employees control their own channel preferences except mandatory notices. | V1 |
| NTF-003 | Digest mode (daily summary for managers); quiet hours. | V2 |
| NTF-004 | Delivery log with status per channel; retries; bounce handling. | V1 |
| NTF-005 | Scheduled reminders: document expiry, probation end, contract end, birthdays, work anniversaries, pending timesheets/approvals. | V1 |

---

## 10. Files & documents storage

| ID | Requirement | Ver |
|---|---|---|
| FIL-001 | S3-compatible storage, tenant-prefixed, encrypted at rest, presigned upload/download, size/type limits, malware scan before availability. | V1 |
| FIL-002 | Image processing (thumbnails, avatar crop), PDF preview in-browser. | V1 |

---

## 11. Import, export & bulk operations

| ID | Requirement | Ver |
|---|---|---|
| IMP-001 | Guided import (CSV/XLSX) for employees, assignments, compensation, leave balances, attendance, loans, opening payroll YTD balances: template download → column mapping → validation preview → commit → rollback window. | V1 |
| IMP-002 | Bulk operations go through the same use cases as single edits (same validation, audit, events). | V1 |
| IMP-003 | AI column-mapping and data clean-up suggestions on import. | V2 |
| IMP-004 | Every list view exports to XLSX/CSV, respecting permissions and sensitive-field masking. | V1 |

---

## 12. Search & navigation

| ID | Requirement | Ver |
|---|---|---|
| SRCH-001 | Global search (Ctrl/Cmd-K): people, pages, requests, documents, actions ("apply leave"), respecting permissions. | V1 |
| SRCH-002 | Saved filters/views on every list. | V1 |

---

## 13. Subscription, billing & feature gating

| ID | Requirement | Ver |
|---|---|---|
| BIL-001 | Plans priced per active employee per month with module add-ons (⚠️ pricing model to decide). | V1 |
| BIL-002 | Self-serve trial signup → setup wizard (entities, locations, policies from country templates, import) → go-live. | V1 |
| BIL-003 | Payment: local PK gateways/bank transfer + invoicing [V1 ⚠️ which gateway], Stripe for international [V2]. | V1 |
| BIL-004 | Module + feature flags per tenant (plan-driven and manual overrides by platform admin). | V1 |
| BIL-005 | AI usage metering per tenant/feature with plan quotas. | V1 |

---

## 14. Platform admin console (Orcavia staff)

| ID | Requirement | Ver |
|---|---|---|
| PLT-001 | Tenant list, provisioning, plan/flags, usage, health, suspension. | V1 |
| PLT-002 | Country pack management: tax years, rates, statutory tables published as versioned data. | V1 |
| PLT-003 | Support impersonation with tenant consent + full audit. | V1 |
| PLT-004 | Global announcements / maintenance banners. | V1 |
