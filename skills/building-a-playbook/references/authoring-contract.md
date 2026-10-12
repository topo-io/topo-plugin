# Playbook authoring contract

Everything sent to `edit_playbook_draft` is snake_case: structural keys
(`depends_on`, `node_id`, `output_fields`, `min_success_ratio`) and field
names alike. Input names, trigger payload fields and output fields are copied
verbatim from `get_playbook_building_blocks`; never invent one.

## Operations

One `edit_playbook_draft` call takes `playbook_id`, `expected_revision` and up
to 50 `operations`, applied in order and saved as one revision. If one cannot
apply, nothing is saved.

| `op` | Fields | Effect |
|---|---|---|
| `set_name` | `name` | Renames the draft. |
| `set_description` | `description` (null clears) | Saved at once, not versioned. |
| `set_trigger` | `trigger` | Replaces the trigger binding. |
| `set_policy` | `policy` | Only the given fields change; quotas merge cap by cap. |
| `set_parameter_requirements` | `parameters` | Replaces the Vault requirement list. |
| `upsert_node` | `node`, optional anchor | Adds the step, or replaces the step with the same id **in full**. |
| `set_node_inputs` | `node_id`, `inputs` | Merges inputs; a null value removes one. |
| `move_node` | `node_id`, anchor | Moves a step. |
| `remove_node` | `node_id` | Removes a step. |
| `replace_definition` | `definition` | Replaces the whole draft. Only for a new playbook or a rewrite. |

An anchor is `after_node_id` or `before_node_id` (not both), optionally with
`parent_map_node_id` to place the step inside a map. Without an anchor a new
step goes last. Position is display order only: execution order comes from
dependencies.

## Definition

```json
{
  "name": "…",
  "trigger": {"type": "INBOUND_REPLY", "categories": ["INTERESTED"]},
  "policy": {"stop_on_first_error": false, "max_runs_per_day": 100, "quotas": {"enrolls_per_day": 50}},
  "parameters": [{"key": "hot_list_id", "type": "VALUE", "description": "…", "required": true, "option_source": "contact_lists"}],
  "nodes": []
}
```

- **Trigger** — `type` plus that type's config fields, as listed by
  `get_playbook_building_blocks`: `SIGNAL`, `SCHEDULE`, `EVENT` (a
  first-party event), `INBOUND_REPLY` (`categories`), `LEAD_IMPORT_COMPLETED`
  (`contact_list_id`), `MANUAL`, or `CSV`. Every id comes from
  `list_playbook_field_options`. A `MANUAL` trigger may declare the inputs a
  run asks for: `fields: [{"key", "label", "type":
  "string|number|boolean|email|url", "required", "description"}]`, read as
  `trigger.<key>`.
- **Policy** — `dry_run`, `require_approval_for_irreversible`,
  `max_concurrency` (1–20), `stop_on_first_error`, `max_runs_per_day` (null
  removes the cap), and `quotas`: `enrolls_per_day`,
  `notifications_per_day`, `messages_per_day`, `enrichments_per_day`,
  `sourced_leads_per_day`. Set a quota for every metered action you add.
- **Parameters** — the Vault items the playbook reads: `key`
  (`^[a-z][a-z0-9_]*$`), `type` `VALUE` or `CREDENTIAL`, `description`,
  `required`, and `option_source` when the value is the id of a Topo object
  (`contact_lists`, `sequence_templates`, …) so the editor offers a picker.

## Mappings

Every step input is one mapping object, discriminated by `kind`:

| Mapping | Reads |
|---|---|
| `{"kind": "constant", "value": <any JSON>}` | A literal. |
| `{"kind": "trigger", "field": "<payload field>"}` | A trigger payload field. |
| `{"kind": "node", "node_id": "<id>", "field": "<output field>"}` | An earlier step's output; **creates a dependency** on that step. |
| `{"kind": "parameter", "key": "<vault key>"}` | A Vault item declared in `parameters`. |
| `{"kind": "item", "field": "<field or null>"}` | The current map item; **only inside a map**. |
| `{"kind": "expression", "code": "<python>"}` | A sandboxed Python expression over `trigger`, `nodes`, `parameters`, `item`, e.g. `int(nodes["company"]["employee_count"])`. `nodes["<id>"]` creates a dependency. |
| `{"kind": "composite", "fields": {"<name>": <mapping>}}` | An object built field by field. |

`trigger`, `node` and `item` mappings accept a `path` list to read inside a
nested value. Inside a constant string, `{{trigger.<field>}}` and
`{{<node_id>.<field>}}` are filled at run time. A token that cannot be filled
stays as-is, except in an irreversible step (a send, an enrollment…), which
fails instead: fix every `UNRESOLVED_TOKEN` warning first.

For a map-typed input (custom fields, CRM properties by name) use a composite
whose keys are the map's keys. For a list of `{property, value}` rows
(`update_crm_field` `fields`) use a constant list of rows.

## Step kinds

Every step has `id`, `kind`, a plain-words `title`, `depends_on`, and exactly
the payload of its kind:

- **`CAPABILITY`** — `capability` (an action from the building blocks) and
  `inputs` keyed by its input names.
- **`CODE`** — `code: {"source": "<python>"}` and `output_fields`
  (`[{"name", "type": "string|number|boolean|string_list|json"}]`), no
  `inputs`. The source reads `steps.<node_id>.<field>`, `trigger.<field>`,
  `vault.<key>` and `item.<field>` (inside a map); a missing value reads
  `None`. No network or filesystem. The result is the **last top-level
  expression**: an object whose keys are exactly the `output_fields`. Use it
  for any deterministic transformation (scoring, parsing, reshaping a list).
- **`CONDITION`** — `predicates` (Yes / Else), or `cases`
  (`[{"id": "case_1", "title": "…", "predicates": [...]}]`) for if / else if
  / else; the first matching case wins. A predicate is
  `{"source": <mapping>, "operator": "<OPERATOR>", "value": <JSON>}` with an
  operator from the building blocks (`EQ`, `NEQ`, `GT`, `GTE`, `LT`, `LTE`,
  `IN`, `NOT_IN`, `CONTAINS`). `value_from` (a mapping) compares with a value
  read at run time instead.
- **`MAP`** — `map: {"items": <mapping to a list>, "nodes": [...],
  "min_success_ratio": 0.0–1.0}`. No `MAP`, `WAIT` or `APPROVAL` inside a map.
- **`BATCH`** — `batch` with at least one of `max_items` (≥1),
  `max_age_seconds` (≥60), `debounce_seconds` (≥10), `drain_cron` (5-field
  UTC cron). Optional `partition_by` keeps one batch per value. A step after
  a batch reads `trigger.batch_node_id` and `trigger.partition`, not the
  steps before it.
- **`WAIT`** — `wait` with exactly one of `duration_seconds` (≥1) or `until`
  (a mapping).
- **`APPROVAL`** — `approval: {"title": "…", "body": "…", "timeout_seconds":
  86400, "on_timeout": "skip|fail"}`. A live run waits for a person in the
  tasks inbox (a `PLAYBOOK_APPROVAL` task). Optional `assignee`
  (`{"type": "user", "user_id": <mapping>}` or `{"type": "owner_of",
  "owner_of": "contact|account|sequence", "record_id": <mapping>}`) and
  `editable_fields` (up to 10). Outputs `approved`, `comment`, `values`.

Any step may set `on_failure` (`stop` or `skip`). Only a `CODE` step or a
retryable action may set `retry` (`never`, `once`, `three_times`). Set these
only when asked, and keep existing values when editing.

## Guards

A guard runs a step on one branch only:

- **Branch of a Condition** — `{"condition_node_id": "<id>", "expected":
  true|false, "case_id": "<case id>"}`. Omit `case_id` for Yes / Else of a
  predicates-only Condition.
- **Value of an output** — `{"source_node_id": "<id>", "field": "<output
  field>", "equals": <value>}`, to fan sibling steps out of one categorizing
  step.

## Hard rules

- Step ids are snake_case and unique across the whole definition, nested
  steps included. Keep existing ids when editing.
- Every dependency names an existing step, with no cycle.
- `output_fields` appear only on `CAPABILITY` and `CODE` steps.
- When a value is not knowable from the trigger or an earlier step, declare
  a parameter instead of guessing.
- Irreversible actions (sends, enrollments, CRM writes) raise
  `IRREVERSIBLE_WITHOUT_APPROVAL` unless an approval step or
  `require_approval_for_irreversible` covers them. Reversible steps a person
  should review (CRM field updates, owner assignment, unenrolling) raise
  `APPROVAL_REQUIRED_WITHOUT_GATE` unless an approval step precedes them.

## Example — interested replies above a fit score go to a list

Request: "When a reply is categorized as interested and the lead scores above
80, add them to my Hot replies list." Ground first:
`get_playbook_building_blocks(capability_names=["qualify_lead",
"add_contact_list_members"])`, then `list_playbook_field_options` with
`inbound_reply_categories` and `contact_lists`. The list id below is a
placeholder for the returned value.

```json
[
  {"op": "set_name", "name": "Interested replies to Hot replies"},
  {"op": "set_description", "description": "Scores every interested reply against the ICP and adds the strong fits to Hot replies."},
  {"op": "set_trigger", "trigger": {"type": "INBOUND_REPLY", "categories": ["INTERESTED"]}},
  {
    "op": "upsert_node",
    "node": {
      "id": "score",
      "kind": "CAPABILITY",
      "title": "Score the lead against the ICP",
      "capability": "qualify_lead",
      "inputs": {"contact_id": {"kind": "trigger", "field": "contact_id"}}
    }
  },
  {
    "op": "upsert_node",
    "node": {
      "id": "high_score",
      "kind": "CONDITION",
      "title": "Score above 80",
      "predicates": [
        {"source": {"kind": "node", "node_id": "score", "field": "score"}, "operator": "GT", "value": 80}
      ]
    }
  },
  {
    "op": "upsert_node",
    "node": {
      "id": "add_to_list",
      "kind": "CAPABILITY",
      "title": "Add the lead to Hot replies",
      "capability": "add_contact_list_members",
      "guard": {"condition_node_id": "high_score", "expected": true},
      "inputs": {
        "contact_list_id": {"kind": "constant", "value": "<contact_list_id from list_playbook_field_options>"},
        "items": {"kind": "constant", "value": [{"contact_id": "{{trigger.contact_id}}"}]},
        "apply_list_default_sequence": {"kind": "constant", "value": false}
      }
    }
  }
]
```

Then test `score` and `high_score` with `test_playbook_step`, and run two
end-to-end tests with `forced_branches` `{"high_score": "yes"}` and
`{"high_score": "__else__"}`.
