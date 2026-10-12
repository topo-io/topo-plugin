---
name: running-and-monitoring-playbooks
description: Inventory Topo playbooks, enable or pause them, start a manual or CSV run, and diagnose failed or stuck runs from their step outputs. Use when the user asks which automations are on, why a playbook failed or did nothing, to turn one on or off, to run one now or on a file, or to cancel a run.
---

# Running and monitoring playbooks

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

Building or editing steps is `building-a-playbook`. This skill covers
playbooks that already exist.

## Inventory and status

1. `list_playbooks(status=ENABLED|PAUSED|DRAFT, search=…)` — page with
   `page` while there are more.
2. `get_playbook(playbook_id)` — status, trigger type, `live_version`,
   whether the draft has unpublished changes, `run_fields` for a manual
   run, and `run_stats` **for the last 7 days only** (runs, succeeded,
   failed, waiting on an approval). Quote counts with their window; never
   turn them into a rate you did not measure.
3. `get_playbook_draft(playbook_id)` when the user asks what the playbook
   does step by step. Describe steps by their title.

## Diagnose a run

1. `list_playbook_runs(playbook_id, status=FAILED, is_test=false)` — newest
   first, with each failed run's `error_message`. Quote it verbatim and
   check whether recent runs failed the same way.
2. `get_playbook_run(run_id)` — every step's status, input, output, and
   error. A step that failed and was skipped does not fail the run: open the
   run before calling it clean.
3. Read the causes in this order:
   - **Waiting on approval** — blocked, not broken. The approval is a
     `PLAYBOOK_APPROVAL` task: hand off to `clearing-todays-tasks`.
   - **Never triggered** — status not `ENABLED`, or the trigger never fired
     (no matching reply, signal import, event, or list import).
     `list_signal_imports` and `list_events` show whether the source fired.
   - **Configuration** — a missing Vault item (`list_playbook_parameters`
     shows which are `configured`), a deleted list or template, a quota hit.
   - **Data** — a lead without email or company data; the step output shows
     the empty value.
4. Fixing a step is a draft change: switch to `building-a-playbook`, test,
   publish.

## Turn on, pause, run

- `set_playbook_status(playbook_id, ENABLED|PAUSED|DRAFT)`. Enabling
  re-validates the definition and the Vault, so a refusal is a configuration
  gap to report, not a transient error. A playbook never published
  (`live_version` 0) cannot be enabled. State the change from
  `previous_status` to `new_status`.
- Pausing stops future runs, not outreach earlier runs already scheduled:
  use the sequence tools (`holding-or-resuming-outreach`) for that. A live
  run waiting when its playbook is paused ends `CANCELLED`.
- `run_playbook(playbook_id, trigger_payload, dry_run?)` — one run of an
  `ENABLED` `MANUAL` playbook. Ask for each required field in `run_fields`
  and pass them as `trigger_payload` keys. `dry_run=true` shows the shape of
  a run without side effects. `deduplicated` means an existing run was
  returned.
- `run_playbook_with_csv(playbook_id, attachment_id | csv_content)` — an
  `ENABLED` `CSV` playbook on a file. Re-sending the same file starts nothing
  new. Large files queue in the background: report `queued_rows` and point to
  the runs list.
- `cancel_playbook_run(run_id)` stops a live run still in progress; its
  pending approvals expire.
- `set_playbook_parameter(key, value)` changes a plain Vault value; an
  enabled playbook reads the new value from its next run.

A playbook whose steps use an action reserved to the Topo app (an HTTP call,
a teammate notification, a free-form AI step) is refused by `run_playbook`
and `run_playbook_with_csv`: run it from the Topo app.

## Rules

- Confirm before `set_playbook_status`, a live `run_playbook`,
  `run_playbook_with_csv`, `cancel_playbook_run`, `set_playbook_parameter`,
  `rollback_playbook`, and `delete_playbook`. Each one changes what runs in
  production or what a run sends.
- Never enable a playbook the user did not ask to turn on.
- Never call a playbook a sequence to the user: a playbook produces runs, a
  sequence produces messages.
