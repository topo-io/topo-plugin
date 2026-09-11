---
name: diagnosing-sequence-health
description: Diagnose why a Topo sequence is not sending or underperforming using funnel counts and operational settings, then apply only the ops fixes MCP allows. Use when the user asks why a sequence is quiet, how it performed this month, to pause it, attach a sender, or switch copilot vs autopilot.
---

# Diagnosing sequence health

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

"It's not sending" has several causes. Check operations before blaming copy.
Weak copy after sends have happened is `creating-a-sequence`
(`update_sequence_template_steps`), not an ops setting.

## Workflow

1. `list_sequence_templates(query=…)` →
   `get_sequence_template(sequence_template_id)`. Prefer
   `sequence_template_id` from the detail (same value as `id`). Read `status`,
   `sender_identities` (`sender_identity_id`),
   `settings.lead_approval_mode`, `settings.daily_leads_target`, timezone /
   schedule, and `enrollments.by_status`.
2. `get_sequence_metrics(sequence_template_id, days=30)` (omit the id for
   the whole portfolio). These are **windowed counts**, not rates or
   lifetime totals. Compute a rate yourself and name the denominator:
   per-recipient reply rate = `unique_replies / unique_sent`.
3. Split the backlog with `list_sequences(sequence_template_id, status=…)`:
   - `FOUND` — copilot pile; hand off to `reviewing-waiting-leads`
   - `ACTIVE` — should be sending
   - `WAITING_FOR_USER` — humans (inbox / tasks), not the sender
   - `FAILED` — sample `get_sequence` for the error
4. `list_sender_identities` vs identities on the template. Empty senders →
   cannot send. Identities use `id` as `attach_identity_ids`.
5. If FOUND is high: `list_tasks(task_type=NEW_LEAD_REVIEW, status=PENDING)`.
   Each row includes `sequence_template_id` — keep the ones for this
   template (MCP `list_tasks` has no template query param).
6. Optional: `list_playbooks` / `list_playbook_runs(status=FAILED)` if they
   believe a playbook feeds this sequence.
7. Allowed ops writes, only if they asked:
   - `set_sequence_template_status(ACTIVE|INACTIVE|ARCHIVED)` — `ACTIVE`
     re-validates readiness
   - `update_sequence_template_settings` — daily target, schedule,
     `lead_approval_mode`, tracking, channel failure behavior
   - `set_sequence_template_senders` — pass **both** `attach_identity_ids`
     and `detach_identity_ids` (use `[]` for the unused side)

## Diagnostic order

Stop at the first failing gate.

1. Template not `ACTIVE`, or no sender identity.
2. Many `FOUND`, few `ACTIVE` — approval mode, not copy.
3. `unique_found` >> `unique_sent` — daily cap, schedule, or missing contact
   data (`settings.missing_contact_data_behavior`).
4. `WAITING_FOR_USER` — replies sitting in the inbox.
5. Sends happened, replies weak — copy or audience; edit steps with
   `creating-a-sequence` or send them to the web app.

Under 100 `unique_sent` in the window: refuse a performance verdict.

## Rules

- Cannot connect OAuth mailboxes. Attach identities that already exist.
- Cannot invent bounce / unsubscribe / spam rates. They are not in
  `get_sequence_metrics`.
- Do not silently `approve_leads`. Switch to `reviewing-waiting-leads`.
- Open rate is directional only (privacy proxies, tracking off).
