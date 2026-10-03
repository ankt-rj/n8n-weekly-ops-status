# Data contract

The workflow reads two Google Sheet tabs by column header. Header names must match exactly. Extra columns are ignored.

## Tab: Workstreams

One row per workstream.

| Column | Required | Format | Notes |
|---|---|---|---|
| Workstream | Yes | Text | Rows without a name are skipped |
| Owner | Yes | Text | One named person |
| Status | Yes | `Green`, `Amber` or `Red` | Use a dropdown. Any other value shows as "No status" |
| % Complete | No | Number, `65` or `65%` | A value of `100` stops overdue flagging |
| This Week Progress | Yes | Text | One or two sentences. What changed, not what exists |
| Next Milestone | Yes | Text | The next dated deliverable |
| Milestone Due | Yes | Date, `YYYY-MM-DD` preferred | Drives overdue and upcoming flags |
| Blockers | No | Text | Anything that needs someone else to act |
| Last Updated | Yes | Date | Drives the stale update flag |

Suggested status definitions (agree these with leadership once):

- **Green**: on plan, no help needed
- **Amber**: at risk, owner has a recovery path
- **Red**: off plan, needs a leadership decision or intervention

## Tab: Risks and Decisions

One row per risk or decision.

| Column | Required | Format | Notes |
|---|---|---|---|
| ID | Yes | Text, e.g. R-01, D-01 | Stable reference used in meetings |
| Type | Yes | `Risk` or `Decision` | Anything other than `Decision` is treated as a risk |
| Description | Yes | Text | Rows without a description are skipped |
| Owner | Yes | Text | |
| Severity | Risks only | `High`, `Medium`, `Low` | Used for sorting |
| Decision Needed By | Decisions only | Date | Sorts decisions by urgency. Within 7 days counts as "due this week" |
| Status | Yes | `Open` or `Closed` | `Closed` rows are excluded |
| Mitigation / Options | Recommended | Text | For decisions, list the options and the cost of each |

## Calendar

No setup beyond access. The workflow reads the next 7 days and keeps events whose title matches the `calendarKeywords` pattern in the Config node. Name leadership checkpoints consistently ("Steering committee", "Board pack review") and they appear automatically.
