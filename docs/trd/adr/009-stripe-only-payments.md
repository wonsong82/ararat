# ADR-009: Stripe-Only Payments

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

US market means USD payments only. Need to support recurring billing (monthly memberships), one-time charges (심사 fees, equipment), refunds, and family billing (multiple children on one parent's card). Korean payment gateways (KG이니시스, 네이버페이, 카카오페이) are not needed for US operations.

## Decision

Use Stripe as the sole payment processor. Single Stripe account with metadata-based routing (tenant_id in Stripe metadata) rather than Stripe Connect with separate accounts per gym. Supports credit card, ACH, Apple Pay, and Google Pay.

## Alternatives Considered

1. **Stripe Connect (separate accounts per gym)**: Each gym has their own Stripe account. More complex onboarding, each gym handles own payouts. Rejected for MVP — simpler to use single account.
2. **Square**: Good for POS but weaker on recurring billing and API-first design. Rejected.
3. **PayPal**: Less developer-friendly API, higher fees for recurring. Rejected.
4. **Multiple processors**: Support Stripe + Square + PayPal. Over-engineered for MVP. Rejected.

## Consequences

**Positive**:
- Single integration point for all payment types
- Excellent developer documentation and SDK
- Built-in support for subscriptions, invoices, webhooks
- Apple Pay + Google Pay out of the box
- PCI DSS compliance handled by Stripe

**Negative**:
- Stripe fees: 2.9% + $0.30 per transaction
- Vendor lock-in (switching payment processor is non-trivial)
- Single Stripe account means platform handles payout distribution to gyms
- No Korean payment methods (not needed for US market)

**Affects**: [10-payments](../10-payments.md)
