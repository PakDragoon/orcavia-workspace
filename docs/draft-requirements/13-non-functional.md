# 13 — Non-functional Requirements

## 1. Security

| ID | Requirement |
|---|---|
| SEC-001 | OWASP ASVS L2 as baseline; annual third-party penetration test before V1 GA and yearly after. |
| SEC-002 | TLS 1.2+ everywhere; HSTS; strict CSP on web. |
| SEC-003 | Encryption at rest (RDS, S3); **application-level field encryption** (envelope encryption via KMS) for CNIC/passport/Emirates ID numbers, bank accounts/IBAN, salary amounts in audit diffs, medical notes. Searchable via blind index where needed. |
| SEC-004 | Secrets in AWS Secrets Manager/SSM; never in images, repos, or `.claude/`. |
| SEC-005 | Rate limiting per IP, per user and per tenant; WAF in front of API. |
| SEC-006 | Dependency scanning (Renovate + audit in CI), SAST, container scanning; no known-critical vulns at release. |
| SEC-007 | Least-privilege IAM; separate AWS accounts for prod/staging. |
| SEC-008 | Security event logging (logins, MFA changes, permission changes, exports, impersonation) with alerting. |
| SEC-009 | Session-recording analytics (if used) in strict masking mode and excluded from sensitive routes; consent where required. |

## 2. Privacy & compliance

| ID | Requirement |
|---|---|
| PRV-001 | Data processing agreement + privacy policy per tenant; sub-processor list. |
| PRV-002 | Comply with Pakistan PECA 2016 and the Personal Data Protection Bill (when enacted), UAE PDPL [V2], GDPR principles for multinational customers. ⚠️ legal review. |
| PRV-003 | Consent management for biometrics (face, fingerprint), location tracking, optional analytics. |
| PRV-004 | Data minimization, retention schedules (AUD-005), right to access/correct. |
| PRV-005 | SOC 2 Type I target by V2, Type II by V3 ⚠️ decide; ISO 27001 optional. |

## 3. Performance & scale

| ID | Requirement |
|---|---|
| PRF-N01 | P95 API latency < 300 ms for reads, < 600 ms for writes (excluding async jobs). |
| PRF-N02 | Web LCP < 2.5 s on 4G mid-range device; mobile cold start < 3 s. |
| PRF-N03 | Payroll 500 employees ≤ 2 min, 10,000 ≤ 15 min. |
| PRF-N04 | Design target: 1,000 tenants, 500k employees total, largest tenant 20k employees. |
| PRF-N05 | All lists paginated server-side; no endpoint returns unbounded collections (HR Direct loaded the full directory into the browser). |

## 4. Availability & recovery

| ID | Requirement |
|---|---|
| AV-001 | 99.9% monthly availability (excluding announced maintenance). |
| AV-002 | Multi-AZ database; automated backups, point-in-time recovery 7–35 days; RPO ≤ 15 min, RTO ≤ 4 h. |
| AV-003 | Restore drill quarterly. |
| AV-004 | Zero-downtime deployments; additive-first DB migrations. |

## 5. Observability

Structured logs with request id + tenant id (PII redacted); metrics and traces (OpenTelemetry); error
tracking (Sentry) on API, web, mobile; queue dashboards; uptime monitoring; status page; per-tenant usage metrics.

## 6. Usability & accessibility

WCAG 2.2 AA; keyboard navigable; dark mode; responsive down to 360 px; RTL-ready; consistent design system
(documented in Storybook); empty states and onboarding hints; every destructive action confirmable and,
where possible, undoable.

## 7. Domain events (catalog sketch)

`employee.hired`, `employee.updated`, `employee.terminated`, `assignment.changed`, `compensation.changed`,
`leave.requested|approved|rejected|cancelled`, `attendance.day.computed`, `punch.recorded`,
`request.submitted|step.completed|completed`, `payroll.run.calculated|approved|finalized`, `payslip.published`,
`document.expiring`, `letter.generated`, … — emitted after commit (outbox), consumed by notifications,
realtime, cache invalidation, webhooks and AI indexing.

## 8. Engineering standards

Defined in the Claude workspace setup (CLAUDE.md files, ADRs): NestJS clean/hexagonal layers, Prisma +
PostgreSQL (RLS), BullMQ, React 19 + Vite + TanStack Router/Query + Zustand + Tailwind v4 + one component
library, Expo, zod contracts shared via OpenAPI codegen across separate repos, Vitest/Jest/Playwright,
GitHub Actions CI, AWS (ECS, RDS, ElastiCache, S3, SES, CloudFront).
