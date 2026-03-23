# Example: TaskFlow

This is a fictional worked example demonstrating the agent team patterns described in the skill's best practices. TaskFlow does not exist — it's a reference for how these patterns come together in a realistic project.

## Project context

**TaskFlow** is a task management web app for small teams. Built with React, TypeScript, Next.js, Tailwind CSS, and Prisma with PostgreSQL. It has ~15k lines of code across 80 source files, a moderate database schema (12 tables), and real-time collaboration features via WebSockets.

Key characteristics that shape the agent team:

- **Accessibility matters** — the product targets enterprise teams, some of which have accessibility procurement requirements
- **Security matters** — multi-tenant with role-based access control, team invitations, and API keys
- **One developer** — solo developer building a prototype, balancing quality and cost

## Team design

The team has 6 agents. The three core agents (Lead, Coder, Tester) plus a Scout (cheap context gathering) and two dual-touchpoint specialists (Accessibility, Security).

See `team.md` for the full team specification.

## Patterns demonstrated

### Narration

The Lead explains each delegation before and after. This gives the developer visibility into the workflow without needing to inspect logs or databases.

### Dual-touchpoint specialists

Both the Accessibility Specialist and the Security Specialist are called twice:

1. **Early** — after the Scout gathers context, before the Coder implements. They assess the plan for domain-specific risks and provide recommendations.
2. **Late** — after implementation and testing. They review the actual code.

### Quality gate ordering

After implementation:

1. Tester runs first (functional correctness)
2. Accessibility Specialist reviews second (WCAG compliance)
3. Security Specialist reviews third (auth, input validation, RBAC)

This ordering avoids wasted reviews on code that might change due to test failures.

### Escalation

All specialists escalate to the Lead. The Scout escalates to the Coder (for implementation context) or Lead (for scope clarity). Escalation triggers are concrete: "test failures after an attempt", "change affects >3 files unexpectedly", "spec is ambiguous".
