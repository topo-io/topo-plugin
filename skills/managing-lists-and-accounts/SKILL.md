---
name: managing-lists-and-accounts
description: Organize the Topo workspace's records — browse and create accounts, keep account and contact custom fields up to date, build contact and account lists from names, domains, or a CSV, set a list's default sequence, and clean up members. Use when the user wants a target account list, their account portfolio, a company added, a list renamed, merged, cleaned, or deleted, or a field written onto a contact or account.
---

# Managing lists and accounts

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

A **contact** is a person in the workspace, an **account** is a company.
A **contact list** groups contacts and can enroll its new members into a
default sequence; an **account list** groups companies and feeds signals,
AI variable runs, and lead searches. Starting outreach on named people is
`importing-and-starting-outreach`; finding new people is
`creating-a-lead-search`.

## Accounts

- **Browse:** `list_accounts(query=…, owner_email=…)` — `owner_email` gives
  one member's portfolio. Page with `cursor` from `next_cursor`.
- **Open one:** `get_account(account_id | domain | name, exact_name?)` —
  owner, lead, sequence and hot-lead counts, and the contact roster. Then
  `list_contacts(account_id)` for everyone.
- **Add one:** `find_or_create_account(domain, name?)` — matched on domain,
  so it never duplicates.
- **Research one:** `enrich_account(domain)` — a lookup, not a write.
- **Write custom fields:** `update_account_record(account_id, variables)`
  merges name/value pairs; fields you do not list are kept. They stay in
  Topo and are usable in copy and AI prompts. A CRM write is
  `updating-the-crm`.
- **Change the owner:** `list_organization_members(query=…)` →
  `assign_owner(target=account, account_id, strategy=fixed, user_id)`.

## Contacts

- `list_contacts(query | email | account_id | job_title_contains)` or
  `search(query, types=["lead"])` → `fetch` → `contact_id`.
- `create_or_update_contact(email, …, company_domain?)` — matched on email;
  a domain links or creates the account.
- `update_record(contact_id, variables, email?)` — merges custom fields.
  `email` only fills an empty one; it never overwrites an identity. Check
  `updated` before claiming a change.

## Contact lists

| The user wants | Call |
|---|---|
| See lists | `list_contact_lists(search=…)` or `get_list(name=…)` (also returns the lead search feeding it) |
| See who is on one | `list_list_members(list_kind=CONTACT, list_id)` — page while `has_more` |
| New list | `create_contact_list(name, default_sequence_template_id?)` |
| Add people | `add_contact_list_members(contact_list_id, items=[{contact_id}] or [{email, first_name, …}])`, up to 100 per call |
| Add a CSV | `import_csv(attachment_id or csv_content, destination=CONTACT_LIST, name)` — fills in the background |
| Rename, or auto-enroll new members | `update_contact_list(contact_list_id, name?, default_sequence_template_id?)`; `clear_default_sequence_template=true` unlinks |
| Remove someone | `remove_contact_list_member(contact_entry_id)` with the member's `entry_id` |
| Delete the list | `delete_contact_list(contact_list_id)` |

## Account lists

| The user wants | Call |
|---|---|
| See lists | `list_account_lists(search=…)` or `get_list(name=…)` |
| See the companies | `list_list_members(list_kind=ACCOUNT, list_id)` |
| New list | `create_account_list(name)` |
| Add companies | `add_account_list_members(account_list_id, items=[{name, domain}])`, up to 100 per call |
| Add a CSV | `import_csv(…, destination=ACCOUNT_LIST, name)` — a domain or website column |
| Rename | `update_account_list(account_list_id, name)` |
| Remove a company | `remove_account_list_member(account_entry_id)` — the account stays in the workspace |
| Delete the list | `delete_account_list(account_list_id)` |

An account list is used by `setting-up-a-signal` (`account_list_ids`
filter) and `creating-ai-variables` (`account_list_id` scope).

## Cleaning up

- To subtract people from a list, remove their entries; never start a new
  lead search to do it.
- Merging two lists: read the members of one, `add_contact_list_members`
  with their `contact_id`s into the other, then delete the first only if the
  user asked.
- Before deleting a list, say how many members it has and whether it has a
  default sequence or feeds a signal (`list_signals`).

## Rules

- Confirm before `add_contact_list_members` into a list with a default
  sequence, `update_contact_list` setting a default sequence (new members
  then start outreach), `remove_contact_list_member` (it stops the sequences
  that list started), every delete, `import_csv`, `update_record`, and
  `update_account_record`.
- Never pass a list `entry_id` to `enroll` or `unenroll`, or a `contact_id` to
  a remove call.
- `get_list.list_kind` (`CONTACT` / `ACCOUNT`) is the value `list_list_members`
  takes.
