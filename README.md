# Customer Support & Service Performance Analytics

End-to-end analysis of 10,000 customer support tickets (Jan to Sep 2026) for a SaaS company, using Excel, Python, MySQL and Power BI. The goal is to find where SLA performance is failing, how large the problem is, and what is driving customer dissatisfaction.

Status: Data cleaning and SQL analysis complete. Power BI dashboard in progress.

## Business Problem
Management wants to know:
- Are tickets being resolved within SLA?
- Which priorities, products and issue types have the most breaches?
- Which teams or agents are struggling?
- What is associated with low customer satisfaction (CSAT)?

## Tools Used
| Stage | Tool | Purpose |
|---|---|---|
| Profiling and initial cleaning | Excel | Duplicates, blanks, inconsistent labels, SLA logic checks |
| Cleaning and validation | Python (Pandas) | Loading, date conversion, null checks, final cleaned file |
| Analysis | MySQL | Joins, CTEs, window functions (LAG, RANK), business queries |
| Dashboard | Power BI | 3-page interactive dashboard (in progress) |

## Dataset
5 related tables: tickets (10,000 rows), customers (1,200), agents (55), products (6), sla_rules (4).

## Data Cleaning Summary
| Issue | Rows | Action |
|---|---|---|
| Duplicate Ticket_IDs | 25 | Removed (10,025 to 10,000 rows) |
| Priority "HIGH" vs "High" | 8 | Standardized |
| Resolved tickets with missing CSAT | 83 | Flagged, not filled |
| SLA status mismatch | 9 | Flagged. 6 due to a different target column, 3 due to rounding (seconds over target) |
| Blank Channel | 35 | Filled with "Unknown" and flagged |
| Blank Resolved_Date / Resolution_Time | 1,573 | Kept blank (Open/Pending tickets) |

Row counts and flag counts (83, 9, 35) match across Excel, Python and SQL.

Note: SLA_Status follows the standard priority targets (Critical 4, High 8, Medium 24, Low 48 hours), so SLA compliance is measured against those.

## Key Findings
1. 61.6% of resolved tickets breached SLA (5,193 of 8,427).
2. Critical and High tickets breach the most: Critical 96.3%, High 93.1%, Medium 53.3%, Low 32.8%. The typical Critical ticket takes about 11 hours against a 4-hour target, so the targets look tight relative to actual speed.
3. DataSync has the highest product breach rate (69.9%). Priority mix does not explain it. It is driven by DataSync-only technical issues (Integration, Data Sync, API Error at about 71-72%).
4. Feature Request has the highest breach rate (83.5%) and lowest CSAT (3.04), and it is slow across every priority.
5. Late tickets have lower satisfaction: average CSAT is 4.16 for SLA-met tickets vs 2.95 for breached tickets. The gap holds within every priority (about 1.0 to 1.7 points).
6. Team, agent experience, customer segment, channel and region show no meaningful difference in breach rate (all within about 3 percentage points).
7. Volume and performance are stable from Jan to Sep 2026: about 1,000-1,180 tickets per month, breach 59-65%, CSAT 3.35-3.45. The problem is persistent, not new.

## Recommendations (preliminary)
- Review the Critical and High SLA targets (4 and 8 hours), or add capacity for those tickets.
- Give Feature Request tickets a separate workflow or SLA target.
- Look into DataSync's technical issue types (Integration, Data Sync, API Error).
- Track breach rate and CSAT together, since they move together.

## Repository Structure
```
Customer-Support-Analytics/
├── data/            cleaned CSV files
├── excel/           profiling and cleaning workbook
├── python/          cleaning notebook
├── sql/             data_analysis.sql, advanced_analysis.sql
├── documentation/   project report
├── powerbi/         dashboard (coming soon)
└── README.md
```

## Limitations
- Python EDA and charts were skipped; analysis was done in SQL and will be visualized in Power BI.
- CSV timestamps are minute-level. The rounding root cause of 3 SLA mismatches was verified in Excel with second-level data.
