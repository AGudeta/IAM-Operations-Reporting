# Validation

Testing used synthetic IAM requests and a fixed reporting date of October 28, 2026.

## 1. Baseline Report

With all report selections cleared, confirmed:

| Metric | Expected and observed result |
|---|---:|
| Total Requests | 24 |
| Open Requests | 12 |
| Average Completion Days | 2.50 |
| Within Target Percentage | 75% |
| Overdue Access Removals | 3 |

## 2. Alert Contents

Ran the Power Automate flow and received an email containing:

| Request ID | Application | Age in calendar days |
|---|---|---:|
| IAM005 | Salesforce | 8 |
| IAM017 | Azure | 9 |
| IAM018 | Workday | 7 |

These matched the overdue access removals identified in Power BI.

## 3. Empty Result

Temporarily changed the flow's cutoff date to October 1, 2026, so no requests matched.

Confirmed:
- Filter array returned an empty array.
- The condition evaluated to False.
- The email action was skipped.

Restored the cutoff to October 25, 2026 afterward.

## 4. Shared Source Update

In the OneDrive workbook, temporarily changed IAM017:
- Status: Pending to Completed
- CompletedDate: blank to October 28, 2026

Refreshed the original OneDrive-connected Power BI report and ran the flow.

Confirmed:
- The overdue access removal count decreased from 3 to 2.
- The email included IAM005 and IAM018.
- IAM017 was excluded from the email.

Restored IAM017 to Pending with a blank completion date and confirmed the original results returned.

## 5. Public Report Copy

Changed the public Power BI copy to read the local sample workbook.

Reopened the saved report, refreshed it, and confirmed the five baseline metrics matched.

The public copy requires a local workbook path. The original shared-source test used the OneDrive-connected report.

## Scope

These checks validate the demonstrated scenarios using a small synthetic dataset. They do not establish production readiness or cover every possible data-quality issue.