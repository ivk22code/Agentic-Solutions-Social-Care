# Power Apps — Crisis Assessment Canvas App

## App details

| Setting | Value |
|---|---|
| App name | Crisis Assessment |
| Type | Canvas App |
| Format | Tablet |
| Data source | Dataverse — Crisis Assessments |

## Screens

### Screen 1 — ServiceUserDetails
Collects service user and caseworker information.

| Control | Type | Property | Value |
|---|---|---|---|
| txtServiceUser | Text input | Default | `""` |
| txtServiceUserRef | Text input | Default | `""` |
| txtCaseWorker | Text input | Default | `User().FullName` |
| txtCaseWorker | Text input | DisplayMode | `DisplayMode.Disabled` |
| txtSubmissionDate | Text input | Default | `Text(Now(), "dd/mm/yyyy hh:mm")` |
| txtSubmissionDate | Text input | DisplayMode | `DisplayMode.Disabled` |

### Screen 2 — RiskFlags
Collects structured risk assessment data via toggles and slider.

| Control | Type | Default | Dataverse column |
|---|---|---|---|
| SliderADL | Slider | Min=0, Max=10 | ADL Score |
| tglMentalHealth | Toggle | false | Mental Health Flag |
| tglCarerPresent | Toggle | false | Carer Present |
| tglSafeguarding | Toggle | false | Previous Safeguarding |
| tglIsolation | Toggle | false | Social Isolation |
| tglMedication | Toggle | false | Medication Compliant |
| tglDischarge | Toggle | false | Recent Discharge |

ADL Score label formula:
```
"ADL Score: " & SliderADL.Value & "/10"
```

### Screen 3 — CaseNotes
Collects free-text case notes and optional transcript.

| Control | Type | Mode |
|---|---|---|
| txtCaseNotes | Text input | TextMode.MultiLine |
| txtTranscript | Text input | TextMode.MultiLine |

### Screen 4 — ReviewSubmit
Shows summary and submits to Dataverse.

Submit button OnSelect formula:
```
Notify("Submitting assessment...", NotificationType.Information);
Set(
    varNewRecordID,
    Patch(
        'Crisis Assessments',
        Defaults('Crisis Assessments'),
        {
            ServiceUser: txtServiceUser.Text,
            ServiceUserRef: txtServiceUserRef.Text,
            CaseWorker: txtCaseWorker.Text,
            'Case Notes': txtCaseNotes.Text,
            'Call Transcript': txtTranscript.Text,
            'ADL Score': SliderADL.Value,
            'Mental Health Flag': tglMentalHealth.Value,
            'Carer Present': tglCarerPresent.Value,
            'Previous Safeguarding': tglSafeguarding.Value,
            'Medication Compliant': tglMedication.Value,
            'Recent Discharge': tglDischarge.Value,
            'Submission Date': Now()
        }
    )
);
Set(varSubmittedRef, txtServiceUserRef.Text);
Notify("Assessment submitted successfully", NotificationType.Success);
Navigate(Results, ScreenTransition.Fade)
```

### Screen 5 — Results
Displays AI-generated risk rating and recommended actions.

Key controls and formulas:

| Control | Property | Formula |
|---|---|---|
| rectRAG | Fill | `If(varNewRecordID.'Risk Level' = 'Risk Level (Crisis Assessments)'.Green, RGBA(0,128,0,1), If(varNewRecordID.'Risk Level' = 'Risk Level (Crisis Assessments)'.Amber, RGBA(255,165,0,1), RGBA(255,0,0,1)))` |
| rectRAG | Visible | `!IsBlank(varNewRecordID.'Supervisor Summary')` |
| lblRiskLevel | Text | `If(varNewRecordID.'Risk Level' = 'Risk Level (Crisis Assessments)'.Green, "GREEN — Routine follow up", If(varNewRecordID.'Risk Level' = 'Risk Level (Crisis Assessments)'.Amber, "AMBER — Urgent review needed", "RED — Immediate action required"))` |
| lblSummary | Text | `varNewRecordID.'Supervisor Summary'` |
| lblActions | Text | `varNewRecordID.'Recommended Actions'` |
| lblLoading | Visible | `IsBlank(varNewRecordID.'Supervisor Summary')` |
| tmrRefresh | Duration | `15000` |
| tmrRefresh | Repeat | `true` |
| tmrRefresh | Start | `IsBlank(varNewRecordID.'Supervisor Summary')` |
| tmrRefresh | OnTimerEnd | `Refresh('Crisis Assessments'); Set(varNewRecordID, LookUp(Sort('Crisis Assessments', 'Submission Date', SortOrder.Descending), ServiceUserRef = varSubmittedRef))` |

## Key variables

| Variable | Type | Purpose |
|---|---|---|
| varNewRecordID | Record | Stores the Dataverse row created on submission |
| varSubmittedRef | Text | Stores the ServiceUserRef at submission for timer refresh |

## Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| Column does not exist | Column name mismatch | Check exact Dataverse column names |
| Risk Level requires Integer | Choice column expects number | Map Red=1, Amber=2, Green=3 |
| Results screen shows wrong record | Timer finding old row | Use varSubmittedRef to lock in the correct reference |
| Banner always shows RED | Choice label capitalisation mismatch | Use `'Risk Level (Crisis Assessments)'.Green` not text comparison |
| Descending not valid | Wrong Power Apps syntax | Use `SortOrder.Descending` |

## Navigation

| From | To | Control |
|---|---|---|
| ServiceUserDetails | RiskFlags | Next button |
| RiskFlags | ServiceUserDetails | Back button |
| RiskFlags | CaseNotes | Next button |
| CaseNotes | RiskFlags | Back button |
| CaseNotes | ReviewSubmit | Next button |
| ReviewSubmit | CaseNotes | Back button |
| ReviewSubmit | Results | Submit button (after Patch) |
| Results | ServiceUserDetails | Start New Assessment button |

## Embedding in Teams

1. Go to Teams → select a channel → click + to add a tab
2. Search for Power Apps
3. Select Crisis Assessment app
4. Save
