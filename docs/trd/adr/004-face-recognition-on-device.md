# ADR-004: Face Recognition On-Device Only

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

Face recognition is the primary check-in method at the kiosk. Children (often under 13) are the main users. Biometric data is heavily regulated — COPPA (federal, amended rule effective June 2025, compliance deadline April 22, 2026) explicitly includes biometric identifiers. Illinois BIPA imposes $1,000-$5,000 per-scan penalties for violations. Sending biometric data to the cloud creates compliance risk and parent trust issues.

## Decision

All face recognition processing happens on-device (iPad). Face embeddings are generated, stored, and matched locally. No biometric data is ever transmitted to the cloud. Detection uses Apple Vision Framework; recognition uses KBY-AI/FacePlugin SDK with TensorFlow Lite. Embeddings stored in encrypted local SQLite (iOS Keychain encryption).

## Alternatives Considered

1. **Cloud-based face recognition (AWS Rekognition, Google Cloud Vision)**: Easier to implement, but transmits biometric data to cloud. Rejected — COPPA/BIPA compliance risk, parent trust issue.
2. **Apple Vision Framework only**: Free and built-in, but Apple Vision only does detection (face rectangles, landmarks), NOT recognition (1:N identity matching). Rejected as sole solution.
3. **Hybrid (detection on-device, recognition in cloud)**: Still transmits face data. Rejected.
4. **On-device with KBY-AI/FacePlugin (chosen)**: Apple Vision for detection + third-party SDK for recognition, all on-device. Accepted.

## Consequences

**Positive**:
- Zero cloud transmission of biometric data
- COPPA/BIPA compliant by architecture (not just policy)
- Parent trust — "your child's face data never leaves the iPad"
- No cloud API costs for face recognition
- Works offline (no network dependency for matching)

**Negative**:
- Limited to iPad hardware capabilities (max ~500 enrolled members per device)
- Multi-kiosk sync requires encrypted LAN transfer
- Re-enrollment needed every 6-12 months for children under 10 (growth changes)
- Third-party SDK licensing cost (KBY-AI/FacePlugin)
- Cannot centrally update ML models — must push to each device

**Affects**: [09-kiosk-face-recognition](../09-kiosk-face-recognition.md), [07-registration](../07-registration.md), [19-security-compliance](../19-security-compliance.md)
