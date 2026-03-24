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

### reviewing-accessibility

Reviews UI changes for WCAG Level AA compliance using a dual-touchpoint approach: early risk assessment before implementation, and post-implementation code review. Covers semantic HTML, keyboard navigation, focus management, ARIA, labels, visual contrast, and forms.

```bash
npx skills add mattobee/skills --skill reviewing-accessibility
```

Example prompts:

- "Review this component for accessibility"
- "What accessibility risks should I consider before building this feature?"
- "Audit this page against WCAG AA"
- "Is my form handling accessible?"

### writing-accessibility-tests

Writes Playwright accessibility tests using a two-layer strategy: axe-core scans for broad automated coverage, plus targeted Playwright assertions for things axe cannot catch (accessible names, ARIA states, focus management, shadow DOM, multi-theme contrast).

```bash
npx skills add mattobee/skills --skill writing-accessibility-tests
```

Example prompts:

- "Add accessibility tests for this page"
- "Write axe scans for all routes in light and dark mode"
- "What accessibility test coverage am I missing?"
- "How do I test accessible names on web components with shadow DOM?"

### predicting-accessibility-risks

Identifies accessibility risks in proposed features, designs, or technical plans before implementation begins. Produces a risk assessment with affected user groups, WCAG criteria, likelihood, cost to fix later, and specific mitigations.

```bash
npx skills add mattobee/skills --skill predicting-accessibility-risks
```

Example prompts:

- "What accessibility risks should I watch for in this feature?"
- "Assess this design for accessibility problems before I build it"
- "What could go wrong for disabled users with this approach?"
- "Is there anything about this plan that will be expensive to fix for accessibility later?"

### estimating-accessibility-effort

Estimates the effort required to remediate accessibility issues. Takes audit findings, axe-core results, or backlog items and produces T-shirt-sized effort estimates with rationale, dependencies, and quick wins.

```bash
npx skills add mattobee/skills --skill estimating-accessibility-effort
```

Example prompts:

- "How much effort would it take to fix these accessibility issues?"
- "Size these audit findings for sprint planning"
- "Which of these accessibility fixes are quick wins?"
- "What dependencies exist between these accessibility issues?"

### prioritising-accessibility-fixes

Prioritises a set of accessibility issues for remediation based on severity, user impact, and effort. Produces a scored, tiered remediation plan with batching recommendations and a concrete fix order.

```bash
npx skills add mattobee/skills --skill prioritising-accessibility-fixes
```

Example prompts:

- "Prioritise these accessibility issues for our next sprint"
- "What should we fix first from this audit report?"
- "Triage this accessibility backlog"
- "Which accessibility fixes give the most value for the least effort?"

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
