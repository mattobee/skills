---
name: finding-contribution-opportunities
description: >
  Use this skill to find high-value places for the user to contribute and be more visible:
  conversations on GitHub and Slack where their expertise would add value (engage), plus
  their own in-flight work worth sharing (amplify). It searches, ranks, and drafts suggested
  wording in the user's voice, then surfaces it for review. It never posts, replies, or sends
  anything. Triggers when the user asks where they can add value, what to comment on, where
  to contribute, how to raise their visibility or profile, what they should weigh in on, what
  they've done that's worth sharing, or wants a daily or weekly round-up of opportunities.
  Use it even when they don't name GitHub or Slack directly.
compatibility: >
  Requires the gh CLI authenticated against GitHub. Slack search tools are optional
  (GitHub-only works fine). An optional notes source such as an Obsidian vault can be mined
  for shareable work when its location is provided. A scratch SQL store and an inbox/widget UI
  are used when available, with plain-text fallbacks otherwise.
---

# Finding Contribution Opportunities

Find conversations on GitHub and Slack where the user's input would add value, surface their
own in-flight work that's worth sharing, rank everything, and draft suggested wording they can
review and post themselves.

**The single hard rule: never post, reply, comment, react, or send anything. Ever.** This is
read-only. It surfaces opportunities and drafts suggestions; the user does the posting. If you
ever feel tempted to reach for a tool that writes to Slack or GitHub, stop.

## Step 1: Establish who this is for

The skill needs a few facts about the user. Take them from explicit input, custom
instructions, or a profile if available. Ask only for what you genuinely can't determine.

- **GitHub handle**: required for GitHub search. If unknown, infer from `gh api user --jq .login` and confirm.
- **Slack user ID**: needed only for Slack mention search. Skip Slack if unavailable.
- **Areas of expertise**: the topics where the user can add authoritative value (for example
  a discipline, a product area, or a set of repos). Infer from their recent work, profile, or
  custom instructions; ask if it's unclear. Everything downstream is filtered against these.
- **Relevant orgs and repos**: where their contributions land. Default to the orgs they're
  active in (derive from recent activity); let them narrow it.
- **Notes source (optional)**: a local notes vault (e.g. Obsidian) whose daily notes log
  shipped work. Use only if a path is provided. Treat it as strictly read-only.

If the request is a plain "find me some opportunities", don't interrogate the user. Fill in
sensible defaults and run.

## What counts as a good opportunity

A good opportunity is **relevant**, **timely**, **visible**, and **low-risk**.

**Top priority above everything else: places where the user has been explicitly @-mentioned
or tagged.** A GitHub mention, review request, or assignment, or a Slack @-mention, where they
**haven't replied yet**, is the single highest-value opportunity type. Someone is asking for
their input by name. These always sort to the top and are always surfaced, even if slightly
outside their usual topics, unless they've already responded. Hunt for these first.

- **Relevant**: squarely in the user's areas of expertise. Tangential topics score low.
- **Timely**: active recently (default: last 14 days). A dead thread from months ago is not
  an opportunity.
- **Visible**: somewhere a contribution will be seen: busy threads, multi-participant
  discussions, questions from other teams, places leadership is watching. A reply in a large
  thread raises visibility more than one in a quiet corner.
- **Low-risk**: the user can add genuine value without wading into something contentious,
  political, or outside their expertise. Skip heated disagreements and decisions already made.

### Two kinds of opportunity

1. **Engage**: weigh in on *other people's* conversations (mentions and topic threads).
   Someone else started it; the user adds value by replying.
2. **Amplify**: share *the user's own* in-flight work. Good work goes unnoticed if they don't
   talk about it. This covers their open PRs, the PRs and issues assigned to them, and work
   logged in their notes. The opportunity is to post about it so the right people see it.

Both are surfaced together, ranked against the same bar: is this genuinely worth other
people's attention, or is it noise? A typo-fix PR is not worth a post; shipping a substantial
new pattern is.

**Quality over quantity.** Five strong opportunities beat twenty-five weak ones. If only three
are genuinely good, surface three. Don't pad.

## Don't surface these

- For **engage**, anything the user has **already participated in** (commented, replied,
  reacted). Check before surfacing. (The opposite holds for **amplify**: their own authored
  PRs and assigned work are exactly what you want there.)
- Closed, resolved, or merged items where the conversation is over. (Exception: a just-merged
  PR of theirs can be an amplify "shipped this" item if genuinely worth sharing.)
- Pure noise: bot messages, CI chatter, deploy notifications, standup threads.
- Trivial amplify candidates: typo fixes, version bumps, lockfile changes, routine chores.
- Topics outside their expertise where a comment would look like profile-padding.
- Anything sensitive, confidential, HR-related, or interpersonal, including private notes they
  wouldn't say out loud in public.

## Configurable inputs (use defaults unless told otherwise)

- **Time window**: default last 14 days for engage, last 7 days for amplify. A daily run
  narrows both to the last 24 hours; a weekly run to 7 days.
- **Max opportunities**: default up to 10, fewer if quality drops off.
- **Sources**: default all available: GitHub conversations, the user's own GitHub PRs and
  assigned work, Slack, and the notes source if provided.
- **Mode**: default both engage and amplify. The user may ask for one ("just what I should
  comment on" = engage; "what have I done worth sharing?" = amplify).
- **Focus**: default all their areas; they may narrow it to one.

## Step 2: Pre-flight

1. Confirm `gh auth status` succeeds. If not, tell the user GitHub results will be skipped and
   continue Slack-only, or stop if they wanted GitHub.
2. Confirm Slack search tools are available. If not, continue GitHub-only and note it.
3. If a notes source path was provided, confirm it exists (read-only). If missing, skip it and
   note it. Never write to it.
4. If a SQL/scratch-table tool is available, create a working table to collect and rank
   candidates. If not, track the same fields in a list as you go.

```sql
CREATE TABLE IF NOT EXISTS opportunities (
  id TEXT PRIMARY KEY,
  source TEXT,            -- 'github-issue' | 'github-discussion' | 'github-pr' | 'github-pr-own' | 'github-assigned' | 'notes' | 'slack'
  opp_kind TEXT,         -- 'engage' | 'amplify'
  topic TEXT,            -- one of the user's areas of expertise
  title TEXT,
  url TEXT,
  last_activity TEXT,     -- ISO date
  context TEXT,           -- short summary of the conversation or the work
  why TEXT,               -- why it's a good opportunity
  score INTEGER,          -- 1-100 (see ranking)
  priority TEXT,          -- 'High' | 'Medium' | 'Low'
  suggestion TEXT,        -- drafted wording in the user's voice
  is_mention INTEGER DEFAULT 0,  -- 1 if explicitly tagged/review-requested/assigned
  already_involved INTEGER DEFAULT 0
);
```

## Step 3: Hunt on GitHub

Search the user's relevant orgs and repos. Use the `gh` CLI and `gh search`, running several
focused searches rather than one broad one. Substitute `<handle>`, the date window, and the
user's topic keywords.

**Search for explicit mentions first**: the highest-value opportunities. Find where they've
been tagged, had a review requested, or been assigned, and haven't replied yet:

```bash
gh search issues 'mentions:<handle>' --state=open \
  --updated=">=$(date -v-14d +%Y-%m-%d)" -L 40 \
  --json title,url,updatedAt,repository,number,commentsCount
gh search prs 'mentions:<handle>' --state=open \
  --updated=">=$(date -v-14d +%Y-%m-%d)" -L 40 \
  --json title,url,updatedAt,repository,number
gh search prs --review-requested=<handle> --state=open -L 40 \
  --json title,url,updatedAt,repository,number
gh search issues --assignee=<handle> --state=open -L 40 \
  --json title,url,updatedAt,repository,number
```

For each mention, check whether the user has already replied **after** being tagged. If so,
drop it. If not, it's a High-priority opportunity.

Then run topic searches scoped to their orgs and areas of expertise. Build the query from
their own topic keywords, for example:

```bash
gh search issues --owner=<org> --state=open \
  '<keyword> OR <keyword> OR "<phrase>"' \
  --updated=">=$(date -v-14d +%Y-%m-%d)" -L 40 \
  --json title,url,updatedAt,repository,number,commentsCount,labels
gh search prs --owner=<org> --state=open \
  '<keyword> OR <keyword>' \
  --updated=">=$(date -v-14d +%Y-%m-%d)" -L 30 \
  --json title,url,updatedAt,repository,number
```

Note: `date -v-14d` is macOS (BSD) syntax. On Linux use `date -d '14 days ago' +%Y-%m-%d`.
For discussions, use `gh api` GraphQL search with `type:discussions`.

**Before keeping any GitHub candidate, confirm the user hasn't already participated:**

- Skip if `author` is the user.
- Pull the comments (`gh api repos/{owner}/{repo}/issues/{n}/comments --jq '.[].user.login'`)
  and skip if their handle is already there.
- Prefer items with a real question or a gap their expertise fills, and enough activity to be
  visible.

## Step 4: Hunt on Slack

Use a public Slack search tool (no consent needed for public channels). Run several targeted
searches over the window.

**Search for explicit mentions first.** Find threads where the user was @-mentioned and hasn't
replied (substitute their Slack user ID):

```
to:<@USERID> after:<YYYY-MM-DD>
"<User Name>" after:<YYYY-MM-DD>
```

Also search for their user-ID token directly. For every hit, open the thread and check whether
they've already replied. If so, drop it; if not, it's High priority.

Then run topic searches combining their expertise keywords with `after:` date filters. Favour
**questions** ("how do I...", "does anyone know...", "is there a pattern for...") and
**multi-participant threads**: those are where an authoritative reply adds most value. Read
the full thread before drafting. Skip:

- Anything they've already replied to (check participants for their user ID).
- DMs and private channels, because this is about public visibility. Do not search private channels
  unless the user explicitly asks and consents.
- Bot messages, standups, and noise.

## Step 5: Amplify the user's own work worth sharing

Surface work they're doing or just shipped that the right people would benefit from seeing.
Use the amplify time window (daily run = 24 hours, weekly = 7 days, default 7 days).

### Their own open PRs

```bash
gh search prs --author=<handle> --state=open \
  --updated=">=$(date -v-7d +%Y-%m-%d)" -L 40 \
  --json title,url,updatedAt,repository,number,isDraft,reviewRequests,state
```

Worth surfacing: substantive PRs (a real feature, pattern, or fix others care about),
especially in their key repos. Flag PRs **stuck waiting on review**: the suggested post can be
a polite nudge. Skip drafts unless they want eyes on them, and skip trivial PRs.

### PRs and issues assigned to them or awaiting their review

```bash
gh search prs --review-requested=<handle> --state=open -L 40 \
  --json title,url,updatedAt,repository,number
gh search prs --assignee=<handle> --state=open -L 40 \
  --json title,url,updatedAt,repository,number
gh search issues --assignee=<handle> --state=open -L 40 \
  --json title,url,updatedAt,repository,number,commentsCount
```

If someone's waiting on them, treat it as a mention/engage item (High priority). If it's their
own ongoing work where sharing progress adds value, treat it as amplify. Pick the more useful
framing; don't surface the same item both ways. De-dupe against Step 3.

### Their notes (optional)

If a notes source was provided, mine recent daily notes within the window for shareable work.
Daily notes often log shipped work, PRs, reviews, and decisions. Read the notes (never modify
them) and look for:

- Something shipped, merged, or built that others would find useful.
- A decision, finding, or lesson worth writing up as a short post.
- Work squarely in their areas of expertise.

Ignore meetings, admin, anything sensitive or interpersonal, and anything still half-formed.
When in doubt, leave it out. Notes are private thinking, so only surface what they'd happily
say in public. Capture each shareable item as a `notes`/`amplify` candidate, linking the
underlying PR/issue if the note references one.

## Step 6: Score and rank

Score each surviving candidate out of 100.

**Explicit mentions override the score.** Anything where the user was directly tagged,
review-requested, or assigned and hasn't replied is **automatically High priority** and sorts
above everything else, regardless of numeric score. Label these clearly (e.g. "Mentioned" /
"Review requested") so it's obvious someone is waiting.

For engage items, weight:

- **Relevance to their expertise (0–40):** dead-centre scores high; tangential scores low.
- **Visibility (0–30):** participant count, thread length, cross-team or leadership presence.
- **Value they can add (0–20):** is there a clear gap their knowledge fills?
- **Freshness (0–10):** more recent = higher.

For amplify items, read the dimensions as relevance to their areas (0–40), shareworthiness and
audience (0–30), value of sharing or unblocking (0–20), and freshness (0–10). Be ruthless
about noise: trivial work falls below the bar and is dropped, not padded in.

Map to priority: **High** ≥ 70, **Medium** 45–69, **Low** < 45 (mentions are always High).
Drop anything below ~30 that isn't a mention. Sort mentions first, then by score; keep the top
N (default 10).

## Step 7: Draft suggested wording (in the user's voice)

For each kept opportunity, draft what they could post. **This is the most important output.**
It has to sound like them, not like AI.

Match the user's voice. If you know their writing conventions (from custom instructions, a
style guide, or their prior posts), follow them. Otherwise apply these general principles:

- Write like a human speaks. Natural, spoken rhythm. Read it aloud; if it's stiff, redo it.
- Direct and conversational. Short declarative sentences. Plain language. Contractions are fine.
- First person where natural.
- Speak with authority in their areas of expertise. Take a position and back it; no hedging.
- Don't pad. Say what needs saying, then stop.
- Be concrete. Name the thing, link to the example, point to the specific reference.
- No emoji (one only if it genuinely adds clarity), no ASCII art, no arrow glyphs.

Tailor length to venue:

- **Slack reply**: 1 to 4 sentences, helpful and human. Link to a relevant doc if there is one.
- **GitHub issue/discussion comment**: a short paragraph or two; can be structured as a
  recommendation then the reasoning. Reference specifics where it strengthens the point.
- **Amplify post**: a short, natural post for a relevant channel: what they did, why it
  matters, and the link. 1 to 3 sentences. No humblebragging, no launch-speak. For a stuck PR,
  a one-line nudge naming the PR and the kind of review it needs. Suggest a sensible
  destination channel where you can, and flag it in `[brackets]` if you're unsure it exists.

If a suggestion needs a fact they'd have to verify (a token name, a doc URL), draft it and flag
the bit to check in `[brackets]` rather than inventing it.

## Step 8: Surface (never post)

Lead with any **explicit mentions** (someone is waiting by name). Then split the rest into two
clearly headed groups: **Worth sharing (amplify)** and **Worth weighing in on (engage)**.
Within each, order by priority (High first). For each item:

> **[Title]**: _[source · topic · last active]_
> [one line on why it's worth their input, or why it's worth sharing]
> **Suggested wording:**
> > [the drafted text, in their voice, ready to copy-paste]
> [link, and for amplify items the suggested destination channel if known]

Keep the preamble to a sentence or two. End with a one-line reminder that nothing has been
posted and these are theirs to edit and send.

If an inbox or widget UI is available, additionally render the opportunities there: one item
per opportunity, with the title, a one-line reason plus the drafted suggestion, a deep link,
the last-activity timestamp, and labels for source, kind (Engage/Amplify), priority, topic, and
a stand-out "Mentioned" label for explicit mentions. The chat digest is the primary output;
the widget is an enhancement.

## Reminders

- **Read-only. Never post.** Worth repeating because it's the whole point.
- **Never modify the notes source.** Read it, don't touch it.
- Quality over quantity. Surface fewer, stronger opportunities.
- Both modes matter: weigh in on others' conversations (engage) and surface the user's own
  work worth sharing (amplify). Don't let the amplify list become noise.
- If a search returns nothing good in the window, say so plainly and offer to widen the window
  or broaden the topics rather than padding with weak suggestions.
- Don't surface engage items the user is already in. Do surface their own work for amplify.
- Suggestions must sound like the user, not like a bot.
