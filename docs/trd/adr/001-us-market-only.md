# ADR-001: US Market Only

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

The platform could target multiple markets (Korea, US, global). Korean Taekwondo gym software exists (e.g., 클래스노트, 짐매니저) but primarily serves the Korean domestic market with Korean payment gateways (KG이니시스, 네이버페이) and Korean-specific integrations (Naver, Kakao). US-based Korean gym owners have no dedicated solution.

## Decision

Target the US market exclusively. All payments via Stripe (USD), SMS via Twilio (US numbers), email via SendGrid. No Korean payment gateway integration.

## Alternatives Considered

1. **Korea + US dual market**: Would require two payment gateway integrations, different compliance regimes, and split development focus. Rejected — too broad for MVP.
2. **Global platform**: Even broader scope. Rejected — localization, payment, and compliance complexity is prohibitive.
3. **Korea-first, US later**: Existing Korean competitors are established. Rejected — US market has clearer gap.

## Consequences

**Positive**:
- Simplified payment integration (Stripe only)
- Single compliance regime (US federal + state laws)
- Clear market positioning against Korean-only competitors
- Phone number validation limited to US format

**Negative**:
- Cannot serve Korean domestic gyms
- May miss Korean gym owners who want Korea-compatible features
- Stripe fees (2.9% + $0.30) may be higher than Korean alternatives

**Affects**: [10-payments](../10-payments.md), [12-notifications](../12-notifications.md), [06-i18n](../06-i18n.md), [04-auth](../04-auth.md)
