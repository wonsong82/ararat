# 14. Feature: Parent Activity Feed (오늘 수련 일지)

**Related TRDs**: [08-attendance](./08-attendance.md), [12-notifications](./12-notifications.md), [18-file-storage](./18-file-storage.md)  
**Related ADRs**: _None_  
**Phase**: Phase 2

---

### Feed Data Model

```
ActivityFeedPost:
  - post_id
  - tenant_id
  - class_id (nullable)
  - posted_by (staff_id)
  - type (TrainingNotes, Photo, Observation, Milestone)
  - content (text)
  - visibility (Class, Individual)
  - visible_to_member_id (nullable, if visibility=Individual)
  - created_at
  - updated_at

ActivityFeedPhoto:
  - photo_id
  - post_id
  - photo_url (S3 URL)
  - thumbnail_url (S3 URL)
  - created_at

ActivityFeedReaction:
  - reaction_id
  - post_id
  - parent_id
  - type (Like, Comment)
  - content (nullable, for comments)
  - created_at
```

### Daily Training Notes

Instructor posts what the class practiced today:

1. Instructor visits admin app after class.
2. Instructor clicks "Post Training Notes".
3. Instructor selects class.
4. Instructor enters notes (e.g., "Today we practiced roundhouse kicks and sparring drills").
5. Instructor clicks "Post".
6. System creates ActivityFeedPost with type=TrainingNotes, visibility=Class.
7. System finds all parents of children in that class.
8. For each parent: send notification with training notes summary.
9. Parents see post in their activity feed.

### Photo/Video Sharing

Instructor uploads photos during or after class:

1. Instructor clicks "Upload Photos".
2. Instructor selects photos from device.
3. System uploads to S3: `s3://ararat-photos/tenants/{tenantId}/activity/{postId}/{photoId}.jpg`.
4. System generates thumbnail.
5. System creates ActivityFeedPost with type=Photo.
6. System creates ActivityFeedPhoto records for each photo.
7. **Privacy**: Photos visible only to parents of children who attended that class.
8. System finds parents of attendees and sends notification.

### Instructor Observations

Per-child notes from instructor:

1. Instructor clicks "Add Observation".
2. Instructor selects child.
3. Instructor enters observation (e.g., "Great focus today! Needs work on roundhouse kick form").
4. System creates ActivityFeedPost with type=Observation, visibility=Individual, visible_to_member_id=[child_id].
5. **Privacy**: Visible only to that child's parents.
6. Send notification to parents.

### Weekly Progress Summary

Auto-generated every [configurable day, default Sunday]:

1. Cron job runs on Sunday at 6 PM (gym's timezone).
2. For each active member:
   - Count classes attended this week.
   - Fetch training notes from those classes.
   - Fetch instructor observations for this member.
   - Fetch belt progress (if any).
   - Generate summary: "This week, [member_name] attended [N] classes. Practiced: [skills]. Instructor notes: [observations]. Belt progress: [progress]."
3. Create ActivityFeedPost with type=Milestone (if applicable).
4. Send notification to parents with summary.

### Parent Reactions

Parents can like and comment on feed items:

1. Parent sees post in feed.
2. Parent clicks "Like" button.
3. System creates ActivityFeedReaction with type=Like.
4. Instructor receives notification: "[parent_name] liked your post".
5. Parent clicks "Comment".
6. Parent enters comment text.
7. System creates ActivityFeedReaction with type=Comment.
8. Instructor and other parents who commented receive notification.

### Milestone Celebrations

System auto-generates celebratory posts for:

1. **Belt Promotion**: "Congratulations! [member_name] achieved [new_belt]! 🎉"
2. **Attendance Streaks**: "Milestone! [member_name] has attended [10/50/100] classes! 🌟"
3. **Birthdays**: "Happy Birthday, [member_name]! 🎂"

Posts created automatically, visible to all parents in the gym (or just that child's parents, configurable).


### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
