---
name: creating-ai-variables
description: Create, test, run, and maintain Topo AI variables — prompt-computed AI columns on the Accounts and Contacts tables or a sequence's Leads tab, usable in sequence copy as {{variable_name}} — with a credit estimate before every run. Use when the user wants a new AI column or personalization variable, wants one run on a list or a sequence, asks to read its values, change its prompt, or why a value is wrong.
---

# Creating AI variables

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

An AI variable is a prompt computed per row. As a column it fills the
Accounts table (`ACCOUNT`), the Contacts table (`CONTACT`), or a sequence
template's Leads tab (`SEQUENCE`, one value per enrollment). Any AI variable
is usable in sequence copy as `{{variable_name}}`. It personalizes or
enriches; it never filters a lead search (that is `creating-a-lead-search`).

## Credits

| `output_type` | Cost per row computed |
|---|---|
| `TEXT`, `NUMBER`, `URL` | 1 credit |
| `BOOLEAN`, `SELECT`, `SCORE` | 0.1 credit |
| `web_search=true` | +1 credit |

A failed value is never charged. Pick the cheapest type that answers the
question: a yes/no or pick-one question is `BOOLEAN` or `SELECT`, not `TEXT`.
Turn on `web_search` only when the answer is not in the row's own data.

## Workflow

1. **Inventory.** `list_ai_variables(entity_type=…)`. An existing variable
   that almost fits is updated, not duplicated: sequence copy points at
   `variable_name`, so a near-duplicate leaves live copy on the old one.
2. **Draft.** `draft_ai_variable(entity_type, request)` turns the user's
   words into a proposed `name`, `variable_name`, `prompt`, `output_type`,
   `options` / `score_levels`, and `web_search`. Nothing is saved or charged.
   Review it with the user.
3. **Test before going live.**
   - A text personalization variable:
     `simulate_ai_variable(name, variable_name, prompt, dos, donts, examples,
     contact_ids, sender_id)` on several leads that differ (industry,
     seniority, company size). `sender_id` is a `senders[].id` from
     `list_sender_identities`, never the identity's own `id`. Free.
   - A table column: `create_ai_variable(…, auto_run=false)`, then
     `estimate_ai_variable_run(scope=ROWS, contact_ids | account_ids |
     sequence_ids)` on **5 rows**, state the credits, and
     `run_ai_variable(…, confirmed_by_user=true)` once the user agrees. Read
     the results with `get_ai_variable_values`. For one sequence template, set
     `entity_type=SEQUENCE` and `sequence_template_ids=[…]`.
4. **Show the sample**, adjust with `update_ai_variable` (partial; omitted
   fields keep their value), retest.
5. **Run on the rest.** `estimate_ai_variable_run(scope=ALL,
   only_missing=true)`, narrowed to exactly one of `contact_list_id`,
   `account_list_id`, or `sequence_template_id` when the user means one list
   or sequence. Show the rows, credits per row, total, and balance. Run with
   the same arguments and `confirmed_by_user=true` only after the user
   agrees. `ALL` stops at 1,000 rows (`capped=true`): to go further,
   estimate again with `backfill=true` and run with it only after explicit
   agreement on the full cost.
6. **Keep new rows computed** with `update_ai_variable(auto_run=true)` once
   the sample is approved. Share the returned `url` to watch the table fill.

## Reading values

`get_ai_variable_values(contact_ids | account_ids | sequence_ids,
ai_variable_ids?)` returns per row the `status` (`QUEUED`, `RUNNING`, `DONE`,
`ERROR`), the `value`, `reasoning` (generated types) or `confidence` and
`low_confidence` (classifications), web `sources`, and `outdated` when the
prompt changed since. Reading never spends credits. Re-run outdated rows with
`only_missing=true`.

## Writing a good prompt

- The prompt is rendered per row before it reaches the model, so
  `{{another_variable}}` inside it is valid. Check names with
  `list_personalization_variables`. Never use `[square brackets]` or a
  `sender_*` variable: values are computed before a sender is known.
- `TEXT` output goes into an email: ask for one clause or one sentence in
  `dos`, forbid greetings, sign-offs and quotation marks in `donts`, and give
  real `examples` — they pin length and register.
- A failed value renders as an empty string and the email still sends. Put
  the variable mid-sentence, never alone at the start of a subject line.

## Rules

- Never call `run_ai_variable` without an `estimate_ai_variable_run` with the
  same scope shown to the user, and never pass `confirmed_by_user=true` before
  they agree in their own words. Always state the credits, even when
  `priced=false`; never call a run free.
- `create_ai_variable` with `auto_run` on (the default for table columns)
  computes every new row within about five minutes and spends credits. Create
  with `auto_run=false` until the sample is approved, and confirm first.
- `variable_name` is fixed at creation.
- `delete_ai_variable` archives the variable and breaks live copy that uses
  it. Name what breaks and confirm first.
