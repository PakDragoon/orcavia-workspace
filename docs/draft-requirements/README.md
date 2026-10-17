# Orcavia — Product Requirements

> Status: **DRAFT — input only.** This is the original broad draft (2026-10-17). It is **superseded
> area by area** by the locked specs in [`../specs/`](../specs/). When a spec exists for an area, the spec
> wins. Progress and decisions are tracked in [`../../PROJECT-CONTEXT.md`](../../PROJECT-CONTEXT.md).
> Items marked ⚠️ **ASSUMPTION** are raised during that area's grilling round.

Orcavia is an AI-first, multi-tenant HRMS (SaaS). It takes inspiration from NathanHR's HR Direct
(basic flows) and rebuilds them with a modern architecture, Pakistan-first localization, and AI
woven into every module rather than bolted on.

## How to read this

| File | What it covers |
|---|---|
| [00-product-vision.md](00-product-vision.md) | Vision, personas, positioning, principles, release roadmap |
| [01-platform-foundations.md](01-platform-foundations.md) | Tenancy, org structure, identity, permissions, audit, workflow engine, notifications, files, custom fields, billing — the things every module stands on |
| [02-core-hr.md](02-core-hr.md) | Employee records, lifecycle, org chart, documents, onboarding, offboarding, assets |
| [03-time-attendance.md](03-time-attendance.md) | Attendance, shifts, rosters, overtime, remote work, holidays, timesheets |
| [04-leave.md](04-leave.md) | Leave policies, accruals, balances, requests, encashment |
| [05-payroll.md](05-payroll.md) | Payroll engine + Pakistan and UAE country packs |
| [06-requests-letters-expenses.md](06-requests-letters-expenses.md) | Employee self-service requests, letters, expense claims, loans/advances |
| [07-performance-talent.md](07-performance-talent.md) | Goals, reviews, 360, competencies, 1:1s, learning, succession |
| [08-recruitment.md](08-recruitment.md) | ATS: requisitions, jobs, pipeline, interviews, offers |
| [09-engagement-helpdesk.md](09-engagement-helpdesk.md) | Announcements, surveys, recognition, events, HR helpdesk |
| [10-reports-analytics.md](10-reports-analytics.md) | Dashboards, standard reports, report builder, exports |
| [11-ai-features.md](11-ai-features.md) | Every AI feature, its guardrails, and the AI platform rules |
| [12-mobile-app.md](12-mobile-app.md) | Mobile (Expo) scope |
| [13-non-functional.md](13-non-functional.md) | Security, privacy/compliance, performance, availability, observability, accessibility |
| [14-integrations.md](14-integrations.md) | External systems and the public API |

## Conventions used in every file

- **Requirement IDs**: `<AREA>-<NNN>` (e.g. `LV-012`). IDs never get reused; deleted items are struck through.
- **Version tags**: **[V1]** first commercial release (MVP for Pakistan) · **[V2]** second release
  (talent + UAE) · **[V3]** later / enterprise. Tags describe *when*, not importance.
- **Actors**: Employee (EMP), Line Manager (MGR), HR Admin (HRA), HR Manager (HRM), Payroll Officer
  (PAY), Finance (FIN), Recruiter (REC), Company Owner (OWN), Platform Admin (PLT — Orcavia staff).
  These are *default role templates*; tenants can create their own roles (see `01` §Permissions).
- **"Configurable"** means configurable per tenant (and where stated, per legal entity / location / grade)
  without a code change.

## Repositories

| Repo | Role |
|---|---|
| `orcavia` | NestJS API + workers (Prisma, PostgreSQL, BullMQ/Redis) |
| `orcavia-ui` | React web app (admin + employee self-service) |
| `orcavia-app` | Expo / React Native mobile app |
| `orcavia-workspace` | Claude Code workspace config, requirements, ADRs, plans, reviews (this repo) |
