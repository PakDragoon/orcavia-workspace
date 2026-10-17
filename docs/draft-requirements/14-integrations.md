# 14 — Integrations

| Area | Integration | Ver |
|---|---|---|
| Email | AWS SES (transactional), custom sender domain per tenant (DKIM) | V1 |
| SMS / OTP | Local PK SMS gateway (⚠️ pick: e.g. Jazz/Telenor/aggregator), Twilio for international | V1 |
| WhatsApp | WhatsApp Business Platform (notifications, OTP, approvals) | V2 ⚠️ |
| Push | Firebase Cloud Messaging / APNs via Expo | V1 |
| Realtime | WebSockets (self-hosted gateway) or Pusher/Ably ⚠️ decide | V1 |
| Calendar | Google Calendar, Microsoft 365 (leave → calendar, interview scheduling) | V2 |
| Chat | Microsoft Teams & Slack apps (notifications, approvals, copilot) | V2 |
| SSO / provisioning | Entra ID, Google Workspace, Okta (OIDC/SAML/SCIM) | V2 |
| Biometric devices | ZKTeco and similar: file import [V1], push/pull API connector [V2] | V1 |
| Banks | PK bank bulk-payment file formats [V1]; bank APIs (where available) [V3]; UAE WPS agents [V2] | V1 |
| Accounting | CSV/XLSX journal export [V1]; QuickBooks, Xero, Odoo, SAP B1, MS Dynamics BC [V2–V3] | V1 |
| Job boards | LinkedIn, Rozee.pk, Indeed, Bayt | V2 |
| E-signature | Internal [V1]; DocuSign/Adobe Sign [V3] | V1 |
| Payments (billing) | Local PK gateway (⚠️ pick), Stripe international | V1/V2 |
| Storage export | S3/BigQuery/Snowflake warehouse sync | V3 |
| Public API | REST (OpenAPI documented), OAuth2 client credentials, per-tenant API keys with scopes, webhooks for domain events with signing + retries | V2 |
| Marketplace | Partner integrations directory | V3 |
