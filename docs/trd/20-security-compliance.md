# 20. Security & Compliance Implementation

**Related TRDs**: [09-registration](./09-registration.md), [08-kiosk-app-architecture](./08-kiosk-app-architecture.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md), [ADR-007](./adr/007-no-gps-notifications.md)  
**Phase**: MVP (Phase 1)

---

### COPPA Compliance (Children Under 13)

**Regulatory Requirement**: FTC's amended COPPA Rule (effective June 2025) explicitly includes biometric identifiers in "personal information." Compliance deadline: April 22, 2026.

#### Consent Form Content Specification

The COPPA consent form is a content specification for legal counsel review — not final legal text. The form must be rendered in the parent's preferred language (English, Korean, Spanish) and must include all of the following content elements:

**1. Data Collected from Children**:
- Name (first and last)
- Date of birth
- Belt rank / promotion history
- Attendance records (check-in times, methods, class assignments)
- Profile photo (optional — used for face recognition enrollment on kiosk)
- Class enrollment information

**2. Purpose of Data Collection**:
- Gym management and student record keeping
- Attendance tracking via kiosk check-in (face recognition, QR code, or manual)
- Belt promotion tracking and 심사 (exam) eligibility
- Parent communication (notifications about attendance, payments, promotions)

**3. Third-Party Sharing Disclosure**:
- Ararat does NOT share children's personal data with third parties
- Exception: Stripe (payment processing) — processes parent's payment information only, never child data
- Biometric data (face embeddings) is stored exclusively on the local iPad kiosk device and is never transmitted to servers or third parties (per [ADR-004](./adr/004-face-recognition-on-device.md))

**4. Parental Rights**:
- Right to review all data collected about their child at any time
- Right to request deletion of their child's data (triggers data deletion workflow)
- Right to refuse further data collection — child's membership can continue without face recognition enrollment
- Right to revoke consent at any time via app settings

**5. Verification Method**:
- **In-person registration**: Parent signs paper consent form during first visit to the gym. Staff uploads signed form as document attachment linked to the ConsentRecord.
- **Online registration**: Email verification to the parent's registered email address + knowledge-based verification question (child's current belt rank or enrollment date). Both must pass before consent is recorded.

**6. Consent Timing**:
- Consent MUST be obtained BEFORE collecting any child data
- System blocks all data storage for children under 13 until a valid ConsentRecord exists
- If DOB indicates child is under 13 at registration time, the registration wizard halts at the consent step

**7. Annual Consent Renewal**:
- Consent renewal is required annually from the date of original consent
- System triggers a notification to the parent 30 days before consent expiration
- If consent is not renewed within 30 days of expiration, system suspends the child's profile (status → Inactive) and blocks further data collection
- Parent can renew consent via app, which creates a new ConsentRecord version

#### ConsentRecord Entity (COPPA)

Stored in the `ConsentRecord` table (see [03-data-model](./03-data-model.md)):

| Field | Type | Description |
|-------|------|-------------|
| `consent_id` | UUID (PK) | Unique identifier |
| `tenant_id` | UUID (FK) | Gym tenant |
| `parent_id` | UUID (FK) | Parent who gave consent |
| `member_id` | UUID (FK) | Child the consent applies to |
| `type` | Enum | `coppa_parental` |
| `consent_version` | String | Semantic version of the consent form (e.g., "1.0.0") |
| `full_text_shown` | Text | Complete text of the consent form as displayed to the parent |
| `language` | Enum | Language the form was displayed in (en, ko, es) |
| `parent_signature` | String (nullable) | Digital signature reference or "checkbox" for electronic consent |
| `ip_address` | String | IP address of the parent at time of consent |
| `verification_method` | Enum | `in_person`, `email_knowledge_based` |
| `expires_at` | Timestamp | 1 year from consent date (annual renewal) |
| `revoked_at` | Timestamp (nullable) | When consent was revoked, if applicable |
| `created_at` | Timestamp | When consent was given |

#### Implementation Rules

1. **Consent Collection**: During child registration, if DOB < 13 years: system blocks data storage until ConsentRecord is created.
2. **Consent Display**: Consent form rendered in parent's preferred language (English, Korean, Spanish).
3. **Data Storage**: Only after consent verified — system allows data storage. All child data is linked to `consent_id` for traceability.
4. **Consent Revocation**: Parent can revoke consent at any time via app settings. On revocation: set `revoked_at` timestamp and trigger data deletion workflow (see Data Deletion API below).
5. **Data Retention**: Active member data retained while active + 1 year after withdrawal. Written retention policy published on website (trilingual).

### BIPA Compliance (Biometric Data — Illinois)

**Regulatory Requirement**: Illinois Biometric Information Privacy Act (BIPA) requires written informed consent before collecting biometric identifiers. Applies when the gym operates in Illinois. Each unauthorized scan is a separate violation ($1,000–$5,000 per violation).

#### Written Notice Content Specification

Before any biometric data collection (face enrollment), the system must present a written notice containing all of the following elements:

**1. Type of Biometric Data**: The biometric data being collected is face geometry — a mathematical representation (128-dimensional embedding vector) derived from photographs of the member's face.

**2. Specific Purpose**: The sole purpose of collecting face geometry is attendance verification via facial recognition on the iPad kiosk device at the gym entrance. Face recognition enables hands-free self check-in for students.

**3. Storage Location**: Face embeddings are stored ONLY on the local iPad kiosk device in an encrypted SQLite database (SQLCipher, AES-256). Embeddings are NEVER transmitted to Ararat's cloud servers, third-party services, or any external system (per [ADR-004](./adr/004-face-recognition-on-device.md)).

**4. Retention Period**: Face embeddings are retained on the kiosk device for as long as the member has an active membership and biometric consent has not been revoked.

**5. Destruction Schedule**: Embeddings are permanently deleted from all kiosk devices within 30 days of:
  - Member withdrawal (탈퇴)
  - Consent revocation by parent/member
  - Membership expiration without renewal

#### Written Release

- For minors (under 18): Parent or legal guardian signs the BIPA written release on behalf of the child.
- For adult members (18+): The member signs the written release themselves.
- The written release is a separate, explicit consent action — not bundled into general Terms & Conditions.
- Electronic signature (checkbox + timestamp + IP address) is accepted.

#### Opt-Out

- Members can opt out of face recognition at any time via app settings or by requesting through gym staff.
- On opt-out: kiosk deletes the member's face embedding from all devices within 30 days. The member falls back to QR code or manual name search for check-in.
- Opt-out does NOT affect membership status or service quality.
- Opt-out is logged in AuditLog and the ConsentRecord is updated with `revoked_at` timestamp.

#### ConsentRecord Entity (BIPA)

BIPA consent is stored in the same `ConsentRecord` table with `type = bipa_biometric`:

| Field | Type | Description |
|-------|------|-------------|
| `consent_id` | UUID (PK) | Unique identifier |
| `tenant_id` | UUID (FK) | Gym tenant |
| `parent_id` | UUID (FK, nullable) | Parent who signed (for minors) |
| `member_id` | UUID (FK) | Member whose biometric data is collected |
| `type` | Enum | `bipa_biometric` |
| `consent_version` | String | Version of the BIPA notice/release form |
| `full_text_shown` | Text | Complete text of the BIPA notice as displayed |
| `language` | Enum | Language displayed (en, ko, es) |
| `parent_signature` | String (nullable) | Signature reference (parent for minors, member for adults) |
| `ip_address` | String | IP address at time of consent |
| `verification_method` | Enum | `in_person`, `electronic` |
| `revoked_at` | Timestamp (nullable) | When consent was revoked / opt-out |
| `created_at` | Timestamp | When consent was given |

#### Implementation Rules

1. **Written Consent**: Before face enrollment on kiosk, system displays BIPA consent form with all notice elements above.
2. **Retention Policy**: Publicly available on gym website. States: embeddings retained while member active, deleted on withdrawal or consent revocation. No sale or profit from biometric data.
3. **Deletion Workflow**: On withdrawal or consent revocation — delete embeddings from all kiosk devices. Backend sends deletion command to each kiosk via the sync mechanism (see [08-kiosk-app-architecture](./08-kiosk-app-architecture.md)). Log deletion event in AuditLog.
4. **No Cloud Storage**: Biometric data never exists on cloud servers. The ConsentRecord (metadata only) is stored in the backend database; the actual face embedding exists only on kiosk devices.

### Data Encryption

- **At Rest**: AES-256 encryption for all databases and S3 storage.
- **In Transit**: TLS 1.3 for all API communication.
- **Kiosk Local DB**: Encrypted with device-specific key (iOS Keychain).
- **Stripe Data**: PCI DSS compliant (Stripe handles payment data, Ararat never stores card numbers).

### Audit Trail

Immutable append-only log of all data access and modifications:

```
AuditLog:
  - log_id
  - tenant_id
  - actor_id
  - action (Create, Read, Update, Delete)
  - entity_type
  - entity_id
  - old_value (JSON)
  - new_value (JSON)
  - ip_address
  - created_at (immutable)
```

Retained for 7 years (COPPA requirement).

### Data Deletion API

Data deletion is a multi-step process with a grace period — not an instantaneous operation. This protects against accidental deletion while complying with COPPA, BIPA, and CCPA/CPRA requirements.

#### API Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/api/v1/members/:id/deletion-request` | Initiate a deletion request | Owner, Manager, Self (parent) |
| `GET` | `/api/v1/deletion-requests/:id` | Check deletion request status | Owner, Manager, Self (parent) |
| `POST` | `/api/v1/deletion-requests/:id/cancel` | Cancel a pending deletion request | Owner, Manager, Self (parent) |
| `GET` | `/api/v1/privacy/data-export` | Export all personal data (CCPA right to know) | Self (authenticated user) |

**Note**: A `DELETE` HTTP method is intentionally NOT used — deletion is a process with a grace period, not an instantaneous action.

#### Initiate Deletion Request

**`POST /api/v1/members/:id/deletion-request`**

Request body:
```json
{
  "reason": "string (required) — withdrawal | consent_revocation | ccpa_request | parent_request | other"
}
```

Response (201 Created):
```json
{
  "requestId": "uuid",
  "memberId": "uuid",
  "reason": "withdrawal",
  "scheduledDeletionDate": "2026-04-01T00:00:00Z",
  "status": "pending",
  "createdAt": "2026-03-01T12:00:00Z"
}
```

- `scheduledDeletionDate` is 30 days from the request creation date.
- Admin (Owner/Manager) receives an immediate notification when a deletion request is created.
- An AuditLog entry is created: `action=Create, entity_type=DeletionRequest`.

#### 30-Day Grace Period

- The member/parent can cancel the deletion request at any time during the 30-day grace period via `POST /api/v1/deletion-requests/:id/cancel`.
- During the grace period, the member's account remains active and accessible.
- If not cancelled, the deletion executes automatically after 30 days.

#### Deletion Execution (Cron Job)

A cron job runs daily at 03:00 UTC to process deletion requests past their 30-day grace period.

**What gets DELETED** (personal data removed permanently):
- Personal information: name, email, phone, date of birth, address, health info, emergency contact
- Profile photos: deleted from S3 (originals and thumbnails)
- Face embeddings: backend sends deletion command to all kiosk devices associated with the member's tenant. Kiosk confirms deletion on next sync.
- Consent record content: `full_text_shown` and `parent_signature` fields cleared. Metadata (consent_id, type, timestamps) retained for audit.

**What gets ANONYMIZED** (kept for business analytics with PII stripped):
- Attendance records: `member_id` replaced with a one-way SHA-256 hash. Check-in times, methods, and class references retained for aggregate reporting.
- Payment records: PII fields (name, email, phone) stripped. Amounts, dates, payment types, and Stripe transaction IDs retained for tax and accounting compliance.

**What gets RETAINED as-is** (legal requirements):
- Audit log entries: immutable, retained for legal compliance (3-year retention).
- Payment transaction IDs: retained for tax compliance (7-year retention per IRS requirements).

#### Deletion Execution Steps

1. **Personal Data Deletion**: Delete from Member table: name, email, phone, DOB, health info, emergency contact. Delete from Parent table if no other children linked.
2. **Photo Deletion**: Delete profile photos from S3. Delete class activity photos from S3 (if member's photos). Delete from ActivityFeedPhoto table.
3. **Face Embedding Deletion**: For each kiosk device in the tenant: send deletion command via sync mechanism. Kiosk deletes embedding from encrypted local SQLite. Kiosk confirms deletion to backend.
4. **Stripe Data**: Request Stripe to delete customer data via Stripe API (if applicable). Stripe handles per their data retention policy.
5. **Attendance Anonymization**: Replace `member_id` with SHA-256 hash in all Attendance records for this member. Retain check-in times and aggregate data.
6. **Payment Anonymization**: Strip PII from Payment and Invoice records. Retain amounts, dates, and Stripe references.
7. **Audit Logging**: Create AuditLog entry: `action=Delete, entity_type=Member, reason=<deletion_reason>`. Log all individual deletion events for compliance.

---

### CCPA/CPRA Compliance (California)

**Regulatory Requirement**: California Consumer Privacy Act (CCPA) as amended by the California Privacy Rights Act (CPRA). Applies when gym serves California residents. Grants consumers rights over their personal information.

#### Categories of Personal Information Collected

| CCPA Category | Data Elements | Source |
|---------------|---------------|--------|
| Identifiers | Name, email address, phone number, account ID | Direct from user during registration |
| Protected characteristics | Age, date of birth, gender | Direct from user during registration |
| Commercial information | Payment history, membership plans, invoice records | Generated through service usage |
| Biometric information | Face geometry (128-dim embedding vector) | Derived from photos, stored on-device only (per [ADR-004](./adr/004-face-recognition-on-device.md)) |
| Internet/electronic activity | Login timestamps, IP addresses, user agent strings, pages viewed | Automatically collected during service usage |

#### Right to Know (Data Export)

**`GET /api/v1/privacy/data-export`**

- Authenticated users can request a full export of all personal data the platform holds about them.
- Response format: JSON file containing all personal data organized by category.
- Includes: profile information, children's profiles (for parents), attendance records, payment history, consent records, notification history.
- Excludes: audit log entries (system records, not personal data export), face embeddings (on-device only, not accessible via API).
- Rate limit: 2 requests per user per 12-month period (CCPA maximum).
- Processing time: response generated within 45 days (CCPA requirement). For immediate data, the API returns synchronously. For large datasets, the API returns a `202 Accepted` with a download link sent via email when ready.

#### Right to Delete

- Handled via the Data Deletion API (see above).
- CCPA requires deletion within 45 days of a verifiable request. The 30-day grace period plus processing falls within this window.
- The `reason` field in the deletion request should be set to `ccpa_request` for CCPA-specific requests.

#### Right to Opt-Out of Sale

- Ararat does NOT sell personal data to third parties.
- The privacy policy must explicitly state: "We do not sell your personal information."
- A "Do Not Sell or Share My Personal Information" link is included in the app footer and privacy settings page for transparency, even though no sale occurs. The link navigates to the privacy policy section confirming no data is sold.

#### Right to Non-Discrimination

- Service quality, pricing, and features are unaffected by any privacy rights request.
- Members who exercise data export, deletion, or opt-out rights receive identical service.
- System does not track or flag users who make privacy requests (beyond the AuditLog entry for compliance).

#### Data Retention Schedule

| Data Type | Retention Period | Justification |
|-----------|-----------------|---------------|
| Active member personal data | While active + 1 year after withdrawal | Business operations + grace period for re-enrollment |
| Payment records (amounts, transaction IDs) | 7 years from transaction date | IRS tax compliance requirements |
| Audit log entries | 3 years from creation | Legal compliance and dispute resolution |
| Face embeddings (on-device) | Deleted immediately on withdrawal or consent revocation | BIPA compliance — no retention beyond necessity |
| Consent records (metadata) | 7 years from consent date | Proof of consent for regulatory audit |
| Anonymized attendance data | Indefinite | Aggregate analytics, cannot be linked to individuals |
| Anonymized payment data | 7 years | Tax compliance with PII stripped |

---

### Data Breach Notification Procedure

Multi-tenant data breach notification procedure covering federal (COPPA) and state (CCPA, BIPA) notification requirements.

#### 1. Detection

Automated monitoring via AWS CloudWatch alarms for:
- Unusual API access patterns (e.g., bulk data retrieval outside normal usage)
- Spike in failed authentication attempts (>100 failed logins in 5 minutes per tenant)
- Bulk data export requests outside normal patterns
- Unauthorized access attempts to admin-only endpoints
- Database query anomalies (large result sets, unusual table access patterns)

#### 2. Assessment (Within 72 Hours of Detection)

Incident response team determines:
- **Scope**: Number of records affected, which tenants (gyms) are impacted
- **Data types exposed**: Identifiers, financial data, biometric metadata, children's data
- **Attack vector**: How the breach occurred (API vulnerability, credential compromise, insider threat)
- **Ongoing risk**: Whether the vulnerability is still active
- **Child data involvement**: If data of children under 13 was exposed (triggers COPPA-specific response)

#### 3. Internal Notification (Immediate Upon Confirmed Breach)

- System admin receives automated alert via PagerDuty/email
- Gym owner(s) (관장님) of affected tenant(s) notified via email and in-app notification
- If multi-tenant breach: all affected gym owners notified independently
- Internal incident record created in the system with unique incident ID

#### 4. User Notification

Notification to affected users within timeframes required by applicable state law:
- **California (CCPA)**: "Expedient" notification without unreasonable delay
- **Illinois (BIPA)**: Notification without unreasonable delay
- **General**: Within 72 hours of breach confirmation as best practice

Notification channels (all channels used simultaneously for affected users):
- **Email**: Sent to all affected users' registered email addresses
- **In-app notification**: Persistent banner in parent app and admin app
- **SMS**: For users without email on file (parents who registered with phone only)

Notification content (trilingual — English, Korean, Spanish):
- Description of the breach (what happened)
- Types of personal information involved
- Steps being taken to address the breach
- Steps users should take (e.g., monitor accounts, change passwords)
- Contact information for questions

#### 5. Authority Notification

- If breach affects >500 California residents: notify the California Attorney General
- If children's data (COPPA-covered) is involved: notify the FTC
- State-specific notification requirements are evaluated based on the residency of affected users (determined by gym location / tenant address)

#### 6. Remediation (Immediate)

- Force password reset for all affected user accounts
- Revoke all active sessions (clear Redis session store for affected users)
- Rotate API keys and service credentials if applicable
- Patch the vulnerability that caused the breach
- If kiosk device tokens were compromised: revoke and re-issue device tokens

#### 7. Post-Incident

- Document the full incident in an incident report: timeline, scope, root cause, remediation steps
- Update security measures based on findings (new CloudWatch rules, additional rate limiting, etc.)
- Conduct security review of related systems
- Update this TRD section if new compliance requirements are identified
- Retain incident report for 7 years (legal compliance)

---

### Service Dependencies

#### Services This Feature Consumes

| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| Auth Service | `api/` | `/api/v1/auth/refresh`, `/api/v1/auth/verify` | POST | JWT token | `{ valid: boolean, userId, tenantId, role }` |
| Notification Service | `api/` | Internal service call | — | `{ recipientId, type, templateData }` | `{ notificationId, status }` |
| Stripe API | External | `/v1/customers/:id` | DELETE | Stripe customer ID | `{ deleted: boolean }` |
| Kiosk Sync | `api/` | `/api/v1/kiosks/:id/commands` | POST | `{ command: 'delete_embedding', memberId }` | `{ acknowledged: boolean }` |
| S3 | AWS | S3 API | DELETE | Object key(s) | `{ deleted: boolean }` |

#### Contracts This Feature Exposes

| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/members/:id/deletion-request` | POST | Parent App, Admin App | `{ reason: string }` | `{ requestId, scheduledDeletionDate, status }` |
| `/api/v1/deletion-requests/:id` | GET | Parent App, Admin App | — | `{ requestId, status, scheduledDeletionDate, reason }` |
| `/api/v1/deletion-requests/:id/cancel` | POST | Parent App, Admin App | — | `{ requestId, status: 'cancelled' }` |
| `/api/v1/privacy/data-export` | GET | Parent App | — | JSON file with all personal data |


### Penetration Testing

- Annual third-party security audit.
- Vulnerability scanning: automated weekly.
- Bug bounty program (future).

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
