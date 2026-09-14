---
name: holding-or-resuming-outreach
description: Temporarily pause Topo outreach for out-of-office, vacation, or bad timing, and resume it later, without marking people lost. Use when the user wants to pause a lead until a date, freeze a whole sequence this week, or resume paused enrollments.
---

# Holding or resuming outreach

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

OOO is the case where models call `unenroll` on a future buyer. Pause does
not record a loss and does not create a `CONTACT_REFUSED` exclusion.

## Workflow

1. Decide scope from language:
   - One person, one sequence → enrollment pause
   - One person, all sequences → pause each live `sequence_id` (do not
     `unenroll`)
   - The whole sequence product → template `INACTIVE`
2. Resolve: if they pointed at a thread, `get_thread.sequence_id`. If they
   named a person, `get_contact.active_sequence_ids`. Call
   `list_sequences(contact_id)` only when you need status on each
   enrollment. For the whole product: `get_sequence_template` →
   `sequence_template_id`.
3. Writes:
   - Person / enrollment: `pause_sequence(sequence_id, paused_until)` —
     timezone-aware datetime. There is no open-ended pause on this tool.
   - Whole template: `set_sequence_template_status(sequence_template_id,
     INACTIVE)`
   - Resume person: `resume_sequence(sequence_id)`
   - Resume template: `set_sequence_template_status(..., ACTIVE)` (validates
     senders / readiness)
4. If they also said "don't let new searches pick them up":
   `add_exclusions` then `update_exclusion` with `created[].exclusion_id`
   - `kind` and `until`. Pause alone does not block new enrolls.
5. If an inbound OOO started this: `list_threads` → `last_message_id` (or
   `get_thread` for the body) → `categorize_reply(OUT_OF_OFFICE|BAD_TIMING)`
   then `pause_sequence(get_thread.sequence_id)`. Category alone does not
   stop sending.

## Rules

- Refuse `unenroll` unless they clearly want a 3-month refusal.
- Template `INACTIVE` stops new sends across every enrollment on that
  sequence. Confirm that blast radius.
- Cannot change step delays ("wait 14 days instead of 3") — web-only.
- No "pause everything Marie sends" filter by sender. Resolve enrollments
  or pause the templates you can identify.
