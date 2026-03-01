# Phase 2 — Enhanced Features

**Status**: Not Started  
**Tasks**: 27 | **Completed**: 0 | **Progress**: 0%

---

> Advanced features that enrich the platform. Depends on Phase 1b completion.

### 2.1 Kiosk & Face Recognition — `docs/trd/08-kiosk-app-architecture.md`

- [ ] **Kiosk iPad app scaffold** — Native iOS (Swift) app, Apple Vision Framework setup, TensorFlow Lite integration
- [ ] **Face enrollment flow** — Photos from S3 → secure LAN transfer to kiosk → on-device embedding generation (128-dim) → encrypted local SQLite storage → delete source photos
- [ ] **Face recognition check-in** — Live camera capture → face detection → 1:N embedding match (cosine similarity) → liveness detection → Attendance record → parent notification
- [ ] **Multi-kiosk sync & offline mode** — mDNS discovery, enrollment sync across kiosks, offline queue with auto-sync on reconnect
- [ ] **Re-enrollment for growing children** — Age-based re-enrollment prompts (configurable frequency), parent notification, new photo upload flow
- [ ] **Kiosk liveness detection** — `docs/trd/08-kiosk-app-architecture.md` — Passive liveness detection (no user action required) to prevent photo/video spoofing, integrated into face recognition check-in flow
- [ ] **Kiosk management backend** — `docs/trd/08-kiosk-app-architecture.md` — Kiosk table (device_id, name, location, status, last_heartbeat), device registration, heartbeat monitoring (every 5min), offline alert after 30min, admin remote config (threshold, re-enrollment frequency)

### 2.2 Newsletter System — `docs/trd/14-newsletter.md`

- [ ] **Newsletter CRUD & template editor** — WYSIWYG rich text editor, attachment uploads (S3), save as draft, schedule for future delivery
- [ ] **Audience targeting & delivery** — Filter by class/belt/age/status (evaluated at send time), multi-channel dispatch (in-app, email, SMS), per-language rendering
- [ ] **Read receipts & analytics** — Track delivery/open/read per recipient, admin analytics dashboard (delivery rate, open rate)
- [ ] **Newsletter scheduled publishing** — `docs/trd/14-newsletter.md` — Cron job runs every minute, checks for newsletters where scheduled_at <= now AND status=Scheduled, executes delivery flow

### 2.3 Parent Activity Feed — `docs/trd/15-parent-feed.md`

- [ ] **Training notes & observations** — Instructor posts daily training notes per class, individual observations per member, parent sees in feed
- [ ] **Photo sharing** — Instructor uploads class/individual photos, S3 storage with CDN, thumbnail generation, parent notification
- [ ] **Weekly summary & milestones** — Auto-generated weekly summary (attendance count, training notes), milestone celebrations (belt promotion, attendance streaks)
- [ ] **Reactions & comments** — Parents can like/comment on feed posts, notification to instructor

### 2.4 Monitor App — `docs/trd/18-monitor-app.md`

- [ ] **TV display web app** — Fullscreen browser app, split-screen layout (schedule + live attendance), rotating announcement banner
- [ ] **Real-time data polling** — Poll backend every 30s for schedule/attendance/announcements, auto-recovery on connection loss
- [ ] **Monitor app authentication** — `docs/trd/18-monitor-app.md` — URL token (`?token=<jwt>` with tenant_id, no user_id) or device registration with admin-issued device token, read-only tenant-scoped data access

### 2.5 Advanced Reporting & Performance — `docs/trd/16-reporting.md`, `docs/trd/21-performance.md`

- [ ] **심사 report** — Pass/fail rates by belt level, historical trends, per-instructor analysis
- [ ] **Dashboard visualizations** — `docs/trd/16-reporting.md` — Recharts graphs for attendance trends, revenue trends, membership growth, at-risk members
- [ ] **Performance optimization** — Response time targets (<200ms p95), database query optimization, Redis caching strategy, connection pooling, load testing

### 2.6 Frontend: Parent App Phase 2 Screens

- [ ] **Parent App: Activity feed dashboard** — `docs/trd/15-parent-feed.md` — Chronological feed of child's activities (Card per entry: instructor avatar, timestamp, text note, photo gallery, reactions), child selector Tabs (if multiple children), pull-to-refresh, photo fullscreen viewer, react to posts (heart/thumbs-up), EmptyState ("No activity yet today")

### 2.7 Frontend: Admin App Phase 2 Screens

- [ ] **Admin App: Dashboard screen** — `docs/trd/16-reporting.md` — Dashboard grid layout: Today's Stats Card (check-ins, active members, pending registrations, overdue payments), Attendance line chart (Recharts, 2 weeks), Revenue bar chart (Recharts, 6 months), Member Distribution pie chart (by belt level), Pending Tasks list Card, Recent Activity feed Card
- [ ] **Admin App: Newsletter editor screen** — `docs/trd/14-newsletter.md` — Newsletter list TanStack Table (title, date, audience, read rate, status Badge), WYSIWYG rich text editor with formatting/images/links, multi-language tab switcher (EN/KO/ES), audience selector (all members/by class/belt/age group), preview mode, send/schedule, analytics Card (sent count, read count, read rate), duplicate newsletter
- [ ] **Admin App: Activity feed management screen** — `docs/trd/15-parent-feed.md` — Post list (reverse-chronological Cards with author, class, date, preview, photo thumbnails, reaction count), create post form (class selector, text area, photo uploader with drag-and-drop, member tag selector), edit/delete posts, photo gallery lightbox, view reactions and comments

### 2.8 Frontend: Monitor App Screens

- [ ] **Monitor App: Display screen** — `docs/trd/18-monitor-app.md` — Fullscreen split-screen layout: left panel (60% — today's schedule with current class highlighted, "Up Next" marker), right panel (40% — live attendance feed with Avatar + name + time, new entries animate in), top banner (rotating announcements every 10s — 심사 dates, closures, birthdays, belt promotions), bottom bar (date/time, gym name, connection status Badge). All data polls every 30 seconds, auto-recovery on disconnect
- [ ] **Monitor App: Setup/pairing screen** — `docs/trd/18-monitor-app.md` — 6-character pairing code displayed large, admin enters code in Admin App to link device to tenant, poll for registration status, auto-transition to display screen once paired, "Generate New Code" button, loading/success/error feedback
