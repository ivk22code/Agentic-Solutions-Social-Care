# Power Automate — Crisis Assessment AI Analysis Flow

## Flow details

| Setting | Value |
|---|---|
| Flow name | Crisis Assessment - AI Analysis |
| Type | Automated cloud flow |
| Trigger | When a row is added (Dataverse) |
| Table | Crisis Assessments |
| Scope | Organisation |

## Important note

This flow is triggered by BOTH Power Apps and Copilot Studio submissions. Both routes write to the same Dataverse table, so this single flow handles all assessments regardless of submission route.

## Flow structure

```
Trigger — When a row is added
↓
Run a prompt (AI Builder)
↓
Compose RiskLevel
↓
Compose Summary
↓
Compose Actions
↓
Compose Escalation
↓
Update a row (write AI results back to Dataverse)
↓
Condition — Risk Level = 1 (RED)?
↓
TRUE branch                    FALSE branch
↓                              ↓
Start and wait for approval    Update a row 2 (Status = In Review)
↓                              ↓
Condition — Approved?          Post Teams message to caseworker
↓                              (AMBER/GREEN notification)
TRUE          FALSE
↓             ↓
Update        Update
row 1         row 2
(In Review)   (Escalated)
↓             ↓
Post Teams    Post Teams
message       message
(Approved)    (Escalated)
```

## Step details

### 1. Trigger — When a row is added
Fires automatically when a caseworker submits via Power Apps or Copilot Studio agent.

- **Change type:** Added
- **Table name:** Crisis Assessments
- **Scope:** Organisation

### 2. Run a prompt (AI Builder)
Sends all case data to AI Builder for analysis.

**Prompt:**
```
You are an expert adult social care risk assessor in England 
working under the Care Act 2014.

Analyse the following case information and return a structured 
risk assessment.

CASE INFORMATION:
Case notes: [dynamic content — Case Notes]
ADL Score: [dynamic content — ADL Score] out of 10
Mental health concern: [dynamic content — Mental Health Flag]
Carer present: [dynamic content — Carer Present]
Previous safeguarding: [dynamic content — Previous Safeguarding]
Socially isolated: [dynamic content — Social Isolation]
Medication compliant: [dynamic content — Medication Compliant]
Recent hospital discharge: [dynamic content — Recent Discharge]

RISK LEVELS:
- RED: Immediate risk to life or serious harm. Clear evidence of 
  active suicidal ideation, complete self-neglect, no food AND 
  no medication AND no carer, active abuse. Do NOT rate RED 
  without clear evidence of immediate harm.
- AMBER: Elevated risk needing review within 72 hours. 
  Deteriorating mental health, carer concerns, recent discharge 
  with gaps in support, medication non-compliance with other factors.
- GREEN: No immediate risk. Routine follow-up appropriate. 
  Most routine welfare visits should be GREEN unless clear 
  evidence of risk exists.

IMPORTANT: If ADL score is 7 or above AND carer is present AND 
medication is compliant AND no previous safeguarding, this should 
be GREEN unless case notes contain clear evidence of immediate harm.

Return exactly this format:
RISK LEVEL: [RED, AMBER, or GREEN]
SUMMARY: [2-3 sentences for a duty manager]
ACTIONS: [3 bullet points of recommended next steps]
ESCALATION: [Pre-written escalation message if RED, otherwise NONE]
```

**AI output path:**
```
body/responsev2/predictionOutput/text
```

### 3. Compose steps (x4)

**Compose RiskLevel:**
```
trim(split(split(outputs('Run_a_prompt')?['body/responsev2/predictionOutput/text'],'RISK LEVEL:')[1],'SUMMARY:')[0])
```

**Compose Summary:**
```
trim(split(split(outputs('Run_a_prompt')?['body/responsev2/predictionOutput/text'],'SUMMARY:')[1],'ACTIONS:')[0])
```

**Compose Actions:**
```
trim(split(split(outputs('Run_a_prompt')?['body/responsev2/predictionOutput/text'],'ACTIONS:')[1],'ESCALATION:')[0])
```

**Compose Escalation:**
```
trim(split(outputs('Run_a_prompt')?['body/responsev2/predictionOutput/text'],'ESCALATION:')[1])
```

### 4. Update a row
Writes AI results back to the Crisis Assessments record.

| Dataverse column | Value |
|---|---|
| Risk Level | Compose RiskLevel output (integer: Red=1, Amber=2, Green=3) |
| Supervisor Summary | Compose Summary output |
| Recommended Actions | Compose Actions output |
| Escalation Draft | Compose Escalation output |
| Status | In Review |

### 5. Condition — Is this RED?
```
outputs('Composing_Risk_Level_Output') is equal to 1
```

### 6. TRUE branch — RED cases

**Start and wait for an approval:**
- Approval type: Approve/Reject - First to respond
- Assigned to: Duty manager email
- Title: `URGENT - RED Risk Assessment: [ServiceUser]`
- Details: Full case summary including AI outputs

**Condition — Was it approved?**
- Left: Outcome from approval
- Operator: is equal to
- Right: Approve

**If Approved:**
- Update a row 1 → Status = In Review
- Post Teams message to caseworker:
```
Update on [ServiceUser]
Your RED assessment has been APPROVED by the duty manager.
Please proceed with the recommended actions immediately.
Approved by: [responder name]
Time: [utcNow()]
```

**If Rejected/Escalated:**
- Update a row 2 → Status = Escalated
- Post Teams message to caseworker:
```
Update on [ServiceUser]
Your RED assessment has been ESCALATED to senior management.
Please continue with your immediate safeguarding actions.
Escalated by: [responder name]
Time: [utcNow()]
```

### 7. FALSE branch — AMBER/GREEN cases

- Update a row → Status = In Review
- Post Teams message to caseworker with summary and actions
- Message tone: calm, no escalation language

## Key technical notes

- Compose steps must run vertically not in parallel branches
- AI Builder output path is `body/responsev2/predictionOutput/text`
- Choice column Risk Level requires integers not text (Red=1, Amber=2, Green=3)
- Condition checks integer 1 not string "RED"
- FALSE branch message must not use escalation language for AMBER/GREEN cases
- Both Power Apps and Copilot Studio submissions trigger this same flow
