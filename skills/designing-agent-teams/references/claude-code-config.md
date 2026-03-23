# Claude Code Agent Configuration Reference

Claude Code uses markdown-based configuration through `CLAUDE.md` files and supports subagent orchestration natively.

## CLAUDE.md structure

`CLAUDE.md` lives in the project root and provides instructions to the main Claude Code agent. For multi-agent setups, it defines the orchestration pattern and references subagent configurations.

```markdown
# Agent Team Configuration

## Orchestration Model

This project uses a plan-execute-review pattern:
- **Planning**: Use Opus for architectural decisions and task decomposition
- **Execution**: Use Sonnet for implementation, testing, and standard refactoring
- **Exploration**: Use Haiku for file searches, context gathering, and boilerplate

## Agent Roles

### Planner (Opus)
Responsible for: analysing requirements, decomposing tasks, architectural decisions.
Before implementing any feature, create a plan that identifies affected files,
required changes, and potential risks. Do NOT write code in planning mode.

### Implementer (Sonnet)
Responsible for: writing code, creating tests, standard refactoring.
Follow the plan from the Planner. Keep changes scoped to the planned task.
If you encounter unexpected complexity, stop and escalate rather than improvising.

### Explorer (Haiku)
Responsible for: finding files, searching for patterns, gathering context.
Used automatically for codebase exploration. Returns summaries, not raw file contents.

### Reviewer (Opus)
Responsible for: pre-merge review of all changes.
Check for: correctness, edge cases, security issues, missing tests, style consistency.
```

## Model configuration in Claude Code

Claude Code supports model assignment through its settings:

```json
{
  "model": "claude-sonnet-4-6",
  "smallModel": "claude-haiku-4-5-20251001",
  "planModel": "claude-opus-4-6"
}
```

- `model` — The primary model for code generation and implementation (Sonnet recommended)
- `smallModel` — Used for exploration, file searches, and lightweight tasks (Haiku recommended)
- `planModel` — Used when the `opusplan` command is invoked for planning (Opus recommended)

## Built-in multi-agent features

Claude Code has several built-in agent patterns:

### opusplan mode
Invoke with `/opusplan` or configure as default. Opus handles planning with read-only tools, then Sonnet executes the plan. This is the most battle-tested multi-model pattern in Claude Code.

### Explore subagent
Automatically uses the `smallModel` (Haiku by default) for codebase exploration. Runs in its own context window and returns summaries to the parent agent.

### Smart model switching
Claude Code can automatically route ~30–40% of tasks to the cheaper model tier when it detects the task is bounded and well-defined. This happens transparently.

## Subagent definitions

Each subagent gets a brief entry in `CLAUDE.md` that references its full instruction file in `.agents/`:

```markdown
## Agent Team

### @coder
Model: claude-sonnet-4-6
Instructions: See .agents/coder.md

### @tester
Model: claude-sonnet-4-6
Instructions: See .agents/tester.md

### @reviewer
Model: claude-opus-4-6
Instructions: See .agents/reviewer.md
```

The full instructions — role description, tool access, escalation rules, domain-specific guidance, MCP references — live in the `.agents/*.md` files, not inline in `CLAUDE.md`. This keeps `CLAUDE.md` scannable and avoids bloating the main context with agent details that are only needed when that agent is invoked.

## CLAUDE.md best practices for agent teams

- Put the `## Agent Team` section in CLAUDE.md with brief entries referencing `.agents/*.md` files
- Full instructions, escalation rules, and domain guidance go in the `.agents/*.md` files, not inline
- Define escalation triggers explicitly in each agent's file: "If you encounter X, escalate to @reviewer"
- Scope each agent's filesystem access in its `.agents/*.md` file
- Use the `@agent-name` convention for easy reference

## Cost control in Claude Code

- The Max plan ($100/month or $200/month for 20x) includes usage caps. Monitor with `/usage`.
- API mode gives direct control over model routing and costs
- Prompt caching is automatic in Claude Code — shared context across agent calls is cached
- For heavy Opus usage, consider API mode over subscription to control costs precisely
