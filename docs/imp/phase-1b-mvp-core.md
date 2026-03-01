# Phase 1b — MVP Core: Billing (cont.), Simsa, Notifications, Reporting, Admin, Security

**Status**: Not Started  
**Tasks**: 30 | **Completed**: 0 | **Progress**: 0%

---

> Remaining MVP features: family billing, simsa, notifications, reporting, admin tools, security. Depends on Phase 1a completion.

### 1.4 Payments & Billing (continued) — `docs/trd/11-payments.md`

- [ ] **Family billing & sibling discounts** — Aggregate children's memberships into single invoice, auto-apply configurable sibling discount (percentage or fixed)
- [ ] **Invoice generation system** — `docs/trd/11-payments.md` — Multi-line-item Invoice model, InvoiceLineItem records, tax calculation per US state (configurable rate in SystemSetting), invoice numbering
- [ ] **Refunds** — Prorated/full/none refund calculation per gym policy, Stripe refund API, refund record, parent notification, discount clawback logic
- [ ] **Equipment & retail one-time charges** — `docs/trd/11-payments.md` — Admin product CRUD (uniform, belt, sparring gear), parent purchase via app, Stripe PaymentIntent, Payment/Invoice record creation, receipt notification

### 1.5 심사 Belt Promotion — `docs/trd/12-simsa.md`

- [ ] **Exam scheduling & eligibility** — Admin schedules exam (date, belt levels, fees), system calculates eligible members (attendance count + days in rank thresholds)
- [ ] **Exam registration flow** — Parent notification → consent form → fee payment (Stripe PaymentIntent) → SimsaRegistration record
- [ ] **Simsa form deadline reminders** — `docs/trd/12-simsa.md` — Auto-send reminder if forms not submitted within 3 days, configurable deadline
- [ ] **Result entry & belt promotion** — Admin enters pass/fail results, auto-update MemberBelt on pass, parent notification (congratulations for pass, encouragement for fail)
- [ ] **Certificate generation** — `docs/trd/12-simsa.md` — PDF generation from template (member name, belt level, exam date, gym name/logo, signature line), S3 storage, URL in SimsaResult, parent download

### 1.6 Notifications & Communication — `docs/trd/13-notifications.md`

- [ ] **Notification dispatch engine** — Event-driven dispatch: determine type/recipients → fetch language-aware template → render with tokens → route to channels → log delivery status
- [ ] **Push notifications (FCM)** — Firebase Cloud Messaging integration, device token management, push delivery
- [ ] **SMS notifications (Twilio)** — Twilio SMS adapter, delivery status tracking
- [ ] **Email notifications (SendGrid)** — SendGrid adapter, HTML email templates, delivery tracking
- [ ] **In-app notification center** — Store notifications in DB, read/unread status, notification list API for parent/admin apps
- [ ] **Alert rules engine** — Configurable AlertRule records (Absence, LatePayment, MembershipExpiry, SimsaDeadline), threshold evaluation, escalation to admin Task on consecutive ignores
- [ ] **Messenger adapter — KakaoTalk** — KakaoTalk Business API integration via agnostic adapter pattern (extensible to WhatsApp/LINE later)
- [ ] **Notification template management** — `docs/trd/13-notifications.md` — Admin CRUD for notification templates per type × language, preview with sample data, seed initial templates for all notification types in EN/KO/ES

### 1.7 Basic Reporting — `docs/trd/16-reporting.md`

- [ ] **Attendance report** — Filterable by date range/class/belt/age group, attendance rates, export to PDF/Excel
- [ ] **Financial report** — Revenue by category, outstanding balances, refunds, monthly totals, export to PDF/Excel
- [ ] **Member roster** — Filterable by status/belt/class, contact info, export to PDF/Excel
- [ ] **Retention report** — New enrollments vs withdrawals, net growth, churn rate by month
- [ ] **Scheduled report generation** — `docs/trd/16-reporting.md` — Admin configures recurring reports (e.g., "Monthly Financial Report every 1st"), cron job generates and emails PDF to admin

### 1.8 Admin Tools — `docs/trd/17-admin-tools.md`

- [ ] **Audit log** — Log all data mutations (actor, action, entity, old/new values, IP, timestamp), admin view with filters, CSV export
- [ ] **Task system** — CRUD tasks with assignee, due date, status (Open/InProgress/Done), auto-created tasks from escalation rules and workflows
- [ ] **Gym announcements** — Admin creates announcements, audience targeting (all/class/belt/group), channel selection, immediate or scheduled delivery
- [ ] **Gym settings management** — Admin UI for all SystemSetting fields (belt progression, thresholds, discounts, notification channels, etc.)

### 1.9 Security & Compliance — `docs/trd/20-security-compliance.md`

- [ ] **Data encryption** — Encryption at rest (database, S3), encryption in transit (TLS), sensitive field encryption (messenger API keys)
- [ ] **Data deletion workflow** — Consent revocation → soft delete → retention period → hard delete, audit trail for all deletion events
- [ ] **BIPA consent forms** — Biometric consent collection (separate from COPPA), retention policy, deletion on withdrawal; consent infrastructure for Phase 2 kiosk
- [ ] **File retention policies** — `docs/trd/19-file-storage.md` — Cron job or S3 lifecycle policies: auto-delete class photos and newsletter attachments after 1 year, delete profile photos on withdrawal, retain certificates indefinitely
