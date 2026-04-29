# 4. Authentication & Authorization

**Related TRDs**: [02-multi-tenancy](./02-multi-tenancy.md), [03-data-model](./03-data-model.md), [05-api-design](./05-api-design.md), [07-web-frontend-architecture](./07-web-frontend-architecture.md)  
**Related ADRs**: [ADR-001](./adr/001-us-market-only.md), [ADR-012](./adr/012-react-vite-frontend.md)  
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

### Frontend Authentication

This section specifies the client-side authentication UX for all four applications. For component library details, see [TRD 07 — Web Frontend Architecture](./07-web-frontend-architecture.md). For backend endpoint contracts, see the flows above.

#### Parent App — Phone OTP Frontend Flow

1. Parent navigates to `/login`.
2. Screen shows: phone number `Input` field (US +1 format), language selector (EN/KO/ES `RadioGroup`), "Send Code" `Button`.
3. Client validates phone format with Zod schema before sending.
4. On "Send Code": call `POST /api/v1/auth/otp/send` with `{ phone }`, button shows Spinner.
5. Screen transitions to OTP entry: 6 individual digit `Input` fields with auto-advance focus on each digit entry.
6. 60-second countdown timer before "Resend Code" button becomes active.
7. On "Verify": call `POST /api/v1/auth/otp/verify` with `{ phone, code }`.
8. If valid: store access token in Zustand auth store (memory only, NEVER localStorage), decode JWT for user info, redirect to `/`.
9. If invalid: inline error "Invalid code. Please try again." Up to 3 attempts, then "Too many attempts. Please request a new code." with resend button.
10. If phone not registered: server auto-creates account, frontend proceeds same as valid.

#### Admin App — Email + Password + 2FA Frontend Flow

1. Admin navigates to `/login`.
2. Screen shows: email `Input`, password `Input` (with visibility toggle), "Login" `Button`, "Forgot password" link.
3. On "Login": call `POST /api/v1/auth/login` with `{ email, password }`.
4. If credentials valid: server returns `{ requires2fa: true, tempToken }`, screen transitions to 2FA entry.
5. 2FA screen: 6-digit TOTP `Input` with auto-advance, "Verify" `Button`.
6. On "Verify": call `POST /api/v1/auth/2fa/verify` with `{ tempToken, code }`.
7. If valid: store access token, redirect to `/`.
8. If credentials invalid: "Invalid email or password." After 5 failures: "Account locked. Try again in 15 minutes."
9. If 2FA invalid: "Invalid code. Please try again."

#### Token Storage Strategy

| Token | Storage | Lifetime |
|-------|---------|----------|
| Access token (JWT) | Zustand store (in-memory only) | 1 hour (parent), 8 hours (admin) |
| Refresh token | httpOnly, Secure, SameSite=Strict cookie (set by server) | 30 days |

Access tokens NEVER written to localStorage/sessionStorage. Page refresh requires silent refresh.

#### Auto-Refresh on App Load

1. On app initialization (before rendering protected routes): call `POST /api/v1/auth/refresh` (cookie sent automatically).
2. If success: store new access token in Zustand, proceed to requested route.
3. If failure: redirect to `/login`.

#### Protected Route Guard (TanStack Router)

- `_auth.tsx` layout route checks `isAuthenticated` in Zustand auth store via `beforeLoad` hook.
- If not authenticated: `throw redirect({ to: '/login' })`.
- If token expired: attempt silent refresh before redirecting.
- Role-based protection: admin-only routes check `user.role` and redirect if insufficient.

#### Auth Zustand Store Interface

```typescript
interface AuthState {
  user: {
    id: string;
    tenantId: string;
    role: "Owner" | "Manager" | "Instructor" | "Parent" | "Member";
    language: "en" | "ko" | "es";
    name: string;
    phone?: string;
    email?: string;
  } | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  setAuth: (user: AuthState["user"], token: string) => void;
  clearAuth: () => void;
}
```

#### Logout Flow

1. Call `POST /api/v1/auth/logout` (server revokes refresh token, clears cookie).
2. Call `clearAuth()` in Zustand.
3. Call `queryClient.clear()` to purge cached data.
4. Redirect to `/login`.

#### Kiosk App — No User Auth

- Kiosk authenticates via device token (issued during device registration by admin).
- Device token stored in iOS Keychain.
- No user login screen on kiosk — it's a shared device.

#### Monitor App — Token via URL or Device Registration

Both authentication methods are supported:
- **URL token** (quick setup): `https://monitor.ararat.app?token=<jwt>` — JWT contains `tenant_id`, no `user_id`, read-only scope. Ideal for temporary or demo setups.
- **Device registration** (permanent installs): Admin registers the display device in Admin App, system issues a long-lived device token stored in `localStorage`. Ideal for fixed TV displays.
- No login screen — display auto-authenticates using whichever method was configured.
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


### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| Twilio | External | `POST /Messages.json` | POST | `{ To, From, Body }` | `{ sid, status }` |
| Redis (ElastiCache) | Infrastructure | N/A (client library) | SET/GET/DEL | Refresh tokens, rate limit counters | Token data, counter values |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/auth/otp/send` | POST | Parent App | `{ phone }` | `{ message }` |
| `/api/v1/auth/otp/verify` | POST | Parent App | `{ phone, code }` | `{ accessToken, user }` |
| `/api/v1/auth/login` | POST | Admin App | `{ email, password }` | `{ requires2fa, tempToken }` |
| `/api/v1/auth/2fa/verify` | POST | Admin App | `{ tempToken, code }` | `{ accessToken, user }` |
| `/api/v1/auth/refresh` | POST | All Apps | Cookie (httpOnly) | `{ accessToken }` |
| `/api/v1/auth/logout` | POST | All Apps | Cookie (httpOnly) | `204 No Content` |


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
