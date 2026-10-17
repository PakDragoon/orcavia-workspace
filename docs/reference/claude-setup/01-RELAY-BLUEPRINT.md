# Claude Code Workspace Blueprint (modeled on "Relay")

> **For the Claude Code instance reading this:** you are setting up a new project, an
> **HRMS with AI features**, following the same Claude Code structure as an existing
> production project called **Relay**. This document describes how Relay is organized.
> Copy the **structure, mechanics and conventions**. Do **not** copy Relay's domain
> content (dealerships, deals, scheduling). Ask the user for the HRMS domain facts
> wherever a template below has a `<placeholder>`.
>
> If `02-IMPROVEMENTS.md` is also provided, it **overrides** any section of this file it
> names. Apply it on top of this blueprint.

---

## 0. What Relay is (so the examples make sense)

Relay is a multi-tenant automotive dealership SaaS. Three apps ship together as one product:

| Folder | Stack | Role |
|---|---|---|
| `relay-automotive/` | NestJS + Prisma + PostgreSQL, BullMQ/Redis, Clean/Hexagonal architecture | Backend API + workers |
| `relay-automotive-ui/` | React 19 + Vite + TypeScript + Zustand + TanStack Query/Router + Tailwind/MUI | Web SPA |
| `mobile-app/` | Expo / React Native (EAS) | Mobile app on the same API |

Each app is its **own git repo**. The parent folder holding them is **not** a git repo. It
is an "orchestrator" workspace that holds the shared Claude Code config. Claude is always
launched **from the workspace root**, so all three repos and all config are in scope.

---

## 1. Folder layout to reproduce (rename for HRMS)

```
<Workspace>/                         ← launch `claude` HERE (not inside a repo)
├── CLAUDE.md                        ⚡ always loaded: product primer, cross-repo rules, glossary, tooling map
├── .gitignore                       belt-and-braces: .mcp.json, .env*, *.local, .claude/settings.local.json
├── .mcp.json                        project MCP servers (gitignored, holds tokens)
├── .mcp.json.example                committed-style template with placeholder tokens
│
├── .claude/
│   ├── settings.json                team rails: allow-list, deny-list, hooks
│   ├── settings.local.json          personal permission grants (never shared)
│   ├── SETUP.md                     human-readable map of this whole setup (not loaded by Claude)
│   ├── CHEATSHEET.md                human quick reference for commands/agents (not loaded = 0 tokens)
│   ├── commands/                    slash-command workflows (one .md each)
│   ├── agents/                      specialist subagents (one .md each)
│   └── hooks/                       hook scripts (e.g. auto-format on edit)
│
├── .plans/                          implementation plans, phase handouts, eval baselines
├── .pr-reviews/                     every PR review saved (searchable archive = house style by example)
│   └── handoff/                     cross-session handoff notes
├── .<tracker>/                      snapshots of the ticket board (e.g. .trello/), refreshed by a command
│
├── <backend-repo>/                  own git repo
│   ├── CLAUDE.md                    repo root context
│   ├── prisma/CLAUDE.md             schema/migration/seeding rules
│   ├── scripts/check-context-size.sh   enforces CLAUDE.md line budgets (lint-staged)
│   └── src/**/CLAUDE.md             nested, per layer / per feature (see §2.3)
│
├── <web-repo>/                      own git repo
│   ├── CLAUDE.md
│   └── src/**/CLAUDE.md
│
└── <mobile-repo>/                   own git repo
    ├── CLAUDE.md
    └── **/CLAUDE.md
```

Suggested HRMS names: `hrms-api/`, `hrms-web/`, `hrms-mobile/` (confirm with the user).

Persistent per-user memory lives outside the workspace, in
`~/.claude/projects/<workspace-path>/memory/` (`MEMORY.md` index + one file per fact).

---

## 1.1 Code structure inside each repo (reproduce this)

### Backend: NestJS + Prisma, Clean/Hexagonal

```
<backend-repo>/
├── prisma/                  schema.prisma, migrations/ (never hand-edited), seeders/
├── src/
│   ├── main.ts              HTTP app bootstrap (global ValidationPipe whitelist + forbidNonWhitelisted)
│   ├── worker-main.ts       separate process for queue workers (worker.module.ts)
│   ├── app.module.ts
│   ├── domain/              ← innermost layer, no framework imports
│   │   ├── entities/        <name>.entity.ts (domain types, not Prisma models)
│   │   ├── interfaces/  constants/  factory/
│   ├── application/         ← depends on domain only
│   │   ├── use-cases/<feature>/   one class per business operation (e.g. CreateLeaveRequestUseCase)
│   │   ├── ports/           interfaces the use cases depend on (repos, external services)
│   │   ├── dtos/            request/response DTOs (class-validator)
│   │   └── services/        cross-cutting + 3rd-party services (timezone, email, etc.)
│   ├── adapters/            ← HTTP edge
│   │   ├── controllers/<feature>/   thin: validate → call use case → wrap response
│   │   └── guards/          (+ jwt strategy)
│   ├── infrastructure/      ← I/O, implements ports
│   │   ├── orm/             prisma.service.ts, prisma.module.ts, repositories/
│   │   ├── queues/<queue>/  BullMQ processors (one folder per queue)
│   │   └── <integration>/   external systems (Relay: dms/)
│   └── common/              config, constants, decorators, filters, pipes, middleware,
│                            logger, security/guards, utils, emails, cron, helpers
└── test/
    ├── jest-e2e.json  setup-e2e-env.ts
    ├── utils/               app factory + fixture builders
    └── <feature>/           e2e specs (supertest against a dedicated test DB)
```

The **feature folder name repeats across layers**: `use-cases/employee/`,
`controllers/employee/`, `orm/repositories/employee…`, `test/employee/`. Dependency rule:
inner layers never import outer ones, and use cases never import `PrismaService` (they get
repositories through ports). Responses are wrapped in `{ data, statusCode, message?, timestamp }`.

### Web: React 19 + Vite + TS

```
<web-repo>/src/
├── core/
│   ├── config/              api.ts (the ONE axios instance), queryClient.ts, tableConfig…
│   ├── entities/            TS types mirroring backend response DTOs
│   └── store/               global Zustand stores
├── common/
│   ├── components/  hooks/  types/  utils/  constants/     shared across features
├── features/<feature>/      self-contained slice
│   ├── components/          functional components only
│   ├── hooks/               TanStack Query wrappers + derived state
│   ├── services/<x>Api.ts   API calls on the shared axios instance
│   ├── stores/              feature-scoped Zustand
│   ├── schemas/             zod schemas for react-hook-form
│   ├── types/  utils/  constants/
│   └── index.ts             public surface of the feature
├── pages/                   route-level screens
├── layout/                  BlankLayout (public) vs DefaultLayout (authed shell)
├── styles/  assets/
└── AppRoutes.tsx            TanStack Router, beforeLoad guards (auth, role, product gating)
```

Data flow: `services/*Api.ts` → hook in `hooks/` (TanStack Query) → component. Server data
never goes into Zustand. **No sibling-feature imports.** Shared code moves to `common/` or
`core/`. Import alias `@/* → src/*`.

### Mobile: Expo + React Native

Stack: Expo SDK (EAS builds) · expo-router (file-based routing) · TanStack Query · Zustand ·
axios · react-hook-form · NativeWind (Tailwind) · pusher-js · Firebase (push + analytics).

```
<mobile-repo>/
├── app/                     expo-router routes
│   ├── _layout.tsx  index.tsx
│   ├── (auth)/login.tsx     route group: unauthenticated
│   ├── (tabs)/_layout.tsx   tab bar; one folder per tab (dashboard/, profile/, notifications/…)
│   └── onboarding/
├── components/              Base/ UI/ FormElements/ Typography/ Icon/ TabBar/ Views/ Providers/ common/
├── hooks/                   data hooks (useGetPaginatedX, usePusherUpdates…)
├── lib/                     api/ (axios client) · store/ (Zustand) · hooks/ · config.ts · analytics.ts · utils/
├── constants/               Colors, Paths, realtime channel names (must match the backend)
├── types/  utils/           (authStorage, biometricAuth, toast)
├── plugins/                 Expo config plugins
└── app.config.js  eas.json  tailwind.config.js  metro.config.js
```

Same rules as web: one API client, server state in TanStack Query, client state in Zustand,
and role/permission decisions come from the backend. Env/config goes through `app.config.js`
and EAS profiles, never hard-coded.

---

## 1.2 Packages, tooling & infrastructure (Relay's actual stack)

> Reproduce the **categories**. For HRMS, use the latest stable versions, and apply the
> "pick one" cleanups in `02-IMPROVEMENTS.md` §A11 instead of copying Relay's duplicates.

### Backend (`relay-automotive`)

| Area | Packages |
|---|---|
| Framework | `@nestjs/{common,core,platform-express,config,axios,schedule,serve-static,swagger}` v11, `rxjs`, `reflect-metadata` |
| DB / ORM | `prisma` + `@prisma/client` v6 (PostgreSQL); `mysql2` only for a legacy read-only DB |
| Auth & security | `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`, `passport-local`, `bcrypt`, `otpauth` (MFA/TOTP), `helmet`, `@nestjs/throttler` |
| Validation | `class-validator`, `class-transformer` (DTOs), global `ValidationPipe` with whitelist + forbidNonWhitelisted |
| Queues / Redis | `bullmq` + `@nestjs/bullmq` (legacy `bull` v4 coexists), `ioredis`, `@bull-board/*` (queue dashboard) |
| AWS | `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner` (signed URLs), `@aws-sdk/lib-storage` (multipart upload), `@aws-sdk/client-ses` (email) + `nodemailer` |
| Realtime / push | `pusher` (server), `firebase-admin` (push notifications) |
| Payments | `stripe` |
| Files / docs | `exceljs`, `xlsx`, `fast-csv`, `pdfmake`, `sharp` (images), `fast-xml-parser` (DMS XML), `he` |
| Dates | `dayjs`, `date-fns` |
| Logging | `winston` + `winston-daily-rotate-file` |
| API docs | `@nestjs/swagger` + `swagger-ui-express` |
| Misc | `uuid`, `dotenv` |
| Dev / test | `jest` 30, `@swc/jest` + `@swc/core` (fast TS transform), `ts-jest`, `supertest`, `@nestjs/testing`, `@nestjs/cli` |
| Lint / format | `eslint` (flat config `eslint.config.mjs`), `typescript-eslint`, `eslint-config-prettier`, `eslint-plugin-prettier`, `prettier` |
| Git hooks | `husky` v9 + `lint-staged` |

### Web (`relay-automotive-ui`)

| Area | Packages |
|---|---|
| Core | `react` 19, `react-dom`, `vite` 6, `@vitejs/plugin-react`, `typescript` |
| Routing | `@tanstack/react-router` |
| Server state | `@tanstack/react-query` (+ `react-query-devtools`) |
| Client state | `zustand` |
| HTTP | `axios` (single instance) |
| Forms / validation | `react-hook-form`, `@hookform/resolvers`, `zod`, `zod-to-json-schema`, `@rjsf/core` + `@rjsf/validator-ajv8` (JSON-schema dynamic forms) |
| Styling / UI | `tailwindcss` 3 + `postcss` + `autoprefixer` + `prettier-plugin-tailwindcss`; `@mui/material` 7 + `@mui/icons-material` (+ `@emotion/*`); `styled-components`; `framer-motion`; `react-icons` |
| Calendar / dates | `@fullcalendar/{react,daygrid,interaction}`, `react-datepicker`, `react-day-picker`, `date-fns` |
| Charts / widgets | `recharts`, `react-gauge-component`, `react-circular-progressbar`, `react-select`, `react-colorful`, `react-qr-code`, `react-easy-crop` |
| Drag & drop | `react-dnd` + `react-dnd-html5-backend` |
| Export | `jspdf`, `html2canvas-pro`, `pptxgenjs`, `xlsx` |
| Realtime / chat | `pusher-js`, `@cometchat/chat-uikit-react` |
| Analytics | `firebase` (GA4 analytics), **`@microsoft/clarity`** (session recordings + heatmaps) |
| Security UX | `zxcvbn` (password strength) |
| Test libs (installed, no runner wired yet) | `@testing-library/{react,dom,jest-dom,user-event}` |
| Lint / format | `eslint` 9 flat config + `eslint-plugin-{react-hooks,react-refresh,import,unused-imports,prettier}`, `prettier` + `prettier-plugin-tailwindcss` |
| Git hooks | `husky` v8 + `lint-staged` |
| Build | `@svgr/rollup` (SVG as components) |

### Mobile (`mobile-app`)

| Area | Packages |
|---|---|
| Core | `expo` SDK 53, `react-native` 0.79, `react` 19, `expo-router`, `expo-dev-client`, `expo-updates` (OTA), `expo-build-properties` |
| Navigation | `expo-router` (built on `@react-navigation/{native,bottom-tabs}`), `react-native-screens`, `react-native-safe-area-context`, `react-native-gesture-handler`, `react-native-reanimated` |
| State / data | `@tanstack/react-query`, `zustand`, `axios`, `jwt-decode` |
| Forms | `react-hook-form`, `@hookform/resolvers`, `yup` |
| Styling | `nativewind` 4 + `tailwindcss` 3, `lucide-react-native`, `@expo/vector-icons`, `expo-linear-gradient`, `expo-blur` |
| Device | `expo-{notifications,location,local-authentication (biometrics),secure-store,image-picker,document-picker,file-system,media-library,sharing,haptics,background-task,task-manager}` |
| Storage | `@react-native-async-storage/async-storage`, `expo-secure-store` (tokens) |
| UI widgets | `@gorhom/bottom-sheet`, `react-native-calendars`, `react-native-gifted-charts`, `react-native-modal`, `react-native-toast-message`, `react-native-svg` |
| Realtime / chat | `pusher-js`, `@cometchat/chat-sdk-react-native` + `chat-uikit-react-native` |
| Identity | `react-native-persona` (ID verification) |
| Analytics | `@react-native-firebase/{app,analytics}`, **`@microsoft/react-native-clarity`** |
| App updates | `sp-react-native-in-app-updates` (store update prompt) |
| Lint / format | `eslint-config-expo`, `eslint-plugin-{react,react-native,react-hooks,import,jsx-a11y,prettier}`, `prettier` + `prettier-plugin-tailwindcss` |
| Git hooks | `husky` v9 + `lint-staged` |

### Pre-commit pipeline (all three repos)

`husky` → `.husky/pre-commit` runs `npx lint-staged --no-stash`. lint-staged config in `package.json`:

```json
"lint-staged": {
  "**/*.{js,ts,tsx}": ["eslint --fix", "prettier --write"],
  "**/CLAUDE.md": "bash scripts/check-context-size.sh"
}
```

(Mobile also runs `prettier --write` on `*.{json,md,css}`.) The `prepare` script installs
husky on `npm install`. Claude's PostToolUse Prettier hook formats each file as it's edited.
Lint-staged then checks again at commit time. **Never bypass with `--no-verify`.**

Prettier: each repo has its own `.prettierrc` (printWidth 100, 2 spaces, LF; quote and
semicolon style differ per repo; see §A11 in improvements).

### CI/CD & infrastructure (Bitbucket Pipelines + AWS)

| Repo | Pipeline |
|---|---|
| Backend | Renovate bot step · **CI:** `npm ci → lint → test → build` (node 20) · **Deploy:** Docker build (multi-stage `node:22-bookworm-slim`, openssl for Prisma) → push to **ECR** → new **ECS** task definition → one-off ECS task runs `prisma migrate deploy` (waits for it, fails the deploy if the migration fails) → service update. Separate staging and prod. |
| Web | `npm ci → build` → `aws s3 sync dist` to an **S3** bucket. `index.html` and `version.json` are uploaded with `no-cache` so clients pick up new deploys. Staging and prod buckets. |
| Mobile | **EAS** builds: Android preview APK, iOS production build, iOS submit to TestFlight (`eas-cli`, `--non-interactive --no-wait`). OTA updates via `expo-updates` (`update:production` / `update:main` scripts). |

AWS services in use: **ECS** (API + worker containers), **ECR**, **RDS PostgreSQL**,
**ElastiCache Redis** (BullMQ), **S3** (files + web hosting), **SES** (email, with a
configuration set logging to CloudWatch), **CloudWatch** logs, **IAM**.

Dependency updates: **Renovate** (`renovate.json`): base branch `staging`, patch updates
grouped and auto-merged, minor/major manual, Prisma and NestJS grouped, security alerts
auto-merged, lockfile maintenance on, a short `ignoreDeps` list for packages pinned on purpose.

### Third-party services

| Service | Used for |
|---|---|
| Stripe | Subscription billing + onboarding payment (webhooks) |
| Pusher | Realtime events (channels scoped per tenant/department) |
| CometChat | In-app chat (web + mobile) |
| Firebase | Push notifications (FCM via `firebase-admin`), GA4 analytics |
| Microsoft Clarity | Session recordings / heatmaps (web + mobile) |
| Persona | Identity verification (mobile) |
| AWS SES / S3 | Email / file storage |
| Expo EAS | Mobile builds, store submission, OTA updates |
| Bitbucket | Git hosting + CI/CD (Pipelines) |
| Trello | Ticket board (read-only via MCP for Claude) |

---

## 2. Layer ①: Context (progressive disclosure)

**Principle:** the root `CLAUDE.md` is always loaded and stays short. Deep, local detail
lives in **nested `CLAUDE.md` files next to the code they describe**, so it costs nothing
until someone works in that folder. Each rule has **exactly one canonical home**; other
files point to it ("see `src/common/CLAUDE.md` §Dates") instead of repeating it.

### 2.1 Workspace root `CLAUDE.md`: sections, in this order

1. **Title + workspace table:** path, stack, repo role, for each repo. State that the root
   is not a git repo.
2. **Product & domain primer:** what the product does, its main modules, the tenant
   hierarchy, the role model, and integrations. (Relay: `Company → Dealership → Departments
   & Teams → Employees`; access = role **and** tenant scope.)
3. **Progressive Disclosure rule (verbatim idea):** "Before editing any file, read the
   nearest `CLAUDE.md` walking up from that file (feature → repo root → this file). Never
   edit across repos without checking the API contract (use `api-contract-tracer` /
   `/sync-types`). Deploy BE before FE."
4. **Branch model:** the real PR base branch (Relay: `staging`), and which branches don't exist.
5. **Cross-cutting rules (all repos):** no `any`, no non-null `!`, no secrets, no
   `console.log`, never expose sensitive fields.
6. **Team conventions & non-obvious rules:** the short list of things that cause real bugs
   when forgotten. Relay's examples: every query filters the tenant id taken from the JWT
   (never the body); commits have no AI co-author trailers; phase large refactors with stop
   points; impersonation is resolved on the backend; realtime channel-name helpers must be
   byte-identical in BE and FE; automation never overwrites manual edits.
7. **Glossary:** domain acronyms.
8. **Tooling map:** a table of lint / format / test / build commands per repo.
9. **Tracker:** board name and id, where the snapshots live, how to refresh them.
10. **Available commands & subagents:** a one-line list.

Relay's root file is about 130 lines. **Target ≤ 150.**

### 2.2 Repo root `CLAUDE.md`: sections

**Backend** (Relay's BE file, in order): one-paragraph identity · product primer (short, or a
pointer) · working agreements · **layers & dependency rule** (ASCII diagram: domain ←
application ← adapters / infrastructure; inner layers never import outer ones) · stack ·
auth & roles (incl. a **"adding a new role touches N places"** checklist) · **multi-tenancy
(critical)** · API conventions (guard stack, `request.user` shape, response envelope
`{ data, statusCode, message?, timestamp }`, pagination, error handling) · impersonation ·
**domain map** (each domain → its nested CLAUDE.md) · queues · timezone · **known traps** ·
testing (how e2e runs, test DB, env gotchas, fixtures) · commands.

**Web** (Relay's FE file): stack · folder structure (`features/<f>/{components,hooks,
services,stores,types,index.ts}`, `common/`, `core/{store,entities,config}`, `pages/`) ·
**data-fetching pattern** (one axios instance → service file → TanStack Query hook →
component; server data never copied into Zustand; mutations invalidate every related key) ·
**stores table** (store, location, persistence, contents) · impersonation helper
(`useEffectiveUser`) · routing & product gating (FE gating is presentation; BE enforces) ·
realtime · the FE half of the "new role" checklist · hard rules (functional components, no
cross-feature imports, date handling, check narrow widths) · commands.

**Mobile:** same shape as web (navigation, data layer, push, build/EAS profiles, env rules).

### 2.3 Nested `CLAUDE.md`: where and what

Relay has 27 of them (15–150 lines each). Put one in a folder when that folder has
**non-obvious rules a newcomer would break**. Relay's placements, as a pattern:

| Placement (Relay) | Holds |
|---|---|
| `prisma/` | schema conventions, migration rules (additive/nullable, never hand-edit migrations), seeding |
| `src/adapters/` | controller/DTO/guard usage rules, sensitive-field discipline |
| `src/adapters/controllers/<hot-feature>/` | the hottest business rules for that feature's endpoints |
| `src/application/use-cases/` | layer rules + "when adding a use-case: 1…4" |
| `src/application/use-cases/<feature>/` | feature-specific invariants, documented exceptions |
| `src/infrastructure/orm/` | N+1, `$transaction` discipline, query hygiene |
| `src/infrastructure/<integration>/` | third-party sync rules, ports, event flows |
| `src/infrastructure/queues/<queue>/` | one per queue: job shape, locking, retries |
| `src/common/` and `src/common/utils/<area>/` | shared utils map, date systems, realtime parity spec |
| `test/<suite>/` | how that test harness works, how to run it, golden-file rules |
| FE `src/features/` | feature-slice anatomy, no sibling imports |
| FE `src/features/<feature>/` | e.g. query-invalidation key lists (Relay's #1 FE bug source) |
| FE `src/features/<feature>/<sub>/` | small decision tables, e.g. a who-sees-what access matrix |
| FE `src/common/` | canonical vs legacy UI primitives ("use DropdownV2, not Dropdown") |
| FE `src/core/store/` | Zustand conventions |

**What goes in a CLAUDE.md:** durable rules, conventions, traps, "touches N places"
checklists, decision tables with the date they were decided, and pointers. **What stays
out:** activity logs, restating what the code says, long backstory. Trim the story, keep
the rule.

**Writing style that worked:**
- State the rule, then why, then where (`path/file.ts`).
- Flag fail-open traps loudly (⚠️ "without X the admin bypass fires silently").
- Use numbered checklists for multi-place changes ("Adding a RoleName takes five BE places: 1…5").
- Use tables for matrices (roles × capabilities, stores, queues).

### 2.4 Size budgets (enforced)

`<repo>/scripts/check-context-size.sh`: repo root CLAUDE.md ≤ 200 lines, nested ≤ 150.
It fails over budget and warns at 90%. It runs from lint-staged when any `CLAUDE.md` is
staged (`"**/CLAUDE.md": "bash scripts/check-context-size.sh"`).

```bash
#!/usr/bin/env bash
# Keeps AI context files small enough to stay useful.
#   repo-root CLAUDE.md -> at most 200 lines; any nested CLAUDE.md -> at most 150 lines
set -euo pipefail
cd "$(git rev-parse --show-toplevel)"
status=0
while IFS= read -r file; do
  [ -f "$file" ] || continue
  limit=150; [ "$file" = "CLAUDE.md" ] && limit=200
  lines=$(wc -l < "$file" | tr -d ' ')
  if [ "$lines" -gt "$limit" ]; then
    echo "✗ $file: $lines lines (budget $limit). Move rules into a nested CLAUDE.md or trim the story, not the rule."
    status=1
  elif [ $((lines * 10)) -ge $((limit * 9)) ]; then
    echo "⚠ $file: $lines/$limit lines (near budget)"
  fi
done < <(git ls-files --cached --others --exclude-standard | grep -E '(^|/)CLAUDE\.md$')
exit "$status"
```

---

## 3. Layer ②: Workflows (slash commands in `.claude/commands/`)

Each command is a markdown file with frontmatter (`description`, `argument-hint`,
`allowed-tools`) and a numbered procedure. Heavy commands **delegate to a subagent**.
Relay's ten, mapped to HRMS:

| Command | What it does | Delegates to |
|---|---|---|
| `/plan-ticket <id\|url>` | Ticket → PM-minded cross-repo plan: objective, risks, impacted files, steps, verify commands. Plan only, no code. | `ticket-planner` |
| `/review-changes` | Reviews **uncommitted** work in all repos against the architecture rules → severity table `Repo·File·Line·Issue·Severity` | (inline) |
| `/review-pr <branch \| be:<b> fe:<b>> [ticket] [base]` | Reviews a colleague's PR against its base **and against the ticket's intent**. Handles same-name or different-name branches across repos. Asks about ticket ambiguities, then **saves the review to `.pr-reviews/`** (a re-review appends a delta). | `pr-reviewer` |
| `/sync-types [name]` | Drift check between backend response DTOs and the FE/mobile mirrored types | (inline / `api-contract-tracer`) |
| `/sync-<tracker> [scope]` | Refreshes the `.<tracker>/` snapshot files from the live board | `<tracker>-sync` |
| `/commit [repo]` | Conventional commit (`type(scope): summary`) in the right repo. Never `--no-verify`. No push unless asked. | (inline) |
| `/migrate <name>` | Safe Prisma migration (additive/nullable, tenant-scoped, never hand-edit `migrations/`), then a safety review | `prisma-guardian` |
| `/test [target] [unit\|e2e]` | Writes tests **and runs them**; reports real pass/fail | `test-author` |
| `/erd <domain>` | Domain slice of `schema.prisma` → Mermaid ERD file | (inline) |
| `/day-end [window]` | End-of-day sweep: the curator reads the day's commits (all branches) and uncommitted work, **proposes** minimal CLAUDE.md/memory edits, the user approves, then the status report is written | `context-curator` |

Example command file (Relay's `/commit`, trimmed):

```markdown
---
description: Stage and create a conventional commit in the correct repo
argument-hint: [repo: hrms-api | hrms-web | hrms-mobile]
allowed-tools: Bash(git -C *)
---
1. Pick the repo: `$1` if given, else the repo with uncommitted changes. If several, ask.
2. Review: `git -C <repo> diff` and `git -C <repo> diff --staged`.
3. Stage the intended files (confirm if ambiguous).
4. Message: `type(scope): summary`, type ∈ feat|fix|refactor|chore|test|docs|perf.
   Body explains *why* when non-trivial. Reference the ticket #.
5. Commit. If lint-staged fails, report and stop. Never bypass with --no-verify.
Do not push unless explicitly asked.
```

---

## 4. Layer ③: Specialist subagents (`.claude/agents/`)

Each agent has its own isolated context and **least-privilege tools**. Reviewers and the
curator **cannot write**. A sync agent writes only its snapshot folder. Frontmatter
carries `name`, `description` (when to use it), `tools`, `model`.

| Agent | Model | Tools | Job |
|---|---|---|---|
| `pr-reviewer` | opus | Bash, Read, Grep, Glob + **read-only** tracker tools | Evidence-based, severity-graded review. Cites file·function·line. Must-checks: tenant scoping, authz, impersonation path, sensitive-field leaks, N+1/transactions, enum exhaustiveness, timezone handling, FE coupling. |
| `ticket-planner` | opus | all | Ticket → strict cross-repo plan. Thinks like the client/PM first and flags scope creep. Plans only. |
| `api-contract-tracer` | sonnet | Read, Grep, Glob | Changed BE endpoint/DTO → every FE/mobile consumer with file:line + break risk (None/Low/High) + required updates. |
| `prisma-guardian` | sonnet | Read, Grep, Glob, Bash | Schema/migration safety + query performance (N+1, missing `$transaction`, unbounded reads). |
| `test-author` | sonnet | Read, Grep, Glob, Write, Edit, Bash | BE Jest unit (fake ports) + e2e (supertest against a dedicated test DB); FE component/hook tests. Runs them. |
| `<tracker>-sync` | sonnet | all (writes only `.<tracker>/`) | Rewrites the board snapshots. |
| `context-curator` | opus | Read, Grep, Glob, Bash | Day's diff → the **minimal** context edits as exact before → after text. **Read-only, proposes only.** Biased toward "no change needed" and willing to propose deletions. |

Example agent file (Relay's `api-contract-tracer`):

```markdown
---
name: api-contract-tracer
description: Given a changed NestJS endpoint, route, or response DTO in the backend, finds every consumer in the web and mobile apps. Use to assess cross-repo blast radius before/after a backend API change.
tools: Read, Grep, Glob
model: sonnet
---
You map the seam between the repos.
1. Identify HTTP method + path from the controller, and the response DTO shape.
2. Search web + mobile for callers: URL strings, API client functions, `*Api.ts`
   files, hooks, mirrored types in `core/entities` and feature `types/`.
3. Report each consumer with file:line, what it reads/sends, and whether the change breaks it.
Output: **Endpoint/contract** · **Consumers** table (`file:line` → usage → risk) · **Required updates**.
Read-only; never modify files.
```

**The curator's bar for changing a context file** (copy into `context-curator.md`): propose
an edit only when the day's work introduced a convention the next person must follow,
invalidated a documented rule (rename/move/delete), surfaced a trap that cost real debugging
time and will recur, or changed an architectural boundary, guard, enum or response contract.
Most days the right answer is "no context change needed." The `/day-end` command must
**stop and ask** before applying anything. These files load into every future session, so a
bad edit degrades work silently for weeks.

---

## 5. Layer ④: Rails (mechanical, not aspirational)

### 5.1 `.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(git -C * diff:*)", "Bash(git -C * log:*)", "Bash(git -C * status:*)",
      "Bash(git -C * branch:*)", "Bash(git -C * merge-base:*)", "Bash(git -C * show:*)",
      "Bash(npm run lint:*)", "Bash(npm run format:check:*)", "Bash(npm test:*)",
      "Bash(npx prettier:*)", "Bash(npx eslint:*)"
    ],
    "deny": [
      "Read(./.mcp.json)",
      "Read(./hrms-api/.env)", "Read(./hrms-web/.env)", "Read(./hrms-mobile/.env)",
      "Edit(./hrms-api/prisma/migrations/**)", "Write(./hrms-api/prisma/migrations/**)",
      "Read(./**/dist/**)", "Read(./**/coverage/**)", "Read(./**/package-lock.json)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/format-edited-file.sh" }
        ]
      }
    ]
  }
}
```

(Relay hard-codes an absolute path in the hook command. Use `$CLAUDE_PROJECT_DIR` so the
config works on any machine.)

### 5.2 `.claude/hooks/format-edited-file.sh`: Prettier in the edited file's own repo

```bash
#!/usr/bin/env bash
# PostToolUse: prettier-format the file Claude just edited, using that repo's config.
# Reads hook JSON from stdin; uses node because jq may not be installed.
root="${CLAUDE_PROJECT_DIR:-$(pwd)}"
file=$(node -e "let s='';process.stdin.on('data',d=>s+=d).on('end',()=>{try{process.stdout.write(JSON.parse(s).tool_input.file_path||'')}catch(e){}})")
case "$file" in
  *.ts|*.tsx|*.js|*.jsx|*.json|*.css)
    for repo in hrms-api hrms-web hrms-mobile; do
      if [[ "$file" == "$root/$repo/"* ]]; then
        (cd "$root/$repo" && npx prettier --write "$file" >/dev/null 2>&1)
      fi
    done ;;
esac
exit 0
```

### 5.3 In each repo

husky + lint-staged (eslint --fix + prettier on staged files, plus the context-size check
for `CLAUDE.md`). Encode critical invariants as tests where possible. Relay has a **parity
spec** that diffs the BE and FE realtime channel-name helpers, because prose alone kept
failing.

### 5.4 Workspace `.gitignore`

```
.mcp.json
.env
.env.*
*.local
.claude/settings.local.json
```

---

## 6. Compounding knowledge stores

| Store | Purpose | Rule |
|---|---|---|
| `.<tracker>/` snapshots | Board context without live MCP calls | May be stale; refresh via `/sync-<tracker>`. Tracker MCP is **read-only** for Claude; the user does all board writes. |
| `.plans/` | Phased implementation plans with **verification stop points**, phase handouts, eval baselines | Large or hot-path work is always phased; never one-shot. |
| `.pr-reviews/` | Every saved PR review | Later reviews match the house style by example. Re-reviews append a delta. |
| Memory (`~/.claude/projects/.../memory/`) | Cross-session facts: user preferences (`feedback_*`), project findings (`project_*`), external pointers (`reference_*`) | One fact per file + one line in `MEMORY.md`. Update rather than duplicate. Promote durable rules into the right CLAUDE.md. |

**Maintenance loop:** `/day-end` daily keeps CLAUDE.md and memory true to the code.

---

## 7. MCP servers

| Server | Scope | Use |
|---|---|---|
| Ticket tracker (Relay: `@delorenj/mcp-server-trello`) | project `.mcp.json` | Read tickets, acceptance criteria, comments |
| `context7` | user-local | Current library docs |
| `playwright` | user-local | Drive/screenshot the running app to verify UI |

`.mcp.json.example` (committed-style, placeholders only) explains how to get tokens and
which board id to use. Restart Claude after MCP config changes; verify with `claude mcp list`.

---

## 8. Team conventions Relay bakes in everywhere (adopt unless the user says otherwise)

- PR base branch is a real integration branch (`staging`); reviewers find the base with `git merge-base`.
- **Commits and PRs: plain conventional messages. No `Co-Authored-By` / AI-attribution / "Generated with" lines.** (State this in root CLAUDE.md and in `/commit`.)
- Tenant id scopes every query, taken from the auth token and never the request body.
- Impersonation ("view as employee") is resolved on the backend; the FE sends an acting-id and never rebuilds permission filters.
- Automation never overwrites manual human edits unless explicitly told to.
- Phase large refactors with stop points; the user reviews diffs before moving on.
- Before calling a change done, list **every mutation path** that reaches the changed state (create, update, bulk, convert/fork, recurring, scope=ALL) and check UI at narrow widths.
- PR reviews always check timezone handling and the admin-viewing-another-user path.

---

## 9. Bootstrap order (for the Claude Code doing the setup)

1. Ask the user: repo names, stacks (default: same as Relay), tenant model, role list,
   tracker (and board id), PR base branch, integrations, and the first AI features.
2. Create the workspace layout (§1). Create or clone the three repos and scaffold each
   with the code structure in §1.1, the package categories, husky/lint-staged pre-commit,
   Prettier/ESLint, Renovate and CI/CD in §1.2.
3. Write the workspace root `CLAUDE.md` (§2.1, ≤ 150 lines) from the user's answers.
4. Write each repo-root `CLAUDE.md` (§2.2). Add nested ones only where real rules exist
   (§2.3). Empty placeholder files add nothing; create them as rules emerge.
5. Add `.claude/settings.json`, the hook, `.gitignore`, `.mcp.json.example` (§5, §7).
6. Add the agents (§4) and commands (§3), swapping Relay names for HRMS ones.
7. Add `check-context-size.sh` + lint-staged in **each** repo (§2.4).
8. Write `.claude/SETUP.md` (a tree diagram of all of the above) and `CHEATSHEET.md`.
9. Show the user the result and ask before committing anything.
