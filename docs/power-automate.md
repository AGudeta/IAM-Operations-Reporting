# Power Automate Alert

## Purpose

Identify overdue access removal requests and email a summary for review.

The flow uses synthetic data and a fixed reporting date of October 28, 2026. The completion target is 3 calendar days.

## Data Source

- Connector: Excel Online (Business)
- Location: OneDrive for Business
- Workbook: IAM_Access_Requests.xlsx
- Excel table: IAMRequests
- DateTime Format: ISO 8601

The original working Power BI report and this flow read the same OneDrive workbook. The public Power BI copy uses a local sample workbook.

## Flow Structure

1. Manually trigger a flow.
2. List rows present in a table.
3. Filter array to identify overdue access removals.
4. Check whether the filtered array contains any rows.
5. If matches exist, create an HTML table and send an email.
6. Otherwise, skip the email.

## Filter Array

From: the `value` output of List rows present in a table.

Advanced mode expression:

```text
@and(
    equals(item()?['RequestType'], 'Access Removal'),
    or(
        equals(item()?['Status'], 'Pending'),
        equals(item()?['Status'], 'In Progress')
    ),
    less(
        ticks(item()?['SubmittedDate']),
        ticks('2026-10-25T00:00:00Z')
    )
)
```

The cutoff selects requests submitted before October 25. These requests are older than 3 calendar days as of October 28. Requests exactly 3 days old are excluded.

## Condition

Left expression:

```text
length(body('Filter_array'))
```

Operator: is greater than  
Right value: 0

The True branch contains the HTML table and email actions. The False branch is empty.

## Create HTML Table

From expression:

```text
body('Filter_array')
```

Columns: Custom

| Header | Value expression |
|---|---|
| Request ID | `item()?['RequestID']` |
| Department | `item()?['Department']` |
| Application | `item()?['Application']` |
| Status | `item()?['Status']` |
| Submitted Date | `formatDateTime(item()?['SubmittedDate'], 'yyyy-MM-dd')` |

Enter each value through the expression editor so it is evaluated rather than displayed as literal text.

## Send Email

- Connector: Office 365 Outlook
- Action: Send an email (V2)
- Recipient: the project owner's test mailbox
- Body: an explanation of the alert, the reporting date, and the Output from Create HTML table

The message identifies the records as fictional practice data.

## Expected Matches

| Request ID | Application | Age in calendar days |
|---|---|---:|
| IAM005 | Salesforce | 8 |
| IAM017 | Azure | 9 |
| IAM018 | Workday | 7 |

## Recreating the Flow

1. Upload the sample workbook to your own OneDrive for Business.
2. Confirm the data is formatted as an Excel table named IAMRequests.
3. Create an instant cloud flow with a manual trigger.
4. Add the actions described above.
5. Connect the Excel and Outlook actions using your own accounts.
6. Select your workbook and your own test email address.
7. Set the Excel action's DateTime Format to ISO 8601.
8. Save and run the flow manually.

Expressions reference the action name `Filter_array`. Adjust references if your action has a different name.

## Limitations

- The flow runs only when manually triggered.
- The reporting date and cutoff are fixed for reproducible results.
- The flow sends a summary each time matching records are found.
- The alert does not remove access or confirm whether access remains active.