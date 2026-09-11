---
name: clearing-todays-tasks
description: Work the Topo task inbox — list what is due, then complete, skip, snooze, or reassign with the payload each task type requires. Use when the user asks what is on their plate, to clear hot tasks, log a call, snooze a follow-up, or hand a task to a teammate.
---

# Clearing today's tasks

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

`list_tasks` on MCP takes one `status`, one `task_type`, one `priority` — not
arrays. `resolve_task` is the write (completed or skipped). The in-app
complete/skip task tools are not on this surface.

## Workflow

1. `whoami`. `list_tasks(assignee_email="me")` needs a user-bound
   credential. On an org API key, `whoami.user_id` is null — pass the
   person's email from `whoami.principal` or `list_organization_members`.
   Use `whoami.user_id` as `assignee_user_id` when they said "assign to
   me" / "a todo for me".
2. `list_tasks(assignee_email="me", status=PENDING)`. Optionally
   `priority=HOT` or `TODAY`. Use `total_count`, not the page length.
   Each row includes `task_id`, `assignee_user_id`, `contact_id`, and
   `sequence_template_id`.
3. For each `task_id`: `get_task(task_id)` when the list row is too thin
   — `task_id` is the same as `id`, `type` decides the legal write,
   `notes` are on the detail. Both the list row and `get_task` include
   `sequence_id` when the task came from an enrollment. Reassigning can
   use the list row's `assignee_user_id` without this extra get.
4. If `contact_id` is set: `get_contact(contact_id)`. For reply types:
   `list_threads(contact_id)` — prefer `last_message_id` when
   `last_message_direction` is inbound. `get_task` has no
   `thread_id` / `message_id`.
5. Branch on `type`:

   | Type | Next tool | How it closes |
   |---|---|---|
   | `EMAIL_REPLY` / `LINKEDIN_REPLY` | `reply_to_message(message_id, body)` using `last_message_id` or the inbound `messages[].message_id` | Sending the reply closes it. Do **not** also `resolve_task`. |
   | `NEW_LEAD_REVIEW` | `qualify_lead(contact_id)` | `resolve_task(..., outcome=completed, new_lead_review_action=APPROVE\|REFUSE)` |
   | `CALL` | (the user's call result) | `resolve_task(..., call_disposition=ANSWERED\|VOICEMAIL_LEFT\|NO_ANSWER\|INTERESTED\|NOT_INTERESTED\|WRONG_NUMBER)` |
   | `PLAYBOOK_APPROVAL` | `get_playbook` only if they asked what it does | `resolve_task(..., playbook_approval_action=APPROVE\|REJECT)` |
   | `TODO` | `update_todo` if the work changed | `resolve_task(outcome=completed\|skipped)` |

6. After `NOT_INTERESTED` or `WRONG_NUMBER` on a call: offer
   `recording-outreach-outcomes`. Resolving the task does not stop mail.
7. Hygiene: `snooze_task(task_id, snoozed_until)` (timezone-aware, future);
   `list_organization_members(query=…)` → `user_id` →
   `assign_task(task_id, assignee_user_id)`; `reopen_task` only if the close
   was a mistake. Re-resolving a closed task errors.

## Rules

- Never complete a lead review, call, or playbook approval without its
  outcome field.
- Cannot place a phone call or edit a playbook graph.
- Skipping is not completing. Use `outcome=skipped` only when the work no
  longer applies.
- `create_todo(title, due_date)` for new trackable work. Prefer a due date
  over a promise in prose.
