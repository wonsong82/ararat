# 10. Feature: Attendance System

**Related TRDs**: [03-data-model](./03-data-model.md), [08-kiosk-app-architecture](./08-kiosk-app-architecture.md), [13-notifications](./13-notifications.md), [18-monitor-app](./18-monitor-app.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md)  
**Phase**: MVP (Phase 1)

---

## Overview

The Attendance System is the operational core of Ararat — it tracks every student check-in across multiple methods (face recognition, QR code, name search, staff manual), enforces an optional staff validation workflow, maintains an immutable audit trail for child safety, and powers absence alerting. The system spans all four applications:

- **Kiosk App**: Primary check-in interface (face → QR → name search → staff fallback)
- **Parent/Member App**: Attendance history, calendar heatmap, real-time check-in status
- **Admin App**: Live attendance monitoring, historical reporting, manual overrides, at-risk member tracking
- **Monitor App**: Real-time attendance display on gym TV screens

All attendance data is tenant-scoped. The system supports offline kiosk operation with batch sync when connectivity is restored.

---

## Business Rules

### Check-In Methods (Priority Order)

The kiosk attempts methods in this order, falling through on failure:

1. **Face Recognition** (Primary) — On-device face recognition via iPad TrueDepth/front camera. Requires prior enrollment (see [08-kiosk-app-architecture](./08-kiosk-app-architecture.md)). No biometric data leaves the device (ADR-004).
2. **QR Code** (Secondary) — Scan QR code from physical membership card or parent's phone. QR payload contains an opaque member token (not raw member_id).
3. **Name Search** (Fallback) — Manual search by name with photo confirmation on child-friendly oversized keyboard.
4. **Staff Manual** (Last Resort) — Staff manually marks member as checked in from Admin App dashboard.

### Check-In Data Captured

Every check-in creates an `Attendance` record with:

| Field | Type | Description |
|-------|------|-------------|
| `member_id` | UUID | Who checked in |
| `tenant_id` | UUID | Gym this attendance belongs to |
| `timestamp` | timestamptz | When check-in occurred (kiosk local time, stored as UTC) |
| `method` | enum | `FaceRecognition`, `QRCode`, `NameSearch`, `StaffManual` |
| `kiosk_id` | UUID (nullable) | Which kiosk recorded it (null for staff manual) |
| `staff_validator_id` | UUID (nullable) | Staff who validated (if dual-check enabled) |
| `validation_status` | enum | `Confirmed`, `PendingValidation`, `Unconfirmed` |
| `validation_time` | timestamptz (nullable) | When staff confirmed |
| `class_id` | UUID (nullable) | Class being attended (auto-matched by schedule or manually selected) |
| `checked_out_at` | timestamptz (nullable) | Manual check-out time (if recorded) |
| `synced_at` | timestamptz (nullable) | When offline record was synced to server |
| `offline` | boolean | Whether this record was created while kiosk was offline |

### Class Auto-Matching

When a check-in occurs, the system attempts to match it to a scheduled class:

1. Query active classes for the current day/time window (class start − 15 min to class end).
2. If exactly one class matches: auto-assign `class_id`.
3. If multiple classes overlap: assign to the class the member is enrolled in. If enrolled in multiple, leave `class_id` null for staff to resolve.
4. If no class matches (e.g., open gym time): leave `class_id` null.

### Dual-Check (Staff Validation) Flow

Configurable per gym via `SystemSetting.dual_check_enabled` (default: `false`).

When enabled:

1. Member checks in at kiosk (any method).
2. Kiosk records check-in with `validation_status = PendingValidation`.
3. Kiosk displays "Awaiting staff confirmation" message to the child.
4. Staff receives real-time notification on Admin App dashboard: "Pending validation: [member_name]".
5. Staff reviews check-in on dashboard and clicks **Confirm**.
6. System updates: `validation_status = Confirmed`, `validation_time = now()`, `staff_validator_id = staff.id`.
7. Parent receives confirmation notification via configured channels.

**Unconfirmed Alert** (configurable timeout, default: 5 minutes):

- If staff does not confirm within the timeout: system sets `validation_status = Unconfirmed`.
- Alert sent to staff dashboard: "Unconfirmed check-in: [member_name] ([N] min ago)".
- Alert sent to parent: "Your child's check-in is pending staff confirmation. Please contact the gym if this is incorrect."
- Unconfirmed records are highlighted in the Admin attendance view for resolution.

When dual-check is **disabled**: all check-ins are recorded directly as `validation_status = Confirmed`.

### Drop-Off Audit Trail

Every attendance-related event creates an immutable `AttendanceAudit` record:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `attendance_id` | UUID | Related attendance record |
| `event_type` | enum | `CheckIn`, `Validation`, `Unconfirmed`, `ManualOverride`, `CheckOut`, `Correction`, `Sync` |
| `event_data` | jsonb | Full details: method, kiosk, staff, validation status, previous values (for corrections) |
| `actor_id` | UUID (nullable) | User who triggered the event (null for system events) |
| `created_at` | timestamptz | Immutable timestamp |

Audit records are **append-only** — never updated or deleted. This provides a complete chain-of-custody trail for child safety and liability protection.

### Absence Alert Engine

#### Configuration

Gym owner configures absence thresholds in `SystemSetting`:

```json
{
  "absence_alert_thresholds": [3, 7, 14],
  "absence_alert_enabled": true
}
```

- `absence_alert_thresholds`: Array of day counts that trigger alerts (default: `[3, 7, 14]`).
- `absence_alert_enabled`: Master toggle (default: `true`).

#### Daily Cron Job

Runs daily at 11:59 PM in the gym's configured timezone:

1. For each active member with `status = Active`: calculate days since last `Attendance` record.
2. For each configured threshold: check if the member has crossed it.
3. If crossed and not already alerted for this threshold: create a `MemberAbsenceAlert` record and trigger notification.
4. `MemberAbsenceAlert` tracks `(member_id, threshold_days, alerted_at)` to prevent duplicate alerts.
5. Alert resets when the member next checks in (delete their `MemberAbsenceAlert` records).

#### Notification Dispatch

When a threshold is crossed:

1. Fetch `NotificationTemplate` for absence alert (language-aware, per [13-notifications](./13-notifications.md)).
2. Render template with personalization tokens: `{{member_name}}`, `{{days_absent}}`, `{{gym_name}}`, `{{gym_phone}}`.
3. Dispatch to parent's configured channels (push, SMS, email — in priority order).
4. Log in `NotificationLog`.

### Pre-Arrival Reservation (Phase 2)

Parent can reserve a drop-off time slot via Parent App:

1. Parent selects class and date.
2. System shows available time slots based on class capacity.
3. Parent selects slot and confirms.
4. System creates `Reservation` record.
5. Gym owner receives notification: "Reservation: [member_name] for [class] on [date]".
6. When member checks in: system matches to reservation and marks as fulfilled.

> _This feature is Phase 2 scope. The `Reservation` entity and endpoints are not included in MVP._

---

## Backend

### API Endpoints

#### Record Check-In

```
POST /api/v1/tenants/{tenantId}/attendance
```

**Auth**: Kiosk API key or Staff JWT  
**Body**:
```json
{
  "memberId": "uuid",
  "method": "FaceRecognition | QRCode | NameSearch | StaffManual",
  "kioskId": "uuid | null",
  "classId": "uuid | null",
  "timestamp": "ISO8601 | null",
  "offline": false
}
```

**Response**: `201 Created` with full `Attendance` object.

**Logic**:
1. Validate member belongs to tenant and is active.
2. Check for duplicate check-in within 5-minute window (same member + same method = idempotent).
3. If `classId` is null, attempt class auto-matching.
4. If dual-check enabled: set `validation_status = PendingValidation`, schedule unconfirmed alert timer.
5. If dual-check disabled: set `validation_status = Confirmed`.
6. Create `AttendanceAudit` with `event_type = CheckIn`.
7. If dual-check disabled or method is `StaffManual`: dispatch parent notification immediately.
8. Return attendance record.

#### Batch Sync from Offline Kiosk

```
POST /api/v1/kiosks/{kioskId}/attendance/batch
```

**Auth**: Kiosk API key  
**Body**:
```json
{
  "records": [
    {
      "memberId": "uuid",
      "method": "FaceRecognition | QRCode | NameSearch",
      "timestamp": "ISO8601",
      "localId": "uuid"
    }
  ]
}
```

**Response**: `200 OK` with array of `{ localId, serverId, status: "created" | "duplicate" | "error" }`.

**Logic**:
1. Validate kiosk belongs to tenant.
2. Process each record: apply same logic as single check-in, with `offline = true` and `synced_at = now()`.
3. Skip duplicates (same member + timestamp within 1-minute window).
4. Create `AttendanceAudit` with `event_type = Sync` for each synced record.
5. Dispatch batched parent notifications (debounce: one notification per member per sync batch).

#### Today's Live Attendance

```
GET /api/v1/tenants/{tenantId}/attendance/today
```

**Auth**: Staff JWT (role: `Admin`, `Instructor`)  
**Query params**: `classId` (optional filter)  
**Response**: Array of today's attendance records with member details (photo URL, name, belt level).

Used by Admin App (live view) and Monitor App (display panel). Supports `If-Modified-Since` header for efficient polling.

#### Historical Attendance

```
GET /api/v1/tenants/{tenantId}/attendance
```

**Auth**: Staff JWT  
**Query params**:
- `startDate`, `endDate` (required, max 90-day range)
- `classId` (optional)
- `memberId` (optional)
- `method` (optional)
- `page`, `limit` (pagination, default limit=50)

**Response**: Paginated attendance records with member details.

#### Attendance Audit Trail

```
GET /api/v1/tenants/{tenantId}/attendance/audit
```

**Auth**: Staff JWT (role: `Admin` only)  
**Query params**:
- `attendanceId` (optional — filter to single record's audit trail)
- `startDate`, `endDate`
- `eventType` (optional)
- `page`, `limit`

**Response**: Paginated audit records.

#### Staff Validates Check-In

```
PATCH /api/v1/tenants/{tenantId}/attendance/{id}/validate
```

**Auth**: Staff JWT (role: `Admin`, `Instructor`)  
**Body**:
```json
{
  "action": "confirm | reject",
  "note": "optional reason for rejection"
}
```

**Logic**:
1. Verify attendance record exists and `validation_status = PendingValidation`.
2. If confirm: set `validation_status = Confirmed`, `staff_validator_id`, `validation_time`.
3. If reject: set `validation_status = Rejected`, create audit record with rejection reason.
4. Create `AttendanceAudit` with `event_type = Validation`.
5. Dispatch parent notification.

#### Member Attendance History

```
GET /api/v1/tenants/{tenantId}/members/{id}/attendance
```

**Auth**: Parent JWT (own children only) or Staff JWT  
**Query params**: `startDate`, `endDate`, `page`, `limit`  
**Response**: Paginated attendance records for the member. Includes summary stats: `totalDays`, `currentStreak`, `longestStreak`.

#### At-Risk Members

```
GET /api/v1/tenants/{tenantId}/attendance/at-risk
```

**Auth**: Staff JWT (role: `Admin`)  
**Query params**: `threshold` (optional, filter by specific threshold level)  
**Response**: Array of members who have exceeded absence thresholds, sorted by days absent descending. Each entry includes: member info, days absent, last attendance date, which thresholds triggered.

### Service Logic

#### AttendanceService

Core service handling all attendance operations:

- **`recordCheckIn(dto)`**: Validates input, deduplicates, auto-matches class, applies dual-check logic, creates attendance + audit records, dispatches notifications. Returns the created attendance record.
- **`validateCheckIn(id, staffId, action)`**: Processes staff confirmation or rejection. Updates validation status, creates audit record, notifies parent.
- **`batchSync(kioskId, records[])`**: Processes offline kiosk records. Deduplicates, creates records with `offline=true`, creates sync audit entries, batches notifications.
- **`getAtRiskMembers(tenantId, threshold?)`**: Queries members by days since last attendance against configured thresholds. Joins with `MemberAbsenceAlert` to show alert history.

#### AbsenceAlertCronService

Scheduled service running daily per tenant timezone:

- **`processAbsenceAlerts()`**: Iterates all tenants, calculates absence days for each active member, compares against thresholds, creates `MemberAbsenceAlert` records, dispatches notifications via the notification system ([13-notifications](./13-notifications.md)).
- **`resetAlerts(memberId)`**: Called when a member checks in. Deletes their `MemberAbsenceAlert` records so alerts can re-trigger if they become absent again.

#### AttendanceAuditService

Append-only audit logger:

- **`log(attendanceId, eventType, eventData, actorId?)`**: Creates an immutable `AttendanceAudit` record. Never updates or deletes existing records.

---

## Kiosk App (Check-In UX)

The Kiosk App runs on iPad and is the primary attendance interface for students. The check-in flow follows a progressive fallback pattern: Face → QR → Name Search → Staff Alert.

> _For kiosk hardware setup, device management, face enrollment, and offline architecture, see [08-kiosk-app-architecture](./08-kiosk-app-architecture.md)._

### Screen: Idle / Attract Screen

**Purpose**: Attract approaching students and indicate the kiosk is ready.

- Displays gym logo/branding (fetched from tenant settings), current time, and date.
- Prominent message: **"Step up to check in!"** in the gym's default language.
- Secondary text cycles through enrolled languages (EN/KO/ES) every 5 seconds.
- Background: subtle animated gradient or gym-branded wallpaper.
- **Motion detection**: Front camera runs a low-power face detection loop. When a face is detected within the camera frame, transition to Camera/Face Recognition View.
- **Inactivity timeout**: If no interaction for 10 seconds on any subsequent screen, return to Idle.
- **Offline indicator**: If the kiosk is offline, show a small yellow dot in the corner with "Offline — check-ins will sync later" text.

### Screen: Camera / Face Recognition View

**Purpose**: Attempt face-based check-in (primary method).

- **Layout**: Full-screen camera preview from the front-facing camera.
- **Face detection overlay**: When a face is detected, draw a rounded bounding box around it. The box color indicates state:
  - 🔵 Blue: Face detected, processing...
  - 🟢 Green: Match found
  - 🔴 Red: No match
- **Processing indicator**: Spinning ring animation around the face bounding box while the on-device model computes the embedding and compares against enrolled faces.
- **Match threshold**: Cosine similarity ≥ 0.85 (configurable in kiosk settings).

**On successful match** (similarity ≥ threshold):
1. Bounding box turns green.
2. Brief haptic feedback (if iPad supports it).
3. Play confirmation chime sound (short, pleasant tone).
4. Transition to Welcome/Success screen with matched member's photo and name.
5. Log attendance locally: `method = FaceRecognition`.
6. If online: POST to attendance API immediately. If offline: queue for batch sync.
7. Dispatch parent notification (via API, or queued if offline).

**On no match** (3-second timeout with no face matching above threshold):
1. Bounding box turns red.
2. Display prompt: **"We didn't recognize you. Try scanning your QR code!"** with a large QR icon button.
3. Auto-transition to QR Code Scan View after 2 seconds, or immediately on tap.

**On no face detected** (5-second timeout with no face in frame):
1. Display: **"Step closer to the camera"** with an arrow animation.
2. If still no face after another 5 seconds: return to Idle.

### Screen: QR Code Scan View

**Purpose**: Secondary check-in method via QR code scan.

- **Layout**: Camera preview switches to rear camera (or continues front camera if no rear). Display a semi-transparent scan guide frame (rounded rectangle) in the center of the screen.
- **Prompt text**: **"Scan your QR code"** with an animated scan line moving vertically within the guide frame.
- **Supported QR formats**: Ararat-issued QR codes containing an opaque member token. The token is a signed JWT with `{ memberId, tenantId, exp }` — expiration set to 1 year, rotatable by parent.

**On successful scan**:
1. Decode QR payload, validate token signature and expiration.
2. Look up member by `memberId` from token. Verify member belongs to current tenant and is active.
3. Play confirmation chime.
4. Log attendance: `method = QRCode`.
5. Transition to Welcome/Success screen.

**On invalid QR** (unrecognized format, expired token, wrong tenant):
1. Display: **"Invalid QR code. Please try again or search by name."**
2. Show "Search by name" button.

**On no scan** (10-second timeout):
1. Display: **"No QR code detected."**
2. Show prominent button: **"Search by Name"** — transitions to Name Search View.
3. Show smaller link: **"Back to Camera"** — returns to Face Recognition View.

### Screen: Name Search View

**Purpose**: Fallback check-in for when face and QR methods fail.

- **Layout**: Top half — large search input with oversized, child-friendly keyboard (letters only, large touch targets ≥ 60pt). Bottom half — results grid.
- **Keyboard design**: 
  - Letters arranged in QWERTY layout with large keys (minimum 60×60 pt touch targets).
  - Backspace and Clear buttons prominently placed.
  - No numbers or special characters needed.
  - High-contrast colors for readability.
- **Search behavior**: As the user types (debounced 300ms), show matching members in a photo grid below.
  - Match against first name and last name (case-insensitive, starts-with matching).
  - Show member photo (circular avatar), first name, last name, and belt color indicator.
  - Maximum 12 results displayed (3×4 grid). If more matches exist, show "Type more letters to narrow results."
  - Results sourced from local on-device member cache (synced from server).

**On member tap**:
1. Show confirmation overlay: Large photo + full name + "Is this you?" with **Yes** and **No** buttons.
2. On **Yes**: Play confirmation chime. Log attendance: `method = NameSearch`. Transition to Welcome/Success screen.
3. On **No**: Return to search results.

**On no results**:
1. Display: **"No members found. Please ask staff for help."**
2. Show "Ask Staff" button — transitions to Error/Staff Alert View.

### Screen: Welcome / Success View

**Purpose**: Confirm successful check-in with positive reinforcement.

- **Layout**: Centered, full-screen celebration.
  - Large member photo (circular, 200pt diameter) with a green checkmark badge overlay.
  - Greeting: **"Welcome, [First Name]!"** in the member's preferred language.
    - English: "Welcome, [Name]!"
    - Korean: "[Name], 안녕!"
    - Spanish: "¡Bienvenido, [Name]!"
  - Belt level badge displayed below the name (colored circle matching their belt color + belt name).
  - Fun animation: Confetti particles burst from the center, green checkmark scales up with a bounce effect.
  - Confirmation chime plays (same sound for all methods — consistent positive reinforcement).
- **Dual-check mode**: If enabled, show additional text below greeting: "Staff will confirm your check-in shortly." in a subtle gray font.
- **Auto-dismiss**: Screen displays for 3 seconds, then transitions back to Idle screen.
- **Manual dismiss**: Tapping anywhere immediately returns to Idle (for busy check-in times).

### Screen: Error / Staff Alert View

**Purpose**: Handle cases where all automatic methods fail.

- **Layout**: Friendly error screen (not scary — these are children).
  - Large icon: Friendly staff character illustration or help icon.
  - Message: **"Please ask a staff member for help!"** in gym's default language.
  - Subtext: "A staff member has been notified." in smaller font.
- **Staff alert**: When this screen appears:
  1. Send push notification to all staff with `Admin` or `Instructor` role: "A student needs help checking in at [Kiosk Name]."
  2. Add alert entry to Admin App dashboard's notification center.
- **Actions**:
  - **"Try Again"** button — returns to Idle/Camera screen.
  - **"I'm a Visitor"** button — shows message: "Please ask the front desk for assistance." (visitors are not in the system).
- **Auto-dismiss**: Returns to Idle after 30 seconds of inactivity.

---

## Parent/Member App

### Screen: Child Detail — Attendance Tab

**Route**: `/children/:id` → Attendance tab  
**Auth**: Parent JWT (own children only)  
**Data source**: `GET /api/v1/tenants/{tenantId}/members/{id}/attendance`

#### Calendar Heatmap

- Monthly calendar view (current month by default, swipeable to navigate months).
- Each day cell is color-coded:
  - **Green** (filled): Check-in recorded that day.
  - **Light gray**: No check-in (weekday — potential absence).
  - **Dark gray / hatched**: No class scheduled (weekend, holiday, gym closed).
- Tapping a green day expands to show check-in details for that day.

#### Attendance Record List

Below the calendar, a chronological list (most recent first) of attendance records:

| Column | Content |
|--------|---------|
| Date | Formatted date (e.g., "Mon, Feb 23") |
| Time | Check-in time (e.g., "4:32 PM") |
| Method | Icon + label (📷 Face, 📱 QR, 🔍 Search, 👤 Staff) |
| Class | Class name or "Open Gym" if no class matched |

- Pull-to-refresh to get latest data.
- Infinite scroll for history (paginated via API).

#### Absence Alert Indicator

- If the child has an active absence alert (crossed a threshold), show a banner at the top of the Attendance tab:
  - **Yellow banner**: "It's been [N] days since [Child Name]'s last visit."
  - Tapping the banner shows a reassuring message and gym contact info.

#### Stats Summary

At the top of the tab, show summary stats:
- **This month**: X visits
- **Current streak**: X days
- **Longest streak**: X days

### Screen: Dashboard — Attendance Summary Widget

**Route**: `/` (Parent dashboard)  
**Data source**: `GET /api/v1/tenants/{tenantId}/attendance/today` (filtered to parent's children)

Widget card on the parent dashboard showing today's attendance status for each child:

- **Child photo** (small avatar) + **name**.
- **Status**: 
  - ✅ "Checked in at 4:15 PM" (green text) — if checked in today.
  - ⏳ "Not checked in yet" (gray text) — if not checked in.
- If checked in: show method icon and class name.
- Tapping the widget navigates to the Child Detail Attendance Tab.

---

## Admin App

### Screen: Attendance Dashboard

**Route**: `/attendance`  
**Auth**: Staff JWT (role: `Admin`, `Instructor`)

#### Today's Attendance Section

**Data source**: `GET /api/v1/tenants/{tenantId}/attendance/today`  
**Refresh**: Auto-refreshes every 30 seconds (`refetchInterval: 30000`).

- **Header**: "Today's Attendance" with total count badge (e.g., "Today's Attendance (23)").
- **Live list**: Table/card view of members checked in today:

| Column | Content |
|--------|---------|
| Photo | Member avatar (40px circular) |
| Name | Full name (linked to member profile) |
| Time | Check-in time |
| Method | Icon badge (Face/QR/Name/Manual) |
| Class | Class name or "—" |
| Status | Validation badge: ✅ Confirmed, ⏳ Pending, ⚠️ Unconfirmed |

- **Pending validations** are highlighted with a yellow background and shown at the top of the list.
- Each pending row has a **Confirm** button (green) and **Reject** button (red).
- Clicking Confirm: calls `PATCH .../validate` with `action: "confirm"`.
- Clicking Reject: opens a modal for optional rejection reason, then calls validate with `action: "reject"`.

**Manual Check-In Action**:
- Button: "+ Manual Check-In" in the header.
- Opens a modal with member search (typeahead). Staff selects member, optionally selects class, confirms.
- Calls `POST .../attendance` with `method: StaffManual`.

**Manual Check-Out Action**:
- Each checked-in row has a "Check Out" action (in the row's action menu).
- Sets `checked_out_at` on the attendance record.
- Creates `AttendanceAudit` with `event_type = CheckOut`.

#### Historical Attendance Section

**Data source**: `GET /api/v1/tenants/{tenantId}/attendance`

- **Filters bar**: Date range picker (default: last 7 days), class dropdown, member search, method dropdown.
- **Table view**: Same columns as today's view, plus date column.
- **Pagination**: 50 records per page with page controls.
- **Export**: "Export CSV" button downloads filtered results as CSV file. Columns: Date, Time, Member Name, Method, Class, Status.

#### Audit Trail Section

**Data source**: `GET /api/v1/tenants/{tenantId}/attendance/audit`

- **Access**: `Admin` role only (not `Instructor`).
- **Tab or collapsible section** within the Attendance page.
- **Timeline view**: Chronological list of audit events with:
  - Event type badge (color-coded: CheckIn=blue, Validation=green, ManualOverride=orange, Correction=red)
  - Timestamp
  - Actor (staff name or "System")
  - Description (human-readable summary of what changed)
  - Expandable detail (raw event_data JSON for debugging)
- **Filters**: Date range, event type, member.
- Used for investigating disputes, corrections, and compliance audits.

### Screen: Dashboard — At-Risk Members Widget

**Route**: `/` (Admin dashboard)  
**Data source**: `GET /api/v1/tenants/{tenantId}/attendance/at-risk`

Dashboard widget showing members who have exceeded absence thresholds:

- **Header**: "At-Risk Members" with count badge.
- **List**: Members ranked by days absent (most absent first).
  - Member avatar + name.
  - "Last seen: [date]" with days-absent count in bold.
  - Threshold indicator: colored dot (🟡 3 days, 🟠 7 days, 🔴 14 days).
- **Filter toggle**: Tabs or dropdown to filter by threshold level (All, 3+, 7+, 14+).
- **Quick actions** per member:
  - **"Send Message"**: Opens a pre-filled message modal to contact the parent (via notification system).
  - **"View Profile"**: Navigates to member profile page.
- If no at-risk members: show a positive empty state ("All members are active! 🎉").

---

## Monitor App

### Display: Live Attendance Panel

**Position**: Right panel of the monitor display layout (40% width). See [18-monitor-app](./18-monitor-app.md) for full layout spec.  
**Data source**: `GET /api/v1/tenants/{tenantId}/attendance/today`  
**Refresh**: Polls every 15 seconds.

#### Layout

- **Panel header**: "Today's Attendance" with total count (e.g., "오늘 출석 (23)") — displayed in gym's default language.
- **Scrolling list**: Members checked in today, sorted by most recent first.
- Each entry is a horizontal row:
  - **Member photo**: Circular avatar (48px).
  - **Name**: Member's display name (first name + last initial for privacy on public display).
  - **Check-in time**: Formatted as "4:32 PM".
  - **Belt indicator**: Small colored circle matching the member's current belt.

#### Animations

- **New check-in**: When a new record appears in the poll response, the entry animates in from the top with a slide-down + fade-in effect (300ms ease-out).
- **Auto-scroll**: The list automatically scrolls through all entries at a steady pace (1 entry per 3 seconds). Pauses for 5 seconds when a new check-in animates in, then resumes scrolling.
- **Transition**: When the list reaches the end, it smoothly scrolls back to the top and repeats.

#### Privacy Considerations

- Only first name + last initial displayed (e.g., "Minji K." not "Minji Kim") — configurable in gym settings.
- No check-in method or validation status shown on public display.
- Photos can be disabled in gym settings (replaced with generic belt-colored avatar).

---

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| Notification Service | Internal (TRD 13) | Dispatch check-in confirmations, absence alerts | Internal call | `{ recipientId, templateId, data }` | `{ notificationId }` |
| Member Service | Internal (TRD 09) | Validate member exists and is active | Internal call | `{ memberId }` | `{ member }` |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/tenants/{tenantId}/attendance/check-in` | POST | Kiosk App, Admin App | `{ memberId, method, classId }` | Check-in confirmation |
| `/api/v1/tenants/{tenantId}/attendance` | GET | Admin App, Parent App | `?date=&classId=&memberId=` | Attendance records |
| `/api/v1/tenants/{tenantId}/attendance/{id}/confirm` | POST | Admin App | `{ confirmed }` | Confirmation status |
| `/api/v1/tenants/{tenantId}/attendance/audit` | GET | Admin App | `?date=&memberId=` | Audit trail entries |
| `/api/v1/tenants/{tenantId}/alert-rules` | GET/POST/PATCH | Admin App | Alert rule config JSON | Alert rule list or detail |

---


## Phase 2 Features (Not Yet Specified)

The following features are identified in the PRD for Phase 2 and will be fully specified before implementation:

### Group Check-In
- **Use case**: Instructor checks in multiple students at once (e.g., entire class arrival)
- **Admin App UX**: Class roster view with multi-select checkboxes, "Check in all" button
- **Audit trail**: Group check-ins recorded with `method: 'group'` and `performed_by` instructor ID
- **Validation**: Cannot group-check-in students not enrolled in the class

### NFC Check-In
- **Use case**: Students tap NFC-enabled ID card on a reader device for attendance
- **Hardware**: Compatible NFC reader connected to Kiosk iPad or standalone reader
- **Flow**: NFC tag read → match to member record → check-in recorded with `method: 'nfc'`
- **Kiosk integration**: Extends existing Kiosk check-in flow (alongside face recognition and PIN)
- **Fallback**: If NFC read fails, student can use PIN or face recognition

---

## Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
