# Power BI — Crisis Prevention Dashboard

## Dashboard details

| Setting | Value |
|---|---|
| Data source | Dataverse — Crisis Assessments |
| Refresh rate | Hourly |
| Audience | Managers, safeguarding leads, team leads |

## Pages

### Page 1 — Live risk overview
| Visual | Type | Data |
|---|---|---|
| Total open cases | Card | Count where Status != Closed |
| RED cases today | Card | Count where Risk Level = 1 and Submission Date = today |
| AMBER cases | Card | Count where Risk Level = 2 and Status != Closed |
| RAG breakdown | Donut chart | Count by Risk Level |
| Cases by caseworker | Bar chart | Count by Caseworker grouped by Risk Level |

### Page 2 — Case list
| Visual | Type | Notes |
|---|---|---|
| All open cases | Table | Columns: ServiceUser, Caseworker, Risk Level, CWRisklevel, Status, Submission Date |
| Conditional formatting | Risk Level column | Red=red, Amber=amber, Green=green |
| Override indicator | Calculated column | Flag where Risk Level != CWRisklevel |

### Page 3 — Trends
| Visual | Type | Data |
|---|---|---|
| Cases over time | Line chart | Count by Submission Date grouped by Risk Level |
| Average ADL score | Card | Average of ADL Score across open cases |
| Escalation rate | Card | Count Escalated / Count RED |
| Override rate | Card | Count where Risk Level != CWRisklevel / Total |

### Page 4 — AI vs Caseworker
| Visual | Type | Notes |
|---|---|---|
| Agreement rate | Card | % where Risk Level = CWRisklevel |
| Override breakdown | Table | Cases where AI and caseworker disagreed |
| Override reasons | Table | Override Reason column for all overridden cases |

## DAX measures

**Total open cases:**
```
Open Cases = CALCULATE(
    COUNTROWS('Crisis Assessments'),
    'Crisis Assessments'[Status] <> "Closed"
)
```

**RED cases today:**
```
RED Today = CALCULATE(
    COUNTROWS('Crisis Assessments'),
    'Crisis Assessments'[Risk Level] = 1,
    DATESBETWEEN('Crisis Assessments'[Submission Date], TODAY(), TODAY())
)
```

**Escalation rate:**
```
Escalation Rate = DIVIDE(
    CALCULATE(COUNTROWS('Crisis Assessments'), 'Crisis Assessments'[Status] = "Escalated"),
    CALCULATE(COUNTROWS('Crisis Assessments'), 'Crisis Assessments'[Risk Level] = 1),
    0
)
```

**Override rate:**
```
Override Rate = DIVIDE(
    CALCULATE(COUNTROWS('Crisis Assessments'), 'Crisis Assessments'[Risk Level] <> 'Crisis Assessments'[CWRisklevel]),
    COUNTROWS('Crisis Assessments'),
    0
)
```

## Setup steps

1. Open Power BI Desktop
2. Get data → Dataverse
3. Connect to your environment
4. Select Crisis Assessments table
5. Load data
6. Build visuals as above
7. Publish to Power BI service
8. Set scheduled refresh to hourly
9. Share dashboard with managers via Power BI workspace
