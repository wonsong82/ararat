# ADR-007: GPS-Triggered Notifications Removed

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

An early design included GPS-triggered notifications — when a parent's phone approaches the gym location, the system would auto-notify about their child's attendance status. This was part of a "smart attendance" concept.

## Decision

Remove GPS-triggered notification feature. Not required. Adds complexity without proportional value. Kiosk-based check-in (face recognition, QR code) already provides attendance confirmation to parents.

## Alternatives Considered

1. **GPS geofence notifications**: Requires continuous location access, battery drain, privacy concerns, inconsistent reliability. Rejected.
2. **Bluetooth beacon proximity**: Requires hardware at gym entrance, adds cost and maintenance. Rejected.
3. **Rely on kiosk check-in notifications only (chosen)**: Simpler, more reliable, privacy-friendly. Accepted.

## Consequences

**Positive**:
- No continuous location tracking — better privacy posture
- No background location permission required in app
- Simpler architecture
- No geofencing accuracy issues

**Negative**:
- Parents only learn about attendance after check-in (not on approach)
- No "pre-arrival" functionality

**Affects**: [10-attendance](../10-attendance.md), [08-kiosk-app-architecture](../08-kiosk-app-architecture.md)
