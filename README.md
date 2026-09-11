# Topo

Daily-run workflows for [Claude](https://code.claude.com/docs/en/plugins),
[ChatGPT](https://developers.openai.com/plugins/build/plugins), Cursor, and
other [Agent Plugins](https://agent-plugins.org/) clients. Skills teach the
assistant how to use the [Topo MCP](https://mcp.topo.io/mcp) server — when to
call `stop_sequence` vs `unenroll`, which id to pass, and the rest of the
daily-run product rules.

A Topo account is required. Connecting MCP without this plugin exposes tools
only; it does not load the workflows.

## Install (Claude Code)

The Claude plugin directory requires this repository to stay **public**. Until
the listing is approved, install from the repo:

```text
/plugin marketplace add topo-io/topo-plugin
/plugin install topo@topo-plugins
```

Then sign in to Topo when the host prompts for OAuth.

Local test from a checkout:

```bash
claude --plugin-dir .
claude plugin validate .
```

## Skills

| Skill | Job |
|---|---|
| `recording-outreach-outcomes` | Apply a known win/loss/stop without corrupting the funnel |
| `triaging-inbound-replies` | Work inbound email/LinkedIn replies and apply the matching outcome |
| `reviewing-waiting-leads` | Qualify FOUND leads; approve or refuse without a 3-month block |
| `clearing-todays-tasks` | Clear the personal task queue with the type-specific payload |
| `creating-a-sequence` | Author a new sequence, clone one, or replace its steps and copy |
| `importing-and-starting-outreach` | Ingest named people onto a list and enroll them in an existing sequence |
| `creating-a-lead-search` | Source new leads from criteria, preview or run, and import into a contact list |
| `suppressing-or-releasing-targets` | Block or lift emails/domains and stop anything already sending |
| `briefing-an-account` | Assemble who we know, who's in sequence, and what they said |
| `diagnosing-sequence-health` | Explain why a sequence is quiet and apply allowed ops fixes |
| `holding-or-resuming-outreach` | Pause for OOO / bad timing without marking people lost |

## Layout

| File | Reader |
|---|---|
| `plugin.json` + `mcp.json` + `skills/` | Agent Plugins 1.0.0 |
| `.claude-plugin/plugin.json` + `.mcp.json` | Claude Code |
| `.codex-plugin/plugin.json` + `.mcp.json` | ChatGPT / Codex |

MCP server: `https://mcp.topo.io/mcp`.

## Source of truth

Authored in the private [`topo-io/agent`](https://github.com/topo-io/agent)
monorepo (`packages/topo-plugin/`) and mirrored here so the Claude directory
can review a public GitHub URL.
