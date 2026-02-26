# ADR-013: AWS Cloud Platform

**Status**: Accepted  
**Date**: 2026-02-25  
**Deciders**: Project owner, AI agent

## Context

Ararat needs a cloud platform for hosting the backend API, database, cache, file storage, CDN, and supporting services. The platform must support managed PostgreSQL, Redis, S3-compatible storage, container orchestration, and CI/CD integration.

## Decision

Use **AWS** as the cloud platform with the following services:

- **Compute**: ECS Fargate (serverless containers) — migrate to EKS (Kubernetes) when platform scale demands it
- **Database**: RDS PostgreSQL (managed, Multi-AZ)
- **Cache/Queue**: ElastiCache Redis
- **Storage**: S3 + CloudFront (CDN)
- **Queue**: SQS for notification dispatch
- **Monitoring**: CloudWatch (metrics, logs, alarms)
- **Container Registry**: ECR (Elastic Container Registry)
- **Secrets**: AWS Secrets Manager

## Alternatives Considered

- **GCP**: Simpler setup with Cloud Run and Cloud SQL. Lower operational overhead for small teams. However, AWS has a broader ecosystem, more marketplace integrations (relevant for Stripe, Twilio, SendGrid), and larger talent pool. Cloud Run's cold start could affect kiosk API response times.

## Consequences

**Positive:**
- Broadest ecosystem and marketplace integrations
- ECS Fargate provides serverless containers — no cluster management overhead
- Clear migration path from ECS Fargate → EKS when scale demands Kubernetes
- Mature managed services (RDS, ElastiCache, S3) with proven reliability
- S3 + CloudFront is the industry standard for file storage + CDN
- Largest talent pool for hiring and AI-assisted development

**Negative:**
- More complex pricing model than GCP
- ECS Fargate is AWS-specific (vendor lock-in for orchestration layer — mitigated by Docker containers being portable)
- Higher operational complexity than GCP Cloud Run for initial setup

**Affects**: [01-system-architecture](../01-system-architecture.md), [18-file-storage](../18-file-storage.md), [21-infrastructure](../21-infrastructure.md)