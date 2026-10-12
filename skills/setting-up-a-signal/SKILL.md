---
name: setting-up-a-signal
description: Create, monitor, and maintain Topo signals — standing daily watches on buying signals (hiring, job changes, technology adoption, the workspace's own product events — whichever presets the workspace has enabled) that import qualified leads or companies into a list and can enroll them in a sequence. Use when the user wants to be told when something happens at target accounts, asks what their signals brought in, or wants a signal paused, resumed, retargeted, duplicated, or retired.
---

# Setting up a signal

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

A signal is a **standing watch**, not a search: it checks its provider every
day and drips what it finds into a destination list it owns. Reach for it
when the request has a *when* in it ("tell me when a company starts hiring
SDRs", "tell me when a champion changes jobs"). "Give me 500 leads now" is
`creating-a-lead-search`, never a signal.

## Create a signal

1. `list_signal_presets` — the presets this workspace can build, each with
   its `signal_type`, `scope` (what the provider detects), `filter_keys`,
   `required_filter_keys`, `preview_supported`, and `backfill_windows_days`.
   A preset that is not listed cannot be created: never invent a
   `signal_type` or a filter key. Quote the preset's title to the user.
2. Fill every `required_filter_keys` value with something the user actually
   said. A placeholder filter imports the wrong companies every day.
3. When `preview_supported` is true:
   `preview_signal(signal_type, search_filters)` — up to 10 recent matches
   over 30 days, no watch started, nothing imported. Summarize the sample in
   a sentence or two (how many, which companies or roles, anything
   off-target). It is wider than a live run (no import filter, no dedup).
4. Agree with the user, before creating:
   - **Who lands in the list** — **import_search_filters** (titles,
     seniority, geography, company size). They are not the detection
     filters: `search_filters` narrow the fact at the source,
     **import_search_filters** narrow who is imported afterwards.
   - **People or companies** — a company preset imports companies by
     default. Set `import_target=PEOPLE` when the user wants the people at
     those companies, and cap with `max_companies_per_run` plus
     `max_contacts_per_company` (never with `max_per_run` too).
   - **Daily volume and enrichment** — `max_per_run`, `enrich_email`,
     `enrich_phones`. Enrichment spends credits on every run; never call it
     free or automatic.
   - **Enrollment** — `sequence_template_id` only when the user has a
     sequence ready (people destinations only). Imported leads then start
     outreach without another confirmation.
   - **Past matches** — `backfill` (`target`, plus `windowDays` from
     `backfill_windows_days` when that list is not empty) only when they
     asked for the list to start full. It is a one-time import on top of the
     daily watch.
5. `create_signal(signal_type, name, search_filters, …)` after the user's
   go-ahead. It starts **active** and runs a first check immediately: there
   is no draft. Name it after what it watches ("VP Sales hires, DACH") — the
   list carries the same name. Report the `max_per_day` and enrichment that
   came back, and share its `url`.

`CUSTOM_EVENT` watches the workspace's own product events. It needs
`event_subject` (`PEOPLE` for events identified by email, `COMPANIES` for
domains) and the event name in `first_party_event_name`. Check the event
exists with `list_events(event_name=…)` first. `CRM_LIST_ENTRY` cannot be
built here: send the user to Signals in the Topo web app.

## Monitor

- `list_signals` — status, destination list, the sequence template it
  enrolls into, caps, and `imported_count`. Page with `page` while
  `has_more` is true. `get_signal(signal_id)` adds the filters and the
  enrichment settings. `show_signal(signal_id)` renders the signal card when
  the host supports it.
- `list_signal_imports(signal_id=…)` is the only honest answer to "what did
  my signals find": `IMPORTED`, `ALREADY_IN_LIST` (a normal duplicate),
  `REJECTED` (filtered out, with the reason), `EXPIRED`, `FOUND` (not
  processed yet). Never answer from `imported_count` alone.
- A quiet signal is usually a filter problem: `REJECTED` entries mean the
  import filters are too tight; no entries at all mean the detection
  filters match nothing. Excluded people and domains never land in the
  list — that is suppression working.
- Events: `list_events(resolution_status=UNRESOLVED)` shows product events
  that matched no lead; `get_event(event_id)` reads one.

## Change or stop

| The user wants | Call |
|---|---|
| Stop it for now | `pause_signal(signal_id)` |
| Start it again | `resume_signal(signal_id)` — checks right away |
| Rename it (and its list) | `update_signal(signal_id, name)` |
| Change filters, caps, enrichment, import filters, enrollment | `get_signal` then `update_signal_config(signal_id, …)` — omitted fields keep their value; new filters restart the baseline |
| Same watch, another list or market | `duplicate_signal(signal_id, name)` — starts active |
| Retire it for good | `archive_signal(signal_id)` — `resume_signal` then refuses it |
| Bring back an archived one | `unarchive_signal` (comes back `PAUSED`), then `resume_signal` only if they want it live |

The preset and people-vs-companies cannot change in place: duplicate or
create a new signal, then archive the old one. None of these calls touch
what was already imported.

## Recording product events

`ingest_event(event_name, occurred_at, subject, external_event_id?, payload?)`
records a first-party event (signup, demo request, product action) against
an email or a domain. Once matched, it starts event-triggered playbooks and
`CUSTOM_EVENT` signals and reaches webhook subscriptions, so confirm first.
Resending the same `external_event_id` is a replay, not a second event.

## Rules

- Confirm before `create_signal`, `duplicate_signal`, `resume_signal`,
  `update_signal_config`, `archive_signal`, and `ingest_event`: each one
  starts or changes a daily import that can spend credits.
- Prefer `pause_signal` to `archive_signal` unless the watch is over for good.
- `update_signal` only renames. Never present it as a retarget.
- When a tool says signals are not enabled for the workspace, report it as a
  workspace setting; do not retry.
