# 17. Feature: Admin Tools

**Related TRDs**: [04-auth](./04-auth.md), [13-notifications](./13-notifications.md), [16-reporting](./16-reporting.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1)

---

### Overview

Administrative tools for gym operations: audit logging, task management, announcements, gym settings, staff account management, belt/age group configuration, payment policies, notification settings, and kiosk configuration. These tools are used exclusively by admin users (Owner, Manager) through the Admin App.

---

### Business Rules

#### RBAC

Role-based access control governs all admin tool endpoints. See [TRD 04 — Authentication](./04-auth.md) for the full RBAC permission matrix. Key constraints:

- **Owner**: Full access to all admin tools, settings, staff management, audit log, and configuration.
- **Manager**: Access to tasks, announcements, and class management. Cannot modify gym settings, staff accounts, or belt configuration.
- **Instructor**: Read-only access to tasks assigned to them. No access to audit log or settings.

#### Audit Log

Every data mutation across the system is recorded in an append-only audit log.

**Data Model**:

```
AuditLog:
  - log_id (UUID, PK)
  - tenant_id (UUID, FK → Tenant)
  - actor_id (UUID, FK → User)
  - actor_role (Owner, Manager, Instructor, Parent)
  - action (Create, Update, Delete)
  - entity_type (Member, Payment, Class, Simsa, Setting, etc.)
  - entity_id (UUID — which record was changed)
  - old_value (JSON — previous state, null for Create)
  - new_value (JSON — new state, null for Delete)
  - ip_address (string)
  - created_at (timestamp, immutable)
```

**Rules**:
- Audit log records are immutable — no updates or deletes.
- Admin can filter by date range, actor, action type, entity type, and free-text search.
- Admin can view full before/after JSON diff for any entry.
- Admin can export filtered results to CSV.
- Retention: configurable per gym (default 2 years). Older records archived to cold storage.

#### Task System

Operational task tracking for staff follow-ups, auto-created by system events or manually by admin.

**Data Model**:

```
Task:
  - task_id (UUID, PK)
  - tenant_id (UUID, FK → Tenant)
  - title (string, required)
  - description (text, optional)
  - assignee_id (UUID, FK → User, optional)
  - due_date (date, optional)
  - status (Open, InProgress, Done)
  - related_entity_type (Member, Payment, Simsa, etc., optional)
  - related_entity_id (UUID, optional)
  - created_at (timestamp)
  - updated_at (timestamp)
```

**Auto-Created Tasks**:
- **Escalation alerts**: When alert rules (absence, late payment) are ignored past threshold, a task is created for admin review. See [TRD 13 — Notifications](./13-notifications.md) alert rules engine.
- **Age group transfers**: When a member's age crosses an age group boundary, a task is created for admin to review and approve the group transfer.
- **Withdrawal requests**: When a parent submits a withdrawal request, a task is created for admin approval.

**Manual Tasks**:
- Admin can create tasks with title, description, assignee, and due date.
- Tasks can be linked to any entity (member, payment, etc.) via `related_entity_type` and `related_entity_id`.

**Status Transitions**: Open → InProgress → Done. Tasks cannot be reopened once Done.

#### Announcement System

Admin broadcasts announcements to members through multiple channels.

**Flow**:
1. Admin clicks "Create Announcement".
2. Admin enters title and body (rich text).
3. Admin selects audience: all members, or filter by class, belt level, or age group.
4. Admin selects delivery channels: push, SMS, email, in-app (one or more).
5. Admin schedules for future delivery or sends immediately.
6. System dispatches announcement to selected audience via [TRD 13 — Notifications](./13-notifications.md) dispatch infrastructure.
7. Delivery status tracked in notification history.

#### Gym Settings Management

All configurable settings are organized by category and managed through a single settings interface. Changes are logged in the audit log.

**Setting Categories**:
- **Gym Profile**: name, address, phone, email, logo, business hours.
- **Belt Configuration**: ordered list of belt levels with colors and names.
- **Age Group Thresholds**: age ranges for automatic group assignment (e.g., Little Kids 4–6, Kids 7–9, Teens 10–14, Adults 15+).
- **Payment Policies**: grace period days, refund clawback percentage, sibling discount percentage, auto-pay settings.
- **Notification Settings**: default channels per notification type, quiet hours (no notifications outside these hours), template customizations.
- **Alert Thresholds**: consecutive absence days before alert (default 3, 7, 14), overdue payment days before escalation.
- **Kiosk Configuration**: registered devices, check-in mode (face recognition, QR code, manual), idle timeout.
- **System**: timezone, default language (EN/KO/ES), data retention policy (audit log, notifications).

---

### Backend

#### API Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/audit-log` | Query audit log (filterable, paginated) | Owner |
| `GET` | `/api/v1/tenants/{tenantId}/tasks` | List tasks (filterable by status, assignee) | Owner, Manager, Instructor (own only) |
| `POST` | `/api/v1/tenants/{tenantId}/tasks` | Create a manual task | Owner, Manager |
| `PATCH` | `/api/v1/tenants/{tenantId}/tasks/{id}` | Update task (status, assignee, due date) | Owner, Manager |
| `POST` | `/api/v1/tenants/{tenantId}/announcements` | Create and dispatch announcement | Owner, Manager |
| `GET` | `/api/v1/tenants/{tenantId}/settings` | Get all gym settings by category | Owner |
| `PATCH` | `/api/v1/tenants/{tenantId}/settings` | Update gym settings (partial update) | Owner |
| `GET` | `/api/v1/tenants/{tenantId}/staff` | List staff accounts | Owner |
| `POST` | `/api/v1/tenants/{tenantId}/staff` | Create staff account | Owner |
| `PATCH` | `/api/v1/tenants/{tenantId}/staff/{id}` | Update staff (role, status) | Owner |
| `GET` | `/api/v1/tenants/{tenantId}/belt-config` | Get belt configuration (ordered list) | Owner, Manager |
| `PUT` | `/api/v1/tenants/{tenantId}/belt-config` | Replace full belt configuration (ordered list) | Owner |

**Query parameters** for `GET /audit-log`:
- `from`, `to` — date range filter
- `actor_id` — filter by actor
- `action` — filter by action type (Create, Update, Delete)
- `entity_type` — filter by entity type
- `search` — free-text search across entity type, actor name, summary
- Standard pagination (`page`, `limit`)

**Query parameters** for `GET /tasks`:
- `status` — Open, InProgress, Done
- `assignee_id` — filter by assignee
- Standard pagination (`page`, `limit`)

**Request body** for `POST /announcements`:
```json
{
  "title": "string",
  "body": "string",
  "audience": {
    "type": "all" | "class" | "belt" | "age_group",
    "ids": ["uuid"]
  },
  "channels": ["Push", "SMS", "Email", "InApp"],
  "scheduled_at": "ISO 8601 timestamp or null for immediate"
}
```

**Request body** for `POST /staff`:
```json
{
  "email": "string",
  "name": "string",
  "role": "Manager" | "Instructor",
  "temp_password": "string"
}
```

#### Service Logic

- **AuditLogService**: Append-only writes via interceptor/decorator on all mutation endpoints. Query with filtering and pagination. CSV export generates a downloadable file via presigned S3 URL.
- **TaskService**: CRUD operations with status transitions. Auto-creation triggered by domain events (escalation, age transfer, withdrawal). Notification to assignee on task creation/assignment.
- **AnnouncementService**: Validates audience filters, resolves recipient list, delegates to NotificationService (TRD 13) for multi-channel dispatch.
- **SettingsService**: Read/update gym settings by category. Validates setting values against allowed ranges/types. Logs all changes to audit log.
- **StaffService**: CRUD for staff accounts. Create sends invitation email with temporary password. Deactivation revokes all active sessions. Role changes logged to audit log.
- **BeltConfigService**: Manages ordered belt level list. `PUT` replaces the entire list (preserves existing member belt associations). Validates no duplicate names.

---

### Admin App

#### Screen: Settings (`/settings`)

Owner role only. Central hub for all gym configuration organized into tabbed sections.

**Layout**: Left sidebar tab navigation (desktop) / top tab bar (mobile) with content area on the right.

##### Tab: Gym Profile

- Form fields: gym name, address (street, city, state, zip), phone, email
- Logo upload with preview (drag-and-drop or click, max 2MB, JPEG/PNG)
- Business hours: day-of-week grid with open/close time pickers
- Save button with optimistic update

##### Tab: Staff Accounts

- **TanStack Table** with columns: Name, Email, Role (badge), Status (Active/Inactive badge), Last Login
- Actions column: Edit, Deactivate/Activate toggle
- "Add Staff" button → Dialog form: email, name, role dropdown (Manager, Instructor), auto-generated temporary password with copy button
- Edit dialog: change role, deactivate account
- Deactivation requires confirmation dialog ("This will revoke all active sessions")

##### Tab: Belt Configuration

- Ordered list with drag-and-drop reorder (using `@dnd-kit/sortable`)
- Each row: color swatch, belt name, actions (edit, remove)
- "Add Belt Level" button → inline form at bottom of list
- Remove requires confirmation if members currently hold that belt
- Save button persists the full ordered list via `PUT /belt-config`

##### Tab: Age Group Thresholds

- List of age groups with editable min/max age fields
- Default groups: Little Kids (4–6), Kids (7–9), Teens (10–14), Adults (15+)
- Validation: ranges must not overlap, must cover all ages 4+

##### Tab: Payment Policies

- Form fields:
  - Grace period days (number input, default 7)
  - Refund clawback percentage (number input, 0–100, default 0)
  - Sibling discount percentage (number input, 0–50, default 10)
  - Auto-pay enabled (toggle switch)
- Save with validation

##### Tab: Notification Settings

- Default channels grid: notification type × channel toggle matrix (mirrors TRD 13 channel routing table)
- Quiet hours: start time and end time pickers
- Template customization link → navigates to `/notifications` Templates tab (TRD 13)

##### Tab: Alert Thresholds

- Absence alert days: multi-step threshold input (e.g., 3, 7, 14 days)
- Overdue payment days: threshold input
- Membership expiry warning days: threshold input
- Each threshold shows current value with inline edit

##### Tab: Kiosk Configuration

- Registered devices table: device name, device ID, last seen, status
- Check-in mode: radio group (Face Recognition, QR Code, Manual)
- Idle timeout: number input (seconds)
- Device registration instructions / QR code for pairing

##### Tab: System

- Timezone: dropdown selector (US timezones)
- Default language: radio group (English, Korean, Spanish)
- Data retention policy: audit log retention period (dropdown: 1 year, 2 years, 5 years), notification retention period

**Key components**: Tabbed layout with `Tabs` component, forms with React Hook Form + Zod validation, `TanStack Table` for staff list, `@dnd-kit/sortable` for belt drag-and-drop, policy config forms with number inputs and toggles.

**Data sources**: `useGymSettings()`, `useStaffList()`, `useBeltConfig()`.

---

#### Screen: Audit Log (`/audit-log`)

Owner role only. Full audit trail of all data mutations across the gym.

**Layout**: Full-width table with filter bar above.

**Key components**:
- **Filter bar**: date range picker, actor dropdown, action type dropdown (Create/Update/Delete), entity type dropdown, free-text search input
- **TanStack Table** with virtual scrolling for large datasets. Columns:
  - Timestamp (formatted to gym timezone)
  - Actor (name + role badge)
  - Action (Create/Update/Delete with color coding)
  - Entity Type
  - Entity ID (truncated UUID)
  - Summary (auto-generated change description)
  - IP Address
- **Row click** → Side `Sheet` panel with full detail view:
  - All audit log fields
  - Before/after JSON diff with syntax highlighting (additions in green, deletions in red)
- **Export button** → CSV download of current filtered view

**Data source**: `useAuditLog(filters)` — server-side pagination and filtering.

**Empty state**: "No audit log entries match your filters."

---

#### Screen: Task Management (`/tasks`)

Owner and Manager roles. Operational task board for tracking follow-ups.

**Layout**: Two-section layout with Open/InProgress tasks on top, Completed tasks below (collapsible).

##### Open Tasks Section

- Card list / compact list view (toggle)
- Each task card:
  - Priority badge (derived from due date proximity: overdue = red, due soon = yellow, normal = gray)
  - Title (bold)
  - Description (truncated, 2 lines)
  - Assignee avatar + name
  - Due date (with relative time: "Due in 3 days", "Overdue by 2 days")
  - Related entity link (if linked — e.g., "Member: John Kim")
  - Action button: context-dependent — navigates to relevant screen (e.g., member profile, payment detail)
- Actions: Mark as In Progress, Mark as Complete, Reassign

##### Completed Tasks Section

- Collapsible section showing archived completed tasks
- Same card layout but muted styling
- Filter by completion date range

##### Task Actions

- **Create Task** button → Dialog form: title (required), description, assignee dropdown (staff list), due date picker, related entity type + entity search
- **Filter/Sort bar**: status filter, assignee filter, sort by due date / created date
- **Bulk actions**: mark multiple tasks complete

**Data source**: `useTasks(status)` — separate queries for open and completed.

---

#### Screen: Class Management (`/classes`)

Owner and Manager roles. Schedule and manage class sessions.

**Layout**: Toggle between weekly calendar grid view and list view.

##### Calendar View

- Weekly grid: columns = days (Mon–Sat), rows = time slots (30-min increments)
- Class cards placed in calendar cells:
  - Class name
  - Time range (e.g., "4:00–5:00 PM")
  - Instructor name
  - Capacity indicator: `enrolled / capacity` with color (green < 80%, yellow 80–95%, red > 95%)
- Click empty slot → Create class dialog (pre-filled with day/time)
- Click existing class → Edit class dialog

##### List View

- TanStack Table with columns: Day, Time, Class Name, Instructor, Enrolled/Capacity, Actions
- Sortable by any column

##### Class Dialog (Create/Edit)

- Form fields: class name, day of week (multi-select for recurring), start time, end time, instructor (dropdown from staff), capacity (number), description, belt level requirements (optional)
- Validation: no time overlap for same instructor, capacity ≥ 1
- Delete button (edit mode only) with confirmation dialog

**Data sources**: `useClasses()`, `useInstructors()`.

**Actions**: Create class, edit class, delete class (with confirmation), view enrolled members (navigates to filtered member list).

---

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
