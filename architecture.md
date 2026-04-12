# Solution Architecture & Build Guide

## Build order

Always build in this sequence — each layer depends on the previous one.

### Stage 1 — Dataverse
Set up the Crisis Assessments table with all columns before building anything else.

See: `/dataverse/README.md`

### Stage 2 — Power Apps
Build the Canvas App connected to Dataverse. Test that a submission creates a row in Dataverse before moving on.

See: `/power-apps/README.md`

### Stage 3 — Power Automate
Build the AI analysis flow triggered by Dataverse row creation. Test that the flow fires, AI Builder returns a result, and all fields are written back to Dataverse.

See: `/power-automate/README.md`

### Stage 4 — Copilot Studio
Build the agent and both agent flows. The Submit Confirmed Assessment flow writes to Dataverse which triggers the Power Automate flow automatically.

See: `/copilot-studio/README.md`

### Stage 5 — Power BI
Connect to Dataverse and build the dashboard once real data exists.

See: `/power-bi/README.md`

## How the two submission routes connect

```
Power Apps submission          Copilot Studio submission
        ↓                               ↓
Patch() writes row             Submit Confirmed Assessment
to Dataverse                   flow writes row to Dataverse
        ↓                               ↓
        └───────────────┬───────────────┘
                        ↓
        Crisis Assessment - AI Analysis
        Power Automate flow triggers
                        ↓
              AI Builder analysis
                        ↓
         Results written back to Dataverse
                        ↓
              RED? → Teams approval
              AMBER/GREEN? → Teams notification
```

Both routes use the same Power Automate flow — no duplication.

## Human-AI collaboration model

This solution uses Model 1 — AI suggests, caseworker decides:

```
AI analyses case
↓
AI suggests RED/AMBER/GREEN with reasoning
↓
Caseworker reviews suggestion
↓
Caseworker types AGREE or overrides with RED/AMBER/GREEN
↓
If override — caseworker gives reason
↓
Final rating submitted with both AI suggestion and caseworker decision
↓
Both stored separately in Dataverse for audit trail
```

This is aligned with Care Act 2014 principles — the professional always makes the final decision.

## Key technical decisions

| Decision | Rationale |
|---|---|
| Canvas App not Model-driven | Full layout control for caseworker form |
| Individual controls not Form control | Patch formula gives more control over submission |
| Caseworker as Text not Lookup | Avoids related table, auto-fills from User().FullName |
| AI Builder not Anthropic API | No external API key, stays within M365 tenant |
| Single Power Automate flow | Both submission routes trigger the same flow via Dataverse |
| Two agent flows in Copilot Studio | Flow 1 analyses, Flow 2 submits — keeps concerns separated |
| varNewRecordID captures row at submission | Avoids LookUp issues on Results screen |
| varSubmittedRef locks the reference | Prevents timer from finding wrong record |
| Choice labels are Red/Amber/Green not RED/AMBER/GREEN | Must match exactly in Power Apps comparisons |
| 60 second delay in agent flow | Gives Power Automate time to complete AI analysis |

## Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| Column does not exist | Column name mismatch | Check exact Dataverse column names |
| Risk Level requires Integer | Choice column expects number | Map Red=1, Amber=2, Green=3 |
| Split parameter is Null | AI Builder output path wrong | Use `body/responsev2/predictionOutput/text` |
| Descending not valid | Wrong Power Apps syntax | Use `SortOrder.Descending` |
| Output binding not found | Agent flow republished | Republish flow then refresh binding in topic |
| Flow timeout | AI analysis takes too long | Set flow timeout to PT2M, delay to 60 seconds |
| Results show wrong record | Timer finding old row | Use varSubmittedRef to lock reference |
| Banner always shows RED | Choice label case mismatch | Use `'Risk Level (Crisis Assessments)'.Green` |
| InvalidBindingInvokeAction | Flow binding stale | Delete and re-add flow action node in topic |
| text_X property not found | Wrong triggerBody reference | Check Code view of flow trigger for exact property names |
| GREEN case going to TRUE branch | Condition evaluating wrong output | Verify condition left side references Composing_Risk_Level_Output |
| Approval firing for all cases | Approval outside condition | Move approval inside TRUE branch of RED condition |
| FALSE branch wrong message | Escalation text in non-RED message | Update FALSE branch message to calm AMBER/GREEN notification |

## Pipeline stages

| Stage | Status | Component |
|---|---|---|
| 1 — Caseworker submits | Complete | Power Apps + Copilot Studio |
| 2 — AI analyses and suggests | Complete | AI Builder |
| 3 — Caseworker confirms or overrides | Complete | Copilot Studio agent |
| 4 — RED alert to duty manager | Complete | Power Automate + Teams |
| 5 — Manager approves or escalates | Complete | Power Automate Approvals |
| 6 — Result shown to caseworker | Complete | Power Apps + Teams |
| 7 — Case closure | Not started | Power Automate + Power Apps |

## Remaining work

- Case closure workflow (Stage 7)
- Power BI dashboard
- Caseworker email capture for Teams notifications
- 2-hour chase flow for unreviewed RED cases
- Additional Copilot Studio topics (query caseload, draft communications)
