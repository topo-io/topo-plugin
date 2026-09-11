---
name: recording-outreach-outcomes
description: Apply a known outreach outcome in Topo — meeting booked, won, not interested, unsubscribe, wrong person — using stop_sequence, unenroll, or refuse_leads. Use when the user already knows the result from a call, Slack, or CRM and wants someone taken off a sequence without opening the inbox.
---

# Recording outreach outcomes

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first. The user's
words decide the write. "Take them off the sequence" is three different tools.

## Workflow

1. Resolve the lead. Prefer `get_contact(email=…)` or `linkedin_url`. If only
   a name is known: `search(query, types=["lead"])` → `fetch(id)` →
   `metadata.contact_id`. Never pass a `lead:<uuid>` typed id to another tool.
   `get_contact` now also returns `account_id`, `account_domain`, and
   `active_sequence_ids`.
2. Need enrollment ids? Prefer `get_contact.active_sequence_ids`. If this
   came from a thread, `get_thread.sequence_id` is the one enrollment.
   Call `list_sequences(contact_id)` only when you need status / template
   on each row. Ignore already-terminal statuses (`SUCCEEDED`, `FAILED`,
   `COMPLETED`, `STOPPED_EARLY`, `REFUSED`).
3. If they named a sequence: `list_sequence_templates(query=…)` or
   `get_sequence_template(name=…)` and keep `sequence_template_id` / `id`.
   Pick the matching live enrollment.
4. Optional, if they also care about inbox metrics: `list_threads(contact_id)`
   → `get_thread` → inbound `message_id` (or `last_message_id` when inbound)
   → `categorize_reply`. This does **not** replace step 5.
5. Write exactly one path:

   - **Booked / won / closed-won interest** →
     `stop_sequence(sequence_id, reason=WIN, scope=CONTACT)`. Use
     `scope=ACCOUNT` only if they said stop the whole company (domain
     exclusion).
   - **Hard stop this person everywhere** (unsubscribe, never again, left
     company) → `unenroll(contact_id)`. Pass `sequence_template_id` only when
     they named one sequence.
   - **Lost this sequence only** →
     `stop_sequence(sequence_id, reason=LOSE, scope=CONTACT)`.
   - **Still FOUND, just not a fit for this sequence** → stop. Hand off to
     `reviewing-waiting-leads` (`refuse_leads`). Do not unenroll.
6. Optional follow-through they asked for: `create_todo(contact_id, due_date,
   title)` with `whoami.user_id` when it is for them;
   `list_organization_members(query=name)` → `user_id` for a teammate;
   `assign_owner` with `account_id` from `get_contact`; `update_crm_field`
   only for a named CRM property. Domain exclude: `add_exclusions` with
   `get_contact.account_domain`.

## Rules

- Never call `unenroll` because they booked a meeting.
- `get_contact.active_sequence_ids` are the live enrollments. Use
  `list_sequences` only when you need status or template on each one.
- `update_crm_field` writes properties on an already-connected record. It
  cannot create an opportunity or a meeting.
- If nothing is in flight and they only want a future block, use
  `suppressing-or-releasing-targets`.
