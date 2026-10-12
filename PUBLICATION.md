---
title: Publish the Topo plugin
description: Prepare the public-review package, demonstration and reviewer access
owner: team
status: draft
last_verified: '2026-10-12'
expires: '2027-01-12'
tags: [mcp, plugins, publication]
---

# Publish the Topo plugin

The development connection and public directory listing are separate.
This package is prepared for review; it has not been submitted or published.

Author changes in `topo-io/agent/packages/topo-plugin/`. The public repository
`topo-io/topo-plugin` is a mirror, not the source of truth. Merge source changes
and let the existing publish workflow synchronize the mirror.

## Prepared metadata

| Item | Value |
| --- | --- |
| Package | `topo`, version `1.4.0` |
| Intended publisher | Topo, business identity; verification remains to be checked |
| Public integration | `https://mcp.topo.io/mcp`, Streamable HTTP, OAuth |
| Workflow skills | 20 (seven added in 1.4.0) |
| Distribution | All countries offered by the portal; `countries: []` removes restrictions |
| Commerce | No purchases, checkout or payment processing in the plugin |
| Review cases | Five positive and three negative, embedded in both OpenAI manifests |
| Release notes | Embedded under `extensions.com.openai.publication` |

The original icons are reused. The listing logo is a 612 × 612 PNG, and the
composer icon is a 512 × 512 PNG. Both are below 5 MiB.
The root manifest owns the OpenAI metadata; the compatibility manifest retains
the same review/publication values and listing.

## Remaining preparation

1. Verify the intended business identity and organization in OpenAI's portal.
   The company name in a manifest is not evidence of business verification.
2. Prepare a dedicated reviewer account and demo workspace with sample data.
   It must have permissions and subscription access for every case.
3. Record the walkthrough below and host an actual playable video at a
   reviewer-accessible URL. Add that verified URL to
   `extensions.com.openai.review.demo_recording_url` in both OpenAI manifests.
4. Validate the public listing URLs and policy coverage. The website, support,
   privacy and terms URLs are already present. Website, support and legal pages
   were accessible during preparation; explicit ChatGPT/MCP data handling and
   applicability still need confirmation. Verify support offers a working
   way to contact Topo. Do not invent retention, deletion or sharing commitments.
5. Run the complete cases on the demo account, then on the exact saved portal
   version. A connector smoke check does not replace a case execution.

The connector guide lives at `https://docs.topo.io/mcp-connector`. Mintlify
serves its own documentation MCP server at `docs.topo.io/mcp`, which returns
JSON, so never link that path. Check that the listing and data-source/removal
links open the human-readable page; a successful status is not enough.

## Demo account and fixtures

Use synthetic business data. Never use a production user account or real
prospect conversations in the recording.

Prepare:

- At least one sequence template and permission to create a separate draft.
- Signals enabled on the workspace, with one active hiring signal (for
  example "SDR hires") that enrolls into that sequence template and has
  already imported a few leads into its list. The preview in positive case 1
  queries the signal provider for real, so the workspace needs provider
  access, but it creates nothing and spends no credits.
- A pending task assigned to the reviewer; allow an empty-queue variant.
- Two sample inbound email threads, one interested and one out of office.
- One account named `Topo Review Acme`, two contacts, an enrollment, an email
  thread and an activity. Populate its company data without paid enrichment.
- No existing template named `Topo Review Draft` when running the draft case.
  Reset that fixture between runs without touching production records.

Use a supported login method that reviewers can complete independently.
For password login, a dedicated account must not depend on the creator's MFA,
phone or mailbox. For magic-link/email-code login, follow the portal's reviewer
email instructions. Put login credentials and access instructions only in
secure dashboard fields, never in this package, release notes, GitHub or chat.

## Record a 3–5 minute walkthrough

Use the existing development installation with this package's skills loaded.
First rehearse the five positive cases and three negative cases from
`extensions.com.openai.review.test_cases`.

| Segment | Show |
| --- | --- |
| Connection | Topo selected in ChatGPT, signed-in demo workspace, identity and sequence list |
| Signals | Active signals and what they imported, then a hiring-signal preview; nothing created |
| Tasks | Pending tasks grouped by type, without resolving any |
| Inbox | Actual inbound results, thread reads and draft responses; no sending |
| Account | The brief of Topo Review Acme, supported by returned contacts and conversations |
| Draft | Confirm creation, show the two email steps and PENDING_SETUP status; no activation or enrollment |
| Boundary | Request a calendar booking; show the unsupported-action explanation |

Capture real prompts and tool results with enough time to read them. Keep tokens,
passwords and unrelated windows off screen. Computer control or screenshots alone
do not produce a video: use a screen recorder. Computer-assisted walkthrough is
optional if a recording capability and demo session are available.

Play back the recording, verify its access without a private login, and add its
actual URL to the manifest. A script, mockup or unverified link is not a recording.

## Evidence recorded during preparation

| Check | Status | Evidence |
| --- | --- | --- |
| OAuth identity | Passed | Existing development connection returned authenticated user/workspace and scopes |
| Template discovery/detail | Passed | One-item list returned a cursor; detail read found the returned template |
| Contact-list discovery | Passed | Existing connection returned a paginated list |
| Personal pending tasks | Passed, empty boundary | Query for the signed-in user's pending tasks returned zero results |
| Source tool annotations | Passed, source only | All 175 tools in the inspected MCP contract explicitly declare the three boolean hints |
| Five positive review cases | Not run end to end | Dedicated reviewer fixtures and matching package installation still required |
| Three negative review cases | Not run | Draft expectations only; verify model behavior in the intended host |
| Video | Recorded | Unlisted walkthrough at <https://youtu.be/nA_XxfGYNSE>, set as `demo_recording_url` |
| Portal identity, scans and reviewer access | Not verified | Requires the intended submission organization and reviewer setup |

These are preparation results, not a public-review verdict. No outreach,
enrichment, credit spend, task mutation or template creation was performed in
the connected live workspace.

## Final portal setup

After preparation, open the Plugins page in the intended OpenAI organization.
Upload the complete ZIP as a draft, select the verified Topo identity, connect
the actual MCP server with OAuth, inspect imported metadata and run the cases
against that saved version.

Complete required scans and resolve findings. The MCP domain challenge endpoint
was reachable during preparation; verify the portal's own domain status.
The authorized developer completes legal/policy attestations. Submit for review;
after approval, publish the approved release as a separate action.

Use the current [submission documentation](https://developers.openai.com/plugins/deploy/submission)
and [remote MCP review requirements](https://developers.openai.com/plugins/deploy/app-review).
Do not place a development app ID or `.app.json` in the author-supplied public
upload. Keep the actual MCP endpoint and let the portal generate its bindings.

## Build and inspect

Archive one top-level directory named `topo`, containing the manifests, MCP
configuration, skills and referenced assets. Keep the ZIP outside that directory.
Exclude Git internals, credentials, dependency folders and symlinks.

Inspect the ZIP itself: no root `.app.json`, no non-null `apps` declarations,
20 skill frontmatters, synchronized versions, all icons included, five positive
and three negative cases, and supported publication values. Until the recording,
access and portal checks above are complete, label the archive as a review draft.
