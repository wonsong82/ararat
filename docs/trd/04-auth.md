# 4. Authentication & Authorization

**Related TRDs**: [02-multi-tenancy](./02-multi-tenancy.md), [03-data-model](./03-data-model.md), [05-api-design](./05-api-design.md)  
**Related ADRs**: [ADR-001](./adr/001-us-market-only.md)  
**Phase**: MVP (Phase 1)

---

### Parent Authentication Flow

1. Parent visits app and enters phone number.
2. System validates phone format (US number).
3. System sends SMS via Twilio with 6-digit OTP code.
4. Parent enters OTP in app.
5. System validates OTP (valid for 10 minutes).
6. If OTP is correct:
   - System checks if parent account exists.
   - If new: create User record with role=Parent, status=Active.
   - Parent selects preferred language (English, Korean, Spanish).
   - System creates JWT token with payload: `{ userId, tenantId, role: "Parent", language }`.
   - Parent is logged in.
7. If OTP is incorrect or expired: show error, allow retry (max 3 attempts per OTP).

**JWT Token Structure**:
```json
{
  "userId": "uuid",
  "tenantId": "uuid",
  "role": "Parent",
  "language": "English|Korean|Spanish",
  "iat": 1234567890,
  "exp": 1234571490
}
```

**Token Expiry**: 1 hour. Refresh tokens stored in Redis with 30-day expiry.

### Admin Authentication Flow

1. Admin visits app and enters email and password.
2. System validates email format and looks up User record.
3. If user not found: show error.
4. If user found: validate password hash.
5. If password incorrect: increment failed login counter. After 5 failed attempts, lock account for 15 minutes.
6. If password correct:
   - System prompts for 2FA (TOTP).
   - Admin enters 6-digit code from authenticator app.
   - System validates TOTP (valid for 30 seconds).
7. If TOTP is correct:
   - System creates JWT token with payload: `{ userId, tenantId, role: "Owner|Manager|Instructor", language }`.
   - Admin is logged in.

**JWT Token Structure**: Same as parent, but role is Owner/Manager/Instructor.

**Token Expiry**: 8 hours. Refresh tokens stored in Redis with 30-day expiry.

### Token Refresh

- Client includes JWT in `Authorization: Bearer <token>` header.
- When token expires, client sends refresh token to `/api/v1/auth/refresh` endpoint.
- Auth Service validates refresh token (must exist in Redis and not be revoked).
- If valid: issue new JWT and refresh token.
- If invalid: return 401 Unauthorized, client must re-authenticate.

### Role-Based Access Control (RBAC)

| Resource | Owner | Manager | Instructor | Parent | Member |
|----------|-------|---------|------------|--------|--------|
| **Members** | CRUD | CRUD | Read | Read (own) | Read (own) |
| **Classes** | CRUD | CRUD | Read | Read | Read |
| **Attendance** | CRUD | CRUD | Create/Read | Read (own) | Read (own) |
| **Payments** | CRUD | CRUD | Read | Read (own) | Read (own) |
| **Simsa** | CRUD | CRUD | Read | Read | Read |
| **Notifications** | CRUD | CRUD | Create | Read | Read |
| **Reports** | Read | Read | None | None | None |
| **Settings** | CRUD | Read | None | None | None |
| **Newsletter** | CRUD | CRUD | None | Read | Read |
| **Activity Feed** | CRUD | CRUD | CRUD | Read | Read |
| **Staff Accounts** | CRUD | None | None | None | None |
| **Audit Log** | Read | None | None | None | None |

**Detailed Permissions**:

- **Owner**: Full access to all resources. Can create/edit/delete staff accounts, configure gym settings, view audit logs.
- **Manager**: Can manage members, classes, attendance, payments, simsa, notifications, newsletters. Cannot access settings or staff accounts.
- **Instructor**: Can view class roster, mark attendance, post to activity feed, view member progress. Cannot access payments or settings.
- **Parent**: Can view own child's profile, attendance, progress, payments. Can submit simsa forms, read newsletters, react to activity feed.
- **Member**: Can view own profile, attendance, progress, payments. Can read newsletters, react to activity feed.

### Permission Enforcement

Every API endpoint checks the user's role and tenant_id before processing the request:

```
1. Extract JWT from Authorization header
2. Validate JWT signature and expiry
3. Extract userId, tenantId, role from JWT
4. Check if user's role has permission for this endpoint
5. Check if the requested resource belongs to the user's tenant
6. If both checks pass: process request
7. If either check fails: return 403 Forbidden
```


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
