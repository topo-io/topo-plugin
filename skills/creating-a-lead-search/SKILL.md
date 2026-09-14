---
name: creating-a-lead-search
description: Source new leads from criteria using Topo lead search, preview or run the search with explicit user confirmation, and import matching leads into a contact list. Use when the user wants to find new prospects, build a target list from scratch, or source leads matching specific titles, locations, or companies.
---

# Creating a lead search

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

Lead search sources new prospects from the Topo lead database. It is a
multi-step journey with three mandatory human gates: filter review, spend
approval, and result review before import.

When to use this skill:

- **Discovering new prospects:** this skill (`create_lead_search`).
- **Searching existing workspace contacts:** `list_contacts` or `search`.
- **Ingesting people you already have:** `importing-and-starting-outreach`
  (`import_leads` / `add_contact_list_members`).

## Workflow

1. `create_lead_search(description="...", contact_list_id=...)` →
   `lead_search_id`, `name`, `filters`, `state` (`draft`), `url`, `next_step`.
2. **Gate 1 (filter review):** present the generated `filters` summary
   verbatim and wait for confirmation or feedback.
   - Changes requested: `refine_lead_search(lead_search_id, instruction="...")`
     → updated `filters` and `changed`. Show the new summary and wait again.
   - Existing drafts: `list_lead_searches(state=draft)`.
3. Choose the execution path:
   - **Preview (cheap test):** `preview_lead_search(lead_search_id)` runs a
     capped batch of up to 25 leads. No spend confirmation needed.
   - **Full run (credit spend):** ask the user to approve sourcing up to
     `max_results` leads and confirm the credit spend.
4. **Gate 2 (spend confirmation):**
   `run_lead_search(lead_search_id, max_results=N, confirmed_by_user=true)`.
   Never pass `confirmed_by_user=true` unless the user agreed in their own
   words. Both `preview_lead_search` and `run_lead_search` return immediately
   with `run_id`, `state=running`, and `poll_after_ms`.
5. **Polling:**
   - Call `get_lead_search(lead_search_id)` after waiting `poll_after_ms`
     (escalates 5s → 15s → 60s).
   - Read `progress` (`entries_fetched`, `total_available`, `step`).
   - Still `running` after a few rounds: share the `url` so the user can watch
     progress in Topo, and stop polling until they ask for an update.
   - Session dropped: `get_lead_search(lead_search_id)` alone recovers the
     full state.
6. **Gate 3 (result review):** when `state=results_ready`, inspect `sample`
   (up to 5 leads in `get_lead_search`) or page through results with
   `list_lead_search_results(lead_search_id, page=1, size=25)`. Each result
   carries `lead_key` (LinkedIn URL or email). Never dump large result sets
   into the conversation; review in small pages or via `url`.
7. **Import into a contact list:**
   `import_lead_search_results(lead_search_id, ...)` → `contact_list_id`,
   `created`, `skipped_duplicates`.
   - **Selection (mutually exclusive):** `select_all=true` (server-side import
     of every result, capped at 5,000 over MCP) or explicit `lead_keys=[...]`.
   - **Destination (mutually exclusive):** `contact_list_id` for an existing
     list, `new_list_name` to create one, or neither to use the draft's
     destination list.
   - Optional: `enrich_phones=true` for phone enrichment on imported contacts.
8. **After import:** `state` becomes `imported`. Enroll with
   `importing-and-starting-outreach` (`enroll`).

## Failure handling

- **Failed runs:** `get_lead_search` reports `state=failed` with the reason in
  `status_message` and `progress.error`. Refine with `refine_lead_search` and
  re-run.
- **Searches owned by a signal source:** only searches against the Topo lead
  database run over MCP. Other searches are rejected with an error that links
  to the Topo web app; direct the user there.

## Rules

- Never set `confirmed_by_user=true` on `run_lead_search` without explicit
  user confirmation.
- Always show the filter summary and wait for approval before running.
- `preview_lead_search` fetches at most 25 leads and needs no spend
  confirmation.
- Do not dump hundreds of leads into chat: page with `size<=25` or share the
  `url`.
- `select_all=true` and `lead_keys` are mutually exclusive in
  `import_lead_search_results`; so are `contact_list_id` and `new_list_name`.
