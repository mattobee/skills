# TaskFlow Agent Team

## Team Overview

TaskFlow is a task management web app (~15k lines TypeScript/React, 80 source files) with a Prisma/PostgreSQL backend (12 tables), real-time WebSocket collaboration, and enterprise accessibility and security requirements. Solo developer, balancing quality and cost.

### Why a Team?

The project has 5 separable concerns with independent file boundaries: frontend implementation, testing, accessibility review, security review, and database operations. A single agent would context-switch between WCAG review, RBAC auditing, React components, and test authoring. The independent Tester is the highest-impact structural decision: a coder testing its own code has confirmation bias. The dual-touchpoint specialists (Accessibility, Security) catch domain issues at planning time rather than after implementation.

## Agent Roster

### 1. Lead (frontier)

- **Model**: Frontier
- **Rationale**: Task decomposition across 80 source files, real-time collaboration features, and multiple domain layers. Top-tier reasoning for cross-cutting architectural decisions.
- **Tools**: read, search (read-only), git commands
- **Scope**: Task planning, architectural decisions, review, escalation target
- **Escalation**: Terminal
- **Instructions**: `.agents/lead.md`

### 2. Coder (mid-tier)

- **Model**: Mid-tier
- **Rationale**: Bulk implementation at the best quality/speed/cost balance. Handles React components, API routes, WebSocket handlers, and Tailwind styling.
- **Tools**: read, write, shell, search
- **Scope**: `src/` (excluding test files and Prisma schema)
- **Escalation**: Lead for multi-system complexity or architectural ambiguity
- **Instructions**: `.agents/coder.md`

### 3. Tester (mid-tier)

- **Model**: Mid-tier
- **Rationale**: Independent test authorship avoids confirmation bias. Writes tests from the spec, not from the implementation.
- **Tools**: read, write (test files only), shell (test runners only)
- **Scope**: `src/**/*.test.ts(x)`, `e2e/**/*.spec.ts`, `e2e/fixtures/`, `e2e/helpers/`
- **Escalation**: Lead when spec is ambiguous or test failures reveal architectural issues
- **Instructions**: `.agents/tester.md`

### 4. Scout (fast)

- **Model**: Fast
- **Rationale**: Fastest and cheapest for file exploration. 80 files is well within a fast model's context window.
- **Tools**: read, search (read-only)
- **Scope**: File discovery, pattern searching, dependency tracing, context gathering
- **Escalation**: Coder (implementation context) or Lead (scope unclear)
- **Instructions**: `.agents/scout.md`

### 5. Accessibility Specialist (mid-tier)

- **Model**: Mid-tier
- **Rationale**: Enterprise clients require WCAG AA compliance. Called at two points: early (risk assessment before implementation) and late (code review after implementation).
- **Tools**: read, search, shell (`npm run test:a11y` only)
- **Scope**: Assesses plans for a11y risks; reviews all UI changes; does not write code
- **Escalation**: Lead for architectural a11y decisions
- **Instructions**: `.agents/accessibility.md`

### 6. Security Specialist (mid-tier)

- **Model**: Mid-tier
- **Rationale**: Multi-tenant RBAC, API keys, team invitations — security review is a cross-cutting concern. Called at two points: early (threat assessment before implementation) and late (code review after implementation).
- **Tools**: read, search, shell (`npm audit` only)
- **Scope**: Assesses plans for security risks; reviews auth flows, input validation, RBAC policies; does not write code
- **Escalation**: Lead for security architecture decisions
- **Instructions**: `.agents/security.md`

## Orchestration Pattern

```
User Request
    |
    v
Lead (frontier)
    |
    +-- 1. @scout -> gather context, find affected files
    |
    +-- 2. @accessibility -> early risk assessment (UI tasks only)
    |
    +-- 3. @security -> early threat assessment (auth/data tasks only)
    |
    +-- 4. @coder -> implement feature/fix (with specialist recommendations)
    |
    +-- 5. @tester -> write tests independently from coder
    |
    +-- 6. @accessibility -> post-implementation review (UI tasks only)
    |
    +-- 7. @security -> post-implementation review (auth/data tasks only)
    |
    +-- 8. Lead reviews and synthesises
```

**Flow**: Scout first (cheap context), then specialists for early risk/threat assessment on relevant tasks, then Coder (with specialist recommendations), then Tester, then specialists again as quality gates. Lead bookends the workflow.

**Parallel paths**: Scout runs first alone. Early a11y and security assessments can run in parallel (independent concerns). Coder -> Tester -> late reviews are sequential. Late a11y and security reviews can run in parallel.

**Quality gates are deterministic**: `npm run typecheck`, `npm run lint`, `npm run test`, `npm run build`. Specialist reviews add value for nuanced issues but are not the primary gate.

**Narration**: The Lead explains each delegation before and after calling an agent. This is critical for user visibility into the multi-agent workflow.

## Escalation Map

```
Scout (fast) ----------> Coder (mid-tier) --> Lead (frontier)
                                           /
Tester (mid-tier) -----------------------'
                                           /
A11y Specialist (mid-tier) --------------'
                                           /
Security Specialist (mid-tier) ----------'
```

**Triggers**:

- Scout -> Coder: needs implementation context to understand a pattern
- Scout -> Lead: scope of search unclear, needs task decomposition
- Coder -> Lead: multi-system change, architectural ambiguity, >3 files affected unexpectedly
- Tester -> Lead: spec is ambiguous, test failures reveal architectural issues
- A11y Specialist -> Lead: accessibility fix requires architectural redesign
- Security Specialist -> Lead: security fix requires architectural redesign

## Cost Projection

| Agent               | Tier     | Est. Call Volume | Relative Cost |
| ------------------- | -------- | ---------------- | ------------- |
| Scout               | Fast     | ~50%             | Low           |
| Coder               | Mid-tier | ~15%             | Medium        |
| Tester              | Mid-tier | ~10%             | Medium        |
| A11y Specialist     | Mid-tier | ~8%              | Medium        |
| Security Specialist | Mid-tier | ~8%              | Medium        |
| Lead                | Frontier | ~9%              | High          |

**Distribution**: ~50% fast, ~41% mid-tier, ~9% frontier. Slightly below the 70/20/10 target for fast because dual-touchpoint specialists increase mid-tier volume. Justified by the project's enterprise requirements.

**Cost levers**: Prompt caching reduces frontier cost (shared context). Scout's high volume at fast-tier pricing keeps total cost low. Both specialists are skipped entirely for tasks that don't touch their domain (pure refactoring skips both; backend-only work skips a11y).
