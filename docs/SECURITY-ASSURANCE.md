<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# Security assurance case — typo3-a11y-skill

This document states what a user can expect from this repository in terms of security, and argues why that expectation holds. Every claim names the file or setting that implements it. Reporting a vulnerability: see the [security policy](https://github.com/netresearch/.github/blob/main/SECURITY.md). Components: [ARCHITECTURE.md](ARCHITECTURE.md).

## What the repository ships

| Part | Files | Runs where |
| --- | --- | --- |
| Skill instructions for an AI agent | `skills/typo3-a11y/SKILL.md`, `skills/typo3-a11y/references/*.md` | Read by the agent as instructions; not executed. The Fluid, HTML, SCSS and TypeScript snippets in them are examples the agent or the user adapts into a TYPO3 sitepackage. |
| Evals | `evals/evals.json` | Read by `validate-evals.sh` in CI; the prompts and regular expressions are data. |
| Package metadata | `composer.json`, `plugin.json`, `.claude-plugin/plugin.json` | Read by Composer and by Claude Code when the skill is installed. |
| Repository configuration | `.github/workflows/*.yml`, `.pre-commit-config.yaml`, `.yamllint.yml`, `.markdownlint-cli2.jsonc`, `renovate.json` | In this repository's CI and on contributors' machines. |

The repository ships no script, no server component and no container image. It stores nothing and handles no user accounts or credentials. The release archives contain `SKILL.md`, the references and the two licence files; the plugin archive also contains `.claude-plugin/`. The skill-repo-skill release reusable called by `release.yml` copies only `SKILL.md`, `references`, `scripts`, `assets`, `templates`, `examples`, `checkpoints.yaml`, the licence files and, for the plugin archive, `.claude-plugin` and `hooks`; of those, only the paths named above exist here.

## Security requirements

1. The skill content does not recommend frontend code that inserts text into the page as markup.
2. A change reaches `main` through a pull request whose commits are signed and signed off and that passes the required checks. Branch protection is not enforced for repository administrators.
3. Nothing committed to this repository contains a secret.
4. A release carries the version that `.claude-plugin/plugin.json` states, and its archives can be verified against the build that produced them.

## Actors and trust boundaries

- **Skill user and agent.** The agent reads `SKILL.md` and the references and writes code into the user's project with the tools the user has given it. What it writes and runs is decided by the agent and the user, not by this repository. `SKILL.md` declares no `allowed-tools`.
- **Consumer project.** Code derived from the snippets runs in the consumer's TYPO3 frontend, in visitors' browsers. From that point it is the consumer's code; this repository has no runtime connection to it.
- **Contributors.** Changes reach `main` through pull requests, checked by the workflows in `.github/workflows/` and by the required checks listed below. A contributor who runs `composer install` gets the pre-commit hooks from `.pre-commit-config.yaml` installed (`composer.json`, script `install-hooks`).
- **CI.** Workflows run on GitHub-hosted runners. The two `pull_request_target` callers (`pr-quality.yml`, `auto-merge-deps.yml`) set `permissions: {}` at the top level, grant the job only `pull-requests: write` (plus `contents: write` for the merge), and call reusables that approve or merge without checking out pull request code; `pr-quality.yml` states this in its header comment.
- **Dependency bot.** Renovate (`renovate.json`, preset `github>netresearch/renovate-config`) opens pull requests that bump the pinned `rev:` of the pre-commit hooks; `auto-merge-deps.yml` approves and merges pull requests whose author is `renovate[bot]` or `dependabot[bot]`.

## Threats and countermeasures

| Threat | Countermeasure | Evidence |
| --- | --- | --- |
| The skill teaches markup injection (CWE-79): a snippet writes text into the DOM as HTML | The TypeScript snippets build elements with `document.createElement` and set text with `textContent`; no snippet assigns `innerHTML` or `outerHTML`, calls `insertAdjacentHTML` or `eval`, or writes to the document stream. The breadcrumb JSON-LD encodes titles and links with `f:format.json()` | `references/patterns-toast-notification.md`, `references/patterns-accessible-filter.md`, `references/patterns-skeleton-loading.md`, `references/patterns-breadcrumb.md` |
| A link opened in a new tab gets access to the opener page | The new-window example sets `rel="noopener noreferrer"` with `target="_blank"` | `references/accessibility.md` ("Links Opening in New Window") |
| A change to the skill content or CI is merged without its checks | Branch protection of `main` requires the status checks `Skill Validation`, `Eval Validation`, `Analyze (actions)` and `DCO`, up to date with `main`, signed commits and resolved conversations, and blocks force pushes and branch deletion (repository settings, read 2026-09-30) | `.github/workflows/validate.yml`, `.github/workflows/eval-validate.yml` |
| Pull request code runs with a write token (CWE-829) | The `pull_request_target` callers check out no pull request code and grant only the scopes their reusable needs | `.github/workflows/pr-quality.yml`, `.github/workflows/auto-merge-deps.yml` |
| Insecure workflow patterns | CodeQL default setup analyses the GitHub Actions workflows on pull requests (`Analyze (actions)`, required); actionlint runs in Skill Validation and in the pre-commit hooks | repository settings; `.github/workflows/validate.yml`, `.pre-commit-config.yaml` |
| A secret is committed | GitHub secret scanning with push protection is enabled for the repository | repository settings, read 2026-09-30 |
| A malformed manifest or eval reaches `main` | Skill Validation parses every tracked `*.json`, compares the two plugin manifests and checks the version format; Eval Validation checks `evals/evals.json` and, for a new or changed eval, that its `samples` match its assertions | `.github/workflows/validate.yml`, `.github/workflows/eval-validate.yml` |
| A release is built from a forged tag or with a version that disagrees with `plugin.json` | The release reusable accepts only annotated tags that GitHub reports as signed, and fails when the tag differs from `.claude-plugin/plugin.json` | `.github/workflows/release.yml` |
| A released archive is tampered with | The release reusable publishes a Cosign-signed (keyless) `SHA256SUMS.txt` and build-provenance attestations for the archives | `.github/workflows/release.yml` |
| A pre-commit hook changes underneath contributors | Each hook is pinned by `rev:`; a new revision arrives only as a Renovate pull request | `.pre-commit-config.yaml`, `renovate.json` |

## Secure design principles applied

- **Economy of mechanism:** the repository is Markdown and configuration. It ships no executable code, so the product's attack surface is the text an agent reads.
- **Least privilege:** workflows grant each job the scopes its reusable needs; the read-only checks run with `contents: read`.
- **Safe defaults in examples:** the snippets create DOM nodes and set text rather than parse markup, and use native elements (`<button>`, `<dialog>`, `<nav>`) whose keyboard and focus behaviour the browser provides.

## Dynamic analysis

The repository ships no executable code that takes input: the TypeScript and Fluid snippets are examples inside Markdown and are not built or run here. Dynamic analysis and runtime assertions therefore do not apply to this repository. The evals in `evals/evals.json` are structural checks of expected answers, not a dynamic analysis.

## What a user cannot expect

- The required checks are automated. Branch protection requires no approving review, `pr-quality.yml` approves pull requests of collaborators with write access, and repository administrators are exempt from branch protection (repository settings, read 2026-09-30).
- The skill gives guidance; it does not enforce it. The agent decides what to write, and the result needs the same review as any other code change.
- The snippets are examples to adapt. They do not validate URLs or other values taken from data attributes or TYPO3 records; the consumer's templates decide what reaches them.
- No dependency-vulnerability check (dependency review, Composer Audit) and no SAST for the snippet languages run on pull requests. The only declared Composer dependency is `netresearch/composer-agent-skill-plugin` with the constraint `*`, and no lock file is committed.
- The reusable workflows are referenced at `@main` of `netresearch/skill-repo-skill` and `netresearch/.github`, so a change there applies here without a change in this repository.
- Security fixes follow the supported-versions rules of the organisation's security policy; older releases may not receive them.
