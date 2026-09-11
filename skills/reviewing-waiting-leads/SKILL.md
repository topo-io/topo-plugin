---
name: reviewing-waiting-leads
description: Review leads waiting for approval (FOUND) on a sequence — qualify against the ICP, approve fits so outreach starts, refuse the rest without blocking other sequences. Use when the user asks to review new leads, approve a backlog, or refuse people who are not a fit.
---

# Reviewing waiting leads

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

FOUND means enrolled and waiting. It is not "missing." Refusing a FOUND lead
does **not** create an exclusion. Unenrolling does.

## Workflow

1. `list_sequence_templates(query=…, status=ACTIVE)` →
   `sequence_template_id`.
2. `get_sequence_template(sequence_template_id)` — read
   `settings.lead_approval_mode`, `sender_identities`, and
   `enrollments.by_status`. Prefer `sequence_template_id` (same value as
   `id`). If mode is
   `AUTOPILOT`, say so: there should be little or no FOUND backlog.
3. `list_sequences(sequence_template_id, status=FOUND)`. Page with `cursor`.
   Each row has `contact_id` and `sequence_id`.
4. Once per session: `get_organization_context` (ICP).
5. For each lead (or the named subset):
   - `get_contact(contact_id)`
   - `get_and_enrich_contact(contact_id)` only if email, title, or domain is
     missing **and** the user wants a real look (spends credits; not saved)
   - `qualify_lead(contact_id)` — add `criteria` when they named extra rules
6. Present a short approve / refuse list (name, title, company, score, reason).
7. Writes:
   - Fits: `approve_leads(sequence_template_id)`. Omit `contact_ids` when they
     said "all." Pass `contact_ids` only for a small named subset. Never paste
     a large id list — the server pages FOUND leads itself.
   - Not a fit for **this** sequence: `refuse_leads(sequence_template_id,
     contact_ids=[…])`.
   - Truly do-not-contact (competitor, customer, unsubscribe): do not refuse.
     Use `suppressing-or-releasing-targets`.
8. If the work arrived as tasks instead of a template backlog:
   `list_tasks(task_type=NEW_LEAD_REVIEW, status=PENDING)`. Each row now
   includes `sequence_template_id` and `contact_id`. Completing the task
   **is** the write:
   `resolve_task(task_id, outcome=completed,
   new_lead_review_action=APPROVE|REFUSE)`. Do not also call
   `approve_leads` for the same person. Use the row's
   `sequence_template_id` only when they asked to approve the rest of
   that template's FOUND pile in one call.

## Rules

- Cannot create or edit the lead search that produced these leads.
- `qualify_lead` does not enroll or approve.
- Enrichment is ephemeral unless you then `update_record` / `upsert_contact`.
- Approving starts real email or LinkedIn. Say that before the call.
