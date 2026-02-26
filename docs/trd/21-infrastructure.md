# 21. Infrastructure & Deployment

**Related TRDs**: All sections  
**Related ADRs**: [ADR-011](./adr/011-nestjs-backend.md), [ADR-013](./adr/013-aws-cloud-platform.md), [ADR-014](./adr/014-github-actions-cicd.md)  
**Phase**: MVP (Phase 1)

---

### Cloud Platform — AWS ([ADR-013](./adr/013-aws-cloud-platform.md))

- **Compute**: ECS Fargate (→ EKS migration path when platform scales to 500+ gyms)
- **Database**: RDS PostgreSQL (managed)
- **Cache**: ElastiCache Redis
- **Queue**: SQS (Simple Queue Service)
- **Storage**: S3 + CloudFront
- **Monitoring**: CloudWatch
- **Logging**: CloudWatch Logs

### Containerization

- **Docker**: All services containerized.
- **Amazon ECS Fargate**: Container orchestration and auto-scaling. Migration to EKS when operational complexity warrants (500+ gyms, multi-region).
- **Amazon ECR**: Container image registry.

### CI/CD Pipeline

1. **Source Control**: GitHub.
2. **CI/CD**: GitHub Actions ([ADR-014](./adr/014-github-actions-cicd.md)).
3. **Automated Testing**: Unit tests, integration tests, E2E tests on every push.
4. **Build**: Docker image built and pushed to Amazon ECR.
5. **Staging Deployment**: Auto-deploy to staging environment.
6. **Staging Tests**: Smoke tests, performance tests.
7. **Manual Approval**: Team reviews and approves for production.
8. **Production Deployment**: Blue-green deployment (zero downtime).
9. **Monitoring**: Automated rollback if error rate spikes.

### Database

**PostgreSQL** (managed via Amazon RDS):
- Multi-AZ deployment for high availability.
- Automated daily backups with 30-day retention.
- Point-in-time recovery capability.
- Read replicas for scaling read-heavy workloads.
- Row-level security (RLS) for multi-tenancy enforcement.

### Caching

**Redis** (managed via Amazon ElastiCache):
- Session storage (JWT tokens, refresh tokens).
- Rate limiting counters.
- Async notification queue.
- Real-time attendance cache.
- TTL: 30 days for sessions, 1 hour for rate limiting, 1 day for cache.

### Queue

**Amazon SQS**:
- Async notification dispatch (push, SMS, email, messenger).
- Batch processing (daily cron jobs, report generation).
- Retry logic with exponential backoff.
- Dead-letter queue for failed messages.

### Monitoring & Alerting

**Metrics**:
- API response time (p50, p95, p99).
- Error rate (4xx, 5xx).
- Database query latency.
- Cache hit rate.
- Queue depth.
- Kiosk heartbeat status.

**Alerts**:
- Error rate > 1%: page on-call engineer.
- API response time p95 > 500ms: alert team.
- Database CPU > 80%: alert team.
- Kiosk offline > 30 minutes: alert gym owner.

**Dashboards**:
- Real-time system health dashboard.
- Per-service metrics dashboard.
- Business metrics dashboard (MRR, churn, NPS).

### Backup & Disaster Recovery

- **Daily Automated Backups**: Full database backup every 24 hours.
- **Retention**: 30-day retention (can restore to any point in last 30 days).
- **Point-in-Time Recovery**: Restore database to any timestamp.
- **Cross-Region Replication**: Backup replicated to secondary region.
- **RTO** (Recovery Time Objective): < 1 hour.
- **RPO** (Recovery Point Objective): < 1 hour.

### Environments

| Environment | Purpose | Data | Deployment |
|-------------|---------|------|-----------|
| **Development** | Local development | Synthetic test data | Manual |
| **Staging** | Pre-production testing | Copy of production data (anonymized) | Auto on every commit |
| **Production** | Live customer data | Real customer data | Manual approval required |

### Scaling Strategy

- **Horizontal Scaling**: Auto-scaling groups for API servers (scale based on CPU/memory).
- **Database Scaling**: Read replicas for read-heavy workloads. Sharding by tenant_id if needed (500+ gyms).
- **Cache Scaling**: Redis cluster mode for distributed caching.
- **CDN Scaling**: CloudFront handles global distribution automatically.

### Cost Optimization

- **Reserved Instances**: Purchase 1-year or 3-year reserved instances for baseline capacity.
- **Spot Instances**: Use spot instances for non-critical workloads (batch jobs, testing).
- **Auto-Scaling**: Scale down during off-peak hours.
- **Data Transfer**: Minimize cross-region data transfer.
- **Storage**: Lifecycle policies to archive old data (e.g., attendance records > 1 year).

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
