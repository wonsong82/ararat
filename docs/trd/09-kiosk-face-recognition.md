# 9. Feature: Kiosk & Face Recognition

**Related TRDs**: [07-registration](./07-registration.md), [08-attendance](./08-attendance.md), [19-security-compliance](./19-security-compliance.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md), [ADR-007](./adr/007-no-gps-notifications.md)  
**Phase**: Phase 2

---

### Architecture

The kiosk app runs natively on iPad (iOS) with full on-device face recognition:

- **Detection Layer**: Apple Vision Framework (`VNDetectFaceRectanglesRequest`, `VNDetectFaceLandmarksRequest`) for face detection and landmark extraction.
- **Recognition Layer**: KBY-AI Face Recognition SDK or FacePlugin SDK for 1:N identity matching via TensorFlow Lite.
- **Liveness Detection**: Passive liveness detection (no user action required) to prevent photo/video spoofing.
- **Processing**: 100% on-device. No biometric data transmitted to cloud.

### Enrollment Flow

1. Parent uploads 3-5 photos of child during registration via web app.
2. Photos stored temporarily in S3.
3. Kiosk device polls backend for new enrollments (daily or on-demand).
4. Backend sends photos to kiosk via secure LAN transfer (mDNS discovery, TLS).
5. Kiosk app processes photos:
   - Detect face in each photo.
   - Extract face landmarks.
   - Generate 128-dimensional face embedding using on-device ML model.
   - Store embedding in encrypted local SQLite database (iOS Keychain encryption).
   - Delete original photos from kiosk (not stored).
6. Kiosk confirms enrollment to backend: "Enrollment complete for [member_id]".
7. Backend deletes temporary photos from S3.

### Check-In Flow

1. Child approaches kiosk.
2. Kiosk camera captures frame.
3. Apple Vision detects face in frame.
4. If face detected:
   - Extract face landmarks.
   - Generate embedding from live frame.
   - Compare embedding against all enrolled embeddings using cosine similarity.
   - If match found (similarity >= threshold, default 0.85):
     - Display greeting: member photo + name + "Welcome, [name]!"
     - Play confirmation sound.
     - Log attendance with method=FaceRecognition.
     - Send parent notification.
   - If no match:
     - Fall through to QR code scan.
5. If no face detected:
   - Fall through to QR code scan.
6. QR Code Scan:
   - Prompt to scan QR code from membership card or parent's phone.
   - If QR scanned: look up member, log attendance with method=QRCode.
   - If no QR: fall through to name search.
7. Name Search:
   - Display keyboard for manual search.
   - As parent types, show matching members with photos.
   - Parent selects member.
   - Log attendance with method=NameSearch.
8. Staff Assistance:
   - If all methods fail: alert staff.
   - Staff manually marks member as checked in via staff app.
   - Log attendance with method=Manual.

### Offline Mode

Kiosk can operate offline for up to 24 hours:

1. When offline: check-ins stored in local SQLite database.
2. When connectivity restored: sync check-ins to backend via batch API.
3. Backend processes batch: creates Attendance records, triggers notifications.
4. Parent notifications sent on sync (may be delayed).

### Re-Enrollment

For members under 10 years old:
- System prompts re-enrollment every 6-12 months (configurable).
- Parent receives notification: "Please update [child_name]'s photos for face recognition".
- Parent uploads new photos via app.
- Kiosk downloads and processes new photos.
- Old embeddings replaced with new ones.

### Multi-Kiosk Sync

For gyms with multiple kiosks:
1. Embeddings sync between devices via encrypted local network (mDNS discovery, TLS).
2. No cloud intermediary for biometric data.
3. Sync happens daily or on-demand.

### Data Lifecycle

Face embeddings are deleted when:
1. Member withdraws.
2. Parent revokes face recognition consent.
3. Gym offboards.
4. Data retention period expires (configurable, default 7 years).

Deletion is logged in AuditLog with: actor, timestamp, member_id, reason.

### Kiosk Management

Backend tracks registered kiosks per gym:
- `Kiosk` table: kiosk_id, tenant_id, device_id, name, location, status, last_heartbeat.
- Kiosk sends heartbeat every 5 minutes.
- If heartbeat missing for 30 minutes: mark as offline, alert gym owner.
- Admin can remotely configure kiosk: face similarity threshold, re-enrollment frequency, etc.


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
