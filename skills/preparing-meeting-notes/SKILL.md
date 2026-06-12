---
name: preparing-meeting-notes
description: Use this skill to create Obsidian event notes for the day's calendar meetings, each pre-populated with grounded talking points. Triggers when the user asks to prepare for their meetings, create notes for today's calendar events, set up meeting notes, or get ready for the day. Pulls the events they have accepted, creates one note per meeting following the vault's event conventions, and fills a prep section with talking points grounded in prior meeting notes, related project notes, recent GitHub activity, and Slack threads. Use it for daily meeting prep even when the user doesn't mention Obsidian or notes by name.
compatibility: Requires a calendar source the agent can query (for example Microsoft 365 via WorkIQ, or another calendar tool) and read/write access to an Obsidian vault. Talking-point research optionally uses the gh CLI for GitHub and a Slack search tool; both are skipped gracefully when unavailable.
---

# Preparing Meeting Notes

Create one Obsidian event note per calendar meeting for a given day, each pre-populated with a short list of suggested talking points grounded in real context. Run this as morning prep: the meetings haven't happened yet, so capture what's worth raising, not what was decided.

This is additive work. It only creates new notes and never overwrites existing ones.

## Step 1: Find the day's accepted meetings

Default to today. If the user names a date, use that instead. Format dates as `YYYY-MM-DD`.

Query the available calendar source for that day's events. Include only meetings the user has **accepted** (RSVP'd yes). Exclude:

- Events declined, marked tentative, or with no response
- All-day events
- Personal focus blocks, holds, and reminders with no other attendees, unless one clearly represents a real meeting

If no calendar source is available, stop and tell the user. There's nothing to do without one.

## Step 2: Convert every time to the user's local timezone

Calendar sources frequently return times in a different timezone from the user's own (for example UTC, or a US timezone for a user based elsewhere). Convert every start and end time to the user's local timezone *before* writing it into a note.

Sanity-check the result against the current local time and the meeting's expected slot. A meeting that lands at an implausible hour is the signal that a conversion was missed. This is the single most common mistake, so get it right.

## Step 3: Decide where notes live, and skip duplicates

Match the vault's existing convention for event notes. Look for an existing `Events/` (or similarly named) folder and a date-based structure such as `Events/YYYY/MM/DD/`. If a clear convention exists, follow it. Otherwise default to `Events/YYYY/MM/DD/<Meeting Name> (YYYY-MM-DD).md` and create the folders as needed.

Before creating anything, check whether a note for that meeting already exists in the day's folder (match on the meeting name). If it does, skip it. Never overwrite an existing note.

## Step 4: Create the note

If the vault has an event template (commonly `Templates/Event.md`), follow its frontmatter and structure exactly. Otherwise use this layout:

```markdown
---
date: 2026-06-12T10:30:00
dateLink: "[[2026-06-12]]"
people:
organizer:
related:
tags:
---
# <Meeting Name> (YYYY-MM-DD)

## Purpose

## Suggested talking points

## Discussion

## Actions
```

Fill the frontmatter from real data only:

- `date`: the meeting's local start time, full datetime.
- `dateLink`: a `"[[YYYY-MM-DD]]"` wikilink to the day, for linking to a daily note.
- `people`: wikilinks to the other attendees. Check the vault's people folder (commonly `People/`) for matching notes and link with the exact filename, using a display alias where the filename carries a handle, e.g. `"[[Jane Doe (@janedoe)|Jane Doe]]"`. Link only people who already have a note; list the rest as plain text.
- `organizer`: a wikilink to whoever organised the meeting. If the user organised it, follow the template's convention (often the user's own note, or left blank). Do not guess.
- `related` (or whatever the template calls it, e.g. `project`): wikilinks to obviously related project or topic notes. Leave blank when unclear. Do not invent links.
- `tags`: leave blank unless an obvious tag applies.

Leave `## Purpose`, `## Discussion`, and `## Actions` empty for the user to fill in during or after the meeting. Only `## Suggested talking points` is populated ahead of time.

## Step 5: Research and write the talking points

Populate `## Suggested talking points` with a short bullet list of prep points. Every point must be grounded in real, citable context. Never invent one. A good talking point traces back to something concrete: an open action from the last sync, a project's current status, or a recent piece of activity.

Draw on these sources, in priority order. Skip any that aren't available rather than blocking.

### Vault (primary)

This is the richest and most reliable source.

- Search the events folder for prior notes from the same or a recurring meeting, matching on a similar name or the same attendees.
- Surface any open, unchecked action items (e.g. unticked `- [ ]` tasks or `#Task` items) and any "to discuss" threads left hanging in those notes.
- Check related project notes and the attendees' people notes for current status, open questions, or commitments.

### GitHub (if the gh CLI is available)

- Use `gh` to find the user's recent activity tied to the meeting's topic or attendees: open PRs and issues they authored or are assigned to, review requests, and items clearly related to the linked projects.
- Prefer things updated in roughly the last two weeks. Read enough of an item (`gh pr view`, `gh issue view`) to describe why it's worth raising.

### Slack (if a Slack search tool is available)

- Search for recent threads (roughly the last week or two) involving the attendees or the meeting's topic that have an open question or a pending decision.
- Read the surrounding thread before deciding it's worth a talking point.

### Writing the points

- One line each, plain and specific.
- Link the source wherever possible: a `"[[wikilink]]"` to the vault note, or a markdown link to the GitHub item or Slack thread.
- Keep the list short: the few things genuinely worth raising, not an exhaustive dump.
- If nothing relevant turns up across all sources, write a single line and move on: `_No prep points found. Add your own._` Do not pad with generic filler.

## Step 6: Summarise

After processing all meetings, give the user a short summary: which notes were created (and how many talking points each got), and which meetings were skipped and why (already had a note, declined, etc.).

## Gotchas

- **Timezones are the main trap.** Calendar sources often report times in a different timezone from the user's. Always convert to local time and sanity-check before writing. See Step 2.
- **Never fabricate.** These notes are made before the meetings happen. `## Discussion` and `## Actions` stay empty, and every talking point must come from a real, citable source. An empty prep section is better than an invented one.
- **Never overwrite.** Only create new notes. If a note for a meeting already exists, skip it, since the user may have already started editing it.
- **Match the vault, don't impose a structure.** Folder layout, the event template, frontmatter field names, and people-note filename conventions vary between vaults. Detect and follow what already exists; only fall back to the defaults here when there's no existing convention.
- **Filter on RSVP status, not just presence on the calendar.** A calendar is full of declined invites, tentative holds, and focus blocks. Only accepted meetings with real attendees should get a note.
- **Degrade gracefully.** GitHub and Slack are optional enrichment. If their tools aren't available or return nothing, fall back to vault-only talking points rather than failing.
