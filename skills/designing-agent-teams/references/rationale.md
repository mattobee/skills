# Rationale

Design decisions and where they come from. The agent never loads this file — it's for humans reviewing the skill.

## Separate tester from coder

[AgentCoder](https://arxiv.org/abs/2312.13010) (2023) separated code generation and test generation into distinct agents, achieving 96.3% on HumanEval vs 90.2% prior best. The key insight is that a coder testing its own work has confirmation bias — it tests what it thinks it implemented, not what it should have. This directly shaped the skill's core team structure: every team includes an independent Tester agent that writes tests from the spec, never from the Coder's implementation. The skill flags a missing independent tester as the first thing to fix in any team review.

## Plan-implement-verify core team

Multiple prominent multi-agent systems converge on a three-role pattern — planning, implementation, and verification — with distinct agents for each phase. [MetaGPT](https://arxiv.org/abs/2308.00352) uses structured SOPs mapping to these stages. AgentCoder separates programmer, test designer, and test executor. [ChatDev](https://arxiv.org/abs/2307.07924) runs a separated testing phase that catches bugs inline testing misses. Anthropic's [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) describes the same pattern as an evaluator-optimizer loop. The skill codifies this as the mandatory core team: Orchestrator (plan), Coder (implement), Tester (verify) — present in every team the skill generates, with specialists added on top per project.

## Gate each step with deterministic checks, not LLM self-evaluation

Accuracy compounds across steps — even 85% per step drops to 20% over a 10-step chain. [LLMs Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798) (Huang et al., 2023) showed that without external feedback, LLMs struggle to identify their own errors. The Agent Skills spec arrives at the same conclusion with its [validation loops](https://agentskills.io/skill-creation/best-practices) pattern. This is why the skill requires deterministic quality gates (linting, type-checking, test pass rates) at each step rather than relying on LLM self-evaluation, and flags LLM-only quality gates as a review finding.

## Keep agent instructions short

The Agent Skills [best practices](https://agentskills.io/skill-creation/best-practices) recommend SKILL.md under 500 lines / 5,000 tokens: "Overly comprehensive skills can hurt more than they help — the agent struggles to extract what's relevant." The skill applies this same principle to individual agent instruction files, capping each at 500 words. Longer prompts waste context on every call and dilute the instructions that actually matter — the agent's capabilities come from the model and its tools, not from lengthy persona descriptions.

## Ground specialists in official docs, not training data

LLMs confidently generate outdated API calls from stale training data. The Agent Skills [best practices](https://agentskills.io/skill-creation/best-practices) call this out directly: "A common pitfall is asking an LLM to generate a skill without providing domain-specific context — relying solely on the LLM's general training knowledge." This is why the skill instructs every specialist agent to wire in MCP servers or documentation URLs and explicitly tells them to check official sources before generating platform-specific code. A Supabase agent relying on training data alone might use deprecated RLS syntax or old client methods — the skill's gotchas section warns about this specifically.

## Platform-agnostic core outputs

The [Agent Skills specification](https://agentskills.io/specification) is platform-agnostic by design — skills provide knowledge and workflows, not platform config. Following this principle, the core outputs (`.agents/*.md` instruction files and `AGENTS.md`) are plain markdown that works with any agent platform. Platform-specific config files (OpenCode, Claude Code, Cursor, etc.) are generated as thin wrappers only when the platform is detected or specified, and the skill consults the platform's own documentation for current config syntax rather than bundling format references that go stale.

## Generic model tiers instead of provider-specific names

Frontier / mid-tier / fast as capability tiers, with a provider mapping table in `model-guide.md`. This follows the Agent Skills spec's portability principle — skills should work across agents and providers. The skill's design principles, cost projections, and assignment heuristics all use tier names, so teams designed with this skill remain valid when switching between Anthropic, OpenAI, Google, or other providers. Only the mapping table needs updating as new models release.

## Multi-agent isn't always better

[More Agents Is All You Need](https://arxiv.org/abs/2402.05120) (2024) showed that performance gains from adding agents correlate with task difficulty — when single-agent accuracy is already high, more agents add coordination overhead without meaningful improvement. This is why the skill explicitly asks "would a single agent suffice?" in its review checklist and includes guidance that a well-prompted single agent with a clear spec handles many projects better than a poorly-coordinated team. The skill sets a rough threshold: multi-agent coordination adds the most value when single-agent accuracy is below ~45% or when the project has clearly parallelisable work streams.

## Practitioner heuristics (no formal source)

These are experience-derived rules of thumb. They're useful but not backed by published research:

### 70/20/10 call volume split

~70% fast, ~20% mid-tier, ~10% frontier by call volume. The skill uses this as a budgeting guideline and a smell test — if more than 30% of calls go to frontier models, re-examine whether those tasks genuinely need top-tier reasoning. The specific numbers are chosen to be memorable and to flag cost creep early, not because they represent a measured optimum.

### Mid-tier handles 80–90% of coding tasks

Most implementation work — feature building, test writing, standard refactoring — doesn't need frontier reasoning. This observation drives the skill's model assignment heuristic: default to mid-tier for the Coder, Tester, and most specialists, reserving frontier for the Orchestrator's planning and review work.

### Frontier models tend to refactor beyond scope

Widely observed in practice, not formally studied. Frontier models given a focused task will often "improve" adjacent code unprompted. The skill addresses this by recommending explicit scope constraints in system prompts for frontier-powered agents: "make only the changes described in the plan, do not refactor adjacent code."

### Dual-touchpoint specialists

Calling domain specialists early (planning) and late (review) catches issues cheaper than a single review gate at the end. Early assessment shapes the implementation rather than just judging it after the fact. The skill recommends this pattern for concerns that are expensive to retrofit — accessibility, security — while noting that not every specialist needs two touchpoints (a documentation writer only needs a late pass).
