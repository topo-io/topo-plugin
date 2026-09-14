---
name: creating-a-sequence
description: Author a new Topo outreach sequence from a name and ordered steps, clone an existing one, or replace the copy of a template. Use when the user wants to create a sequence, duplicate one, or edit subjects, bodies, delays, or channels.
---

# Creating a sequence

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

A sequence template is the reusable outreach definition (name, steps, copy).
Leads enroll into it later. Creating one does not send. It starts
`PENDING_SETUP` until senders are attached and status is `ACTIVE`.

When to use this skill:

- **New sequence from scratch:** `create_sequence_template`.
- **Duplicate an existing one:** `clone_sequence_template`.
- **Change copy on an existing template:** `get_sequence_template(include_steps=true)` then `update_sequence_template_steps` with the full step list.
- **Change copy on one already-enrolled lead:** `update_sequence_steps` (not the template). `list_sibling_edited_steps` reuses a sibling's edit.
- **Enroll people into a sequence that already exists:** `importing-and-starting-outreach`.

## Workflow (create)

1. `list_personalization_variables`. Insert names as `{{variable_name}}`.
   Unknown names are rejected on save. Use `list_ai_variables` when they
   want an AI field.
2. Agree the steps with them: `action` (`EMAIL`, `LINKEDIN_INVITATION`,
   `LINKEDIN_MESSAGE`, `LINKEDIN_INMAIL`, `MANUAL`), `subject` (first email
   or a later email with `new_thread=true`), `body`, `delay` in days after
   the previous step. The first step has no delay. Optional
   `sender_identity_id` on a step overrides the template's identity for
   that step only.
3. `create_sequence_template(name, steps, optional summary /
   daily_leads_target / lead_approval_mode)` → `sequence_template_id`,
   `status=PENDING_SETUP`, `url`, `next_step`. The host asks the user to
   confirm before the write.
4. Launch only if they asked to start sending:
   - `list_sender_identities` → `set_sequence_template_senders`
     (`attach_identity_ids` covering every channel the steps use,
     `detach_identity_ids=[]` when unused).
   - `set_sequence_template_status` to `ACTIVE`.
5. Enroll with `importing-and-starting-outreach`. Do not enroll while
   `PENDING_SETUP`.

## Workflow (clone)

1. `list_sequence_templates(query=…)` →
   `get_sequence_template(include_steps=true)` to confirm the source.
2. `clone_sequence_template(sequence_template_id, name)` → a new
   `sequence_template_id`, still `PENDING_SETUP`. Steps and operational
   settings copy; senders do not. Then the same launch steps as create.

## Workflow (edit steps)

1. `get_sequence_template(sequence_template_id, include_steps=true)`.
2. Send the **complete** new list to `update_sequence_template_steps`.
   Future enrollments use the new copy; already-enrolled leads keep theirs.
3. Daily target, schedule, approval mode, and `bypass_account_exclusions`
   stay on `update_sequence_template_settings`.

## Workflow (per-lead copy)

1. `list_sequences` or `get_sequence` → `sequence_id` (the enrollment).
2. Optional: `list_sibling_edited_steps(sequence_id)` to reuse copy already
   edited on another lead at the same company.
3. `update_sequence_steps(sequence_id, steps=[…])` patches upcoming steps
   only. Executed steps are refused. The template is unchanged.

## Workflow (tags, assignment, exclusions, delete)

- Tags: `list_sequence_tags` → `create_sequence_tag` if missing →
  `set_sequence_template_tags` with the full id list (`[]` clears).
- Task owners: `get_sequence_template_assignment` then
  `update_sequence_template_assignment` (`assignment_pool_user_ids` from
  `list_organization_members`).
- Reply-exclusion override: `update_sequence_template_exclusion_settings`.
  Pass `settings=null` to inherit org defaults. Account-exclusion bypass
  is `update_sequence_template_settings.bypass_account_exclusions`.
- Delete: `delete_sequence_template`. Refuses while any enrollment is still
  active.

## Rules

- Never invent `{{variable}}` names. Call `list_personalization_variables`
  first.
- Cannot OAuth a mailbox. Attach identities that already exist.
- Cannot author a playbook graph.
- Do not `set_sequence_template_status(ACTIVE)` until they asked to go live
  and senders cover the channels.
- Template edits never rewrite already-enrolled leads. Per-lead copy is
  `update_sequence_steps`.
