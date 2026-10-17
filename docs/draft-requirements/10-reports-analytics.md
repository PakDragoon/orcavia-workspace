# 10 — Reports & Analytics

| ID | Requirement | Ver |
|---|---|---|
| RPT-001 | Role-based home dashboards: Employee (balances, tasks, payslip, announcements), Manager (team today, approvals, upcoming leave/birthdays/probation ends), HR (headcount, joiners/leavers, expiries, open requests, attendance exceptions), Payroll (run status, variances), Owner (headcount, cost, attrition). Widgets configurable per user. | V1 |
| RPT-002 | Standard report library (~40) across modules: headcount, demographics (aggregated), joiners/leavers, attrition, tenure, attendance summary/late/absent, overtime, leave balances/liability, payroll register/cost by department/cost centre, statutory reports, expense spend, loans outstanding, document expiries, assets, helpdesk SLA. Filters, grouping, charts, export (XLSX/CSV/PDF), schedule by email. | V1 |
| RPT-003 | All reports and dashboards respect the viewer's permission scope and sensitive-field rules. | V1 |
| RPT-004 | Report builder: pick dataset (semantic layer of curated entities + relationships), fields, filters, grouping, pivot, chart; save/share with roles. | V2 |
| RPT-005 | **AI natural-language analytics**: "headcount by department and gender in Lahore, last 6 months" → generated query on the semantic layer (never raw SQL against tables) → chart + the query shown + "save as report". Respects permissions; refuses individual-level sensitive data the user can't access. | V2 |
| RPT-006 | People analytics: attrition trends, absence patterns (Bradford factor optional), overtime cost, diversity (opt-in, aggregated with minimum group size), span of control. | V2 |
| RPT-007 | AI monthly HR digest for leadership: key movements and anomalies in plain language with links to the underlying reports. | V2 |
| RPT-008 | Data warehouse export (scheduled to S3/BigQuery/Snowflake) and BI connector (Power BI/Looker). | V3 |
| RPT-009 | Attrition risk insights (opt-in per tenant, aggregated/team-level by default, explanation of drivers, no automatic action). | V3 |
