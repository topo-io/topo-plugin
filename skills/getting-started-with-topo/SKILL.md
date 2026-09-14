---
name: getting-started-with-topo
description: Orient a Topo session — confirm who is signed in, take stock of the workspace, and route the request to the right Topo skill. Use when the user asks what Topo can do, whether they are connected, what is in their workspace, or gives an outbound sales request that does not obviously match another Topo skill.
---

# Getting started with Topo

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) before any write.

Topo is an outbound sales workspace: sequence templates send email and
LinkedIn steps to enrolled leads, replies land in the inbox, and follow-up
work becomes tasks. Every tool on this MCP is scoped to the workspace the
signed-in user belongs to.

## First call of a session

1. `whoami` — confirms the connection, the organization, and `user_id`
   (null on an organization API key). If it fails with an authentication
   error, the host has not completed the Topo sign-in yet: ask the user to
   finish the OAuth prompt, then retry. Do not guess credentials.
2. Once per session when qualification or copy comes up:
   `get_organization_context` for the ICP and positioning.

## Taking stock

Answer "what is going on?" from three reads, in this order:

- `list_threads(last_message_direction=INBOUND)` — who replied and needs an
  answer.
- `list_tasks(assignee_email="me", status=PENDING)` — what is due today.
- `list_sequence_templates(status=ACTIVE)` — which sequences are sending.

Use `search(query)` then `fetch` when the user names a person or company you
cannot resolve, then continue with the `contact_id` or `account_id` from the
metadata.

## Route to the right skill

| The user wants to | Skill |
|---|---|
| Read or answer replies, handle the inbox | `triaging-inbound-replies` |
| Record a booked meeting, win, loss, unsubscribe | `recording-outreach-outcomes` |
| Approve or refuse leads waiting on a sequence | `reviewing-waiting-leads` |
| Clear, snooze, or reassign today's tasks | `clearing-todays-tasks` |
| Add known people to a list and start outreach | `importing-and-starting-outreach` |
| Write, clone, or edit a sequence | `creating-a-sequence` |
| Find new prospects from criteria | `creating-a-lead-search` |
| Block or release an email or domain | `suppressing-or-releasing-targets` |
| Prepare for a call on an account | `briefing-an-account` |
| Understand why a sequence is quiet or how it performed | `diagnosing-sequence-health` |
| Pause someone until a date, or resume | `holding-or-resuming-outreach` |

## Rules

- Reads are free. Sending, stopping, blocking, or spending credits waits for
  the user's explicit go-ahead; state the plan in one sentence first.
- Share the `url` a tool returns so the user can open the record in Topo.
- Send the user to the Topo web app for what MCP cannot do: connecting a
  mailbox or CRM, billing, inviting members, playbook graphs.
