# 07 — Performance & Talent

> Regulatory note: AI that influences performance ratings, promotion or termination is high-risk under
> the EU AI Act and similar rules. In Orcavia AI may **summarize and assist writing**, never produce a
> rating or decision. See `11` §Guardrails.

## 0. KPI & Evaluation System [V1] — fully configurable per tenant

Decided 2026-10-18: companies must be able to evaluate employees properly with whatever settings they want,
so a configurable KPI system ships in **V1**. 360, calibration, OKRs and 1:1s stay in V2.

### 0.1 Building blocks

| Block | What a company configures |
|---|---|
| **KPI library** | Name, description, category/perspective (e.g. Financial, Customer, Process, People — Balanced Scorecard, or own categories), unit (number, %, currency, days, count, yes/no, milestone, rating-only), direction (higher is better / lower is better / target is best), frequency of measurement (monthly, quarterly, per cycle), evidence required (yes/no), data source (see 0.3), owner. Tag by job family/department. |
| **Targets** | Fixed target · range (min/target/stretch) · threshold bands (e.g. <80% → 1, 80–95% → 3, 95–105% → 4, >105% → 5) · per-period targets (monthly phasing) · team/shared targets. |
| **Scorecard templates** | Assigned by job title, grade, department, location or individually. Sections with weights (e.g. KPIs 70%, Competencies 20%, Values/behaviours 10%; any number of sections; weights must total 100%). KPI weights inside a section (must total 100%), min/max KPIs per person, mandatory KPIs that cannot be removed, KPIs employees may add themselves (with approval). |
| **Rating scales** | Numeric (1–3, 1–4, 1–5, 1–10), percentage, or labelled; descriptors per level; half-points allowed or not; N/A allowed or not. Multiple scales per tenant. |
| **Scoring rules** | Achievement % = actual ÷ target (higher-better) or target ÷ actual (lower-better); cap (e.g. 120%) and floor (0%); banded mapping to scale; custom formula (safe DSL, same as payroll); rounding. Overall score = Σ(section weight × section score). |
| **Final rating bands** | Score → label (Outstanding / Exceeds / Meets / Needs Improvement / Unsatisfactory or own labels), boundaries configurable; optional forced/recommended distribution with warnings (V1) or enforcement (V2). |
| **Raters & weights** | Self, manager, skip-level/reviewer, HR; per-rater weights (e.g. manager 80%, self 20%) or "manager decides, self is advisory". |
| **Cycle types** | Annual, semi-annual, quarterly, monthly, probation, project-end, ad-hoc. Multiple concurrent cycles allowed (e.g. quarterly sales KPIs + annual review). |
| **Visibility** | Whether employee sees manager scores before sign-off; whether employees see each other's KPIs (team transparency); who sees final ratings. |

### 0.2 Cycle flow (each phase can be switched on/off and given deadlines)

1. **Setup** (HR): choose template(s), participants (filters + exclusions such as joined after cut-off date), timeline, rating scale, raters.
2. **Goal setting**: KPIs pushed from templates/cascade → manager adjusts targets/weights within allowed limits → employee reviews and acknowledges (or discusses) → manager approves; optional HR approval.
3. **Tracking**: periodic actuals entry (manual, imported, or auto from system metrics), check-in comments, evidence uploads; progress dashboard with RAG status.
4. **Self-assessment**: employee scores/comments per KPI (if enabled).
5. **Manager assessment**: scores per KPI with mandatory comment rules (e.g. comment required for scores ≤2 or ≥5).
6. **Reviewer (skip-level)**: approve or send back.
7. **HR review / moderation**: distribution check, outlier review, adjustments with justification (V1); full calibration sessions (V2).
8. **Sign-off**: employee acknowledges, may add comments or raise an appeal → appeal workflow.
9. **Outcomes**: final rating stored on the employee timeline; feeds bonus calculation (payroll input), increment recommendation matrix, PIP trigger, promotion eligibility.

### 0.3 KPI data sources

| Source | Example | Ver |
|---|---|---|
| Manual entry by employee/manager | Sales closed, projects delivered | V1 |
| Bulk import (CSV/XLSX per period) | Monthly sales figures from CRM | V1 |
| System metric from Orcavia | Attendance %, punctuality %, overtime hours, training completed, helpdesk SLA met | V1 |
| Formula over other KPIs | Profit margin = profit ÷ revenue | V1 |
| API/integration push | CRM/ERP integration via public API | V2 |

### 0.4 Requirements

| ID | Requirement | Ver |
|---|---|---|
| KPI-001 | KPI library CRUD with versioning; KPIs used in an active cycle are frozen (edits create a new version). | V1 |
| KPI-002 | Scorecard template designer with sections, weights, validation (weights = 100%), preview against a sample employee. | V1 |
| KPI-003 | Cascading: company → department → team → individual KPIs; child progress can roll up to the parent. Shared KPIs (one actual for a whole team). | V1 |
| KPI-004 | Cycle engine with configurable phases, deadlines, reminders, escalations, bulk actions for HR (reopen, extend, reassign reviewer). | V1 |
| KPI-005 | Scoring engine supports all scoring rules in 0.1, with an explainable breakdown per KPI and overall. | V1 |
| KPI-006 | Mid-cycle events: new joiner (prorated cycle or excluded by cut-off rule), transfer/manager change (split scorecard; final score weighted by time under each manager, both managers rate their period), long leave (exclude or prorate), KPI change mid-cycle (requires approval, history kept), termination (cycle closed, no rating or final rating per policy). | V1 |
| KPI-007 | Employee self-service: my KPIs, progress, check-ins, self-assessment, history of past cycles. | V1 |
| KPI-008 | Manager view: team scorecards, who is behind, bulk reminders, assessment screen with evidence side panel. | V1 |
| KPI-009 | HR dashboards: completion %, rating distribution by department/manager, KPI achievement trends, top/bottom performers, managers who rate everyone the same. | V1 |
| KPI-010 | Outcomes: bonus formula (e.g. bonus % by rating band × eligible salary × company multiplier) → generates payroll one-time earning after approval. Increment matrix (rating × position in salary band) → recommendation for compensation change workflow. | V1 (matrix V2) |
| KPI-011 | Appeals workflow after sign-off. | V1 |
| KPI-012 | Exports: individual scorecard PDF, cycle results XLSX. | V1 |
| KPI-013 | AI: suggest KPIs for a role from the library and job description; SMART rewrite; evidence summary for managers; comment writing help; bias-language and "everyone rated 4" pattern warnings. Never proposes a score. | V1 |

### 0.5 Edge cases that must work

- Weights not totalling 100% → cannot publish template.
- Employee has two managers in the cycle → both rate their period; weighted by days.
- Target = 0 with lower-is-better (e.g. zero accidents) → banded scoring required, no division.
- Actual missing at deadline → configurable: score 0, exclude KPI and re-weight, or block submission.
- Manager is also the reviewer (small company) → reviewer step auto-skips.
- Manager leaves mid-cycle → assessment reassigned to the new manager or the skip-level; history kept.
- Employee disagrees → acknowledge with comment or appeal; appeal goes to HR/skip-level, outcome logged.
- KPI library edited during a cycle → running scorecards keep their version.
- Cycle reopened after sign-off → only by HR permission, with audit and re-sign-off.

## 1. Goals / OKRs [V2]

| ID | Requirement |
|---|---|
| PRF-001 | Goal frameworks per tenant: OKRs, KPIs, or simple goals; cascading company → department → individual; weights; measurable targets (number, %, currency, milestone). |
| PRF-002 | Check-ins with progress updates and confidence; goal library (KPI bank, as in HR Direct) per job family. |
| PRF-003 | AI goal suggestions from job description and department goals (SMART rewrite). |

## 2. Review cycles [V2]

| ID | Requirement |
|---|---|
| PRF-010 | Cycle configuration: participants (filters), period, phases (self → manager → peer/360 → calibration → sign-off), deadlines, templates per job family. |
| PRF-011 | Review templates: goals section (auto-pulled), competencies (from competency framework with level descriptors), open questions, rating scales (configurable), overall rating calculation (weighted, or manager-set). |
| PRF-012 | 360 feedback: nominations by employee, approval by manager, anonymity threshold (min N responses before showing). |
| PRF-013 | Calibration: 9-box / distribution view across teams, rating changes with justification, audit. |
| PRF-014 | Sign-off and employee acknowledgement; dispute/comment. |
| PRF-015 | Outcomes link to compensation planning [V3] and PIP. |
| PRF-016 | AI review assistant: summarizes the period's evidence (goal check-ins, feedback received, recognitions, 1:1 notes the employee shared) for the manager; writing help; bias-language detection ("abrasive", gendered terms); never suggests a rating. |
| PRF-017 | Probation and project-end reviews using the same engine. |

## 3. Continuous performance [V2]

| ID | Requirement |
|---|---|
| PRF-020 | 1:1 meetings: recurring agenda, shared/private notes, action items; AI agenda suggestions from open items. |
| PRF-021 | Continuous feedback (request/give, private to manager or public praise → Engagement recognition). |
| PRF-022 | Performance Improvement Plans with goals, check-ins, outcomes; strictly permissioned. |

## 4. Competencies & skills [V2]

| ID | Requirement |
|---|---|
| PRF-030 | Competency framework (core, functional, leadership) with proficiency levels mapped to job titles/grades. |
| PRF-031 | Employee skills profile (self-declared + manager-verified + inferred by AI from CV/projects, flagged as inferred). |
| PRF-032 | Development plans (IDP) with actions linked to learning [V3]. |

## 5. Learning (LMS) [V3]

Course catalog (internal content: video, documents, quizzes; external links), assignments by rule (e.g. all
new joiners → Code of Conduct), due dates, completion tracking, certificates, compliance training reminders,
AI-generated quizzes from a policy document, learning paths per role.

## 6. Succession & workforce planning [V3]

Critical roles, successor nominations with readiness, talent pools, headcount planning by department with
budget, position management (positions as first-class objects with vacancy tracking).
