# Architecture — typo3-a11y-skill

## Purpose

Provides accessibility guidance for TYPO3 v13 and v14.3 LTS sitepackage frontend
work as an agent skill, so an AI coding assistant produces markup, ARIA and CSS
that meet WCAG 2.2 Level AA rather than plausible-looking approximations.

## Skill Structure

The repo follows the [Agent Skills specification](https://agentskills.io/specification):

```
skills/typo3-a11y/
├── SKILL.md      # Entry point — frontmatter metadata + the rules that always apply
└── references/   # Detailed documents, loaded on demand
```

## Key Components

### SKILL.md (entry point)

YAML frontmatter (name, description) plus the recommendations that hold for every
sitepackage: skip links, underlined body links, never disabling buttons, no
`role="menu"` for navigation, zoomable viewport, `aria-expanded` on disclosure
triggers, and the per-content-element checklist. Agents read this first, so it
stays under the 500-word cap the `skill-repo` validator enforces (`wc -w` over
the whole file, frontmatter included) — anything longer belongs in a reference.

### References (lazy-loaded)

- **accessibility.md** — the comprehensive guide: landmarks, headings, links,
  buttons, colour and contrast (including the WCAG-versus-APCA policy), CSS units
  and user preferences, focus management, ARIA, automated testing, the WCAG 2.2
  additions, and the content-element checklist.
- **verifying-accessible-names.md** — how to check what a screen reader actually
  announces, rather than what the markup suggests.
- **patterns-\*.md** — one file per UI pattern (navigation, forms, filters,
  tables, disclosure widgets, skip links, clickable cards, sticky header, lazy
  loading, breadcrumb, language switcher, animations, scroll-to-anchor, skeleton
  loading, toast notifications, back-to-top). Each carries accessible markup, the
  keyboard and focus behaviour, and the TypeScript or SCSS that goes with it.

## Contrast policy

Two independent goals, kept apart deliberately: WCAG 2.2 AA is the compliance
gate and is not negotiable; APCA is an additional readability measurement and
never waives a WCAG failure. The reasoning, the thresholds and the common
misconceptions live in `references/accessibility.md`.

## Evaluation

`evals/evals.json` is the behavioural test suite: prompts an agent might receive,
with assertions on what a correct answer must contain. It is the place where a
rule that is easy to state and easy to get wrong (large-text thresholds, the
APCA-does-not-waive-WCAG rule) gets pinned down.

## CI

Workflows call the shared reusables in `netresearch/.github` and
`netresearch/skill-repo-skill`: skill-structure validation, eval validation,
harness consistency, DCO, CodeQL, dependency review and secret scanning. Releases
are cut by bumping `.claude-plugin/plugin.json`, merging, then pushing a signed
`vX.Y.Z` tag.
