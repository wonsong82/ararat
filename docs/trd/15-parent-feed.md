# 15. Feature: Parent Activity Feed (오늘 수련 일지)

**Related TRDs**: [10-attendance](./10-attendance.md), [13-notifications](./13-notifications.md), [19-file-storage](./19-file-storage.md)  
**Related ADRs**: _None_  
**Phase**: Phase 2

---

## 15.1 Overview

Brightwheel-style daily activity feed where instructors post training notes, photos, and observations about children's classes. Parents see a chronological feed of their children's activity. The system auto-generates weekly progress summaries and milestone celebrations (belt promotions, attendance streaks, birthdays).

The feed serves as the primary communication channel between instructors and parents — replacing paper newsletters and ad-hoc messaging with a structured, media-rich activity log.

---

## 15.2 Business Rules

### 15.2.1 Feed Data Model

#### ActivityFeedPost

| Field | Type | Description |
|-------|------|-------------|
| `post_id` | UUID | Primary key |
| `tenant_id` | UUID | FK → Tenant |
| `class_id` | UUID | FK → Class (which class this post is about) |
| `posted_by` | UUID | FK → User (instructor or system) |
| `type` | ENUM | `TrainingNotes`, `Photo`, `Observation`, `Milestone` |
| `content` | TEXT | Post body text |
| `visibility` | ENUM | `Class` (all attendees' parents), `Individual` (one child's parents) |
| `visible_to_member_id` | UUID | FK → Member (only when visibility = `Individual`) |
| `created_at` | TIMESTAMP | Post creation time |
| `updated_at` | TIMESTAMP | Last edit time |

#### ActivityFeedPhoto

| Field | Type | Description |
|-------|------|-------------|
| `photo_id` | UUID | Primary key |
| `post_id` | UUID | FK → ActivityFeedPost |
| `photo_url` | VARCHAR | S3 URL for full-size photo |
| `thumbnail_url` | VARCHAR | S3 URL for thumbnail |
| `sort_order` | INT | Display order within the post |
| `created_at` | TIMESTAMP | Upload time |

#### ActivityFeedReaction

| Field | Type | Description |
|-------|------|-------------|
| `reaction_id` | UUID | Primary key |
| `post_id` | UUID | FK → ActivityFeedPost |
| `parent_id` | UUID | FK → User (reacting parent) |
| `type` | ENUM | `Like`, `Comment` |
| `content` | TEXT | Comment text (null for Likes) |
| `created_at` | TIMESTAMP | Reaction time |

### 15.2.2 Daily Training Notes

1. Instructor completes a class session
2. Instructor creates a post with `type=TrainingNotes`, `visibility=Class`
3. System identifies all members who attended that class (via attendance records from TRD 10)
4. System sends push notification to each attendee's parent(s): "New training notes for {childName}'s {className} class"
5. Parents see the post in their feed

### 15.2.3 Photo/Video Sharing

- Photos are uploaded to S3 at path: `s3://ararat-photos/tenants/{tenantId}/activity/{postId}/{photoId}.jpg`
- Thumbnails are generated server-side (max 400×400px) and stored alongside originals
- **Privacy rule**: Only parents of children who attended the class can see class-visibility photos. The system cross-references the post's `class_id` + class date against attendance records
- Multiple photos per post (up to 10)
- Accepted formats: JPEG, PNG, HEIC (converted to JPEG on upload)
- Max file size: 10MB per photo

### 15.2.4 Instructor Observations

- Per-child private notes: `type=Observation`, `visibility=Individual`, `visible_to_member_id` set to the specific child
- Only that child's linked parent(s) can see the observation
- Notification sent to parent(s): "Instructor left a note about {childName}"
- Use cases: behavior notes, skill progress, areas for improvement, positive reinforcement

### 15.2.5 Weekly Progress Summary

- **Cron job**: Runs every Sunday at 6:00 PM (gym's local timezone)
- For each active member, the system generates a summary post:
  1. Count classes attended that week
  2. Fetch any training notes and observations from the week
  3. Check for belt progress (from TRD 10 belt/rank data)
  4. Generate a structured summary text
- Post created with `type=Milestone`, `visibility=Individual`, `visible_to_member_id` = the member
- Summary format:
  ```
  📊 Weekly Summary for {childName}
  Classes attended: {count}/{total}
  {trainingHighlights}
  {beltProgress if any}
  Keep up the great work! 🥋
  ```

### 15.2.6 Parent Reactions

- Parents can react to any post visible to them
- **Like**: Toggle on/off, one per parent per post
- **Comment**: Free-text, multiple per parent per post
- When a parent comments:
  - Notify the instructor who created the post
  - Notify other parents who previously commented on the same post (no duplicates)
- Comments are visible only to the post author (instructor) and parents who can see the post

### 15.2.7 Milestone Celebrations

Auto-generated posts (`type=Milestone`, `visibility=Individual`) for:

| Milestone | Trigger | Message Template |
|-----------|---------|-----------------|
| Belt Promotion | Member's belt rank changes | "🥋 Congratulations! {childName} earned their {beltColor} belt!" |
| Attendance Streak 10 | 10 consecutive classes attended | "🔥 {childName} hit a 10-class attendance streak!" |
| Attendance Streak 50 | 50 consecutive classes attended | "⭐ Amazing! {childName} reached 50 classes in a row!" |
| Attendance Streak 100 | 100 consecutive classes attended | "🏆 Incredible! {childName} achieved 100 consecutive classes!" |
| Birthday | Member's birthday (runs daily at 8 AM) | "🎂 Happy Birthday, {childName}! 🎉" |

---

## 15.3 Backend

### 15.3.1 API Endpoints

#### Feed Posts

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/feed` | List feed posts (parent sees their children's posts) | Parent, Instructor, Admin |
| `POST` | `/api/v1/tenants/{tenantId}/feed` | Create a new post | Instructor, Admin |
| `GET` | `/api/v1/tenants/{tenantId}/feed/{id}` | Get post detail with photos and reactions | Parent, Instructor, Admin |
| `PATCH` | `/api/v1/tenants/{tenantId}/feed/{id}` | Update post content | Post author, Admin |
| `DELETE` | `/api/v1/tenants/{tenantId}/feed/{id}` | Delete post | Post author, Admin |

#### Photos

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/api/v1/tenants/{tenantId}/feed/{id}/photos` | Upload photos to a post (multipart) | Post author, Admin |

#### Reactions

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/api/v1/tenants/{tenantId}/feed/{id}/reactions` | Add a reaction (Like or Comment) | Parent |

#### Member-Specific Feed

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/members/{id}/feed` | Feed filtered to a specific member | Parent (own children), Instructor, Admin |

#### Query Parameters (GET endpoints)

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | INT | 1 | Page number |
| `limit` | INT | 20 | Posts per page (max 50) |
| `type` | STRING | — | Filter by post type |
| `from` | DATE | — | Posts from this date |
| `to` | DATE | — | Posts until this date |

### 15.3.2 Service Logic

#### Photo Upload Flow

1. Instructor uploads photos via multipart form data
2. Service validates file type (JPEG, PNG, HEIC) and size (≤ 10MB)
3. HEIC files are converted to JPEG
4. Full-size image stored at `s3://ararat-photos/tenants/{tenantId}/activity/{postId}/{photoId}.jpg`
5. Thumbnail generated (max 400×400px, maintain aspect ratio) and stored at `s3://ararat-photos/tenants/{tenantId}/activity/{postId}/{photoId}_thumb.jpg`
6. `ActivityFeedPhoto` record created with both URLs

#### Privacy Filtering

- When a parent requests their feed (`GET /feed`), the service:
  1. Identifies all children linked to the parent
  2. Fetches posts where `visibility=Class` AND the child attended the class on the post date (join with attendance records)
  3. Fetches posts where `visibility=Individual` AND `visible_to_member_id` matches one of the parent's children
  4. Merges and sorts by `created_at` descending
- Instructors and admins see all posts for their tenant

#### Weekly Summary Cron

- Scheduled: Sunday 6:00 PM per gym's configured timezone
- For each active member in the tenant:
  1. Query attendance records for the past 7 days
  2. Query `ActivityFeedPost` records (TrainingNotes, Observations) referencing classes the member attended
  3. Check member's belt/rank for any changes in the past 7 days
  4. Compose summary text
  5. Create `ActivityFeedPost` with `type=Milestone`, `visibility=Individual`
  6. Send push notification to parent(s)

#### Milestone Auto-Generation

- **Belt Promotion**: Triggered by belt rank change event (from attendance/promotion module). Creates milestone post immediately.
- **Attendance Streaks**: After each attendance check-in, query consecutive attendance count. If count hits 10/50/100, create milestone post.
- **Birthdays**: Daily cron at 8:00 AM, queries members with today's birthday, creates milestone post for each.

#### Reaction Notifications

- On Like: Notify post author (instructor)
- On Comment: Notify post author + all previous commenters (deduplicated, exclude the commenter themselves)
- Notification delivery via TRD 13 notification system

---

## 15.4 Parent/Member App

### Screen: Dashboard — Activity Feed (`/`)

The activity feed is the primary dashboard view for parents — a Brightwheel-style chronological feed of their children's daily activities.

#### Layout

- **Child Selector Tabs** (top): Horizontal tab bar showing each child's name. Tapping a tab filters the feed to that child. "All" tab shows combined feed.
- **Feed List** (scrollable): Reverse-chronological list of activity cards
- **Pull-to-Refresh**: Pull down to reload latest posts
- **Empty State**: "No activity yet today" with illustration

#### Feed Card Structure

Each feed entry is rendered as a card containing:

| Element | Description |
|---------|-------------|
| Instructor avatar | Profile photo or initials badge |
| Instructor name | Posted by line |
| Timestamp | Relative time ("2 hours ago") or date |
| Post type badge | Color-coded label: Training Notes (blue), Observation (green), Milestone (gold) |
| Text content | Post body with markdown-lite formatting |
| Photo gallery | Horizontal scrollable thumbnails; tap for fullscreen lightbox |
| Reaction bar | Heart/thumbs-up count + comment count |
| Comment section | Expandable; shows latest 2 comments with "View all" link |

#### Components

| Component | Purpose |
|-----------|---------|
| `ChildSelectorTabs` | Tab bar for switching between children |
| `FeedCard` | Individual post card with all elements above |
| `PhotoGallery` | Horizontal thumbnail strip with lightbox on tap |
| `PhotoLightbox` | Fullscreen photo viewer with swipe navigation |
| `ReactionBar` | Like button (toggle) + comment button + counts |
| `CommentThread` | Expandable comment list with input field |

#### Data Sources

| Hook | Description |
|------|-------------|
| `useChildren()` | Fetches parent's linked children for tab selector |
| `useFeedPosts(childId)` | Paginated feed posts for selected child; supports infinite scroll |
| `useReactions(postId)` | Reactions and comments for a specific post |

#### Actions

- Switch children via tabs → refetches feed for selected child
- Scroll feed → infinite scroll loads older posts
- Tap photo thumbnail → opens fullscreen lightbox with swipe
- Tap heart/thumbs-up → toggles like reaction (optimistic update)
- Tap comment → expands comment thread, shows input field
- Pull to refresh → reloads feed from server

---

## 15.5 Admin App

### Screen: Activity Feed Management (`/feed`)

Instructors and admins use this screen to create, manage, and review activity feed posts.

#### Layout

- **Post List** (main area): Reverse-chronological table/card list with columns:
  - Author (instructor name + avatar)
  - Class name
  - Date/time
  - Content preview (truncated to 100 chars)
  - Photo count (📷 icon + number)
  - Reaction count (❤️ icon + number)
  - Actions (Edit, Delete)

- **Create Post Button** (top right): Opens create post form

#### Create Post Form

| Field | Type | Description |
|-------|------|-------------|
| Class selector | Dropdown | Select the class this post is about (required) |
| Post type | Radio | TrainingNotes, Observation, Photo |
| Text area | Rich text | Post content body |
| Photo uploader | Drag-and-drop zone | Multiple file upload, up to 10 photos, preview thumbnails before submit |
| Member tag selector | Multi-select | For Observation type: select specific member(s). Sets `visibility=Individual` |
| Visibility | Auto-set | `Class` for TrainingNotes/Photo, `Individual` for Observation |

#### Photo Gallery Lightbox

- Clicking any photo thumbnail in the post list opens a fullscreen lightbox
- Swipe/arrow navigation between photos in the same post

#### Data Sources

| Hook | Description |
|------|-------------|
| `useFeedPosts(filters)` | Paginated feed posts with optional filters (date range, type, class, author) |
| `useClasses()` | Available classes for the class selector dropdown |
| `useMembers()` | Members list for the member tag selector |

#### Actions

| Action | Who | Description |
|--------|-----|-------------|
| Create post | Instructor, Admin | Select class → write note → upload photos → tag members → submit |
| Edit post | Post author, Admin | Modify text content or photos of an existing post |
| Delete post | Post author (own), Admin (any) | Soft-delete with confirmation dialog |
| View reactions | Instructor, Admin | See likes and read comments on any post |

#### Permissions

- Instructors can create, edit, and delete their own posts
- Admins can edit and delete any post
- Both can view all reactions and comments

---

## 15.6 Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
