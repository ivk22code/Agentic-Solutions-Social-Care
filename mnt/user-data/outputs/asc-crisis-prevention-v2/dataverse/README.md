# Dataverse — Crisis Assessments Table

## Table details

| Setting | Value |
|---|---|
| Table name | Crisis Assessments |
| Primary column | ServiceUser |
| Environment | Your Power Platform environment |

## Columns

| Column name | Type | Notes |
|---|---|---|
| ServiceUser | Single line of text | Service user full name |
| ServiceUserRef | Single line of text | Case reference number — must be unique per submission |
| Caseworker | Single line of text | Auto-populated from M365 login via User().FullName |
| Case Notes | Multiline text | Free-text caseworker notes |
| Call Transcript | Multiline text | Optional call or visit transcript |
| ADL Score | Whole number (0-10) | Activities of Daily Living score |
| Mental Health Flag | Yes/No | Mental health concern flagged |
| Carer Present | Yes/No | Active carer supporting service user |
| Previous Safeguarding | Yes/No | Prior safeguarding incidents on record |
| Social Isolation | Yes/No | Service user is socially isolated |
| Medication Compliant | Yes/No | Taking medication as prescribed |
| Recent Discharge | Yes/No | Discharged from hospital in last 30 days |
| Risk Level | Choice (1=Red, 2=Amber, 3=Green) | AI-suggested risk rating |
| CWRisklevel | Choice (1=Red, 2=Amber, 3=Green) | Caseworker's confirmed or overridden rating |
| Override Reason | Multiline text | Caseworker's reason if they overrode the AI suggestion |
| Supervisor Summary | Multiline text | AI-generated summary for duty manager |
| Recommended Actions | Multiline text | AI-generated next steps |
| Escalation Draft | Multiline text | AI-generated escalation message (RED only) |
| Status | Choice | New, In Review, Escalated, Closed |
| Submission Date | Date/Time | Auto-set on submission |

## Choice column values

### Risk Level and CWRisklevel
| Label | Value |
|---|---|
| Red | 1 |
| Amber | 2 |
| Green | 3 |

### Status
| Label | Value |
|---|---|
| New | 1 |
| In Review | 2 |
| Escalated | 3 |
| Closed | 4 |

## Key technical notes

- The `contoso_` prefix on column names will differ per organisation
- Risk Level stores the AI suggestion — CWRisklevel stores the caseworker's final decision
- Override Reason is populated when caseworker disagrees with the AI suggestion
- Both submission routes (Power Apps and Copilot Studio) write to the same table
- The same Power Automate flow triggers regardless of submission route
- Caseworker column uses plain text not a Lookup — auto-fills from User().FullName
- ServiceUserRef must be unique per submission to ensure correct LookUp in Power Apps Results screen
