---
name: importing-and-starting-outreach
description: Add named people to a contact list and enroll them in an existing Topo sequence, including the hidden list membership and FOUND vs ACTIVE steps. Use when the user wants to add conference contacts, a short list of emails, or a CSV to a sequence and start outreach.
---

# Importing and starting outreach

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

`enroll` needs a sequence template and `contact_id`s. Pass `contact_list_id`
when the people already sit on a list. Omit it after `import_leads` or when
enrolling from a lead page — those contacts are already in the workspace.
`import_leads` creates contacts but does not put them on a list.
`add_contact_list_members` returns `entry_ids`; member rows also carry
`contact_id` once resolved. To author a new sequence first, use
`creating-a-sequence`.

## Workflow

1. `list_sequence_templates(query=…, status=ACTIVE)` →
   `get_sequence_template(sequence_template_id)`. Confirm `status=ACTIVE`,
   `sender_identities` is non-empty, and read
   `settings.lead_approval_mode` and `settings.daily_leads_target`. If
   `PENDING_SETUP` or no senders: stop and send them to the web app to
   finish setup. You may `set_sequence_template_senders` with existing
   identities (`attach_identity_ids` and `detach_identity_ids` are both
   required — pass `[]` for the unused side) but you cannot OAuth a mailbox.
   A missing template is `creating-a-sequence`, not this skill.
2. Resolve or create the list only when they asked to keep people on one:
   `get_list(name=…)` or `list_contact_lists(search=…)`. Prefer
   `contact_list_id` (same value as `id`). Else `create_contact_list(name)`
   → `contact_list_id`. Confirm `get_list.list_kind` is `CONTACT` (not
   `ACCOUNT`). `list_list_members` accepts `CONTACT` or lowercase `contact`.
   Skip this step after `import_leads` when they only want enrollment.
3. DNC check: `list_exclusions(kind=CONTACT, search=email)` and
   `list_exclusions(kind=DOMAIN, search=domain)`. Skip anyone already blocked.
4. Ingest (pick one):

   - **Pasted handful of people:** `add_contact_list_members(contact_list_id,
     items=[{email or linkedin_url, …}])` → `entry_ids`. Then
     `list_list_members(list_kind=CONTACT, list_id=contact_list_id)` and
     collect `contact_id`. Member rows include `email` and `linkedin_url`.
     Skip rows where `contact_id` is null (unresolved import row — they
     cannot enroll yet).
   - **Already-known workspace contacts:** `upsert_contact` / `import_leads`
     → `id` / `contact_ids`. Enroll those `contact_id`s directly (omit
     `contact_list_id`). To also put them on a list, pass
     `add_contact_list_members(items=[{contact_id}])` — do not combine
     `contact_id` with email or LinkedIn on the same item.
   - **CSV:** if the host gave `attachment_id`,
     `import_csv(attachment_id, destination=CONTACT_LIST, name=…)`.
     Otherwise pass `csv_content` (header row included) and optional
     `filename`. Then `list_list_members` for `contact_id`s. Never use
     `import_leads` for a spreadsheet.

5. `enroll(sequence_template_id, contact_ids, conflict_strategy=SKIP,
   optional contact_list_id)`. Default `SKIP`. Use `ADD_IF_ALL_COMPLETED`
   only if they asked to re-run finished people. Never default to `REPLACE`
   or `ADD_ANYWAY`. Result: `enrolled_contact_ids`, `created_sequence_ids`,
   skip counts.
6. If `lead_approval_mode` is `COPILOT`: they land in `FOUND`.
   `approve_leads` only after they say start them now — omit `contact_ids`
   to approve every waiting lead on that template, or pass the small named
   subset. If `AUTOPILOT`, they become `ACTIVE` without that call.
7. Optional: `list_organization_members` → `assign_owner`.

## Rules

- Cannot OAuth a mailbox or connect a CRM. The sequence must already exist;
  authoring steps is `creating-a-sequence`.
- Use `creating-a-lead-search` when discovering new prospects ("find me 200 VPs"). Ingest known people only here.
- Prefer `attachment_id` when the host uploaded a file. Many MCP clients
  have none — pass `csv_content` instead of giving up.
- Never pass `entry_id` to `enroll`. Prefer `contact_list_id` from
  `create_contact_list` / `list_contact_lists` (same value as `id`) when
  they asked for a list.
