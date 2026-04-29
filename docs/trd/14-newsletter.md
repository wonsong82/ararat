# 14. Feature: Newsletter System (가정통신문)

**Related TRDs**: [06-i18n](./06-i18n.md), [13-notifications](./13-notifications.md), [19-file-storage](./19-file-storage.md)  
**Related ADRs**: _None_  
**Phase**: Phase 2

---

### Overview

Formal gym-to-parent communication (가정통신문) with a WYSIWYG editor, audience targeting, trilingual support, multi-channel delivery, and read receipt analytics. Newsletters are the primary way gym owners share structured announcements — belt test schedules, holiday closures, policy changes, event invitations — with parents in their preferred language.

### Newsletter Model

```
Newsletter:
  - newsletter_id (PK, UUID)
  - tenant_id (FK → Tenant)
  - title (JSON — { en: string, ko: string, es: string })
  - body (JSON — { en: string (rich HTML), ko: string, es: string })
  - attachments (JSON array of S3 URLs)
  - audience_filter (JSON — { classes?, belt_levels?, age_groups?, membership_status? })
  - status (enum: Draft, Scheduled, Sent)
  - scheduled_at (nullable, timestamptz)
  - sent_at (nullable, timestamptz)
  - created_by (FK → Staff)
  - created_at (timestamptz)
  - updated_at (timestamptz)

NewsletterRecipient:
  - id (PK, UUID)
  - newsletter_id (FK → Newsletter)
  - parent_id (FK → Parent)
  - language (enum: en, ko, es — language used for rendering)
  - channel (enum: in_app, email, sms)
  - delivered_at (nullable, timestamptz)
  - read_at (nullable, timestamptz)
  - created_at (timestamptz)
```

### Business Rules

1. **Status lifecycle**: Draft → Scheduled (optional) → Sent. Only Draft and Scheduled newsletters can be edited. Sent newsletters are immutable.
2. **Delete**: Only Draft newsletters can be deleted. Scheduled newsletters must be reverted to Draft first.
3. **Trilingual content**: Title and body are stored per-language. Admin writes content in up to three languages (EN/KO/ES). If a language variant is missing, the system falls back to English.
4. **Attachments**: Uploaded via presigned S3 URLs (see [TRD 19](./19-file-storage.md)). Stored as an array of S3 object keys. Maximum 5 attachments, 10 MB each.
5. **Audience filter**: Evaluated at **send time**, not at schedule time — ensures the recipient list reflects current membership state.
6. **Permissions**: Only Admin and Owner roles can create/edit/send newsletters.

### Template Editor

Admin creates newsletters with a WYSIWYG editor:

1. Admin clicks "Create Newsletter".
2. Admin enters title and body using a rich text editor with formatting, images, and links.
3. Admin writes content in up to three languages using the language tab switcher (EN/KO/ES).
4. Admin uploads attachments (documents, PDFs, images) — stored via presigned S3 upload.
5. Admin selects audience filter (class, belt level, age group, membership status).
6. Admin can preview the newsletter as it will appear to parents in each language.
7. Admin saves as Draft or schedules for future delivery.

### Audience Targeting

Audience filter evaluated at send time (not at schedule time):

```json
{
  "classes": ["uuid1", "uuid2"],
  "belt_levels": ["uuid3", "uuid4"],
  "age_groups": ["Kids", "Teens"],
  "membership_status": ["Active"]
}
```

- If **multiple filter categories** are specified: AND logic (all must match).
- Within a single category (e.g., multiple classes): OR logic (member in any listed class).
- If **no filter** specified: all active parents in the tenant receive the newsletter.
- Filter evaluation joins: `Member` → `ParentChild` → `Parent` to resolve recipient parents.
- A parent receives one copy even if multiple children match the filter.

### Delivery

When a newsletter is sent (manually or by scheduled cron):

1. Newsletter status changes to `Sent`, `sent_at` is recorded.
2. System evaluates `audience_filter` against current member data to find matching parents.
3. For each matching parent:
   - Fetch parent's preferred language.
   - Select the newsletter body in that language (fallback to English if missing).
   - Dispatch to channels:
     - **In-App**: Create in-app notification with link to full newsletter. Delegates to NotificationService (see [TRD 13](./13-notifications.md)).
     - **Email**: Send full HTML newsletter via SendGrid. Includes attachments as links.
     - **SMS**: Send summary (title + first 160 chars) via Twilio with link to full newsletter in-app.
   - Create `NewsletterRecipient` record per parent per channel.
4. Delivery is processed asynchronously via SQS job queue to handle large recipient lists without blocking.

### Read Receipts

System tracks which parents have read the newsletter:

1. When a parent opens a newsletter in the app, the system records `read_at` timestamp in `NewsletterRecipient`.
2. Read receipt is sent once — subsequent opens do not overwrite the timestamp.
3. Admin analytics dashboard shows:
   - Total recipients count
   - Read count and read percentage
   - List of parents who read / didn't read (with timestamps)
   - Read rate over time (line graph)

### Scheduled Publishing

1. Admin creates a newsletter and selects "Schedule for later".
2. Admin selects `scheduled_at` date and time (must be in the future).
3. System stores newsletter with `status = Scheduled`.
4. A cron job runs every minute, querying newsletters where `scheduled_at <= NOW()` AND `status = Scheduled`.
5. For each matching newsletter: enqueue a delivery job (same flow as manual send).
6. If the cron picks up multiple newsletters in one run, each is processed independently.

---

### Backend

#### API Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/newsletters` | List newsletters (filterable by status, date range) | Admin |
| `POST` | `/api/v1/tenants/{tenantId}/newsletters` | Create newsletter (Draft) | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/newsletters/{id}` | Get newsletter detail | Admin, Parent |
| `PATCH` | `/api/v1/tenants/{tenantId}/newsletters/{id}` | Update newsletter (Draft/Scheduled only) | Admin |
| `DELETE` | `/api/v1/tenants/{tenantId}/newsletters/{id}` | Delete newsletter (Draft only) | Admin |
| `POST` | `/api/v1/tenants/{tenantId}/newsletters/{id}/send` | Send newsletter immediately | Admin |
| `POST` | `/api/v1/tenants/{tenantId}/newsletters/{id}/schedule` | Schedule newsletter for future delivery | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/newsletters/{id}/analytics` | Read receipt analytics | Admin |
| `POST` | `/api/v1/tenants/{tenantId}/newsletters/{id}/read` | Record read receipt | Parent |

**List newsletters** (`GET /newsletters`):
- Admin: sees all newsletters for the tenant.
- Query params: `status`, `from_date`, `to_date`, `page`, `limit`.

**Get newsletter detail** (`GET /newsletters/{id}`):
- Admin: full detail including all language variants and analytics summary.
- Parent: rendered body in parent's preferred language, attachments, and read status.

**Create/Update** (`POST` / `PATCH`):
- Body: `{ title: { en, ko?, es? }, body: { en, ko?, es? }, attachments?: string[], audience_filter?: object, scheduled_at?: string }`.
- English title and body are required. Korean and Spanish are optional.

**Send** (`POST /newsletters/{id}/send`):
- Only valid for Draft or Scheduled newsletters.
- Enqueues delivery job and returns `202 Accepted`.

**Record read receipt** (`POST /newsletters/{id}/read`):
- Called automatically by the Parent App when a newsletter is opened.
- Idempotent — subsequent calls are no-ops.

#### Service Logic

- **NewsletterService**: Core CRUD, status transitions, validation (e.g., prevent editing Sent newsletters, prevent deleting non-Draft).
- **NewsletterAudienceService**: Evaluates `audience_filter` against current member/parent data. Resolves deduplicated parent list via `Member → ParentChild → Parent` joins.
- **NewsletterDeliveryService**: Orchestrates multi-channel dispatch. For each recipient: resolves language, renders content, delegates channel dispatch to NotificationService (see [TRD 13](./13-notifications.md)), and creates `NewsletterRecipient` records.
- **NewsletterAnalyticsService**: Aggregates read receipt data — total/read counts, read percentage, per-parent read status, read rate time series.
- **NewsletterSchedulerCron**: Runs every minute. Queries `Scheduled` newsletters past their `scheduled_at` time. Enqueues delivery jobs.

**Job queue** (SQS):
- `newsletter.delivery.send` — processes newsletter delivery for a single newsletter (audience evaluation + dispatch).
- `newsletter.delivery.recipient` — processes delivery for a single recipient (fan-out from above for large lists).

---

### Parent/Member App

#### Screen: Newsletters (`/newsletters`)

Displays the list of 가정통신문 from the gym in the parent's preferred language.

**List View**:
- Reverse-chronological list of newsletters.
- Each item shows: title, date sent, preview snippet (first ~100 chars of body, plain text), and read/unread badge.
- Unread newsletters are visually distinguished (bold title, dot indicator).

**Detail View** (`/newsletters/{id}`):
- Full newsletter rendered as HTML content in parent's preferred language.
- Attachments listed as downloadable links.
- System automatically sends read receipt (`POST /newsletters/{id}/read`) when the detail view is opened.

**Key Components**:
- `NewsletterList` — paginated list with `Card` per newsletter item
- `NewsletterCard` — title, date, snippet, read/unread indicator
- `NewsletterViewer` — rendered HTML/markdown content with attachment links
- `ReadReceiptIndicator` — visual badge showing read/unread status

**Data Sources**:
- `useNewsletters()` — paginated list of newsletters for the parent's tenant
- `useNewsletter(id)` — single newsletter detail with rendered content

**User Actions**:
- Tap a newsletter card to open the full content
- Scroll through the newsletter body
- Download attachments
- Read receipt is sent automatically on open (no user action required)

---

### Admin App

#### Screen: Newsletter Management (`/newsletters`)

Full newsletter creation, editing, audience targeting, and read receipt analytics.

**List View**:
- TanStack Table with columns: Title, Date Sent, Audience (summary tag), Read Rate (%), Status (Draft / Scheduled / Sent).
- Sortable and filterable by status and date range.
- Row actions: Edit (Draft/Scheduled), View Analytics (Sent), Duplicate, Delete (Draft only).

**Editor View** (`/newsletters/new`, `/newsletters/{id}/edit`):
- WYSIWYG rich text editor for newsletter body with formatting toolbar (bold, italic, headings, lists, images, links).
- **Language tab switcher** (EN / KO / ES) — admin writes content in each language in a separate tab. English is required; Korean and Spanish are optional.
- Title input field (per-language, matching the active tab).
- Attachment uploader — drag-and-drop or file picker, shows uploaded file list with remove option.
- **Audience selector**:
  - "All members" toggle (default on).
  - When toggled off: multi-select dropdowns for Class, Belt Level, Age Group, Membership Status.
  - Selected filters shown as tags below the selector.
- **Preview mode** — renders the newsletter as parents will see it, switchable between languages.
- Action buttons: Save Draft, Schedule (opens date/time picker), Send Now (with confirmation dialog).

**Analytics View** (`/newsletters/{id}/analytics`):
- Summary cards: Sent Count, Read Count, Read Rate (%).
- Read rate over time — line chart (x: days since sent, y: cumulative read %).
- Recipient table: Parent Name, Language, Delivered At, Read At, Status (Read / Unread).
- Export recipient list as CSV.

**Key Components**:
- `NewsletterTable` — TanStack Table with sorting, filtering, pagination
- `NewsletterEditor` — WYSIWYG editor with language tabs, audience selector, attachment uploader
- `AudienceSelector` — multi-select filter component with tag display
- `LanguageTabSwitcher` — EN/KO/ES tabs syncing with editor content
- `NewsletterPreview` — read-only rendered view per language
- `AnalyticsCards` — sent count, read count, read rate summary
- `ReadRateChart` — line chart (Recharts) for read rate over time
- `RecipientTable` — TanStack Table with read/unread status per parent

**Data Sources**:
- `useNewsletters()` — paginated list with filters
- `useNewsletter(id)` — newsletter detail for editing
- `useNewsletterAnalytics(id)` — read receipt analytics (summary + per-recipient)

**User Actions**:
- Create new newsletter (opens editor)
- Edit draft or scheduled newsletter
- Write content in WYSIWYG editor with formatting, images, links
- Write content in all three languages (tab per language)
- Select target audience using filter dropdowns
- Preview newsletter as it will appear to parents (per language)
- Send newsletter immediately (with confirmation)
- Schedule newsletter for future delivery
- View read receipt analytics (who read, when, read rate graph)
- Duplicate an existing newsletter as a template for a new one
- Export recipient analytics as CSV

---

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| SendGrid | External | `POST /v3/mail/send` | POST | `{ personalizations, from, subject, content, attachments }` | `202 Accepted` |
| File Service | Internal (TRD 19) | Upload newsletter attachments | Internal call | `{ file, category: "newsletter_attachment" }` | `{ fileId, url }` |
| Notification Service | Internal (TRD 13) | Push notification for new newsletter | Internal call | `{ recipientId, templateId, data }` | `{ notificationId }` |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/tenants/{tenantId}/newsletters` | GET/POST | Admin App, Parent App | Newsletter data JSON | Newsletter list or detail |
| `/api/v1/tenants/{tenantId}/newsletters/{id}` | GET/PATCH/DELETE | Admin App | Newsletter update JSON | Newsletter detail |
| `/api/v1/tenants/{tenantId}/newsletters/{id}/publish` | POST | Admin App | `{ audience, channels }` | Publish confirmation |
| `/api/v1/tenants/{tenantId}/newsletters/{id}/recipients` | GET | Admin App | `?status=&cursor=&limit=` | Recipient list with read status |


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
