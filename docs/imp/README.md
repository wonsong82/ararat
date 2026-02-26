# Implementation Plan: Ararat

**Status**: Not Started  
**Last Updated**: February 25, 2026  
**Total Tasks**: 79 | **Completed**: 0 | **Progress**: 0%

---

## Phase Summary

| Phase | File | Status | Tasks | Progress |
|-------|------|--------|-------|----------|
| 0 | [Foundation](./phase-0-foundation.md) | Not Started | 20 | 0% |
| 1 | [MVP Core](./phase-1-mvp-core.md) | Not Started | 42 | 0% |
| 2 | [Enhanced](./phase-2-enhanced.md) | Not Started | 17 | 0% |

---

## Dependency Notes

### Cross-Phase Dependencies

- **Phase 0 must complete before Phase 1** — All MVP features depend on auth, DB, API framework, and i18n
- **Phase 1 must complete before Phase 2** — Enhanced features depend on core member, attendance, payment, and notification systems

### Within Phase 1

- **§1.1 Registration before §1.2 Attendance** — Members must exist before tracking attendance
- **§1.1 Registration before §1.3 Payments** — Members must exist before billing
- **§1.3 Payments before §1.4 심사** — Exam registration includes fee payment via Stripe
- **§1.5 Notifications depends on §1.2, §1.3, §1.4** — Notification triggers come from attendance, payment, and simsa events
- **§1.6 Reporting depends on §1.2, §1.3, §1.4** — Reports aggregate attendance, financial, and simsa data
- **§1.7 Admin Tools depends on §1.5, §1.6** — Admin dashboard surfaces notifications, reports, and settings

### Within Phase 2

- **§2.1 Kiosk depends on §1.1, §1.2, §1.8** — Needs members, attendance system, and compliance infrastructure (BIPA consent)
- **§2.2 Newsletter depends on §1.5** — Uses notification dispatch engine and file storage
- **§2.3 Parent Feed depends on §1.2, §1.5** — Uses attendance data, notifications, and file storage
- **§2.4 Monitor App depends on §1.2** — Displays live attendance and schedule data

---

## How to Use This Document

1. **Start here** — check the Phase Summary table for current progress
2. **Open the current phase file** — find the next uncompleted task
3. **Before implementing** — read the referenced TRD section for full spec
4. **After implementing** — mark the task complete, update this README's progress counts, and update the TRD file's Implementation Notes