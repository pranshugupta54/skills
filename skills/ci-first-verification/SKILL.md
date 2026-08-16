---
name: ci-first-verification
description: This machine runs many coding agents concurrently — local compute is a shared, scarce resource. Prefer reading CI results on the GitHub PR over running builds, test suites, typechecks, or lint locally. Use before running any heavy local process (build, test, lint, typecheck, install), when verifying whether changes pass, or whenever working alongside other agents on the same machine.
---

# CI-First Verification — Don't Kill the Laptop

## The situation

You are one of several agents working concurrently on this machine. Every full build, test suite, typecheck, or repo-wide lint you launch competes for the same CPU, memory, and disk as every other agent's work. A handful of agents each "just running the tests" brings the whole machine to its knees — and most of those runs duplicate verification that CI already does for free on the PR.

## The core rule

> Before launching any heavy local process, ask: **is CI already answering this question?** If the branch has a PR, the answer is usually yes. Read the CI result instead of recomputing it locally.

## Check CI instead

```bash
gh pr checks <number>            # pass/fail summary of all checks on the PR
gh pr checks <number> --watch    # only when you must block on a result
gh run list --branch <branch>    # workflow runs for the branch
gh run view <run-id> --log-failed  # read ONLY the failing steps' logs
```

Typical loop: push → `gh pr checks` → read failures from logs → fix → push → let CI re-verify. No local build ever ran.

## No PR yet?

If the branch has no PR (`gh pr view` fails), don't silently fall back to heavy local runs — **ask the user** which they prefer:

1. **Create a PR now** (draft is fine: `gh pr create --draft`) so CI takes over verification, or
2. **Run the checks locally** this once, scoped as narrowly as possible.

Default recommendation to the user: the draft PR — it costs nothing, CI does the work, and every later verification in the session is free. Only skip the question and go local when the repo has no CI configured at all (no `.github/workflows/`), or the user already told you which way to go.

## Heavy processes — avoid by default

- Full project build (`npm run build`, `cargo build`, `tsc` over the project, bundlers)
- Whole test suite (`npm test`, `pytest`, `go test ./...`)
- Repo-wide lint or typecheck
- Dev servers / watch mode left running after you're done
- Dependency installs when node_modules already exists and the lockfile didn't change

Run one of these locally only when ALL of: (1) no CI covers the question, (2) the answer blocks your next step, (3) you scope it as narrowly as possible. Say in your response that you ran it and why.

## Cheap local alternatives (fine to use)

- **Single test file**, not the suite: `npx jest path/to/one.test.ts`, `pytest tests/test_one.py -k case`
- **Targeted lint**: `npx eslint <changed files>` — never `.`
- **Syntax check one file**: `node --check`, `python -m py_compile`, editor/LSP diagnostics
- **Reading code** — cheapest verification there is; often you can confirm correctness by tracing the change instead of executing anything

## Multi-agent etiquette

- Never run heavy processes "proactively" or "to be safe" — only when the result is needed and CI can't provide it.
- One process at a time: don't parallelize builds with tests locally.
- Kill what you start: no orphaned watchers, servers, or daemons when your task ends.
- If another agent's build is likely running (system feels loaded), prefer CI even harder.

## Litmus test

Before any command that will spin fans: **"Will the PR's checks tell me this within a few minutes anyway?"** If yes, read the checks. Push first, verify remotely.
