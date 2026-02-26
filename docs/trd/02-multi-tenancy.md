# 2. Multi-Tenancy & Gym Configuration

**Related TRDs**: [01-system-architecture](./01-system-architecture.md), [03-data-model](./03-data-model.md), [04-auth](./04-auth.md)  
**Related ADRs**: [ADR-003](./adr/003-multi-tenant-architecture.md)  
**Phase**: MVP (Phase 1)

---

### Data Isolation Strategy

Ararat uses **application-level multi-tenancy** with tenant data isolation:

- Every table in the database includes a `tenant_id` column (foreign key to the `Tenant` table).
- All queries filter by `tenant_id` at the application layer. No query should ever return data from multiple tenants.
- Row-level security (RLS) policies in PostgreSQL provide an additional safety layer: each database role is scoped to a specific tenant.
- Tenant context is established during authentication and propagated through the request lifecycle (JWT payload includes `tenantId`).

### Tenant Context Propagation

1. User logs in → Auth Service validates credentials and tenant membership.
2. Auth Service generates JWT with payload: `{ userId, tenantId, role, language }`.
3. Client includes JWT in `Authorization: Bearer <token>` header on all API requests.
4. API Gateway validates JWT and extracts `tenantId`.
5. All downstream services receive `tenantId` from the request context and use it to filter database queries.
6. If a request attempts to access data from a different tenant, the service returns a 403 Forbidden error.

### Per-Gym Configurable Settings

Each gym (tenant) has a `SystemSetting` record with the following configurable parameters:

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| **Gym Name** | String | Required | Official gym name (displayed in UI, emails, certificates) |
| **Gym Logo** | URL | Optional | Logo image for branding |
| **Gym Address** | String | Required | Physical address |
| **Gym Phone** | String | Required | Contact phone number |
| **Gym Email** | String | Required | Contact email |
| **Timezone** | String | America/New_York | Timezone for scheduling and reporting |
| **Currency** | String | USD | Always USD for US operations |
| **Language Default** | String | English | Default language for gym communications |
| **Belt Progression** | JSON Array | [White, Yellow, Orange, Green, Blue, Red, Brown, Black] | Ordered list of belt levels (customizable per gym) |
| **Absence Alert Thresholds** | JSON Array | [3, 7, 14] | Days of non-attendance that trigger alerts |
| **Grace Period Days** | Integer | 10 | Days after payment failure before membership suspension |
| **Refund Policy** | Enum | Prorated | Refund calculation method: Prorated, Full, None |
| **Sibling Discount** | JSON | { type: "percentage", value: 10 } | Discount for multiple children: percentage or fixed amount |
| **Simsa Fee Schedule** | JSON Object | { "White": 50, "Yellow": 50, ... } | Belt-level-specific exam fees (USD) |
| **Simsa Min Attendance** | Integer | 20 | Minimum classes attended since last promotion to be eligible |
| **Simsa Min Days in Rank** | Integer | 90 | Minimum days at current belt level to be eligible |
| **Newsletter Frequency** | Enum | Weekly | Default newsletter send frequency: Weekly, Bi-weekly, Monthly |
| **Notification Channels** | JSON Object | { push: true, sms: true, email: true, inApp: true, messenger: false } | Which channels are enabled for this gym |
| **Messenger Provider** | String | KakaoTalk | Primary third-party messenger (KakaoTalk, WhatsApp, etc.) |
| **Messenger API Key** | String (encrypted) | Null | API credentials for messenger integration |
| **Payment Terms** | Enum | Monthly | Billing cycle: Monthly, Annual |
| **Tax Rate** | Decimal | 0.0 | Sales tax rate for this gym's state (0.0 - 1.0) |
| **Dual Check Enabled** | Boolean | false | Require staff validation for kiosk check-ins |
| **Dual Check Timeout** | Integer | 5 | Minutes before unconfirmed check-in triggers alert |
| **Face Recognition Enabled** | Boolean | false | Enable face recognition on kiosk (Phase 2+) |
| **Face Similarity Threshold** | Decimal | 0.85 | Cosine similarity threshold for face matching (0.0 - 1.0) |
| **Child Re-enrollment Frequency** | Integer | 12 | Months between face re-enrollment prompts for children under 10 |
| **Age Group Thresholds** | JSON Object | { "Little Kids": 5, "Kids": 8, "Teens": 13, "Adults": 18 } | Age boundaries for automatic group assignment |
| **Class Capacity** | Integer | 20 | Default class size limit |
| **Attendance Retention Days** | Integer | 2555 | Days to retain attendance records (7 years = 2555 days) |
| **Data Deletion Retention Days** | Integer | 2555 | Days to retain deleted member data for audit purposes |

All settings are stored in a single `SystemSetting` table with `tenant_id` as the primary scoping key. Changes to settings are logged in the audit trail with actor, timestamp, and previous/new values.


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
