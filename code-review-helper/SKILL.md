---
name: code-review-helper
description: Analyze a PR or branch diff to help the user understand and review code changes faster. Prioritizes files by importance, explains the mental model of the change, groups related changes, summarizes test coverage, identifies risk signals and missing cases, and gives an optimal review route. Use whenever the user says things like "review this PR", "help me review", "what should I look at", "summarize the changes", "review order", "what's important in this diff", or any request to understand, triage, navigate, or prepare to review code changes. Also trigger when the user pastes a PR URL or asks about changes on a branch.
---

# Code Review Helper

Help the user understand code changes as fast as possible so they can review the
whole diff with a strong mental model. Your job is review navigation, context
building, and risk surfacing: explain what changed, how the pieces relate, what
the tests prove, and where the human reviewer should focus.

## First rule: do not ask what to review when a branch can answer it

When invoked from a repository branch, assume the user wants the current branch's
associated PR or branch diff reviewed. Do the discovery yourself.

Diff source precedence:

1. If the user gives a PR URL or number, use that PR.
2. Otherwise, try the current branch's PR with `gh pr view --json number,title,body,baseRefName,headRefName,url`.
3. If a PR is found, use its base branch and gather `gh pr view` metadata plus `gh pr diff`.
4. If no PR is found, infer the base branch from the repo default, upstream tracking branch, or common bases such as `main`/`master`, then use `git diff <base>...HEAD`.
5. If the user points to a commit, use `git show <commit>`.

Ask a blocking question only when discovery fails and there is no meaningful local
diff, or when multiple plausible PRs/diffs exist and choosing one would be risky.
Do not stop just to ask whether you should review, which branch to review, or what
PR number to use if the current branch already makes that discoverable.

Always gather intent before analysis: PR title/body, commit messages, changed file
list, and enough surrounding code to understand the existing subsystem.

## Analysis workflow

Work through these steps before presenting the guide. Scale depth to diff size:
a tiny PR should get a compact route; a large PR needs stronger grouping and a
clear map.

### 1. Build the change map

Explain the shape of the change before listing files:

- What behavior, architecture, or workflow changed?
- Which subsystem(s), entry points, and data flows are touched?
- What new concepts, types, APIs, routes, jobs, migrations, or state transitions appear?
- Which files are foundational definitions versus downstream usage or wiring?
- What should the reviewer understand before opening the diff?

This is the fast mental model. Keep it short, but make it concrete.

### 2. Identify review units

Cluster changed files into logical units of behavior. A review unit might be:

- New feature flow: entry point, business logic, data model, tests
- API or contract change: route, schema/types, callers, tests
- Refactor: moved abstraction, updated consumers, compatibility layer
- Data/storage change: migration, model, ingestion path, backfill, tests
- UI change: state owner, view components, interaction handlers, tests

Every changed file should belong to exactly one unit unless it is in Skip.

### 3. Determine the fastest review route

Give the user an ordered path through the diff. Optimize for comprehension:

1. PR intent and contract/schema/type changes
2. Main entry points
3. Core logic and state transitions
4. Downstream callers/consumers
5. Tests that define intended behavior
6. Wiring/config/docs
7. Generated or mechanical files to skip

Within each unit, put definitions before usage and invariants before edge cases.
Between units, put the highest-risk or most central behavior first.

### 4. Categorize files by attention level

Use exactly one category for each changed file:

- **Must Review**: meaningful logic, public contract, security/trust boundary,
  data model, concurrency, state transitions, migrations, or behavior-defining tests.
- **Review After Context**: real code worth reading after the core path is understood,
  such as downstream consumers, UI wiring, straightforward adapters, or mirrored tests.
- **Skim**: low-risk config, docs, simple renames, mechanical but not generated changes.
- **Skip**: generated code, lockfiles, minified/bundled assets, pure formatting/import
  sorting, snapshots that only reflect already-reviewed behavior, or codegen outputs.

Be specific about why Skim/Skip files are lower value. Do not hide meaningful test
or migration changes in Skip.

### 5. Summarize tests as coverage, not file trivia

For test files, summarize what behavior is covered and what appears absent. The
goal is to help the user spot missing cases quickly.

For each meaningful test unit, identify:

- Covered happy paths
- Covered edge/error paths
- Contract/invariant being asserted
- Fixtures or mocks that shape the behavior
- Missing or suspiciously thin coverage
- Tests that define product behavior and deserve careful reading
- Tests that mostly mirror implementation and can be skimmed after the core logic

If there are no test changes, say whether that seems reasonable for the diff and
what kinds of tests would normally be expected.

### 6. Surface concrete risks and invariants

Flag specific review concerns, not generic categories. Useful signals include:

- Trust boundaries: auth, permissions, user-controlled input, secrets, external APIs
- Data correctness: schema changes, migrations, serialization, backfills, idempotency
- State transitions: lifecycle changes, retries, cancellation, partial failure
- Error handling: swallowed errors, changed fallback behavior, logging-only failures
- Concurrency: async ordering, shared state, locking, race windows
- Public contracts: API shape, type compatibility, caller expectations
- Missing pieces: caller not updated, migration absent, route lacks auth, tests absent

Phrase risks as things to verify during review. Do not be alarmist, and do not
invent issues unsupported by the diff.

### 7. Use lightweight visual structure when it helps

Prefer structured Markdown over heavy artifacts. Use a small Mermaid diagram only
when it clarifies a non-trivial flow, dependency graph, or state transition.

Do not create HTML or other output files by default. Offer or create a richer visual
artifact only when the user asks for one or the PR is large enough that a separate
review artifact clearly saves effort.

## Output format

Adapt the exact shape to diff size, but default to:

```markdown
## Change Map

[2-4 sentences explaining what changed, where it lives, and how to think about it.]

## Fastest Review Route

| Order | File / Unit | Attention | Why read it here |
|---|---|---|---|
| 1 | `path/to/types.ts` | Must Review | Defines the new contract used everywhere else. |
| 2 | `path/to/handler.ts` | Must Review | Main behavior and error paths. |

## Review Units

### [Unit name]

[One sentence explaining this unit's role.]

| File | Attention | Focus |
|---|---|---|
| `path/to/file.ts` | Must Review | [Specific thing to understand or verify.] |

## Test Coverage Map

### [Test unit or file]

- Covered: [specific behaviors]
- Missing/Thin: [specific gaps, or "No obvious gaps"]
- Review focus: [what the tests establish or fail to establish]

## Risk And Invariant Callouts

- **[Risk/invariant]**: [Specific file/flow and what to verify.]

## Skim / Skip

- `path/to/config` — Skim: [why]
- `generated/file.ts` — Skip: [why]

## Completeness Check

- [Missing tests, migrations, caller updates, docs, or "No obvious gaps detected."]
```

For very small diffs, collapse sections while preserving the same substance. For
very large diffs, lead with the map and route, then group aggressively so the user
can review one coherent unit at a time.

## Style principles

- Be concise, but not shallow. Optimize for faster understanding.
- Prefer concrete file-specific guidance over generic advice.
- Explain tests enough to expose missing coverage.
- Separate "read first" from "highest risk" when those differ.
- Do not replace the human reviewer, but do surface obvious suspected gaps that
  affect review attention.
- Keep generated, lockfile, and mechanical churn out of the user's critical path.
