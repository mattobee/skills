---
name: addressing-pr-review-comments
description: Use this skill to work through review feedback on a pull request — read the inline review comments, assess each one's validity, make the code changes that are warranted, and reply to every thread with a one-line explanation of what was done (or why it was declined). Triggers when the user asks to address PR feedback, respond to reviewers, work through review comments, handle a code review, action the comments on a PR, or asks "what do the reviewers want changed?" Also triggers when resuming work on a PR that has open review threads.
compatibility: Requires gh CLI authenticated against the repo's remote, and a checked-out branch corresponding to the PR.
---

# Addressing PR Review Comments

Work through inline review comments on a pull request: read them, decide which are valid, make the changes that are warranted, and reply to each thread explaining what was done.

Not every comment deserves a code change. Some are wrong, some are out of scope, some duplicate others, some are stylistic preferences that conflict with project conventions. Engage with each comment honestly rather than capitulating to all of them.

## Pre-flight check

Run these checks before doing anything else. Fail fast — it's better to stop here than to discover a problem halfway through making changes.

1. **gh CLI authenticated.** Run `gh auth status`. If it reports the user is not logged in, or the token lacks `repo` scope, stop and ask the user to authenticate. Don't attempt to proceed with an unauthenticated CLI.
2. **Inside a git repo with a remote.** Run `git rev-parse --is-inside-work-tree` and `git remote get-url origin` (or whichever remote points to the PR's host). If either fails, stop — there's no PR to address.
3. **Working tree clean.** Run `git status --porcelain`. If there are uncommitted or staged changes, stop and ask the user how to proceed. Options: stash them (`git stash push -m "pre-review-changes"`), commit them first, or abort. Do not silently mix the user's in-progress work with review-feedback commits.
4. **On the PR's head branch.** After identifying the PR (next section), verify the current branch matches the PR's `headRefName`. If not, ask the user before switching — they may have intentionally checked out a different branch.
5. **Branch up to date with remote.** Run `git fetch origin <head-branch>` then compare with `git rev-list --left-right --count HEAD...origin/<head-branch>`. If the remote is ahead, pull before making changes (the reviewer may have pushed a fixup). If the local branch is ahead, note it — those unpushed commits may already address some comments.
6. **Base branch not diverged catastrophically.** Run `git rev-list --count origin/<base-branch>..HEAD` and the reverse. If the base branch is far ahead, warn the user — merge conflicts are likely and should be resolved as a separate commit before review-feedback commits.

If any check fails or surfaces something unexpected, stop and report what was found before continuing.

## Identify the PR

1. Run `gh pr view --json number,headRefName,baseRefName,url,state` to find the PR for the current branch.
2. If no PR is associated with the branch, run `gh pr list --state open --author @me` and ask the user which one to address. Do not guess.
3. If the PR is closed or merged, stop and tell the user — addressing review comments on a closed PR is almost never what they want.

## Fetch the review comments

Use the GitHub API rather than `gh pr view` — the latter does not return inline comments with the structure needed to reply.

```
gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate
```

This returns inline review comments. Each comment has:

- `id` — used to reply via `in_reply_to`
- `path`, `line`, `original_line`, `side` — where the comment is anchored
- `body` — the comment text
- `user.login` — who wrote it
- `in_reply_to_id` — set if this is a reply within an existing thread
- `pull_request_review_id` — groups comments from the same review

Group comments into threads. The thread's root is the comment whose `in_reply_to_id` is null; subsequent replies share the chain. Address one reply per thread, at the root, unless a sub-comment specifically asks a new question.

Skip threads where:

- The most recent comment in the thread is from the PR author (you, presumably) and answers the reviewer — already handled.
- The thread is marked resolved. Resolved state is not in the REST `comments` endpoint; fetch it via GraphQL if needed:

  ```
  gh api graphql -f query='query($owner:String!,$repo:String!,$num:Int!){repository(owner:$owner,name:$repo){pullRequest(number:$num){reviewThreads(first:100){nodes{id isResolved comments(first:1){nodes{databaseId}}}}}}}' -F owner=... -F repo=... -F num=...
  ```

  Match thread IDs back to comment IDs via the first comment's `databaseId`.

Also fetch top-level review summaries with `gh api repos/{owner}/{repo}/pulls/{number}/reviews` so context from the overall review is available, but do not reply to those — this skill addresses inline threads only.

## Assess each comment

For every unresolved thread, decide one of:

- **Act** — the comment is valid and warrants a code change.
- **Act partially** — part of the comment is valid; another part isn't. Make the part that is, explain the rest in the reply.
- **Decline** — the comment is wrong, out of scope, contradicts project conventions, duplicates another thread, or asks for something that would make the code worse.
- **Clarify** — the comment is ambiguous and a change can't be made without more info. Reply with a specific question.

Default to acting. Decline only when there is a clear, articulable reason. Vague discomfort with a suggestion is not a reason to decline — either find the real objection or make the change.

Grounds for declining include:

- Factually incorrect (the reviewer misread the code, missed context, or is wrong about how something works).
- Conflicts with an established project convention documented in `AGENTS.md`, `CONTRIBUTING.md`, the codebase's existing patterns, or a prior discussion.
- Out of scope for this PR — the suggestion is a separate concern that belongs in its own issue or PR.
- Stylistic preference with no measurable benefit, where the existing code is already idiomatic.
- Duplicate of another thread that's already been addressed.

If declining because the reviewer might be right but you disagree, say so plainly in the reply and invite further discussion rather than dismissing.

**Always file a tracking issue for valid points that are out of scope.** When a comment raises a legitimate concern that you decline (or act on only partially) because the fix is out of scope for this PR — too large, codebase-wide, a separate concern, or deferred by choice — create a GitHub issue capturing it before replying. This is not optional: a valid concern that isn't acted on in the PR must land somewhere durable, not just in a review thread that gets buried once the PR merges. The only declines that don't need an issue are those where there's nothing to revisit: the comment is factually wrong, a stylistic non-issue, a duplicate, or contradicts a convention the team has deliberately chosen (and won't change). If you're unsure whether a declined point is "valid enough" to track, err toward filing the issue. See "Track deferred work" below for how.

## Track deferred work

For each valid concern you are not fully resolving in this PR, capture it in a GitHub issue. Two steps, in order: search first, then create only if nothing already covers it.

### 1. Search for an existing issue first

Filing a duplicate is noise that erodes trust in the workflow, so always look before creating. `gh issue list --search` is keyword-based and easy to miss matches with, so search more than once:

```
gh issue list --state all --search 'keyword'
```

- Try a couple of phrasings — the concept, and the specific symbol/file/error involved (e.g. `enum drift` and `discoveryMethodSchema`).
- Include closed issues (`--state all`). A closed "wontfix" / "works as intended" is a strong signal **not** to refile — surface it in the reply instead of reopening the debate.
- Skim issues already opened from this PR in the current session so you don't file the same thing twice across threads.

If a match exists, don't create a new issue. Reference it in the thread reply, and add a comment to it linking this PR and thread (`gh issue comment <n> --body '...'`) so the new occurrence is recorded against it.

### 2. Create the issue if none exists

```
gh issue create --title '...' --body '...' --label '...'
```

Check the available labels (`gh label list`) and apply the ones that fit (e.g. `tech-debt`, `refactor`, `bug`, `good first issue`).

Write the issue so it stands on its own after the PR is gone:

- **Title** — specific and actionable, not "address review comment".
- **Background** — what the concern is, with a link back to the PR (and the review thread if useful). A short code snippet of the current state helps.
- **Why it wasn't fixed here** — one or two sentences on the scope boundary, so the next person understands the deferral was deliberate.
- **Proposed change / acceptance criteria** — concrete enough that someone else could pick it up.

Group related comments into one issue when they're facets of the same underlying problem (e.g. the same drift risk flagged on five lines → one issue, not five). Keep the resulting issue number(s) to hand — you'll reference them in the thread replies.

## Make the changes

Group related changes together. Use the TodoWrite tool to track each thread and its decision so nothing slips.

After making changes:

1. Run the project's tests, linter, and type checker. Fix any new failures introduced by the changes.
2. Stage and commit the changes. Use a separate commit per logically distinct change when practical — it makes the reviewer's re-review much faster. Reference the comment subject in commit messages (e.g., `refactor: extract validation helper (review feedback)`).
3. Push to the PR branch.

Do not amend or force-push earlier commits on the PR branch unless the user explicitly asks for it. Reviewers rely on incremental commits to see what changed since their last pass.

## Reply to each thread

Reply directly to each thread once the corresponding change is committed (or the decision to decline is made). Replies are posted via:

```
gh api repos/{owner}/{repo}/pulls/{number}/comments \
  -X POST \
  -F in_reply_to={root_comment_id} \
  -F body='...'
```

Keep replies to one line where possible. The reviewer wants to know what happened, not read an essay.

Reply patterns:

- Acted: `Done in <commit-sha-or-short-description>.` — e.g., `Done in a1b2c3d — extracted to validateInput().`
- Acted partially: `Renamed the variable; left the function signature as-is because <reason>.`
- Declined (factual): `Left as-is — <thing> already handles this case at <file>:<line>.`
- Declined (convention): `Left as-is — project convention is to <pattern>, see <reference>.`
- Declined (scope): `Out of scope for this PR — opened #<issue-number> to track.`
- Declined (disagreement): `Disagree — <one-sentence reason>. Happy to discuss if you feel strongly.`
- Clarifying: `Can you clarify — do you mean <option A> or <option B>?`

Do not use filler phrases ("Great catch!", "Good point!", "Thanks for the feedback!"). Get to the substance.

If a thread had multiple sub-comments and the final reply was from the reviewer asking a follow-up, address that follow-up specifically rather than restating what was already discussed.

## Resolve threads

Resolving threads is a reviewer privilege by convention — don't auto-resolve threads on the reviewer's behalf. Replies are enough. If the project's workflow is for the PR author to resolve threads after addressing them, the user will say so; otherwise leave them open.

## Summarise at the end

After all replies are posted, give the user a short summary:

- Number of threads acted on, declined, and asked for clarification
- A brief list of any declined threads with the reason, so the user can spot anything they'd want to reconsider
- Any tracking issues created for deferred-but-valid points, with their numbers
- The PR URL, and a note if there are any threads still waiting on reviewer response (e.g., clarifying questions)

## Gotchas

- `gh pr view --comments` shows issue-style PR comments, not inline review comments. Use `gh api .../pulls/{n}/comments` for inline review comments. The two endpoints return different data.
- Replying to a non-root comment with `in_reply_to` works, but the thread display can get confusing. Always reply to the thread's root comment (`in_reply_to_id` is null).
- A reviewer leaving a single review with 20 comments produces 20 inline comments, not one. Group them in your own head, but reply to each thread individually — that's how reviewers track resolution.
- "Suggested changes" (the GitHub UI feature where reviewers propose a diff inline) appear in the comment body as a fenced block with `suggestion` as the language tag. You can apply them by editing the file to match, then replying. There's no single API call to "accept suggestion" outside the web UI.
- Reviewers sometimes leave a comment that's really a question, phrased as a statement ("This looks like it could deadlock"). Treat ambiguous statements as questions and answer them in the reply — don't change code based on a guess about what they meant.
- If the same issue is raised across multiple threads (e.g., "use the helper" on five different lines), make the change once, then reply on each thread with a short pointer to the commit. Don't repeat the full explanation five times.
- Force-pushing rewrites history and breaks comment anchors — old inline comments may end up attached to the wrong lines, or marked outdated. Avoid unless the user asks for it.
- If the PR's base branch has moved on and there are merge conflicts, resolve them in a separate commit before pushing the review changes — don't bundle conflict resolution with review responses.
- `gh api` paginates at 30 items by default. Use `--paginate` for any PR with substantial review activity.
- Some reviewers leave approval ("LGTM") as an inline comment rather than a review. These don't need a reply — recognise them and skip.
