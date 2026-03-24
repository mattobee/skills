# Skills

Reusable skills for coding agents. I use these across my projects and you can too. Built on the [Agent Skills](https://agentskills.io) open specification, so they work with any agent that supports the spec.

## Available Skills

You can install all skills in one go with:

```bash
npx skills add mattobee/skills
```

### designing-agent-teams

Designs, generates, and refines multi-agent coding teams with optimal model-to-role assignments. Produces team reasoning documents and agent instruction files that work across platforms.

```bash
npx skills add mattobee/skills --skill designing-agent-teams
```

Example prompts:

- "Design an agent team for this codebase"
- "Review my agent team configuration and suggest improvements"
- "Add a security specialist to my existing agent team"
- "Which model tier should I use for my tester agent?"

### suggesting-next-steps

Suggests prioritised next steps for a project based on git history, GitHub issues/PRs, tracking files, and project docs.

```bash
npx skills add mattobee/skills --skill suggesting-next-steps
```

Example prompts:

- "What should I work on next?"
- "I haven't touched this project in a week — what's the state of things?"
- "What's left to do on this feature?"
- "Help me prioritise my backlog"

## License

MIT
