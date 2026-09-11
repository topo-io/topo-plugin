---
name: suppressing-or-releasing-targets
description: Block or lift emails and company domains on the Topo exclusion list, and stop any sequences already sending to those targets. Use when the user wants a do-not-contact, asks why a lead was skipped, adds a customer or competitor list, or wants an exclusion removed.
---

# Suppressing or releasing targets

Read [ids-and-outcomes](../references/ids-and-outcomes.md) first.

An exclusion blocks **future** import and enroll. It does not stop sequences
already running. Lifting an unsubscribe is a compliance decision.

## Workflow

1. Decide grain from the user's words: one email → `CONTACT`; whole company
   → `DOMAIN`. Never infer a domain from a personal Gmail.
2. `list_exclusions(kind=…, search=value)`. Already blocked? Report
   `reason`, `until`, `source`, `exclusion_id`. Do not duplicate.
3. "Why was this lead skipped?": `get_contact(email=…)` +
   `list_sequences(contact_id)` + the exclusion row. `get_contact`
   includes `account_id` and `account_domain` — use `account_domain`
   when they meant the whole company.
4. Check live outreach before writing. For a person: `list_sequences(contact_id)`.
   For a domain: `get_account(domain=…)` (or `get_contact.account_id`) →
   `list_contacts(account_id)` → `list_sequences` per `contact_id`.
5. Writes:
   - Future block: `add_exclusions(items=[{value, kind}], message=…)`.
     Always pass `message`. Result is `created_count` / `skipped_count`
     and `created[]` with `exclusion_id` + `kind` + `value`. Skipped
     duplicates are omitted from `created`.
   - Already in flight: `unenroll(contact_ids=[…])`. Not `refuse_leads`, not
     `stop_sequence(WIN)`.
   - Time-bound "bad timing" on one enrollment: prefer
     `holding-or-resuming-outreach` (`pause_sequence`). An exclusion with
     `update_exclusion(..., until=…)` also blocks new enrolls.
6. Lift: `list_exclusions` → `exclusion_id` + `kind` → `remove_exclusion`.
   Extra warning when `reason` is `UNSUBSCRIBED`, `BOUNCED`, or
   `CONTACT_REFUSED`.
7. CSV of emails/domains: `import_csv(destination=CONTACT_EXCLUSIONS|DOMAIN_EXCLUSIONS)`
   with `attachment_id` when the host uploaded a file, otherwise
   `csv_content` (header row included). Still `unenroll` anyone already
   enrolled.

## Rules

- Never exclude a domain when the user named one person, and never the reverse.
- Excluding a customer's domain also blocks expansion there. Say that.
- Cannot pull CRM "do not contact" from here. `EXCLUDE_FROM_CRM` rows are
  readable, not creatable as a CRM sync.
- The list is paginated. It is a lookup, not an export.
