# skills

[![skills.sh](https://skills.sh/b/pranshugupta54/skills)](https://skills.sh/pranshugupta54/skills)

Agent skills for coding agents — Claude Code, Cursor, Codex, and anything else that reads a `SKILL.md`.

The theme: **keeping codebases clean when agents do the writing.** Agents ship working code, but they leave fingerprints — docstrings written for the PR instead of the codebase, walls of diff nobody can review, comments talking to a reviewer who's long gone. These skills are the cleanup and comprehension passes that remove those fingerprints.

Browse them with full docs at [p54.dev/skills](https://p54.dev/skills).

## Install

All skills:

```bash
npx skills add pranshugupta54/skills
```

One skill:

```bash
npx skills add pranshugupta54/skills --skill docstring-cleanup
```

## Reference

- **[docstring-cleanup](./skills/docstring-cleanup/SKILL.md)** — Remove feature-coupled docstrings and comments after a feature or PR. Docs should describe code as if it had always existed — never the feature, ticket, or diff that introduced it. Includes an anti-pattern catalog (change narration, temporal language, reviewer-directed comments) and a scan-classify-rewrite procedure that runs fine on cheap models.
- **[walkthrough-pr](./skills/walkthrough-pr/SKILL.md)** — Walk through a PR one file at a time, ordered by data flow (entry → transform → persist → output) instead of alphabetically, pausing for confirmation after each file. Skips tests, lockfiles, and generated code; handles renames and huge PRs gracefully.

## Why

A codebase outlives every feature that passes through it. Most agent output is written from inside one feature's point of view — these skills push the result back to something a stranger can read two years later without knowing which PR it came from.

They also share a design principle: the work is mechanical pattern-matching, so each skill explicitly tells the agent it can delegate to cheaper/faster subagents and reserve judgment for the borderline calls.

## Structure

Each skill is a directory under `skills/` with a `SKILL.md` — YAML frontmatter (`name`, `description`) followed by the instructions. That's the whole format; no build step, no dependencies.

```
skills/
├── docstring-cleanup/
│   └── SKILL.md
└── walkthrough-pr/
    └── SKILL.md
```

## License

[MIT](./LICENSE)
