# Node reference

Eleven working nodes plus two canvas notes.

| # | Node | Type | What it does | Why it is built this way |
|---|---|---|---|---|
| 1 | Monday 07:00 | Schedule Trigger | Starts the run every Monday at 07:00 in the workflow timezone | Report is ready before the week's first meetings |
| 2 | Manual Run | Manual Trigger | Starts the run on demand | For testing and ad hoc reports mid-week |
| 3 | Config | Set | Holds every setting in one place | Nobody edits code to change a recipient or a tab name |
| 4 | Read Workstreams | Google Sheets | Reads all rows of the Workstreams tab | Sheet is the single source of truth and owners already work in it |
| 5 | Read Risks and Decisions | Google Sheets | Reads all rows of the log tab | One log for risks and decisions keeps ownership in one place |
| 6 | Read Calendar | Google Calendar | Reads the next 7 days of events | Shows leadership which checkpoints are coming |
| 7 | Consolidate | Code | Normalises the three sources and computes the roll-up and flags | All numbers are deterministic. Builds the request for the AI step |
| 8 | AI Executive Summary | HTTP Request | Calls the Claude Messages API for the headline, attention and watch lists | On error it continues, and the report falls back to rules |
| 9 | Build Report | Code | Merges numbers and summary, renders HTML, Markdown and Slack text | One place to change layout |
| 10 | Email for Review | Gmail | Sends the HTML report to `reviewRecipient` | Human review before leadership sees it |
| 11 | Slack Post (optional) | Slack | Posts a short version to a channel | Disabled by default |

## Design notes

**Execute Once on the read nodes.** n8n runs a node once per incoming item. The first Sheets node returns many rows, so without Execute Once the next node would run that many times. Each reader runs once, and **Consolidate** gathers the rows by node name.

**Always Output Data on the read nodes.** An empty tab should produce an empty section, not stop the run.

**Config referenced by name.** Nodes read settings with `$('Config').first().json`, which keeps the data flowing between nodes simple and lets you add nodes without re-wiring settings.

**AI request is built in Consolidate, not in the HTTP node.** The prompt and payload sit in version-controlled code next to the facts they describe, and the HTTP node only sends them.

**Failure behaviour.** The AI node is set to *continue on error*. **Build Report** then finds no usable AI response and uses the rule-based headline and lists. The footer of the report states which path was used.

## What Consolidate computes

| Output | Rule |
|---|---|
| Status counts | Green, Amber, Red, or "No status" for anything else |
| Overdue milestone | `Milestone Due` before today and `% Complete` not 100 |
| Stale update | `Last Updated` older than `staleAfterDays`, or missing |
| Needs attention (rule-based) | Red workstreams and overdue milestones, plus decisions needed within 7 days |
| Watch list (rule-based) | Amber workstreams that are not overdue |
| Upcoming milestones | Due within `milestoneHorizonDays` |
| Open risks | Not closed. Sorted High, Medium, Low |
| Open decisions | Not closed. Sorted by `Decision Needed By` |
| Calendar checkpoints | Events whose title matches `calendarKeywords`, maximum 10 |

## What the AI step receives and returns

Receives: a trimmed JSON of the computed facts (counts, red and overdue items, amber items, open decisions, high risks, upcoming milestones, stale owners).

Returns JSON only:

```json
{
  "headline": "max two sentences",
  "attention": ["max three items leadership must act on or decide this week"],
  "watch": ["max three items likely to become problems within two weeks"]
}
```

The system prompt tells the model to use only the supplied facts and not to invent numbers, names, causes or dates. Read the first few reports against the tracker to confirm it holds to that.
