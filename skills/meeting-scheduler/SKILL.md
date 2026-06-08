---
name: meeting-scheduler
description: "Handles end-to-end meeting scheduling via the Superhuman Mail MCP server — resolves contacts, checks calendar availability across time zones, detects scheduling conflicts, books calendar events, drafts proposal emails, creates recurring meetings, and blocks focus time. Use this skill whenever someone asks to \"schedule a meeting with [person]\", \"find a time to meet\", \"book a call\", \"set up a meeting\", \"when am I free to meet with [person]\", \"propose times to [person]\", \"send my availability\", \"create a meeting invite\", \"schedule a 1:1\", \"find overlap in our calendars\", \"reschedule my meeting with [person]\", \"set up a recurring sync\", \"block time for [task]\", or when an email thread involves scheduling and the user wants to act on it."
---

# Meeting Scheduler

## Step 1: Parse the request

Extract from the user's prompt:
- **Who** — names or email addresses
- **When** — date range, or "next week", "this afternoon", etc.
- **How long** — duration (default 30 minutes)
- **Type** — video call, in-person, phone (follow the user's Superhuman personalization settings for defaults)
- **Constraints** — "not before 10am", "mornings only", "avoid Friday"

Infer reasonable defaults before asking. Most users want a 30-minute meeting in the next week.

### Step 1a: Resolve the user's own email

Call `Superhuman_Mail.query_email_and_calendar` (e.g., "What is my email address?") to determine the user's email. Required for availability checks and calendar invites.

### Step 1b: Resolve participant identity

Resolve each name to the **correct** person before checking availability — a first name alone may match multiple contacts.

1. **Query broadly.** Use `Superhuman_Mail.query_email_and_calendar`: "List all contacts named Andrew with their full names, email addresses, and companies." Do **not** query "What is Andrew's email?" — this may silently return the wrong person.
2. **Handle ambiguity:**
   - **One match:** Confirm full name and email with the user before proceeding.
   - **Multiple matches:** Present all matches with full names, emails, and companies — ask the user to pick.
   - **No matches:** Ask for the person's full name or email directly.
3. **Use the confirmed email** for all subsequent steps. Never proceed with an unverified match.

## Step 2: Check availability

Call `Superhuman_Mail.get_availability` with:
- `participants`: verified emails from Step 1b — **always include the user** so their calendar is checked too
- `start_date` / `end_date`: time window in RFC3339 format
- `duration_minutes`: meeting length
- `working_hours_only`: true (unless user specified otherwise)

### Step 2b: Cross-timezone working hours filter

The API's `working_hours_only` flag only enforces the **user's** timezone. Ensure proposed slots fall within 9am–5pm for **every participant** in their local timezone.

1. **Look up each participant's timezone** via `Superhuman_Mail.query_email_and_calendar`. If unavailable, ask the user.
2. **Convert and filter.** Discard any slot outside 9am–5pm for anyone. Example: 9:00am ET = 6:00am PT — exclude if any participant is Pacific.
3. **If all slots are filtered out**, tell the user and offer to widen the window, shorten duration, or relax working-hours for specific participants.

## Step 3: Present options

Show 3–5 available slots clearly. Always display times in the user's timezone; for cross-timezone meetings, show conversions (e.g., "10:00am PT / 1:00pm ET").

```
## Available times for a 30-min call with Andrew Chen

1. Tuesday, April 14 at 10:00am PT
2. Tuesday, April 14 at 2:30pm PT
3. Wednesday, April 15 at 11:00am PT
4. Thursday, April 16 at 9:00am PT

Would you like me to:
- **Book one of these** directly (creates a calendar invite)?
- **Email Andrew** with these options so he can pick?
```

If no slots are available, suggest widening the window or adjusting duration.

## Step 4a: Direct booking

Call `Superhuman_Mail.create_or_update_event` with:
- `title`: Infer from context (e.g., "Lorilyn <> Andrew — Sync") or ask
- `start` / `end`: Selected slot in RFC3339 format
- `attendees`: All participants — **always include the user** unless they explicitly opt out
- `conference`: follow Superhuman personalization settings; override only if user explicitly requests or declines a video link
- `timezone`: User's timezone
- `description`: If the meeting relates to an email thread, include a brief summary
- `recurrence`: For recurring meetings, use RRULE (e.g., `RRULE:FREQ=WEEKLY;COUNT=10`)

Confirm: "Done — I've sent a calendar invite to Andrew for Tuesday at 10am PT with a video link."

**Never create a calendar event without explicit user confirmation.**

## Step 4b: Email with proposed times

Call `Superhuman_Mail.create_or_update_draft` with:
- `type`: "new" (or "reply" if in an existing thread)
- `to`: The other person's email
- `thread_id`: Include if replying to a scheduling thread
- `instructions`: e.g., "Propose meeting times to Andrew. Suggest [time 1], [time 2], [time 3] for a 30-minute video call. Keep it friendly and brief."

Present the draft for review, then send via `Superhuman_Mail.send_draft` when approved.

## Step 4c: Time blocking (solo tasks)

Call `Superhuman_Mail.create_or_update_event` with:
- `title`: Task description (e.g., "Deep work: Q3 proposal")
- `start` / `end`: Chosen block
- No attendees, `conference`: false

## Handling scheduling threads

When the user's prompt is in context of an existing scheduling email thread:

1. Read the thread via `Superhuman_Mail.get_thread`
2. Check availability via `Superhuman_Mail.get_availability` for proposed times
3. Draft a reply via `Superhuman_Mail.create_or_update_draft` — confirm a time, propose alternatives, or share availability

When meeting context is available from an email thread or deal, weave it into the calendar event description and scheduling email.
