# 13. Feature: Newsletter System (가정통신문)

**Related TRDs**: [06-i18n](./06-i18n.md), [12-notifications](./12-notifications.md), [18-file-storage](./18-file-storage.md)  
**Related ADRs**: _None_  
**Phase**: Phase 2

---

### Newsletter Model

```
Newsletter:
  - newsletter_id
  - tenant_id
  - title
  - body (rich HTML content)
  - attachments (JSON array of S3 URLs)
  - audience_filter (JSON, e.g., { classes: [uuid1, uuid2], belt_levels: [uuid3], age_groups: ["Kids"], membership_status: ["Active"] })
  - status (Draft, Scheduled, Sent)
  - scheduled_at (nullable)
  - sent_at (nullable)
  - created_by (staff_id)
  - created_at
  - updated_at
```

### Template Editor

Admin creates newsletters with WYSIWYG editor:
1. Admin clicks "Create Newsletter".
2. Admin enters title and body (rich text editor with formatting, images, etc.).
3. Admin uploads attachments (documents, PDFs, etc.).
4. Admin selects audience filter (class, belt level, age group, membership status).
5. Admin selects delivery channels (in-app, email, SMS).
6. Admin can preview newsletter.
7. Admin saves as draft or schedules for future delivery.

### Audience Targeting

Audience filter evaluated at send time (not at schedule time):

```
audience_filter: {
  classes: [uuid1, uuid2],           // Only parents of children in these classes
  belt_levels: [uuid3, uuid4],       // Only parents of children at these belt levels
  age_groups: ["Kids", "Teens"],     // Only parents of children in these age groups
  membership_status: ["Active"]      // Only parents of active members
}
```

If multiple filters specified: use AND logic (all must match).

### Delivery

1. Newsletter status changes to Sent.
2. System finds all matching parents based on audience_filter.
3. For each parent:
   - Fetch preferred language.
   - Render newsletter in that language (if multilingual variants exist).
   - Dispatch to selected channels:
     - **In-App**: Create notification, store in database.
     - **Email**: Send via SendGrid.
     - **SMS**: Send via Twilio (summary only, link to full newsletter in app).
4. Create NewsletterRecipient record for each parent.
5. Log delivery in NotificationLog.

### Read Receipts

System tracks which parents have read the newsletter:

1. When parent opens newsletter in app: system records read_at timestamp in NewsletterRecipient.
2. Admin dashboard shows:
   - Total recipients
   - Read count and percentage
   - List of parents who read/didn't read
   - Read rate over time (graph)

### Scheduled Publishing

1. Admin creates newsletter and selects "Schedule for later".
2. Admin selects scheduled_at date/time.
3. System stores newsletter with status=Scheduled.
4. Cron job runs every minute and checks for newsletters where scheduled_at <= now AND status=Scheduled.
5. For matching newsletters: execute delivery flow (see above).


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
