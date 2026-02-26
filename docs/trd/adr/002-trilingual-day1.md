# ADR-002: Trilingual Support from Day 1

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

US Taekwondo gyms are primarily run by Korean-American owners (관장님) serving diverse members. The member base is typically: Korean families, Hispanic families, and English-speaking families. Communication must reach all demographics effectively.

## Decision

Support three languages from Day 1: English (default), Korean (ko-KR), and Spanish (es-ES). All UI, notifications, emails, newsletters, and system messages must be available in all three languages. i18n infrastructure is built into the platform from the start — not retrofitted.

## Alternatives Considered

1. **English only, add languages later**: Cheaper upfront, but retrofitting i18n is significantly harder than building it in from the start. Rejected.
2. **English + Korean only**: Would miss the significant Hispanic member demographic. Rejected.
3. **Dynamic language support (any language)**: Over-engineered for current needs. Rejected — 3 languages is sufficient for US market.

## Consequences

**Positive**:
- All parents receive communications in their preferred language
- Korean gym owners can operate admin in Korean
- Hispanic families feel included from day one
- i18n architecture is clean and maintainable (built-in, not bolted-on)

**Negative**:
- All notification templates need 3 variants
- Translation cost for every new feature/message
- Cultural UX patterns differ (honorifics in Korean, formal "usted" in Spanish)
- Date/time/currency formatting varies by locale

**Affects**: [06-i18n](../06-i18n.md), [12-notifications](../12-notifications.md), [13-newsletter](../13-newsletter.md)
