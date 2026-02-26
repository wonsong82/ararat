# 5. API Design

**Related TRDs**: [04-auth](./04-auth.md), [18-file-storage](./18-file-storage.md)  
**Related ADRs**: [ADR-008](./adr/008-api-first-restful-backend.md)  
**Phase**: MVP (Phase 1)

---

### RESTful Conventions

All API endpoints follow RESTful conventions:

- **Base URL**: `https://api.ararat.app/api/v1`
- **Tenant Scoping**: `/api/v1/tenants/{tenantId}/resource`
- **Resource Naming**: Plural nouns (e.g., `/members`, `/classes`, `/payments`)
- **HTTP Methods**:
  - `GET /resource` - List resources (paginated)
  - `GET /resource/{id}` - Get single resource
  - `POST /resource` - Create resource
  - `PUT /resource/{id}` - Update resource (full replacement)
  - `PATCH /resource/{id}` - Partial update
  - `DELETE /resource/{id}` - Delete resource

### Standard Response Envelope

All API responses follow a consistent envelope format:

**Success Response (2xx)**:
```json
{
  "data": {
    "id": "uuid",
    "name": "John Doe",
    ...
  },
  "meta": {
    "timestamp": "2026-02-25T10:30:00Z",
    "version": "1.0"
  }
}
```

**List Response with Pagination**:
```json
{
  "data": [
    { "id": "uuid1", ... },
    { "id": "uuid2", ... }
  ],
  "meta": {
    "timestamp": "2026-02-25T10:30:00Z",
    "version": "1.0",
    "pagination": {
      "cursor": "eyJpZCI6InV1aWQyIn0=",
      "limit": 20,
      "has_more": true
    }
  }
}
```

**Error Response (4xx, 5xx)**:
```json
{
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Phone number is invalid",
      "field": "phone",
      "details": {
        "expected_format": "+1XXXXXXXXXX"
      }
    }
  ],
  "meta": {
    "timestamp": "2026-02-25T10:30:00Z",
    "version": "1.0"
  }
}
```

### Pagination

- **Cursor-Based**: Use opaque cursors (base64-encoded JSON with ID and sort key) instead of offset-based pagination.
- **Default Limit**: 20 items per page.
- **Max Limit**: 100 items per page.
- **Query Parameters**: `?limit=20&cursor=<base64>`
- **Response**: Includes `has_more` boolean and next cursor in `meta.pagination`.

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| UNAUTHORIZED | 401 | Missing or invalid authentication |
| FORBIDDEN | 403 | User lacks permission for this resource |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists or state conflict |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |

### Rate Limiting

- **Per-User Limit**: 1000 requests per hour.
- **Per-IP Limit**: 10,000 requests per hour.
- **Burst Limit**: 100 requests per minute.
- **Response Headers**:
  - `X-RateLimit-Limit: 1000`
  - `X-RateLimit-Remaining: 999`
  - `X-RateLimit-Reset: 1234567890`
- **Enforcement**: Implemented in API Gateway using Redis.

### Webhook Design (Stripe Events)

Ararat receives webhooks from Stripe for payment events:

**Webhook Endpoint**: `POST /api/v1/webhooks/stripe`

**Signature Verification**:
- Stripe sends `Stripe-Signature` header with timestamp and signature.
- System verifies signature using Stripe webhook secret.
- Reject requests older than 5 minutes.

**Supported Events**:
- `invoice.paid` - Recurring billing payment succeeded
- `invoice.payment_failed` - Recurring billing payment failed
- `charge.refunded` - Refund processed
- `customer.subscription.deleted` - Subscription cancelled

**Processing**:
1. Verify webhook signature.
2. Extract event data.
3. Update database state (payment status, membership status, etc.).
4. Trigger notifications (payment receipt, payment failure alert, etc.).
5. Return 200 OK to Stripe.

### File Upload Endpoints

**Upload Endpoint**: `POST /api/v1/tenants/{tenantId}/files/upload`

**Request**:
```
Content-Type: multipart/form-data
file: <binary file data>
type: "profile_photo|class_photo|newsletter_attachment|certificate"
```

**Response**:
```json
{
  "data": {
    "file_id": "uuid",
    "url": "https://cdn.ararat.app/tenants/{tenantId}/files/{fileId}",
    "size": 1024000,
    "mime_type": "image/jpeg"
  }
}
```

**Size Limits**:
- Profile photos: 10 MB
- Class photos: 10 MB
- Newsletter attachments: 50 MB
- Certificates: 10 MB

**Allowed Types**:
- Images: JPEG, PNG, WebP
- Documents: PDF
- Videos: MP4, MOV (future)


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
