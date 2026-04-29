# 9. Feature: Registration & Member Management

**Related TRDs**: [03-data-model](./03-data-model.md), [04-auth](./04-auth.md), [08-kiosk-app-architecture](./08-kiosk-app-architecture.md), [11-payments](./11-payments.md), [20-security-compliance](./20-security-compliance.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md)  
**Phase**: MVP (Phase 1)

---

## Overview

Registration & Member Management is the core onboarding and lifecycle feature for Ararat. It handles how parents register their children, how adult members self-register, and how gym owners manually enroll students — from initial sign-up through active membership to eventual withdrawal (탈퇴).

**Business value**: Streamlined onboarding reduces drop-off during registration, ensures COPPA compliance for children under 13, and gives gym owners (관장님) full control over member approval and profile management.

**Key actors**:
- **Parents (학부모)**: Register children, manage profiles, initiate withdrawals
- **Adult Members**: Self-register for their own membership
- **Gym Owner / Admin (관장님)**: Approve/reject registrations, manually register members, manage member profiles, handle withdrawals
- **System**: Duplicate detection, age group auto-assignment, annual re-registration reminders, belt auto-upgrade

---

## Business Rules

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
    - Parent uploads 1–3 profile photos for face recognition enrollment.
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
8. On revocation: system triggers data deletion workflow (see [20-security-compliance](./20-security-compliance.md)).

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
4. System generates thumbnail (200×200px) and stores: `s3://ararat-photos/tenants/{tenantId}/members/{memberId}/profile/{photoId}_thumb.jpg`.
5. System stores photo URL in Member record.
6. For face recognition: system syncs photo to kiosk device via secure LAN (see [08-kiosk-app-architecture](./08-kiosk-app-architecture.md)).

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
   - System calculates prorated refund (see [11-payments](./11-payments.md)).
   - System updates Member.status = Withdrawn, withdrawal_date = today.
   - System triggers data deletion workflow (see [20-security-compliance](./20-security-compliance.md)).
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
- Refund: (15/28) × $100 = $53.57.
- If sibling discount of $10 was applied: refund = $53.57 − $5 (50% clawback) = $48.57.

#### Data Handling on Withdrawal

On withdrawal approval:
1. Member.status = Withdrawn.
2. Membership.status = Cancelled.
3. All ClassEnrollment records for this member set to status=Withdrawn.
4. Trigger data deletion workflow (see [20-security-compliance](./20-security-compliance.md)):
   - Delete personal data from database (name, email, phone, health info).
   - Delete profile photos from S3.
   - Delete face embeddings from kiosk devices.
   - Retain anonymized analytics data (attendance counts, payment history for reporting).
   - Log deletion event in AuditLog.

---

## Backend

### API Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/api/v1/auth/register/parent` | Parent registration (phone OTP verified) | Public |
| `POST` | `/api/v1/auth/register/member` | Self-registration (adult member) | Public |
| `POST` | `/api/v1/tenants/{tenantId}/members` | Admin manual registration | Owner, Manager |
| `GET` | `/api/v1/tenants/{tenantId}/members` | List members (filterable, paginated) | Instructor+ |
| `GET` | `/api/v1/tenants/{tenantId}/members/{id}` | Get member detail | Instructor+ |
| `PATCH` | `/api/v1/tenants/{tenantId}/members/{id}` | Update member profile | Owner, Manager, Self |
| `GET` | `/api/v1/tenants/{tenantId}/registrations` | List pending registrations | Manager+ |
| `PATCH` | `/api/v1/tenants/{tenantId}/registrations/{id}/approve` | Approve registration | Manager+ |
| `PATCH` | `/api/v1/tenants/{tenantId}/registrations/{id}/reject` | Reject registration (requires reason) | Manager+ |
| `POST` | `/api/v1/tenants/{tenantId}/members/{id}/withdraw` | Initiate withdrawal request | Owner, Manager, Self |
| `POST` | `/api/v1/tenants/{tenantId}/members/{id}/photos` | Upload member photo (multipart) | Owner, Manager, Self |
| `DELETE` | `/api/v1/tenants/{tenantId}/members/{id}/photos/{photoId}` | Delete member photo | Owner, Manager, Self |

**Query parameters for `GET /members`**:
- `search` (string) — Full-text search on name, phone, email
- `status` (enum) — Active, Inactive, Pending, Withdrawn
- `beltId` (UUID) — Filter by current belt level
- `ageGroup` (string) — Filter by age group
- `classGroup` (string) — Filter by class group
- `cursor` (string) — Cursor-based pagination token
- `limit` (integer, default 25, max 100) — Page size
- `sortBy` (string) — Field to sort by (name, joinDate, lastAttendance)
- `sortOrder` (enum) — asc, desc

### Service Logic

#### Duplicate Detection Service

On every new registration (parent, self, or admin):
1. Query existing members within the same tenant using Match Criteria 1 (first_name + last_name + date_of_birth) and Match Criteria 2 (phone number).
2. If any match found: set registration status to `DuplicateFlagged` and create an admin task with the potential match details.
3. Admin reviews via the Pending Registrations screen and can approve as new, reject, or merge with existing member.

#### COPPA Consent Verification

Before persisting any child member data:
1. Calculate age from date_of_birth.
2. If age < 13: require a signed consent record (parent_id, member_id, consent_date, language, form_version).
3. Block data storage and face enrollment until consent is verified.
4. On consent revocation: trigger data deletion workflow from [20-security-compliance](./20-security-compliance.md).

#### Photo Upload to S3

1. Validate file type (JPEG, PNG, WebP) and size (≤ 10 MB).
2. Generate a UUID for `photoId`.
3. Upload original to `s3://ararat-photos/tenants/{tenantId}/members/{memberId}/profile/{photoId}.jpg`.
4. Generate 200×200px thumbnail and upload to `.../{photoId}_thumb.jpg`.
5. If this is the newest photo, set it as primary (`is_primary = true`, unset previous primary).
6. Enforce max 5 photos per member — reject upload if limit reached.
7. Return photo URL and thumbnail URL in response.

#### Face Enrollment Trigger

After a photo is uploaded for a member with COPPA consent (if applicable):
1. Queue a face enrollment event to the kiosk sync service.
2. Kiosk device downloads the photo via secure LAN and processes face embedding on-device.
3. No biometric data is stored in the cloud (see [ADR-004](./adr/004-face-recognition-on-device.md)).

#### Age Group Auto-Calculation

On member creation and daily via cron job:
1. Calculate member's current age from date_of_birth.
2. Compare against gym's configured `age_group_thresholds` in SystemSetting.
3. Assign the matching age group to Member.age_group.
4. If age group changed (birthday threshold crossed): create admin task "Review group transfer for [member_name]" with old and new group names.

#### Belt Auto-Upgrade

When a 심사 result is recorded as "Passed":
1. Update Member.current_belt_id to the new belt.
2. Create MemberBelt record with achieved_date and simsa_result_id.
3. Set previous MemberBelt.is_current = false.
4. Send congratulations notification to parent via notification service.

#### Withdrawal with Refund Calculation

1. Validate the member has an active membership.
2. Calculate prorated refund using the formula: `(remaining_days / total_days) * paid_amount - discount_clawback`.
3. Create withdrawal request (status=Pending) for admin approval.
4. On approval: update Member.status, Membership.status, ClassEnrollment statuses.
5. Trigger Stripe refund via payment service (see [11-payments](./11-payments.md)).
6. Trigger data deletion workflow (see [20-security-compliance](./20-security-compliance.md)).

#### Annual Re-Registration Cron

Runs daily at 02:00 UTC:
1. Query all active members where `join_date` anniversary is within N days (configurable, default 30).
2. For each qualifying member: send re-registration reminder notification to parent.
3. Notification includes a deep link to the profile update / re-registration form.
4. Log reminder event in AuditLog.

---

## Parent/Member App

### Screen: Registration Wizard (during first login)

Multi-step form presented after a parent creates their account (completes OTP verification). Guides the parent through adding their first child.

**Steps**:

1. **Parent Profile** — Name, email (optional). Pre-populated from auth if available.
2. **Add Child** — First name, last name, date of birth (date picker), gender (radio group), health info (allergies textarea, medical conditions textarea), emergency contact (name, phone, relationship).
3. **COPPA Consent** (conditional — shown only if child is under 13) — Consent form rendered in parent's preferred language. Explains data collection, usage, and face recognition enrollment. Checkbox: "I confirm I am the parent/legal guardian of this child and consent to the data collection described above." Stores consent_date, language, form_version.
4. **Upload Photos** — Upload 1–3 profile photos for face recognition. Drag-and-drop or camera capture. Preview thumbnails with delete option. Minimum 1 photo required.
5. **Review & Submit** — Summary of all entered data. Edit button per section to go back. Submit button sends registration for approval.

**Key components**: Step wizard (progress bar with step labels), `FormField` components for each input, `DatePicker` (locale-aware), `RadioGroup` (gender), `Textarea` (health info), photo uploader with preview, `Checkbox` (COPPA consent), summary `Card`.

**Data sources**: `useCreateRegistration()` mutation on final submit.

**Loading/error states**:
- Photo upload: inline progress bar per photo, retry on failure.
- Submit: loading spinner on button, disable all inputs during submission.
- Server validation errors: mapped to individual fields via `setError()`.
- Form-level errors (e.g., "Registration limit reached"): alert banner above form.

**Post-submit**: Success screen with message "Registration submitted! You'll be notified when the gym approves your registration." Link to add another child.

### Screen: Children List (`/children`)

**What it shows**: List of parent's registered children with name, profile photo, belt level badge, attendance streak indicator, and membership status.

**Key components**: `Card` per child with:
- `Avatar` — Profile photo (primary) with fallback initials
- `Badge` — Belt color (hex from Belt entity, e.g., yellow badge for Yellow belt)
- Attendance streak indicator (e.g., "🔥 12 days" or "—" if no recent check-ins)
- Status text — Active (green), Pending (amber), Inactive (gray)
- "Add Child" `Button` at the bottom — links to Registration Wizard for adding another child

**Data sources**: `useChildren()` — returns list of members linked to the current parent via ParentChild.

**User actions**:
- Tap a child card → navigate to Child Detail (`/children/:id`)
- Tap "Add Child" → open Registration Wizard (Step 2 onwards, skipping parent profile)

**Empty state**: `EmptyState` component with illustration and message: "No children registered yet. Tap 'Add Child' to get started."

### Screen: Child Detail (`/children/:id`)

**What it shows**: Full profile for one child with tabbed sections for all related data.

**Tabs**:

- **Profile** — Name, date of birth, calculated age, age group, belt level (with color badge), profile photo (large `Avatar`), health info (allergies, medical conditions), emergency contact. "Edit Profile" button opens edit form in a `Sheet`.
- **Attendance** — Calendar heatmap showing check-in days (green cells) and absent days (gray cells). Below the heatmap: list of recent attendance records with columns: date, check-in time, check-in method (Face/QR/Manual), class name. Scrollable, paginated.
- **Belt Progress** — Vertical timeline of belt promotions. Each entry: belt name, color dot, achieved date, linked 심사 name (if applicable). Current belt is highlighted with a glow effect. Below timeline: "Next eligible 심사" info if an upcoming exam exists.
- **Payments** — List of invoices for this child's membership. Each row: invoice number, amount, status badge (Paid/Pending/Overdue/Refunded), due date. Tap an invoice to view details or make payment.

**Key components**: `Tabs`, large `Avatar` with photo gallery (swipeable if multiple photos), `Badge` (belt color), calendar heatmap component (custom, uses CSS grid with day cells), attendance record `Table`, belt timeline component (vertical line with colored dots), invoice list with status `Badge`.

**Data sources**: `useMember(childId)`, `useMemberAttendance(childId)`, `useMemberBeltHistory(childId)`, `useMemberPayments(childId)`.

**User actions**:
- Switch between tabs
- Tap "Edit Profile" → opens `Sheet` with pre-populated form (partial Zod schema)
- Tap an attendance record → inline expansion showing details (time, method, instructor who confirmed)
- Tap a belt entry → view promotion certificate (PDF) if available
- Tap an invoice → navigate to payment detail / trigger payment flow

### Screen: Withdrawal (`/withdraw`)

**What it shows**: Multi-step withdrawal wizard for cancelling a child's (or self's) membership.

**Steps**:

1. **Select Child** — Choose which child to withdraw (card selector showing name + photo). For adult members, this step shows only the member's own profile.
2. **Reason** — Select reason from dropdown: Moving, Financial hardship, Schedule conflict, Dissatisfied, Other. Optional comments `Textarea`.
3. **Review** — Shows refund calculation `Card`:
   - Current membership plan name and price
   - Membership period (start → renewal date)
   - Remaining days
   - Prorated refund amount (calculated via `useWithdrawalEstimate(childId)`)
   - Discount clawback amount (if applicable)
   - Net refund amount (bold)
   - Data deletion notice: "Your child's personal data and photos will be permanently deleted. Anonymized attendance and payment records will be retained for reporting."
4. **Confirm** — Final confirmation with `Checkbox`: "I understand this action cannot be undone and my child's data will be deleted." Submit `Button` (destructive variant, red).

**Key components**: Step wizard (progress indicator), child selector `Card` grid, reason `Select`, comments `Textarea`, refund calculation `Card` (with line items), data deletion notice `AlertBanner`, confirmation `Checkbox`, destructive `Button`.

**Data sources**: `useChildren()` (step 1), `useWithdrawalEstimate(childId)` (step 3), `useSubmitWithdrawal()` mutation (step 4).

**User actions**:
- Select child and reason
- Review refund estimate
- Confirm and submit withdrawal request
- Cancel and go back at any step (step wizard supports back navigation)

**Post-submit**: Success screen: "Withdrawal request submitted. The gym will review your request and process the refund." Redirect to `/children` after 5 seconds.

### Screen: Settings — Profile Section (`/settings`)

**What it shows**: User preferences and account settings related to registration/profile.

**Sections**:
- **Language**: `RadioGroup` to select English / Korean / Spanish. Change takes effect immediately via `i18n.changeLanguage()`.
- **Notifications**: `Switch` toggles for each notification type: attendance alerts, payment reminders, 심사 announcements, newsletters, activity feed updates.
- **Notification Channel**: Preferred channel selector (Push, SMS, Email, KakaoTalk).
- **Account**: Phone number (`Input`, read-only — displayed for reference), email (`Input`, editable), name (`Input`, editable).

**Key components**: `RadioGroup` (language), `Switch` toggles (notification preferences), `Input` (email, name — editable; phone — read-only), `Button` (save changes).

**Data sources**: `useUserSettings()`, `useUpdateUserSettings()` mutation.

**User actions**:
- Change language (immediate effect)
- Toggle notification preferences
- Update email address and name
- Save changes (success toast on save)

---

## Admin App

### Screen: Member List (`/members`)

**What it shows**: Searchable, filterable, sortable table of all members in the gym. Primary member management interface for admin.

**Table columns** (via TanStack Table):
| Column | Content | Sortable |
|--------|---------|----------|
| Avatar | Profile photo thumbnail (40×40, `Avatar` with fallback initials) | No |
| Name | First + last name (link to detail) | Yes |
| Belt Level | Color `Badge` with belt name | Yes (by belt order) |
| Age Group | Text (e.g., "Kids", "Teens") | Yes |
| Class Group | Text (e.g., "Beginner", "Advanced") | Yes |
| Status | `Badge` — Active (green), Inactive (gray), Pending (amber), Withdrawn (red) | Yes |
| Join Date | Locale-formatted date | Yes |
| Last Attendance | Relative time (e.g., "2 days ago") or "Never" | Yes |

**Filters** (above table):
- Text search `Input` — searches name, phone, email (debounced 300ms)
- Status `Select` dropdown — All, Active, Inactive, Pending, Withdrawn
- Belt Level `Select` dropdown — populated from gym's belt configuration
- Age Group `Select` dropdown — populated from gym's age group thresholds
- Class Group `Select` dropdown — populated from gym's class groups

**Bulk actions** (toolbar appears when 1+ rows selected via checkbox):
- Change Class Group — opens `Select` dialog
- Send Notification — opens notification compose dialog
- Export Selected — downloads CSV/Excel of selected members

**Key components**: TanStack Table with sorting, column visibility toggle, cursor-based pagination (25/50/100 per page). Filter bar with `Input` (search), `Select` (dropdowns). Bulk action toolbar. "Add Member" `Button` (opens manual registration form in `Sheet`). "Export All" `Button`.

**Data sources**: `useMembers(filters)` with cursor-based pagination. Filters stored in URL query params for shareable/bookmarkable views.

**User actions**:
- Search by name, phone, or email
- Filter by status, belt, age group, class group
- Sort by any sortable column (click header)
- Click a row → navigate to member detail (`/members/:id`)
- Select rows → bulk actions (change class group, send notification, export)
- Click "Add Member" → open manual registration form
- Click "Export All" → download filtered list as Excel/PDF

### Screen: Member Detail (`/members/:id`)

**What it shows**: Complete member profile with all related data in a tabbed layout. Central hub for managing an individual member.

**Tabs**:

- **Profile** — Personal info: name, DOB, calculated age, gender, phone, email, health info (allergies, medical conditions), emergency contact. Photos: gallery of uploaded profile photos (up to 5). Parent info: linked parent name and phone (clickable to navigate to parent). COPPA consent status and date (if applicable). "Edit" `Button` opens edit form in `Sheet` with all editable fields.

- **Attendance** — Calendar heatmap (same component as parent app) showing check-in days (green) and absences (gray). Below: filterable `Table` of attendance records with columns: date, check-in time, check-out time, method, class, validation status. Date range filter. "Mark Absent" action for manual attendance correction (requires reason).

- **Belt & 심사** — Current belt (large color badge). Belt history timeline (vertical, same as parent app). Eligible upcoming exams with registration status. Past exam results: date, belt tested, result (pass/fail), score, notes. "Promote" `Button` for manual belt update (opens dialog: select new belt, date, notes — bypasses 심사 flow for corrections).

- **Payments** — Current membership plan name and status. Auto-pay status. Invoice table: invoice number, amount, status badge, due date, paid date. "Create Invoice" `Button` (manual invoice for ad-hoc charges). "Issue Refund" `Button` (opens confirmation dialog with amount calculation).

- **Activity Feed** — Posts about this member from instructors. Each post: author, date, text, photos, reactions count. "Add Note" `Button` opens post creation form (text + optional photos).

- **Audit Log** — Chronological table of all changes to this member's record. Columns: date, action (Created, Updated, Deleted), actor name, actor role, field changed, old value, new value. Filterable by date range and action type. Read-only.

**Key components**: `Tabs`, profile info `Card`, large `Avatar` with photo gallery (lightbox on click), parent link, `Table` (attendance, payments, audit log), belt timeline, calendar heatmap, `Sheet` (edit form), `Dialog` (promote belt, create invoice, issue refund), activity feed post `Card` list.

**Data sources**: `useMember(id)`, `useMemberAttendance(id)`, `useMemberBeltHistory(id)`, `useMemberPayments(id)`, `useMemberFeedPosts(id)`, `useMemberAuditLog(id)`.

**User actions**:
- Edit profile fields (opens `Sheet` with pre-populated form)
- View and navigate to linked parent profile
- View attendance with date range filter; mark manual correction (with reason)
- Manually promote belt level (with date and notes)
- Create manual invoice for ad-hoc charges
- Issue refund (confirmation dialog with amount, reason)
- Add instructor note to activity feed
- Download member data export (PDF summary)

### Screen: Pending Registrations (`/registrations`)

**What it shows**: Table of pending registration applications awaiting admin approval. This is the gym owner's approval queue.

**Table columns** (via TanStack Table):
| Column | Content | Sortable |
|--------|---------|----------|
| Applicant Name | Parent or adult member name | Yes |
| Type | `Badge` — Parent Registration / Self Registration | Yes |
| Phone | Phone number | No |
| Email | Email address (or "—" if not provided) | No |
| Children | Count of children in this registration | Yes |
| Submitted Date | Locale-formatted date + relative time | Yes |
| Status | `Badge` — Pending (amber), Duplicate Flagged (red) | Yes |
| Actions | Approve / Reject buttons | No |

**Key components**: TanStack Table with sorting and pagination. Status `Badge` (amber for Pending, red for Duplicate Flagged). Per-row action buttons: "Approve" (`Button`, primary) and "Reject" (`Button`, destructive). Duplicate warning `AlertBanner` on flagged rows.

**Row expansion / detail Sheet**: Clicking a row opens a `Sheet` panel with full application details:
- Parent info: name, phone, email, language preference
- Children: list of children with name, DOB, gender, health info, uploaded photos
- COPPA consent status (if applicable)
- Duplicate match details (if flagged): shows the existing member record that matched, with side-by-side comparison

**Duplicate Merge Wizard** (opened from flagged rows):
1. Side-by-side comparison of new registration vs. existing member
2. Field-by-field merge selector (choose which value to keep for each field)
3. Confirm merge → system updates existing member record with selected values, discards new registration

**Data sources**: `usePendingRegistrations()` with optional status filter.

**User actions**:
- Click a row to view full application details (opens `Sheet`)
- Approve registration (one-click → confirmation toast → member status changes to Active, parent notified)
- Reject registration with reason (opens `Dialog` with reason `Input` → member status changes to Rejected, parent notified with reason)
- Merge duplicate (if flagged) → opens merge wizard with side-by-side comparison
- Filter by status (Pending, Duplicate Flagged)
- Sort by submitted date (default: oldest first)

---

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| Notification Service | Internal (TRD 13) | Dispatch registration confirmations, withdrawal notices | Internal call | `{ recipientId, templateId, data }` | `{ notificationId }` |
| Payment Service | Internal (TRD 11) | Create initial membership on registration | Internal call | `{ memberId, planId }` | `{ membershipId }` |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/tenants/{tenantId}/members` | GET/POST | Admin App, Parent App | Member data JSON | Member list or detail |
| `/api/v1/tenants/{tenantId}/members/{id}` | GET/PATCH/DELETE | Admin App, Parent App | Member update JSON | Member detail |
| `/api/v1/tenants/{tenantId}/parents` | GET/POST | Admin App | Parent data JSON | Parent list or detail |
| `/api/v1/tenants/{tenantId}/parents/{id}/children` | GET/POST/DELETE | Admin App, Parent App | Child link JSON | Parent-child associations |
| `/api/v1/tenants/{tenantId}/registrations` | GET/POST/PATCH | Admin App | Registration data JSON | Registration list or detail |
| `/api/v1/tenants/{tenantId}/members/{id}/withdraw` | POST | Admin App, Parent App | `{ reason }` | Withdrawal confirmation |

---


## Phase 2 Features (Not Yet Specified)

The following features are identified in the PRD for Phase 2 and will be fully specified before implementation:

### Exit Survey
- **Trigger**: When a member's status is changed to `withdrawn` (via Admin App or Parent App withdrawal flow)
- **Survey delivery**: Optional survey link sent via notification to the parent/member upon withdrawal
- **Data captured**: Reason for leaving (predefined categories + free text), satisfaction rating, likelihood to return
- **Admin view**: Aggregated exit survey results in Admin App reporting dashboard
- **Privacy**: Survey responses stored anonymized after 90 days, COPPA-compliant for minors

---

## Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
