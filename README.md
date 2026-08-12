# skills

Agent skills for coding agents (Claude Code, Cursor, Codex, and anything else that reads `SKILL.md`). Focused on keeping codebases clean when agents do the writing.

Presented nicely at [p54.dev/skills](https://p54.dev/skills).

## Install

```bash
npx skills add pranshugupta54/skills
```

Or install a single skill:

```bash
npx skills add pranshugupta54/skills --skill docstring-cleanup
```

## Skills

| Skill | Description |
| --- | --- |
| [docstring-cleanup](skills/docstring-cleanup/SKILL.md) | Clean up feature-coupled docstrings and comments after working on a feature or PR. Docstrings must describe code generically, as it exists — never reference the feature, PR, or ticket that introduced them. |
| [walkthrough-pr](skills/walkthrough-pr/SKILL.md) | Walk through a PR's changes one file at a time, ordered by data flow (entry → transform → persist → output), pausing for confirmation after each file. Skips tests, lockfiles, and generated code. |

## Why

Agents working on a feature tend to write docstrings scoped to that feature: "new helper for the bulk export flow", "added as part of retry logic". The codebase outlives any single PR — these skills push docs and comments back to timeless, code-first descriptions.

## Structure

Each skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the skill instructions.

```
skills/
└── docstring-cleanup/
    └── SKILL.md
```

## License

MIT
