# Phase 2 — Enhanced Features

**Status**: Not Started  
**Tasks**: 17 | **Completed**: 0 | **Progress**: 0%

---

> Advanced features that enrich the platform. Depends on Phase 1 completion.

### 2.1 Kiosk & Face Recognition — `docs/trd/09-kiosk-face-recognition.md`

- [ ] **Kiosk iPad app scaffold** — Native iOS (Swift) app, Apple Vision Framework setup, TensorFlow Lite integration
- [ ] **Face enrollment flow** — Photos from S3 → secure LAN transfer to kiosk → on-device embedding generation (128-dim) → encrypted local SQLite storage → delete source photos
- [ ] **Face recognition check-in** — Live camera capture → face detection → 1:N embedding match (cosine similarity) → liveness detection → Attendance record → parent notification
- [ ] **Multi-kiosk sync & offline mode** — mDNS discovery, enrollment sync across kiosks, offline queue with auto-sync on reconnect
- [ ] **Re-enrollment for growing children** — Age-based re-enrollment prompts (configurable frequency), parent notification, new photo upload flow

### 2.2 Newsletter System — `docs/trd/13-newsletter.md`

- [ ] **Newsletter CRUD & template editor** — WYSIWYG rich text editor, attachment uploads (S3), save as draft, schedule for future delivery
- [ ] **Audience targeting & delivery** — Filter by class/belt/age/status (evaluated at send time), multi-channel dispatch (in-app, email, SMS), per-language rendering
- [ ] **Read receipts & analytics** — Track delivery/open/read per recipient, admin analytics dashboard (delivery rate, open rate)

### 2.3 Parent Activity Feed — `docs/trd/14-parent-feed.md`

- [ ] **Training notes & observations** — Instructor posts daily training notes per class, individual observations per member, parent sees in feed
- [ ] **Photo sharing** — Instructor uploads class/individual photos, S3 storage with CDN, thumbnail generation, parent notification
- [ ] **Weekly summary & milestones** — Auto-generated weekly summary (attendance count, training notes), milestone celebrations (belt promotion, attendance streaks)
- [ ] **Reactions & comments** — Parents can like/comment on feed posts, notification to instructor

### 2.4 Monitor App — `docs/trd/17-monitor-app.md`

- [ ] **TV display web app** — Fullscreen browser app, split-screen layout (schedule + live attendance), rotating announcement banner
- [ ] **Real-time data polling** — Poll backend every 30s for schedule/attendance/announcements, auto-recovery on connection loss, device token auth

### 2.5 Advanced Reporting & Performance — `docs/trd/15-reporting.md`, `docs/trd/20-performance.md`

- [ ] **심사 report** — Pass/fail rates by belt level, historical trends, per-instructor analysis
- [ ] **Dashboard visualizations** — Chart.js/D3 graphs for attendance trends, revenue trends, membership growth, at-risk members
- [ ] **Performance optimization** — Response time targets (<200ms p95), database query optimization, Redis caching strategy, connection pooling, load testing