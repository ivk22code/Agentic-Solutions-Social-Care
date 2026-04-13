# Adult Social Care — Crisis Prevention Solution

An AI-powered crisis prevention tool for adult social care caseworkers, built entirely on Microsoft Power Platform with no code.

## What it does

Caseworkers submit case assessments either through a Power Apps form or by chatting with an AI agent in Microsoft Teams. The system automatically analyses the case, suggests a RED / AMBER / GREEN risk rating, asks the caseworker to confirm or override the suggestion, then alerts the duty manager and guides the caseworker through next steps — all within seconds of submission.

## Key design principle — AI suggests, caseworker decides

This solution follows Model 1 of human-AI collaboration. The AI never makes the final risk decision — it suggests a rating based on the case evidence, and the caseworker confirms or overrides it. Both the AI suggestion and the caseworker's final rating are stored separately in Dataverse, providing a full audit trail of any overrides.

This approach is aligned with the Care Act 2014 principle that assessments must be led by a qualified professional.

## Architecture

```
Caseworker (Power Apps or Copilot Studio agent)
        ↓
AI analysis (AI Builder via Power Automate or agent flow)
        ↓
Caseworker confirms or overrides risk rating
        ↓
Dataverse (Crisis Assessments table)
        ↓
Power Automate (approval + notifications)
        ↓
Duty Manager (Teams approval for RED cases)
        ↓
Power BI (team dashboard)
```

## Components

| Component | Purpose |
|---|---|
| Power Apps Canvas App | 5-screen form for caseworkers to submit assessments |
| Dataverse | Central data store for all assessments and AI outputs |
| Power Automate | Triggers AI analysis, sends alerts, handles approvals |
| AI Builder | Generates risk rating, summary, actions and escalation draft |
| Copilot Studio | Conversational AI agent for Teams |
| Power BI | Live risk dashboard for managers and the team |

## Risk levels

| Level | Meaning | Response |
|---|---|---|
| RED | Immediate risk to life or serious harm | Same-day response, duty manager approval required |
| AMBER | Elevated risk, urgent review needed | Review within 72 hours |
| GREEN | Monitored risk, no immediate danger | Routine follow-up |

## Pipeline stages

| Stage | Status | Component |
|---|---|---|
| 1 — Caseworker submits | Complete | Power Apps + Copilot Studio |
| 2 — AI analyses and suggests rating | Complete | AI Builder |
| 3 — Caseworker confirms or overrides | Complete | Copilot Studio agent |
| 4 — RED alert to duty manager | Complete | Power Automate + Teams |
| 5 — Manager approves or escalates | Complete | Power Automate Approvals |
| 6 — Result shown to caseworker | Complete | Power Apps + Teams |
| 7 — Case closure | Not started | Power Automate + Power Apps |

## Safeguarding notice

This tool is designed to support professional decision-making, not replace it. Every assessment includes a mandatory disclaimer. The caseworker and their manager retain full legal responsibility for all decisions under the Care Act 2014.

## Build order

1. Dataverse table
2. Power Apps form
3. Power Automate flow
4. Copilot Studio agent
5. Power BI dashboard

## Solution Tour

### 1. AI Agent Experience (Teams/Copilot Studio)
![Agent Instructions](./Screenshots/Agent%20Description.png)
*The agent is governed by strict rules to ensure caseworker oversight.*

### 2. Mobile Assessment Form (Power Apps)
![Canvas App](./Screenshots/Canvas%20App.png)
*A structured form for high-speed data entry in the field.*

### 3. Crisis Escalation (Microsoft Teams)
![Teams Approval](./Screenshots/Teams%20Approval.png)
*RED cases trigger immediate Adaptive Card approvals for managers.*



## Licence

Private — for internal use only.
