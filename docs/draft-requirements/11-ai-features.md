# 11 — AI Features

AI provider: **Claude** (Anthropic API directly, or via AWS Bedrock where data residency requires).
Default models are chosen per feature by cost/latency (⚠️ finalize per feature during design).

## 1. AI platform rules (apply to every feature)

| ID | Rule | Ver |
|---|---|---|
| AI-P01 | One AI module behind a port (`application/ports/ai.port.ts` → `infrastructure/ai/`). No SDK imports elsewhere. | V1 |
| AI-P02 | Prompts are versioned files in the repo; outputs are structured and validated with zod; invalid output → retry/fallback, never pass-through. | V1 |
| AI-P03 | **Permission-aware context**: the model only receives data the requesting user is allowed to see, fetched via the same use cases/permission checks as the UI. Tools the copilot calls run as the user. | V1 |
| AI-P04 | **PII minimization**: send only fields the feature needs; redact identifiers (CNIC, IBAN, passport) unless the task is about them. | V1 |
| AI-P05 | Retrieval (RAG) uses pgvector; every embedding row carries `tenantId` and source permissions; retrieval filters server-side. | V1 |
| AI-P06 | **Human-in-the-loop**: AI never commits a write. It drafts; the user confirms. Pay, employment status, performance and hiring outcomes always require an explicit human decision recorded with the AI suggestion id. | V1 |
| AI-P07 | Every AI suggestion is logged (feature, prompt version, model, inputs hash, output, accepted/edited/rejected, by whom). Log retention configurable; prompt/response bodies retained N days (⚠️ decide). | V1 |
| AI-P08 | **Evals as tests**: each feature has a golden dataset + automated eval in CI on prompt/model change; regression blocks merge. | V1 |
| AI-P09 | **Metering & limits**: tokens and cost per tenant/feature/user; plan quotas; per-tenant rate limits; graceful degradation when quota hit. | V1 |
| AI-P10 | **Tenant controls**: per-feature toggles; option to disable AI entirely; high-risk features (hiring, performance) default off. | V1 |
| AI-P11 | Transparency: AI-generated content is labelled; "why am I seeing this?" explanations; citations for policy answers. | V1 |
| AI-P12 | Expensive/long tasks run on a queue with progress; streaming for chat. | V1 |
| AI-P13 | Prompt-injection defences: uploaded documents and user content treated as data; tool calls validated against permissions; no tool can exfiltrate across tenants. | V1 |
| AI-P14 | Multilingual: understand English, Roman Urdu and Urdu [V1]; Arabic [V2]. | V1 |

## 2. Feature catalog

| ID | Feature | Who | What it does | Guardrail | Ver |
|---|---|---|---|---|---|
| AI-COP | **Orcavia Copilot** (chat in web, mobile, and Teams/Slack [V2]) | Everyone | Answers questions about my data (balances, payslip, attendance, team) and policies; performs actions via tools that create *drafts* (leave request, letter request, expense from receipt, regularization) which the user confirms; navigates ("open John's profile") | Runs as the user; drafts only; citations | V1 |
| AI-POL | Policy & handbook Q&A | Everyone | RAG over tenant's knowledge base, policies and country-pack explanations with citations; "I don't know" + raise ticket when unsure | Citations required; gaps report | V1 |
| AI-DOC | Document intelligence | EMP, HRA | Extract fields from CNIC, passport, visa, Emirates ID, degrees, experience letters, bank letters; classify document type; detect expiry; flag mismatches with profile | Human confirms extracted values | V1 |
| AI-RCP | Receipt scanning | EMP | Expense line auto-fill, duplicate detection, policy breach flags | User confirms | V1 |
| AI-PAY | Payroll anomaly review | PAY, FIN | Per-employee variance explanation ("+18% due to 12h overtime and arrears for March increment"), outlier detection (missing allowance, unusual deduction, ghost employee signals), pre-approval checklist | Flags only; reviewer acknowledges | V1 |
| AI-EXP | Explain my payslip / balance | EMP | Plain-language explanation of any payslip line or balance change using the explainability graph | Read-only | V1 |
| AI-APR | Approval context card | MGR | Summarizes request + relevant context (balance, team coverage, policy fit, history) with a neutral recommendation note | Never auto-approves | V1 |
| AI-WRT | Writing assistant | HR, MGR | Drafts letters, announcements, emails, job descriptions; rewrite tone; translate (Urdu/Arabic) | Human edits & sends | V1 |
| AI-HD | Helpdesk triage & reply drafts | HR agents | Categorize, prioritize, route, draft answer from KB | Agent sends | V1 |
| AI-IMP | Smart import | HRA | Column mapping, data clean-up (name casing, phone formats, date formats), anomaly detection in uploaded files | Preview before commit | V2 |
| AI-NLR | Natural-language reports | HR, OWN | NL → semantic-layer query → chart; shows query; save as report | Permission-scoped; aggregates for sensitive data | V2 |
| AI-REV | Performance review assistant | MGR | Evidence summary, writing help, bias-language detection | No ratings suggested | V2 |
| AI-GOAL | Goal assistant | EMP, MGR | SMART goal rewriting, suggestions from role and team goals | Suggestions only | V2 |
| AI-1on1 | 1:1 assistant | MGR, EMP | Agenda from open items; meeting note summaries & action items | Private notes stay private | V2 |
| AI-SRV | Survey insights | HRM | Themes, sentiment, anonymized quotes, actions | Respects anonymity threshold | V2 |
| AI-REC | Recruitment assistant | REC | JD drafting, CV parsing, candidate-vs-requirements notes with evidence, interview kits, email drafts | Feature-flagged; no auto-reject; bias audit report | V2 |
| AI-ONB | Onboarding plan generator | MGR | 30/60/90-day plan from JD | Manager edits | V2 |
| AI-DIG | Leadership HR digest | OWN, HRM | Monthly narrative of key metrics/anomalies with links | Aggregated | V2 |
| AI-ATT | Attendance anomaly insights | HRA | Patterns (buddy punching, impossible travel) as insights | No automatic penalties | V2 |
| AI-WFB | Workflow builder by description | HRA | "Expense over 50k needs CFO after manager" → draft workflow + simulation | Admin reviews & publishes | V3 |
| AI-LMS | Course/quiz generator | HRM | Quizzes and microlearning from policy docs | Author reviews | V3 |
| AI-ATR | Attrition insights | HRM, OWN | Team-level risk drivers | Opt-in; aggregated; no action | V3 |
| AI-AGT | Autonomous agents | HR ops | Multi-step jobs (e.g. "prepare month-end: chase missing timesheets, reconcile, draft payroll notes") with step-by-step approval | Approval at each side-effect | V3 |

## 3. Copilot tool surface (V1)

Read tools: `get_my_profile`, `get_leave_balances`, `get_payslip(period)`, `explain_payslip_line`, `get_attendance(range)`,
`get_team_status` (managers), `search_people`, `search_policies`, `get_request_status`.
Draft tools (require confirmation card): `draft_leave_request`, `draft_letter_request`, `draft_expense_from_receipt`,
`draft_regularization`, `draft_helpdesk_ticket`. Approver tools: `get_pending_approvals`, `approve/reject` (confirmation card, never silent).

## 4. Compliance notes

- EU AI Act (employment = high-risk), NYC LL144 (AEDT bias audits) matter if we sell into those markets or to
  multinationals; Pakistan's draft Personal Data Protection Bill and UAE PDPL apply to personal data processing.
  ⚠️ Legal review before V2 hiring/performance AI.
- Model provider: zero data retention / no training on customer data agreement required.
