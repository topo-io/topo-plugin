# Changelog

## 1.3.4

- `destructiveHint` is now `false` only on writes that add something
  (`create_*`, `add_*`, `submit_*`, …). Updates, removals, pauses, snoozes,
  reassignments, archive toggles, draft edits and recategorizations ask for
  confirmation, and so do `create_ai_variable` (auto-run spends credits),
  `start_playbook_test_run` (paid reads set to RUN) and `ingest_event`.
- `ingest_event` says that a recorded event starts event-triggered playbooks
  and first-party signals and reaches webhook subscriptions, and is
  open-world.
- Signal, lead-data, enrichment and AI web research tools state where the
  data comes from and how a person stays in control, with the link to the
  data sources and removal page.

## 1.3.3

- Skills describe email-only outreach, matching the connector's published
  tools: step actions are `EMAIL` and `MANUAL`, leads resolve by email, and
  the inbox and task flows cover email replies.

## 1.3.2

- `openWorldHint` now covers every tool that reaches outside the workspace
  directly or through what it turns on: outreach (`enroll`, `approve_leads`,
  `resolve_task`, `resume_sequence`, `set_sequence_template_status`,
  `update_contact_list`, `add_contact_list_members`), signal providers
  (`preview_signal`, `create_signal`, `duplicate_signal`, `resume_signal`,
  `update_signal_config`), the lead database and enrichment
  (`import_lead_search_results`), AI web research (`create_ai_variable`),
  webhook URLs (`create_webhook`, `update_webhook`), and playbooks
  (`run_playbook`, `run_playbook_with_csv`, `set_playbook_status`).
- `destructiveHint` now also covers the overwrites that keep no previous
  value (`update_record`, `update_account_record`,
  `update_sequence_variables`, `set_sequence_template_tags`,
  `update_ai_variable`, `update_webhook`, `create_or_update_contact`),
  `remove_contact_list_member` (it stops the sequences the list started),
  `resume_sequence`, `stop_playbook_test_run`, and the list writes that can
  enroll members into a default sequence.
- `upsert_contact` is now `create_or_update_contact` and `upsert_account`
  `find_or_create_account`; the old names still answer for hosts holding a
  cached tool list.
- `execute_task` is no longer an MCP tool: `reply_to_message` sends replies
  and `resolve_task` decides reviews, calls and playbook approvals.
- Playbooks only run or test over MCP when every step is itself an MCP tool;
  `get_playbook_building_blocks` lists those actions only.
- Shorter server instructions that lead with the confirmation rule.

## 1.3.1

- The shared id and outcome rules are now the `ids-and-outcomes` skill, so
  every directory under `skills/` is a skill with its own `SKILL.md` and the
  other skills load it by name on hosts that import skills one by one.
- `interface.supportURL` points at the Topo knowledge base
  (`https://support.topo.io`), which the ChatGPT directory requires alongside
  the website, privacy, and terms links.
- `destructiveHint` now also covers the deletes (`delete_contact_list`,
  `delete_account_list`, `delete_webhook`), the calls that can send or start
  outreach (`execute_task`, `resolve_task`), the CRM overwrite
  (`update_crm_field`), and the credit spends (`run_lead_search`,
  `import_lead_search_results`); `execute_task` is open-world because it can
  send a reply. Hosts now ask before running them.

## 1.3.0

- Directory-ready listing for ChatGPT and Codex under `extensions.com.openai`
  in `plugin.json`: display name, 30-character summary, long description,
  outcome-based capabilities, three starter prompts, privacy and terms links,
  square logo and composer icon under `assets/`. `.codex-plugin/plugin.json`
  mirrors it as the compatibility fallback.
- Claude Code manifest gains `displayName` and a schema; both marketplaces
  describe the plugin fully (display name, category, tags, links).
- New `getting-started-with-topo` skill: confirms the connection, takes stock
  of the workspace, and routes a request to the right skill.
- Skills no longer reference internal specifications or provider enum names;
  `creating-a-lead-search` describes signal-owned searches in plain language.
- `.mcp.json` relies on the server's RFC 9728 metadata for OAuth discovery.
- Every MCP tool now publishes `readOnlyHint`, `destructiveHint`, and
  `openWorldHint` explicitly; tools that reach beyond the workspace
  (replies, enrichment, lead-database runs, URL imports, webhook tests) are
  open-world.
- Every tool parameter carries its type in place: enums and nested objects
  are inlined in `inputSchema` instead of `$ref` pointers into `$defs`, so
  directory scanners no longer report them as untyped.

## 1.2.0

- `importing-and-starting-outreach` covers `import_leads`, `upsert_contact`,
  `import_csv` with `csv_content`, and enrolling without a list.
- `add_contact_list_members` accepts `contact_id` items; `enroll` accepts
  `contact_ids` alone after an import.
- `creating-a-sequence` covers tags, task assignment, the reply-exclusion
  override, per-lead copy edits, and deletion.

## 1.1.0

- New `creating-a-sequence` skill: create, clone, edit template steps, and
  patch one lead's remaining copy.

## 1.0.0

- Initial release as an Agent Plugins 1.0.0 package with Claude Code and
  ChatGPT manifests and ten daily-run skills.
