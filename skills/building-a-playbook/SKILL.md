---
name: building-a-playbook
description: Build or change a Topo playbook — a trigger-based automation (inbound reply, signal import, product event, schedule, list import, manual run, CSV upload) that runs a graph of steps — then test it step by step and end to end, and publish it. Use when the user wants to automate something ("when X happens, do Y"), create or edit a playbook or one of its steps, start from a template, set a Vault value, test a playbook, or publish or roll it back.
---

# Building a playbook

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

You write the **draft**, test it, and propose publishing; the person
approves everything that reaches production. Nothing you write runs until a
published version is enabled. The step grammar is in
[authoring-contract.md](references/authoring-contract.md): read it before
the first `edit_playbook_draft` of a conversation.

A playbook is not a sequence: it automates work on a trigger and produces
runs. For outreach copy use `creating-a-sequence`. To run, enable, or
diagnose an existing playbook use `running-and-monitoring-playbooks`.

## What works over this connector

A step can only use an action that is also a tool of this server.
`get_playbook_building_blocks` lists exactly those. A playbook built in the
Topo app with an app-only action (an HTTP call, a teammate notification, a
free-form AI step) can still be read here, but testing or running it is
refused: send the user to the Topo app for it. Code, Condition, Map, Batch,
Wait, and Approval steps are always available.

## Build or change

1. **Locate or create.**
   - New: `list_playbook_templates` first; when one fits,
     `create_playbook(name, template_key=…)`. Otherwise
     `create_playbook(name)` blank, or `duplicate_of_playbook_id` to copy.
   - Existing: `list_playbooks(search=…)` → `get_playbook_draft(playbook_id)`
     for the steps, the `revision`, `live_version`, validation issues, Vault
     requirements, and the latest test.
2. **Ground.** `get_playbook_building_blocks(capability_names=[…])` with the
   actions you plan to use: their exact input and output fields, the trigger
   families and their payload fields, the step kinds, the Condition
   operators. For every input that names a record (list, sequence template,
   signal, event name, member, CRM property, reply category, deal stage…),
   `list_playbook_field_options(source_key=…)` and use a returned `value`.
   When several fit, ask which one.
3. **Write the draft.** One `edit_playbook_draft(playbook_id,
   expected_revision, operations)` per logical change, every operation of it
   batched. Read the returned `issues`: fix every `error`, fix or explain
   every `warning`. On a revision conflict, re-read with `get_playbook_draft`
   and re-apply on the new revision; never retry blindly.
4. **Make the Vault ready.** Read `parameter_requirements`. Ask for each
   missing plain value and store it with `set_playbook_parameter(key, value,
   description)`. For a secret or credential, give the returned `vault_url`
   and wait until the user says it is saved. A test run refuses to start
   while a required item is missing.

## Test

1. **Pick real input.** `list_playbook_test_cases(playbook_id)` returns saved
   test cases, `recent_trigger_payloads`, and `trigger_samples` built from
   real occurrences of the trigger. Reuse one. Write a payload by hand only
   when all are empty, with ids and domains read from the workspace.
   `create_playbook_test_case(source=AI, description=…)` generates one for a
   situation, but holds no record ids.
2. **Test each step.** `test_playbook_step(playbook_id, node_id,
   trigger_payload, upstream_outputs)` on every Code step, Condition, and new
   or changed action. Reads run for real; writes and sends are simulated.
   `passed` → check the output; `simulated` → check the resolved inputs (the
   right list, recipient, text); `failed` → fix and retest; `unsupported`
   (map, wait, batch, approval) → cover it end to end.
3. **Test end to end.** `start_playbook_test_run(playbook_id, test_case_id
   or trigger_payload, step_modes={}, forced_branches={}, pins={})`, then
   `get_playbook_run(run_id, wait_seconds=…)` until it settles. Force every
   Condition branch that leads to an action in at least one run
   (`"yes"`, a case id, or `"__else__"`). After a fix, re-run with
   `from_node_id` and `reuse_run_id`. `stop_playbook_test_run` cancels a
   stuck test.
4. **Judge outputs, not the status.** A succeeded run that would enroll the
   wrong list, a message with an empty value, or a branch nobody reached is
   a failed test. Report in three sentences at most: what passed with the
   evidence, what failed and why, and that nothing was sent or written. End
   with one concrete yes-or-no proposal.

Test modes per step (`step_modes`): reads `RUN` / `PIN` / `SKIP`; paid reads
default to `SIMULATE` (say so when you `RUN` one: it spends credits); writes
`SIMULATE` / `SKIP`; notifications `SIMULATE` / `SEND_TO_ME` / `SKIP`;
approvals `APPROVE` / `REJECT`. Never a live mode.

## Publish

1. Only after a test run of the **current** draft succeeded and you reported
   it: ask "Want me to publish it?".
2. On yes: `publish_playbook(playbook_id, expected_live_version, note)` with
   the `live_version` you read and a note saying what changed. Publishing is
   refused without a passing test; pass `confirm_untested=true` only when the
   person explicitly asked to publish untested, after saying "this version
   has not been tested".
3. Enabling is a separate decision: `set_playbook_status(ENABLED)` only when
   the user wants it on (see `running-and-monitoring-playbooks`).

Editing a published playbook follows the same loop; the live version keeps
running unchanged until the next publish. `discard_playbook_draft(playbook_id,
expected_revision)` drops unpublished changes; `list_playbook_versions` then
`rollback_playbook(playbook_id, version, expected_live_version)` brings back
a past version.

## Rules

- Never invent ids, actions, inputs, outputs, or trigger fields: they come
  from `list_playbook_field_options`, `get_playbook_building_blocks`, and
  `list_playbook_test_cases`.
- Never ask for, accept, store, or repeat a secret. If one is pasted, do not
  echo it: point to the `vault_url` and suggest rotating it.
- Title every step in plain words; talk about steps by title, never by id,
  revision, or tool name.
- Confirm before `publish_playbook`, `rollback_playbook`,
  `discard_playbook_draft`, `delete_playbook`, `set_playbook_parameter` on a
  key an enabled playbook reads, and `start_playbook_test_run` with a paid
  read set to `RUN`.
- Irreversible steps (sends, enrollments, CRM writes) raise a warning unless
  an approval step or `require_approval_for_irreversible` covers them.
  Explain it; add an approval only when the user wants one.
