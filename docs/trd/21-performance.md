# 21. Performance Requirements

**Related TRDs**: All sections  
**Related ADRs**: _None_  
**Phase**: All Phases

---

### Measurement Methodology

All performance metrics use consistent measurement points and statistical methods:

- **Response times**: Measured at API gateway level (before response reaches client network). Report P50, P95, P99 percentiles. All targets below are P95 unless otherwise noted.
- **Uptime**: Measured via external synthetic health check (`GET /api/v1/health`) every 60 seconds from an independent monitoring endpoint. Excludes scheduled maintenance windows.
- **Error rate**: Count of 5xx responses / total responses, measured per-minute rolling window. Alert threshold: >1% sustained for 5 minutes.
- **Throughput**: Requests per second per ECS task, measured via CloudWatch metrics. Baseline capacity: 200 req/s per task.

---

### SLA Definition

| SLA Metric | Target | Details |
|------------|--------|---------|
| **Uptime** | 99.9% monthly | Allows ~43 minutes unplanned downtime per month |
| **Scheduled maintenance** | Sundays 2:00–4:00 AM ET | 48-hour advance email notice to gym owners |
| **RTO** (Recovery Time Objective) | 1 hour | Service restored within 1 hour of outage detection |
| **RPO** (Recovery Point Objective) | 1 hour | Max 1 hour of data loss (RDS automated backups every hour + transaction logs) |

**Degraded mode behavior**:
- If Redis is down → API continues without caching (slower but functional). Sessions fall back to DB-backed validation.
- If S3 is down → File uploads fail but core features (attendance, payments, notifications) continue.
- If SQS is down → Notifications queue in-memory with retry; dispatch delayed but not lost.

---

### Per-Endpoint Response Time Targets

P95 targets measured under normal load (baseline profile). All targets are per-tenant, not global aggregate.

| Category | Example Endpoints | P95 Target |
|----------|-------------------|------------|
| Auth | `POST /auth/login`, `POST /auth/refresh` | < 300ms |
| Simple Read | `GET /members/:id`, `GET /classes/:id` | < 150ms |
| Simple Write | `POST /members`, `PATCH /members/:id` | < 250ms |
| List/Search | `GET /members?search=`, `GET /attendance?date=` | < 400ms |
| Complex Read | `GET /reports/attendance-summary` | < 1500ms |
| File Operations | `POST /files/presigned-url` | < 300ms |
| Notifications | `POST /notifications/dispatch` | < 500ms |
| Kiosk Check-in | `POST /attendance/check-in` (with face match result) | < 200ms |

**Non-API targets** (retained from original spec):

| Metric | Target | Notes |
|--------|--------|-------|
| Face Recognition | < 2 seconds for 1:N matching against 500 enrolled members | On-device (iPad), no cloud latency |
| Notification Delivery | Push/SMS within 30 seconds of trigger event | Via FCM, Twilio |
| Photo Upload (10 MB) | < 5 seconds | Direct S3 upload via presigned URL |
| Report Generation (500 members) | < 30 seconds for monthly report | Pre-computed daily rollups |
| Kiosk Offline Tolerance | Up to 24 hours of offline operation with full sync on reconnect | Critical for gym reliability |

---

### Frontend Performance Targets

Applies to the three React + Vite web apps (Parent App, Admin App, Monitor App). Kiosk is native Swift — only API response time applies.

| Metric | Target | Measurement |
|--------|--------|-------------|
| Largest Contentful Paint (LCP) | < 2.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals API |
| Cumulative Layout Shift (CLS) | < 0.1 | Lighthouse CI |
| Time to Interactive (TTI) | < 3.5s | Lighthouse CI |
| Initial JS bundle (gzipped) | < 200KB per app | Build output |
| Route-level code splitting | All non-critical routes lazy loaded | Build config |
| Image optimization | WebP format, responsive srcset | Build pipeline |

---

### Database Query Performance Targets

Measured at the TypeORM query level (excludes network round-trip to application). All targets assume properly indexed tables at scale (100K total members across 500 tenants).

| Query Type | Target | Example |
|------------|--------|---------|
| Primary key lookup | < 5ms | Get member by ID |
| Indexed single-table query | < 10ms | Get attendance by `tenantId` + `date` |
| Two-table join | < 50ms | Member with active membership |
| Complex join (3+ tables) | < 100ms | Member + membership + payments |
| Aggregation (report queries) | < 500ms | Monthly attendance summary |
| Full-text search | < 200ms | Member name search |

---

### Caching Strategy

**Cache layer**: Redis (ElastiCache) — same cluster used for sessions and rate limiting (see [TRD 22](./22-infrastructure.md)).

**Pattern**: Cache-aside — check cache → miss → query DB → write to cache → return.

**Target hit rate**: >85% for cached endpoints.

| Data | TTL | Invalidation |
|------|-----|--------------|
| Gym settings/config | 1 hour | Write-through on update |
| Class schedules | 30 min | Write-through on update |
| Member profiles | 15 min | Write-through on update |
| Attendance (today) | 5 min | Write-through on check-in |
| Report data | 10 min | TTL-based only |
| Session/auth tokens | Matches JWT expiry | On logout/revoke |

---

### Capacity Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Concurrent admin users | 100 simultaneous | Per deployment |
| Concurrent parent users | 1,000 simultaneous | Per deployment |
| Concurrent kiosks | 50 per deployment | Each polling every 30s |
| Database scale | 500 gyms × avg 200 members = 100K total members | Sharding by `tenantId` if needed |

---

### Load Testing

**Tool**: k6 (open-source, scriptable, CI-compatible).

**Test profiles**:

| Profile | Concurrent Users | Duration | When |
|---------|-----------------|----------|------|
| Smoke | 5 per tenant, 3 tenants | 2 min | Every PR |
| Baseline | 50 per tenant, 10 tenants | 10 min | Pre-release |
| Peak | 200 per tenant, 10 tenants | 15 min | Monthly |
| Stress | 500 per tenant, 10 tenants | 10 min | Quarterly |

**Peak scenario**: Models morning check-in rush (8:00–9:00 AM local) — 80% kiosk check-in requests, 20% parent app queries.

**Success criteria**:
- P95 response times within per-endpoint targets above
- Zero 5xx errors under baseline load
- < 0.1% 5xx under peak load
- No memory leaks (RSS stable over test duration)

---

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| _None_ | | | | | |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| _None_ | | | | |

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
