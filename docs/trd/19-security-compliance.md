# 19. Security & Compliance Implementation

**Related TRDs**: [07-registration](./07-registration.md), [09-kiosk-face-recognition](./09-kiosk-face-recognition.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md), [ADR-007](./adr/007-no-gps-notifications.md)  
**Phase**: MVP (Phase 1)

---

### COPPA Compliance (Children Under 13)

**Regulatory Requirement**: FTC's amended COPPA Rule (effective June 2025) explicitly includes biometric identifiers in "personal information." Compliance deadline: April 22, 2026.

**Implementation**:

1. **Consent Collection**:
   - During child registration, if DOB < 13 years: system blocks data storage.
   - System displays consent form in parent's preferred language (English, Korean, Spanish).
   - Consent form includes:
     - Explanation of data collection (name, DOB, health info, photos).
     - Explanation of face recognition enrollment (if applicable).
     - Parent's acknowledgment of parental authority.
   - Parent electronically signs (checkbox + timestamp).
   - System stores Consent record: { parent_id, member_id, consent_date, language, form_version }.

2. **Data Storage**:
   - Only after consent verified: system allows data storage.
   - All data marked with consent_id for tracking.

3. **Consent Revocation**:
   - Parent can revoke consent at any time via app settings.
   - On revocation: trigger data deletion workflow (see below).

4. **Data Retention Policy**:
   - Written policy published on website (trilingual).
   - Policy states: data retained for 7 years (COPPA requirement) or until member withdraws.
   - Policy available in English, Korean, Spanish.

### BIPA Compliance (Biometric Data)

**Regulatory Requirement**: Illinois Biometric Information Privacy Act (BIPA) requires written consent before collecting biometric identifiers. Each scan is a separate violation ($1,000-$5,000).

**Implementation**:

1. **Written Consent**:
   - Before face enrollment: system displays BIPA consent form.
   - Form includes:
     - Explanation of biometric data collection and use.
     - Explanation of data retention and deletion.
     - Acknowledgment that parent understands and consents.
   - Parent electronically signs.
   - System stores BiometricConsent record.

2. **Retention Policy**:
   - Publicly available on website.
   - States: embeddings retained while member active, deleted on withdrawal or consent revocation.
   - No sale or profit from biometric data.

3. **Deletion Workflow**:
   - On withdrawal or consent revocation: delete embeddings from all kiosk devices.
   - Log deletion event in AuditLog.

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

### Data Deletion Workflow

On withdrawal or consent revocation:

1. **Personal Data Deletion**:
   - Delete from Member table: name, email, phone, health info.
   - Delete from Parent table: name, email, phone.
   - Delete from Attendance table: member_id references (but keep anonymized attendance counts for analytics).

2. **Photo Deletion**:
   - Delete profile photos from S3.
   - Delete class activity photos from S3 (if member's photos).
   - Delete from ActivityFeedPhoto table.

3. **Face Embedding Deletion**:
   - For each kiosk device: delete embeddings from local encrypted database.
   - Kiosk confirms deletion to backend.
   - Log deletion in AuditLog.

4. **Stripe Data**:
   - Request Stripe to delete customer data (if applicable).
   - Stripe handles per their data retention policy.

5. **Audit Logging**:
   - Create AuditLog entry: action=Delete, entity_type=Member, reason=Withdrawal/ConsentRevocation.
   - Log all deletion events for compliance.

6. **Anonymized Data Retention**:
   - Retain anonymized analytics data (attendance counts, payment history) for reporting.
   - Data cannot be linked back to individual.

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
