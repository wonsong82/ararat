# Phase 1 — MVP Core Features

**Status**: Not Started  
**Tasks**: 42 | **Completed**: 0 | **Progress**: 0%

---

> Core business features that deliver value to gym owners. Depends on Phase 0 completion.

### 1.1 Registration & Member Management — `docs/trd/07-registration.md`

- [ ] **Parent registration flow** — Phone OTP → language selection → profile creation → User record (role=Parent)
- [ ] **Child profile creation** — Parent adds child: name, DOB, gender, health info, emergency contact, profile photos; creates Member record (status=Pending)
- [ ] **Admin manual registration** — Admin creates members directly (status=Active, no approval needed), welcome notification
- [ ] **Self-registration (adult member)** — Adult OTP → profile → Member record (status=Pending) → admin approval
- [ ] **Approval workflow** — Admin dashboard: pending registrations list, approve/reject with reason, status transitions, notification to parent
- [ ] **Duplicate detection** — Match on (first+last+DOB) or phone number, flag as "Potential Duplicate", admin review/merge UI
- [ ] **COPPA consent flow** — `docs/trd/19-security-compliance.md` — Age check on DOB, trilingual consent form, electronic signature, consent record storage, revocation flow
- [ ] **Member profile management** — View/edit profile, belt history, attendance history, photo management
- [ ] **Membership withdrawal** — Withdrawal request → admin approval → data retention per policy → status=Withdrawn

### 1.2 Attendance System — `docs/trd/08-attendance.md`

- [ ] **QR code check-in** — Generate member QR codes, scan endpoint, create Attendance record with method=QR
- [ ] **Name search check-in** — Search by name with photo confirmation, create Attendance record with method=Name
- [ ] **Staff manual check-in** — Staff marks member as checked in, method=Manual
- [ ] **Dual-check (staff validation) flow** — PendingValidation status, staff confirmation dashboard, 5min timeout alert, Unconfirmed status handling
- [ ] **Attendance audit trail** — Immutable AttendanceAudit records for every check-in event (CheckIn, Validation, Unconfirmed, Sync)
- [ ] **Absence alert engine** — Daily cron job: calculate days since last attendance per member, trigger alerts at configurable thresholds (3/7/14 days), dedup via MemberAbsenceAlert table

### 1.3 Payments & Billing — `docs/trd/10-payments.md`

- [ ] **Stripe integration setup** — Stripe Customer creation, API key management, webhook endpoint with signature verification
- [ ] **Membership plans CRUD** — Admin creates/edits plans: name, price, billing cycle (monthly/annual), trial days
- [ ] **Recurring billing flow** — Stripe Subscription creation, `invoice.paid` webhook processing, Membership status updates, Payment/Invoice record creation
- [ ] **Grace period & payment failure handling** — `invoice.payment_failed` webhook, GracePeriod status, escalating reminders (Day 1 email → Day 5 SMS → Day 10 admin alert), Suspended status on expiry
- [ ] **Family billing & sibling discounts** — Aggregate children's memberships into single invoice, auto-apply configurable sibling discount (percentage or fixed)
- [ ] **Refunds** — Prorated/full/none refund calculation per gym policy, Stripe refund API, refund record, parent notification

### 1.4 심사 Belt Promotion — `docs/trd/11-simsa.md`

- [ ] **Exam scheduling & eligibility** — Admin schedules exam (date, belt levels, fees), system calculates eligible members (attendance count + days in rank thresholds)
- [ ] **Exam registration flow** — Parent notification → consent form → fee payment (Stripe PaymentIntent) → SimsaRegistration record
- [ ] **Result entry & belt promotion** — Admin enters pass/fail results, auto-update MemberBelt on pass, generate certificate, parent notification

### 1.5 Notifications & Communication — `docs/trd/12-notifications.md`

- [ ] **Notification dispatch engine** — Event-driven dispatch: determine type/recipients → fetch language-aware template → render with tokens → route to channels → log delivery status
- [ ] **Push notifications (FCM)** — Firebase Cloud Messaging integration, device token management, push delivery
- [ ] **SMS notifications (Twilio)** — Twilio SMS adapter, delivery status tracking
- [ ] **Email notifications (SendGrid)** — SendGrid adapter, HTML email templates, delivery tracking
- [ ] **In-app notification center** — Store notifications in DB, read/unread status, notification list API for parent/admin apps
- [ ] **Alert rules engine** — Configurable AlertRule records (Absence, LatePayment, MembershipExpiry, SimsaDeadline), threshold evaluation, escalation to admin Task on consecutive ignores
- [ ] **Messenger adapter — KakaoTalk** — KakaoTalk Business API integration via agnostic adapter pattern (extensible to WhatsApp/LINE later)

### 1.6 Basic Reporting — `docs/trd/15-reporting.md`

- [ ] **Attendance report** — Filterable by date range/class/belt/age group, attendance rates, export to PDF/Excel
- [ ] **Financial report** — Revenue by category, outstanding balances, refunds, monthly totals, export to PDF/Excel
- [ ] **Member roster** — Filterable by status/belt/class, contact info, export to PDF/Excel
- [ ] **Retention report** — New enrollments vs withdrawals, net growth, churn rate by month

### 1.7 Admin Tools — `docs/trd/16-admin-tools.md`

- [ ] **Audit log** — Log all data mutations (actor, action, entity, old/new values, IP, timestamp), admin view with filters, CSV export
- [ ] **Task system** — CRUD tasks with assignee, due date, status (Open/InProgress/Done), auto-created tasks from escalation rules and workflows
- [ ] **Gym announcements** — Admin creates announcements, displayed in parent app and monitor app
- [ ] **Gym settings management** — Admin UI for all SystemSetting fields (belt progression, thresholds, discounts, notification channels, etc.)

### 1.8 Security & Compliance — `docs/trd/19-security-compliance.md`

- [ ] **Data encryption** — Encryption at rest (database, S3), encryption in transit (TLS), sensitive field encryption (messenger API keys)
- [ ] **Data deletion workflow** — Consent revocation → soft delete → retention period → hard delete, audit trail for all deletion events
- [ ] **BIPA consent forms** — Biometric consent collection (separate from COPPA), retention policy, deletion on withdrawal; consent infrastructure for Phase 2 kiosk