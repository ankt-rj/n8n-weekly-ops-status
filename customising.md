# Customising

## Change the report layout

All layout lives in the **Build Report** node. Markdown and HTML are built separately in the same script. Edit the strings, save, and run *Manual Run*. Run `node tests/run_local.js` afterwards if you work from a copy of the repo.

## Add a data source

Pattern: add the reader node, set **Execute Once** and **Always Output Data** on it, chain it before **Consolidate**, then read it in the code with `rows('Node Name')`.

Common additions:

- **Jira, Asana, Linear, Notion, Airtable**: replace or supplement the Workstreams tab. Map their fields to the columns in the data contract inside Consolidate and nothing downstream changes.
- **Finance**: a monthly budget sheet read into a `finance` block and shown as one extra table.
- **Hiring**: open roles and offers from your applicant tracking tool as a workstream row.

## Keep a weekly history

Add a Google Sheets *Append* node after **Build Report** writing one row per week: date, green, amber, red, overdue milestones, open decisions, high risks. After a quarter you can chart the trend and show leadership whether the operation is stabilising. Map the values from `$('Consolidate').first().json.facts.counts`.

## Escalate reds by direct message

Add an IF node on `{{ $('Consolidate').first().json.facts.counts.red }}` greater than 0, then a Slack or Gmail node to the executive sponsor with the red workstreams' blockers.

## Change the tone or length of the AI summary

Edit the `system` string in **Consolidate**. Keep the instruction to use only supplied facts and keep the JSON output shape, because **Build Report** parses it.

## Use a different model

Change `model` in **Config**. A smaller, faster model is usually enough for a summary of this size.

## Stop sending data to an AI provider

Disable **AI Executive Summary**. The report keeps working with a rule-based headline and lists.

## Send to different audiences

Duplicate the workflow, point the copy at a different tab or filter by workstream tag, and change `reviewRecipient`. Keep one workflow per audience rather than building conditional branches.
