# Orcavia — Project Context (start every new session here)

> **For Claude:** read this whole file first, then the locked specs it points to. Don't re-ask anything
> listed as decided. Continue from **§7 Next step**. Keep this file updated at the end of every session
> (status table, decisions, next step).
>
> Last updated: 2026-10-18

---

## 1. What we're building

**Orcavia** — an AI-first, multi-tenant **HRMS (SaaS)**. Our own product: NathanHR's "HR Direct" is
**inspiration only** (basic flows), rebuilt with a modern architecture, many improvements and AI in every
module. No data migration from HR Direct.

- **Markets:** Pakistan first, then UAE.
- **Releases:** V1 "Run HR in Pakistan" = **all HRMS basics + a fully configurable KPI evaluation system** + AI copilot.
  V2 = advanced performance (OKRs, 360, calibration, 1:1s), recruitment, surveys, UAE pack, Urdu/Arabic, SSO.
  V3 = learning, succession, compensation cycles, multi-country, PEO, public API.
- **AI:** Claude. AI drafts/explains/flags; a person always makes the final decision (pay, employment, ratings, hiring).

## 2. Who the user is and how they want to work

- The user is the project owner/lead, technical, building a large-scale product.
- **Models:** plan with **Opus 5.5**, execute with **Sonnet 5.5**.
- **Long text is hard to review.** Prefer short answers, tables, visuals. Big documents only as reference.
- **Wants every case covered** for every feature — nothing silently assumed.
- **Locks requirements one area at a time** using the **grilling** method (below). Doesn't want to move to
  design or code until requirements and scope are fully locked.
- After all features are locked: **re-validate the stack against every feature**, then a **full UI/UX design**, then build.
- Tone that works: plain, direct, concise; recommendation first; no fluff; no hype.
- **Git rules:** commit author email is **pagan1581998@gmail.com** (never the Nathan Digital address; set per repo
  with `git config user.email`). **No AI attribution** in commits or PRs: no `Co-Authored-By`, no "Generated with" lines.
  Plain conventional commit messages.

## 3. Working method (grilling)

- Skills installed at user level: `~/.claude/skills/grill-me` and `~/.claude/skills/grilling` (from Matt Pocock's skills repo).
- Each area is grilled in **rounds**. Each question is numbered, has a recommended answer, and is worded so
  **"yes" accepts the recommendation**. Format:
  ```
  ❓ **Q1** - **<title>**: <question, options, cases>
  ➡️ <recommended answer>
  ```
- User answers "yes", "yes to the rest", or changes specific numbers. Questions depending on open answers go to the next round.
- When an area is done → write **`docs/specs/NN-<area>.md`** with *every* decision (all of them, with IDs), mark it LOCKED,
  update this file's status table.

## 4. Status of every area

| # | Area | Status | File |
|---|---|---|---|
| 01 | Environment & stack | **LOCKED** (cloud/hosting + IaC deferred to setup) | `docs/specs/01-environment.md` |
| 02 | Entities, structure, roles & permissions | **LOCKED** | `docs/specs/02-entities.md` |
| 03 | Employee profile & lifecycle (Core HR) | **NEXT** | — |
| 04 | Onboarding | Pending | — |
| 05 | Attendance & shifts | Pending | — |
| 06 | Leave | Pending | — |
| 07 | WFH / remote work | Pending | — |
| 08 | Requests & letters | Pending | — |
| 09 | Assets / IT equipment | Pending | — |
| 10 | Expenses & loans | Pending | — |
| 11 | KPI evaluation (V1, fully configurable) | Pending | — |
| 12 | Offboarding & final settlement | Pending | — |
| 13 | Payroll (Pakistan pack) | Pending | — |
| 14 | Announcements & policies, HR helpdesk, reports & dashboards, AI copilot | Pending | — |
| — | Stack re-validation against all features | After 03–14 | — |
| — | UI/UX design | After re-validation | — |

Feature specs number from 03 in the order above (the order itself is locked in `02-entities.md` §15).

## 5. Key decisions (summary — specs are the source of truth)

**Environment (`01`):** separate repos `orcavia` (NestJS API + worker), `orcavia-ui` (React), `orcavia-app` (Expo),
`orcavia-workspace` (this repo) · GitHub · Jira key `ORC` · npm scope `@orcavia` · Node 24 LTS + pnpm ·
NestJS modular monolith (clean/hexagonal) · PostgreSQL + pgvector + Prisma + RLS · zod → OpenAPI → generated
`@orcavia/api-client` · in-house auth (argon2, JWT, TOTP, phone OTP) · Redis + BullMQ · Socket.IO ·
React 19 + Vite + TanStack Router/Query + Zustand + Tailwind v4 + shadcn/ui · Expo + expo-router + NativeWind ·
Claude + Voyage embeddings · Gotenberg PDFs · exceljs · date-fns · Vitest/Jest/Playwright · Sentry + OTel ·
branches **`dev → staging → main`** (feature PRs squash into dev; promotions use merge commits; hotfix from main with back-merge).
**Deferred:** cloud provider/region (proposed AWS `me-central-1`) and IaC (proposed CDK).

**Entities (`02`):** tenant = one customer account (groups = one tenant, many companies) · subdomain per tenant ·
Company → own department tree · locations can serve several companies · **visibility by role scope, not structure** ·
user ≠ employee (multi-tenant users; users without employee records) · one active employment per tenant in V1 ·
one direct manager, person-based structure (positions in V3) · seven worker types, probation is a status ·
three levels of required employee data (create / payroll-ready / complete) · Pakistani name + CNIC rules ·
effective-dated job/salary/bank/status · inter-company transfer rules · archive-not-delete ·
10 default roles, 7 scopes, sensitive field groups, separation of duties, read-only "view as", dated role assignments.

## 6. Where things live

| What | Where |
|---|---|
| Locked specs (source of truth) | `docs/specs/` |
| Original broad draft (V1/V2/V3 ideas per module; used as **input** for each grilling round, superseded by specs) | `docs/draft-requirements/` |
| Claude Code workspace blueprint (Relay structure + improvements) — used later when setting up `.claude/`, CLAUDE.md files, agents | `docs/reference/claude-setup/` |
| Reference system (read-only inspiration) | `D:\Projects\NathanHR\hrdirect-admin` (Nuxt 2/Vue), `D:\Projects\NathanHR\hrdirect-mcs-api` (Express/Mongo) |
| Visual overview of the original draft (not updated with locked specs) | https://claude.ai/artifact/Mu51faDhyPi5t7JAsydbfZ |
| GitHub repos (empty until setup) | github.com/PakDragoon/orcavia-workspace, /orcavia, /orcavia-ui, /orcavia-app |

**HR Direct lessons (don't repeat):** JWT decoded without signature check, boolean role flags checked by URL prefix,
one "requests" table for everything, manual `company_ID` filtering, logic in route files, full directory loaded into the
browser, duplicate libraries, UAE-only payroll.

## 7. Next step

Start **Area 03 — Employee profile & lifecycle (Core HR)** with the grilling skill: round 1 covers profile sections
and fields, self-service edits and approvals, lifecycle actions (probation, confirmation, transfer, promotion,
salary change, contract renewal, suspension, long leave), documents and expiries, org chart and directory —
with every case. Use `docs/draft-requirements/02-core-hr.md` as input.

## 8. Parked questions (answer when their feature comes up)

| Question | Comes up in | Draft answer |
|---|---|---|
| Target customer size and industries | Payroll / pricing | 50–2,000 employees, mixed desk + field |
| Who validates Pakistan tax/EOBI/social-security rules | Payroll | Bring in a payroll consultant before building |
| First Pakistani bank salary file formats | Payroll | HBL, Meezan, UBL, MCB, Bank Alfalah |
| Pay frequencies in V1 | Payroll | Monthly only; others V2 |
| Payslip PDF password | Payroll | Last digits of CNIC, configurable |
| Urdu UI timing | Cross-cutting | V2; RTL-ready from day one |
| Pricing model and payment gateway | Billing | Per active employee per month, tiered |
| Biometric device brands | Attendance | ZKTeco import V1, face recognition V2 |
| Minimum Android version | Mobile | Android 8+ |
| AI log retention | AI | 30 days bodies, 2 years metadata |
| SOC 2 timeline, legal review owner | Non-functional | SOC 2 Type I by V2 |
| Brand: logo, colours | Before UI/UX | Needed before design |
| Team size and V1 timeline | Planning | Drives phase plan |
| Cloud provider & region | Setup | AWS `me-central-1` |

## 9. Housekeeping state

- Local folder: `D:\Projects\Orcavia\orcavia-workspace` (app repos will be cloned inside it, gitignored).
- Git: branch `main`, pushed to github.com/PakDragoon/orcavia-workspace. The `dev → staging → main` flow applies to
  the app repos; this docs repo commits straight to `main`.
