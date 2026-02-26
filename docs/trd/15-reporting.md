# 15. Feature: Reporting & Analytics

**Related TRDs**: [03-data-model](./03-data-model.md), [08-attendance](./08-attendance.md), [10-payments](./10-payments.md), [11-simsa](./11-simsa.md)  
**Related ADRs**: _None_  
**Phase**: MVP (Phase 1) for basic reports, Phase 2 for advanced visualizations

---

### Standard Reports

#### Attendance Report

**Data Source**: Attendance table.

**Filters**: Date range, class, age group, belt level.

**Columns**: Member name, classes attended, classes missed, attendance rate (%), last attendance date.

**Aggregation**: Daily/weekly/monthly rates.

**Export Formats**: PDF, Excel.

**Example**: "February 2026 Attendance Report - All Classes: 85% average attendance rate. Breakdown by class: Beginner (88%), Advanced (82%)."

#### Financial Report

**Data Source**: Payment, Invoice, Refund tables.

**Filters**: Date range, payment type (membership, simsa, equipment).

**Columns**: Revenue by category, total revenue, outstanding balances, refunds issued, discounts applied.

**Aggregation**: Monthly totals.

**Export Formats**: PDF, Excel.

**Example**: "February 2026 Financial Report: Total revenue $12,450. Membership: $10,000 (80%), Simsa fees: $1,500 (12%), Equipment: $950 (8%). Outstanding balances: $2,100."

#### Member Roster

**Data Source**: Member table.

**Filters**: Status (active, inactive, withdrawn), belt level, class group.

**Columns**: Member name, belt level, age group, class group, join date, status, contact info.

**Export Formats**: PDF, Excel.

**Example**: "Current Member Roster: 145 active members. 32 white belts, 28 yellow belts, ... 15 black belts."

#### Retention Report

**Data Source**: Member table (join_date, withdrawal_date).

**Filters**: Date range.

**Columns**: New enrollments, withdrawals, net growth, churn rate (%).

**Aggregation**: Monthly.

**Export Formats**: PDF, Excel.

**Example**: "February 2026 Retention Report: 12 new enrollments, 3 withdrawals, net growth +9 members. Churn rate: 2.1%."

#### 심사 Report

**Data Source**: SimsaResult table.

**Filters**: Date range, belt level.

**Columns**: Belt level, total tested, passed, failed, pass rate (%).

**Aggregation**: By belt level.

**Export Formats**: PDF, Excel.

**Example**: "Spring 2026 Belt Test Results: White belt (20 tested, 18 passed, 90%). Yellow belt (15 tested, 12 passed, 80%)."

### Dashboard Visualizations

#### Member Distribution

Pie charts showing:
- Members by age group (Little Kids, Kids, Teens, Adults).
- Members by belt level (White, Yellow, ..., Black).
- Members by membership status (Active, Inactive, Withdrawn).

#### Attendance Trends

- **Line Chart**: Daily/weekly attendance over time (last 12 weeks).
- **Heatmap**: Attendance by day-of-week and time-of-day (e.g., Monday 4 PM has highest attendance).

#### Revenue Trends

- **Line Chart**: Monthly revenue over time (last 12 months).
- **Pie Chart**: Revenue breakdown (membership vs. simsa vs. equipment).

#### Retention Funnel

Funnel chart showing: Registration → Active → At-Risk (absent 7+ days) → Churned.

### Report Generation

**On-Demand**:
1. Admin clicks "Generate Report".
2. Admin selects report type and filters.
3. System generates report and displays preview.
4. Admin clicks "Export" to download PDF/Excel.

**Scheduled**:
1. Admin configures scheduled reports (e.g., "Monthly Financial Report every 1st of month").
2. Cron job generates report and emails to admin.

### Implementation Notes

> _This section will be updated as the feature is implemented._

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
