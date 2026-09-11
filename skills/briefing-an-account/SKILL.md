---
name: briefing-an-account
description: Assemble a pre-call account brief in Topo — roster, open sequences, inbox threads, recent activity, and ICP fit — then leave a next step if asked. Use when the user asks to brief an account, who we are talking to at a company, or what happened with a lead this month.
---

# Briefing an account

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

Default path is read-only. Join account, contacts, enrollments, threads, and
activities. Do not stop at `get_account`.

## Workflow

1. Identify the account. Prefer `get_account(name=…|domain=…|account_id=…)`.
   If the user named a person, `get_contact` returns `account_id` and
   `account_domain` — pass `account_id` straight into `get_account`.
   If unknown: `search(query, types=["account"])` → `fetch` →
   `metadata.account_id`. Keep `account_id`, counts, and
   `contacts[].contact_id` (roster, capped).
2. If the record is thin: `enrich_account(domain)` — research only, not a
   workspace write.
3. For the full roster, or when the cap is hit:
   `list_contacts(account_id)` — each row has `contact_id` and `account_id`.
4. For the people who matter (named, or `job_title_contains`):
   - `get_contact(contact_id)`
   - `list_sequences(contact_id)` — status + `sequence_id`
   - `list_threads(contact_id)` → `get_thread` for any real conversation
   - `list_activities(contact_id, since=…)` — sent / opened / replied / hot lead
5. Once: `get_organization_context`, then `qualify_lead(contact_id)` on the
   primary contact (and others if they asked).
6. Writes only if they asked: `create_todo(contact_id, due_date, title)` after
   `list_organization_members` for an assignee; `assign_owner(target=account,
   account_id, user_id)`; a decided outcome → `recording-outreach-outcomes`.

## Rules

- No CRM read (stage, ARR, opportunity). `update_crm_field` writes only.
- No buying-signal feed and no web-browse tool on this MCP.
- Inbox threads exist only if the lead was enrolled.
- `get_and_enrich_contact` spends credits and does not save. Confirm first.
- Present a brief: who, role, sequence status, last inbound quote, ICP
  verdict, recommended next step.
