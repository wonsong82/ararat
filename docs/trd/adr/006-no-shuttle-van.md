# ADR-006: Shuttle Van Features Removed

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

Korean competitors (e.g., 클래스노트, 짐매니저) include shuttle van management features (차량 관리) — vehicle tracking, pickup schedules, driver assignment, parent notifications for pickup/drop-off. This is common in Korean Taekwondo gyms where shuttle service is standard.

## Decision

Remove all shuttle van / vehicle management features from scope. US Taekwondo gyms generally do not offer shuttle services — parents drive their children directly.

## Alternatives Considered

1. **Include shuttle features for Korean-style gyms**: Some US Korean gyms may offer shuttles. Rejected — too niche for MVP, adds significant complexity (real-time tracking, driver management, route planning).
2. **Include as Phase 3+ optional module**: Possible future addition if demand exists. Deferred.

## Consequences

**Positive**:
- Reduced development scope and complexity
- Cleaner product focused on core gym operations
- No GPS/tracking compliance concerns for vehicle features

**Negative**:
- Korean gym owners familiar with shuttle features may miss this
- May need to add later if US market demands it

**Affects**: Overall product scope (PRD)
