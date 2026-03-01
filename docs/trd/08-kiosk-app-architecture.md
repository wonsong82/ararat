# 8. Kiosk App Architecture

**Related TRDs**: [01-system-architecture](./01-system-architecture.md), [04-auth](./04-auth.md), [10-attendance](./10-attendance.md), [20-security-compliance](./20-security-compliance.md)  
**Related ADRs**: [ADR-004](./adr/004-face-recognition-on-device.md), [ADR-007](./adr/007-no-gps-notifications.md)  
**Phase**: Phase 2

---

## 8.1 Overview

The Ararat Kiosk App is a native iOS application designed for iPad deployment at Taekwondo gym entrances. It serves as the primary self check-in station where children (수련생) independently record their attendance without staff assistance.

### Target Users

- **Primary**: Children ages 4–16 performing self check-in
- **Secondary**: Gym staff (사범, 관장님) for enrollment, configuration, and troubleshooting

### Platform

- **Device**: iPad (10th generation or newer recommended)
- **OS**: iOS 16+
- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI
- **Orientation**: Landscape only (locked)
- **Deployment**: Single-app kiosk mode via Guided Access or MDM

### Key Design Principles

| Principle | Rationale |
|-----------|-----------|
| **Child-friendly** | Primary users are young children — UI must be intuitive without reading ability |
| **Minimal text** | Rely on icons, photos, animations, and audio instead of written instructions |
| **Visual & audio feedback** | Every action produces immediate, clear feedback (sounds, animations, colors) |
| **Zero-training-needed** | A child should be able to check in on their first visit without any instruction |
| **COPPA/BIPA compliant** | All biometric processing happens 100% on-device. No biometric data is ever transmitted to the cloud. Parental consent is required for face enrollment |
| **Offline-resilient** | The kiosk must function for at least 24 hours without network connectivity |

---

## 8.2 App Architecture

The kiosk app follows the **MVVM (Model-View-ViewModel)** pattern with SwiftUI. ViewModels own business logic and state; Views are declarative and stateless; Models represent domain data; Services encapsulate infrastructure concerns (camera, ML, network, storage).

### Project Position

The kiosk app lives at `kiosk/` in the Ararat project root. It is a native Xcode project — completely independent of the JavaScript/TypeScript tooling used by `api/` and `web/`. No pnpm, no npm, no shared packages.

```
ararat/
├── api/           # NestJS backend (standalone)
├── web/           # React monorepo (pnpm workspaces)
├── kiosk/         # ← iPad app (Swift/Xcode) — this TRD
│   ├── AraratKiosk/
│   ├── AraratKiosk.xcodeproj
│   ├── AraratKioskTests/
│   └── README.md
├── docs/
└── ...
```

The kiosk communicates with the backend exclusively via HTTPS REST API calls (see Section 8.8). It does NOT import any shared types, schemas, or packages from the web frontend.

### Directory Structure

```
kiosk/AraratKiosk/
├── App/
│   ├── AraratKioskApp.swift         # App entry point
│   └── AppDelegate.swift            # Background tasks, push notifications
├── Models/
│   ├── Member.swift                 # Member data model
│   ├── AttendanceRecord.swift       # Local attendance record
│   ├── KioskConfig.swift            # Configuration from server
│   └── FaceEmbedding.swift          # Face embedding data
├── ViewModels/
│   ├── CheckInViewModel.swift       # Main check-in flow logic
│   ├── EnrollmentViewModel.swift    # Face enrollment logic
│   ├── SyncViewModel.swift          # Backend sync logic
│   └── SettingsViewModel.swift      # Admin settings
├── Views/
│   ├── CheckIn/
│   │   ├── IdleView.swift           # Screensaver/attract screen
│   │   ├── CameraView.swift         # Face recognition camera
│   │   ├── WelcomeView.swift        # Success greeting
│   │   ├── QRScanView.swift         # QR code fallback
│   │   ├── NameSearchView.swift     # Manual name search
│   │   └── ErrorView.swift          # Failure/staff alert
│   ├── Enrollment/
│   │   ├── EnrollmentView.swift     # Face photo capture
│   │   └── ProgressView.swift       # Enrollment processing
│   └── Settings/
│       └── AdminSettingsView.swift   # Staff-only settings
├── Services/
│   ├── FaceRecognitionService.swift # Vision + TFLite pipeline
│   ├── CameraService.swift          # AVFoundation camera
│   ├── APIService.swift             # Backend communication
│   ├── SyncService.swift            # Offline sync engine
│   ├── AudioService.swift           # Sound feedback
│   └── StorageService.swift         # Encrypted SQLite
├── ML/
│   ├── FaceDetector.swift           # Apple Vision wrapper
│   ├── EmbeddingGenerator.swift     # TFLite model wrapper
│   └── LivenessChecker.swift        # Anti-spoofing
├── Utilities/
│   ├── KeychainHelper.swift         # iOS Keychain access
│   ├── NetworkMonitor.swift         # Reachability
│   └── Logger.swift                 # Structured logging
└── Resources/
    ├── face_recognition.tflite      # On-device ML model
    ├── Sounds/                      # Audio feedback files
    └── Assets.xcassets               # Images, colors
```

### Dependency Graph

```
Views → ViewModels → Services → Models
                  → ML (face pipeline)
                  → Utilities
```

- **Views** depend only on their corresponding ViewModel (injected via `@StateObject` / `@ObservedObject`)
- **ViewModels** orchestrate Services and expose published state to Views
- **Services** are stateless singletons injected into ViewModels
- **ML** modules are consumed exclusively by `FaceRecognitionService`
- **Models** are plain value types (`struct`) with `Codable` conformance

### Key Dependencies

| Library | Purpose | Notes |
|---------|---------|-------|
| SwiftUI | Declarative UI | iOS 16+ required |
| AVFoundation | Camera capture | Front-facing camera access |
| Apple Vision | Face detection & landmarks | `VNDetectFaceRectanglesRequest`, `VNDetectFaceLandmarksRequest` |
| TensorFlow Lite | Face embedding generation | On-device inference, 128-dim output |
| SQLCipher | Encrypted local database | SQLite with AES-256 encryption |
| Network (NWPathMonitor) | Connectivity monitoring | Built-in iOS framework |

---

## 8.3 Face Recognition Pipeline

The face recognition pipeline runs entirely on-device. No biometric data (face images, embeddings, or intermediate representations) is ever transmitted to the cloud or any external service.

### Pipeline Layers

```
Camera Frame (AVFoundation)
    │
    ▼
┌─────────────────────────────┐
│  1. Face Detection           │  Apple Vision Framework
│     VNDetectFaceRectangles   │  Target: < 100ms
│     VNDetectFaceLandmarks    │
└─────────────────────────────┘
    │ Detected face region + landmarks
    ▼
┌─────────────────────────────┐
│  2. Liveness Detection       │  Passive anti-spoofing
│     Texture analysis         │  No user action required
│     Depth estimation (if     │
│     available via TrueDepth) │
└─────────────────────────────┘
    │ Verified live face
    ▼
┌─────────────────────────────┐
│  3. Embedding Generation     │  TensorFlow Lite model
│     Crop & align face        │  128-dimensional vector
│     Run TFLite inference     │  Target: < 200ms
└─────────────────────────────┘
    │ 128-dim float vector
    ▼
┌─────────────────────────────┐
│  4. Matching                 │  Cosine similarity
│     Compare against enrolled │  Threshold: 0.85 (configurable)
│     embeddings in local DB   │  Top match returned
└─────────────────────────────┘
    │ Match result (member_id or no match)
    ▼
  CheckInViewModel
```

### Detection Layer

- **Framework**: Apple Vision (`import Vision`)
- **Requests**: `VNDetectFaceRectanglesRequest` for bounding box detection, `VNDetectFaceLandmarksRequest` for facial landmark extraction (eyes, nose, mouth positions)
- **Input**: Live camera frame (`CMSampleBuffer` from AVFoundation)
- **Output**: Face bounding box coordinates, facial landmarks for alignment
- **Multiple faces**: If multiple faces detected, use the largest bounding box (closest to camera)

### Recognition Layer

- **Framework**: TensorFlow Lite for iOS
- **Model**: Pre-trained face recognition model bundled in app (`face_recognition.tflite`)
- **Input**: Cropped and aligned face image (112×112 pixels, normalized)
- **Output**: 128-dimensional float32 embedding vector
- **Alignment**: Face is rotated and scaled using detected landmarks to a canonical position before inference

### Liveness Detection

- **Type**: Passive — requires no user action (no blink, no head turn)
- **Techniques**:
  - Texture analysis: detect flat/printed photo artifacts via Laplacian variance
  - Moiré pattern detection: screen replay attack prevention
  - Depth estimation: TrueDepth camera (Face ID iPads) provides depth map when available
- **Fallback**: On iPads without TrueDepth, texture analysis alone is used
- **Purpose**: Prevents check-in via holding up a photo or video of another member

### Matching

- **Algorithm**: Cosine similarity between query embedding and all enrolled embeddings
- **Threshold**: 0.85 (configurable via `KioskConfig.faceSimilarityThreshold`)
- **Result**: If highest similarity score ≥ threshold → match (return `member_id`). Otherwise → no match
- **Optimization**: For gyms with 200+ members, embeddings are indexed in memory at app launch for fast comparison

### Performance Targets

| Stage | Target Latency | Notes |
|-------|---------------|-------|
| Face detection | < 100ms | Apple Vision, hardware-accelerated |
| Liveness check | < 50ms | Lightweight texture analysis |
| Embedding generation | < 200ms | TFLite on Neural Engine / GPU |
| Matching (200 members) | < 50ms | In-memory cosine similarity |
| **Total pipeline** | **< 500ms** | From camera frame to match result |

---

## 8.4 Face Enrollment Architecture

Enrollment is the process of creating face embeddings for a member so the kiosk can recognize them during check-in. This section covers the technical pipeline — the parent-facing enrollment UX is specified in [TRD 10 (Attendance)](./10-attendance.md).

### Enrollment Pipeline

```
Parent uploads 3-5 photos (web app)
    │
    ▼
S3 temporary storage (encrypted, auto-expiry 7 days)
    │
    ▼
Backend marks member as "pending_enrollment"
    │
    ▼
Kiosk polls for pending enrollments (daily or on-demand)
    │
    ▼
Backend sends photos to kiosk via secure LAN transfer
  ├── Discovery: mDNS (Bonjour) on local network
  └── Transport: TLS-encrypted HTTP between backend and kiosk
    │
    ▼
Kiosk processes each photo:
  1. Detect face (Vision framework)
  2. Extract landmarks
  3. Generate 128-dim embedding (TFLite)
  4. Average embeddings from all photos → final embedding
    │
    ▼
Store embedding in encrypted local SQLite
  └── Encryption key in iOS Keychain
    │
    ▼
Delete original photos from kiosk device
    │
    ▼
Kiosk confirms enrollment to backend (POST /api/v1/kiosks/{id}/enrollments/{member_id}/confirm)
    │
    ▼
Backend deletes temporary photos from S3
Backend updates member status: "enrolled"
```

### Photo Requirements

- **Count**: 3–5 photos per member
- **Format**: JPEG, minimum 640×480 resolution
- **Content**: Clear frontal face, adequate lighting, no sunglasses or face coverings
- **Validation**: Backend validates photos contain exactly one detectable face before sending to kiosk

### Embedding Storage

- Each member has one averaged embedding (128 float32 values = 512 bytes)
- Stored in encrypted SQLite alongside `member_id` and enrollment timestamp
- Multiple photos are averaged into a single embedding for robustness

### Re-enrollment

- **Trigger**: Children under 10 years old — every 6–12 months (configurable via `KioskConfig.reEnrollmentMonths`)
- **Reason**: Children's facial features change significantly during growth
- **Notification**: System sends push notification to parent when re-enrollment is due
- **Process**: Same pipeline as initial enrollment
- **Old embedding**: Replaced by new embedding upon successful re-enrollment

---

## 8.5 Local Data Storage

The kiosk maintains a local encrypted SQLite database for all persistent data. This enables offline operation and ensures biometric data never leaves the device.

### Database Engine

- **SQLite** with **SQLCipher** for AES-256 encryption at rest
- **Encryption key**: Generated on first launch, stored in iOS Keychain (hardware-backed on devices with Secure Enclave)
- **Location**: App sandbox (`/Documents/ararat_kiosk.db`)

### Schema Overview

| Table | Contents | Approximate Size |
|-------|----------|-----------------|
| `members` | id, name, photo_url, belt_level, language_preference, status | ~1KB per member |
| `embeddings` | member_id, embedding_vector (BLOB, 512 bytes), enrolled_at, source | ~600 bytes per member |
| `pending_attendance` | id, member_id, check_in_time, method (face/qr/manual), synced | ~200 bytes per record |
| `kiosk_config` | key-value pairs for all configurable settings | < 10KB total |
| `sync_log` | last_sync_time, sync_type (attendance/enrollment/config), status | ~100 bytes per entry |

### Data Lifecycle

Biometric data (embeddings) is deleted from the kiosk under any of these conditions:

| Trigger | Action | Initiated By |
|---------|--------|-------------|
| Member withdrawal (탈퇴) | Delete embedding + member record | Backend push or next sync |
| Parent consent revocation | Delete embedding + member record | Backend push or next sync |
| Gym offboarding | Wipe entire kiosk database | Backend command or MDM |
| Retention expiry | Delete embedding | Automated (default: 7 years) |

- All deletions are logged in the backend `AuditLog` table with timestamp, reason, and actor
- Pending attendance records for deleted members are synced first (if possible), then purged

### Backup & Recovery

- **No cloud backup** of biometric data — by design (COPPA/BIPA compliance)
- Member profile data (non-biometric) is recoverable from backend on re-pairing
- Embeddings must be re-enrolled if the device is replaced or wiped

---

## 8.6 Multi-Kiosk Sync

Gyms with multiple kiosks (e.g., front entrance and back entrance) need all devices to share the same enrollment data. Biometric sync occurs exclusively over the local network — never via cloud.

### Discovery

- **Protocol**: mDNS (Bonjour) — standard iOS local network discovery
- **Service type**: `_ararat-kiosk._tcp.local.`
- Each kiosk broadcasts its presence and listens for peers on the same LAN

### Sync Protocol

```
Kiosk A                          Kiosk B
   │                                │
   │──── mDNS discovery ───────────▶│
   │◀─── mDNS response ────────────│
   │                                │
   │──── TLS handshake ────────────▶│
   │     (mutual device cert auth)  │
   │                                │
   │──── Sync manifest ───────────▶│
   │     (member_ids + timestamps)  │
   │                                │
   │◀─── Delta response ───────────│
   │     (new/updated embeddings)   │
   │                                │
   │──── Acknowledge ──────────────▶│
   └────────────────────────────────┘
```

### Sync Rules

| Setting | Value |
|---------|-------|
| Sync frequency | Daily automatic + on-demand manual trigger |
| Transport encryption | TLS 1.3 with mutual certificate authentication |
| Conflict resolution | Latest timestamp wins for embedding updates |
| Scope | Embeddings and member profiles only — attendance records sync to backend, not between kiosks |
| Cloud intermediary | **None** — biometric data never passes through cloud |

### Sync Trigger Events

- Daily scheduled sync (configurable time, default 2:00 AM)
- New enrollment completed on any kiosk
- Manual trigger from admin settings
- Kiosk app launch (sync check on startup)

---

## 8.7 Offline Mode

The kiosk is designed to operate for at least **24 hours** without network connectivity. All check-in operations continue using locally cached data.

### Offline Capabilities

| Feature | Offline Behavior |
|---------|-----------------|
| Face recognition check-in | ✅ Fully functional (local embeddings + local ML) |
| QR code check-in | ✅ Functional (validates against local member list) |
| Name search check-in | ✅ Functional (local member database) |
| Attendance recording | ✅ Stored locally in `pending_attendance` table |
| Parent notifications | ⏸ Delayed — sent when kiosk reconnects and syncs |
| New enrollments | ❌ Requires network (photos from backend) |
| Config updates | ❌ Uses last-known config |

### Network Monitoring

- **Framework**: `NWPathMonitor` (built-in iOS Network framework)
- Monitors connectivity state continuously in background
- State changes trigger `SyncViewModel` to attempt sync when connectivity returns

### Sync on Reconnect

When the kiosk regains connectivity:

1. **Heartbeat resumes** — backend marks kiosk as online
2. **Attendance batch sync** — all pending records sent via `POST /api/v1/kiosks/{id}/attendance/batch`
3. **Backend processing** — creates `Attendance` records, triggers delayed parent notifications
4. **Enrollment check** — polls for any pending enrollments that arrived while offline
5. **Config refresh** — fetches latest `KioskConfig` from backend

### Batch Attendance Payload

```json
{
  "records": [
    {
      "member_id": "uuid",
      "check_in_time": "2025-03-01T15:30:00Z",
      "method": "face",
      "confidence": 0.92
    }
  ]
}
```

### Visual Indicator

- **Green dot** (top-right corner): online, syncing normally
- **Orange dot** (top-right corner): offline, operating on local data
- Dot is subtle — visible to staff, not distracting to children

---

## 8.8 Backend Communication

The kiosk communicates with the Ararat backend API over HTTPS. All communication uses a device-level authentication token — not user-level auth.

### Authentication

- **Mechanism**: Long-lived device token issued during initial kiosk pairing
- **Pairing flow**: Admin enters a one-time pairing code on the kiosk → kiosk exchanges code for device token with backend
- **Token storage**: iOS Keychain (hardware-backed)
- **Token refresh**: Device tokens are long-lived (1 year) with silent refresh
- **Header**: `Authorization: Bearer <device_token>`

### Heartbeat

| Setting | Value |
|---------|-------|
| Endpoint | `POST /api/v1/kiosks/{id}/heartbeat` |
| Frequency | Every 5 minutes |
| Payload | `{ "status": "online", "battery": 85, "pending_records": 3, "version": "1.2.0" }` |
| Offline alert | If heartbeat missing for 30 minutes, backend marks kiosk offline and sends alert to gym owner (관장님) |

### API Endpoints Consumed

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/kiosks/{id}/heartbeat` | POST | Status heartbeat |
| `/api/v1/kiosks/{id}/config` | GET | Fetch kiosk configuration |
| `/api/v1/kiosks/{id}/members` | GET | Fetch member list (paginated) |
| `/api/v1/kiosks/{id}/enrollments/pending` | GET | Fetch pending enrollment photos |
| `/api/v1/kiosks/{id}/enrollments/{member_id}/confirm` | POST | Confirm enrollment complete |
| `/api/v1/kiosks/{id}/attendance` | POST | Report single check-in |
| `/api/v1/kiosks/{id}/attendance/batch` | POST | Batch sync offline records |

### Communication Rules

- All communication over **HTTPS** (TLS 1.2+)
- Request timeout: 10 seconds (prevent UI blocking)
- Retry policy: exponential backoff (1s, 2s, 4s, 8s, max 60s) for failed requests
- Large payloads (enrollment photos): chunked transfer with resume support

---

## 8.9 Kiosk Management

The backend maintains a registry of all kiosks across all tenants for monitoring, configuration, and remote management.

### Kiosk Data Model

```
Kiosk
├── kiosk_id          UUID (PK)
├── tenant_id         UUID (FK → Tenant)
├── device_id         String (iOS device identifier)
├── name              String (e.g., "Front Entrance Kiosk")
├── location          String (e.g., "Main lobby")
├── status            Enum: online | offline | pairing | decommissioned
├── last_heartbeat    DateTime
├── app_version       String
├── os_version        String
├── paired_at         DateTime
├── paired_by         UUID (FK → Admin who paired)
├── config            JSONB (kiosk-specific config overrides)
├── created_at        DateTime
└── updated_at        DateTime
```

### Remote Configuration

Administrators can remotely configure kiosk behavior from the Admin App. Configuration is delivered to the kiosk via polling (piggy-backed on the heartbeat cycle, every 5 minutes).

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `faceSimilarityThreshold` | Float | 0.85 | Minimum cosine similarity for face match |
| `reEnrollmentMonths` | Integer | 9 | Months between re-enrollment prompts (children < 10) |
| `idleTimeoutSeconds` | Integer | 10 | Seconds of inactivity before returning to idle screen |
| `checkInMode` | Enum | `all` | `face_only`, `face_qr`, `all` (face + QR + name search) |
| `enableAudio` | Boolean | true | Play sound effects on check-in |
| `enableTTS` | Boolean | false | Speak child's name on check-in (text-to-speech) |
| `offlineAlertMinutes` | Integer | 30 | Minutes before backend raises offline alert |
| `language` | String | `en` | Kiosk UI language (en, ko, es) |

### Kiosk Lifecycle

```
     ┌──────────┐
     │ Pairing  │ ◀── Admin enters pairing code
     └────┬─────┘
          │ Device token issued
          ▼
     ┌──────────┐
     │  Online  │ ◀── Normal operation, heartbeat every 5min
     └────┬─────┘
          │ Heartbeat missing > 30min
          ▼
     ┌──────────┐
     │ Offline  │ ◀── Alert sent to gym owner
     └────┬─────┘
          │ Heartbeat resumes
          ▼
     ┌──────────┐
     │  Online  │
     └────┬─────┘
          │ Admin decommissions
          ▼
  ┌────────────────┐
  │ Decommissioned │ ── Embeddings wiped, token revoked
  └────────────────┘
```

---

## 8.10 Child-Friendly UX Principles

These are architectural design guidelines for the kiosk UI — not screen-by-screen specifications. Individual screen layouts and check-in flows are defined in [TRD 10 (Attendance)](./10-attendance.md).

### Visual Design

- **Large, colorful, high-contrast** UI elements — designed for visibility from 2–3 feet away
- **Minimal text** — rely on icons, member photos, animations, and color coding
- **Touch targets**: Minimum **60px × 60px** (larger than Apple's standard 44px — children's fingers are less precise and less coordinated)
- **Font sizes**: Minimum 24pt for any displayed text; member names at 32pt+
- **Color coding**: Green = success, Red = error/try again, Blue = informational
- **Belt color display**: Show the member's belt color prominently on welcome screen (familiar visual for Taekwondo students)

### Audio Feedback

| Event | Audio |
|-------|-------|
| Face detected | Subtle camera shutter sound |
| Check-in success | Cheerful chime + optional TTS greeting ("Welcome, [Name]!") |
| Check-in failure | Gentle error tone (not alarming — children should not feel embarrassed) |
| QR scan success | Confirmation beep |
| Idle wake | Soft attention sound |

- TTS (text-to-speech): Optional, configurable per kiosk. Uses iOS `AVSpeechSynthesizer`
- All sounds configurable: enable/disable per sound type in admin settings
- Volume: Controlled via kiosk config, not device hardware buttons

### Animations

- **Face detection overlay**: Real-time bounding box around detected face (green when aligned, yellow when adjusting)
- **Success animation**: Checkmark with confetti/sparkle effect, member photo slide-in with name
- **Welcome slide-in**: Member's name and belt color slide in from the side
- **Idle screensaver**: Gym branding, rotating announcements, or gentle animation; motion detection (via camera) to wake

### Interaction Model

- **No keyboard input** during normal check-in flow — face recognition and QR are zero-touch
- Keyboard appears **only** for name search fallback (on-screen keyboard)
- **Auto-return to idle**: After 10 seconds of inactivity on any screen, return to idle/attract screen
- **No swipe gestures** — children may accidentally trigger them. Tap-only interaction
- **Landscape orientation**: Locked. No rotation support

### Kiosk Mode

- **Guided Access** (built-in iOS): Locks iPad to single app, disables hardware buttons
- **MDM (Mobile Device Management)**: For enterprise deployment — remote app install, config, wipe
- **Auto-launch**: App configured to launch on device boot
- **No status bar**: Hidden in kiosk mode for immersive experience

---

## 8.11 Security & Compliance

Kiosk-specific security measures supplement the platform-wide security policies defined in [TRD 20 (Security & Compliance)](./20-security-compliance.md).

### COPPA Compliance (Children's Online Privacy Protection Act)

- **Parental consent required**: Face enrollment only proceeds after parent explicitly opts in via the web app
- **No child data collection without consent**: Kiosk cannot enroll a child whose parent has not consented
- **Minimal data collection**: Only face embedding (mathematical representation) is stored — not face images
- **Data deletion**: Parents can revoke consent at any time; embedding is deleted on next kiosk sync
- **No third-party sharing**: Biometric data never leaves the device

### BIPA Compliance (Biometric Information Privacy Act)

- **Written consent**: Obtained from parent/guardian via web app before any biometric data is created
- **On-device processing**: All biometric computation (detection, embedding, matching) runs on the iPad
- **No cloud transmission**: Zero biometric data (images, embeddings, scores) transmitted over any network except encrypted local kiosk-to-kiosk sync
- **Retention policy**: Embeddings stored no longer than necessary; deleted on withdrawal, consent revocation, or retention expiry (default 7 years, configurable)
- **Destruction protocol**: When embeddings are deleted, the record is securely overwritten and deletion is logged in backend AuditLog

### Data Encryption

| Layer | Mechanism |
|-------|-----------|
| At rest (database) | SQLCipher — AES-256 encrypted SQLite |
| At rest (encryption key) | iOS Keychain — hardware-backed Secure Enclave |
| At rest (device token) | iOS Keychain |
| In transit (backend API) | TLS 1.2+ (HTTPS) |
| In transit (kiosk-to-kiosk) | TLS 1.3 with mutual certificate authentication |
| In transit (LAN enrollment) | TLS-encrypted HTTP |

### Physical Security

- **Guided Access**: Prevents app switching, disables Home button, disables Control Center
- **MDM enrollment**: Remote wipe capability if device is stolen
- **Auto-lock**: Returns to idle screen after configurable inactivity period — no sensitive data visible
- **No data on screen**: Member list and embeddings are never displayed on the kiosk screen during normal operation
- **Admin access**: Settings screen protected by staff PIN code (4–6 digits, configurable)

### Incident Response

- If a kiosk device is lost or stolen:
  1. Admin marks kiosk as decommissioned in Admin App
  2. Backend revokes device token immediately
  3. MDM triggers remote wipe (if enrolled)
  4. SQLCipher encryption protects data at rest until wipe completes
  5. Incident logged in AuditLog

---

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
