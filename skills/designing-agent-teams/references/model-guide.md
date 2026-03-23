# Model Guide — Assignment Heuristics

Use the latest Opus, Sonnet, and Haiku models available. Look up current model strings, pricing, and context limits from https://platform.claude.com/docs/en/about-claude/pricing when generating config files.

## Tier assignments

**Opus — The architect and safety net.** Most capable, slowest, most expensive. Use for orchestration, architectural planning, complex multi-file debugging, and pre-merge review. These are low-volume, high-leverage tasks where reasoning depth matters and mistakes are expensive. Avoid for single-file tasks, boilerplate, or anything high-volume.

**Sonnet — The workhorse.** Handles 80–90% of coding tasks at the best quality/cost/speed balance. Use for feature implementation, test writing, standard refactoring, and multi-file changes within a well-understood scope. Less prone to overengineering than Opus — tends to make the requested change rather than refactoring everything it touches. Default to Sonnet unless there is a specific reason to use another tier.

**Haiku — The speed specialist.** Fastest and cheapest, with a smaller context window. Use for bounded, well-defined tasks: file exploration, grep, boilerplate, docstrings, commit messages, simple edits. Ideal for high-volume operations and preliminary exploration before handing off to a more capable agent. If escalation rate from Haiku exceeds ~30%, the tasks assigned to it are too complex for this tier.

## The 70/20/10 rule

Target call distribution by volume: ~70% Haiku, ~20% Sonnet, ~10% Opus. The orchestrator runs roughly 10 times per workflow while workers run 100+ times — volume differences make per-call cost compound fast.

Red flags in a team's distribution:
- More than 30% Opus calls → tasks are likely over-assigned to the top tier
- Less than 50% Haiku calls → look for bounded subtasks to delegate down
- Escalation rate above 30% from any tier → tasks are miscategorised for that tier

## Escalation design

Build concrete escalation triggers, not vague "if it's hard" rules. Effective triggers: test failures after an attempt, error counts exceeding a threshold, the agent detecting it needs to reason across more files than its scope allows. Escalation should flow upward (Haiku → Sonnet → Opus) and be the exception, not the norm.

## Cost levers

- **Prompt caching** — shared context replayed across agent calls gets cached automatically; the savings are large
- **Effort control** — Opus and Sonnet support adaptive effort levels; at reduced effort, Opus delivers strong results with far fewer output tokens
- **Batch API** — significant discount for non-time-critical work (doc generation, test expansion, analysis)
- **Context isolation** — each subagent gets its own context window and returns only a summary, preventing context bloat in the orchestrator
