---
name: walkthrough-pr
description: Walk a user through a PR's changes one file at a time, pausing for confirmation after each. Use when the user wants to review, understand, or be walked through a pull request or diff file-by-file, or says "walk me through this PR".
---

# walkthrough-pr

Guide the user through a PR's changes **one file per step**. Never dump the whole diff. Each step covers a single file, in 2–3 lines, then **stop and wait** for the user to confirm they reviewed it before moving on.

## Procedure

1. **Resolve the diff.** Determine the change set:
   - If the user named a PR number, fetch it: `gh pr diff <number> --name-only` for the file list, `gh pr diff <number>` for content.
   - Otherwise use the local branch vs base: `git diff --name-only <base>...HEAD`. Detect the base instead of guessing: `git symbolic-ref refs/remotes/origin/HEAD` or `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`; ask the user only if both fail.
   - Detect renames/moves (`git diff -M --name-status`) so a moved file isn't walked as a full delete + add. A pure move gets one cheap step: `moved X → Y, logic unchanged`, plus a line on any edits made in transit.
   - **Exclude test files** from the walkthrough list. Drop paths that are clearly tests (e.g. `*.test.*`, `*.spec.*`, `__tests__/`, `test/`, `tests/`, `*_test.go`, `*_test.py`, snapshots/fixtures used only by tests). Mention in the opening line how many test files were skipped if any. If the user asks to include tests, walk them separately.
   - **Exclude generated and vendored files.** Lockfiles (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Cargo.lock`, `*.lock`), build output (`dist/`, `build/`), `*.generated.*`, `*.min.*`, vendored deps, snapshots. Never walk a user through a lockfile diff — note it in one line ("lockfile updated for the new dep") and move on.
   - **Huge PRs (~30+ production files):** don't walk every file. List the independent sub-flows with file counts and ask which to walk (default: riskiest / most central first). Within a chosen flow, proceed file-by-file as normal.

2. **Order by data flow, not by file.** Before presenting anything, read enough of the diff to understand *what the change actually does end-to-end*, then sequence the files to follow that flow — the path a request/value takes through the system. Never present in arbitrary, alphabetical, or `git`-listed order.
   - Find the **entry point** of the change (the trigger: a route/handler, CLI command, event, user action, schema/migration that everything else builds on).
   - Follow the call chain / data movement outward from there: entry → validation/parsing → core logic/transform → persistence or side effects → response/output → consumers of that output. (Do not include test files in this sequence.)
   - When two files are peers in the flow, present the producer before the consumer (the thing that creates a value before the thing that reads it).
   - If the PR has independent sub-flows, walk one complete flow to its end before starting the next; announce the switch.
   - State this ordering rationale in one line up front so the user knows the spine they're following.

3. **Announce the plan, briefly.** One line naming the flow and file count (production files only). Example: `7 files (2 test files skipped). Following the request flow: route → validator → service → repo → response. Starting 1/7.` Do not summarize the files yet.

4. **Per-file step.** For the current file output ONLY:
   - Header line: `[N/total] path/to/file.ts`
   - 2–3 lines max on what changed and *why* it matters (the working, not a line-by-line dump). Name the key function/symbol and the behavior change.
   - Anchor it in the flow: state where this file sits in the chain and how data arrives here from the previous step and leaves toward the next (`receives the parsed body from the validator, writes it via repo.save()`). Each step should connect to the one before it, not stand alone.
   - If the file is large or non-obvious, point to the specific hunk worth reading (`see the change to handleAuth() around the token check`).
   - End with: `Reviewed? Reply to continue, or ask about this file.`
   - **Then stop.** Do not output the next file.

5. **Wait for confirmation.** Only advance when the user signals they're done (e.g. "next", "ok", "got it", "continue"). If they ask a question, answer it scoped to the current file, then re-prompt `Reviewed?` — do not advance on a question alone.

6. **Repeat** in flow order until all files done.

7. **Closing.** After the last file, give a 3–5 line synthesis that retraces the full flow end-to-end: how a value enters, moves through each file you covered, and exits. Call out cross-file interactions and anything risky to double-check. Offer next actions (approve, request changes, run tests).

## Rules

- **Skip test files.** Walk production/source changes only; omit test paths unless the user explicitly asks to include them.
- **Skip generated files.** Lockfiles, build output, minified/generated code get a one-line mention at most, never a step.
- **Order = data flow, always.** Sequence files along the path data takes through the system (entry → transform → persist → output → consumers). Never alphabetical, never arbitrary. Each step links to the previous one.
- **One file per message. No walls of text.** If tempted to cover two files, split into two steps.
- Keep each file's explanation to 2–3 lines. Detail comes on request, not by default.
- Show the actual diff hunk only if the user asks or the change is subtle; otherwise describe.
- Track progress with the `[N/total]` counter every step so the user knows how far in they are.
- Never advance past a file the user hasn't confirmed.
- Don't editorialize on style unless it affects correctness; this is comprehension, not review nitpicking.
