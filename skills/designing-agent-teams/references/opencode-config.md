# OpenCode Agent Configuration Reference

OpenCode agents are configured either as markdown files or in `opencode.json`. Each agent is either a **primary** agent (user interacts directly, switch with Tab) or a **subagent** (invoked by primary agents or via `@mention`).

## Markdown agents (preferred for this skill)

Place one `.md` file per agent in `.opencode/agents/` (project-level) or `~/.config/opencode/agents/` (global). The filename becomes the agent name.

Example: `.opencode/agents/coder.md`

```markdown
---
description: Implements features and fixes within scoped boundaries
mode: subagent
model: anthropic/claude-sonnet-4-20250514
permission:
  edit: allow
  bash:
    "*": ask
    "npm test": allow
    "npx vitest *": allow
---

You are the Coder. Read .agents/coder.md for your full instructions.
```

### Frontmatter fields

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | What the agent does and when to use it |
| `mode` | No | `primary`, `subagent`, or `all` (default: `all`) |
| `model` | No | Provider/model ID (e.g., `anthropic/claude-sonnet-4-20250514`). Subagents inherit from the invoking primary agent if not set. |
| `permission` | No | Tool access: `edit`, `bash`, `webfetch` each set to `allow`, `ask`, or `deny` |
| `temperature` | No | 0.0–1.0. Lower = deterministic, higher = creative |
| `steps` | No | Max agentic iterations before forced text response |
| `hidden` | No | Hide from `@` autocomplete (subagents only) |
| `color` | No | Hex colour or theme name for UI display |

The markdown body after frontmatter is the agent's system prompt.

### Model ID format

Use `provider/model-id`:
- `anthropic/claude-opus-4-20250514`
- `anthropic/claude-sonnet-4-20250514`
- `anthropic/claude-haiku-4-20250514`

### Permissions (replaces deprecated `tools`)

```yaml
permission:
  edit: allow          # allow, ask, or deny
  bash:
    "*": ask           # default for all commands
    "npm test": allow  # specific command override
    "git push": ask
  webfetch: deny
```

Last matching rule wins. Put `*` wildcard first, specific rules after.

### Task permissions

Control which subagents a primary agent can invoke:

```yaml
permission:
  task:
    "*": deny
    "coder": allow
    "tester": allow
    "security-*": ask
```

## JSON configuration (alternative)

Agents can also be defined in `opencode.json`. Use `{file:./path}` to reference external prompt files:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "orchestrator": {
      "description": "Decomposes tasks, coordinates agents, reviews results",
      "mode": "primary",
      "model": "anthropic/claude-opus-4-20250514",
      "prompt": "{file:./.agents/orchestrator.md}",
      "permission": {
        "edit": "deny",
        "bash": {
          "*": "deny",
          "git status *": "allow"
        }
      }
    },
    "coder": {
      "description": "Implements features and fixes",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "prompt": "{file:./.agents/coder.md}",
      "permission": {
        "edit": "allow",
        "bash": {
          "*": "ask"
        }
      }
    }
  }
}
```

## Built-in agents

OpenCode ships with these agents (customise, do not replace):

- **build** (primary) — default agent with all tools enabled
- **plan** (primary) — read-only analysis and planning, edit and bash set to `ask`
- **general** (subagent) — full tool access for multi-step tasks
- **explore** (subagent) — fast, read-only codebase exploration

## How this skill generates OpenCode agents

For each agent in the team, generate a markdown file in `.opencode/agents/`:

- **Orchestrator** → `.opencode/agents/orchestrator.md` (mode: `primary`)
- **Coder** → `.opencode/agents/coder.md` (mode: `subagent`)
- **Tester** → `.opencode/agents/tester.md` (mode: `subagent`)
- **Specialists** → `.opencode/agents/[name].md` (mode: `subagent`)

Each agent's markdown body should reference its `.agents/*.md` instruction file for the full role description, then add any brief context that's useful as a system prompt preamble.

When adding an agent to an existing team, create a new `.opencode/agents/[name].md` file. Do not modify existing agent files unless asked.
