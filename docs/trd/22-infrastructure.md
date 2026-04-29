# 22. Infrastructure & Deployment

**Related TRDs**: All sections  
**Related ADRs**: [ADR-011](./adr/011-nestjs-backend.md), [ADR-013](./adr/013-aws-cloud-platform.md), [ADR-014](./adr/014-github-actions-cicd.md), [ADR-016](./adr/016-backend-testing-jest.md), [ADR-017](./adr/017-frontend-testing-vitest.md), [ADR-018](./adr/018-e2e-testing-playwright.md), [ADR-019](./adr/019-api-documentation-swagger.md), [ADR-020](./adr/020-logging-nestjs-pino.md)  
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
3. **Automated Testing**: Backend unit/integration via **Jest** ([ADR-016](./adr/016-backend-testing-jest.md)), frontend unit/component via **Vitest** ([ADR-017](./adr/017-frontend-testing-vitest.md)), E2E via **Playwright** ([ADR-018](./adr/018-e2e-testing-playwright.md)) on every push.
4. **Build**: Docker image built and pushed to Amazon ECR.
5. **Staging Deployment**: Auto-deploy to staging environment.
6. **Staging Tests**: Smoke tests, performance tests.
7. **Manual Approval**: Team reviews and approves for production.
8. **Production Deployment**: Blue-green deployment (zero downtime).
9. **Monitoring**: Automated rollback if error rate spikes.

#### GitHub Actions Workflow Structure

Each workflow has four stages: **lint/test → build → deploy-staging → deploy-production**.

**Backend API (`api.yml`)**:
1. **Lint & Test**: checkout → setup Node 20 → `npm ci` → `npm run lint` → `npm run test` (Jest unit + integration) → `npm run test:e2e` (with test PostgreSQL via service container)
2. **Build & Push**: Docker multi-stage build → tag with git SHA + `latest` → push to ECR
3. **Deploy Staging**: update ECS task definition with new image tag → deploy to staging service → wait for stability → run smoke tests
4. **Deploy Production**: requires manual approval (`environment: production`) → blue-green ECS deployment → health check (`/api/v1/health`) → automatic rollback if error rate > 1% for 5 minutes

**Web Apps (`web-{app}.yml`)**:
1. **Lint & Test**: checkout → setup Node 20 → `pnpm install --frozen-lockfile` → `pnpm --filter {app} lint` → `pnpm --filter {app} typecheck` → `pnpm --filter {app} test` (Vitest)
2. **Build**: `pnpm --filter {app} build` → upload `dist/` as artifact
3. **Deploy Staging**: sync `dist/` to S3 staging bucket → invalidate CloudFront → run Lighthouse CI (fail if LCP > 3s)
4. **Deploy Production**: requires manual approval → sync to S3 production bucket → invalidate CloudFront

**Kiosk (`kiosk.yml`)**:
1. **Build & Test**: `xcodebuild test` with iOS simulator
2. **Archive**: `xcodebuild archive` → export IPA (manual App Store upload via Transporter)

### Database

**PostgreSQL** (managed via Amazon RDS):
- Multi-AZ deployment for high availability.
- Automated daily backups with 30-day retention.
- Point-in-time recovery capability.
- Read replicas for scaling read-heavy workloads.
- Row-level security (RLS) for multi-tenancy enforcement.

#### Database Migration Strategy

- **Tool**: TypeORM CLI migrations (`typeorm migration:generate`, `typeorm migration:run`)
- **Migration files**: `api/src/migrations/{timestamp}-{DescriptiveName}.ts`
- **Development workflow**:
  1. Modify entity files in `api/src/`
  2. `npx typeorm migration:generate -n DescriptiveName` — auto-generates migration from entity diff
  3. Review generated SQL (verify no destructive changes)
  4. `npx typeorm migration:run` — apply locally
  5. Commit migration file alongside entity changes
- **Production deployment**: migrations run automatically on ECS task startup (`synchronize: false`, `migrationsRun: true` in TypeORM config). The first task to start acquires a PostgreSQL advisory lock to prevent concurrent migration execution.
- **Zero-downtime rule**: migrations MUST be backward-compatible. No column renames or drops in a single step. Use multi-step pattern: (1) add new column → deploy → (2) migrate data → deploy → (3) drop old column.
- **Rollback**: each migration has `up()` and `down()` methods. `npx typeorm migration:revert` undoes the last migration. In production, rollback is a new migration (forward-only).
- **Seeding**: development seeds in `api/src/seeds/` run via `npm run seed`. Production initial setup via admin API — never via seeds.

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

### Logging — nestjs-pino ([ADR-020](./adr/020-logging-nestjs-pino.md))

- **Framework**: `nestjs-pino` — Pino logger as NestJS `LoggerService` global replacement
- **Format**: Structured JSON in all environments. `pino-pretty` for human-readable output in development only.
- **Request context**: Auto-attaches `requestId`, `tenantId`, `userId`, `method`, `url` to every log line via NestJS middleware
- **Log levels**: `debug` in development, `info` in production
- **Transport**: stdout → CloudWatch Logs agent picks up container stdout in ECS
- **Configuration**: `LoggerModule.forRoot()` in `app.module.ts`

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

### Service Configuration

| Service | Config | Staging | Production |
|---------|--------|---------|------------|
| ECS Task | CPU / Memory | 0.5 vCPU / 1 GB | 1 vCPU / 2 GB |
| ECS Service | Min / Max tasks | 1 / 2 | 2 / 10 |
| ECS Auto-Scaling | Trigger | CPU > 70% for 3 min | CPU > 70% for 3 min |
| RDS PostgreSQL | Instance type | db.t3.small | db.t3.medium |
| RDS | Storage | 20 GB gp3 | 100 GB gp3 |
| RDS | Multi-AZ | No | Yes |
| RDS | Automated backups | 7-day retention | 30-day retention |
| RDS | Read replicas | 0 | 1 (for reporting queries) |
| ElastiCache Redis | Node type | cache.t3.micro | cache.t3.small |
| ElastiCache | Cluster mode | Disabled (single node) | Disabled (primary + 1 replica) |
| ElastiCache | Max memory policy | allkeys-lru | allkeys-lru |
| SQS Queues | Visibility timeout | 30s | 30s |
| SQS | DLQ max receives | 3 | 3 |
| S3 | Storage class | Standard | Standard (lifecycle → IA after 90 days) |
| CloudFront | Price class | PriceClass_100 (NA+EU) | PriceClass_100 (NA+EU) |

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| AWS ECS Fargate | Infrastructure | Container orchestration | API | Task definitions | Running containers |
| AWS RDS PostgreSQL | Infrastructure | Database hosting | TCP/5432 | SQL queries | Query results |
| AWS ElastiCache Redis | Infrastructure | Cache + session store | TCP/6379 | Redis commands | Cached data |
| AWS S3 | Infrastructure | Object storage | HTTPS | S3 API calls | Objects |
| AWS CloudFront | Infrastructure | CDN + signed URLs | HTTPS | Distribution config | Cached content |
| AWS SQS | Infrastructure | Message queuing | HTTPS | SendMessage | Message ID |
| AWS CloudWatch | Infrastructure | Metrics + logging | HTTPS | PutMetricData, PutLogEvents | Dashboards, alerts |
| AWS ECR | Infrastructure | Container registry | HTTPS | Docker push/pull | Container images |
| GitHub Actions | External | CI/CD pipeline | HTTPS | Workflow triggers | Build + deploy results |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| _None — infrastructure layer, no application endpoints_ | | | | |


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
