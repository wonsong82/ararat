# 16. Feature: Reporting & Analytics

**Related TRDs**: [03-data-model](./03-data-model.md), [10-attendance](./10-attendance.md), [11-payments](./11-payments.md), [12-simsa](./12-simsa.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1) for basic reports, Phase 2 for advanced visualizations

---

## Overview

Report generation and dashboard analytics for gym operations — attendance, financial, roster, retention, and 심사 reports with charts and export capabilities. The reporting system serves gym owners and staff through the Admin App, providing both on-demand report generation with preview/export and scheduled recurring reports delivered via email. Dashboard visualizations give at-a-glance operational insights.

---

## Business Rules

### Standard Reports

Five report types are available, each with defined data sources, filters, columns, aggregation levels, and export formats.

#### 1. Attendance Report

| Aspect | Detail |
|--------|--------|
| **Data Source** | Attendance records (check-in/check-out logs) |
| **Filters** | Date range (required), class, age group, belt level |
| **Columns** | Member name, classes attended, classes missed, attendance rate (%) |
| **Aggregation** | Daily, weekly, monthly |
| **Export Formats** | PDF, Excel |
| **Example** | "March 2026 weekly attendance for Junior class: 45 members, avg 3.2 classes/week, 82% attendance rate" |

#### 2. Financial Report

| Aspect | Detail |
|--------|--------|
| **Data Source** | Payment transactions, invoices |
| **Filters** | Date range (required), payment type (membership, 심사, equipment, other) |
| **Columns** | Revenue by category, totals, outstanding balance, refunds, discounts applied |
| **Aggregation** | Monthly |
| **Export Formats** | PDF, Excel |
| **Example** | "Q1 2026: $45,200 membership revenue, $3,800 심사 fees, $1,200 equipment sales, $890 outstanding, $450 refunds" |

#### 3. Member Roster

| Aspect | Detail |
|--------|--------|
| **Data Source** | Member profiles, enrollment records |
| **Filters** | Membership status (active, inactive, withdrawn), belt level, class group |
| **Columns** | Member name, age, belt level, join date, membership status, contact info (phone, email), guardian name (for minors) |
| **Aggregation** | N/A (list report) |
| **Export Formats** | PDF, Excel |
| **Example** | "Active members in Tiger class: 22 members, belts ranging white–green, ages 6–9" |

#### 4. Retention Report

| Aspect | Detail |
|--------|--------|
| **Data Source** | Enrollment records, withdrawal records, attendance records |
| **Filters** | Date range (required) |
| **Columns** | New enrollments, withdrawals, net growth, churn rate (%) |
| **Aggregation** | Monthly |
| **Export Formats** | PDF, Excel |
| **Example** | "2026 H1: 38 new enrollments, 12 withdrawals, +26 net growth, 4.8% monthly churn rate" |

#### 5. 심사 (Belt Promotion) Report

| Aspect | Detail |
|--------|--------|
| **Data Source** | 심사 events, 심사 results |
| **Filters** | Date range (required), belt level |
| **Columns** | Belt level, total tested, passed, failed, pass rate (%) |
| **Aggregation** | By belt level |
| **Export Formats** | PDF, Excel |
| **Example** | "2026 Spring 심사: White→Yellow 15 tested / 14 passed (93%), Yellow→Green 12 tested / 10 passed (83%)" |

### Dashboard Visualizations

Four visualization types provide at-a-glance operational insights on the Admin App dashboard.

#### 1. Member Distribution

- **Chart type**: Pie charts
- **Breakdowns**: By age group, by belt level, by membership status
- **Data**: Current snapshot (not historical)

#### 2. Attendance Trends

- **Chart type**: Line chart (daily or weekly granularity, last 12 weeks) + heatmap (day-of-week × time-of-day)
- **Purpose**: Identify attendance patterns, peak times, and declining trends
- **Data**: Rolling 12-week window, refreshed on page load

#### 3. Revenue Trends

- **Chart type**: Line chart (monthly, last 12 months) + pie chart (revenue breakdown: membership vs 심사 vs equipment vs other)
- **Purpose**: Track financial health and revenue composition
- **Data**: Rolling 12-month window

#### 4. Retention Funnel

- **Stages**: Registration → Active → At-Risk (absent 7+ consecutive days) → Churned (withdrawn)
- **Chart type**: Funnel visualization
- **Purpose**: Identify drop-off points in the member lifecycle
- **Data**: Current snapshot with counts at each stage

### Report Generation

#### On-Demand Reports

1. Admin selects report type and configures filters
2. System generates report and displays preview (table + chart)
3. Admin can toggle between table view and chart view
4. Admin exports as PDF or Excel

#### Scheduled Reports

1. Admin configures: report type, filters, frequency (daily, weekly, monthly), recipient email(s)
2. Cron job generates report at scheduled time
3. System emails PDF attachment to configured recipients
4. Admin can view, edit, or delete scheduled reports

---

## Backend

### API Endpoints

#### Report Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/reports/attendance` | Attendance report | Admin, Staff |
| `GET` | `/api/v1/tenants/{tenantId}/reports/financial` | Financial report | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/reports/roster` | Member roster | Admin, Staff |
| `GET` | `/api/v1/tenants/{tenantId}/reports/retention` | Retention report | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/reports/simsa` | 심사 report | Admin, Staff |

**Common query parameters** (all report endpoints):

```
?startDate=2026-01-01&endDate=2026-03-31&format=json|pdf|excel
```

- `format=json` (default): Returns JSON data for in-app preview
- `format=pdf`: Returns PDF file download
- `format=excel`: Returns XLSX file download

**Report-specific query parameters**:

- Attendance: `&classId=&ageGroup=&beltLevel=&aggregation=daily|weekly|monthly`
- Financial: `&paymentType=membership|simsa|equipment|other`
- Roster: `&status=active|inactive|withdrawn&beltLevel=&classGroup=`
- Retention: (date range only)
- 심사: `&beltLevel=`

#### Dashboard Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/tenants/{tenantId}/dashboard/stats` | Dashboard summary stats (today's check-ins, active members, pending registrations, overdue payments) | Admin, Staff |
| `GET` | `/api/v1/tenants/{tenantId}/dashboard/attendance-trend` | Attendance trend data (last 12 weeks, daily/weekly) | Admin, Staff |
| `GET` | `/api/v1/tenants/{tenantId}/dashboard/revenue-trend` | Revenue trend data (last 12 months, monthly) | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/dashboard/member-distribution` | Member distribution by age/belt/status | Admin, Staff |

#### Scheduled Report Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/api/v1/tenants/{tenantId}/reports/schedule` | Create scheduled recurring report | Admin |
| `GET` | `/api/v1/tenants/{tenantId}/reports/schedule` | List scheduled reports | Admin |
| `PATCH` | `/api/v1/tenants/{tenantId}/reports/schedule/{scheduleId}` | Update scheduled report | Admin |
| `DELETE` | `/api/v1/tenants/{tenantId}/reports/schedule/{scheduleId}` | Delete scheduled report | Admin |

**Schedule request body**:

```json
{
  "reportType": "attendance",
  "filters": {
    "aggregation": "weekly"
  },
  "frequency": "weekly",
  "dayOfWeek": 1,
  "recipientEmails": ["owner@dojang.com"],
  "format": "pdf"
}
```

### Response Shapes

#### Dashboard Stats Response

```json
{
  "todayCheckIns": 42,
  "activeMembers": 187,
  "pendingRegistrations": 3,
  "overduePayments": 7,
  "upcomingSimsa": null,
  "withdrawalRequests": 1
}
```

#### Attendance Trend Response

```json
{
  "granularity": "daily",
  "data": [
    { "date": "2026-02-15", "checkIns": 38 },
    { "date": "2026-02-16", "checkIns": 41 }
  ]
}
```

#### Revenue Trend Response

```json
{
  "data": [
    { "month": "2026-01", "membership": 12500, "simsa": 800, "equipment": 350, "other": 100, "total": 13750 }
  ]
}
```

#### Member Distribution Response

```json
{
  "byBeltLevel": [
    { "belt": "white", "count": 45 },
    { "belt": "yellow", "count": 32 }
  ],
  "byAgeGroup": [
    { "group": "4-6", "count": 28 },
    { "group": "7-9", "count": 52 }
  ],
  "byStatus": [
    { "status": "active", "count": 187 },
    { "status": "inactive", "count": 14 }
  ]
}
```

#### Report Response (JSON format)

```json
{
  "reportType": "attendance",
  "generatedAt": "2026-03-01T10:00:00Z",
  "filters": {
    "startDate": "2026-02-01",
    "endDate": "2026-02-28",
    "aggregation": "weekly"
  },
  "summary": {
    "totalMembers": 187,
    "averageAttendanceRate": 82.3,
    "totalClassesHeld": 96
  },
  "rows": [
    {
      "memberName": "김민수",
      "classesAttended": 12,
      "classesMissed": 4,
      "attendanceRate": 75.0
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "totalRows": 187
  }
}
```

### Service Logic

#### Report Generation Service

- **SQL aggregation queries**: Each report type has dedicated query builders that construct optimized SQL with `GROUP BY`, `COUNT`, `SUM`, `AVG` aggregations. Queries are tenant-scoped via `WHERE tenant_id = ?`.
- **Pagination**: Report JSON responses use offset-based pagination (`page`, `pageSize`, `totalRows`) — a deliberate deviation from TRD 05's cursor-based convention, since report consumers need random page access ("jump to page N"). PDF/Excel exports include all rows (no pagination).
- **PDF generation**: Use a PDF library (e.g., `pdfkit` or `puppeteer` for HTML-to-PDF) to render report with header (gym name, report title, date range, generated timestamp), data table, and summary statistics.
- **Excel generation**: Use `exceljs` to create XLSX with formatted headers, data rows, summary row, and auto-sized columns.
- **Caching**: Dashboard stats and trend data are cached in Redis with 5-minute TTL. Report generation is not cached (always fresh).

#### Aggregation Formulas

| Metric | Formula | Notes |
|--------|---------|-------|
| Attendance rate | `classes_attended / classes_available_for_member` × 100 | Only count classes the member was enrolled in, not all gym classes |
| Churn rate (monthly) | `withdrawals_this_month / active_members_at_month_start` × 100 | Active = status=active at first day of month |
| Revenue by category | `SUM(amount) WHERE payment_type = ? GROUP BY month` | Amounts in cents, convert to dollars for display |
| Net growth | `new_enrollments - withdrawals` for the period | Includes re-enrollments as new |
| Pass rate (심사) | `passed / total_tested` × 100 per belt level | Only count members with result recorded |
| Retention funnel | Active: attendance in last 30 days; At-Risk: no attendance 7–30 days; Churned: no attendance 30+ days OR status=withdrawn | Snapshot calculation, not historical |

#### Timezone Handling

- All date range filters (`startDate`, `endDate`) are interpreted in the gym's local timezone (`SystemSetting.timezone` from [TRD 02](./02-multi-tenancy.md))
- Server converts to UTC for database queries
- Response timestamps are in UTC; frontend formats per user's locale

#### Dashboard Cache Invalidation

- Dashboard stats and trend data cached in Redis with 5-minute TTL (unchanged)
- Write-through invalidation: attendance check-in → invalidate `attendance-trend` cache; payment received → invalidate `revenue-trend` cache; member status change → invalidate `member-distribution` cache
- Invalidation is best-effort — stale data is acceptable for up to 5 minutes

#### Scheduled Report Cron

- Cron job runs every hour, checks for scheduled reports due for execution
- Generates report in configured format (PDF or Excel)
- Sends email with report attached via SendGrid
- Records last execution timestamp and status
- On failure: retries up to 3 times with exponential backoff, logs error, marks schedule as `failed` after exhausting retries

#### Dashboard Stat Calculation

- **Today's check-ins**: `COUNT(attendance) WHERE date = TODAY AND tenant_id = ?`
- **Active members**: `COUNT(members) WHERE status = 'active' AND tenant_id = ?`
- **Pending registrations**: `COUNT(registrations) WHERE status = 'pending' AND tenant_id = ?`
- **Overdue payments**: `COUNT(invoices) WHERE status = 'overdue' AND tenant_id = ?`
- **Pending tasks**: Aggregation of actionable items from registrations, 심사, payments, withdrawals
- **Recent activity**: Last 20 system events (check-ins, payments, registrations) ordered by timestamp

---

## Admin App

### Screen: Dashboard (`/`)

The default landing page for the Admin App, providing at-a-glance operational overview.

#### Layout

```
┌─────────────────────────────────────────────────────────┐
│  Today's Stats (4 cards in a row)                       │
│  [Check-ins: 42] [Active: 187] [Pending: 3] [Overdue: 7]│
├────────────────────────────┬────────────────────────────┤
│  Attendance Chart           │  Revenue Chart             │
│  (line, last 2 weeks)      │  (bar, last 6 months)      │
├────────────────────────────┬────────────────────────────┤
│  Member Distribution        │  Pending Tasks             │
│  (pie, by belt level)      │  - 3 pending registrations │
│                             │  - 심사 in 5 days          │
│                             │  - 7 overdue payments      │
├─────────────────────────────────────────────────────────┤
│  Recent Activity                                        │
│  - 김민수 checked in at 4:32 PM                          │
│  - Payment received from Park family ($150)             │
│  - New registration: Jessica Lee                        │
└─────────────────────────────────────────────────────────┘
```

#### Components & Behavior

- **Today's Stats Cards**: Four metric cards showing total check-ins today, active members, new registrations pending, and overdue payments. Each card is clickable — navigates to the relevant detail screen.
- **Attendance Chart**: Recharts `LineChart` — daily attendance for the past 2 weeks. X-axis: date, Y-axis: check-in count. Click navigates to full Attendance Report.
- **Revenue Chart**: Recharts `BarChart` — monthly revenue for the past 6 months. Stacked by category (membership, 심사, equipment, other). Click navigates to full Financial Report.
- **Member Distribution**: Recharts `PieChart` — members by belt level. Tooltip shows count and percentage.
- **Pending Tasks**: List of actionable items with counts. Each item is clickable — navigates to the relevant screen (registrations list, 심사 management, overdue payments, withdrawal requests).
- **Recent Activity**: Feed of the last 10–20 system events (check-ins, payments, registrations) with timestamps. "View All" link navigates to full activity log.

#### Data Sources

| Hook | Endpoint | Cache |
|------|----------|-------|
| `useDashboardStats()` | `GET /dashboard/stats` | 5 min |
| `useAttendanceTrend()` | `GET /dashboard/attendance-trend` | 5 min |
| `useRevenueTrend()` | `GET /dashboard/revenue-trend` | 5 min |
| `useMemberDistribution()` | `GET /dashboard/member-distribution` | 5 min |
| `usePendingTasks()` | `GET /dashboard/stats` (derived) | 5 min |
| `useRecentActivity()` | `GET /dashboard/recent-activity` | 1 min |

#### Actions

| Action | Behavior |
|--------|----------|
| Click stat card | Navigate to relevant detail screen |
| Click chart | Navigate to full report (Reports Hub with pre-selected type) |
| Click pending task item | Navigate to relevant management screen |
| Click "View All" on Recent Activity | Navigate to full activity log |

### Screen: Reports Hub (`/reports`)

Unified report generation and exploration interface.

#### Layout

```
┌─────────────────────────────────────────────────────────┐
│  Reports                                    [Schedule ⏰]│
├─────────────────────────────────────────────────────────┤
│  [Attendance] [Financial] [Roster] [Retention] [심사]    │
├─────────────────────────────────────────────────────────┤
│  Filters:                                               │
│  Date Range: [2026-02-01] → [2026-02-28]               │
│  Class: [All ▾]  Belt: [All ▾]  Aggregation: [Weekly ▾]│
│                                    [Generate Report 📊] │
├─────────────────────────────────────────────────────────┤
│  View: [Table] [Chart]                 [Export PDF] [XLS]│
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐│
│  │  (Table view — TanStack Table with sort/filter)     ││
│  │  Name       │ Attended │ Missed │ Rate              ││
│  │  김민수      │ 12       │ 4      │ 75%               ││
│  │  Jessica Lee│ 15       │ 1      │ 94%               ││
│  │  ...                                                ││
│  └─────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────┐│
│  │  (Chart view — Recharts visualization)              ││
│  │  📈 Line/Bar/Pie/Funnel depending on report type    ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

#### Components & Behavior

- **Report Type Tabs**: Tab bar to select report type (Attendance, Financial, Roster, Retention, 심사). Switching tabs updates the filter form and clears previous results.
- **Filter Form**: Dynamic filter form per report type:
  - Attendance: date range (required), class (dropdown), age group (dropdown), belt level (dropdown), aggregation (daily/weekly/monthly)
  - Financial: date range (required), payment type (dropdown)
  - Roster: membership status (dropdown), belt level (dropdown), class group (dropdown)
  - Retention: date range (required)
  - 심사: date range (required), belt level (dropdown)
- **Generate Button**: Triggers report generation. Displays loading spinner during fetch. Results appear in the table/chart area below.
- **View Toggle**: Switch between table view and chart view. Both are rendered but only one visible at a time.
- **Table View**: TanStack Table with column sorting, client-side filtering, and pagination. Columns match the report type's defined columns.
- **Chart View**: Recharts visualization appropriate to the report type:
  - Attendance: `LineChart` (trend over time) or `BarChart` (comparison by class/belt)
  - Financial: `BarChart` (revenue by month) + `PieChart` (revenue breakdown)
  - Roster: No chart (list-only report)
  - Retention: `BarChart` (enrollments vs withdrawals) + funnel visualization
  - 심사: `BarChart` (pass/fail by belt level)
- **Export Buttons**: Download current report as PDF or Excel. Triggers backend request with `format=pdf|excel`.
- **Schedule Dialog**: Modal dialog to configure recurring report generation — report type, filters, frequency (daily/weekly/monthly), day-of-week (for weekly), recipient emails, export format (PDF/Excel).

#### Data Sources

| Hook | Endpoint |
|------|----------|
| `useAttendanceReport(filters)` | `GET /reports/attendance` |
| `useFinancialReport(filters)` | `GET /reports/financial` |
| `useMemberRoster(filters)` | `GET /reports/roster` |
| `useRetentionReport(filters)` | `GET /reports/retention` |
| `useSimsaReport(filters)` | `GET /reports/simsa` |
| `useScheduledReports()` | `GET /reports/schedule` |

#### Actions

| Action | Behavior |
|--------|----------|
| Select report type tab | Update filter form, clear results |
| Configure filters + click Generate | Fetch report data, display in table/chart |
| Toggle Table/Chart view | Switch visible panel |
| Click Export PDF | Download PDF via `format=pdf` query param |
| Click Export Excel | Download XLSX via `format=excel` query param |
| Click Schedule button | Open schedule dialog |
| Submit schedule dialog | `POST /reports/schedule` — create recurring report |

---

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| Redis (ElastiCache) | Infrastructure | Cache read/write | GET/SET | Cache key + TTL | Cached JSON |
| SendGrid | External | `POST /v3/mail/send` | POST | Email with PDF/XLSX attachment | `{ statusCode }` |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| `/api/v1/tenants/{tenantId}/reports/{type}` | GET | Admin App | `?startDate=&endDate=&format=json\|pdf\|excel` | Report data or file download |
| `/api/v1/tenants/{tenantId}/dashboard/stats` | GET | Admin App | — | Dashboard summary JSON |
| `/api/v1/tenants/{tenantId}/dashboard/{trend}` | GET | Admin App | — | Trend data JSON |
| `/api/v1/tenants/{tenantId}/reports/schedule` | POST/GET/PATCH/DELETE | Admin App | Schedule config JSON | Schedule confirmation |

---


## Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
