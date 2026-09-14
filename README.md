# Topo

Run your [Topo](https://topo.io) outbound workspace from
[Claude](https://code.claude.com/docs/en/plugins),
[ChatGPT and Codex](https://developers.openai.com/codex/plugins/build), Cursor,
and other [Agent Plugins](https://agent-plugins.org/) clients. The plugin
connects the [Topo MCP](https://mcp.topo.io/mcp) server and loads the skills
sales teams run every day — when to call `stop_sequence` vs `unenroll`, which
id to pass, and the rest of the product rules the tools alone do not carry.

A Topo account is required. Connecting MCP without this plugin exposes tools
only; it does not load the workflows.

## What you can do

- Triage inbound replies and answer them in-thread.
- Record a booked meeting, a win, a loss, or an unsubscribe so sending stops.
- Review the leads waiting on a sequence and approve or refuse them.
- Clear today's tasks, snooze what can wait, pause or resume outreach.
- Import people from a list or CSV and start a sequence.
- Create or clone a sequence template and edit its copy.
- Source new leads from criteria with a preview before credits are spent.
- Suppress or release emails and domains.
- Brief an account from its contacts, enrollments, and threads.
- Diagnose a quiet sequence and apply the operational fixes MCP allows.

Reads are free. Anything that sends, stops, blocks, or spends credits asks for
your confirmation first.

## Install

### Claude Code

```text
/plugin marketplace add topo-io/topo-plugin
/plugin install topo@topo-plugins
```

Local test from a checkout:

```bash
claude --plugin-dir .
claude plugin validate . --strict
```

### ChatGPT and Codex

Install Topo from the plugin directory, or point a local marketplace at a
checkout of this repository (`plugin.json` at the root is the portable
manifest; `.codex-plugin/plugin.json` is the compatibility fallback).

### Other Agent Plugins clients

Load the directory: `plugin.json`, `mcp.json`, and `skills/` follow Agent
Plugins 1.0.0.

## Sign in

The first tool call opens a Topo sign-in (OAuth). The plugin only sees the
workspace your Topo user belongs to. Remove the plugin from your host to
revoke access; tokens are deleted with it.

## Skills

| Skill | Job |
|---|---|
| `getting-started-with-topo` | Confirm the connection, take stock, and route to the right skill |
| `triaging-inbound-replies` | Work inbound email/LinkedIn replies and apply the matching outcome |
| `recording-outreach-outcomes` | Apply a known win/loss/stop without corrupting the funnel |
| `reviewing-waiting-leads` | Qualify waiting leads; approve or refuse without a 3-month block |
| `clearing-todays-tasks` | Clear the personal task queue with the type-specific payload |
| `importing-and-starting-outreach` | Put named people on a list and enroll them in an existing sequence |
| `creating-a-sequence` | Author a new sequence, clone one, or replace its steps and copy |
| `creating-a-lead-search` | Source new leads from criteria, preview or run, and import into a list |
| `suppressing-or-releasing-targets` | Block or lift emails/domains and stop anything already sending |
| `briefing-an-account` | Assemble who we know, who is in sequence, and what they said |
| `diagnosing-sequence-health` | Explain why a sequence is quiet and apply allowed ops fixes |
| `holding-or-resuming-outreach` | Pause for OOO / bad timing without marking people lost |
| `ids-and-outcomes` | Which id goes to which tool, and which write records each outcome; every other skill reads it first |

## Layout

| File | Reader |
|---|---|
| `plugin.json` + `mcp.json` + `skills/` + `assets/` | Agent Plugins 1.0.0, ChatGPT, Codex |
| `.claude-plugin/plugin.json` + `.mcp.json` | Claude Code |
| `.codex-plugin/plugin.json` | ChatGPT / Codex compatibility fallback |

MCP server: `https://mcp.topo.io/mcp`.

## Support, privacy, terms

- Support: <https://support.topo.io>
- Documentation: <https://docs.topo.io>
- Plugin issues: <https://github.com/topo-io/topo-plugin/issues>
- Privacy policy: <https://www.topo.io/consents/privacy-policy>
- Terms of service: <https://www.topo.io/consents/terms-of-service>

## Source of truth

Authored in the private [`topo-io/agent`](https://github.com/topo-io/agent)
monorepo (`packages/topo-plugin/`) and mirrored here so the plugin directories
can review a public GitHub URL. See [CHANGELOG.md](CHANGELOG.md) for releases.
