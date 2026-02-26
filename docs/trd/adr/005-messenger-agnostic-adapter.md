# ADR-005: Messenger Integration — Agnostic Adapter Pattern

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

Korean-American gym owners and parents frequently use KakaoTalk for communication. However, non-Korean members may prefer WhatsApp, LINE, or standard SMS. The platform needs to support third-party messenger notifications without being locked into a single provider.

## Decision

Implement messenger integration using an agnostic adapter pattern. Define a `MessengerAdapter` interface with `send()` and `validate()` methods. Start with KakaoTalk as the first implementation. Additional adapters (WhatsApp, LINE, FB Messenger) can be added without changing the core notification system. Each gym configures their preferred messenger provider in settings. Each parent selects their preferred messenger channel.

## Alternatives Considered

1. **KakaoTalk only**: Simpler, but locks out non-Korean members. Rejected.
2. **All messengers from Day 1**: Too much integration work for MVP. Rejected.
3. **SMS/Email only, no messengers**: Misses the KakaoTalk-centric Korean communication culture. Rejected.
4. **Adapter pattern, KakaoTalk first (chosen)**: Extensible architecture with practical starting point. Accepted.

## Consequences

**Positive**:
- Adding new messenger = implementing one interface (no core changes)
- Each gym picks the messenger that fits their community
- Each parent picks their preferred channel
- Core notification logic is messenger-agnostic

**Negative**:
- Each messenger API has different capabilities/limitations
- KakaoTalk Business API requires KakaoTalk business account approval
- Multiple messenger SDKs to maintain
- Testing complexity increases with each adapter

**Affects**: [12-notifications](../12-notifications.md)
