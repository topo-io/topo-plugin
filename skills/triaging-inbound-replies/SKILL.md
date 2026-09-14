---
name: triaging-inbound-replies
description: Work inbound email and LinkedIn replies in Topo — read the thread, classify intent, reply when needed, and apply the matching sequence outcome. Use when the user asks what came in overnight, who replied, what a lead said, or to handle the inbox.
---

# Triaging inbound replies

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

`list_threads` and `get_thread` never send, categorize, or stop outreach.
Quote only message text `get_thread` returned.

## Workflow

1. `list_threads(last_message_direction=INBOUND)`. Page with `page` / `size`.
   Narrow with `query` (name or domain) or `lead_status=WAITING_FOR_USER`.
   Each row has `thread_id`, `contact_id`, and `last_message_id` (the latest
   message — pass to `reply_to_message` / `categorize_reply` when you do not
   need the body yet).
2. For each row you will quote or decide on: `get_thread(thread_id)`.
3. From `get_thread` keep:
   - the last inbound `messages[].message_id` (`direction=INBOUND`; confirm
     it matches `last_message_id` before sending)
   - `contact_id`
   - `sequence_id` (this enrollment — pass to `stop_sequence` / `pause_sequence`)
   - `sequence_template_id`
   - verbatim `body` and current `reply_category`
4. `get_contact(contact_id)` for email, `account_id`, `account_domain`, and
   active sequence names. You do not need `list_sequences` just to stop this
   thread's enrollment.
5. Optional: `list_activities(contact_id)` when the thread is thin.
6. Classify from the verbatim inbound body. If the label is wrong:
   `categorize_reply(message_id, category)` —
   `MEETING`, `INTERESTED`, `BAD_TIMING`, `REFERRAL`, `WRONG_PERSON`,
   `LEFT_COMPANY`, `OUT_OF_OFFICE`, `SPAM_FILTER`, `UNKNOWN`,
   `UNSUBSCRIBED`, `BOUNCED`.
7. Apply the matching write (category alone does nothing to sending):

   - Meeting / closed-won → `stop_sequence(sequence_id, reason=WIN, scope=CONTACT)`
   - Hard no / unsubscribe / left company / bounce → `unenroll(contact_id)`
     (or `stop_sequence(LOSE)` if only this enrollment should die)
   - OOO / bad timing → `pause_sequence(sequence_id, paused_until=…)`
   - Soft interest or a question → no stop; reply only
8. To answer: `reply_to_message(message_id, body)`. Set
   `invalidate_scheduled_steps=true` when this reply replaces remaining
   sequence mail.
9. Optional: `archive_thread(thread_id)` after a terminal outcome;
   `create_todo(contact_id, due_date, title)` — pass `whoami.user_id` as
   `assignee_user_id` when they said "for me".

## Rules

- Threads exist only for leads already enrolled. An empty inbox is not "no
  one replied in Gmail."
- Never invent a `message_id`, sender, or quote.
- Cannot create a calendar event. Offer a to-do instead.
- Cannot edit sequence copy from here.
