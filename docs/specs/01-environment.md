# Spec 01 — Environment & Stack

> Status: **LOCKED** 2026-10-18, except items marked ⏸ DEFERRED.
> Checkpoint: after all feature specs are locked, re-validate this stack against every feature
> (see §10) before any setup work starts.

## 1. Repositories & naming

| Item | Decision |
|---|---|
| Repos | `orcavia` (NestJS API + worker), `orcavia-ui` (React web), `orcavia-app` (Expo mobile), `orcavia-workspace` (Claude config, specs, ADRs, plans) |
| Local layout | `D:\Projects\Orcavia\` = `orcavia-workspace` root; app repos cloned inside as gitignored siblings |
| npm scope | `@orcavia/*` |
| Git host | GitHub |
| Tracker | Jira, project key `ORC` |

## 2. Runtime & tooling

| Item | Decision |
|---|---|
| Node | Node 24 LTS, pinned via `.nvmrc` + `engines`; same version in CI and Docker |
| Package manager | pnpm |
| Lint/format | Shared `@orcavia/config` (ESLint flat config + Prettier), husky + lint-staged, conventional commits |
| Dependencies | Renovate in every repo; CodeQL + dependency audit in CI |

## 3. Backend (`orcavia`)

| Item | Decision |
|---|---|
| Framework | NestJS, **modular monolith** with clean/hexagonal layers (domain → application → adapters / infrastructure) |
| Processes | HTTP API + separate **worker** process |
| Database | PostgreSQL (newest version supported by the chosen managed host at setup time) + pgvector |
| ORM | Prisma |
| Tenant isolation | Postgres Row-Level Security; Prisma extension sets tenant per transaction |
| Validation & contracts | zod schemas → OpenAPI → generated `@orcavia/api-client` (orval), published to GitHub Packages; CI fails on contract drift |
| Auth | Built in-house: argon2, JWT access + rotating refresh tokens, TOTP MFA, phone OTP; OIDC/SAML SSO in V2 |
| Jobs & cache | Redis (Valkey-compatible) + BullMQ only |
| Realtime | Self-hosted Socket.IO gateway with Redis adapter |
| Documents | HTML templates → PDF via Gotenberg (headless Chromium) in worker; spreadsheets via exceljs (never npm `xlsx`) |
| Search | Postgres full-text + pg_trgm (no Elasticsearch in V1) |
| Dates | date-fns v4 + `@date-fns/tz` (all repos) |

## 4. Web (`orcavia-ui`)

React 19 · Vite · TypeScript · TanStack Router · TanStack Query · Zustand (UI state only) · Tailwind v4 ·
shadcn/ui (single component library) · react-hook-form + zod · i18next with RTL-ready logical CSS ·
Recharts · TanStack Table.

## 5. Mobile (`orcavia-app`)

Expo (latest SDK) · expo-router · TanStack Query · Zustand · NativeWind · react-hook-form + zod ·
expo-secure-store · EAS Build + OTA updates · Firebase Cloud Messaging.

## 6. AI & messaging

| Item | Decision |
|---|---|
| LLM | Claude via Anthropic API (Bedrock as residency option), behind one AI module/port |
| Embeddings | Voyage AI → pgvector |
| Email | AWS SES (or host-equivalent ⏸ with cloud decision) — Mailpit locally |
| SMS/OTP | Local Pakistani aggregator, shortlist to be prepared; behind a provider interface |
| Push | Firebase Cloud Messaging |
| WhatsApp | V2 |

## 7. Environments, branching, releases

| Item | Decision |
|---|---|
| Environments | local (Docker) · dev · staging · production |
| Branches | `dev` → `staging` → `main` (production); each auto-deploys to its environment |
| Work branches | From `dev`: `feat/ORC-123-…`, `fix/ORC-…`, `chore/…`; PR into `dev` |
| Merge style | Squash into `dev`; merge commits (never squash) for `dev → staging` and `staging → main` |
| Hotfix | `hotfix/ORC-…` from `main` → PR into `main` → mandatory back-merge into `staging` and `dev` |
| Protection | No direct pushes to `dev`/`staging`/`main`; PR + green CI; `main` requires owner approval; branches deleted after merge |
| Migrations | `prisma migrate deploy` before new code starts; failure stops the deploy; additive-first (expand → contract across releases) |
| Rhythm | `dev → staging` roughly weekly; `staging → main` on sign-off; release tag `vX.Y.Z` + notes from Jira keys |
| Local dev | Docker Compose: Postgres+pgvector, Redis, MinIO, Mailpit, Gotenberg; `pnpm dev` + seed (demo tenant, 2 legal entities, 50 employees) |

## 8. Quality & observability

| Item | Decision |
|---|---|
| Tests | API: Vitest unit + Supertest e2e on a real test DB · Web: Vitest + Testing Library · Mobile: Jest + RNTL · E2E: Playwright · two-timezone CI runs · cross-tenant leak test on every endpoint · AI evals in CI · all wired in sprint 1 |
| Monitoring | Sentry (API, web, mobile), OpenTelemetry traces/metrics, structured JSON logs with PII redaction, uptime monitor + status page |

## 9. Why (rationale recorded at decision time)

| Decision | Reason |
|---|---|
| Separate repos + workspace repo | User preference; workspace repo keeps Claude config/specs shared and versioned; OpenAPI client package prevents FE/BE drift |
| pnpm | Faster, stricter dependency resolution |
| Modular monolith + worker | Microservices add ops cost with no benefit at our size; modules can be extracted later |
| Prisma + RLS | Mature migrations and type safety; isolation enforced by the database, not by remembering filters (HR Direct's weakness) |
| In-house auth | Managed providers charge per active user (expensive for large field workforces) and handle employee-code + phone OTP and multi-tenant users poorly |
| Socket.IO self-hosted | No per-message cost; data stays on our infrastructure |
| shadcn/ui only | One component library; components are owned and restyleable |
| BullMQ only / exceljs only / date-fns only | One tool per job so patterns don't fork (HR Direct and Relay both suffered duplicates); npm `xlsx` has known vulnerabilities |
| Voyage AI embeddings | Claude has no embedding model; Voyage is Anthropic's recommendation; all behind one AI module so Bedrock is a config change |
| Local SMS aggregator | Cheaper and better delivery in Pakistan than Twilio |
| `dev → staging → main` | User's chosen flow; one branch per environment, auto-deploy on merge |
| Merge commits for promotions | Squashing promotions makes branches drift and causes false conflicts |
| Additive-first migrations | Rollbacks never break the database |

## 10. ⏸ Deferred

| Item | Status |
|---|---|
| Cloud provider & region (proposed: AWS `me-central-1`, ECS Fargate, RDS, ElastiCache, S3 + CloudFront, SES, Secrets Manager) | Decide at setup time |
| Infrastructure as code (proposed: AWS CDK in TypeScript) | Depends on cloud decision |
| **Stack re-validation** | After all feature specs are locked: walk every feature's needs (scale, jobs, files, realtime, AI, compliance) against this stack and amend before setup |
