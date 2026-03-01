# 13. Feature: Notification & Communication

**Related TRDs**: [06-i18n](./06-i18n.md), [10-attendance](./10-attendance.md), [11-payments](./11-payments.md), [12-simsa](./12-simsa.md), [14-newsletter](./14-newsletter.md), [15-parent-feed](./15-parent-feed.md)  
**Related ADRs**: [ADR-005](./adr/005-messenger-agnostic-adapter.md)  
**Phase**: MVP (Phase 1)

---

### Overview

Multi-channel notification system with templating, an alert rules engine, and a messenger adapter pattern. The system handles event-driven dispatch across five channels (Push, SMS, Email, In-App, Messenger), supports trilingual templates with personalization tokens, and provides configurable alert rules that escalate unacknowledged alerts to admin tasks. Notifications originate from domain events in attendance, payments, 심사, newsletters, and the activity feed — this TRD defines the dispatch infrastructure, not the triggering business rules (see referenced TRDs).

---

### Business Rules

#### Multi-Channel Routing

For each notification type, the system determines which channels to use. Gym owner can customize channels per notification type in settings.

| Notification Type | Push | SMS | Email | In-App | Messenger |
|-------------------|------|-----|-------|--------|-----------|
| Arrival Confirmation | ✓ | ✓ | | ✓ | |
| Class Summary | ✓ | | | ✓ | |
| Activity Photo | ✓ | | | ✓ | |
| Belt Promotion | ✓ | | ✓ | ✓ | |
| Upcoming Simsa | ✓ | | ✓ | ✓ | |
| Attendance Streak | ✓ | | | ✓ | |
| Payment Receipt | | | ✓ | ✓ | |
| Payment Due | ✓ | ✓ | ✓ | ✓ | |
| Membership Expiry | ✓ | | ✓ | ✓ | |
| Schedule Change | ✓ | ✓ | | ✓ | |
| Gym Announcement | ✓ | | | ✓ | |

#### Notification Dispatch Flow

1. **Event occurs** — a domain event is emitted (e.g., member checks in, payment fails, 심사 scheduled).
2. **NotificationService receives event** — determines notification type and resolves recipients.
3. **Template resolution** — for each recipient, fetch their preferred language and load the matching `NotificationTemplate`.
4. **Template rendering** — render the template body (and subject for email) with personalization tokens.
5. **Channel routing** — determine which channels to use based on gym settings + recipient preferences.
6. **Dispatch** — send to each channel via its adapter:
   - **Push**: Firebase Cloud Messaging (FCM).
   - **SMS**: Twilio.
   - **Email**: SendGrid.
   - **In-App**: Create `Notification` record in database, display in app.
   - **Messenger**: Route through the messenger adapter (KakaoTalk or future providers).
7. **Logging** — record delivery status per channel in `NotificationLog`.

#### Alert Rules Engine

##### Data Model

```
AlertRule:
  - rule_id (UUID)
  - tenant_id (UUID, FK)
  - trigger_type (Absence | LatePayment | MembershipExpiry | SimsaDeadline)
  - thresholds (JSON array, e.g., [3, 7, 14] for days)
  - template_id (UUID, FK → NotificationTemplate)
  - channels (JSON array, e.g., ["Push", "SMS", "Email"])
  - escalation_config (JSON, e.g., { "consecutive_ignores": 2, "escalate_to_admin": true })
  - enabled (Boolean)
```

##### Trigger Types

- **Absence**: Days since last attendance crosses threshold.
- **LatePayment**: Payment overdue by N days.
- **MembershipExpiry**: Membership expires in N days.
- **SimsaDeadline**: 심사 exam in N days and member not yet registered.

##### Escalation

If a member receives 2 consecutive alerts without responding (no action taken):
1. System auto-creates admin `Task`: "Follow up with [member_name] — [reason]".
2. Task assigned to gym owner.
3. Task appears in admin dashboard.

##### Evaluation Cron

A scheduled job runs daily (configurable) to evaluate all enabled alert rules:
1. For each enabled `AlertRule`, query members matching the trigger condition.
2. For members crossing any threshold, check if a notification was already sent for that threshold level.
3. If not yet sent, dispatch notification using the rule's template and channels.
4. Track consecutive ignores per member per rule for escalation logic.

#### Template System

##### NotificationTemplate Model

```
NotificationTemplate:
  - template_id (UUID)
  - tenant_id (UUID, FK)
  - type (ArrivalConfirmation | ClassSummary | ActivityPhoto | BeltPromotion |
          UpcomingSimsa | AttendanceStreak | PaymentReceipt | PaymentDue |
          MembershipExpiry | ScheduleChange | GymAnnouncement | Broadcast)
  - language (en | ko | es)
  - subject (String, for email channel)
  - body (String, template with personalization tokens)
  - channels (JSON array, default channels for this type)
```

##### Personalization Tokens

Available tokens rendered from event context:

| Token | Description |
|-------|-------------|
| `{{member_name}}` | Child's name |
| `{{parent_name}}` | Parent's name |
| `{{gym_name}}` | Gym name |
| `{{days_absent}}` | Days since last attendance |
| `{{balance_due}}` | Outstanding balance (formatted as USD) |
| `{{belt_level}}` | Current belt level |
| `{{simsa_date}}` | Exam date |
| `{{check_in_time}}` | Check-in timestamp |
| `{{class_name}}` | Class name |
| `{{instructor_name}}` | Instructor name |
| `{{renewal_date}}` | Membership renewal date |
| `{{action_url}}` | Deep link to relevant screen |

##### Admin Customization

Admin can customize templates via the template editor:
1. Admin selects notification type and language.
2. Admin edits subject and body with token placeholders.
3. Admin can preview with sample data.
4. Admin saves — change logged in `AuditLog`.

#### Third-Party Messenger Adapter Pattern

##### Interface

```typescript
interface MessengerAdapter {
  send(recipientId: string, message: string, metadata: object): Promise<DeliveryStatus>;
  validate(config: object): Promise<boolean>;
}

interface DeliveryStatus {
  status: 'sent' | 'delivered' | 'failed';
  externalId: string;
  error?: string;
}
```

##### Implementations

- **KakaoTalkAdapter**: Uses KakaoTalk Business API to send messages to users who have linked their KakaoTalk account.
- **WhatsAppAdapter** (future): Uses WhatsApp Business API.
- **LineAdapter** (future): Uses LINE Messaging API.

##### Configuration

Gym owner configures messenger in admin settings:
1. Select primary messenger provider (KakaoTalk, WhatsApp, etc.).
2. Enter API credentials (API key, business account ID).
3. System validates credentials via `adapter.validate()`.
4. Credentials stored encrypted in `SystemSetting`.

##### Recipient Preference

Parent selects preferred messenger in app settings:
1. Parent visits Settings → Notification Channel.
2. Parent selects preferred channel (Push, SMS, Email, KakaoTalk, etc.).
3. System stores preference in parent's profile.
4. When dispatching: use parent's preferred messenger if available; fall back to SMS/email.

---

### Backend

#### API Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/notifications` | List notifications for current user (in-app inbox) | Parent, Admin |
| `PATCH` | `/api/v1/tenants/{tenantId}/notifications/{id}/read` | Mark a single notification as read | Parent, Admin |
| `POST` | `/api/v1/tenants/{tenantId}/notifications/read-all` | Mark all notifications as read | Parent, Admin |
| `GET` | `/api/v1/tenants/{tenantId}/notification-history` | List send history (all recipients, all channels) | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/notification-templates` | List all notification templates | Admin |
| `PATCH` | `/api/v1/tenants/{tenantId}/notification-templates/{id}` | Update a template (subject, body, channels) | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/alert-rules` | List all alert rules | Admin |
| `PATCH` | `/api/v1/tenants/{tenantId}/alert-rules/{id}` | Update an alert rule (thresholds, channels, enabled) | Admin |
| `POST` | `/api/v1/tenants/{tenantId}/notifications/broadcast` | Send a broadcast notification | Admin |

**Query parameters** for `GET /notifications`:
- `filter` — notification type filter (all, attendance, payments, announcements)
- `status` — read/unread
- Standard pagination (`page`, `limit`)

**Query parameters** for `GET /notification-history`:
- `type` — notification type
- `channel` — delivery channel
- `status` — sent/delivered/failed
- `from`, `to` — date range
- Standard pagination (`page`, `limit`)

**Request body** for `POST /notifications/broadcast`:
```json
{
  "audience": {
    "type": "all" | "class" | "belt" | "custom",
    "ids": ["uuid"]
  },
  "channels": ["Push", "SMS", "Email"],
  "subject": "string",
  "body": "string"
}
```

#### Service Logic

- **NotificationService**: Core orchestrator — receives domain events, resolves recipients and templates, routes to channel adapters. Implements retry with exponential backoff for failed external dispatches (max 3 retries).
- **TemplateService**: Loads templates by type + language, renders personalization tokens, validates token completeness.
- **AlertRuleService**: Evaluates alert rules on cron schedule, tracks threshold crossings and consecutive ignores, creates admin tasks on escalation.
- **Channel adapters**: `FcmAdapter` (push), `TwilioAdapter` (SMS), `SendGridAdapter` (email), `InAppAdapter` (database), `MessengerAdapter` (KakaoTalk / future providers).
- **NotificationLog**: Append-only log of every dispatch attempt — channel, status, external ID, error, timestamp. Used for history queries and retry logic.

---

### Parent/Member App

#### Screen: Notifications (`/notifications`)

Chronological list of all notifications received — attendance confirmations, payment reminders, 심사 announcements, system alerts. Unread items are highlighted.

**Key components**:
- Notification list with `Card` per item: icon by notification type, title, preview text, timestamp, read/unread indicator
- Filter `Tabs`: All, Attendance, Payments, Announcements

**Data sources**: `useNotifications(filter)` — paginated, real-time via polling or WebSocket for new notification badge count.

**User actions**:
- Tap a notification to view detail (navigates to relevant screen — e.g., tap attendance notification → attendance history)
- Mark notification as read (optimistic update)
- Mark all as read
- Filter by notification type

**Empty state**: "You're all caught up!" with checkmark illustration.

**Badge**: Unread count displayed on the Notifications tab in bottom navigation.

#### Screen: Settings — Notification Preferences (`/settings`)

Within the Settings screen, notification preferences section:

**Sections**:
- **Notification Toggles**: `Switch` toggle for each notification type:
  - Attendance alerts
  - Payment reminders
  - 심사 announcements
  - Newsletters
  - Activity feed updates
- **Preferred Channel**: `RadioGroup` selection — Push, SMS, Email, KakaoTalk

**Data sources**: `useUserSettings()`.

**User actions**:
- Toggle individual notification types on/off
- Select preferred notification channel
- Save changes

---

### Admin App

#### Screen: Notifications Management (`/notifications`)

Tabbed interface for managing all aspects of the notification system.

##### Tab: Send History

**What it shows**: Full log of all sent notifications across all channels and recipients.

**Key components**:
- TanStack Table with columns: Recipient, Type, Channel, Status (sent/delivered/failed with color badges), Timestamp
- Filter bar: type dropdown, channel dropdown, status dropdown, date range picker
- Row action: "Resend" button for failed notifications

**Data sources**: `useNotificationHistory(filters)`.

**User actions**:
- View sent notification history with pagination
- Filter by type, channel, status, date range
- Resend a failed notification

##### Tab: Templates

**What it shows**: All notification templates with per-language editing.

**Key components**:
- Template list grouped by notification type
- Template editor with language tabs (EN / KO / ES)
- Subject field (for email templates)
- Body field with token insertion toolbar (`{{member_name}}`, `{{gym_name}}`, etc.)
- Preview panel with sample data rendering

**Data sources**: `useNotificationTemplates()`.

**User actions**:
- Select a template type to edit
- Switch between language tabs
- Edit subject and body with token placeholders
- Preview template with sample data
- Save template (logged in AuditLog)

##### Tab: Alert Rules

**What it shows**: Configurable automatic notification rules.

**Key components**:
- Alert rule `Card` per rule: trigger type label, threshold values, template name, channel badges, `Switch` toggle (enabled/disabled)
- Edit dialog: threshold inputs, template selector, channel checkboxes, escalation config

**Data sources**: `useAlertRules()`.

**User actions**:
- View all alert rules
- Toggle rule enabled/disabled via `Switch`
- Edit rule: thresholds, template, channels, escalation settings
- Create new alert rule

##### Broadcast Action

- "Send Broadcast" `Button` in page header
- Opens compose dialog:
  - Audience selector (all members, by class, by belt level, custom selection)
  - Channel checkboxes (Push, SMS, Email)
  - Subject and body fields
  - Preview before send
  - Confirm and send

**Data sources**: `POST /notifications/broadcast`.

---

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
