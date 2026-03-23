# Skills

This is Matt Obee's (@mattobee) personal collection of reusable agent skills, built on the [Agent Skills specification](https://agentskills.io/specification).

## Repo Structure

```
skills/
└── [skill-name]/
    ├── SKILL.md              # Required: frontmatter + instructions
    ├── references/           # Optional: supporting docs the agent can read on demand
    ├── scripts/              # Optional: executable code
    └── assets/               # Optional: templates, static resources
README.md                     # Skill listing with install commands
LICENSE                       # MIT
```

## Adding a New Skill

1. Create `skills/[skill-name]/SKILL.md` with required frontmatter (`name`, `description`) and instructions.
2. The `name` field must match the directory name exactly. Lowercase, hyphens only, no consecutive hyphens, 1–64 characters.
3. Skill names use gerund form (e.g., `designing-agent-teams`, `suggesting-next-steps`, not `agent-team-design` or `next-steps`).
4. Add the skill to the "Available Skills" section in `README.md` with a description and install command.
5. Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/`.

## Writing Style

- No "we" — there is no "we".
- No "the skill" as a grammatical subject where avoidable.
- Use headings, not bold text, for list items that serve as section headers.
- Use descriptive link text woven into prose, not bare URLs appended at the end.
- Recommendations, not requirements — frame guidance as suggestions unless there is a strong reason to mandate.

## SKILL.md Conventions

- `description` frontmatter: 1–2 sentences. Include what the skill does and when it triggers. Detailed trigger lists belong in the body.
- Instructions should be actionable steps, not background essays. The agent loads the full body when it activates the skill.
- Reference files (`references/`) are loaded on demand. Keep them focused — one concern per file. Tell the agent *when* to load each file ("Read `references/api-errors.md` if the API returns a non-200 status"), not just "see references/ for details."
- Follow the [Agent Skills spec](https://agentskills.io/specification) for frontmatter fields and directory layout.

## Skill Authoring Guidelines

These are distilled from the [Agent Skills best practices](https://agentskills.io/skill-creation/best-practices) and [description optimisation guide](https://agentskills.io/skill-creation/optimizing-descriptions).

### Ground skills in real expertise

Do not generate skills from scratch using only the LLM's training knowledge. Every skill should be grounded in real tasks, project artifacts, corrections made during actual use, or existing documentation. Generic advice ("handle errors appropriately") is not useful — specific API patterns, edge cases, and project conventions are.

### Write what the agent doesn't know

Focus on what the agent would get wrong without the skill: project-specific conventions, non-obvious edge cases, particular tools or APIs to use. Do not explain basics the agent already knows. If unsure whether an instruction adds value, test it — remove it and see if the agent still gets the task right.

### Descriptions carry the triggering burden

The `description` frontmatter is how agents decide whether to load a skill. Get it wrong and the skill either never triggers or triggers on everything.

- Use imperative phrasing: "Use this skill when..." not "This skill does..."
- Focus on user intent, not implementation details.
- Err on the side of being pushy — list contexts where the skill applies, including cases where the user doesn't name the domain directly.
- Max 1024 characters per the spec.

### Provide defaults, not menus

When multiple approaches could work, pick one default and mention alternatives briefly. Do not present equal-weight options — the agent will hesitate or pick arbitrarily.

### Favor procedures over declarations

Teach the agent *how to approach* a class of problems, not *what to produce* for a specific instance. The approach should generalise even when individual details are specific.

### Gotchas sections are high-value

A list of concrete, environment-specific gotchas is often the most valuable part of a skill. These are facts that defy reasonable assumptions — not general advice, but specific corrections to mistakes the agent will make without being told. When an agent makes a mistake during use, add the correction as a gotcha.

### Use deterministic validation

Instruct the agent to validate its work with scripts, linters, test suites, or reference checklists — not self-evaluation. For batch or destructive operations, have the agent create a plan, validate it against a source of truth, then execute.

## Commit Conventions

- Atomic commits: one logical change per commit.
- Every commit must be signed (SSH key).
- Messages: imperative mood, concise first line summarising the change.

## Model Tiers

Skills in this repo use a three-tier model framework (frontier / mid-tier / fast) rather than provider-specific model names. This keeps skills portable across providers. See `designing-agent-teams/references/model-guide.md` for the full mapping.
