# Changelog

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
