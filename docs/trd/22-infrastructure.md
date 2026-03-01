# 22. Infrastructure & Deployment

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

- **Docker**: Backend API containerized (`api/Dockerfile`). Web apps are static builds deployed to S3 — not containerized. Kiosk is a native iOS app.
- **Amazon ECS Fargate**: Container orchestration and auto-scaling. Migration to EKS when operational complexity warrants (500+ gyms, multi-region).
- **Amazon ECR**: Container image registry.

### Deployment Strategy

Each application deploys independently. A change to the Admin App does not require redeploying the backend API or any other app.

#### Per-App Deployment

| Application | Source | Build | Target | URL Pattern |
|-------------|--------|-------|--------|-------------|
| Backend API | `api/` | `docker build` → ECR image | ECS Fargate (containerized) | `api.ararat.app` |
| Parent App | `web/app/` | `pnpm --filter app build` → static files | S3 + CloudFront (SPA) | `app.ararat.app` |
| Admin App | `web/admin/` | `pnpm --filter admin build` → static files | S3 + CloudFront (SPA) | `admin.ararat.app` |
| Monitor App | `web/monitor/` | `pnpm --filter monitor build` → static files | S3 + CloudFront (SPA) | `monitor.ararat.app` |
| Kiosk App | `kiosk/` | Xcode build → IPA | TestFlight → App Store | N/A (native app) |

#### Backend API Deployment

- **Build**: Docker multi-stage build from `api/Dockerfile`
- **Registry**: Amazon ECR (private repository)
- **Orchestration**: ECS Fargate with auto-scaling
- **Strategy**: Blue-green deployment (zero downtime)
- **Health check**: `/api/v1/health` endpoint
- **Rollback**: Automatic if error rate spikes post-deploy

#### Web App Deployment (Parent, Admin, Monitor)

All three web apps follow the same deployment pattern:

- **Build**: Vite production build (`pnpm build` in each app directory within `web/`)
- **Output**: Static HTML/CSS/JS bundle in `dist/`
- **Upload**: Sync `dist/` to app-specific S3 bucket
- **CDN**: CloudFront distribution per app (separate distributions for separate domains)
- **Cache invalidation**: CloudFront invalidation on deploy (`/*`)
- **SPA routing**: S3 configured to redirect all 404s to `index.html` for client-side routing
- **Environment variables**: Baked into the build via `VITE_*` env vars (API base URL, tenant config)

#### Kiosk App Deployment

- **Build**: Xcode archive → IPA
- **Distribution**: Apple TestFlight (beta) → App Store (production)
- **Updates**: Standard iOS app update mechanism
- **MDM**: Optional enterprise MDM for managed deployment to gym iPads
- **Version management**: Backend tracks `app_version` via kiosk heartbeat; can enforce minimum version

#### CI/CD Per-App Triggers

Each app has its own CI/CD workflow triggered by changes to its directory:

| Workflow | Trigger Path | Actions |
|----------|-------------|---------|
| `api.yml` | `api/**` | Lint → Test → Build Docker → Push ECR → Deploy staging |
| `web-app.yml` | `web/app/**`, `web/packages/**` | Lint → Test → Vite build → Deploy to S3/CloudFront |
| `web-admin.yml` | `web/admin/**`, `web/packages/**` | Lint → Test → Vite build → Deploy to S3/CloudFront |
| `web-monitor.yml` | `web/monitor/**`, `web/packages/**` | Lint → Test → Vite build → Deploy to S3/CloudFront |
| `kiosk.yml` | `kiosk/**` | Build → Test → Archive (manual App Store upload) |

**Note**: Web app workflows also trigger on `web/packages/**` changes because shared packages (`ui`, `api-client`, `shared`) affect all three web apps.

### CI/CD Pipeline

1. **Source Control**: GitHub.
2. **CI/CD**: GitHub Actions ([ADR-014](./adr/014-github-actions-cicd.md)).
   > Each app has its own GitHub Actions workflow file, triggered only by changes to that app's directory (see Deployment Strategy above for trigger paths).
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
