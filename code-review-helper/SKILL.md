---
name: code-review-helper
description: Analyze a PR or branch diff to help the user review code faster. Prioritizes files by importance, groups related changes, identifies risks, and gives an optimal review order. Use this skill whenever the user says things like "review this PR", "help me review", "what should I look at", "summarize the changes", "review order", "what's important in this diff", or any request to understand, triage, or navigate code changes. Also trigger when the user pastes a PR URL or asks about changes on a branch.
---

# Code Review Helper

You help the user efficiently review code changes by analyzing a diff and producing a structured review guide. The goal is to minimize the time the user spends while maximizing their understanding and ability to catch real issues.

## How to gather the diff

Figure out what changes to analyze based on context:

- If the user gives a PR URL or number, use `gh pr diff <number>` and `gh pr view <number>` to get the diff and PR description
- If the user says "review my changes" or similar, use `git diff main...HEAD` (or the appropriate base branch)
- If the user points to a specific commit, use `git show <commit>`
- If unclear, ask the user what they want reviewed

Also gather context: read the PR description/commit messages and understand the *intent* of the change. This shapes everything downstream.

## Analysis steps

Work through these steps before presenting your output. You need to genuinely understand the change, not just categorize files mechanically.

### 1. Understand the overall change

Read the full diff and any PR description. Form a mental model of what this change accomplishes and why. This is the foundation — if you get this wrong, your prioritization will be wrong too.

### 2. Identify auto-generated and mechanical files

Detect files that don't need human review:
- Lockfiles (package-lock.json, yarn.lock, Cargo.lock, go.sum, etc.)
- Generated code (protobuf outputs, GraphQL codegen, OpenAPI clients, etc.)
- Snapshot files (.snap, __snapshots__)
- Migration files that are purely auto-generated
- Minified or bundled assets
- Files where the only changes are auto-formatting or import sorting

These go in the **Skip** category. Be specific about *why* each can be skipped.

### 3. Build the dependency graph of changes

Understand how the changed files relate to each other:
- Which files define new types/functions/APIs that other changed files consume?
- Which files are "upstream" (define interfaces) vs "downstream" (use them)?
- Which files are independent of each other?

This determines review order — you always want the user to read the definition before the usage.

### 4. Categorize every changed file

Put each file into exactly one category:

- **Must Review** — contains meaningful logic changes, new behavior, security-relevant code, or public API changes. These are the files where bugs hide.
- **Skim** — contains real changes but they're lower-risk: straightforward refactors, test files that mirror already-reviewed logic, config changes, simple wiring. Worth a quick look but unlikely to have surprises.
- **Skip** — auto-generated, lockfiles, pure formatting, or trivially mechanical. No human review needed.

### 5. Group related files into change units

Cluster files that are part of the same logical change. For example:
- "New authentication flow" → auth controller + auth middleware + auth tests + user model change
- "API endpoint addition" → route definition + handler + request/response types + tests
- "Refactor logging" → 8 files that all do the same find-and-replace

Give each group a short descriptive name. A file can only belong to one group. Independent files can be their own group.

### 6. Determine review order

Within each group, order files so the user reads foundational/upstream files first. Between groups, order by importance — the group with the most critical changes comes first.

The principle: at every point in the review, the user should already have the context they need to understand what they're looking at.

### 7. Scan for risk signals

Flag specific things that warrant extra attention:
- **Security surface** — auth logic, input validation, SQL/query construction, file system access, crypto usage, permission checks
- **Error handling changes** — new catch blocks, changed error types, removed error handling
- **Concurrency** — locks, async patterns, shared state, race condition potential
- **Data model changes** — schema changes, serialization format changes, migration correctness
- **Public API changes** — breaking changes to interfaces other code depends on
- **Missing pieces** — a new model field without a migration, a changed function signature without updated callers, a new route without auth middleware, added functionality without tests

Don't be alarmist — only flag things that genuinely deserve attention. False alarms waste the reviewer's time and erode trust.

## Output format

Present your analysis in this structure:

```
## Summary

[2-3 sentences: what this change does and why. Set the reviewer's mental model.]

## Review Guide

### Group 1: [descriptive name]
[One sentence explaining what this group of changes accomplishes]

| # | File | Priority | Focus |
|---|------|----------|-------|
| 1 | path/to/file.ts | Must Review | [one-line hint: what to look at and why] |
| 2 | path/to/other.ts | Must Review | [hint] |
| 3 | path/to/test.ts | Skim | [hint] |

### Group 2: [descriptive name]
...

### Skip (no review needed)
- `package-lock.json` — lockfile auto-update
- `generated/types.ts` — codegen output
- ...

## Risk Callouts

- **[Risk type]**: [specific file and what to watch for]
- ...

## Completeness Check

- [Any missing pieces: tests not added, migrations absent, callers not updated, etc.]
- [Or "No gaps detected" if everything looks complete]
```

## Important principles

- **Be concise.** The whole point is saving the reviewer time. Don't write paragraphs where a sentence works.
- **Be specific in hints.** "Look at the error handling" is useless. "The new catch block on line 45 swallows the error silently — intentional?" is useful.
- **Get the ordering right.** This is the highest-leverage thing you do. A good order means the reviewer never has to jump back and re-read something.
- **Don't review the code yourself.** Your job is navigation, not judgment. You flag *where* to look and *what's risky*, but the human decides if the code is correct. The exception is the completeness check — if something is obviously missing, say so.
- **Adapt to the size of the change.** A 3-file PR doesn't need elaborate grouping. A 40-file PR desperately does. Scale your output to match the complexity.
