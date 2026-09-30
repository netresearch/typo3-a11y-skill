<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# typo3-a11y-skill

WCAG 2.2 AA accessibility patterns for TYPO3 v13/v14 LTS sitepackage frontend development, distributed as a Claude Code skill.

## Repo Structure

```
typo3-a11y-skill/
├── skills/typo3-a11y/          # Skill definition and references
│   └── SKILL.md                # Skill metadata, trigger description, inline guidance
├── .claude-plugin/
│   └── plugin.json             # Plugin metadata (name, version, license)
├── .github/workflows/          # CI caller workflows (call netresearch/skill-repo-skill reusables; auto-merge-deps calls netresearch/.github)
├── composer.json               # Composer package definition (type: ai-agent-skill)
├── LICENSE-MIT                 # MIT license (applies to code, configs, CI workflows)
├── LICENSE-CC-BY-SA-4.0        # CC-BY-SA-4.0 (applies to skill content, docs, references)
└── README.md                   # Human-facing documentation
```

## Commands

No Makefile. Key operations:

- `composer install` — install dependencies and, via `post-install-cmd`, the pre-commit hooks
- `composer install-hooks` — install the pre-commit hooks on their own
- Validate skill repo structure: run `skill-repo-skill`'s `validate-skill.sh` against repo root. It counts the lines of the `SKILL.md` body (frontmatter excluded): more than 500 is an error, more than 300 a warning
- Release: bump `.claude-plugin/plugin.json` version, open PR, merge, signed tag `vX.Y.Z`, push tag — the `release.yml` caller builds the GitHub release

## Rules

- Skill behavior is defined by [skills/typo3-a11y/SKILL.md](skills/typo3-a11y/SKILL.md) — its `description` field must begin with `Use when` to activate correctly in Claude Code.
- Licensing follows the Netresearch split model: code under [LICENSE-MIT](LICENSE-MIT), documentation and skill content under [LICENSE-CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0). SPDX expression: `(MIT AND CC-BY-SA-4.0)`.
- Do not add a `version` field to [composer.json](composer.json) — versions are derived from git tags by the Release workflow.
- All CI workflows in [.github/workflows/](.github/workflows/) must remain thin callers of the shared reusable workflows in `netresearch/skill-repo-skill` and `netresearch/.github` — never inline actions in this repo.
- Release discipline: tag only **after** the bump PR is merged to `main`; tag-before-bump runs the release against the wrong version and produces a locked broken release.
- Conformance with the `skill-repo-skill` structural standard is enforced by the `validate.yml` caller on every PR.

## References

- [README.md](README.md) — human-facing documentation
- [skills/typo3-a11y/SKILL.md](skills/typo3-a11y/SKILL.md) — skill content and triggers
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — components and CI
- [docs/SECURITY-ASSURANCE.md](docs/SECURITY-ASSURANCE.md) — security assurance case: threats, trust boundaries, limits
- `netresearch/skill-repo-skill` (external) — source of truth for reusable CI workflows and structural conventions
