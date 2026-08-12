---
name: docstring-cleanup
description: 'Clean up feature-coupled docstrings and comments after working on a feature or PR. Docstrings must describe code generically, as it exists — never reference the feature, PR, ticket, or change that introduced them. Use when the user asks to clean up docstrings/comments, review a diff for comment hygiene, or after completing feature work before raising a PR. Also use proactively as a final pass on any branch with new docstrings.'
---

# Docstring Cleanup — Decouple Docs from the Change

## Why this matters

A codebase lives independently of any single feature or PR. A docstring is read months later by someone who has never heard of the feature that introduced the code. Docstrings that narrate the change ("added as part of X", "new helper for the Y flow") rot instantly: the "change" becomes ancient history, but the text stays. Docs must describe **what the code is and does now**, in terms of the code itself — not the journey that produced it.

## The core rule

> Write every docstring and comment as if the code had always existed. No reader should be able to tell, from the docs alone, which feature or PR introduced it.

## Anti-patterns to find and fix

Scan the diff (or the files touched) for these patterns:

1. **Change narration** — docstrings describing the diff, not the code:
   - ❌ `"""New helper added to support bulk export."""`
   - ❌ `# Added this check as part of the retry-logic feature`
   - ✅ `"""Serialize records into the export wire format."""`

2. **Feature/PR/ticket references** — coupling docs to project management artifacts:
   - ❌ `"""Handles pagination (see PR #482 / JIRA-1234)."""`
   - ❌ `"""Part of the new onboarding flow."""`
   - ✅ `"""Iterate over result pages, yielding one batch per request."""`
   - Exception: a ticket reference is acceptable only when it documents a *permanent external constraint* (e.g. a vendor bug workaround) that the code cannot express.

3. **Caller-specific docs on generic code** — describing a utility in terms of its first caller:
   - ❌ `"""Format a timestamp for the invoice email."""` (on a generic `format_ts()`)
   - ✅ `"""Format a timestamp as ISO-8601 with millisecond precision."""`
   - The function's contract is its signature and behavior, not who happens to call it today.

4. **Temporal language** — "new", "now", "currently", "recently", "updated to", "no longer":
   - ❌ `"""Now uses the v2 client instead of the deprecated v1."""`
   - ✅ `"""Fetch via the v2 client."""` (or delete — the code already says this)

5. **Reviewer-directed comments** — text talking to the PR reviewer, not the next reader:
   - ❌ `# Moved this here so the transaction wraps both writes`
   - ✅ `# Both writes must commit atomically` (keep only if the constraint is non-obvious)

6. **Redundant docstrings added for diff bulk** — docs that restate the name/signature:
   - ❌ `"""Get the user by id."""` on `get_user_by_id(id)`
   - Fix: delete. Absence of a docstring beats a noise docstring. Match the surrounding codebase's docstring density — don't document every helper in touched files when the rest of the codebase doesn't.

## Delegation

This is mechanical pattern-matching work — it does not need a highly intelligent model. Feel free to delegate: spawn subagents (e.g. one per changed file or group of files, on a cheaper/faster model like Haiku) to scan and propose rewrites in parallel, then review and apply their proposals. Reserve your own judgment for the borderline cases (e.g. whether a ticket reference documents a permanent external constraint).

## Procedure

1. **Scope**: get the set of changed files — `git diff main...HEAD --name-only` (or the diff/PR the user points at). Only touch docstrings/comments that the branch added or modified; leave pre-existing docs alone unless asked.
2. **Scan**: for each changed file, read the added docstrings and comments. Also grep the diff for temporal/feature markers: `git diff main...HEAD | grep -inE '^\+.*(new |now |currently|recently|added|part of|this (pr|feature|change)|as part|no longer|updated to|refactored|moved)'` — treat hits as leads, not verdicts; judge each in context.
3. **Classify** each flagged doc: rewrite (has real content buried in change-narration), delete (pure noise or restates the signature), or keep (already generic).
4. **Rewrite** in place: describe behavior, contract, invariants, and non-obvious constraints — in the present tense, feature-agnostic, matching the file's existing doc style and density.
5. **Report**: list each change as `file:line — before → after` (or "deleted") with one-line reasoning only where non-obvious.

## What good docstrings say

- The contract: inputs, outputs, side effects, error behavior.
- Invariants and constraints the signature can't express ("must be called inside a transaction").
- The *why* behind non-obvious choices — but a timeless why ("vendor API silently truncates >1MB payloads"), never a historical one ("changed because the old feature broke").

## Litmus test

Before keeping any docstring, ask: **"If this code had been written two years ago by someone else, would this docstring still make sense, verbatim?"** If no — rewrite or delete.
