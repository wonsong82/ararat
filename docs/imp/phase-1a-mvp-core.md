# Phase 1a — MVP Core: Registration, Attendance, Payments

**Status**: Not Started  
**Tasks**: 24 | **Completed**: 0 | **Progress**: 0%

---

> Core member-facing features: registration, attendance, and payments. Depends on Phase 0 completion.

### 1.1 Registration & Member Management — `docs/trd/09-registration.md`

- [ ] **Parent registration flow** — Phone OTP → language selection → profile creation → User record (role=Parent)
- [ ] **Child profile creation** — Parent adds child: name, DOB, gender, health info, emergency contact, profile photos; creates Member record (status=Pending)
- [ ] **Admin manual registration** — Admin creates members directly (status=Active, no approval needed), welcome notification
- [ ] **Self-registration (adult member)** — Adult OTP → profile → Member record (status=Pending) → admin approval
- [ ] **Approval workflow** — Admin dashboard: pending registrations list, approve/reject with reason, status transitions, notification to parent
- [ ] **Duplicate detection** — Match on (first+last+DOB) or phone number, flag as "Potential Duplicate", admin review/merge UI
- [ ] **COPPA consent flow** — `docs/trd/20-security-compliance.md` — Age check on DOB, trilingual consent form, electronic signature, consent record storage, revocation flow
- [ ] **Member profile management** — View/edit profile, belt history, attendance history, photo management, change logging in AuditLog
- [ ] **Photo management** — `docs/trd/09-registration.md` — Upload/validate profile photos (JPEG/PNG/WebP, 10MB limit), thumbnail generation (200x200), S3 storage with tenant-scoped paths, up to 5 photos per member
- [ ] **Membership withdrawal** — Withdrawal request → admin approval → prorated refund calculation → data retention per policy → status=Withdrawn
- [ ] **Member levels & grouping** — `docs/trd/09-registration.md` — Belt level tracking (per-gym configurable progression), age group auto-assignment based on DOB, class group manual assignment, AuditLog for all changes
- [ ] **Member auto-upgrade** — `docs/trd/09-registration.md` — Daily cron: recalculate age groups on birthday, create admin task when age threshold crossed, auto-update belt on simsa pass, congratulations notification
- [ ] **Annual re-registration** — `docs/trd/09-registration.md` — Daily cron: detect anniversary dates, send reminder notification N days before (configurable, default 30), re-registration form, AuditLog event

### 1.2 Attendance System — `docs/trd/10-attendance.md`

- [ ] **QR code check-in** — Generate member QR codes, scan endpoint, create Attendance record with method=QR
- [ ] **Name search check-in** — Search by name with photo confirmation, create Attendance record with method=Name
- [ ] **Staff manual check-in** — Staff marks member as checked in, method=Manual
- [ ] **Dual-check (staff validation) flow** — PendingValidation status, staff confirmation dashboard, 5min timeout alert, Unconfirmed status handling
- [ ] **Attendance audit trail** — Immutable AttendanceAudit records for every check-in event (CheckIn, Validation, Unconfirmed, Sync)
- [ ] **Absence alert engine** — Daily cron job: calculate days since last attendance per member, trigger alerts at configurable thresholds (3/7/14 days), dedup via MemberAbsenceAlert table
- [ ] **At-risk members dashboard** — `docs/trd/10-attendance.md` — Admin view: members ranked by days absent, filter by threshold level (3/7/14 days), "Send Personal Message" quick action

### 1.3 Payments & Billing — `docs/trd/11-payments.md`

- [ ] **Stripe integration setup** — Stripe Customer creation, API key management, webhook endpoint with signature verification
- [ ] **Membership plans CRUD** — Admin creates/edits plans: name, price, billing cycle (monthly/annual), trial days
- [ ] **Recurring billing flow** — Stripe Subscription creation, `invoice.paid` webhook processing, Membership status updates, Payment/Invoice record creation
- [ ] **Grace period & payment failure handling** — `invoice.payment_failed` webhook, GracePeriod status, escalating reminders (Day 1 email → Day 5 SMS → Day 10 admin alert), Suspended status on expiry, Collections state (30+ days overdue)
