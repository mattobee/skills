# Sources and Evidence

This document covers the claims and heuristics used in the skill's design principles. It exists for human readers auditing the skill — the agent never loads this file.

Each claim is categorised as **research-backed**, **research-adjacent** (grounded in real math or papers but with practitioner-chosen specifics), or **practitioner heuristic** (experience-derived, no formal study).

## Multi-agent adds value when single-agent accuracy is below ~45%

**Category**: Research-adjacent

The underlying logic comes from Condorcet's Jury Theorem: majority voting among N independent agents with individual accuracy p improves on a single agent if and only if p > 0.5. Below 0.5, adding agents makes the group worse.

The ~45% threshold in the skill is a practitioner adjustment — agent errors are correlated (same training data, similar failure modes), which shifts the effective threshold below the theoretical 50%. The specific number is illustrative, not measured.

- Li et al., "More Agents Is All You Need" (2024), TMLR. [arXiv:2402.05120](https://arxiv.org/abs/2402.05120) — Shows that performance gains from sampling-and-voting correlate with task difficulty (lower single-agent accuracy = more benefit from multiple agents). Does not state a specific threshold.
- Du et al., "Improving Factuality and Reasoning in Language Models through Multiagent Debate" (2023). [arXiv:2305.14325](https://arxiv.org/abs/2305.14325) — Multi-agent debate improves over single agents on reasoning tasks.

## Independent test authorship as highest-impact structural decision

**Category**: Research-backed (directional); "highest-impact" is editorial

Multiple systems demonstrate significant gains from separating code generation and test generation into distinct agents. No paper does an ablation study isolating this specific decision against other multi-agent design choices, so the superlative "single highest-impact" is a practitioner judgment, not a measured comparison.

- Huang et al., "AgentCoder: Multi-Agent-based Code Generation with Iterative Testing and Optimisation" (2023). [arXiv:2312.13010](https://arxiv.org/abs/2312.13010) — Separate programmer, test designer, and test executor agents achieve 96.3% pass@1 on HumanEval vs. 90.2% prior SOTA. The separation of test generation is the key architectural innovation.
- Huang, Chen et al., "Large Language Models Cannot Self-Correct Reasoning Yet" (2023), ICLR 2024. [arXiv:2310.01798](https://arxiv.org/abs/2310.01798) — Without external feedback, LLMs struggle to self-correct, supporting the need for an independent verifier.
- Qian et al., "ChatDev: Communicative Agents for Software Development" (2023), ACL 2024. [arXiv:2307.07924](https://arxiv.org/abs/2307.07924) — Separated testing phase catches bugs that inline testing misses.

## Plan-implement-verify as minimum effective team

**Category**: Research-backed (observed pattern); "consistently identifies" overstates consensus

Multiple prominent multi-agent systems converge on a three-phase pattern with distinct planning, implementation, and verification roles. This is an accurate synthesis of the observed trend, though no survey paper formally identifies it as the minimum effective configuration.

- Hong et al., "MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework" (2023). [arXiv:2308.00352](https://arxiv.org/abs/2308.00352) — Uses SOPs mapping to planning, implementation, and verification stages.
- Anthropic, "Building Effective Agents" (Dec 2024). [anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents) — Describes evaluator-optimiser and orchestrator-worker patterns that map to plan-implement-verify.
- Andrew Ng, "Agentic Design Patterns Part 5: Multi-Agent Collaboration" (Apr 2024). [deeplearning.ai/the-batch/agentic-design-patterns-part-5-multi-agent-collaboration](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-5-multi-agent-collaboration/) — Describes role-based decomposition (engineer, QA, etc.) referencing ChatDev and AutoGen.

## 70/20/10 call volume split

**Category**: Practitioner heuristic

No published source. This is a cost-optimisation rule of thumb for budgeting model usage across tiers. Anthropic's agent blog mentions routing easy questions to smaller models and hard ones to larger models but provides no ratio. The specific numbers are chosen to be memorable and to flag when frontier usage is too high (>30%).

## 85% per-step accuracy compounding

**Category**: Research-adjacent

The arithmetic is trivially correct (0.85^10 ≈ 0.197). The 85% baseline is a plausible illustrative figure, not a measured per-step accuracy from any benchmark. It's chosen because it's high enough to seem good individually but compounds dramatically, making it a useful pedagogical example for why agent chains need deterministic gates at each step.

The Agent Skills best practices independently arrive at the same conclusion via their "validation loops" and "plan-validate-execute" patterns: do the work, run a validator, fix issues, repeat. This is deterministic gating at each step rather than trusting the agent to self-evaluate.

For context on actual multi-step agent performance:

- Jimenez et al., "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?" (2023), ICLR 2024. [arXiv:2310.06770](https://arxiv.org/abs/2310.06770) — Best model at publication solved 1.96% of real GitHub issues, implying very low effective accuracy for complex multi-step tasks.
- Yang et al., "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering" (2024). [arXiv:2405.15793](https://arxiv.org/abs/2405.15793) — 12.5% on SWE-bench with GPT-4.
- Andrew Ng, "Agentic Design Patterns Part 1" (Mar 2024). [deeplearning.ai/the-batch/how-agents-can-improve-llm-performance](https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/) — GPT-3.5 at 48.1% zero-shot on HumanEval, GPT-4 at 67%, agentic GPT-3.5 at 95.1% (single-task, not per-step).

## Frontier models tend to refactor beyond requested scope

**Category**: Practitioner heuristic

Widely observed in developer communities but not formally studied. No published paper quantifies the tendency of frontier models to make unsolicited code changes. The advice to constrain scope in system prompts is standard engineering practice for working with highly capable models.

## Training data staleness causing incorrect API calls

**Category**: Practitioner heuristic (well-known real problem)

All major LLM providers document training data cutoff dates. The specific manifestation — agents generating deprecated API calls — is widely reported but never formally studied in isolation. Anthropic's MCP documentation and their agent blog implicitly acknowledge this by recommending tool use and documentation retrieval over parametric knowledge.

The Agent Skills best practices make a related point: effective skills are "grounded in real expertise" and domain-specific context, not LLM general knowledge. Their guidance to "start from real expertise" and "synthesize from existing project artifacts" reflects the same principle — parametric knowledge is insufficient for project-specific or fast-changing APIs.

- Agent Skills, "Best practices for skill creators" (2025). [agentskills.io/skill-creation/best-practices](https://agentskills.io/skill-creation/best-practices) — "A common pitfall in skill creation is asking an LLM to generate a skill without providing domain-specific context — relying solely on the LLM's general training knowledge."

## Mid-tier handles 80–90% of coding tasks

**Category**: Practitioner heuristic

No published source measures this ratio. It reflects the general observation that most coding tasks (feature implementation, test writing, refactoring within a known scope) do not require frontier-level reasoning. The range is deliberately vague to acknowledge variance across projects.

## Keep agent prompts under 500 words

**Category**: Practitioner heuristic, supported by spec guidance

The skill advises keeping each agent's instruction file under 500 words. No study measures the optimal length for agent system prompts, but the principle is supported by the Agent Skills specification's progressive disclosure design and best practices on context efficiency.

- Agent Skills, "Specification" (2025). [agentskills.io/specification](https://agentskills.io/specification) — Recommends keeping `SKILL.md` under 500 lines and 5,000 tokens. "Keep individual reference files focused. Agents load these on demand, so smaller files mean less use of context."
- Agent Skills, "Best practices for skill creators" (2025). [agentskills.io/skill-creation/best-practices](https://agentskills.io/skill-creation/best-practices) — "Overly comprehensive skills can hurt more than they help — the agent struggles to extract what's relevant and may pursue unproductive paths triggered by instructions that don't apply to the current task. Concise, stepwise guidance with a working example tends to outperform exhaustive documentation."
