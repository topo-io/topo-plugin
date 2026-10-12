---
name: ids-and-outcomes
description: Which Topo id to pass to which MCP tool, which write records a booked meeting, a loss, a hard no, or a pause, and which writes need the user's confirmation. Read before any Topo write, and whenever unsure whether to call stop_sequence, unenroll, or refuse_leads.
---

# IDs and outcomes

Read this before any write. Passing the wrong id, or calling `unenroll` for a
meeting booked, corrupts the funnel and can keep sending.

Field names below match the MCP wire: copy the identifier under the **same
name** the next tool declares.

## Which id to pass

| You have | Comes from | Pass it to | Never pass it to |
|---|---|---|---|
| `contact_id` | threads, tasks, sequences, `get_contact`, list members, `import_leads` | `get_contact`, `qualify_lead`, `enroll`, `unenroll`, `list_sequences`, `list_threads`, `create_todo`, `update_crm_field`, `assign_owner` (target=contact), `add_contact_list_members` (item `contact_id`) | A field named `person_id`. A `search` typed id. A list `entry_id`. |
| `sequence_id` | `get_thread`, `list_sequences`, `get_contact.active_sequence_ids`, `list_tasks`, `get_task` | `get_sequence`, `stop_sequence`, `pause_sequence`, `resume_sequence`, `update_sequence_variables`, `update_sequence_steps`, `list_sibling_edited_steps`, `assign_owner` (target=sequence) | `unenroll`, `approve_leads`, `refuse_leads` |
| `sequence_template_id` | template list, `get_sequence_template`, `create_sequence_template`, `clone_sequence_template`, `list_tasks`, `list_sequences`, `get_thread` | `enroll`, `approve_leads`, `refuse_leads`, `get_sequence_metrics`, `get_sequence_template`, `clone_sequence_template`, `update_sequence_template_steps`, settings / senders / status / tags / assignment / exclusion override, `delete_sequence_template` | `stop_sequence` |
| `thread_id` | `list_threads` | `get_thread`, `archive_thread` | `reply_to_message`, `categorize_reply` |
| `last_message_id` / inbound `message_id` | `list_threads.last_message_id`, `get_thread.messages[].message_id` | `reply_to_message`, `categorize_reply` | `archive_thread` |
| `account_id` | `get_account`, `get_contact`, `list_contacts`, `list_accounts`, `find_or_create_account` | `get_account`, `list_contacts`, `assign_owner` (target=account), `update_account_record`, `get_crm_record`, `search_deals`, `create_crm_deal`, `log_crm_activity` | — |
| `account_domain` | `get_contact`, `get_account` | `add_exclusions` (`kind=DOMAIN`), `enrich_account` | — |
| `contact_list_id` | `create_contact_list`, `list_contact_lists` (also aliased as `id`), `get_signal` | `enroll` (optional when contacts are already in the workspace), `add_contact_list_members`, `update_contact_list`, `delete_contact_list`, `estimate_ai_variable_run` / `run_ai_variable` (scope `ALL`) | — |
| `account_list_id` | `create_account_list`, `list_account_lists`, `get_signal` | `add_account_list_members`, `update_account_list`, `delete_account_list`, `estimate_ai_variable_run` / `run_ai_variable` (scope `ALL`) | `enroll` |
| `list_id` + `list_kind` | `get_list` (`list_kind` is `CONTACT` / `ACCOUNT`) | `list_list_members` (lowercase `contact` / `account` also accepted) | — |
| list `entry_id` | `list_list_members`, `add_contact_list_members` | `remove_contact_list_member` / `remove_account_list_member` | `enroll`, `unenroll` |
| `task_id` | `list_tasks`, `get_task` (same value as `id`) | `get_task`, `resolve_task`, `assign_task`, `snooze_task`, `reopen_task`, `update_todo` | — |
| `assignee_user_id` | `whoami.user_id` (user-bound), `list_tasks`, `get_task`, `list_organization_members` | `assign_task`, `create_todo`, `assign_owner` | — |
| `exclusion_id` + `kind` | `list_exclusions`, `add_exclusions.created[]` | `update_exclusion`, `remove_exclusion` | — |
| `lead_search_id` | `create_lead_search`, `list_lead_searches` | `get_lead_search`, `refine_lead_search`, `preview_lead_search`, `run_lead_search`, `list_lead_search_results`, `import_lead_search_results` | — |
| `signal_id` | `list_signals`, `create_signal`, `duplicate_signal` | `get_signal`, `show_signal`, `list_signal_imports`, `update_signal`, `update_signal_config`, `pause_signal`, `resume_signal`, `archive_signal`, `unarchive_signal`, `duplicate_signal` | — |
| `playbook_id` + `revision` / `live_version` | `list_playbooks`, `create_playbook`, `get_playbook_draft` | `get_playbook_draft`, `edit_playbook_draft` (`expected_revision`), `publish_playbook` / `rollback_playbook` (`expected_live_version`), `set_playbook_status`, `run_playbook`, test tools | — |
| playbook `run_id` | `list_playbook_runs`, `start_playbook_test_run`, `run_playbook` | `get_playbook_run`, `cancel_playbook_run`, `stop_playbook_test_run`, `reuse_run_id` | — |
| `ai_variable_id` | `list_ai_variables`, `create_ai_variable` | `update_ai_variable`, `estimate_ai_variable_run`, `run_ai_variable`, `get_ai_variable_values`, `delete_ai_variable` | `simulate_ai_variable` (takes the definition, not an id) |
| `sender_id` | `list_sender_identities` `senders[].id` | `simulate_ai_variable` | — (the identity's own `id` goes to `set_sequence_template_senders`) |
| `audience_id` | `list_audiences`, `create_audience` | `get_audience`, `update_audience`, `delete_audience` | — |
| `document_id` | `list_knowledge_documents`, `search_knowledge_documents` | `get_knowledge_document`, `update_knowledge_document`, `delete_knowledge_document` | — |
| `webhook_id` | `list_webhooks`, `create_webhook` | `get_webhook`, `update_webhook`, `send_test_webhook`, `delete_webhook` | — |
| `search` typed id (`lead:<uuid>`) | `search` | `fetch` only | Any other tool. `fetch` metadata then use `contact_id` / `account_id` / `id`. |

`get_thread` returns `sequence_id` (the enrollment to pause / resume / stop),
`sequence_template_id`, `contact_id`, and `messages[].message_id`. Do not call
`list_sequences` just to recover the enrollment id of this thread.

`get_contact` returns `contact_id`, `account_id`, `account_domain`,
`phone_number`, `active_sequence_names`, and `active_sequence_ids`.
Pass those ids to `stop_sequence` / `pause_sequence`. Use `get_thread`
when you already have a thread, or `list_sequences` when you need
status / template on each enrollment.

`whoami` returns `user_id` for OAuth / user credentials (null on an org API
key). Use that for "assign to me" instead of searching members by email.

`get_sequence_template` returns both `id` and `sequence_template_id`
(same value). Prefer the long name.

## Outcome decision table

| User said | Correct write | Wrong write |
|---|---|---|
| Booked / meeting / we won / they're interested enough to stop as a success | `stop_sequence(sequence_id, reason=WIN, scope=CONTACT)` — `ACCOUNT` only if they said the whole company | `unenroll` (records a **loss** + 3-month `CONTACT_REFUSED`) |
| Hard no / unsubscribe / never email me / left company (already in flight) | `unenroll(contact_id)` for every sequence, or `stop_sequence(LOSE)` for one enrollment | `refuse_leads` (FOUND-only, no exclusion) |
| Not a fit for this sequence (still waiting, status `FOUND`) | `refuse_leads(sequence_template_id, contact_ids?)` | `unenroll` (3-month block) |
| Approve the waiting leads | `approve_leads(sequence_template_id, contact_ids)` with the leads they chose (1 to 100 per call) | `resolve_task` on each review task for the same people |
| Never contact this email/domain again | `add_exclusions` **and**, if already enrolled, `unenroll` | `add_exclusions` alone (does not stop live sequences) |
| OOO / vacation / try again in Q1 | `pause_sequence(sequence_id, paused_until)` | `unenroll` |
| Pause the whole sequence product | `set_sequence_template_status(INACTIVE)` | `unenroll` on every lead |

`categorize_reply` updates the inbox label only. It does not stop sequences,
create exclusions, or pause sending. Do the matching write from the table.

## Confirm before these writes

The host MCP client owns approval. State the plan in one sentence, then call
the tool and let the interrupt fire. There is no dedicated ask-user tool.

Always confirm: `reply_to_message`, `enroll`, `approve_leads`,
`stop_sequence`, `unenroll`, `refuse_leads` (bulk), `add_exclusions` with
`kind=DOMAIN`, `remove_exclusion`, `import_csv` into exclusions,
`create_sequence_template`, `clone_sequence_template`,
`update_sequence_template_steps`, `update_sequence_steps`,
`set_sequence_template_status`, `delete_sequence_template`,
`resume_sequence`, `remove_contact_list_member` (stops the sequences that
list started), `get_and_enrich_contact` (credits).

Also confirm, with the skill that owns them: signal creation and changes
(`setting-up-a-signal`), AI variable creation and every `run_ai_variable`
(`creating-ai-variables`, after `estimate_ai_variable_run`), publishing,
enabling, running, or rolling back a playbook (`building-a-playbook`,
`running-and-monitoring-playbooks`), CRM writes (`updating-the-crm`), company
profile, audience, knowledge base and webhook writes
(`configuring-the-workspace`), and list or account writes
(`managing-lists-and-accounts`).

Not available over MCP, done by a person in the web app: switching a
sequence to `AUTOPILOT`, bypassing the account exclusion list, the
`ADD_ANYWAY` / `REPLACE` enrollment strategies, removing, shortening, or
relabeling an opt-out (`UNSUBSCRIBED`, `CONTACT_REFUSED`), and running or
testing a playbook whose steps use an action that is not an MCP tool (an
HTTP call, a teammate notification, a free-form AI step).

## Known gaps

- `get_task` has no `thread_id` / `message_id`. Reply tasks go
  `list_threads(contact_id)` → `last_message_id` (or `get_thread` for the
  body). `sequence_id` is on the task when it came from an enrollment.
- `simulate_ai_variable` wants `sender_id`. Identities return `id` and
  `senders[].id` — do not pass the identity id.

## What MCP cannot do

Send the user to the Topo web app for: lead searches owned by a signal
source, `CRM_LIST_ENTRY` signals, testing or running a playbook that uses an
app-only action, connecting a mailbox or CRM, billing, inviting members,
and secrets in the playbook Vault. DNS records (SPF, DKIM) are set by the
user at their own DNS provider; mailboxes bought through Topo come already
configured. Do not pretend a workaround exists.
