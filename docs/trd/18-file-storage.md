# 18. File & Media Storage

**Related TRDs**: [07-registration](./07-registration.md), [13-newsletter](./13-newsletter.md), [14-parent-feed](./14-parent-feed.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1)

---

### Cloud Storage

AWS S3 (or GCP Cloud Storage):
- Bucket: `ararat-media`
- Path structure: `s3://ararat-media/tenants/{tenantId}/{type}/{entityId}/{fileId}`

### Photo Types & Privacy

| Photo Type | Path | Accessible By | Retention |
|-----------|------|---------------|-----------|
| Profile Photos | `members/{memberId}/profile/` | Staff, parent who uploaded | While member active |
| Class Activity Photos | `activity/{postId}/` | Parents of children in class | 1 year |
| Certificates | `certificates/{resultId}/` | Member, parents | Indefinitely |
| Newsletter Attachments | `newsletters/{newsletterId}/` | Recipients | 1 year |

### CDN

CloudFront (or equivalent) for serving photos:
- Origin: S3 bucket
- Distribution: Global
- Cache TTL: 1 day for photos, 1 hour for certificates
- Signed URLs for private content (class photos, certificates)

### Upload Flow

1. Client requests presigned URL: `POST /api/v1/tenants/{tenantId}/files/presigned-url`
2. Backend generates presigned URL valid for 15 minutes.
3. Client uploads directly to S3 using presigned URL.
4. Client notifies backend: `POST /api/v1/tenants/{tenantId}/files/confirm-upload`
5. Backend records file metadata in database.

### Size Limits

- Profile photos: 10 MB
- Class photos: 10 MB
- Newsletter attachments: 50 MB
- Certificates: 10 MB

### Retention

- Profile photos: Deleted when member withdraws or consent revoked.
- Class photos: Deleted after 1 year.
- Certificates: Retained indefinitely.
- Newsletter attachments: Deleted after 1 year.

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
