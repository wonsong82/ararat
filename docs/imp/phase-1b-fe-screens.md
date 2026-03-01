# Phase 1b-fe — MVP Core: Simsa, Notifications, Reporting, Admin (Frontend)

**Status**: Not Started  
**Tasks**: 11 | **Completed**: 0 | **Progress**: 0%

---

> Frontend screens for remaining MVP features: simsa, notifications, newsletters, reporting, admin tools. Depends on Phase 0-fe (app shells) and Phase 1b (backend APIs).

### 1.6 Frontend: Parent App Screens (cont.)

- [ ] **Parent App: 심사 belt test screen** — `docs/trd/12-simsa.md` — Upcoming exams list with eligibility Badge per child, registration button + multi-step form (consent, medical, fee payment), past results table with certificate download, eligibility requirements view
- [ ] **Parent App: Notifications screen** — `docs/trd/13-notifications.md` — Chronological notification list with Card per item (icon by type, title, preview, timestamp, read/unread indicator), filter Tabs (All, Attendance, Payments, Announcements), mark as read (optimistic update), mark all as read
- [ ] **Parent App: Newsletters screen** — `docs/trd/14-newsletter.md` — Newsletter list with Card per item (title, date, preview, read status), full content viewer (rendered HTML/markdown), automatic read receipt on open
- [ ] **Parent App: Settings screen** — `docs/trd/07-web-frontend-architecture.md` — Language RadioGroup (EN/KO/ES, immediate effect), notification toggles per type (Switch), notification channel preference, account info (phone read-only, email editable), save button
- [ ] **Parent App: Withdrawal flow** — `docs/trd/09-registration.md` — Multi-step wizard: select child → reason Select + comments → review (prorated refund calculation Card, data deletion notice) → confirm (checkbox + submit), cancel at any step

### 1.7 Frontend: Admin App Screens (cont.)

- [ ] **Admin App: 심사 management & detail screens** — `docs/trd/12-simsa.md` — 심사 list TanStack Table (name, date, belt levels, registered count, capacity, status Badge), schedule new exam form (date, location, belts, fee, capacity, deadline), detail screen: exam info Card, registered members table with eligibility/payment status, batch result entry form (pass/fail per member + notes), send results notifications, generate certificates, export to PDF
- [ ] **Admin App: Notifications management screen** — `docs/trd/13-notifications.md` — Tabbed layout: Send History (TanStack Table with recipient, type, channel, status, timestamp filters), Templates (list with multi-language tab editor), Alert Rules (Card per rule with condition, template, channel, Switch toggle, create/edit). Send broadcast Dialog (audience selector, channel, message composer)
- [ ] **Admin App: Reports hub screen** — `docs/trd/16-reporting.md` — 5 report types via Tabs (Attendance, Financial, Member Roster, Retention, 심사), each with filter form (date pickers, dropdowns), TanStack Table for results, Recharts visualizations (line chart attendance, bar/pie charts financial, funnel retention), table/chart view toggle, export to PDF/Excel, schedule recurring report Dialog
- [ ] **Admin App: Settings screen** — `docs/trd/17-admin-tools.md` — Tabbed sections: Gym Profile (name, address, phone, email, logo upload, hours), Staff Accounts (TanStack Table with CRUD, role assignment), Belt Configuration (ordered list with drag-and-drop reorder), Age Group Thresholds, Payment Policies (grace period, refund, discounts), Notification Settings (defaults, quiet hours), Kiosk Config (registered devices), System (timezone, language, retention)
- [ ] **Admin App: Audit log screen** — `docs/trd/17-admin-tools.md` — TanStack Table with virtual scrolling (timestamp, actor, action, entity type, entity ID, summary, IP), filters (date range, actor, action type, entity type, text search), click row to view full before/after JSON diff in Sheet, export to CSV
- [ ] **Admin App: Task management screen** — `docs/trd/17-admin-tools.md` — Open Tasks section (Card per task: priority Badge, title, description, assignee, due date, action button navigating to relevant screen), Completed Tasks archive, filter/sort bar, complete/dismiss actions, create manual task form (title, description, assignee, due date)
