# Product Requirements Document: Ararat

## Taekwondo Gym Management Platform

**Version**: 1.0
**Last Updated**: February 25, 2026
**Status**: Draft

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Target Users & Personas](#3-target-users--personas)
4. [Product Vision & Principles](#4-product-vision--principles)
5. [Platform Architecture](#5-platform-architecture)
6. [Feature Requirements](#6-feature-requirements)
7. [Competitive Analysis](#7-competitive-analysis)
8. [Technical Considerations](#8-technical-considerations)
9. [Privacy, Security & Compliance](#9-privacy-security--compliance)
10. [Release Phases & Roadmap](#10-release-phases--roadmap)
11. [Success Metrics](#11-success-metrics)
12. [Appendix](#12-appendix)

---

## 1. Executive Summary

Ararat is a **multi-tenant SaaS platform** purpose-built for Taekwondo gym owners in the **United States** to manage their members, operations, and parent communications. The US TKD gym market is largely run by Korean-American owners (관장님), serving a diverse member base with a high proportion of Korean-American families. The platform supports **multilingual operation (English, Korean, Spanish)** out of the box, and is designed to be **universal** — any Taekwondo gym can onboard independently and configure the system to match their operations.

The platform consists of **four applications**:

| Application | Platform | Primary User |
|---|---|---|
| **Parent/Member App** | Web (mobile-responsive) | Parents, adult members |
| **Admin App** | Web | Gym owner, staff |
| **Attendance Kiosk App** | iPad/Tablet | Children (self check-in) |
| **Monitor App** | TV/Large Display | Staff, visitors (optional) |

**Core differentiators** vs. existing solutions (Zen Planner, Kicksite, Gymdesk, etc.):

- **Automated attendance via face recognition kiosk** — children check in by simply walking up to the kiosk. No cards, no QR codes needed (with fallbacks).
- **Real-time parent alignment (Brightwheel model for TKD)** — every gym activity (attendance, belt progress, class photos, 심사 results, daily training notes) is instantly pushed to parents. Modeled after how Brightwheel/HiMama transformed parent communication in childcare.
- **Multilingual, Korean-heritage design** — built for how Korean-American 관장님 actually run their 도장, with full multilingual support (English/Korean/Spanish), 심사 (belt promotion exam) workflows, 가정통신문 (home-school newsletters), and pluggable third-party messenger integration (starting with KakaoTalk, extensible to WhatsApp, etc.).
- **Childcare-grade safety features** — drop-off verification, unconfirmed-arrival alerts, and full audit trails for child safety.
- **Kukkiwon TCON integration** — connection to the World Taekwondo headquarters' official 승품·단 (belt promotion) exam registration system, eliminating manual re-entry of student data.

---

## 2. Problem Statement

### For Gym Owners (관장님)

Korean-American Taekwondo gym owners in the US juggle member registration, attendance tracking, payment collection, belt promotions, and parent communications — mostly through manual processes, group texts, KakaoTalk messages, and spreadsheets. They serve a diverse community (Korean, Hispanic, and other families) but existing gym management software (Zen Planner, Mindbody, etc.) is:

- **English-only** — no multilingual support. Korean-speaking parents and Spanish-speaking parents struggle with English-only interfaces. Korean-heritage gym workflows (심사, 가정통신문) have no software support.
- **Adult-focused** — not designed around the parent-child dynamic central to TKD gyms where 80%+ of members are minors.
- **Attendance is manual** — staff must physically check in each child, creating bottlenecks during busy drop-off times.
- **Parent communication is fragmented** — no unified channel for progress updates, resulting in parent churn. Gym owners resort to personal messaging apps (KakaoTalk, WhatsApp, group texts) per parent, creating operational chaos.

### For Parents

Parents drop off their children and have **zero visibility** into what happens at the gym. They don’t know:
- Did my child actually check in at the gym?
- What did they practice today?
- How close are they to their next belt?
- Is their membership about to expire?
- When is the next 심사 and is my child eligible?

This lack of transparency erodes trust and contributes to member attrition.

---

## 3. Target Users & Personas

### Primary Personas

| Persona | Description | Key Needs |
|---|---|---|
| **관장님 (Gym Owner)** | Korean-American owner operating 1-3 TKD gyms in the US. Manages 50-300 students but not limited to. Handles business operations end-to-end. Often bilingual (Korean/English). | Reduce admin overhead, retain members, collect payments on time, communicate with diverse parent base efficiently. |
| **Parent (English-speaking)** | Has 1-3 children enrolled. Drops off/picks up daily. Primary decision-maker for enrollment. Expects clear English communication. | Know my child is safe, see progress, manage payments, receive timely updates in English. |
| **학부모 (Korean-speaking Parent)** | Korean-American parent with 1-3 children enrolled. Prefers Korean-language communication. May prefer third-party messengers (KakaoTalk, WhatsApp). | Same needs as English-speaking parent, but in Korean. Comfortable with Korean-heritage gym culture (심사, 가정통신문). |
| **Padre/Madre (Spanish-speaking Parent)** | Hispanic parent with children enrolled. Prefers Spanish-language communication. | Same needs, but in Spanish. Clear payment and schedule information. |
| **수련생 (Student/Child)** | Ages 5-15. Diverse backgrounds. Attends 3-5 times/week. Interacts with the kiosk at check-in. | Quick, fun check-in experience. |
| **사범 (Instructor/Staff)** | Teaches classes, manages attendance, assists with belt testing. May be bilingual. | View class roster, mark attendance, communicate updates in parent's preferred language. |

### Secondary Personas

| Persona | Description |
|---|---|
| **Adult Member** | Self-registers, manages own profile and payments. |
| **Multi-Gym Admin** | Franchise or regional operator managing multiple locations. |

---

## 4. Product Vision & Principles

### Vision

> Make every Taekwondo gym owner feel like they have a full-time operations manager — and make every parent feel like they have a window into their child's training.

### Design Principles

1. **Parent-First Transparency** — If it happens at the gym, parents know about it. Attendance, progress, schedule changes, and announcements flow to parents in real-time.
2. **Zero-Friction Attendance** — A child walks in, the kiosk recognizes them, and the parent gets a notification. No cards, no codes, no staff intervention needed.
3. **Universal by Default** — Every feature must work for any gym without custom development. Configuration over customization.
4. **Multilingual by Design** — English as default, Korean and Spanish as first-class languages. Every notification, newsletter, and UI element supports the parent’s preferred language. Built for Korean-heritage gym workflows (심사, 띄) while being accessible to all families.
5. **Child Safety is Non-Negotiable** — Every attendance record has timestamps, audit trails, and dual-verification options. Biometric data is processed on-device and never stored in the cloud.

---

## 5. Platform Architecture

### 5.1 Parent/Member App (Web — Mobile-Responsive)

The primary interface for parents and adult members. Accessible via mobile browser (PWA candidate for future native-like experience).

**Core Capabilities:**

- **Registration** — Self-registration or parent registration (with child profiles)
- **Profile Management** — Manage personal info, add/edit children, health info (allergies, etc.)
- **Activity Feed (오늘 수련 일지)** — Brightwheel-style real-time feed showing what the child did in class today, with instructor notes and photos. This is the **primary parent engagement surface**.
- **Membership** — View status, renewal dates, payment history
- **Attendance History** — View child’s check-in/check-out records with timestamps
- **Progress Tracking** — Belt level, skill assessments, 심사 results, attendance streaks
- **Payments** — View invoices, make payments, manage auto-pay (Stripe — US credit card/ACH)
- **Notifications** — Real-time push/SMS/email for attendance, announcements, payment reminders. Optional third-party messenger integration (KakaoTalk, WhatsApp, etc.) based on parent preference.
- **심사 (Belt Test)** — Submit required forms, view upcoming test dates, receive results
- **Newsletter** — Read formal newsletters from the gym, with read receipts (multilingual)
- **Direct Messaging** — Two-way messaging with the gym owner/instructors in preferred language
- **Withdrawal** — Self-service with refund calculation per gym policy

### 5.2 Admin App (Web)

The gym owner and staff command center.

**Core Capabilities:**

- **Registration Management** — Review pending applications, approve/reject, manual registration
- **Member Management** — Full member directory with search/filter (age, belt, group, status)
- **Class & Schedule Management** — Create/edit classes, assign instructors, manage capacity
- **Attendance Dashboard** — Real-time attendance view, historical reports, absence alerts
- **Payment & Billing** — Invoice management, recurring billing, refunds, discount management
- **Belt & Level Management** — Track belt progression, manage 심사 scheduling, batch promotions
- **Activity Feed Management** — Instructors post class photos, daily training notes, skill observations per class
- **가정통신문 (Newsletter) Editor** — Create and send formal newsletters with templates, track read receipts
- **Parent Messaging** — 1:1 and broadcast messaging to parents (third-party messenger, SMS, in-app)
- **Notifications & Alerts** — Configure alert rules, send announcements, view notification history
- **Reporting** — Financial reports, attendance trends, retention analytics, parent engagement metrics, exportable (PDF/Excel)
- **Settings** — Gym profile, staff accounts, payment policies, notification templates, alert thresholds

### 5.3 Attendance Kiosk App (iPad/Tablet)

A dedicated app running on an iPad (with stand) placed at the gym entrance for automated child check-in.

**Core Capabilities:**

- **Face Recognition Check-In (Primary)** — Camera detects and identifies enrolled members using on-device face recognition. Check-in is confirmed with a visual greeting (photo + name display).
- **QR Code Scan (Secondary)** — Members can hold up their membership card QR code to the camera.
- **Name Search (Fallback)** — Manual search by name for cases where face/QR fails.
- **Attendee Display** — Shows a scrolling list of today's checked-in members with photos.
- **Staff Validation Mode** — Optional dual-check where staff confirms the kiosk check-in.
- **Offline Mode** — Records attendance locally and syncs when connectivity is restored.

**Check-In Flow:**

```
Child approaches kiosk
        │
        ▼
┌─────────────────┐
│ Face Recognition │──── Match found ──── ✅ Check-in confirmed
│   (< 2 sec)     │                            │
└────────┬────────┘                            ▼
         │ No match                    Display greeting
         ▼                            (name + photo)
┌─────────────────┐                            │
│  QR Code Scan   │──── Match found ────────────┤
└────────┬────────┘                            │
         │ No QR                               ▼
         ▼                           Send notification
┌─────────────────┐                   to parent (SMS/Push)
│  Name Search    │──── Found ──────────────────┤
│  (manual)       │                            │
└────────┬────────┘                            ▼
         │ Not found               Log: timestamp, method,
         ▼                         staff ID, location
┌─────────────────┐
│ Staff Assistance │
│  (alert staff)   │
└─────────────────┘
```

### 5.4 Monitor App (TV Display — Optional)

A read-only display for the gym lobby or training area.

**Core Capabilities:**

- **Today's Schedule** — Classes, times, instructors
- **Attendance Board** — Live check-in feed showing who has arrived
- **Announcements** — Gym news, upcoming 심사 dates, events
- **Leaderboard / Recognition** — Belt promotions, attendance streaks, birthdays

---

## 6. Feature Requirements

### 6.1 Member Registration & Profile Management

#### 6.1.1 Registration Flow

| Requirement | Description | Priority |
|---|---|---|
| **Parent Registration** | Parent creates account → selects preferred language (English/Korean/Spanish) → verifies phone (SMS OTP) → accepts Terms & Conditions → adds child profile(s). | P0 |
| **Self Registration (Adult)** | Adult member registers directly for themselves. | P0 |
| **Child Profile Linking** | Parent account can link multiple children. Each child has: name, date of birth, photo, health info (allergies, medical conditions), emergency contact. | P0 |
| **Admin Approval Workflow** | New registrations enter "Pending" state. Admin reviews and approves/rejects via dashboard. Option to auto-approve based on configurable rules. | P0 |
| **Manual Registration** | Admin can register members directly (walk-in registrations). | P0 |
| **Duplicate Detection** | System flags potential duplicate registrations (matching name + DOB or phone). | P1 |
| **Data Minimization (COPPA)** | For children under 13: collect only essential data, require verified parental consent before storing any personal information. All consent forms available in English, Korean, and Spanish. | P0 |

#### 6.1.2 Profile Management

| Requirement | Description | Priority |
|---|---|---|
| **Profile Editing** | Members/parents can update profile info. All changes are logged with timestamp and previous values. | P0 |
| **Admin Notifications** | Admin receives notification when member updates profile (configurable). | P1 |
| **Photo Management** | Profile photos used for kiosk face recognition. Photo upload during registration, updatable anytime. Multiple photos per member for recognition accuracy. | P0 |
| **Health Information** | Allergy, medical condition, and emergency contact fields. Visible to staff during class. | P0 |
| **Annual Re-Registration** | System sends re-registration reminder annually. Configurable timing (e.g., 30 days before anniversary). | P1 |

#### 6.1.3 Member Levels & Grouping

| Requirement | Description | Priority |
|---|---|---|
| **Belt Level Tracking** | Track current belt (white → black, with all intermediary levels). Each gym configures its own belt progression system. | P0 |
| **Age Group Assignment** | Automatic group assignment based on date of birth. Groups recalculate on birthday (e.g., move from "어린이" to "청소년"). | P0 |
| **Class Group Assignment** | Assign members to specific class groups (beginner, advanced, competition, private lesson). | P0 |
| **Auto-Upgrade** | When age threshold is crossed, system prompts admin to confirm group transfer. Belt promotions update automatically after 심사 results are entered. | P1 |

#### 6.1.4 Withdrawal (탈퇴)

| Requirement | Description | Priority |
|---|---|---|
| **Self-Service Withdrawal** | Member/parent can initiate withdrawal through app. | P0 |
| **Refund Calculation** | System calculates prorated refund based on gym's configurable refund policy. Accounts for discounts that were applied. | P0 |
| **Admin Approval** | Withdrawal request goes to admin for review before processing. | P0 |
| **Data Handling** | Upon withdrawal confirmation: archive member data per retention policy, delete personal data per legal requirements (COPPA, PIPA). | P0 |
| **Exit Survey** | Optional survey to capture withdrawal reason (for retention analytics). | P2 |

---

### 6.2 Attendance Management

#### 6.2.1 Kiosk Check-In (Primary Method)

| Requirement | Description | Priority |
|---|---|---|
| **Face Recognition** | On-device face recognition using device camera. Recognition must complete in < 2 seconds. All biometric processing happens on-device — no face data transmitted to cloud. | P0 |
| **QR Code Check-In** | Kiosk camera reads QR code from membership card or parent's phone. | P0 |
| **Name Search** | Manual search by name with photo confirmation on screen. | P0 |
| **Check-In Confirmation** | On successful check-in: display member photo + name + greeting on kiosk, play confirmation sound, add to today's attendee list. | P0 |
| **Parent Notification** | Immediately upon check-in: send push notification and/or SMS to parent with timestamp. | P0 |
| **Offline Mode** | Kiosk stores check-ins locally when offline. Auto-syncs when connectivity resumes. Parent notification sent upon sync. | P1 |
| **Attendee Display** | Scrolling display of all checked-in members with photos (for the current class/day). | P1 |

#### 6.2.2 Staff Validation (Dual-Check)

| Requirement | Description | Priority |
|---|---|---|
| **Optional Dual Verification** | Gym can enable "staff validation" mode where kiosk check-in requires a secondary staff confirmation via staff app. | P1 |
| **Unconfirmed Alert** | If a child checks in but staff hasn't validated within 5 minutes, system triggers alert on staff dashboard + parent notification. | P0 |
| **Staff ID Logging** | Every validation records which staff member confirmed, with timestamp. | P0 |

#### 6.2.3 Parent Drop-Off Flow

| Requirement | Description | Priority |
|---|---|---|
| **Pre-Arrival Reservation** | Parent can reserve drop-off time slot via app. | P2 |
| **Drop-Off Confirmation** | After child checks in at kiosk, parent receives confirmation notification. | P0 |
| **Full Audit Trail** | Every drop-off event logs: timestamp, check-in method, staff validator (if applicable). | P0 |

#### 6.2.4 Absence & Retention Alerts

| Requirement | Description | Priority |
|---|---|---|
| **Configurable Absence Rules** | Admin sets thresholds: e.g., 3 days / 7 days / 14 days of non-attendance. | P0 |
| **Automated Absence Alerts** | When member crosses a threshold, system sends encouragement email/SMS to member/parent. | P0 |
| **Admin Absence Dashboard** | Admin sees at-risk members ranked by days absent. | P0 |
| **Retention Attention** | Admin gets notification when a member approaches a configurable absence threshold, prompting personal outreach. | P1 |

#### 6.2.5 Advanced Attendance Features (Future)

| Requirement | Description | Priority |
|---|---|---|
| **AI Attendance Prediction** | ML model predicts members at risk of dropping off based on attendance patterns. | P3 |
| **Group Check-In** | Batch check-in for an entire class (staff-initiated). | P2 |
| **Real-Time Attendance Dashboard** | Admin dashboard with live attendance rate, daily/weekly/monthly trends, class fill rates. | P1 |
| **NFC Tag Check-In** | Alternative to face/QR — child taps NFC badge on kiosk. | P2 |

---

### 6.3 Payment & Financial Management

#### 6.3.1 Membership Billing

| Requirement | Description | Priority |
|---|---|---|
| **Recurring Billing** | Support monthly and annual subscription models with automatic charge on renewal date. | P0 |
| **Payment Gateway** | Integration with Stripe for credit card and ACH processing. | P0 |
| **Auto-Renewal** | Memberships auto-renew unless cancelled. Send renewal reminder 7 days before charge. | P0 |
| **Membership Hold/Suspend** | Admin or member can temporarily hold membership (e.g., vacation, illness). Billing paused during hold. | P1 |
| **Grace Period** | Configurable grace period (default ~10 days) after payment failure before membership suspension. | P0 |
| **Family Billing** | Single invoice for families with multiple enrolled children. Family discount configuration. | P1 |

#### 6.3.2 One-Time Payments

| Requirement | Description | Priority |
|---|---|---|
| **심사 (Belt Test) Fees** | Separate charge for belt promotion exams. Auto-calculated based on belt level (configurable per gym). | P0 |
| **Equipment/Retail Sales** | Accept payment for uniforms, belts, sparring gear, etc. | P2 |
| **Event Fees** | Charge for special events, workshops, tournaments. | P2 |

#### 6.3.3 Discounts & Refunds

| Requirement | Description | Priority |
|---|---|---|
| **One-Time Discount** | Admin can apply a one-time discount to any invoice (fixed amount or percentage). | P0 |
| **Sibling/Family Discount** | Configurable automatic discount for multiple-child enrollment. | P1 |
| **Prorated Refund** | On withdrawal: system calculates prorated refund based on remaining days and gym's refund policy. Handles cases where a discount was applied. | P0 |
| **Manual Refund Override** | Admin can override calculated refund with manual amount (logged with reason). | P0 |

#### 6.3.4 Late Payment Management

| Requirement | Description | Priority |
|---|---|---|
| **Auto-Detection** | System detects failed/missed payments immediately. | P0 |
| **Escalating Reminders** | Configurable: Day 1 → email reminder. Day 5 → SMS reminder. Day 10 → admin alert + membership warning. | P0 |
| **Blacklist Management** | Admin can flag accounts with chronic non-payment. Configurable restrictions (e.g., cannot attend until balance cleared). | P2 |
| **Receipt & Invoice** | Auto-generated receipts emailed after every successful payment. | P0 |

#### 6.3.5 Financial Reporting

| Requirement | Description | Priority |
|---|---|---|
| **Monthly Revenue Report** | Total revenue, breakdown by category (membership, 심사, retail). Export to Excel. | P0 |
| **Outstanding Balance Report** | List of all members with unpaid balances. | P0 |
| **Tax Calculation** | Configurable tax rates by US state. Auto-applied to invoices. | P2 |
| **Payout Report** | Track Stripe payouts to gym bank account. | P1 |

---

### 6.4 Notification & Communication System

#### 6.4.1 Notification Channels

| Channel | Use Cases | Integration | Priority |
|---|---|---|---|
| **Push Notification** | Attendance confirmation, announcements, reminders | Firebase Cloud Messaging (FCM) / APNs | P0 |
| **SMS** | Attendance alerts, payment reminders, urgent notices | Twilio | P0 |
| **Email** | Receipts, reports, newsletters, 심사 follow-ups | SendGrid or AWS SES | P0 |
| **In-App Notification** | All events, searchable notification history | Internal | P0 |
| **Third-Party Messenger** | Pluggable messenger integration for parents who prefer messaging apps over SMS/email. Architecture supports multiple providers. | KakaoTalk Business API (initial), WhatsApp Business API, etc. | P2 |

#### 6.4.2 Alert Rules Engine

| Requirement | Description | Priority |
|---|---|---|
| **Configurable Rules** | Admin defines alert rules per trigger type (absence, late payment, membership expiry, 심사 deadline). | P0 |
| **Threshold Configuration** | Each rule has configurable thresholds (e.g., days absent: 3, 7, 14). | P0 |
| **Template Editor** | Admin can customize email/SMS templates per language (English, Korean, Spanish). Supports personalization tokens (member name, days absent, balance due, etc.). | P1 |
| **Attachment Support** | Emails can include attachments (attendance reports, invoices). | P2 |
| **Escalation Rules** | If member doesn't respond to 2 consecutive alerts, system auto-creates a task for admin follow-up. | P1 |
| **Send Log & Analytics** | Full log of all sent notifications with delivery status. Open rate / click rate tracking for email campaigns. | P2 |

#### 6.4.3 Parent Alignment Notifications

These notifications are the **core differentiator** — keeping parents informed about everything that happens at the gym.

| Notification | Trigger | Channel | Priority |
|---|---|---|---|
| **Arrival Confirmation** | Child checks in at kiosk | Push + SMS | P0 |
| **Class Summary** | End of class | Push | P1 |
| **Activity Photo** | Instructor uploads class photo | Push + In-App | P1 |
| **Belt Promotion** | 심사 result entered by admin | Push + Email | P0 |
| **Upcoming 심사** | 심사 scheduled, X days before | Push + Email | P0 |
| **Attendance Streak** | Child hits attendance milestone (10, 50, 100 classes) | Push | P2 |
| **Payment Receipt** | Successful payment | Email | P0 |
| **Payment Due** | Upcoming / overdue payment | Push + SMS + Email | P0 |
| **Membership Expiry** | X days before membership expires | Push + Email | P0 |
| **Schedule Change** | Class time/instructor changed | Push + SMS | P0 |
| **Gym Announcement** | Admin publishes announcement | Push + In-App | P0 |

---

### 6.5 심사 (Belt Promotion Exam) Management

This is a **TKD-specific workflow** that no existing competitor handles well.

| Requirement | Description | Priority |
|---|---|---|
| **심사 Scheduling** | Admin creates a 심사 event with date, time, eligible belt levels, and fee. | P0 |
| **Eligibility Check** | System identifies eligible members based on current belt level, attendance count since last promotion, and time-in-rank. | P1 |
| **Parent Notification** | Parents of eligible children are automatically notified with exam details and fee. | P0 |
| **Form Submission** | Parent submits required forms/consent through the app. | P0 |
| **Fee Payment** | 심사 fee charged through the platform. | P0 |
| **Document Follow-Up** | If required forms not submitted within 3 days of notification, auto-send reminder. | P1 |
| **Result Entry** | Admin enters pass/fail results per member after 심사. | P0 |
| **Belt Update** | On pass: member's belt level auto-updates in the system. | P0 |
| **Certificate Generation** | Generate promotion certificate (PDF) for download/print. | P2 |
| **Result Notification** | Parents receive 심사 results with congratulations/encouragement message. | P0 |

---

### 6.6 Reporting & Analytics

#### 6.6.1 Standard Reports

| Report | Description | Format | Priority |
|---|---|---|---|
| **Attendance Report** | Daily/weekly/monthly attendance rates by class, age group, belt level. | PDF, Excel | P0 |
| **Financial Report** | Revenue breakdown, outstanding balances, refunds, discounts applied. | PDF, Excel | P0 |
| **Member Roster** | Current member list with status, belt level, group, join date. Filterable. | PDF, Excel | P0 |
| **Retention Report** | New enrollments, withdrawals, net growth, churn rate by month. | PDF, Excel | P1 |
| **심사 Report** | Pass/fail rates, promotion history, upcoming eligibility. | PDF, Excel | P1 |
| **Alert Effectiveness** | Notification send counts, open rates, click-through rates, action rates. | Dashboard | P2 |

#### 6.6.2 Dashboard Visualizations

| Visualization | Description | Priority |
|---|---|---|
| **Member Distribution** | Charts: by age group, belt level, membership type. | P1 |
| **Attendance Trends** | Line chart: daily/weekly attendance over time. Heatmap: attendance by day-of-week and time. | P1 |
| **Revenue Trends** | Monthly revenue trend line. Breakdown pie chart (membership vs. 심사 vs. retail). | P1 |
| **Retention Funnel** | Registration → Active → At-Risk → Churned member funnel. | P2 |

#### 6.6.3 Advanced Analytics (Future)

| Capability | Description | Priority |
|---|---|---|
| **Churn Prediction** | ML model predicts members likely to churn based on attendance patterns, payment history, engagement. | P3 |
| **Optimal Class Scheduling** | Analyze attendance data to recommend optimal class times and sizes. | P3 |
| **Revenue Forecasting** | Predict monthly revenue based on current membership, renewal rates, and seasonal patterns. | P3 |

---

### 6.7 Admin Operational Tools

| Requirement | Description | Priority |
|---|---|---|
| **TODO/Task System** | Admin can create and track operational tasks (follow up with parent, schedule equipment order, etc.). | P1 |
| **Staff Accounts & Roles** | Role-based access: Owner (full access), Manager (limited admin), Instructor (class & attendance only). | P0 |
| **Audit Log** | Every data change logged: who, what, when, previous value. | P0 |
| **Announcement System** | Broadcast announcements to all members or filtered groups (by class, belt level, etc.). | P0 |
| **Calendar Integration** | Sync class schedule to Google Calendar. | P2 |
| **AI Chatbot** | Auto-respond to common parent inquiries (hours, pricing, schedule) via web chat. | P3 |

---

### 6.8 Korean-Heritage & Community Features

These features address workflows rooted in Korean Taekwondo gym culture that **zero Western competitors support**, and that Korean-American gym owners bring to their US operations.

#### 6.8.1 가정통신문 (Home-School Newsletter System)

Korean-heritage gym owners often maintain the formal newsletter tradition (가정통신문) from Korean education culture. This also serves as a professional communication channel for all parents regardless of background.

| Requirement | Description | Priority |
|---|---|---|
| **Template Editor** | WYSIWYG editor with gym-branded templates for newsletters. | P1 |
| **Audience Targeting** | Send to all parents, or filter by class, belt level, age group. | P1 |
| **Multi-Channel Delivery** | Deliver via in-app, email, or SMS based on parent preference. Multilingual support. | P1 |
| **Read Receipts** | Track which parents have read the newsletter. Admin sees read/unread dashboard. | P2 |
| **Scheduled Publishing** | Schedule newsletters for future delivery. | P2 |
| **Attachment Support** | Attach documents (gym policies, event schedules, etc.). | P2 |

#### 6.8.2 Kukkiwon (국기원) Integration

Kukkiwon is the World Taekwondo headquarters (based in Seoul) that manages official 승품·단 (belt promotion) certification globally. US TKD gyms regularly submit exam applications to Kukkiwon. Their TCON platform handles exam registration.

| Requirement | Description | Priority |
|---|---|---|
| **Data Export for TCON** | Generate TCON-compatible data export (student info, current rank, exam application) to eliminate manual re-entry by gym owners. | P2 |
| **API Integration** | If Kukkiwon TCON API becomes available: direct submission of 승품·단 exam applications from the platform. | P3 |
| **Certification Tracking** | Store Kukkiwon certificate numbers per member. Track official vs. gym-internal belt levels separately. | P1 |

#### 6.8.3 Multilingual Communication Stack

| Requirement | Description | Priority |
|---|---|---|
| **Trilingual UI** | Full platform UI in English (default), Korean, and Spanish. User selects preferred language at registration. | P0 |
| **Language-Aware Notifications** | All automated notifications (attendance, payment, 심사) sent in parent’s preferred language. | P0 |
| **Multilingual Templates** | Admin creates notification/newsletter templates with per-language variants. | P1 |
| **Third-Party Messenger (Pluggable)** | Agnostic messenger integration layer. Initial implementation: KakaoTalk for Korean-speaking parents. Architecture supports adding WhatsApp, Facebook Messenger, LINE, etc. via adapter pattern. | P2 |
| **Cultural UX Patterns** | Korean-language communications use appropriate honorific forms (존칭어). Spanish communications use formal usted form. | P1 |

#### 6.8.4 Parent Activity Feed (오늘 수련 일지)

Modeled after Brightwheel's daily activity feed — the primary parent engagement surface.

| Requirement | Description | Priority |
|---|---|---|
| **Daily Training Notes** | Instructor posts what the class practiced today (forms, sparring, conditioning, etc.). Per-class, visible to parents of attending children. | P1 |
| **Photo/Video Sharing** | Staff takes photos during class and shares to parent app. Privacy controls: photos visible only to that child's parents. | P1 |
| **Instructor Observations** | Per-child notes from instructor ("Great focus today" / "Needs work on roundhouse kick form"). | P2 |
| **Weekly Progress Summary** | Auto-generated weekly summary: classes attended, skills practiced, instructor notes. Sent to parent on configurable day. | P2 |
| **Parent Reactions** | Parents can react to feed items (like, comment). Two-way engagement, not just broadcast. | P2 |
| **Milestone Celebrations** | Automatic celebratory posts for: belt promotion, attendance streak (10/50/100 classes), birthday. Shareable by parents to social media. | P2 |
---

## 7. Competitive Analysis

### 7.1 Market Landscape

Analysis of 11 existing gym management platforms reveals consistent gaps in the Korean TKD market:

| Competitor | Annual Revenue (Est.) | Martial Arts Focus | Belt Tracking | Kiosk Mode | Parent Communication | Korean Support |
|---|---|---|---|---|---|---|
| **Mindbody** | $315M | No (general fitness) | Basic | No | Basic email/SMS | No |
| **WellnessLiving** | $74M | Partial | Yes | No | Email/SMS/Push | No |
| **Glofox** | $42M | No (general fitness) | No | No | Basic | No |
| **Zen Planner** | $22M | Yes | Yes (strong) | No | Email | No |
| **Clubworx** | $20M | Yes | Yes (auto-grading) | **Yes** | App notifications | No |
| **PushPress** | $14M | Partial | Basic | No | SMS/Email | No |
| **Gymdesk** | $7M | Yes | Yes (drag-drop) | No | Basic | No |
| **Kicksite** | $4M | Yes (strong) | Yes | No | Email/Text | No |
| **Martialytics** | N/A | Yes (strong) | Yes | No | Email/SMS | No |
| **Spark Membership** | N/A | Yes | No | No | SMS/Email | No |
| **RhinoFit** | N/A | Partial | Yes | No | Email | No |

### 7.2 Key Gaps We Fill

| Gap | Current State | Ararat's Approach |
|---|---|---|
| **No face recognition attendance** | All competitors use QR, card swipe, or manual check-in. Clubworx has kiosk mode but no face recognition. | On-device face recognition with liveness detection as primary check-in method, with QR/NFC/manual as fallbacks. |
| **No parent activity feed** | All competitors treat parents as passive payers. No real-time visibility into what the child did in class. No photo sharing, no daily notes, no progress feed. | Brightwheel-style daily activity feed (오늘 수련 일지): training notes, class photos, instructor observations, milestone celebrations. Parents become active participants, not passive payers. |
| **No multilingual support** | Zero Western competitors offer Korean or Spanish language support. Korean-American gym owners cannot communicate with all parents through one platform. | Trilingual UI/UX (English, Korean, Spanish). Language-aware notifications. Pluggable third-party messenger integration (KakaoTalk first, WhatsApp next). |
| **심사 workflow missing** | Most offer belt “tracking” (static field). None manage the full 심사 lifecycle. Kukkiwon TCON only handles exam registration, not gym-side management. | Complete 심사 management: scheduling → eligibility → parent notification → form collection → fee payment → results → belt update → certificate → Kukkiwon data export. |
| **No newsletter system** | No competitor has formal newsletter capability. Korean-heritage gym owners lose the 가정통신문 tradition; all owners lack a professional broadcast tool. | Multilingual templated newsletter editor with audience targeting, multi-channel delivery, and read receipts. |
| **Child safety at drop-off** | No competitor addresses the parent drop-off verification problem. | Dual-check kiosk verification, 5-minute unconfirmed alerts, full audit trail with timestamps and staff IDs. |
| **No parent satisfaction tracking** | No competitor measures parent engagement or satisfaction as a KPI. | Parent NPS tracking, post-class feedback collection, engagement-based churn signals. |
| **Retention intelligence** | Basic absence alerts at most. No predictive analytics. | Configurable multi-tier absence alerts, retention dashboard, parent engagement metrics, and (future) ML-based churn prediction. |

### 7.3 Features Adopted from Competitors

| Feature | Source Competitor | Our Implementation |
|---|---|---|
| Belt rank tracking with drag-drop management | Gymdesk | Belt management UI with drag-drop promotion, batch operations. |
| Automated grading/promotion system | Clubworx | Full 심사 lifecycle — goes further than Clubworx's auto-grade. |
| Family account with unified billing | Zen Planner, WellnessLiving, Mindbody | Parent account linking multiple children, single family invoice, sibling discounts. |
| Curriculum/skill sharing via video | Zen Planner | Instructor can share technique videos/resources per belt level (future). |
| Custom branded experience | WellnessLiving, Glofox | Each gym gets configurable branding (logo, colors, name) within the platform. |
| Lead capture & trial management | Kicksite, Spark Membership | Trial member workflow with automated conversion follow-ups (future). |
| Class kiosk self-check-in | Clubworx | Upgraded with face recognition + QR + search, plus parent notification. |
| Marketing campaign tools | WellnessLiving, Spark | Email/SMS campaign builder with templates and analytics (future). |
| POS & inventory | Spark Membership, RhinoFit | Equipment and uniform sales tracking (future). |

### 7.4 Pricing Positioning

Competitor pricing ranges from **$0 (limited) to $699+/month**, with most martial arts-specific tools in the **$75-$199/month** range for 50-200 members.

**Ararat target pricing** (to be validated):

| Tier | Target Price | Members | Rationale |
|---|---|---|---|
| Starter | ~$79/month | Up to 50 | Below Gymdesk ($75-$100), competitive entry point. |
| Standard | ~$149/month | Up to 150 | In line with Kicksite/Spark, but with face recognition + parent comms included. |
| Professional | ~$229/month | Up to 300 | Comparable to Zen Planner, but with full 심사 workflow + kiosk. |
| Enterprise | Custom | 300+ / Multi-location | For franchise operators. |

All tiers include: kiosk app, parent notifications, 심사 management. No feature gating on core differentiators.
Payment processing fee: **2.9% + $0.30/transaction** (industry standard via Stripe).

### 7.5 Korean-Origin Tools (Reference Only)

These tools exist in Korea but do **not** compete in the US market. They serve as feature references and cautionary tales:

| Tool | Relevance | Key Takeaway |
|---|---|---|
| **태권프렌즈 (TKFriends)** | TKD-specific with parent photo albums, tuition. Stagnant (last updated 2023). | Validates demand for parent photo sharing. Proves the market wants these features — but execution failed. |
| **클래스노트 (ClassNote)** | KidsNote spin-off for sports/arts education. Growing 340% YoY in Korea. | The Brightwheel model works for Korean parents. Strongest validation of the parent activity feed approach. |
| **국기원 TCON** | Official Kukkiwon 승품·단 exam platform. Launched mobile July 2024. | Integration target — US gyms submit to TCON regularly. Data export compatibility is a real differentiator. |

> **Core Positioning Insight**: US competitors built for **gym operators**. But in TKD gyms, **parents** are the real retention lever — they decide whether their child continues. **The platform that wins is the one that makes parents feel like active participants in their child’s martial arts journey, not passive payers.** Brightwheel won in childcare with this exact playbook. Applied to TKD, the opportunity is wide open.

---

## 8. Technical Considerations

### 8.1 Architecture

| Aspect | Decision | Rationale |
|---|---|---|
| **Multi-Tenancy** | Single deployment, data-isolated per gym (tenant). | Universal platform — any gym onboards without custom deployment. |
| **Kiosk Face Recognition** | On-device processing using KBY-AI or FacePlugin SDK on iPad (Apple Vision for detection layer + dedicated SDK for 1:N recognition). | No biometric data leaves the device. COPPA/BIPA compliance. Face embeddings stored only on-device, encrypted. |
| **Face Data Enrollment** | During registration, parent uploads child's photos via app → photos sent to kiosk locally (LAN or secure sync) → kiosk generates face vectors on-device. | Eliminates cloud biometric storage entirely. |
| **Notification Infrastructure** | Firebase Cloud Messaging (push), Twilio (SMS), SendGrid (email). Pluggable third-party messenger adapter (initial: KakaoTalk Business API, extensible to WhatsApp, LINE, etc.). | Multi-channel with per-gym configurable preferences. Notifications sent in parent’s preferred language. Messenger integration via adapter pattern for easy provider addition. |
| **Payment Processing** | Stripe (US credit card, ACH, Apple Pay, Google Pay). | US-only payments. Stripe handles all payment methods relevant to US market. |
| **Offline Support** | Kiosk app: full offline check-in with sync. Web apps: online-only (acceptable for admin/parent use cases). | Kiosk reliability is critical — gym may have unstable WiFi. |
| **API-First Design** | RESTful API backend serving all four applications. | Enables future mobile native apps, third-party integrations, and API access for gyms. |

### 8.2 Face Recognition Technical Design

**Important**: Apple Vision Framework provides face *detection* (bounding boxes + landmarks) but NOT face *recognition* (1:N identity matching). A dedicated on-device recognition SDK is required.

| Component | Specification |
|---|---|
| **Detection Layer** | Apple Vision Framework (`VNDetectFaceRectanglesRequest`, `VNDetectFaceLandmarksRequest`) for face detection + landmark extraction on iPad. |
| **Recognition Layer** | KBY-AI Face Recognition SDK (top NIST-ranked, full on-device 1:N matching via TensorFlow Lite) or FacePlugin SDK (99.85% NIST accuracy, on-premise deployment). Apple Vision alone is insufficient for identity matching. |
| **Liveness Detection** | **Mandatory** — passive liveness detection (no user action required) to prevent photo/video spoofing. Both KBY-AI and FacePlugin include liveness. Critical for child safety. |
| **Processing** | 100% on-device. No cloud calls for recognition. Server receives only member ID + timestamp after successful match — never biometric data. |
| **Enrollment** | Parent uploads 3-5 photos per child during registration. Photos synced to kiosk device via secure LAN/local sync. Kiosk generates 128-dimensional face embeddings using on-device ML model. Original photos deleted from kiosk after embedding generation. |
| **Storage** | Face embeddings stored in encrypted local database on kiosk device (iOS Keychain or encrypted SQLite). Embeddings are mathematical vectors that cannot reconstruct the original face. |
| **Matching** | Cosine similarity between live capture embedding and stored embeddings. Threshold: configurable (default 0.85 similarity). |
| **Performance** | Target: < 2 second recognition time for up to 500 enrolled members per kiosk. |
| **Fallback Chain** | Face recognition → QR code (app or printed card) → NFC tag → Name search → PIN code → Staff assistance. |
| **Child-Specific Consideration** | Children’s faces change rapidly due to growth. System prompts re-enrollment every 6-12 months for members under age 10. Recognition model must handle age-related facial changes. |
| **Multi-Kiosk Sync** | For gyms with multiple kiosks: face embeddings sync between devices via encrypted local network transfer. No cloud intermediary for biometric data. |
| **Data Lifecycle** | Embeddings deleted from kiosk when: member withdraws, parent revokes consent, or gym offboards. Deletion is logged in audit trail. |

### 8.3 Integrations

| Integration | Service | Purpose | Phase |
|---|---|---|---|
| Payment | Stripe | US credit card, ACH, Apple Pay, Google Pay | MVP |
| Email | SendGrid / AWS SES | Transactional + marketing email (multilingual) | MVP |
| SMS | Twilio | SMS notifications (US numbers) | MVP |
| Push Notifications | Firebase Cloud Messaging | Real-time push to mobile | MVP |
| Cloud Storage | AWS S3 / GCP Cloud Storage | Photo storage, report archives | MVP |
| Calendar | Google Calendar API | Schedule sync | Phase 2 |
| Face Recognition | KBY-AI SDK / FacePlugin SDK | On-device 1:N face recognition for kiosk | Phase 2 |
| Third-Party Messenger | KakaoTalk Business API (initial) | Pluggable messenger adapter — KakaoTalk first, WhatsApp/LINE/FB Messenger extensible | Phase 2 |
| Kukkiwon TCON | Kukkiwon TCON API (if available) / data export | 승품·단 exam registration data sync | Phase 3 |

### 8.4 Scalability

| Consideration | Approach |
|---|---|
| **Auto-Scaling** | Cloud-native deployment (AWS/GCP) with auto-scaling groups. |
| **Database** | Per-tenant data isolation at the application layer. Scalable managed database (e.g., RDS PostgreSQL or Cloud SQL). |
| **Media Storage** | CDN-backed object storage for photos and documents. |
| **Multi-Language** | i18n framework from Day 1. Launch languages: **English** (default), **Korean**, **Spanish**. All UI, notifications, and templates trilingual. |
| **Multi-Location** | Support for gym chains — single admin account managing multiple locations with independent member rosters. |

---

## 9. Privacy, Security & Compliance

### 9.1 Regulatory Compliance

**⚠️ Critical Timeline**: The FTC’s amended COPPA Rule (effective June 2025) now explicitly includes biometric identifiers (facial templates/faceprints) in the definition of “personal information.” **Compliance deadline: April 22, 2026.** Civil penalties up to $53,088 per violation.

| Regulation | Scope | Key Requirements | Penalty |
|---|---|---|---|
| **COPPA** (US Federal) | Children under 13 | Verifiable parental consent required before collecting child data. **Biometric data now explicitly covered** (amended rule 2025). Written data retention policy required. Child-specific information security program with annual risk assessment. Third-party disclosure consent for any cloud SDK. | Up to **$53,088 per violation** |
| **PIPA** (개인정보보호법, Korea) | Reference only — not directly applicable (US operations). However, if future expansion to Korea is considered, biometric data is classified as sensitive information (민감정보) under Article 23. Under-14 requires guardian consent. | Up to 3% of global revenue |
| **BIPA** (Illinois, US) | Biometric data (all ages) | Informed written consent before collecting biometric identifiers. Publicly available retention/destruction policy. No sale or profit from biometric data. **Each scan is a separate violation** (IL Supreme Court ruling). | **$1,000–$5,000 per scan** (private right of action) |
| **State Privacy Laws** (US) | Varies by state | Texas (CUBI), Washington (HB 1493), New York City (§22-12), Portland OR (34-10) all require notice + consent for biometric collection. Treat BIPA as baseline for all states. | Varies |
| **CCPA/CPRA** (California) | California residents | Biometric data is “sensitive personal information.” Consumer can limit use. Right to deletion. | $2,500–$7,500 per violation |

**Mandatory pre-launch action**: Conduct a **Privacy Impact Assessment (PIA)** before deploying face recognition. Strongly recommended under COPPA and required by BIPA best practices. Parental consent is the only valid legal basis for collecting children’s biometric data.

### 9.2 Biometric Data Policy

| Policy | Detail |
|---|---|
| **Consent** | Written parental consent required during registration before any face data processing. Consent is revocable at any time. |
| **On-Device Only** | Face embeddings are generated and stored exclusively on the kiosk device. Never transmitted to cloud servers. |
| **No Raw Storage** | Original face photos are not retained on kiosk after embedding generation. Only mathematical embeddings (non-reversible) are stored. |
| **Deletion** | Embeddings purged when: member withdraws, consent revoked, or data retention period expires. |
| **Alternative Required** | Face recognition is never mandatory. QR code and manual search are always available as alternatives. Parents can opt out of face recognition entirely. |
| **Audit Trail** | System logs when embeddings are created, accessed for matching, and deleted — but never logs the embeddings themselves. |

### 9.3 Security Measures

| Measure | Implementation |
|---|---|
| **Transport** | TLS 1.3 for all API communication. Certificate pinning on kiosk app. |
| **Authentication** | Phone-based OTP for parent accounts. Email + password with 2FA (TOTP) for admin accounts. |
| **Authorization** | Role-based access control (RBAC): Owner, Manager, Instructor, Parent, Member. |
| **Data Encryption** | At-rest encryption for all databases and file storage. Kiosk local database encrypted with device-specific key. |
| **Audit Logging** | All data access, modifications, and deletions logged with actor, timestamp, and IP. |
| **Data Backup** | Daily automated backups with 30-day retention. Point-in-time recovery capability. |
| **Penetration Testing** | Annual third-party security audit. |

---

## 10. Release Phases & Roadmap

### Phase 1 — MVP (Months 1-6)

**Goal**: Core gym operations + parent visibility. Enough to replace spreadsheets and manual processes for a single gym.

| Feature | Scope |
|---|---|
| **Parent/Member App** | Registration (parent + child), profile management, membership status, attendance history, payment history, notifications inbox. |
| **Admin App** | Registration approval, member directory, class management, basic attendance dashboard, payment/billing (Stripe), manual notification send, basic reporting (attendance + financial). |
| **Kiosk App** | QR code check-in, name search check-in, attendee display, parent SMS notification on check-in. |
| **Notifications** | Check-in alerts (SMS + push), payment reminders (email), membership expiry alerts. |
| **Payments** | Monthly recurring billing via Stripe (US), one-time charges, basic refund, receipt generation. |
| **심사** | Basic: schedule exam, collect fee, enter results, update belt level. |
| **Multilingual** | Full trilingual UI: English (default), Korean, Spanish. Language-aware notifications from Day 1. |

### Phase 2 — Smart Attendance + Community Features (Months 7-12)

**Goal**: Face recognition kiosk, parent engagement features, advanced notification engine.

| Feature | Scope |
|---|---|
| **Kiosk App** | Face recognition check-in (on-device via KBY-AI/FacePlugin), liveness detection, offline mode with sync, staff validation dual-check, unconfirmed arrival alert, re-enrollment prompts for children under 10. |
| **Parent Activity Feed** | Brightwheel-style daily training notes, class photo sharing, instructor observations, milestone celebrations. |
| **Newsletter System** | Multilingual newsletter editor with templates, audience targeting, multi-channel delivery, read receipts. |
| **Alert Rules Engine** | Configurable absence thresholds, late payment escalation, multilingual template editor with personalization. |
| **Parent Messaging** | 1:1 and broadcast messaging between gym owner/instructors and parents. Language-aware. |
| **Advanced Billing** | Membership hold/suspend, family billing, sibling discounts, prorated refund calculator. |
| **Reporting** | Retention report, member distribution charts, attendance trend visualizations, parent engagement metrics. |
| **Monitor App** | TV display: today’s schedule, live attendance board, announcements. |
| **Third-Party Messenger** | Pluggable messenger integration — KakaoTalk as initial provider. Adapter architecture for adding WhatsApp, LINE, etc. |

### Phase 3 — Intelligence + Growth (Months 13-18)

**Goal**: AI-driven insights, marketing tools, and scaling features.

| Feature | Scope |
|---|---|
| **AI/ML** | Churn prediction model, attendance pattern analysis, optimal class scheduling recommendations. |
| **Marketing** | Email/SMS campaign builder, lead capture, trial member workflow, referral tracking. |
| **Advanced 심사** | Eligibility auto-check, certificate generation, 심사 history analytics, Kukkiwon TCON data export. |
| **Multi-Location** | Single admin managing multiple gym locations. Cross-location reporting. |
| **Kukkiwon Integration** | TCON-compatible data export for 승품·단 exam registration. API integration if available. |
| **Parent NPS** | Post-class feedback collection, parent satisfaction tracking, engagement-based churn signals. |
| **POS/Retail** | Equipment and uniform sales, inventory tracking. |
| **Calendar Sync** | Google Calendar integration for class schedules. |
| **AI Chatbot** | Auto-respond to common parent inquiries on the website. |

### Phase 4 — Platform Maturity (Months 19-24)

| Feature | Scope |
|---|---|
| **Native Mobile Apps** | iOS and Android native apps for parents (wrapping the responsive web or full native). |
| **Custom Branded App** | White-label option for large gyms. |
| **Curriculum Sharing** | Instructors share technique videos per belt level. |
| **Social/Community** | Gym social feed, parent community features. |
| **Advanced Analytics** | Revenue forecasting, marketing ROI analysis, campaign optimization. |
| **API for Third Parties** | Public API for gyms to integrate with their own tools. |
| **NFC Tag Check-In** | NFC badge alternative on kiosk. |

---

## 11. Success Metrics

### Product Metrics

| Metric | Target (12 months post-launch) |
|---|---|
| **Gym Onboarding** | 50+ gyms active on the platform. |
| **Member Activation** | 80%+ of registered members have active parent accounts. |
| **Kiosk Check-In Rate** | 90%+ of daily check-ins happen via kiosk (face/QR) vs. manual. |
| **Parent Notification Engagement** | 70%+ of parents open check-in notifications. |
| **Payment Collection Rate** | 95%+ of recurring payments collected on time (vs. industry ~85%). |
| **Member Retention** | Gyms on Ararat see 15%+ improvement in member retention vs. prior system. |

### Business Metrics

| Metric | Target |
|---|---|
| **MRR** | $50K+ MRR by month 12. |
| **Churn** | < 5% monthly gym churn. |
| **NPS** | 50+ among gym owners. |
| **ARPU** | $150+ per gym per month. |

### Operational Metrics

| Metric | Target |
|---|---|
| **Face Recognition Accuracy** | > 98% correct identification rate. < 0.1% false positive rate. |
| **Check-In Speed** | < 2 seconds from face detection to confirmation. |
| **Notification Delivery** | 99%+ delivery rate for push/SMS within 30 seconds of trigger. |
| **System Uptime** | 99.9% uptime. |

---

## 12. Appendix

### A. Glossary

| Term | Definition |
|---|---|
| **관장님** (Gwanjangnim) | Gym owner/master. The primary admin user of the platform. |
| **도장** (Dojang) | Taekwondo gym/training hall. |
| **심사** (Simsa) | Belt promotion examination. A formal test where students demonstrate skills to advance to the next belt level. |
| **띠** (Tti) | Belt. Represents the student's rank/level in Taekwondo. |
| **수련생** (Suryeonsaeng) | Student/trainee. |
| **사범** (Sabeom) | Instructor/master instructor. |
| **학부모** (Hakbumo) | Parent/guardian of a student. |
| **탈퇴** (Talhoe) | Membership withdrawal/cancellation. |

### B. Competitor Pricing Summary

| Competitor | Entry Price | Mid-Tier | High-Tier | Payment Fee |
|---|---|---|---|---|
| Zen Planner | $99/mo | $189-$239/mo | $348+/mo | 2.9% + $0.30 |
| WellnessLiving | $69/mo | $199/mo | $349/mo | 2.9% + $0.30 |
| Gymdesk | $75/mo | $100-$150/mo | $200+/mo | 2.9% + $0.30 |
| Mindbody | $99/mo | $129-$199/mo | $699+/mo | 2.75% + $0.30 |
| Kicksite | $49/mo | $99-$149/mo | $199/mo | 2.9% + $0.30 |
| Spark Membership | $149/mo | $199/mo | $299+/mo | 2.9% + $0.30 |
| RhinoFit | $0 (20 members) | $57/mo | — | 2.9% + $0.30 |
| PushPress | $0 (basic) | $159/mo | — | 1.99-2.19% + $0.30 |
| Glofox | $110-$150/mo | $160-$250/mo | $250+/mo | 2.9% + $0.30 |
| Clubworx | $0 (20 members) | $89/mo | $169/mo | 2.9% + $0.30 |
| Martialytics | $69/mo | $99-$149/mo | $199/mo | 2.0% |

### C. Estimated Development Budget

Based on competitor analysis and scope:

| Phase | Duration | Estimated Cost |
|---|---|---|
| Phase 1 (MVP) | 6 months | $60,000 - $90,000 |
| Phase 2 (Smart + Korean) | 6 months | $40,000 - $60,000 |
| Phase 3 (Intelligence) | 6 months | $30,000 - $50,000 |
| Phase 4 (Maturity) | 6 months | $20,000 - $40,000 |
| **Total** | **24 months** | **$150,000 - $240,000** |

**Ongoing costs**: ~$500-$2,000/month (cloud hosting, third-party APIs, SMS/email services) scaling with number of gyms.

### D. Open Questions

1. **Kiosk Hardware** — iPad-only or also support Android tablets? KBY-AI supports both, but iPad has better Neural Engine performance. Android tablets are cheaper for gym owners ($200 vs $500+).
2. **Face Recognition SDK Final Selection** — KBY-AI (freemium, top NIST) vs. FacePlugin (99.85% accuracy, on-premise) vs. custom Core ML model. Need proof-of-concept with children’s faces specifically.
3. **Face Recognition Legal Review** — **URGENT**: COPPA amended rule compliance deadline is **April 22, 2026**. Need legal counsel review of biometric data handling across US states (BIPA: $1K-$5K per scan) before Phase 2 launch. Privacy Impact Assessment (PIA) is mandatory.
4. **Child Re-Enrollment Frequency** — Research suggests re-enrollment every 6-12 months for children under 10 due to facial growth. Need to validate this cadence and design a frictionless re-enrollment UX.
5. **Third-Party Messenger Priority** — KakaoTalk is the initial messenger integration. What should be the second provider? WhatsApp (high Hispanic-American adoption) vs. Facebook Messenger (broad US reach)? Survey gym owner parent demographics to decide.
6. **Spanish Language Priority** — Should Spanish launch in MVP alongside English/Korean, or is Phase 2 acceptable? Depends on member demographic data from early gym partners.
7. **Kukkiwon TCON API Availability** — TCON launched mobile in July 2024 but it’s unclear if they offer a public API. If not, start with data export format and pursue partnership/API access.
8. **White Label vs. Branded** — Should each gym get a custom-branded experience (like WellnessLiving) or use Ararat branding throughout?
9. **Pricing Validation** — Target pricing needs validation through US gym owner (관장님) interviews before finalizing.
11. **Competitor Response** — Kicksite and Gymdesk are the strongest US martial arts competitors. Monitor for parent communication features or multilingual support additions.
