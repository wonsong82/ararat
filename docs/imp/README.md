# Implementation Plan: Ararat

**Status**: Not Started  
**Last Updated**: February 26, 2026  
**Total Tasks**: 135 | **Completed**: 0 | **Progress**: 0%

---

## Phase Summary

| Phase | File | Status | Tasks | Progress |
|-------|------|--------|-------|----------|
| 0 | [Foundation: Backend & Infrastructure](./phase-0-foundation.md) | Not Started | 24 | 0% |
| 0-fe | [Foundation: Frontend Scaffolding](./phase-0-fe-foundation.md) | Not Started | 8 | 0% |
| 1a | [MVP Core: Registration, Attendance, Payments (Backend)](./phase-1a-mvp-core.md) | Not Started | 24 | 0% |
| 1a-fe | [MVP Core: Registration, Attendance, Payments (Frontend)](./phase-1a-fe-screens.md) | Not Started | 11 | 0% |
| 1b | [MVP Core: Simsa, Notifications, Reporting, Admin, Security (Backend)](./phase-1b-mvp-core.md) | Not Started | 30 | 0% |
| 1b-fe | [MVP Core: Simsa, Notifications, Reporting, Admin (Frontend)](./phase-1b-fe-screens.md) | Not Started | 11 | 0% |
| 2 | [Enhanced Features](./phase-2-enhanced.md) | Not Started | 27 | 0% |

---

## Dependency Notes

### Cross-Phase Dependencies

- **Phase 0 must complete before Phase 1a** — All MVP features depend on auth, DB, API framework, and i18n
- **Phase 0-fe can start in parallel with Phase 0** — Frontend scaffolding does not depend on backend infrastructure (except shared types package may reference backend DTO shapes for Zod schemas)
- **Phase 1a-fe depends on Phase 0-fe AND Phase 1a** — Frontend screens need both the app shells and the backend APIs they call
- **Phase 1a must complete before Phase 1b** — Simsa depends on payments, notifications depend on attendance/payments
- **Phase 1b-fe depends on Phase 0-fe AND Phase 1b** — Frontend screens need both the app shells and the backend APIs they call
- **Phase 1b must complete before Phase 2** — Enhanced features depend on core member, attendance, payment, and notification systems
- **Phase 2 frontend tasks depend on Phase 0-fe** — Monitor App and Phase 2 screens need app shells

### Within Phase 1a

- **§1.1 Registration before §1.2 Attendance** — Members must exist before tracking attendance
- **§1.1 Registration before §1.3 Payments** — Members must exist before billing

### Within Phase 1b

- **§1.4 Payments (cont.) before §1.5 심사** — Exam registration includes fee payment via Stripe
- **§1.6 Notifications depends on §1.2, §1.3, §1.5** — Notification triggers come from attendance, payment, and simsa events
- **§1.7 Reporting depends on §1.2, §1.3, §1.5** — Reports aggregate attendance, financial, and simsa data
- **§1.8 Admin Tools depends on §1.6, §1.7** — Admin dashboard surfaces notifications, reports, and settings

### Within Phase 2

- **§2.1 Kiosk depends on §1.1, §1.2, §1.9** — Needs members, attendance system, and compliance infrastructure (BIPA consent)
- **§2.2 Newsletter depends on §1.6** — Uses notification dispatch engine and file storage
- **§2.3 Parent Feed depends on §1.2, §1.6** — Uses attendance data, notifications, and file storage
- **§2.4 Monitor App depends on §1.2** — Displays live attendance and schedule data
- **§2.5 Admin Dashboard visualization depends on §1.2, §1.3, §1.5** — Dashboard charts need attendance, payment, and reporting data
- **§2.6 Parent Activity Feed screen depends on §2.3** — Frontend screen needs backend feed API
- **§2.7 Admin Dashboard/Newsletter/Feed screens depend on §2.2, §2.3** — Frontend screens need backend newsletter and feed APIs
- **§2.8 Monitor App screens depend on §2.4** — Frontend display needs backend monitor API

---

## How to Use This Document

1. **Start here** — check the Phase Summary table for current progress
2. **Open the current phase file** — find the next uncompleted task
3. **Before implementing** — read the referenced TRD section for full spec
4. **After implementing** — mark the task complete, update this README's progress counts, and update the TRD file's Implementation Notes