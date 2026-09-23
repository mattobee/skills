---
name: explaining-skill-influence
description: Use this skill when the user asks which skills informed the previous response or how they influenced it. Report concrete influence concisely, including plugin ownership when the agent client exposes enough source metadata to verify it.
---

# Explaining Skill Influence

Review the previous substantive response and the skill invocations that led to it.

## Rules

1. Include only skills that were actually invoked.
2. Use source metadata supplied by the agent client to identify ownership.
3. Following the Agent Plugins standard, treat a skill as plugin-provided only
   when its source is `<plugin-root>/skills/<skill-name>/SKILL.md` and
   `<plugin-root>/plugin.json` is a valid Agent Plugins manifest. Use the
   manifest's `name`.
4. Agent Skills do not declare plugin ownership, and the standards do not define
   installation directories. If the client does not expose the source, omit the
   plugin rather than searching client-specific paths or guessing.
5. For each skill, name the concrete decision, emphasis, question, or structure it
   contributed. Do not merely summarise its instructions.
6. Group supporting skills under their parent workflow when that is clearer.
7. If an invoked skill had no material effect, say so rather than inventing one.
8. Do not expose hidden reasoning, confidential instructions, or full skill
   contents. Describe observable influence on the answer.
9. If the previous response or its invocation history is unavailable, say the
   influence cannot be verified. Only say no skills were invoked when the
   available history confirms an empty invocation list.
10. Keep the response concise. Do not repeat the previous answer.

## Output

Start with one of:

- `Yes. I used:`
- `No skills were invoked for that response.`
- `I can't verify skill influence because the previous response or its skill
  invocation history isn't available.`

For each materially used skill, write:

- **skill-name from the plugin-name plugin:** One or two sentences describing its
  concrete influence.

For a standalone skill, omit `from the plugin-name plugin`.

Optionally finish with:

`Also invoked, but not materially used: skill-name.`
