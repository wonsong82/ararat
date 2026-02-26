# 16. Feature: Admin Tools

**Related TRDs**: [04-auth](./04-auth.md), [12-notifications](./12-notifications.md), [15-reporting](./15-reporting.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1)

---

### RBAC Permission Matrix

(See section 4 for full matrix.)

### Audit Log

Every data mutation is logged:

```
AuditLog:
  - log_id
  - tenant_id
  - actor_id (user who made the change)
  - actor_role (Owner, Manager, Instructor, etc.)
  - action (Create, Update, Delete)
  - entity_type (Member, Payment, Class, etc.)
  - entity_id (which record was changed)
  - old_value (JSON, previous state)
  - new_value (JSON, new state)
  - ip_address
  - created_at (immutable)
```

Admin can view audit log:
1. Admin visits "Audit Log" page.
2. Admin filters by: date range, actor, entity type, action.
3. Admin sees list of changes with before/after values.
4. Admin can export audit log to CSV.

### Task System

Simple CRUD tasks for operational follow-ups:

```
Task:
  - task_id
  - tenant_id
  - title
  - description
  - assignee_id (staff member)
  - due_date
  - status (Open, InProgress, Done)
  - related_entity_type (Member, Payment, etc.)
  - related_entity_id
  - created_at
  - updated_at
```

**Auto-Created Tasks**:
- Escalation rules create tasks when alerts are ignored.
- Age group transfers create tasks for admin review.
- Withdrawal requests create tasks for admin approval.

**Manual Tasks**:
- Admin can create tasks manually for any operational need.

### Announcement System

Admin broadcasts announcements to members:

1. Admin clicks "Create Announcement".
2. Admin enters title and body.
3. Admin selects audience (all members, or filter by class/belt/group).
4. Admin selects channels (push, SMS, email, in-app).
5. Admin can schedule for future delivery or send immediately.
6. System dispatches announcement to selected audience.

### Gym Settings Management

All configurable settings in one admin page:

1. Admin visits "Settings".
2. Admin sees all settings organized by category (Billing, Notifications, Attendance, etc.).
3. Admin can edit any setting.
4. Changes logged in AuditLog.

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
