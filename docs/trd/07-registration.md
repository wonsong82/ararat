# 7. Feature: Registration & Member Management

**Related TRDs**: [03-data-model](./03-data-model.md), [04-auth](./04-auth.md), [09-kiosk-face-recognition](./09-kiosk-face-recognition.md), [10-payments](./10-payments.md), [19-security-compliance](./19-security-compliance.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md)  
**Phase**: MVP (Phase 1)

---

### Parent Registration Flow

1. Parent visits app and enters phone number.
2. System validates phone format (US number, 10 digits).
3. System sends SMS OTP via Twilio.
4. Parent enters OTP.
5. System validates OTP.
6. Parent selects preferred language (English, Korean, Spanish).
7. Parent enters name and email (optional).
8. Parent accepts Terms & Conditions.
9. System creates User record (role=Parent, status=Active).
10. Parent is prompted to add child profile(s).
11. For each child:
    - Parent enters: first name, last name, date of birth, gender, health info (allergies, medical conditions), emergency contact.
    - If child is under 13: system requires verified parental consent before storing data (COPPA compliance).
    - Parent uploads 1-3 profile photos for face recognition enrollment.
    - System creates Member record (status=Pending).
12. Parent submits registration.
13. System sends notification to gym owner: "New registration pending approval: [child name]".
14. Gym owner reviews and approves/rejects via admin dashboard.
15. If approved: Member status changes to Active, parent receives confirmation notification.
16. If rejected: Member status changes to Rejected, parent receives rejection notification with reason.

### Self-Registration (Adult Member)

1. Adult visits app and enters phone number.
2. System validates phone format.
3. System sends SMS OTP.
4. Adult enters OTP.
5. Adult selects preferred language.
6. Adult enters name, email, date of birth, gender, health info.
7. Adult accepts Terms & Conditions.
8. System creates User record (role=Member, status=Active) and Member record (status=Pending).
9. Gym owner approves/rejects.

### Admin Manual Registration

1. Gym owner visits admin app and clicks "Register Member".
2. Gym owner enters member details (name, DOB, gender, health info, phone, email).
3. Gym owner optionally uploads profile photos.
4. Gym owner clicks "Register".
5. System creates User and Member records with status=Active (no approval needed).
6. System sends welcome notification to parent (if parent account exists) or to member (if adult).

### Duplicate Detection

When a new registration is submitted, system checks for potential duplicates:

- **Match Criteria 1**: First name + Last name + Date of birth (exact match).
- **Match Criteria 2**: Phone number (exact match).

If a match is found:
- System flags the registration as "Potential Duplicate".
- Gym owner is notified and must manually review.
- Gym owner can either approve (create new record) or merge with existing member.

### COPPA Compliance (Children Under 13)

If child's date of birth makes them under 13 years old:

1. System blocks data storage until parental consent is verified.
2. System displays consent form in parent's preferred language (English, Korean, Spanish).
3. Consent form includes:
   - Explanation of what data will be collected.
   - Explanation of how data will be used.
   - Explanation of face recognition enrollment (if applicable).
   - Parent's acknowledgment that they are the child's parent/guardian.
4. Parent electronically signs consent form (e.g., checkbox + timestamp).
5. System stores consent record with: parent_id, member_id, consent_date, language, form_version.
6. Only after consent is verified: system allows data storage and face enrollment.
7. Parent can revoke consent at any time via app settings.
8. On revocation: system triggers data deletion workflow (see section 19).

### Profile Management

Members and parents can update profile information via the app:

**Editable Fields**:
- Name, email, phone, health info (allergies, medical conditions), emergency contact.
- Profile photo.
- Preferred language.
- Notification preferences.

**Change Logging**:
- Every change is logged in AuditLog with: actor_id, action=Update, entity_type=Member, old_value, new_value, timestamp.
- Admin can view change history for any member.

**Admin Notifications** (configurable):
- When member updates profile, gym owner receives notification (optional, configurable per gym).

### Photo Management

Profile photos are used for:
1. Kiosk face recognition enrollment.
2. Display in parent app and admin dashboard.

**Upload Process**:
1. Parent/admin uploads photo via app.
2. System validates file type (JPEG, PNG, WebP) and size (max 10 MB).
3. System uploads to S3 with path: `s3://ararat-photos/tenants/{tenantId}/members/{memberId}/profile/{photoId}.jpg`.
4. System generates thumbnail (200x200px) and stores: `s3://ararat-photos/tenants/{tenantId}/members/{memberId}/profile/{photoId}_thumb.jpg`.
5. System stores photo URL in Member record.
6. For face recognition: system syncs photo to kiosk device via secure LAN (see section 9).

**Multiple Photos**:
- System allows up to 5 profile photos per member for face recognition accuracy.
- Photos stored with timestamps, most recent marked as "primary".

### Annual Re-Registration

System sends re-registration reminder annually:

1. Cron job runs daily and checks for members with anniversary dates.
2. If anniversary date is N days away (configurable, default 30 days): send notification to parent.
3. Notification includes link to re-registration form.
4. Parent updates profile information and confirms.
5. System logs re-registration event in AuditLog.

### Member Levels & Grouping

#### Belt Level Tracking

Each gym configures its own belt progression system:

1. Gym owner defines belt levels in SystemSetting: `belt_progression: ["White", "Yellow", "Orange", "Green", "Blue", "Red", "Brown", "Black"]`.
2. System creates Belt records for each level with order (0 = lowest, N = highest).
3. Each Member has `current_belt_id` pointing to their current belt.
4. MemberBelt table tracks belt history (one record per belt achieved).

#### Age Group Assignment

System automatically assigns age groups based on date of birth:

1. Gym owner configures age thresholds in SystemSetting: `age_group_thresholds: { "Little Kids": 5, "Kids": 8, "Teens": 13, "Adults": 18 }`.
2. On member creation or birthday: system calculates age and assigns age_group.
3. On birthday: system recalculates age group. If threshold crossed, system creates admin task: "Review group transfer for [member_name]".

#### Class Group Assignment

Admin assigns members to class groups (e.g., "Beginner", "Advanced", "Competition"):

1. Admin selects member and clicks "Assign to Group".
2. Admin selects target group.
3. System updates Member.class_group.
4. System logs change in AuditLog.

#### Auto-Upgrade

When age threshold is crossed:
1. System detects birthday and recalculates age group.
2. If age group changed: system creates admin task with recommendation to transfer member to new class group.
3. Admin reviews and confirms transfer (or declines).

When 심사 result is entered as "Passed":
1. System auto-updates Member.current_belt_id to new belt.
2. System creates MemberBelt record with achieved_date = today.
3. System sends congratulations notification to parent.

### Withdrawal (탈퇴)

#### Self-Service Withdrawal

1. Member/parent visits app and clicks "Withdraw".
2. System displays withdrawal form with fields: reason (dropdown), comments (optional).
3. Member/parent submits form.
4. System creates withdrawal request with status=Pending.
5. Gym owner receives notification: "Withdrawal request from [member_name]".
6. Gym owner reviews and approves/rejects.
7. If approved:
   - System calculates prorated refund (see section 10).
   - System updates Member.status = Withdrawn, withdrawal_date = today.
   - System triggers data deletion workflow (see section 19).
   - System sends refund notification to parent.
8. If rejected: system notifies member with reason.

#### Refund Calculation

Formula: `refund = (remaining_days / total_days) * paid_amount - discount_clawback`

- `remaining_days`: Days from withdrawal date to membership renewal date.
- `total_days`: Days from membership start date to renewal date.
- `paid_amount`: Amount paid for current membership cycle.
- `discount_clawback`: If a discount was applied (e.g., sibling discount), gym policy determines if discount is clawed back. Configurable per gym.

Example:
- Membership: $100/month, started Feb 1, renewal Mar 1.
- Withdrawal: Feb 15 (15 days remaining out of 28 days).
- Refund: (15/28) * $100 = $53.57.
- If sibling discount of $10 was applied: refund = $53.57 - $5 (50% clawback) = $48.57.

#### Data Handling

On withdrawal approval:
1. Member.status = Withdrawn.
2. Membership.status = Cancelled.
3. All ClassEnrollment records for this member set to status=Withdrawn.
4. Trigger data deletion workflow (see section 19):
   - Delete personal data from database (name, email, phone, health info).
   - Delete profile photos from S3.
   - Delete face embeddings from kiosk devices.
   - Retain anonymized analytics data (attendance counts, payment history for reporting).
   - Log deletion event in AuditLog.


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
