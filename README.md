<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# typo3-a11y-skill

WCAG 2.2 AA accessibility patterns for TYPO3 v13+ sitepackage frontend development. A Claude Code skill that provides comprehensive accessibility guidelines, HTML/ARIA patterns, SCSS examples, and TypeScript implementations.

## Installation

### Claude Code Marketplace

```bash
claude install netresearch/typo3-a11y-skill
```

### Composer

```bash
composer require netresearch/typo3-a11y-skill
```

## References

| File | Description |
|---|---|
| `accessibility.md` | WCAG 2.2 AA comprehensive guide -- language, landmarks, headings, links, buttons, color, focus, ARIA, testing |
| `patterns-skiplinks.md` | Mandatory skip link navigation with Fluid, SCSS, and Playwright tests |
| `patterns-accessible-navigation.md` | Main navigation, submenus, mobile toggle with b13/menus TreeMenu |
| `patterns-accessible-forms.md` | Form labels, error handling, fieldsets, multi-step forms |
| `patterns-accessible-filter.md` | Filtering, pagination, sorting, semantic table structure |
| `patterns-disclosure-widget.md` | Accordions, collapsible sections, content hiding techniques |
| `patterns-clickable-cards.md` | Accessible clickable-card pattern, with rejected alternatives noted |
| `patterns-responsive-tables.md` | Horizontal scroll and card reflow patterns for mobile tables |
| `patterns-sticky-header.md` | Scroll-triggered fixed header with IntersectionObserver |
| `patterns-lazy-loading.md` | Deferred component initialization with placeholder content |
| `patterns-breadcrumb.md` | Breadcrumb navigation with JSON-LD structured data |
| `patterns-language-switcher.md` | Multi-language navigation with b13/menus LanguageMenu |
| `patterns-animations.md` | Scroll animations with prefers-reduced-motion support |
| `patterns-scroll-to-anchor.md` | Smooth scroll with sticky header offset compensation |
| `patterns-skeleton-loading.md` | CSS placeholder animations for content loading |
| `patterns-toast-notification.md` | Auto-dismiss notifications with ARIA live region |
| `patterns-back-to-top.md` | Scroll-to-top button with visibility threshold |

## Tests

The repository ships Markdown and configuration, no executable code. Its tests are the structural validators and the evals:

- `evals/evals.json` holds one eval per rule that is easy to state and easy to get wrong: a prompt an agent might receive, regular-expression assertions a correct answer must match, and `samples` of a passing and failing answer.
- `validate-evals.sh` (from `netresearch/skill-repo-skill`) checks the structure of every eval and runs its assertions against its samples with the same `grep -E` the grader uses.
- `validate-skill.sh` (same source) checks the skill layout, the `SKILL.md` frontmatter and size, the manifests and the presence of `README.md`, the licence files and `.gitignore`.
- The pre-commit hooks in `.pre-commit-config.yaml` run `validate-skill.sh`, the version-parity check, markdownlint, yamllint, actionlint, JSON and YAML syntax, ruff and ShellCheck.

Run them locally from the repository root:

```bash
pre-commit run --all-files

base=https://raw.githubusercontent.com/netresearch/skill-repo-skill/main/skills/skill-repo/scripts
curl -fsSLO "$base/validate-skill.sh" && bash validate-skill.sh .
curl -fsSLO "$base/validate-evals.sh" && bash validate-evals.sh evals/evals.json
```

`validate-skill.sh` ends with `Errors:` and `Warnings:` counts and exits 1 when there is at least one error; warnings do not fail it. `validate-evals.sh` ends with `Results: N passed, M failed, K warnings` and exits 1 when `M` is not 0; each failing line names the eval and the check. A failed pre-commit hook prints its name followed by `Failed` and the tool's own output.

In CI, `validate.yml` (Skill Validation) and `eval-validate.yml` (Eval Validation) run these checks on every pull request and on pushes to `main`; both are required status checks for merging into `main`. On a pull request, Eval Validation compares `evals/evals.json` with the base branch, and every new eval, or eval with changed assertions, whose assertions carry a pattern must carry `samples.passing`. A pull request that adds a rule to the skill adds an eval that pins it.

## Dependencies

- **Composer:** `composer.json` requires `netresearch/composer-agent-skill-plugin` (constraint `*`), the Composer plugin for packages of type `ai-agent-skill`; `extra.ai-agent-skill` names the skill file. No lock file is committed: the package is installed as a dependency of other projects, whose lock files pin it.
- **Pre-commit hooks:** each hook repository in `.pre-commit-config.yaml` is pinned by `rev:`. `composer install` installs the hooks when `pre-commit` is available.
- **CI:** the workflows call reusable workflows of `netresearch/skill-repo-skill` and `netresearch/.github` at `@main`; those pin their actions by commit SHA.
- **Updates:** Renovate (`renovate.json`, preset `github>netresearch/renovate-config`) opens pull requests for new hook revisions; `auto-merge-deps.yml` merges Renovate and Dependabot pull requests once the required checks pass.
- **Selection:** a new dependency is added only when the skill or its tooling needs it, from its upstream source (Packagist, the tool's own repository), under a licence compatible with this repository's.

## Governance and policies

This repository follows the Netresearch organisation policies:

- [Governance](https://github.com/netresearch/.github/blob/main/GOVERNANCE.md): ownership, roles, how decisions are made and disputes resolved.
- [Roadmap](https://github.com/netresearch/.github/blob/main/ROADMAP.md): planned and explicitly excluded work for the coming year.
- [Handling of dependency and code analysis findings](https://github.com/netresearch/.github/blob/main/SECURITY.md#handling-of-dependency-and-code-analysis-findings): thresholds, deadlines and the exception process for dependency (SCA) and static analysis (SAST) findings.
- [Secret management](https://github.com/netresearch/.github/blob/main/SECURITY.md#secret-management): how CI and release credentials are stored, accessed and rotated.
- [Access roster](https://github.com/netresearch/.github/blob/main/docs/access-roster.md): who holds administrative access to this repository and the organisation.

The security assurance case for this skill (threat model, trust boundaries, countermeasures and limits) is in [docs/SECURITY-ASSURANCE.md](docs/SECURITY-ASSURANCE.md).

Checks that run on pull requests in this repository:

- Every pull request: Skill Validation (`validate.yml`: skill structure, manifest sync, markdownlint, yamllint, actionlint, JSON syntax, ShellCheck, ruff, checkpoint schemas) and Eval Validation (`eval-validate.yml`).
- Pull requests to `main`: Harness Verification (`harness-verify.yml`), CodeQL analysis of the GitHub Actions workflows (default setup) and the DCO sign-off check. Secret scanning with push protection is enabled for the repository.
- No dependency-vulnerability check (dependency review, Composer Audit) and no static security analysis of the Markdown snippets run here.

## License

This project uses split licensing:

- Code (`scripts/**`, `.github/workflows/**`, config files) is licensed under the [MIT License](LICENSE-MIT).
- Documentation and skill content (`skills/**`, `references/**`, `README.md`) is licensed under [CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0).

SPDX expression: `(MIT AND CC-BY-SA-4.0)`.

## Maintainer

Maintained by [Netresearch DTT GmbH](https://www.netresearch.de).
