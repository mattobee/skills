---
name: running-a-session-retro
description: >
  Run a structured retrospective at the end of a work session: reconstruct the
  moments where output was wrong, misunderstood, or corrected; work out which were
  systemic rather than one-off; and upstream durable mitigations into the layer
  that will actually stop them recurring (a test, a lint or CI rule, a validator, a
  doc or runbook, or agent and skill instructions). Analyse autonomously, then
  present the findings and proposed fixes for approval before writing anything, and
  implement approved changes on a separate branch and draft PR, kept apart from the
  session's own work. Triggers on "do a retro", "session retro", "post-mortem",
  "what did we learn", "should we upstream anything", "how do we stop this
  happening again", or any request to turn this session's mistakes into lasting
  improvements.
compatibility: >
  Uses git for branching and diffs. The pull request step needs the gh CLI
  authenticated against the repo's remote. Read-only until you approve changes.
---

# Running a session retro

At the end of a work session, look back at what went wrong or had to be corrected,
work out which of those were systemic (a tool or process should have caught them),
and upstream a durable fix into the layer that will actually stop the class of
error recurring. Keep the process improvements separate from the work the session
was actually about.

The value is not writing up mistakes. It is converting this session's corrections
into guards that make the next person, or the next agent run, less likely to
repeat them.

## Non-negotiables

These are fixed. Do not trade them away for a tidier result.

- **Real failures only.** Ground every finding in something that actually happened
  this session. Do not invent or pad failure modes to look thorough. A retro that
  concludes "nothing worth upstreaming" is a valid, common outcome.
- **Never falsify to justify a fix.** The mitigation must reflect the real failure.
  Do not overstate the problem, fake a reproduction, or tune a check against
  fabricated data to make it look clean.
- **Fix the layer, not the instance.** Patching this one output is not a retro. The
  point is to stop the class of mistake, so the fix belongs in a durable, reusable
  place.
- **Separate branch and PR.** Process and tooling changes go on their own branch
  and draft PR, kept apart from the session's deliverable, so they can land on
  their own schedule and review cleanly.
- **Checkpoint before writing.** Do the whole analysis autonomously, but present
  findings and proposed mitigations and get a go-ahead before you change any files.

## When to run it, when to skip

Run it when a session did real work and hit friction: output that was wrong and
had to be redone, a misunderstanding you corrected, a review comment or CI failure,
a convention nobody had written down, a near-miss caught late.

Skip it (or stop after step 2) when the session was trivial, or when the only
mistakes were genuine one-offs with no realistic guard. Not every slip has a
systemic fix, and bureaucratising every mistake into a new rule makes the tooling
worse, not better.

## The loop

### 1. Reconstruct what happened

Pull the correction moments out of the session. Look for: places where output was
wrong and had to be redone, where you misread the task, where the user pushed back
or re-steered you, where a check or review or CI failed, where something surprised
you, and near-misses caught late.

Draw on more than memory. Read the actual evidence:

- `git log` and `git diff` on the session's branch, to see what really changed and
  what got reverted or rewritten.
- Review comments and CI results on any PR from the session.
- The conversation itself, for the corrections the user made.

Write each one down concretely: what happened, and what the correct outcome was.

### 2. Classify each: one-off or systemic

For every correction, ask: could a reasonable tool or process have caught or
prevented this?

- **One-off slip**, no realistic guard: note it and move on. No upstream.
- **Systemic gap**: a missing check, an ambiguous or missing instruction, an
  undocumented convention, a validator that should have flagged the shape of the
  mistake. This is an upstream candidate.

Be honest about which is which. If you are stretching to call something systemic,
it is probably a one-off.

### 3. Locate the right layer

Map each systemic gap to the highest-leverage durable home. Roughly from most to
least deterministic:

- **Automated check**: a test, a type, a lint rule, a validator, a CI step. Best
  when the failure is mechanically detectable.
- **Advisory guardrail**: a non-blocking warning or preview nudge, when a hard gate
  would over-trigger on legitimate cases. The nudge prompts a conscious second
  look without blocking honest work.
- **Process doc or runbook**: when the fix is a step a human or agent must take,
  not something code can enforce.
- **Instructions**: agent or skill instructions, CONTRIBUTING, code comments, when
  the fix is about how the work should be approached.

Push each fix as far toward automated as it will *honestly* go, but do not force a
brittle check where guidance is the right tool. One mistake can legitimately touch
several layers at once (for example a validator nudge plus a workflow line plus a
skill rule). Pick the set that actually closes the gap, not the largest set.

Prefer the lightest durable fix that works. A non-blocking nudge or a single
documented line often beats a hard gate that will generate false positives and get
disabled.

### 4. Propose, then wait

Present a short retro before touching any files:

- The systemic findings, each with the evidence behind it.
- The proposed mitigation, its target layer, and how heavy it is (nudge versus
  gate).
- A recommendation on which to action now and which to park.

Then stop and get approval. Some findings are legitimately not worth the
maintenance cost. Declining to fix one is a fine result to recommend.

### 5. Implement on a separate branch

Once approved:

- Branch off the appropriate base, usually the default branch, not the session's
  own feature branch, so the process changes do not ride on unmerged work.
- One logical change per commit. Add regression coverage next to each mitigation
  wherever the layer supports it (a test that fails without the fix).
- Keep each mitigation minimal and honest. It should fire on the real failure. If
  it can also trip legitimate cases, make it a nudge, not a blocker, and say so.
- Run the existing tests and lint for the area you touched. Do not add new tooling
  the repo does not already use just to satisfy the retro.
- Open the PR as a draft. Describe what the session surfaced and what each change
  guards against, in plain language. Do not restate the diff.

## Worked example

In one session, an analysis tool emitted a claim about "all N implementations"
that had only been checked against some of them, and a research takeaway that named
a specific product surface instead of a reusable principle. The existing validator
could not catch either.

The retro classified both as systemic: the tool could flag the *shape* of these
mistakes even if it could not prove them wrong. Because both patterns also occur in
legitimate, substantiated claims, the fix was a set of non-blocking advisories in
the tool's preview rather than hard gates, each with a regression test, plus a
matching line in the workflow doc and the skill instructions. All of it landed on a
separate branch and draft PR, apart from the session's actual data work.

Note the calibration: detectable-but-not-provable failure, legitimate cases exist,
so an advisory nudge, not a blocking gate. That is the judgement this skill is
teaching.

## Gotchas

- The session's feature branch is often unmerged. Branch process changes off the
  default branch so they can land independently of the work that prompted them.
- Advisory nudges are *supposed* to fire on some legitimate cases too. The value is
  the conscious second look. Do not gut their coverage chasing zero false
  positives.
- Do not bundle unrelated cleanup into the retro PR. Keep it to the mitigations the
  retro justified.
- If the thing being improved is itself a skill or workflow, the same rules apply:
  separate branch, regression coverage, no falsifying.
- Resist inventing findings to make the retro feel worthwhile. "Nothing to
  upstream" is an honest and frequent answer.
