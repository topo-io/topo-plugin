---
name: configuring-the-workspace
description: Maintain what Topo knows about the user's own company and who it targets — the company profile, audiences (ICP filters), and knowledge base documents — and review the workspace setup (members, senders, mailbox deliverability, webhooks). Use when the user wants to update their positioning, competitors or pain points, define or edit an ICP audience, add, import, or fix a knowledge base article, check sender health, or send Topo events to another system.
---

# Configuring the workspace

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

Three records shape every qualification, lead search, and AI-written message
in Topo. Changing them changes downstream automation for the whole team, so
read first, show the change, and confirm.

| Record | Holds | Read | Write |
|---|---|---|---|
| Company profile | Value proposition, positioning, differentiators, competitors, pain points | `get_organization_context` | `update_organization_context` |
| Audiences | Who the organization targets, as saved lead-search filters | `list_audiences`, `get_audience` | `create_audience`, `update_audience`, `delete_audience` |
| Knowledge base | Markdown documents about the company: product, pricing, FAQs, policies, collateral | `list_knowledge_documents`, `search_knowledge_documents`, `get_knowledge_document` | `create_knowledge_document`, `update_knowledge_document`, `import_knowledge_document_url`, `delete_knowledge_document` |

## Company profile

1. `get_organization_context` — quote the current values of the fields you
   will touch.
2. Propose the new wording; on agreement,
   `update_organization_context(field=…)` with only the fields that change.
   `differentiators` and `competitors` are lists replaced as a whole.
3. Who the company targets is not in the profile: edit an audience instead.

## Audiences

- `list_audiences` → `get_audience(audience_id)` returns `flat_filters`,
  `filter_groups`, and the live matching contact and account counts.
- `create_audience(name, flat_filters, filter_groups)` — names are unique;
  the first audience becomes the default.
- `update_audience(audience_id, …)` replaces filters **as a whole**: pass
  `flat_filters` and `filter_groups` together, in the shape `get_audience`
  returned, or one side is cleared. `is_default=true` promotes it; the
  default cannot be unset.
- `delete_audience` — lead searches that already copied its filters keep
  them.
- When the user asks for a lead search or outreach without naming a target,
  ask which audience applies, then describe its filters in the
  `create_lead_search` description (`creating-a-lead-search`).

## Knowledge base

- **Answer from it.** `search_knowledge_documents(query)` then
  `get_knowledge_document` for every hit that could carry part of the answer.
  Name the documents you used. When they do not cover the question, say so;
  never present general knowledge as company knowledge.
- **Inventory before writing.** `list_knowledge_documents`; a close document
  is updated, not duplicated.
- **Write.** Show the proposed markdown, then `create_knowledge_document(title,
  content, description, tags)` or `update_knowledge_document(document_id, …)`
  (partial). For a public page or Google Doc,
  `import_knowledge_document_url(url)` instead of pasting it.
- **From another connector** the host has (a wiki, notes, call recordings):
  read a bounded set, distill it into 1–5 articles (no one-document-per-page
  dump, no personal data), show the drafts, and save them one by one.

## Workspace setup (read-mostly)

- `list_organization_members(query=…)` — roles and `user_id`s.
- `list_sender_identities` — identities and their `senders[]`.
- `list_sender_deliverability` — each mailbox's latest inbox-placement
  score (0–10) and band. For mailboxes Topo hosts, below 7 the daily sending
  limit is 0 until the score recovers. It holds no bounce or spam rates.
- Webhooks: `list_webhooks`, `get_webhook`;
  `create_webhook(name, url, event_types, sequence_template_ids?)` — the
  signing secret is returned **only** in that response: tell the user to
  store it, never repeat it later. `send_test_webhook(webhook_id)` checks the
  receiver. `update_webhook(status=PAUSED)` stops deliveries reversibly;
  `delete_webhook` revokes the secret.

## What stays in the Topo web app

Connecting a mailbox or a CRM, billing and credits, inviting or removing
members, notification settings, and opt-outs. Say so plainly; do not pretend a
workaround exists.

DNS records (SPF, DKIM) are not set in Topo at all: the user configures them
at their own DNS provider. Mailboxes bought through Topo come already
configured.

## Rules

- Confirm before `update_organization_context`, every audience write, every
  knowledge base write or import, and every webhook create, update, or
  delete.
- Never paste markdown for the user to copy into Settings: apply it with the
  tools above once they agree.
- Never put a secret, password, or personal data in a knowledge base document
  or a profile field.
