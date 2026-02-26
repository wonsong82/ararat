# 17. Feature: Monitor App (TV Display)

**Related TRDs**: [08-attendance](./08-attendance.md)  
**Related ADRs**: _None_  
**Phase**: Phase 2

---

### Architecture

Web app running in fullscreen browser mode on smart TV, Chromecast, or HDMI stick.

### Data & Display

Monitor polls backend API every 30 seconds for:
1. Today's schedule (classes, times, instructors).
2. Live attendance feed (members who checked in today).
3. Announcements (rotating banner).

### Display Layout

Split-screen:
- **Left**: Today's schedule (classes, times, instructors).
- **Right**: Live attendance feed (scrolling list of checked-in members with photos).
- **Top Banner**: Rotating announcements (gym news, upcoming simsa dates, events).

### Authentication

No authentication required on the display itself. Data is read-only and scoped to tenant via:
- URL token: `https://monitor.ararat.app?token=<jwt>` (JWT with tenant_id, no user_id).
- Or device registration: admin registers display device in settings, system issues device token.

### Auto-Recovery

If connection lost:
- Display shows "Connecting..." message.
- Auto-retries every 5 seconds.
- Displays last known data while reconnecting.

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
