# Project Report: Customer Support & Service Performance Analytics

## 1. Objective

Analyze 10,000 customer support tickets (Jan to Sep 2026) to find where SLA performance is failing, how large the problem is, and what is associated with low customer satisfaction (CSAT).

## 2. Business Questions

1. How many tickets are resolved within SLA?
2. Which priority, product, issue type, team and agent have the most breaches?
3. What is associated with low CSAT?
4. Is performance improving or declining over time?
5. What should management change?

## 3. Approach

| Stage | Tool | What was done |
|---|---|---|
| 1. Profiling | Excel | Checked duplicates, blanks, inconsistent labels, invalid values, date logic and SLA logic. Raw data was left untouched. |
| 2. Cleaning | Excel and Python (Pandas) | Applied cleaning rules, created flag columns, converted dates, ran null checks. |
| 3. Analysis | MySQL | Joins across 5 tables, aggregations, CTEs, window functions (LAG, RANK). |
| 4. Dashboard | Power BI | 3-page dashboard (in progress). |

## 4. Dataset

| Table | Rows |
|---|---|
| tickets | 10,000 |
| customers | 1,200 |
| agents | 55 |
| products | 6 |
| sla_rules | 4 |

## 5. Data Profiling and Cleaning Log

Principle followed: identify an anomaly, validate whether it is really an error, understand the business context, and only then correct, keep or flag it. Values were never invented.

| # | Issue | Rows | Decision | Reason |
|---|---|---|---|---|
| 1 | Duplicate Ticket_IDs (exact duplicate rows) | 25 extra rows | Removed (10,025 to 10,000) | Rows were identical |
| 2 | Priority "HIGH" vs "High" | 8 | Standardized to "High" | Capitalization inconsistency |
| 3 | Blank Resolved_Date and Resolution_Time_Hours | 1,573 | Kept blank | Valid: Open/Pending tickets are not resolved yet |
| 4 | Resolved tickets with missing CSAT | 83 | Flagged (`CSAT_Missing_Flag`), not filled | Scores cannot be invented |
| 5 | SLA status mismatch (time within target but status Breached) | 9 | Flagged (`SLA_Mismatch_Flag`), original kept | See section 6 |
| 6 | Blank Channel | 35 (30 Resolved, 5 Pending) | Filled with "Unknown", flagged (`Channel_Missing_Flag`) | Channel is a category, guessing would create false patterns |
| 7 | 45 empty columns from Excel export | n/a | Dropped in Python | Export artifact |
| 8 | Dates stored as text | n/a | Converted to datetime | Needed for date calculations |

Checks that found no problem: Status, Channel and Issue_Type spelling, SLA_Target_Hours (no blank, zero or negative), Resolution_Time_Hours (no zero or negative), Created vs Resolved date order (0 invalid), Customer_Satisfaction range (1 to 5).

Reconciliation: 1,573 unresolved + 83 resolved-without-score = 1,656 blank CSAT values. Flag counts (83, 9, 35) match across Excel, Python and SQL.

## 6. SLA Mismatch Investigation

`SLA_Target_Hours` in the tickets table has 11 different values, but `SLA_Status` follows the 4 standard priority targets (Critical 4, High 8, Medium 24, Low 48 hours). Comparing status against the standard targets matched 9,997 of 10,000 rows. All SLA compliance in this project is measured against the standard targets.

Of the 9 flagged rows:

- 6 are explained by the different target column.
- 3 are rounding cases: the ticket was resolved a few seconds over the target (for example 24 hours 11 seconds), which displays as 24.00 hours after rounding. Verified in Excel with second-level timestamps.

`SLA_Status` was not changed. Open business question: which target is the official one, the standard target or the extended `SLA_Target_Hours`?

## 7. Findings

* SLA breach % is calculated on resolved tickets only (8,427).*

### 7.1 Overall SLA performance

61.6% of resolved tickets breached SLA (5,193 of 8,427).

### 7.2 By priority

| Priority | Resolved | Breach % | Median resolution (hrs) | Standard target (hrs) |
|---|---|---|---|---|
| Critical | 480 | 96.3 | 10.75 | 4 |
| High | 2,042 | 93.1 | 15.9 | 8 |
| Medium | 4,358 | 53.3 | 24.7 | 24 |
| Low | 1,547 | 32.8 | 41.5 | 48 |

Higher priority tickets are resolved faster, but the 4 and 8 hour targets are far below the typical resolution time, so breach is almost certain.

### 7.3 By product and issue type

- DataSync has the highest product breach rate (69.9%); other products are 56.9% to 64.3%.
- DataSync's Critical/High share (31.7%) is similar to other products (28.6% to 31.9%), so priority mix does not explain it.
- Inside DataSync, Integration (72.4%), Data Sync (71.5%) and API Error (71.2%) drive the breach. These issue types appear only in DataSync, so product and issue type cannot be separated. 15 of 19 issue types appear in only one product.
- Feature Request has the highest breach rate overall (83.5%, 689 tickets) and the lowest CSAT (3.04). It appears in CloudCRM, WorkHub and InsightPro, and is slow across every priority.

### 7.4 By team, experience, segment, channel and region

| Dimension | Breach % range | Conclusion |
|---|---|---|
| Team | 60.9 to 63.4 | No meaningful difference |
| Agent experience | 60.7 to 62.6 | No meaningful difference |
| Customer segment | 60.3 to 62.8 | No meaningful difference |
| Channel | 60.6 to 62.4 | No meaningful difference |
| Customer region | 60.7 to 63.7 | No meaningful difference |

Technical Support handles about 50% of resolved tickets, which is a workload finding, not a performance one. Individual agents range from 50.7% to 69.0% breach (129 to 183 tickets each), which likely reflects ticket mix and small samples.

### 7.5 Customer satisfaction

| SLA status | Tickets | Avg CSAT |
|---|---|---|
| Met | 3,201 | 4.16 |
| Breached | 5,143 | 2.95 |

The gap holds within every priority (Met about 4.1 to 4.2; Breached 2.42 for Critical to 3.22 for Low). Issue types with higher breach tend to have lower CSAT.

### 7.6 Trend (Jan to Sep 2026)

- Ticket volume is stable at about 1,000 to 1,180 per month.
- Monthly breach % ranges from 59.0% to 64.8%; monthly CSAT from 3.35 to 3.45. No clear improving or declining trend, so the problem is persistent.

### 7.7 Customer concentration

1,200 customers raised tickets, averaging 8.3 each. The top 10 raised 16 to 21 tickets with no sharp outlier.

## 8. Recommendations (preliminary)

1. **Review Critical and High SLA targets.** Typical resolution is about 11 and 16 hours against 4 and 8 hour targets. Either make targets realistic or add capacity for these tickets.
2. **Give Feature Request tickets a separate workflow or SLA target.** They breach heavily at every priority and have the lowest CSAT.
3. **Investigate DataSync technical issues** (Integration, Data Sync, API Error) for process or product causes.
4. **Do not focus on individual teams or agents.** The data shows no team, experience, segment, channel or region effect.
5. **Monitor breach rate and CSAT together.** Breached tickets score about 1.2 points lower.
6. **Resolve the official SLA target definition** (standard vs extended) so reporting is consistent.

## 9. Limitations

- Timestamps in the exported CSV are minute-level; the rounding cause of 3 SLA mismatches was verified in Excel with second-level data.
- 83 resolved tickets have no CSAT score and are excluded from CSAT averages.
- Critical tickets with SLA met are only 17, so that group's CSAT is not reliable.

## 10. Next Steps

- Build the 3-page Power BI dashboard (Executive Overview, Support Operations, Customer and Issue Insights).
- Finalize recommendations after the dashboard.
