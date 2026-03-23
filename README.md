# Skills

Agent skills for coding agents. Built on the [Agent Skills](https://agentskills.io) open specification, it's platform-agnostic by design and works with any agent that supports the spec.

## Available Skills

### designing-agent-teams

Designs, generates, and refines multi-agent coding teams with optimal model-to-role assignments. Produces team reasoning documents and agent instruction files that work across platforms.

## Installation

```bash
npx skills add mattobee/skills
```

Or install a specific skill:

```bash
npx skills add mattobee/skills --skill designing-agent-teams
```

## Setting Model Preferences

The skill uses three generic tiers — **frontier**, **mid-tier**, and **fast** — instead of provider-specific model names. When it generates agent configs, you substitute the model IDs from your provider.

| Tier     | Role                      | Anthropic     | OpenAI              | Google                |
| -------- | ------------------------- | ------------- | ------------------- | --------------------- |
| Frontier | Orchestration, planning   | Claude Opus   | GPT-4o / o3         | Gemini 2.5 Pro        |
| Mid-tier | Implementation, testing   | Claude Sonnet | GPT-4.1             | Gemini 2.5 Flash      |
| Fast     | Exploration, simple edits | Claude Haiku  | GPT-4.1 mini / nano | Gemini 2.0 Flash-Lite |

Models and naming change frequently. Check your provider's current model lineup when configuring agents. See `skills/designing-agent-teams/references/model-guide.md` for full assignment heuristics.

## Sources

The skill's design principles draw on published research, the [Agent Skills](https://agentskills.io) specification, and practitioner experience. See `skills/designing-agent-teams/references/sources.md` for claim-by-claim sourcing and evidence categories.

## License

MIT
