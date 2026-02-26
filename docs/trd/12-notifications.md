# 12. Feature: Notification & Communication

**Related TRDs**: [06-i18n](./06-i18n.md), [08-attendance](./08-attendance.md), [10-payments](./10-payments.md), [11-simsa](./11-simsa.md), [13-newsletter](./13-newsletter.md), [14-parent-feed](./14-parent-feed.md)  
**Related ADRs**: [ADR-005](./adr/005-messenger-agnostic-adapter.md)  
**Phase**: MVP (Phase 1)

---

### Multi-Channel Routing

For each notification type, system determines which channels to use:

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

Gym owner can customize channels per notification type in settings.

### Notification Dispatch Flow

1. Event occurs (e.g., member checks in).
2. NotificationService receives event.
3. Service determines notification type and recipients.
4. For each recipient:
   - Fetch recipient's preferred language.
   - Fetch NotificationTemplate for this type in that language.
   - Render template with personalization tokens.
   - Determine which channels to use (from gym settings + recipient preferences).
   - Create Notification record.
   - Dispatch to each channel via adapter:
     - **Push**: Firebase Cloud Messaging (FCM).
     - **SMS**: Twilio.
     - **Email**: SendGrid.
     - **In-App**: Store in database, display in app.
     - **Messenger**: KakaoTalk Business API (or other provider).
5. Log delivery status in NotificationLog.

### Alert Rules Engine

#### Data Model

```
AlertRule:
  - rule_id
  - tenant_id
  - trigger_type (Absence, LatePayment, MembershipExpiry, SimsaDeadline)
  - thresholds (JSON array, e.g., [3, 7, 14] for days)
  - template_id (NotificationTemplate to use)
  - channels (JSON array, e.g., [Push, SMS, Email])
  - escalation_config (JSON, e.g., { consecutive_ignores: 2, escalate_to_admin: true })
  - enabled (Boolean)
```

#### Trigger Types

- **Absence**: Days since last attendance crosses threshold.
- **LatePayment**: Payment overdue by N days.
- **MembershipExpiry**: Membership expires in N days.
- **SimsaDeadline**: Simsa exam in N days and member not yet registered.

#### Escalation

If member receives 2 consecutive alerts without responding (no action taken):
1. System auto-creates admin Task: "Follow up with [member_name] - [reason]".
2. Task assigned to gym owner.
3. Task appears in admin dashboard.

### Template System

#### Template Storage

```
NotificationTemplate:
  - template_id
  - tenant_id
  - type (ArrivalConfirmation, ClassSummary, etc.)
  - language (English, Korean, Spanish)
  - subject (for email)
  - body (template with personalization tokens)
  - channels (default channels for this type)
```

#### Personalization Tokens

Available tokens (rendered based on context):
- `{{member_name}}` - Child's name
- `{{parent_name}}` - Parent's name
- `{{gym_name}}` - Gym name
- `{{days_absent}}` - Days since last attendance
- `{{balance_due}}` - Outstanding balance
- `{{belt_level}}` - Current belt level
- `{{simsa_date}}` - Exam date
- `{{check_in_time}}` - Check-in timestamp
- `{{class_name}}` - Class name
- `{{instructor_name}}` - Instructor name
- `{{renewal_date}}` - Membership renewal date

#### Admin Customization

Admin can customize templates via template editor:
1. Admin selects notification type and language.
2. Admin edits subject and body.
3. Admin can preview with sample data.
4. Admin saves template.
5. Change logged in AuditLog.

### Third-Party Messenger Adapter Pattern

#### Interface

```
interface MessengerAdapter {
  send(recipientId: string, message: string, metadata: object): Promise<DeliveryStatus>
  validate(config: object): Promise<boolean>
}

interface DeliveryStatus {
  status: 'sent' | 'delivered' | 'failed'
  externalId: string
  error?: string
}
```

#### Implementation

- **KakaoTalkAdapter**: Uses KakaoTalk Business API to send messages to users who have linked their KakaoTalk account.
- **WhatsAppAdapter** (future): Uses WhatsApp Business API.
- **LineAdapter** (future): Uses LINE Messaging API.

#### Configuration

Gym owner configures messenger in settings:
1. Select primary messenger (KakaoTalk, WhatsApp, etc.).
2. Enter API credentials (API key, business account ID, etc.).
3. System validates credentials.
4. Credentials stored encrypted in SystemSetting.

#### Recipient Preference

Parent selects preferred messenger in app settings:
1. Parent visits settings.
2. Parent selects "Preferred Messenger" (SMS, Email, KakaoTalk, WhatsApp, etc.).
3. System stores preference in Parent record.
4. When sending notifications: use parent's preferred messenger if available, fall back to SMS/email.


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
