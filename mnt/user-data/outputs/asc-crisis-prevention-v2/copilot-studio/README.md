# Copilot Studio — Adult Social Care Assistant

## Agent details

| Setting | Value |
|---|---|
| Agent name | Adult Social Care Assistant |
| Environment | Your Power Platform environment |
| Channels | Microsoft Teams, Copilot Studio web chat |

## Agent instructions

```
You are a digital worker supporting adult social care caseworkers 
in England under the Care Act 2014.

Your role is to save caseworkers time by helping them submit crisis 
assessments conversationally, query their caseload, and navigate 
safeguarding procedures.

CRITICAL RULES:
1. Never submit, send or escalate anything without explicit 
   confirmation from the caseworker
2. Always show a summary and ask for YES before any action is taken
3. Always include the safeguarding disclaimer on every risk assessment
4. Always err on the side of caution — if unsure, rate higher not lower
5. Use plain English — no jargon
6. Be concise — caseworkers are busy
7. The AI suggests the risk level — the caseworker always confirms 
   or overrides before submission
```

## Agent flows

### Flow 1 — Crisis Assessment Flow (AI analysis)

Analyses the case and returns an AI suggested risk rating.
Does NOT write to Dataverse — analysis only.

**Inputs:**

| Input | Type |
|---|---|
| ServiceUserName | Text |
| CaseRef | Text |
| VisitDescription | Text |
| CarerPresent | Boolean |
| Medication | Boolean |
| Safeguarding | Boolean |
| ADLScore | Number |
| Caseworker | Text |

**Steps:**
1. Create text with GPT (AI Builder) — same prompt as Power Automate flow
2. Compose RiskLevel — same split expression as Power Automate flow
3. Compose Summary
4. Compose Actions
5. Compose Escalation
6. Respond to the agent

**Outputs:**

| Output | Type |
|---|---|
| AISuggestedRisk | Text (RED/AMBER/GREEN) |
| Summary | Text |
| Actions | Text |
| Escalation | Text |

### Flow 2 — Submit Confirmed Assessment

Writes the final confirmed assessment to Dataverse after caseworker confirms or overrides.
Triggers the Crisis Assessment - AI Analysis Power Automate flow automatically.

**Inputs:**

| Input | Property | Type |
|---|---|---|
| ServiceUserName | text | Text |
| CaseRef | text_1 | Text |
| VisitDescription | text_2 | Text |
| Caseworker | text_7 | Text |
| AISuggestedRisk | text_8 | Text |
| CaseworkerRiskLevel | text_6 | Text |
| OverrideReason | text_9 | Text |
| ADLScore | number | Number |
| Medication | boolean | Boolean |
| Safeguarding | boolean_1 | Boolean |
| CarerPresent | boolean_2 | Boolean |

**Steps:**
1. Add a new row (Dataverse — Crisis Assessments)
2. Respond to the agent

**Integer conversion expressions:**

Risk Level (AI suggested):
```
if(equals(triggerBody()['text_8'],'RED'),1,if(equals(triggerBody()['text_8'],'AMBER'),2,3))
```

CWRisklevel (caseworker confirmed):
```
if(equals(triggerBody()['text_6'],'RED'),1,if(equals(triggerBody()['text_6'],'AMBER'),2,3))
```

**Outputs:**

| Output | Value |
|---|---|
| SubmissionStatus | Text — "Submitted" |

## Topics

### Submit Assessment

**Trigger phrases:**
- I just did a visit
- Just visited
- I need to submit an assessment
- New assessment
- I've been to see
- Just been with
- Home visit done

**Variables:**

| Variable | Type | Source |
|---|---|---|
| varVisitDescription | String | Question node |
| varServiceUserName | String | Question node |
| varCaseRef | String | Question node |
| varCarerPresent | String | Question node |
| varMedication | String | Question node |
| varSafeguarding | String | Question node |
| varADLScore | Number | Question node |
| varConfirmation | String | Question node |
| varCorrection | String | Question node |
| varAISuggestedRisk | String | Crisis Assessment Flow output |
| varSummary | String | Crisis Assessment Flow output |
| varActions | String | Crisis Assessment Flow output |
| varEscalation | String | Crisis Assessment Flow output |
| varCaseworkerResponse | String | Question node — AGREE or override |
| varFinalRiskLevel | String | Question node — caseworker's chosen level |
| varOverrideReason | String | Question node — reason for override |

**Conversation flow:**

```
Trigger
↓
Ask about visit → varVisitDescription
↓
Ask service user name → varServiceUserName
↓
Ask case reference → varCaseRef
↓
Ask carer present → varCarerPresent
↓
Ask medication compliant → varMedication
↓
Ask previous safeguarding → varSafeguarding
↓
Ask ADL score → varADLScore
↓
Show confirmation summary
↓
Ask YES/NO → varConfirmation
↓
Condition: varConfirmation = YES
↓
YES: Call Crisis Assessment Flow (AI analysis)
↓
Show AI suggestion with summary and actions
↓
Ask AGREE or override → varCaseworkerResponse
↓
Condition: varCaseworkerResponse contains AGREE
↓
YES (agrees):                    NO (overrides):
Set varFinalRiskLevel            Ask for override level → varFinalRiskLevel
= varAISuggestedRisk             Ask for reason → varOverrideReason
↓                                ↓
Call Submit Confirmed Assessment flow
↓
Final message with result and disclaimer
```

**Confirmation summary message:**
```
Before I submit, please confirm this is correct:

Service user: {varServiceUserName}
Case reference: {varCaseRef}
Visit notes: {varVisitDescription}
Carer present: {varCarerPresent}
Medication compliant: {varMedication}
Previous safeguarding: {varSafeguarding}
ADL Score: {varADLScore}/10

Type YES to submit or NO to make a change.
```

**AI suggestion message:**
```
Based on my analysis, I suggest this case is:

RISK LEVEL: {varAISuggestedRisk}

REASON:
{varSummary}

RECOMMENDED ACTIONS:
{varActions}

Do you agree with this rating?
Type AGREE to confirm, or type RED, AMBER or GREEN to override.
```

**Final result message:**
```
Assessment submitted for {varServiceUserName}.

FINAL RISK LEVEL: {varFinalRiskLevel}
SUBMITTED BY: {System.User.DisplayName}

The duty manager has been alerted if this is a RED case.
You will receive a Teams message once they respond.

IMPORTANT: This assessment is AI-generated decision support only.
It does not replace your professional judgement. You remain 
responsible for all decisions relating to this service user.
```

**NO branch (correction loop):**
```
Tell me what needs changing.
↓
Ask what needs changing → varCorrection
↓
Show updated summary with correction noted
↓
Ask YES to submit or NO to change again → varConfirmation
↓
Condition: varConfirmation = YES → proceed
         : varConfirmation = NO → loop back
```

## Deploying to Teams

1. In Copilot Studio go to Channels → Microsoft Teams
2. Click Add to Teams
3. Follow the prompts to publish to your organisation
4. Caseworkers find the agent by searching Adult Social Care Assistant in Teams

## Key technical notes

- Agent flow timeout must be set to PT2M in the trigger settings
- Delay in Crisis Assessment Flow set to 60 seconds
- Both flows must be Published not just saved as Draft
- If InvalidBindingInvokeAction error appears — delete and re-add the flow action node
- Output bindings go stale when flow is republished — always refresh bindings in topic after republishing
- varCarerPresent, varMedication, varSafeguarding collected as text (Yes/No) not boolean in the topic
