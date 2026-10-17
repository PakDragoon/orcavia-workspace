# Improvements over the Relay blueprint (apply on top of 01-RELAY-BLUEPRINT.md)

> **For the Claude Code instance reading this:** these are changes the Relay team would make
> if starting over. Each one comes from a real problem Relay hit, noted in its **Why**. Where
> an item conflicts with the blueprint, this file wins. Items marked **[ASK]** are
> architecture decisions. Confirm them with the user before building.

---

## Part A: Claude Code setup

### A1. Put the workspace config under version control
**Why:** Relay's root folder isn't a git repo, so the root `CLAUDE.md`, `.claude/` agents,
commands, `.plans/` and `.pr-reviews/` exist only on one machine. Teammates never got them,
and a later migration plan had to move them into a repo and symlink them back.
**Do:** if you go monorepo (B1), this is automatic. Otherwise create a small
`hrms-workspace` git repo for the root files, with the app repos as sibling clones
(gitignored, not submodules) and a `scripts/bootstrap.sh` that clones them.

### A2. Don't rely on nested CLAUDE.md auto-loading. Make the path explicit.
**Why:** in a measured eval on Relay (18 sessions), **0 sessions auto-loaded a nested
CLAUDE.md while planning**, because Claude explored with Bash (`grep`/`sed`/`cat`), which
doesn't trigger the load. Nested files were read only because the root file said to.
**Do:**
- Each repo root `CLAUDE.md` gets this as its **first** rule: "Before planning or editing in
  a folder, list and read every `CLAUDE.md` on the path from that folder up to the repo root
  (`find <dir> -name CLAUDE.md` / walk up)."
- Optional mechanical version: a hook on `Read|Edit|Write` that finds the nearest
  `CLAUDE.md` files for `tool_input.file_path` and returns them as additional context.
  Check the current Claude Code hooks docs for the exact output field before building it.

### A3. Budget 150 lines for every CLAUDE.md, with the most important rules first
**Why:** agents usually `head -150` a file. Relay's BE root is 193 lines, and its last 40
lines (Known traps, Testing, Commands) went unread in the eval.
**Do:** cap repo roots at **150**, not 200 (update `check-context-size.sh`). Order sections
by cost-of-forgetting: tenancy/security/traps before stack descriptions. Install the check
in **all three** repos from day one. (Relay's mobile repo has no CLAUDE.md at all.)

### A4. Make the config portable and secret-free
**Why:** Relay's hook uses an absolute `/home/<user>/...` path, `.mcp.json` must be
gitignored because it holds raw tokens, and a cloud service-account key ended up inside
`.claude/`.
**Do:**
- Use `$CLAUDE_PROJECT_DIR` in every hook path (already done in the blueprint templates).
- Commit `.mcp.json` and reference secrets with env-var expansion (`"TOKEN": "${TRACKER_TOKEN}"`);
  the real values live in the user's shell env or a gitignored `.env.claude`.
- **Never** store keys or credentials anywhere under `.claude/`. Add `*.pem`, `*key*.json`,
  `credentials*` to `.gitignore`.

### A5. Deny rules don't cover Bash. Close the gap.
**Why:** `Read(./.env)` blocks only the built-in Read tool. `cat .env` through Bash still works.
**Do:** enable Claude Code's sandbox for OS-level file restrictions, and/or add Bash deny
patterns (`Bash(cat *.env*)`, `Bash(* .env*)`, `Bash(printenv*)`). Treat the deny list as a
guardrail, not a security boundary. Real secrets for prod never live on dev machines.

### A6. Use skills instead of plain commands for multi-step workflows
**Why:** `.claude/commands/*.md` still works, but `.claude/skills/<name>/SKILL.md` can bundle
helper scripts and reference files and can be auto-invoked when relevant.
**Do:** author the ten workflows as skills (same names). Put scripts they need
(e.g. the ERD extractor, the context-size check) inside the skill folder.

### A7. Project knowledge goes in the repo, not personal memory
**Why:** Relay built up ~50 memory files, and dozens of them were really project rules
(role checklists, date conventions, test fixtures). Memory is per-user and per-machine, so
none of it reached teammates or a second machine. It later had to be "promoted" into repo
files by hand.
**Do:** the default home for a durable fact is the nearest `CLAUDE.md` or an ADR (A8).
Memory is only for personal preferences and in-flight notes. `/day-end`'s curator should
propose **promotions** out of memory, not just memory edits.

### A8. Add ADRs for "why" decisions
**Why:** several Relay decisions have no recorded reason (e.g. "why is this a table and not
columns?" was undocumented), so nobody could safely change them later.
**Do:** `docs/adr/NNNN-title.md` (context → decision → consequences), one per non-obvious
architectural choice. CLAUDE.md states the rule and links the ADR for the why.

### A9. Treat mobile as first-class from day one
**Why:** Relay's `api-contract-tracer` and `/sync-types` check only the web app. Mobile is a
silent consumer of every DTO change.
**Do:** every cross-repo agent/command (contract tracer, sync-types, review-changes,
review-pr, day-end, commit) covers **api + web + mobile**.

### A10. Keep a context eval set from the start
**Why:** Relay measured its context quality only after months of edits (and an early
"40–58% savings" estimate turned out to be ~7% when measured).
**Do:** `.plans/context-eval/` with 3–5 fixed realistic prompts plus a scoring rubric.
Re-run them headless (plan mode, fresh clone, no memory) after any big CLAUDE.md
restructure, and compare scores.

### A11. Packages: pick one of each, and keep versions consistent
**Why:** Relay grew duplicates that each add a second way to do the same thing (seen in
the blueprint's §1.2 inventory). Claude copies whichever pattern it sees first, so
duplicates spread.

| Area | Relay has | HRMS: use one |
|---|---|---|
| Web styling | Tailwind **+** MUI **+** Emotion **+** styled-components | Tailwind + one component library (shadcn/ui on Radix, or MUI alone). Never four. |
| Dates | `dayjs` (BE, mobile) **and** `date-fns` (BE, web) | One library in all three apps, wrapped in shared helpers (see B4) |
| Validation | `class-validator` (BE), `zod` (web), `yup` (mobile) | `zod` everywhere, schemas in `packages/contracts` (e.g. `nestjs-zod` on the BE) |
| Queues | `bullmq` **and** legacy `bull` v4 | `bullmq` only |
| Spreadsheets | `xlsx` 0.18.5 **and** `exceljs` | `exceljs` only. The npm `xlsx` 0.18.5 is the last version SheetJS published to npm and has known vulnerabilities (CVE-2023-30533, CVE-2024-22363); fixes ship only from SheetJS's own CDN. |
| Date pickers | `react-datepicker` **and** `react-day-picker` | One |
| Tailwind | v3 config with a v4 `@tailwindcss/postcss` dev dep (web) | Tailwind v4 in web; match NativeWind's supported version in mobile |
| Node | CI on `node:20`, Docker on `node:22` | One Node LTS pinned in `.nvmrc` + `engines` + the same image in CI and Docker |
| Tool versions | husky 8 vs 9, eslint 9 vs 10, three different Prettier styles (semicolons, quotes, width 80 vs 100) | Same versions and **one shared** Prettier + ESLint config across apps (a monorepo makes this automatic) |
| Dependency bot | Renovate on the backend only | Renovate on every repo/app, same rules |

Also:
- **Microsoft Clarity on HR screens:** session recordings can capture salaries, IDs and
  health/leave details. Use Clarity's **strict masking** mode, exclude sensitive routes
  entirely, and load analytics only after consent where GDPR or similar applies.
- **AI SDK:** add `@anthropic-ai/sdk` only inside `infrastructure/ai/` (see B10). No other
  layer imports it.
- **Secrets on AWS:** keep app secrets in Secrets Manager / SSM Parameter Store and reference
  them from the ECS task definition. Never put them in the image or in plain task-definition env.

### Keep as-is (these worked well)
Progressive disclosure · least-privilege agents · read-only curator plus `/day-end` approval
gate · saved PR reviews as house style · read-only tracker MCP · phased plans with stop
points · "touches N places" checklists · parity tests for must-match code.

---

## Part B: Product architecture (lessons from Relay's bug history)

HRMS data (salaries, IDs, health and leave records, performance reviews) is more sensitive
than Relay's, so these matter more here.

### B1. [ASK] Monorepo with a shared contracts package
**Why:** many of Relay's recurring bugs are copies drifting apart: BE DTO vs FE types (needs
`/sync-types`), an FE sending a field the deployed DTO doesn't declare (400s on every call),
realtime channel names that must be byte-identical in two repos, and a new role touching
5 BE + 4 FE places.
**Do:** a pnpm/Turborepo (or Nx) monorepo: `apps/api`, `apps/web`, `apps/mobile`,
`packages/contracts` (zod schemas → types, enums, permission keys, realtime event catalog
and channel builders), `packages/ui` if web/mobile share anything. Alternative if separate
repos are required: generate an OpenAPI spec from NestJS and codegen typed clients for
web + mobile in CI, and fail CI on drift.

### B2. Enforce tenant isolation in the data layer, not by convention
**Why:** Relay has no auto-scoping. Every query must remember the tenant filter, and "a
missing filter = cross-tenant leak" is a rule people must remember.
**Do:** Postgres Row-Level Security keyed on a per-request `app.tenant_id` setting, or a
Prisma client extension that injects `tenantId` on every model that has it (and throws
if missing). Add an e2e test that calls every endpoint as tenant B against tenant A's ids
and expects 404/403.

### B3. Permission-based access control, not role checks
**Why:** Relay checks 23 role names directly, so adding a role is a code change in ~10
places plus a data script. HRMS roles vary per customer (HR admin, payroll, recruiter,
line manager, department head, employee).
**Do:** roles are data (`Role → Permission[]`, per tenant). Code checks permissions plus
scope: `can('leave.approve', { orgUnitId })`. Permission keys live in `packages/contracts`.
A new role is a row, not a deploy.

### B4. One date and time convention, decided on day one
**Why:** Relay ended up with three stored-time conventions (midday UTC, 00:00 UTC, Texas
midnight) for "a calendar day". This produced duplicate rows and off-by-one-day bugs for a
team and customers on opposite sides of UTC.
**Do:** calendar days (leave dates, attendance date, holidays, pay-period bounds) use the
Postgres `date` type with no time component. Instants (clock-in/out, approvals) use
`timestamptz`. Timezone is stored on tenant → location → employee and resolved server-side.
Run CI tests under two timezones (e.g. `TZ=America/Chicago` and `TZ=Asia/Karachi`).
Write the convention into `packages/contracts` helpers plus a nested CLAUDE.md.

### B5. Impersonation that fails closed
**Why:** Relay's "View Dashboard" relies on the FE sending `actingEmployeeId` on every write.
If it's missing, the admin's full bypass fires silently (fails open).
**Do:** impersonation is a server-side session: the token carries an "acting as" claim with
the real actor in it. Every request is evaluated as the viewed user automatically. Every
impersonated write is audit-logged with both ids. No per-request parameter to forget.

### B6. One principal id space
**Why:** Relay's `request.user.id` is an Employee, Admin or Executive id from independent
sequences. They collide, and writing an admin id into an employee FK throws.
**Do:** UUIDs everywhere. `request.user` carries `userId` plus a typed `actor`
(`{ kind: 'employee' | 'platformAdmin', id }`). Use branded TS types (`EmployeeId`,
`UserId`) so mixing them doesn't compile.

### B7. Audit log and effective dating are core, not features
**Why (HRMS-specific):** "who changed this salary/title/manager, when, and effective from
when" is a legal and payroll requirement. Retrofitting history tables is painful.
**Do:** an append-only `AuditEvent` table written in the same transaction as the change.
Effective-dated tables for job, compensation and org assignment (`effectiveFrom`,
`effectiveTo`). Soft delete for people records. Define a field-level PII classification
and use it to drive log redaction and API response filtering.

### B8. Every mutation emits a typed event
**Why:** Relay had silent writes (e.g. CSV bulk creates never pushed realtime updates) and a
paginated FE cache that discarded refetches, which left stale dropdowns.
**Do:** a domain-event catalog in `packages/contracts`. Use cases emit after commit (outbox
pattern if you need guarantees). Realtime, notifications and cache invalidation all consume
the same events. Bulk/import paths go through the same use cases as single writes.

### B9. Test tooling from day one, isolated from real services
**Why:** Relay's FE never got a test runner, and BE e2e once loaded real Pusher keys from
`.env` and broadcast test events to live clients.
**Do:** Vitest + Testing Library (web), Jest/RNTL (mobile), Playwright (e2e) set up in the
first sprint. The test env loads **only** `.env.test`. External services are faked or point
at sandbox accounts by construction. Use a dedicated seeded test tenant id.

### B10. AI features: design rules
- **One AI module behind a port** (`application/ports/ai.port.ts` → `infrastructure/ai/`).
  No SDK calls scattered across use cases. Prompts are versioned files in the repo.
  Outputs are structured and validated with zod. Long or expensive calls run through a queue.
- **Tenant isolation for retrieval:** every embedding and vector row carries `tenantId`, and
  retrieval filters on it server-side (same rule as B2). Never share an index across tenants
  without that filter.
- **PII minimization:** send the model only the fields a feature needs. Redact identifiers
  where possible. Decide and document what prompts and responses are logged and for how long.
- **Evals as tests:** each AI feature gets a golden dataset and an automated eval that runs
  in CI on prompt or model changes.
- **Metering:** token usage and cost per tenant and per feature, rate limits per tenant.
- **Regulation (important for HR):** AI used in hiring, promotion, performance or
  termination decisions is regulated in several markets. The EU AI Act classifies
  employment uses as high-risk, and NYC Local Law 144 requires bias audits for automated
  employment decision tools. Build in human-in-the-loop approval, explanations, an audit
  trail of AI suggestions vs human decisions, and per-tenant feature toggles. Confirm the
  target markets' requirements with the user and legal.
- Put these rules in `apps/api/src/infrastructure/ai/CLAUDE.md` (and the feature folders
  that use AI).
