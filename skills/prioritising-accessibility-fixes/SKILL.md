---
name: prioritising-accessibility-fixes
description: Use this skill to prioritise a set of accessibility issues for remediation based on severity, user impact, and effort. Triggers when triaging an accessibility backlog, deciding what to fix first after an audit, planning an accessibility sprint, or asking which accessibility issues matter most.
compatibility: Works best when issues already have effort estimates (from estimating-accessibility-effort) but can assess effort independently.
---

# Prioritising Accessibility Fixes

Given a set of accessibility issues, produce a prioritised remediation order based on severity, user impact, and effort. The output is a ranked list that helps decide what to fix first, what to batch together, and what can wait.

This skill works on a set of known issues. It does not find new issues (use `reviewing-accessibility` for that) and it does not estimate effort from scratch (use `estimating-accessibility-effort` for that, or provide estimates as input). If issues arrive without effort estimates, assess effort by reading the affected code before prioritising.

## Inputs

A set of accessibility issues, ideally with:

- Description of the problem
- WCAG success criterion
- Severity (Critical / Serious / Moderate)
- Affected pages or components
- Effort estimate (XS / S / M / L / XL)

If effort estimates are missing, read the affected code and estimate before prioritising. Do not prioritise without understanding effort — a "Critical" issue that takes 5 minutes should be fixed immediately, while a "Critical" issue requiring an architectural redesign needs different planning.

### Effort sizes

If issues arrive without effort estimates and `estimating-accessibility-effort` is not available, use this scale:

| Size | Typical scope |
|------|--------------|
| **XS** | Single attribute or property change, one file |
| **S** | Localised change within one component, a few attributes |
| **M** | Changes to one component plus its consumers, or 2-5 files |
| **L** | Structural changes affecting multiple components or a shared layout |
| **XL** | Architectural changes affecting routing, state management, or component library |

## Scoring model

Each issue is scored on three dimensions. The priority score determines the remediation order.

### Severity (WCAG compliance level)

| Rating | Definition | Score |
|--------|-----------|-------|
| Critical | WCAG Level A violation. The feature is unusable for some disability group. | 3 |
| Serious | WCAG Level AA violation. The feature is degraded but partially usable. | 2 |
| Moderate | Best practice or WCAG AAA. Usability improvement, not a compliance failure. | 1 |

### User impact

User impact measures how many people are affected and how severely their experience is degraded. This is distinct from WCAG severity — a Serious (AA) issue that affects every keyboard user on every page has higher user impact than a Critical (A) issue on a rarely visited admin screen.

| Rating | Definition | Score |
|--------|-----------|-------|
| High | Affects a core user flow (navigation, auth, primary actions) or affects users across many pages. Multiple disability groups impacted. | 3 |
| Medium | Affects a secondary flow or a specific page. One disability group primarily impacted. | 2 |
| Low | Affects an edge case, a rarely used feature, or a minor inconvenience rather than a barrier. | 1 |

### Effort (inverse — lower effort scores higher)

Lower effort means faster delivery of accessibility improvement. All else being equal, fix the easy things first.

| Rating | Effort sizes | Score |
|--------|-------------|-------|
| Low effort | XS, S | 3 |
| Medium effort | M | 2 |
| High effort | L, XL | 1 |

### Priority score

```
Priority = Severity + User Impact + Effort Score
```

Maximum score: 9 (Critical severity, High user impact, Low effort).
Minimum score: 3 (Moderate severity, Low user impact, High effort).

Issues with the same score are ordered by severity first, then user impact, then effort.

## Priority tiers

Group scored issues into tiers for practical planning:

### Tier 1 - Fix immediately (score 7-9)

High severity and impact issues that are quick to fix. These deliver the most accessibility improvement per unit of effort. In a sprint, do these first. Many are "quick wins" — a missing `alt` text on a hero image, a button without an accessible name, a form field without a label.

Also include in Tier 1: any issue scoring 6+ where severity is Critical, regardless of effort. WCAG Level A violations are legal and ethical blockers that should not wait for a convenient sprint.

### Tier 2 - Plan for next sprint (score 5-6)

Significant issues that need more effort or planning. These should be scheduled, not deferred indefinitely. Typical examples: implementing keyboard navigation for a custom widget, adding focus management to modal flows, building an error handling pattern for forms.

### Tier 3 - Schedule when capacity allows (score 3-4)

Low-impact or high-effort issues. Still worth fixing, but other work delivers more accessibility value per effort. Examples: AAA contrast improvements, optimising for edge-case assistive technology combinations, progressive enhancement of already-functional features.

### Exceptions to the scoring model

The scoring model is a starting point, not a rigid formula. Override the score when:

- **Legal or regulatory pressure** — if a specific WCAG criterion is legally required (e.g., public sector procurement, legal settlement), promote it to Tier 1 regardless of score.
- **User-reported issues** — a real user hitting a real barrier takes priority over theoretical audit findings at the same severity level.
- **Dependency chains** — an issue that blocks other fixes should be promoted. If "establish focus management utility" (L effort, score 5) is a prerequisite for five other focus-related fixes, do it first.
- **Regression** — a feature that was previously accessible and regressed should be prioritised over a feature that was never accessible. Regressions indicate a process problem that will recur.

## Batch related fixes

After scoring, identify issues that should be fixed together because:

- **Same component** — three accessibility issues in the same dialog are one task, not three.
- **Same pattern** — adding `aria-describedby` to 12 form fields is one effort with the pattern established on the first.
- **Dependency** — the focus management utility must exist before individual focus fixes can use it.

Present batches as grouped items with a combined effort estimate and a note on why they belong together.

## Report format

```markdown
## Accessibility Prioritisation - [scope]

### Tier 1 - Fix Immediately

| # | Issue | WCAG | Severity | Impact | Effort | Score |
|---|-------|------|----------|--------|--------|-------|
| 1 | ... | ... | ... | ... | ... | ... |

### Tier 2 - Plan for Next Sprint

| # | Issue | WCAG | Severity | Impact | Effort | Score |
|---|-------|------|----------|--------|--------|-------|
| 1 | ... | ... | ... | ... | ... | ... |

### Tier 3 - Schedule When Capacity Allows

| # | Issue | WCAG | Severity | Impact | Effort | Score |
|---|-------|------|----------|--------|--------|-------|
| 1 | ... | ... | ... | ... | ... | ... |

### Batches

- **[batch name]**: [issue A], [issue B], [issue C] - [combined effort] - [why batched]

### Recommended Order

[Numbered list of the specific order to tackle fixes, accounting for dependencies, batches, and tier placement. This is the actionable output — a developer should be able to start at #1 and work down.]

### Summary

- Tier 1: [n] issues ([n] quick wins)
- Tier 2: [n] issues
- Tier 3: [n] issues
- Total: [n] issues
```

Omit empty tiers.

## Gotchas

- **Severity alone is not priority.** A Critical issue requiring an XL architectural redesign ranks lower than a Serious issue with an XS fix. The scoring model exists to prevent severity-only triage, which leads to large Critical items blocking all other progress.
- **User impact requires judgment, not just WCAG mapping.** Two issues can both violate 2.1.1 (Keyboard) — one on the main navigation (High impact, every user on every page) and one on a settings toggle (Low impact, rare interaction). Same criterion, very different priority.
- **Quick wins are disproportionately valuable.** A batch of 10 XS/S fixes might take less time than one L fix but improve accessibility across more pages and for more users. Always surface quick wins prominently.
- **Do not defer all large items.** If the scoring model puts all L/XL items in Tier 3, the project will never fix structural accessibility problems. Ensure at least one L or XL item is scheduled in Tier 2 per planning cycle if structural issues exist.
- **Priorities change.** A Tier 3 issue becomes Tier 1 when a user reports it, when a legal requirement emerges, or when a dependent fix promotes it. Revisit prioritisation periodically rather than treating it as a one-time exercise.
- **Do not invent impact assessments.** If the project has analytics, use them — a page with 10,000 daily visits has higher impact than one with 10. If there are no analytics, assess based on the feature's centrality to the product's purpose, not assumptions about traffic.
- **Accessibility debt compounds.** Five Moderate issues in the same component often indicate a pattern problem. Consider whether fixing the pattern (one M/L task) is more effective than five individual fixes.
