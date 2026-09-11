---
name: creating-a-lead-search
description: Source new leads from criteria using Topo lead search, preview or run the search with explicit user confirmation, and import matching leads into a contact list. Use when the user wants to find new prospects, build a target list from scratch, or source leads matching specific titles, locations, or companies.
---

# Creating a lead search

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

Lead search sources new prospects from the Topo lead database. It is a multi-step
journey with three mandatory human gates: filter review, spend approval, and
result review before import.

When to use this skill:

- **Discovering new prospects:** Use this skill (`create_lead_search`).
- **Searching existing workspace contacts:** Use `list_contacts` or `search`.
- **Ingesting people you already have:** Use `importing-and-starting-outreach` (`import_leads` / `add_contact_list_members`).

## Workflow

1. `create_lead_search(description="...", contact_list_id=...)` -> `lead_search_id`,
   `name`, `filters`, `state` (`draft`), `url`, `next_step`.
2. **Gate 1 (Filter review):** Present the generated `filters` summary verbatim to the user
   and wait for their confirmation or feedback.
   - If changes are requested: `refine_lead_search(lead_search_id, instruction="...")` ->
     returns updated `filters` and `changed`. Show the new summary and wait again.
   - To inspect existing drafts: `list_lead_searches(state=draft)`.
3. Choose execution path:
   - **Preview (cheap test):** `preview_lead_search(lead_search_id)` -> runs a capped
     batch of up to 25 leads. Requires no user confirmation.
   - **Full run (quota spend):** Ask the user to approve sourcing up to `max_results` leads
     and confirm provider credit spend.
4. **Gate 2 (Spend confirmation):** Call `run_lead_search(lead_search_id, max_results=N, confirmed_by_user=true)`.
   Never pass `confirmed_by_user=true` unless the user explicitly agreed in their own words.
   Both `preview_lead_search` and `run_lead_search` return immediately with `run_id`,
   `state=running`, and `poll_after_ms`.
5. **Polling contract:**
   - Call `get_lead_search(lead_search_id)` after waiting `poll_after_ms` (escalates 5s -> 15s -> 60s).
   - Read `progress` (`entries_fetched`, `total_available`, `step`).
   - If still `running` after a few polling rounds: share the `url` so the user can watch progress
     in Topo, and stop polling until they ask for an update.
   - If the session disconnects or drops: `get_lead_search(lead_search_id)` alone recovers the full state.
6. **Gate 3 (Result review & Import):**
   - When `state=results_ready`: inspect `sample` (<=5 leads in `get_lead_search`) or page through
     results with `list_lead_search_results(lead_search_id, page=1, size=25)`.
   - Each result entry contains `lead_key` (LinkedIn URL or email).
   - Never dump large result sets into the conversation context; review in small pages or via `url`.
7. **Import selection into a contact list:**
   `import_lead_search_results(lead_search_id, ...)` -> returns `contact_list_id`, `created`, `skipped_duplicates`.
   - **Selection mode (mutually exclusive):** Set `select_all=true` (server-side import of all results,
     capped at 5,000 over MCP) OR pass explicit `lead_keys=[...]`.
   - **Destination list (mutually exclusive):** Pass `contact_list_id` to add to an existing list,
     `new_list_name` to create a new list, or leave both empty to use the draft's destination list.
   - Optional: `enrich_phones=true` for phone enrichment on imported contacts.
8. **Next steps after import:**
   - State becomes `imported`.
   - Proceed to enrollment using `importing-and-starting-outreach` (`enroll`).

## Failure handling and provider rules

- **Failed search runs:** `get_lead_search` reports `state=failed` with the failure reason in `status_message`
  and `progress.error`. Refine criteria with `refine_lead_search` and re-run.
- **Non-lead-database searches:** Only `DataProvider.LEAD_DATABASE` searches are supported over MCP.
  Searches owned by a signal source (e.g. `THEIRSTACK`) are rejected with an error pointing
  to the Topo web app URL. Direct the user to manage them in Topo.

## Rules

- Never set `confirmed_by_user=true` on `run_lead_search` without explicit user confirmation.
- Always display filter summaries to the user and wait for their approval before running a search.
- `preview_lead_search` fetches at most 25 leads and requires no spend confirmation.
- Do not dump hundreds of leads into chat: page results with `size<=25` or provide the `url`.
- `select_all=true` and `lead_keys` are mutually exclusive in `import_lead_search_results`.
- `contact_list_id` and `new_list_name` are mutually exclusive in `import_lead_search_results`.
