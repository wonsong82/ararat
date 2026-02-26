# 8. Feature: Attendance System

**Related TRDs**: [03-data-model](./03-data-model.md), [09-kiosk-face-recognition](./09-kiosk-face-recognition.md), [12-notifications](./12-notifications.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1)

---

### Check-In Methods (Priority Order)

1. **Face Recognition** (Primary) - On-device face recognition on kiosk (Phase 2+).
2. **QR Code** (Secondary) - Scan QR code from membership card or parent's phone.
3. **Name Search** (Fallback) - Manual search by name with photo confirmation.
4. **Staff Manual** (Last Resort) - Staff manually marks member as checked in.

### Check-In Data Captured

Every check-in creates an Attendance record with:
- `member_id` - Who checked in
- `timestamp` - When they checked in
- `method` - How they checked in (face/QR/name/manual)
- `kiosk_id` - Which kiosk device recorded it
- `staff_validator_id` - Staff member who validated (if dual-check enabled)
- `validation_status` - Confirmed, PendingValidation, or Unconfirmed
- `class_id` - Class they're attending (if known)

### Dual-Check (Staff Validation) Flow

If gym enables dual-check mode:

1. Member checks in at kiosk (face/QR/name/manual).
2. Kiosk records check-in with `validation_status = PendingValidation`.
3. Kiosk displays "Awaiting staff confirmation" message.
4. Staff receives notification on dashboard: "Pending validation: [member_name]".
5. Staff reviews check-in on dashboard and clicks "Confirm".
6. System updates Attendance.validation_status = Confirmed, validation_time = now.
7. Parent receives confirmation notification.

**Unconfirmed Alert**:
- If staff doesn't confirm within 5 minutes (configurable): system triggers alert.
- Alert sent to staff dashboard: "Unconfirmed check-in: [member_name] (5 min ago)".
- Alert sent to parent: "Your child's check-in is pending staff confirmation. Please contact the gym if this is incorrect."
- Attendance.validation_status = Unconfirmed.

### Drop-Off Audit Trail

Every check-in event creates an immutable AttendanceAudit record:
- `event_type` - CheckIn, Validation, Unconfirmed, Sync
- `event_data` - Full details: method, kiosk, staff, validation status, etc.
- `created_at` - Immutable timestamp

This provides a complete audit trail for child safety and liability.

### Absence Alert Engine

#### Configuration

Gym owner configures absence thresholds in SystemSetting:
- `absence_alert_thresholds: [3, 7, 14]` - Days of non-attendance that trigger alerts.

#### Daily Cron Job

Every day at 11:59 PM (gym's timezone):
1. For each active member: calculate days since last attendance.
2. For each threshold: check if member crossed it.
3. If crossed and not already alerted: trigger notification.
4. Track which thresholds already triggered per member (in a `MemberAbsenceAlert` table) to avoid duplicate alerts.

#### Notification Trigger

When threshold is crossed:
1. Fetch NotificationTemplate for absence alert (language-aware).
2. Render template with personalization tokens: `{{member_name}}`, `{{days_absent}}`, `{{gym_name}}`.
3. Dispatch to configured channels (email, SMS, push).
4. Log notification in NotificationLog.

#### Admin Dashboard

Admin sees "At-Risk Members" dashboard:
- List of members ranked by days absent.
- Filter by threshold level (3, 7, 14 days).
- Quick action: "Send Personal Message" to parent.

### Pre-Arrival Reservation (Future)

Parent can reserve drop-off time slot via app (Phase 2+):
1. Parent selects class and date.
2. System shows available time slots.
3. Parent selects slot and confirms.
4. System creates Reservation record.
5. Gym owner receives notification: "Reservation: [member_name] for [class] on [date]".
6. When member checks in: system matches to reservation and marks as confirmed.


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
