# 20. Performance Requirements

**Related TRDs**: All sections  
**Related ADRs**: _None_  
**Phase**: All Phases

---

| Metric | Target | Notes |
|--------|--------|-------|
| **Face Recognition** | < 2 seconds for 1:N matching against 500 enrolled members | On-device, no cloud latency |
| **Notification Delivery** | Push/SMS within 30 seconds of trigger event | Via FCM, Twilio |
| **API Response Time** | p95 < 200ms for read, p95 < 500ms for write | Measured at API Gateway |
| **System Uptime** | 99.9% | 43 minutes downtime/month acceptable |
| **Kiosk Offline Tolerance** | Up to 24 hours of offline operation with full sync on reconnect | Critical for gym reliability |
| **Concurrent Users** | 100 simultaneous admin users, 1000 parent users, 50 kiosks per deployment | Scalable via auto-scaling |
| **Database Scale** | 500 gyms with avg 200 members each (100K total members) | Managed via sharding if needed |
| **Photo Upload** | < 5 seconds for 10 MB file | Direct S3 upload via presigned URL |
| **Report Generation** | < 30 seconds for monthly report on 500 members | Pre-computed daily rollups |

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
