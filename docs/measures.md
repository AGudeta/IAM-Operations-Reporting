# DAX Calculations

Table: `IAM_Requests`  
Reporting date: October 28, 2026  
Completion target: 3 calendar days

## Measures

Measures calculate summary results within the report's filter context.

### Total Requests

Counts request rows.

```dax
Total Requests =
COUNTROWS(IAM_Requests)
```

### Open Requests

Counts requests that are Pending or In Progress.

```dax
Open Requests =
CALCULATE(
    COUNTROWS(IAM_Requests),
    IAM_Requests[Status] IN {"Pending", "In Progress"}
)
```

### Completed Requests

Counts requests with Completed status.

```dax
Completed Requests =
CALCULATE(
    COUNTROWS(IAM_Requests),
    IAM_Requests[Status] = "Completed"
)
```

### Average Completion Days

Averages nonblank completion durations.

```dax
Average Completion Days =
AVERAGE(IAM_Requests[Completion Days])
```

### Completed Within Target

Counts completed requests with a completion duration of 3 calendar days or less.

```dax
Completed Within Target =
CALCULATE(
    COUNTROWS(IAM_Requests),
    IAM_Requests[Status] = "Completed",
    IAM_Requests[Completion Days] <= 3
)
```

### Within Target Percentage

Divides completed requests within the target by all completed requests. Returns blank when no completed requests exist. Formatted as a percentage in the report.

```dax
Within Target Percentage =
DIVIDE(
    [Completed Within Target],
    [Completed Requests]
)
```

### Overdue Access Removals

Counts open Access Removal requests older than 3 calendar days. Returns 0 when none match.

```dax
Overdue Access Removals =
COALESCE(
    CALCULATE(
        COUNTROWS(IAM_Requests),
        IAM_Requests[RequestType] = "Access Removal",
        IAM_Requests[Status] IN {"Pending", "In Progress"},
        IAM_Requests[Open Age Days] > 3
    ),
    0
)
```

## Calculated Columns

Calculated columns produce a value for each request row.

### Completion Days

Calculates calendar days between submission and completion. Returns blank when CompletedDate is blank.

```dax
Completion Days =
IF(
    ISBLANK(IAM_Requests[CompletedDate]),
    BLANK(),
    DATEDIFF(
        IAM_Requests[SubmittedDate],
        IAM_Requests[CompletedDate],
        DAY
    )
)
```

### Open Age Days

Calculates the age of Pending and In Progress requests using the fixed reporting date. Returns blank for other statuses.

```dax
Open Age Days =
IF(
    IAM_Requests[Status] IN {"Pending", "In Progress"},
    DATEDIFF(
        IAM_Requests[SubmittedDate],
        DATE(2026, 10, 28),
        DAY
    ),
    BLANK()
)
```

## Baseline Results

With all report selections cleared:

| Metric | Expected result |
|---|---:|
| Total Requests | 24 |
| Open Requests | 12 |
| Average Completion Days | 2.50 |
| Within Target Percentage | 75% |
| Overdue Access Removals | 3 |

## Assumptions

- The 3-day target is a project assumption.
- Durations use calendar days, including weekends and holidays.
- Open request ages use October 28, 2026, rather than today's date.
- The synthetic dataset has completion dates for all completed requests.