# ADR-011: NestJS Backend Framework

**Status**: Accepted  
**Date**: 2026-02-25  
**Deciders**: Project owner, AI agent

## Context

Ararat requires a backend framework to serve a REST API consumed by 4 client applications (Parent App, Admin App, Kiosk, Monitor). The platform has 21 feature modules, multi-tenant data isolation, role-based access control, async notification dispatch, scheduled cron jobs (absence alerts), and integrations with Stripe, Twilio, SendGrid, FCM, and KakaoTalk.

The backend must be TypeScript-based to share types with the React frontend.

## Decision

Use **Node.js + NestJS** (TypeScript) as the backend framework.

## Alternatives Considered

- **Node.js + Express**: Most widely used, but too unstructured for a project with 21 feature modules. Would require manually building module/service/controller patterns that NestJS provides out of the box.
- **Node.js + Fastify**: Faster raw performance, but less opinionated. Better suited for microservices than a monolithic API with many interconnected feature modules.
- **Python + FastAPI**: Excellent async framework with auto-generated docs, but would introduce a second language in the stack (Python backend + TypeScript frontend), eliminating shared types. The ML/AI advantage is irrelevant since face recognition runs on-device (iPad), not server-side.

## Consequences

**Positive:**
- TypeScript end-to-end with React frontend — shared types, shared validation schemas
- Enforced module structure maps directly to Ararat's service boundaries (AuthModule, MemberModule, PaymentModule, etc.)
- Built-in Guards (RBAC), Interceptors (response envelope), Pipes (validation), and Middleware (tenant context) align with TRD requirements
- Request-scoped dependency injection simplifies multi-tenant context propagation
- Built-in scheduler (@nestjs/schedule) for cron jobs (absence alerts, report generation)
- Built-in queue support (@nestjs/bull) for async notification dispatch
- First-class Stripe, Twilio, SendGrid, FCM SDKs available in Node.js ecosystem

**Negative:**
- More boilerplate than Express/Fastify (decorators, modules, providers)
- Steeper learning curve for developers unfamiliar with Angular-style patterns
- Slightly larger bundle size due to decorator metadata

**Affects**: [01-system-architecture](../01-system-architecture.md), [05-api-design](../05-api-design.md), [21-infrastructure](../21-infrastructure.md)