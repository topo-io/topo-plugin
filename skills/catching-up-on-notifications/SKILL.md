---
name: catching-up-on-notifications
description: Catch the signed-in user up on their own Topo notification center — what they missed, their latest or unread notifications grouped by category, then mark them read or archive them. Use when the user asks "what did I miss?", "what's new in Topo?", for their latest notifications or unread count, or wants notifications marked read or archived.
---

# Catching up on notifications

Read the `ids-and-outcomes` skill ([SKILL.md](../ids-and-outcomes/SKILL.md)) first.

The notification center belongs to one person: these tools read and change
only the signed-in user's notifications, with the same feed, filters, read
state and archive as the Topo app. They need an OAuth sign-in: on an
organization API key (`whoami.user_id` is null) they refuse, so tell the user
to connect with their Topo account.

## Workflow

1. **What did I miss?** `summarize_missed_notifications` — the unread count,
   the unread notifications grouped by `category` and `event_type` with their
   counts, and the most recent ones in `latest`. Pass `since` (timezone-aware)
   when the user names a period. Answer as a short digest: the count first,
   then one line per category, largest first, naming who or what is behind
   the most recent items. Say so when `has_more` is true.
2. **Latest or filtered notifications.** `list_notifications` — one page,
   latest first. `state=UNREAD` keeps what the user has not read
   (`total_count` is then the unread count), `ALL` is the feed without the
   archive, `ARCHIVED` the archive only. Narrow with `categories`,
   `event_types`, `subject_kind` and `subject_id`, `resolved=false`, and
   `occurred_after` / `occurred_before`.
3. **Only mine.** `only_mine=true` leaves out what reached the user for a
   role or as a fallback recipient, like the app's "for me" toggle.
4. **Act on a notification.** Each item has a `title` (who did what), a
   `summary`, a `url` into the Topo app, and `contact_id` / `account_id` when
   it is about a lead or an account: continue with `get_contact`,
   `list_threads(contact_id)` or `get_account`.

## Marking and archiving

- Reading never marks anything read. Mark or archive only when the user
  asks.
- `mark_notifications_read(ids)` with ids from a list, or
  `mark_notifications_read(all_unread=true, confirmed_by_user=true)` only when
  the user asked to mark everything read (`only_mine=true` for only theirs).
  It cannot be undone: unless they explicitly asked for all of them, tell
  them how many it marks and wait for their yes before confirming. `read=false` with ids
  marks them unread again.
- `archive_notifications(ids)` takes them out of the feed and marks them
  read; `archived=false` puts them back.
- Both return `unread_count` and `total_count`: say what is left unread.
