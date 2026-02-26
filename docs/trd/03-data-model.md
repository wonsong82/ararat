# 3. Data Model

**Related TRDs**: [01-system-architecture](./01-system-architecture.md), [02-multi-tenancy](./02-multi-tenancy.md), [04-auth](./04-auth.md)  
**Related ADRs**: [ADR-003](./adr/003-multi-tenant-architecture.md)  
**Phase**: MVP (Phase 1)

---

### Entity Relationship Diagram

```mermaid
erDiagram
    TENANT ||--o{ USER : has
    TENANT ||--o{ MEMBER : has
    TENANT ||--o{ CLASS : has
    TENANT ||--o{ STAFF : has
    TENANT ||--o{ BELT : has
    TENANT ||--o{ SIMSA : has
    TENANT ||--o{ MEMBERSHIP_PLAN : has
    TENANT ||--o{ PAYMENT : has
    TENANT ||--o{ NOTIFICATION_TEMPLATE : has
    TENANT ||--o{ SYSTEM_SETTING : has

    USER ||--o{ PARENT : is
    USER ||--o{ ADMIN : is
    USER ||--o{ MEMBER : is

    PARENT ||--o{ PARENT_CHILD : has
    MEMBER ||--o{ PARENT_CHILD : has

    MEMBER ||--o{ MEMBER_BELT : has
    BELT ||--o{ MEMBER_BELT : has

    MEMBER ||--o{ ATTENDANCE : has
    CLASS ||--o{ ATTENDANCE : has
    STAFF ||--o{ ATTENDANCE : validates

    MEMBER ||--o{ SIMSA_REGISTRATION : registers_for
    SIMSA ||--o{ SIMSA_REGISTRATION : has
    SIMSA_REGISTRATION ||--o{ SIMSA_RESULT : has

    MEMBER ||--o{ MEMBERSHIP : has
    MEMBERSHIP_PLAN ||--o{ MEMBERSHIP : subscribes_to

    MEMBER ||--o{ PAYMENT : makes
    PAYMENT ||--o{ INVOICE : generates

    MEMBER ||--o{ NOTIFICATION : receives
    NOTIFICATION_TEMPLATE ||--o{ NOTIFICATION : uses

    CLASS ||--o{ CLASS_SCHEDULE : has
    CLASS ||--o{ CLASS_ENROLLMENT : has
    MEMBER ||--o{ CLASS_ENROLLMENT : enrolls_in

    MEMBER ||--o{ ACTIVITY_FEED_POST : creates
    ACTIVITY_FEED_POST ||--o{ ACTIVITY_FEED_PHOTO : contains
    ACTIVITY_FEED_POST ||--o{ ACTIVITY_FEED_REACTION : receives

    MEMBER ||--o{ AUDIT_LOG : triggers
    USER ||--o{ AUDIT_LOG : performs

    MEMBER ||--o{ USER_SESSION : has
```

### Core Entities

#### Tenant
Represents a single Taekwondo gym (the customer).

**Key Fields**:
- `tenant_id` (UUID, PK)
- `name` (String) - Gym name
- `email` (String) - Primary contact email
- `phone` (String) - Primary contact phone
- `address` (String) - Physical address
- `timezone` (String) - Timezone for scheduling
- `status` (Enum: Active, Suspended, Deleted) - Gym account status
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### User
Represents any user in the system (parent, admin, member, staff).

**Key Fields**:
- `user_id` (UUID, PK)
- `tenant_id` (UUID, FK) - Scopes user to a gym
- `email` (String, nullable) - Email address (required for admins, optional for parents)
- `phone` (String, nullable) - Phone number (required for parents, optional for admins)
- `password_hash` (String, nullable) - Hashed password (for admin/staff)
- `role` (Enum: Owner, Manager, Instructor, Parent, Member) - Primary role
- `language` (Enum: English, Korean, Spanish) - Preferred language
- `status` (Enum: Active, Inactive, Suspended) - Account status
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Member
Represents a student/trainee enrolled at the gym.

**Key Fields**:
- `member_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `user_id` (UUID, FK) - Links to User record (if member has own account)
- `first_name` (String)
- `last_name` (String)
- `date_of_birth` (Date)
- `gender` (Enum: Male, Female, Other)
- `phone` (String, nullable)
- `email` (String, nullable)
- `profile_photo_url` (String, nullable) - URL to profile photo in S3
- `health_info` (JSON) - { allergies: [], medical_conditions: [], emergency_contact: {} }
- `status` (Enum: Pending, Active, Inactive, Withdrawn) - Registration/membership status
- `current_belt_id` (UUID, FK) - Current belt level
- `age_group` (String) - Auto-assigned based on DOB (e.g., "Little Kids", "Teens")
- `class_group` (String) - Assigned class group (e.g., "Beginner", "Advanced")
- `join_date` (Date)
- `withdrawal_date` (Date, nullable)
- `withdrawal_reason` (String, nullable)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Parent
Represents a parent/guardian account.

**Key Fields**:
- `parent_id` (UUID, PK)
- `user_id` (UUID, FK) - Links to User record
- `tenant_id` (UUID, FK)
- `first_name` (String)
- `last_name` (String)
- `phone` (String) - Primary contact phone
- `email` (String, nullable)
- `preferred_messenger` (Enum: SMS, Email, KakaoTalk, WhatsApp, etc.) - Preferred communication channel
- `notification_preferences` (JSON) - { push: true, sms: true, email: true, inApp: true, messenger: false }
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### ParentChild
Junction table linking parents to their children.

**Key Fields**:
- `parent_child_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `parent_id` (UUID, FK)
- `member_id` (UUID, FK)
- `relationship` (Enum: Mother, Father, Guardian, Other)
- `created_at` (Timestamp)

#### Staff
Represents an instructor or staff member.

**Key Fields**:
- `staff_id` (UUID, PK)
- `user_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `first_name` (String)
- `last_name` (String)
- `role` (Enum: Owner, Manager, Instructor)
- `phone` (String)
- `email` (String)
- `bio` (String, nullable)
- `photo_url` (String, nullable)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Class
Represents a class offering (e.g., "Monday Evening Beginner").

**Key Fields**:
- `class_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `name` (String) - Class name
- `description` (String, nullable)
- `instructor_id` (UUID, FK) - Primary instructor
- `capacity` (Integer) - Max students
- `status` (Enum: Active, Inactive, Archived)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### ClassSchedule
Represents recurring schedule for a class.

**Key Fields**:
- `schedule_id` (UUID, PK)
- `class_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `day_of_week` (Enum: Monday - Sunday)
- `start_time` (Time) - HH:MM format
- `end_time` (Time)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### ClassEnrollment
Represents a member's enrollment in a class.

**Key Fields**:
- `enrollment_id` (UUID, PK)
- `class_id` (UUID, FK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `status` (Enum: Active, Paused, Withdrawn)
- `enrolled_date` (Date)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Attendance
Represents a single check-in event.

**Key Fields**:
- `attendance_id` (UUID, PK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `class_id` (UUID, FK, nullable) - Class attended (if known)
- `check_in_time` (Timestamp) - When the member checked in
- `check_out_time` (Timestamp, nullable) - When the member left
- `method` (Enum: FaceRecognition, QRCode, NFC, NameSearch, Manual) - How they checked in
- `kiosk_id` (String, nullable) - Which kiosk device recorded the check-in
- `staff_validator_id` (UUID, FK, nullable) - Staff member who validated (if dual-check enabled)
- `validation_status` (Enum: Confirmed, PendingValidation, Unconfirmed) - Dual-check status
- `validation_time` (Timestamp, nullable) - When staff validated
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### AttendanceAudit
Immutable audit log for attendance events (append-only).

**Key Fields**:
- `audit_id` (UUID, PK)
- `attendance_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `member_id` (UUID, FK)
- `event_type` (Enum: CheckIn, Validation, Unconfirmed, Sync) - What happened
- `event_data` (JSON) - Full event details (method, kiosk, staff, etc.)
- `created_at` (Timestamp) - Immutable

#### Belt
Represents a belt level in the gym's progression system.

**Key Fields**:
- `belt_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `name` (String) - Belt name (e.g., "White", "Yellow", "Black")
- `order` (Integer) - Progression order (0 = lowest, N = highest)
- `color` (String, nullable) - Hex color code for UI display
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### MemberBelt
Tracks belt history for a member (one record per belt level achieved).

**Key Fields**:
- `member_belt_id` (UUID, PK)
- `member_id` (UUID, FK)
- `belt_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `achieved_date` (Date) - When the member achieved this belt
- `simsa_result_id` (UUID, FK, nullable) - Links to the simsa result that promoted them
- `is_current` (Boolean) - True if this is the member's current belt
- `created_at` (Timestamp)

#### Simsa
Represents a belt promotion exam event.

**Key Fields**:
- `simsa_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `name` (String) - Exam name (e.g., "Spring 2026 Belt Test")
- `scheduled_date` (Date)
- `scheduled_time` (Time)
- `location` (String, nullable)
- `eligible_belt_ids` (UUID Array) - Which belt levels can test
- `status` (Enum: Scheduled, NotificationSent, FormCollecting, Registered, Testing, ResultsEntered, Completed)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### SimsaRegistration
Represents a member's registration for a specific simsa event.

**Key Fields**:
- `registration_id` (UUID, PK)
- `simsa_id` (UUID, FK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `status` (Enum: Eligible, NotifiedParent, FormSubmitted, FeePaid, Testing, Completed)
- `form_submission_date` (Date, nullable)
- `fee_amount` (Decimal) - USD amount charged
- `payment_id` (UUID, FK, nullable) - Links to Payment record
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### SimsaResult
Represents the outcome of a simsa for a member.

**Key Fields**:
- `result_id` (UUID, PK)
- `registration_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `member_id` (UUID, FK)
- `passed` (Boolean) - True if member passed
- `score` (Integer, nullable) - Numeric score if applicable
- `notes` (String, nullable) - Admin notes
- `result_date` (Date)
- `new_belt_id` (UUID, FK, nullable) - Belt achieved if passed
- `certificate_url` (String, nullable) - URL to generated certificate PDF
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Membership
Represents a member's subscription to a membership plan.

**Key Fields**:
- `membership_id` (UUID, PK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `plan_id` (UUID, FK)
- `status` (Enum: Active, GracePeriod, Suspended, Cancelled)
- `start_date` (Date)
- `renewal_date` (Date) - Next billing date
- `end_date` (Date, nullable) - When membership ends
- `stripe_subscription_id` (String) - Stripe subscription ID
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### MembershipPlan
Represents a membership tier offered by the gym.

**Key Fields**:
- `plan_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `name` (String) - Plan name (e.g., "Monthly Unlimited")
- `description` (String, nullable)
- `price` (Decimal) - USD per billing cycle
- `billing_cycle` (Enum: Monthly, Annual)
- `trial_days` (Integer) - Free trial period (0 = no trial)
- `status` (Enum: Active, Inactive)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### MembershipHold
Represents a temporary hold on a membership (e.g., vacation, illness).

**Key Fields**:
- `hold_id` (UUID, PK)
- `membership_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `reason` (String) - Why the hold was placed
- `start_date` (Date)
- `end_date` (Date)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Payment
Represents a payment transaction.

**Key Fields**:
- `payment_id` (UUID, PK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `amount` (Decimal) - USD
- `currency` (String) - Always "USD"
- `type` (Enum: Membership, Simsa, Equipment, Event, Other)
- `status` (Enum: Pending, Completed, Failed, Refunded)
- `stripe_payment_intent_id` (String) - Stripe reference
- `payment_method` (Enum: CreditCard, ACH, ApplePay, GooglePay)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Invoice
Represents a billing invoice (may contain multiple line items).

**Key Fields**:
- `invoice_id` (UUID, PK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `invoice_number` (String) - Human-readable invoice ID
- `status` (Enum: Draft, Sent, Paid, Overdue, Cancelled)
- `subtotal` (Decimal)
- `tax_amount` (Decimal)
- `discount_amount` (Decimal)
- `total_amount` (Decimal)
- `due_date` (Date)
- `paid_date` (Date, nullable)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### InvoiceLineItem
Represents a single line item on an invoice.

**Key Fields**:
- `line_item_id` (UUID, PK)
- `invoice_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `description` (String) - What was charged (e.g., "Monthly Membership", "Belt Test Fee")
- `quantity` (Integer)
- `unit_price` (Decimal)
- `total_price` (Decimal)
- `created_at` (Timestamp)

#### Refund
Represents a refund issued to a member.

**Key Fields**:
- `refund_id` (UUID, PK)
- `payment_id` (UUID, FK)
- `member_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `amount` (Decimal) - USD refunded
- `reason` (Enum: Withdrawal, Overpayment, Dispute, Other)
- `status` (Enum: Pending, Completed, Failed)
- `stripe_refund_id` (String)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Discount
Represents a discount applied to an invoice.

**Key Fields**:
- `discount_id` (UUID, PK)
- `invoice_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `type` (Enum: Percentage, FixedAmount)
- `value` (Decimal) - Percentage (0-100) or fixed USD amount
- `reason` (String) - Why discount was applied (e.g., "Sibling Discount", "Promotional")
- `created_at` (Timestamp)

#### Notification
Represents a notification sent to a user.

**Key Fields**:
- `notification_id` (UUID, PK)
- `recipient_id` (UUID, FK) - User who received it
- `tenant_id` (UUID, FK)
- `type` (Enum: ArrivalConfirmation, ClassSummary, ActivityPhoto, BeltPromotion, UpcomingSimsa, AttendanceStreak, PaymentReceipt, PaymentDue, MembershipExpiry, ScheduleChange, GymAnnouncement, etc.)
- `title` (String)
- `body` (String)
- `language` (Enum: English, Korean, Spanish) - Language sent in
- `channels` (JSON Array) - Which channels it was sent on: [Push, SMS, Email, InApp, Messenger]
- `status` (Enum: Pending, Sent, Delivered, Failed, Read)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### NotificationTemplate
Represents a template for notifications.

**Key Fields**:
- `template_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `type` (Enum) - Notification type
- `language` (Enum: English, Korean, Spanish)
- `subject` (String) - For email
- `body` (String) - Template with personalization tokens: {{member_name}}, {{days_absent}}, {{balance_due}}, {{belt_level}}, {{simsa_date}}, {{gym_name}}, etc.
- `channels` (JSON Array) - Default channels for this notification type
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### NotificationLog
Immutable log of notification delivery attempts.

**Key Fields**:
- `log_id` (UUID, PK)
- `notification_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `channel` (Enum: Push, SMS, Email, InApp, Messenger)
- `status` (Enum: Sent, Delivered, Failed)
- `error_message` (String, nullable)
- `external_id` (String, nullable) - Reference from external service (e.g., Twilio message ID)
- `created_at` (Timestamp)

#### AlertRule
Represents a configurable alert rule.

**Key Fields**:
- `rule_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `trigger_type` (Enum: Absence, LatePayment, MembershipExpiry, SimsaDeadline)
- `thresholds` (JSON Array) - Threshold values (e.g., [3, 7, 14] for days absent)
- `template_id` (UUID, FK) - Notification template to use
- `channels` (JSON Array) - Which channels to send on
- `escalation_config` (JSON) - { consecutive_ignores: 2, escalate_to_admin: true }
- `enabled` (Boolean)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### Newsletter
Represents a formal newsletter (가정통신문).

**Key Fields**:
- `newsletter_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `title` (String)
- `body` (String) - Rich HTML content
- `attachments` (JSON Array) - URLs to attached documents
- `audience_filter` (JSON) - Filter criteria: { classes: [], belt_levels: [], age_groups: [], membership_status: [] }
- `status` (Enum: Draft, Scheduled, Sent)
- `scheduled_at` (Timestamp, nullable)
- `sent_at` (Timestamp, nullable)
- `created_by` (UUID, FK) - Staff member who created it
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### NewsletterRecipient
Tracks which parents received a newsletter.

**Key Fields**:
- `recipient_id` (UUID, PK)
- `newsletter_id` (UUID, FK)
- `parent_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `sent_at` (Timestamp)
- `read_at` (Timestamp, nullable)
- `created_at` (Timestamp)

#### ActivityFeedPost
Represents a post in the parent activity feed.

**Key Fields**:
- `post_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `class_id` (UUID, FK, nullable) - Class this post is about
- `posted_by` (UUID, FK) - Staff member who posted
- `type` (Enum: TrainingNotes, Photo, Observation, Milestone)
- `content` (String) - Text content
- `visibility` (Enum: Class, Individual) - Who can see it
- `visible_to_member_id` (UUID, FK, nullable) - If visibility=Individual, which member
- `created_at` (Timestamp)
- `updated_at` (Timestamp)

#### ActivityFeedPhoto
Represents a photo in an activity feed post.

**Key Fields**:
- `photo_id` (UUID, PK)
- `post_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `photo_url` (String) - S3 URL
- `thumbnail_url` (String) - S3 URL to thumbnail
- `created_at` (Timestamp)

#### ActivityFeedReaction
Represents a parent's reaction to a feed post.

**Key Fields**:
- `reaction_id` (UUID, PK)
- `post_id` (UUID, FK)
- `parent_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `type` (Enum: Like, Comment)
- `content` (String, nullable) - Comment text if type=Comment
- `created_at` (Timestamp)

#### AuditLog
Immutable log of all data mutations.

**Key Fields**:
- `log_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `actor_id` (UUID, FK) - User who made the change
- `actor_role` (Enum) - Role of the actor
- `action` (Enum: Create, Update, Delete)
- `entity_type` (String) - What was changed (e.g., "Member", "Payment")
- `entity_id` (UUID) - Which record was changed
- `old_value` (JSON, nullable) - Previous state
- `new_value` (JSON, nullable) - New state
- `ip_address` (String, nullable)
- `created_at` (Timestamp) - Immutable

#### UserSession
Represents an active user session.

**Key Fields**:
- `session_id` (UUID, PK)
- `user_id` (UUID, FK)
- `tenant_id` (UUID, FK)
- `jwt_token` (String) - The JWT token
- `ip_address` (String)
- `user_agent` (String)
- `expires_at` (Timestamp)
- `created_at` (Timestamp)

#### SystemSetting
Stores per-gym configuration (see section 2 for full list of settings).

**Key Fields**:
- `setting_id` (UUID, PK)
- `tenant_id` (UUID, FK)
- `key` (String) - Setting name
- `value` (JSON) - Setting value (type varies)
- `created_at` (Timestamp)
- `updated_at` (Timestamp)


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
