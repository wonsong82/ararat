# 7. Web Frontend Architecture

**Related TRDs**: [01-system-architecture](./01-system-architecture.md), [04-auth](./04-auth.md), [05-api-design](./05-api-design.md), [06-i18n](./06-i18n.md)  
**Related ADRs**: [ADR-012](./adr/012-react-vite-frontend.md), [ADR-015](./adr/015-frontend-library-stack.md)  
**Phase**: MVP (Phase 1)

---

This document specifies the cross-cutting frontend architecture shared by all three web applications (Parent App, Admin App, Monitor App). It covers monorepo structure, shared packages, routing, authentication, state management, forms, responsive design, and error handling.

Screen inventories and feature-specific UX flows are defined in the respective feature TRDs (09–18). The Kiosk App (native iOS/Swift) is out of scope — see [08-kiosk-app-architecture](./08-kiosk-app-architecture.md).

All technology choices reference [ADR-012](./adr/012-react-vite-frontend.md) (React + Vite) and [ADR-015](./adr/015-frontend-library-stack.md) (library stack). Client-side internationalization details (namespaces, file structure, language detection, date/currency formatting) are specified in [06-i18n](./06-i18n.md).

---

## 1. Monorepo Frontend Structure

All three web applications and shared packages live in a single monorepo managed with **pnpm workspaces**. Each app is an independent Vite build target; shared packages are consumed as internal workspace dependencies.

```
ararat/
├── apps/
│   ├── parent/                        # Parent/Member App
│   │   ├── src/
│   │   │   ├── routes/                # TanStack Router file-based routes
│   │   │   ├── components/            # App-specific components
│   │   │   ├── stores/                # Zustand stores (auth, ui)
│   │   │   ├── hooks/                 # App-specific hooks
│   │   │   ├── lib/                   # Utilities specific to parent app
│   │   │   ├── main.tsx               # Entry point
│   │   │   └── routeTree.gen.ts       # Auto-generated route tree
│   │   ├── public/
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   ├── tailwind.config.ts         # Extends shared preset
│   │   └── tsconfig.json              # Extends root tsconfig
│   ├── admin/                         # Admin App (same structure)
│   └── monitor/                       # Monitor App (same structure)
├── packages/
│   ├── ui/                            # Shared component library
│   │   ├── src/
│   │   │   ├── components/            # shadcn/ui + custom components
│   │   │   ├── hooks/                 # Shared UI hooks
│   │   │   ├── lib/                   # cn() utility, theme helpers
│   │   │   └── index.ts               # Named re-exports
│   │   ├── tailwind.config.ts         # Base Tailwind preset (design tokens)
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── api-client/                    # Shared API client + TanStack Query hooks
│   │   ├── src/
│   │   │   ├── client.ts              # Typed fetch wrapper (ky)
│   │   │   ├── hooks/                 # Query/mutation hooks by domain
│   │   │   │   ├── members.ts
│   │   │   │   ├── attendance.ts
│   │   │   │   ├── payments.ts
│   │   │   │   ├── simsa.ts
│   │   │   │   ├── notifications.ts
│   │   │   │   ├── newsletters.ts
│   │   │   │   ├── feed.ts
│   │   │   │   ├── classes.ts
│   │   │   │   ├── reports.ts
│   │   │   │   ├── auth.ts
│   │   │   │   ├── settings.ts
│   │   │   │   └── files.ts
│   │   │   ├── types/                 # API response/request types
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── shared/                        # Shared types, schemas, constants, i18n
│       ├── src/
│       │   ├── schemas/               # Zod validation schemas (shared with backend)
│       │   │   ├── member.ts
│       │   │   ├── attendance.ts
│       │   │   ├── payment.ts
│       │   │   ├── simsa.ts
│       │   │   ├── auth.ts
│       │   │   ├── notification.ts
│       │   │   ├── newsletter.ts
│       │   │   ├── class.ts
│       │   │   └── settings.ts
│       │   ├── types/                 # TypeScript types & interfaces
│       │   ├── constants/             # Enums, belt levels, roles, error codes
│       │   ├── i18n/                  # Translation files (en/, ko/, es/)
│       │   └── index.ts
│       ├── package.json
│       └── tsconfig.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── .eslintrc.cjs
└── .prettierrc
```

### Workspace Configuration

`pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

### Tailwind Preset Extension

Each app's `tailwind.config.ts` extends the shared preset from `packages/ui/`:

```typescript
import { sharedPreset } from "@ararat/ui/tailwind.config";

export const config = {
  presets: [sharedPreset],
  content: [
    "./src/**/*.{ts,tsx}",
    "../../packages/ui/src/**/*.{ts,tsx}",
  ],
};
```

### TypeScript Config Extension

Each app's `tsconfig.json` extends the root `tsconfig.base.json` and adds path aliases:

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "paths": {
      "@ararat/ui": ["../../packages/ui/src"],
      "@ararat/api-client": ["../../packages/api-client/src"],
      "@ararat/shared": ["../../packages/shared/src"],
      "~/*": ["./src/*"]
    }
  }
}
```

---

## 2. Shared Component Library (`packages/ui/`)

The shared UI package provides a consistent design language across all three web apps. It is built on **shadcn/ui** (copy-paste components backed by Radix primitives) styled with **Tailwind CSS v4+**.

### Design Tokens

Design tokens are defined as CSS custom properties in a global stylesheet and mapped into the Tailwind theme. This enables consistent theming across all apps and potential future dark mode.

**Color Tokens** (semantic, not raw):

| Token | Purpose |
|-------|---------|
| `--color-primary` | Primary brand action (buttons, links) |
| `--color-primary-foreground` | Text on primary background |
| `--color-secondary` | Secondary actions |
| `--color-destructive` | Delete, cancel, error states |
| `--color-muted` | Disabled, placeholder |
| `--color-accent` | Highlight, badge backgrounds |
| `--color-background` | Page background |
| `--color-foreground` | Default text |
| `--color-card` | Card backgrounds |
| `--color-border` | Borders and dividers |
| `--color-ring` | Focus rings |

**Spacing Scale**: Tailwind default (4px base: 0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 56, 64).

**Typography**: System font stack with Korean and Spanish glyph support:

```css
--font-sans: "Inter", "Pretendard", "Apple SD Gothic Neo", system-ui, sans-serif;
--font-mono: "JetBrains Mono", "Fira Code", ui-monospace, monospace;
```

**Border Radius**: `--radius: 0.5rem` (consistent rounded corners on all components).

### Component Inventory

All components are **named exports** from `@ararat/ui`.

| Component | Radix Primitive | Description |
|-----------|----------------|-------------|
| `Button` | — | Primary, secondary, destructive, outline, ghost variants |
| `Input` | — | Text input with label, error state, optional icon |
| `Textarea` | — | Multi-line text input |
| `Select` | `@radix-ui/react-select` | Single-value dropdown |
| `Combobox` | `@radix-ui/react-popover` + search | Searchable dropdown for member/class selectors |
| `Checkbox` | `@radix-ui/react-checkbox` | Boolean toggle with label |
| `RadioGroup` | `@radix-ui/react-radio-group` | Single selection from options |
| `Switch` | `@radix-ui/react-switch` | On/off toggle (settings) |
| `Dialog` | `@radix-ui/react-dialog` | Modal dialog (confirmations, forms) |
| `Sheet` | `@radix-ui/react-dialog` | Slide-in panel (mobile nav, detail views) |
| `Popover` | `@radix-ui/react-popover` | Floating content (filters, quick actions) |
| `DropdownMenu` | `@radix-ui/react-dropdown-menu` | Context menus, action menus |
| `Table` | — | Styled table wrapper (used with TanStack Table) |
| `Card` | — | Content container with header, body, footer slots |
| `Badge` | — | Status indicators (belt colors, payment status, member status) |
| `Avatar` | `@radix-ui/react-avatar` | Member/parent profile photos with fallback initials |
| `Tabs` | `@radix-ui/react-tabs` | Tab navigation (detail views, settings) |
| `Toast` | `@radix-ui/react-toast` | Non-blocking notification messages |
| `Tooltip` | `@radix-ui/react-tooltip` | Hover tooltips |
| `Skeleton` | — | Loading placeholder for async content |
| `Separator` | `@radix-ui/react-separator` | Horizontal/vertical dividers |
| `ScrollArea` | `@radix-ui/react-scroll-area` | Custom scrollbar for lists |
| `DatePicker` | `@radix-ui/react-popover` + calendar | Date selection with locale-aware formatting |
| `FormField` | — | Connects React Hook Form to any input component |
| `EmptyState` | — | Illustration + message for empty lists/searches |
| `Spinner` | — | Loading indicator (inline for mutations) |
| `AlertBanner` | — | Full-width info/warning/error banner |

### Import Pattern

Apps import components directly from the shared package:

```typescript
import { Button, Card, Badge, Avatar } from "@ararat/ui";
import { FormField } from "@ararat/ui";
```

---

## 3. Shared API Client (`packages/api-client/`)

The API client package provides type-safe HTTP communication with the NestJS backend. It is built on **ky** (lightweight fetch wrapper) and **TanStack Query v5** for server state management.

### HTTP Client Configuration

A single configured `ky` instance serves as the base HTTP client for all API calls:

- **Base URL**: Read from environment variable `VITE_API_BASE_URL` (e.g., `https://api.ararat.app/api/v1`)
- **Tenant scoping**: Most endpoints are prefixed with `/tenants/${tenantId}/`. The `tenantId` is read from the Zustand auth store and injected automatically.
- **Content-Type**: `application/json` by default
- **Accept-Language**: Set from user's language preference in auth store (e.g., `ko-KR`)

### Auth Token Injection

The `ky` instance uses a `beforeRequest` hook to inject the access token:

1. On every request, read the current access token from the Zustand auth store.
2. If token exists, set `Authorization: Bearer ${accessToken}` header.
3. If token is absent, skip the header (only for unauthenticated endpoints like `/auth/login`).

### Automatic Token Refresh

The `ky` instance uses an `afterResponse` hook to handle expired tokens:

1. If response status is `401 Unauthorized`:
   a. Call `POST /api/v1/auth/refresh` (refresh token is in httpOnly cookie, sent automatically).
   b. If refresh succeeds: update access token in Zustand store, retry the original request with new token.
   c. If refresh fails (refresh token expired or revoked): clear auth store, redirect to `/login`.
2. Only one refresh attempt per failed request (prevent infinite loops).
3. Concurrent 401s are queued — only one refresh call is made; all waiting requests retry after it completes.

### Tenant-Scoped Base URL

Endpoints are organized into tenant-scoped and global paths:

- **Tenant-scoped** (most endpoints): `${API_BASE}/tenants/${tenantId}/members`, `${API_BASE}/tenants/${tenantId}/attendance`, etc.
- **Global** (auth, system): `${API_BASE}/auth/login`, `${API_BASE}/auth/refresh`, `${API_BASE}/auth/logout`

The client provides a helper that automatically prepends the tenant prefix:

```typescript
// tenantUrl("/members") → "/api/v1/tenants/{tenantId}/members"
// globalUrl("/auth/login") → "/api/v1/auth/login"
```

### Type-Safe Endpoint Definitions

API request and response types are defined in `packages/api-client/src/types/` and match the NestJS DTOs. Zod schemas from `packages/shared/src/schemas/` are used to validate responses where needed (defensive parsing for critical data).

### Standard Query Hooks Pattern

Each domain module exports query and mutation hooks following a consistent naming convention:

**Query hooks** (read data):

```typescript
// packages/api-client/src/hooks/members.ts
export const useMembers = (filters?: MemberFilters) => {
  // GET /tenants/{tenantId}/members?...filters
  // Returns: { data: Member[], meta: PaginationMeta }
};

export const useMember = (memberId: string) => {
  // GET /tenants/{tenantId}/members/{memberId}
  // Returns: { data: Member }
};

export const useMemberAttendance = (memberId: string, filters?: DateRangeFilter) => {
  // GET /tenants/{tenantId}/members/{memberId}/attendance
};
```

**Mutation hooks** (write data):

```typescript
export const useCreateMember = () => {
  // POST /tenants/{tenantId}/members
  // onSuccess: invalidates ['members', tenantId]
};

export const useUpdateMember = () => {
  // PATCH /tenants/{tenantId}/members/{memberId}
  // onSuccess: invalidates ['members', tenantId] and ['members', tenantId, memberId]
};
```

### Error Handling

API errors are mapped from the standard error envelope (see [05-api-design](./05-api-design.md)) to a typed `ApiError` object:

```typescript
interface ApiError {
  code: string;        // e.g., "VALIDATION_ERROR", "NOT_FOUND"
  message: string;     // User-friendly message (i18n key or English fallback)
  field?: string;      // For validation errors: which field failed
  status: number;      // HTTP status code
}
```

Hooks expose errors via TanStack Query's `error` property. UI components check `error.code` to display appropriate messages.

---

## 4. Routing Strategy (TanStack Router)

All three apps use **TanStack Router v1** with file-based route generation. Routes are defined as files in each app's `src/routes/` directory. TanStack Router generates the route tree at build time (`routeTree.gen.ts`).

### Route File Convention

```
src/routes/
├── __root.tsx          # Root layout (nav, error boundary, providers)
├── _auth.tsx           # Authenticated layout (checks auth, redirects)
├── _auth/
│   ├── index.tsx       # Dashboard (/)
│   ├── members/
│   │   ├── index.tsx   # Member list (/members)
│   │   └── $id.tsx     # Member detail (/members/:id)
│   └── ...
└── login.tsx           # Public login route (/login)
```

### Protected Route Guard

The `_auth.tsx` layout route uses TanStack Router's `beforeLoad` hook to enforce authentication:

1. Check `isAuthenticated` in Zustand auth store.
2. If not authenticated: throw `redirect({ to: '/login' })`.
3. If authenticated but token is expired: attempt silent refresh before redirecting.
4. If authenticated: proceed to render child route.

Role-based route protection is layered on top: admin-only routes (e.g., `/settings`, `/audit-log`) check `user.role` in `beforeLoad` and throw `redirect` if insufficient permissions.

### Parent App Route Structure

| Route | Component | Auth Required |
|-------|-----------|---------------|
| `/login` | `LoginPage` | No |
| `/` | `DashboardPage` | Yes |
| `/children` | `ChildrenListPage` | Yes |
| `/children/:id` | `ChildDetailPage` | Yes |
| `/payments` | `PaymentsPage` | Yes |
| `/simsa` | `SimsaPage` | Yes |
| `/notifications` | `NotificationsPage` | Yes |
| `/newsletters` | `NewslettersPage` | Yes |
| `/settings` | `SettingsPage` | Yes |
| `/withdraw` | `WithdrawPage` | Yes |

### Admin App Route Structure

| Route | Component | Auth Required | Min Role |
|-------|-----------|---------------|----------|
| `/login` | `AdminLoginPage` | No | — |
| `/` | `DashboardPage` | Yes | Instructor |
| `/members` | `MemberListPage` | Yes | Instructor |
| `/members/:id` | `MemberDetailPage` | Yes | Instructor |
| `/registrations` | `RegistrationsPage` | Yes | Manager |
| `/classes` | `ClassManagementPage` | Yes | Manager |
| `/attendance` | `AttendancePage` | Yes | Instructor |
| `/payments` | `PaymentsOverviewPage` | Yes | Manager |
| `/payments/plans` | `MembershipPlansPage` | Yes | Owner |
| `/simsa` | `SimsaListPage` | Yes | Manager |
| `/simsa/:id` | `SimsaDetailPage` | Yes | Manager |
| `/notifications` | `NotificationsPage` | Yes | Manager |
| `/newsletters` | `NewslettersPage` | Yes | Manager |
| `/feed` | `FeedManagementPage` | Yes | Instructor |
| `/reports` | `ReportsPage` | Yes | Manager |
| `/settings` | `SettingsPage` | Yes | Owner |
| `/audit-log` | `AuditLogPage` | Yes | Owner |
| `/tasks` | `TasksPage` | Yes | Manager |

### Monitor App Route Structure

| Route | Component | Auth Required |
|-------|-----------|---------------|
| `/` | `DisplayPage` | Token via URL query param |
| `/setup` | `SetupPage` | No (device registration) |

---

## 5. Authentication Flow (Frontend)

### Parent App — Phone OTP Flow

1. Parent navigates to `/login`.
2. Parent enters US phone number (+1 format). Client validates format with Zod schema.
3. Client calls `POST /api/v1/auth/otp/send` with `{ phone }`.
4. Server sends 6-digit OTP via Twilio SMS.
5. UI transitions to OTP entry screen (6-digit code input with auto-focus per digit).
6. Parent enters OTP. Client calls `POST /api/v1/auth/otp/verify` with `{ phone, code }`.
7. If valid: server returns JWT access token in response body. Server sets httpOnly refresh token cookie.
8. Client stores access token in Zustand auth store (memory only). Decodes JWT to extract `userId`, `tenantId`, `role`, `language`.
9. Client redirects to `/` (dashboard).
10. If invalid OTP: show inline error "Invalid code. Please try again." Allow up to 3 attempts. After 3 failures, show "Too many attempts. Please request a new code." with resend button.

### Admin App — Email + Password + 2FA Flow

1. Admin navigates to `/login`.
2. Admin enters email and password.
3. Client calls `POST /api/v1/auth/login` with `{ email, password }`.
4. If credentials valid: server returns `{ requires2fa: true, tempToken }`.
5. UI transitions to 2FA screen (6-digit TOTP input).
6. Admin enters authenticator app code. Client calls `POST /api/v1/auth/2fa/verify` with `{ tempToken, code }`.
7. If valid: server returns JWT access token in response body. Server sets httpOnly refresh token cookie.
8. Client stores access token in Zustand auth store. Decodes JWT to extract `userId`, `tenantId`, `role`, `language`.
9. Client redirects to `/` (dashboard).
10. If credentials invalid: show "Invalid email or password." After 5 failed attempts, show "Account locked. Try again in 15 minutes."

### Token Storage

| Token | Storage | Lifetime |
|-------|---------|----------|
| Access token (JWT) | Zustand store (in-memory) | 1 hour (parent), 8 hours (admin) |
| Refresh token | httpOnly, Secure, SameSite=Strict cookie (set by server) | 30 days |

Access tokens are never written to localStorage or sessionStorage — they exist only in memory. This means a page refresh requires a silent refresh call to obtain a new access token from the refresh token cookie.

### Auto-Refresh Mechanism

On app initialization (before rendering protected routes):

1. Call `POST /api/v1/auth/refresh` (cookie sent automatically).
2. If success: store new access token in Zustand, proceed to requested route.
3. If failure (no cookie, expired, revoked): redirect to `/login`.

During active usage, the `ky` afterResponse hook handles 401s as described in Section 3.

### Auth Zustand Store

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
  isLoading: boolean;  // True during initial refresh attempt
  setAuth: (user: AuthState["user"], token: string) => void;
  clearAuth: () => void;
}
```

### Logout

1. Client calls `POST /api/v1/auth/logout` (server revokes refresh token in Redis, clears cookie).
2. Client calls `clearAuth()` in Zustand store.
3. Client calls `queryClient.clear()` to purge all cached data.
4. Redirect to `/login`.

---

## 6. State Management Pattern

### Server State — TanStack Query (95% of state)

All data fetched from the API is managed by TanStack Query. This includes member lists, attendance records, payment history, notifications, etc.

**Default Query Options** (configured in `QueryClient`):

| Option | Value | Rationale |
|--------|-------|-----------|
| `staleTime` | 30 seconds | Data is considered fresh for 30s — no background refetch during this window |
| `gcTime` | 5 minutes | Inactive query data is garbage collected after 5 minutes |
| `retry` | 2 | Retry failed queries twice before surfacing error |
| `refetchOnWindowFocus` | true | Refresh data when user returns to tab |

**Query Key Conventions**:

Query keys are structured arrays that enable targeted invalidation:

| Pattern | Example | Invalidation Scope |
|---------|---------|-------------------|
| Entity list | `['members', tenantId]` | All member lists for this tenant |
| Entity list (filtered) | `['members', tenantId, { status: 'active', belt: 'White' }]` | Specific filtered view |
| Single entity | `['members', tenantId, memberId]` | One member's data |
| Nested resource | `['attendance', tenantId, { date: '2026-02-25', classId }]` | Attendance for specific date/class |
| Dashboard stats | `['dashboard', tenantId]` | Dashboard aggregate data |
| Reports | `['reports', tenantId, reportType, filters]` | Specific report data |

**Mutation Patterns**:

All mutations follow this flow:

1. `useMutation` wraps the API call.
2. `onSuccess`: call `queryClient.invalidateQueries({ queryKey: [...] })` to refetch affected lists.
3. `onSuccess`: show success toast (e.g., "Member created successfully").
4. `onError`: show error toast with message from `ApiError`.

**Optimistic Updates** (used sparingly for instant UX):

| Action | Optimistic Behavior |
|--------|-------------------|
| Mark notification as read | Immediately update notification badge count and item state |
| Confirm attendance check-in | Immediately add member to attendance list |
| Toggle settings switch | Immediately reflect new state |

Optimistic updates use `queryClient.setQueryData` in `onMutate` and roll back in `onError`.

### Client State — Zustand (5% of state)

Client-only state that does not come from the API:

| Store | State | Persisted? |
|-------|-------|------------|
| `authStore` | `user`, `accessToken`, `isAuthenticated`, `isLoading` | No (memory only) |
| `uiStore` | `sidebarOpen`, `activeModal`, `tablePageSize` | Yes (localStorage) |
| `preferencesStore` | `language`, `theme` (future), `timezone` | Yes (localStorage) |

Zustand stores use the `persist` middleware for stores that survive page refreshes (ui, preferences). The auth store is intentionally NOT persisted — access tokens live in memory only.

---

## 7. Form Handling Pattern

All forms across the three apps use **React Hook Form v7** with the **Zod resolver** for validation. Zod schemas are imported from `packages/shared/` — the same schemas used by NestJS validation pipes on the backend.

### Schema Sharing

```
packages/shared/src/schemas/member.ts
  ↓ imported by
packages/api-client/src/hooks/members.ts (useCreateMember mutation)
  ↓ and
apps/admin/src/routes/_auth/members/new.tsx (form validation)
  ↓ and
server/src/members/dto/create-member.dto.ts (NestJS validation pipe)
```

This ensures that client-side validation rules always match server-side validation. If a schema changes, both sides update from the same source.

### FormField Component

The `FormField` component from `@ararat/ui` connects React Hook Form's `Controller` to any shadcn/ui input component:

- Renders label, input, and error message.
- Receives `control` and `name` from the form context.
- Displays field-level errors from Zod validation (client-side).
- Can display server-side errors mapped by field name from API error responses.
- Supports `required`, `disabled`, and `description` (help text) props.

### Validation Strategy

1. **Client-side (immediate)**: Zod schema validates on blur and on submit. Errors appear inline below the field.
2. **Server-side (on submit)**: If the API returns `VALIDATION_ERROR` with `field` names, errors are mapped back to the form using `setError(fieldName, { message })`.
3. **Form-level errors**: Non-field-specific errors (e.g., "Registration pending approval") appear as an alert banner above the form.

### Common Form Patterns

| Form Type | Description |
|-----------|-------------|
| **Create forms** | Full schema validation, all fields required per schema, submit creates new resource |
| **Edit forms** | Partial schema (`.partial()`), pre-populated from query data, submit patches existing resource |
| **Filter forms** | No submit button — values update query params in real-time (debounced) |
| **Multi-step forms** | Registration and withdrawal use step-by-step wizard with per-step Zod schemas |
| **Confirmation dialogs** | Simple yes/no with optional reason text field |

---

## 8. Responsive Design Strategy

### Parent App — Mobile-First

The Parent App is designed for phone-sized viewports (375px+) as the primary experience. It scales up to tablet and desktop but the phone layout is the default.

- **Layout**: Single-column. Content stacks vertically.
- **Navigation**: Bottom tab bar (5 tabs: Home, Children, Payments, Notifications, Settings). On tablet/desktop, tabs move to a top navigation bar.
- **Touch targets**: Minimum 44px × 44px (Apple HIG standard).
- **Typography**: Base font size 16px to prevent iOS zoom on input focus.
- **Breakpoints used**: `sm` (640px) for slight layout adjustments, `md` (768px) for tablet layout (2-column where appropriate).

### Admin App — Desktop-First

The Admin App is designed for desktop viewports (1024px+). It is responsive down to tablet (768px) but not optimized for phone use.

- **Layout**: Sidebar navigation (left, 256px width, collapsible to 64px icon-only) + main content area.
- **Navigation**: Collapsible sidebar with grouped menu items (Members, Classes, Attendance, Payments, 심사, Communications [Notifications + Newsletters + Feed], Reports, Settings). Sidebar state persisted in Zustand `uiStore`.
- **Data tables**: Full-width with horizontal scroll on tablet if needed. Column visibility adapts — fewer columns shown at `md` breakpoint.
- **Breakpoints used**: `lg` (1024px) full layout, `md` (768px) collapsed sidebar + stacked cards for dashboard.

### Monitor App — Fixed Fullscreen

The Monitor App is designed for a single fixed resolution.

- **Target resolutions**: 1920×1080 (Full HD) and 1280×720 (HD).
- **Layout**: Fixed absolute positioning. No scrolling on the main page (attendance feed auto-scrolls internally).
- **No responsive needed**: The app assumes a landscape display. If viewport is too small, show "Please use a larger display" message.

### Shared Breakpoints

All apps share the same Tailwind breakpoint definitions for consistency:

| Breakpoint | Min Width | Primary Use |
|------------|-----------|-------------|
| `sm` | 640px | Parent app: minor adjustments |
| `md` | 768px | Parent app: tablet layout. Admin app: collapsed sidebar |
| `lg` | 1024px | Admin app: full desktop layout |
| `xl` | 1280px | Admin app: wide tables, multi-column dashboard |

---

## 9. Error Handling & Loading States

### Global Error Boundary

Each app wraps its root component in a React error boundary that catches unhandled rendering errors:

- **Fallback UI**: Shows a "Something went wrong" card with the error message (in production, generic; in development, full stack trace).
- **Recovery action**: "Reload Page" button.
- **Logging**: Errors are reported to the console (and optionally to a monitoring service in production).

### API Error Handling

TanStack Query surfaces errors through the `error` property on query/mutation results. The approach varies by error type:

| Error Code | HTTP Status | UI Behavior |
|------------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Map field errors to form inputs. Show inline error messages. |
| `UNAUTHORIZED` | 401 | Trigger token refresh. If refresh fails, redirect to login. |
| `FORBIDDEN` | 403 | Show toast: "You don't have permission for this action." |
| `NOT_FOUND` | 404 | Show empty state or "Resource not found" page. |
| `CONFLICT` | 409 | Show toast with specific conflict message (e.g., "A member with this phone number already exists"). |
| `RATE_LIMITED` | 429 | Show toast: "Too many requests. Please wait and try again." Auto-retry after `Retry-After` header. |
| `INTERNAL_ERROR` | 500 | Show toast: "Something went wrong. Please try again." Log full error. |
| Network error | — | Show banner: "Unable to connect. Check your internet connection." Retry on reconnect. |

### Loading States

| Scenario | Loading Indicator |
|----------|------------------|
| Initial page load | `Skeleton` components matching the layout structure |
| Table data loading | Table skeleton rows (5 placeholder rows) |
| Form submission | Button shows `Spinner` + "Saving..." text, form inputs disabled |
| Navigation between pages | Small top-of-page progress bar |
| Background refetch | No visible indicator (silent) |
| Infinite scroll / pagination | Spinner at bottom of list |

### Empty States

Every list view has a dedicated empty state component:

| Screen | Empty State Message |
|--------|-------------------|
| Activity Feed | "No activity yet today. Check back after class!" (with illustration) |
| Children List | "You haven't registered any children yet." + "Add Child" button |
| Payments | "No payment history yet." |
| Notifications | "You're all caught up!" (with checkmark illustration) |
| Newsletters | "No newsletters yet." |
| Member List (admin) | "No members found matching your filters." + "Clear Filters" button |
| Attendance (admin) | "No attendance records for this date." |
| Search results | "No results for '{query}'." + "Try a different search" |

### Offline Detection

The Parent App and Admin App detect offline status using `navigator.onLine` and the `online`/`offline` events:

- **When offline**: Show a non-dismissible banner at the top: "You are offline. Some features may be unavailable."
- **When back online**: Banner shows "Back online. Refreshing data..." and triggers `queryClient.invalidateQueries()` to refresh stale data.
- **Mutations while offline**: Mutations are blocked with a toast: "Cannot save while offline. Please check your connection."

The Monitor App handles offline differently — see [18-monitor-app](./18-monitor-app.md) Auto-Recovery section.

---

## 10. Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
