# Phase 1a-fe — MVP Core: Registration, Attendance, Payments (Frontend)

**Status**: Not Started  
**Tasks**: 11 | **Completed**: 0 | **Progress**: 0%

---

> Frontend screens for core MVP features: auth, registration, attendance, and payments. Depends on Phase 0-fe (app shells) and Phase 1a (backend APIs).

### 1.4 Frontend: Parent App Core Screens

- [ ] **Parent App: Login & OTP screen** — `docs/trd/04-auth.md` — Phone number entry with US format validation, 6-digit OTP input with auto-advance focus, 60s resend countdown, language selector (EN/KO/ES), error states (invalid code, too many attempts)
- [ ] **Parent App: Children list & registration flow** — `docs/trd/09-registration.md` — Children list with Avatar, belt Badge, status indicator, "Add Child" button; multi-step registration wizard (child profile form with Zod validation, health info, emergency contact, photo upload, COPPA consent with electronic signature)
- [ ] **Parent App: Child detail screen** — `docs/trd/09-registration.md` — Tabbed layout: Profile (name, DOB, belt, photos, edit), Attendance (calendar heatmap + record table), Belt Progress (promotion timeline with certificate download), Payments (invoice list per child)
- [ ] **Parent App: Payments screen** — `docs/trd/11-payments.md` — All invoices grouped by month with status Badge (paid/pending/overdue/refunded), "Pay Now" button (Stripe Checkout or embedded form), auto-pay toggle, "Manage Payment Methods" (Stripe Customer Portal), receipt view

### 1.5 Frontend: Admin App Core Screens

- [ ] **Admin App: Login & 2FA screen** — `docs/trd/04-auth.md` — Email + password form, 2FA TOTP 6-digit input, "Forgot password" link, account lockout message after 5 failures
- [ ] **Admin App: Member list screen** — `docs/trd/09-registration.md` — TanStack Table with columns (Avatar, name, belt Badge, age group, class, status, join date, last attendance), text search + dropdown filters (status, belt, age, class), column sorting, cursor-based pagination, bulk actions (change class, send notification, export), "Add Member" button
- [ ] **Admin App: Member detail screen** — `docs/trd/09-registration.md` — Tabbed layout: Profile (personal info, parent info, COPPA consent, edit in Sheet/Dialog), Attendance (calendar heatmap + table, manual correction), Belt & 심사 (history timeline, manual promote), Payments (invoices, create invoice, issue refund), Activity Feed (notes), Audit Log (change history)
- [ ] **Admin App: Pending registrations screen** — `docs/trd/09-registration.md` — TanStack Table of pending applications (name, type, phone, email, children count, date, status Badge), approve/reject per row (reject with reason Dialog), duplicate warning banner, merge wizard for flagged duplicates
- [ ] **Admin App: Class management screen** — `docs/trd/17-admin-tools.md` — Weekly calendar grid view (default) + list view toggle, class Card in calendar cells (name, time, instructor, capacity bar), create/edit class Dialog, enrolled members view, delete with confirmation
- [ ] **Admin App: Attendance screens** — `docs/trd/10-attendance.md` — Today's live attendance (auto-refresh 30s, Avatar + name + time + method), manual check-in (member search + confirm), historical view (date picker + TanStack Table with filters), audit trail table (corrections, overrides), export
- [ ] **Admin App: Payments overview & membership plans screens** — `docs/trd/11-payments.md` — Payments: summary stat Cards (revenue, outstanding, overdue, auto-pay rate), invoice TanStack Table with filters, send reminders (individual/bulk), create invoice, issue refund Dialog, export. Plans: plan TanStack Table (name, price, cycle, active members, status), create/edit plan Dialog, archive with confirmation
