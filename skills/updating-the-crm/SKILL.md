---
name: updating-the-crm
description: Read and write the CRM connected to Topo (HubSpot, Salesforce, or Pipedrive) — check a contact's or account's CRM record and owner, search deals, create a deal, log a note, or update CRM properties — from Topo contacts and accounts. Use when the user asks what the CRM says about a lead or company, which deals closed, to log a call or meeting in the CRM, open a deal after a booked meeting, or push a field to the CRM.
---

# Updating the CRM

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

Every tool here works the same way for HubSpot, Salesforce, and Pipedrive:
Topo maps `account` to the CRM's company object and `contact` to its contact
(a lead on Salesforce). Records are addressed by Topo `contact_id` and
`account_id`, never by CRM ids.

A CRM write leaves Topo: it lands on the record the whole revenue team works
in, shows in its history, can fire the CRM's own workflows, and Topo cannot
undo it. Name the CRM, the record, and every value before the user confirms.

## Read

- `get_crm_record(object_type=contact|account, contact_id | account_id,
  properties=[…])` — whether the record exists in the CRM, its link, its owner
  (with the matching Topo user), the requested properties, and for an account
  its open and won deal counts. Pass `properties=[]` to only check the link.
- `search_deals(outcome=won|lost|open|any, account_id | account_domain,
  closed_within_days, min_amount, sort, extra_properties=[…])` — live deals,
  never stored in Topo. `list_deal_properties(search=…)` first when the user
  names a custom field.

## Write

| The user wants | Call |
|---|---|
| Log what happened (call, meeting, reply summary) | `log_crm_activity(contact_id and/or account_id, title, body)` — a note, or a completed task on Salesforce; every call adds one |
| Open a deal (meeting booked, opportunity) | `create_crm_deal(account_id, name, contact_id?, amount?, close_date?, stage_id?, owner_user_id?, properties)` — an open deal with the same name on that company is reused |
| Set properties | `get_crm_record` for the current values, then `update_crm_field(object_type, contact_id or account_id, fields=[{property, value}])` |

- Batch every property of one update into one `fields` list. Values are sent
  as text and the CRM coerces them: `3` for a number, `true` / `false` for a
  checkbox, an exact picklist value. An empty value clears the field.
- Quote the previous values in your reply before overwriting, so the user can
  restore them.
- Never guess a property's API name. Read it from `get_crm_record` or
  `list_deal_properties`, or ask.
- Pipeline and stage ids come from the CRM; leave them out to get the default
  pipeline's first open stage.
- A value that only serves Topo personalization belongs in `update_record` /
  `update_account_record`, not in the CRM.

## Failures that are configuration

- No CRM connected: say so; connecting one is done in the Topo web app.
- "Not synced to the CRM yet": the contact or account has no CRM record.
  Report it; do not retry or create one by hand.
- A property the CRM marks read-only, or a value its picklist refuses: report
  the CRM's message.

## Rules

- Confirm before `log_crm_activity`, `create_crm_deal`, and
  `update_crm_field`.
- Recording a booked meeting in Topo is still `stop_sequence(reason=WIN)`
  (`recording-outreach-outcomes`); the CRM deal or note is in addition.
- Never present a deal amount, stage, or owner you did not read from the CRM.
