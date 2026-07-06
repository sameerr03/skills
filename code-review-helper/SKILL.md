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

### 1. Build a rich change map

Explain the shape of the change before listing files. This is the fast mental
model, so make it substantial enough that the user can open the diff already
knowing how the pieces fit.

Include the useful parts of:

- **What changed**: old path -> new path, behavior before/after, and user- or
  system-visible intent.
- **Schema and data shape changes**: database migrations, table/index changes,
  validation schema changes, API request/response shape changes, serialized
  payloads, TypeScript/Python types, Convex schemas, GraphQL/OpenAPI contracts,
  or any other change to the shape of data.
- **Contract and code-shape summaries**: compact input/output summaries for new
  or changed functions, API endpoints, service boundaries, public types, or
  important internal helpers.
- **Where it lives**: subsystem(s), entry points, core logic, data/storage, UI,
  background jobs, external services, and downstream consumers.
- **Concept inventory**: new types, APIs, routes, jobs, tables, migrations,
  components, state transitions, or shared abstractions.
- **Reviewer mental model**: entry point, core contract/invariant, downstream
  effects, test contract, and where bugs are most likely to hide.

### 2. Identify review units

Cluster changed files into logical units of behavior. Name groups for the actual
shape of the PR, not for a fixed template. A review unit might be:

- New feature flow: entry point, business logic, data model, tests
- API or contract change: route, schema/types, callers, tests
- Refactor: moved abstraction, updated consumers, compatibility layer
- Data/storage change: migration, model, ingestion path, backfill, tests
- UI change: state owner, view components, interaction handlers, tests

Every changed file should belong to exactly one unit unless it is in Skip. Test
files belong inside the group whose behavior they validate; do not place tests in
a separate global section.

### 3. Order the review units and files

The Review Guide is both the grouping and the route. Do not create a separate
"Fastest Review Route" table and then repeat the same files in "Review Units".

Order groups and files for comprehension, using a call-stack-aware flow rather
than a rigid list of group names:

- Put significant schema, type, data-shape, or contract changes first because
  they define the shape of everything else.
- Separately identify the reviewer entry point: the UI action, API caller, route,
  command, job, event, or other place where the reviewer can understand the flow
  in context. This may not be the first group if schemas/contracts come first.
- After the entry point, follow the call stack deeper through boundaries, call
  sites, helpers, storage, and external integrations, then back up through
  consumers, rendering, wiring, or cleanup as needed.
- Use semantic PR-specific group names, such as `Principle Score Response
  Contract`, `Portfolio Page Refresh Flow`, `Signed Scoring Service Boundary`,
  `Cache Write And Expiry Logic`, or `Reason Rendering View Model`.
- Put definitions before usage and invariants before edge cases.
- Put the highest-risk or most central behavior before peripheral wiring.
- Put generated or mechanical files in Skip instead of the critical path.

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

### 5. Summarize tests inside their review unit

For test files, summarize what tests exist and what behavior each test case
documents. The goal is to help the user understand every relevant test case
without manually reading the test file.

Rules:

- Put tests under `Tests in this unit` inside the review group they validate.
- Read the relevant full test file or surrounding test group, not only the diff
  hunk.
- Include unchanged tests in the same relevant file, `describe` block, class, or
  function group when they define the behavior under review.
- Enumerate every meaningful test case in plain English.
- Do not collapse many cases into a theme summary like "auth gating and cleanup".
- Do not copy exact `describe`/`it`/`test_*` strings unless exact wording matters.
- For Vitest/Jest-style tests, group related cases by their surrounding
  `describe` concept when useful, then summarize each `it`/`test` case in
  English.
- For pytest/unittest-style tests, list each relevant test function/method as an
  English behavior summary.
- Mention fixtures, mocks, setup, or helpers only when they materially shape the
  behavior being tested.

Keep missing test cases out of the test summary. If an absent test is concrete
and important, mention it later in Risk And Invariant Callouts or Completeness
Check.

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

### 7. Use compact code-shape summaries when they help

Prefer structured Markdown over heavy artifacts. Do not use Mermaid diagrams by
default. For comprehension, prefer compact before/after shapes and input/output
summaries:

- Function signatures and data contracts
- Endpoint request/response shapes
- Schema before/after snippets
- Old behavior -> new behavior summaries
- Small edited-code snippets only when the exact code shape matters

Do not create HTML or other output files by default. Offer or create a richer visual
artifact only when the user asks for one or the PR is large enough that a separate
review artifact clearly saves effort.

## Output format

Adapt the exact shape to diff size, but default to:

```markdown
## Change Map

### What Changed

[Large-scale conceptual changes, old path -> new path behavior, subsystem
touched, and intent.]

### Schema / Data Shape Changes

```text
[Contract or schema name]
Before:
- field: type

After:
- field: type
- newField?: type
```

Include DB migrations, table/index changes, validation schemas, API payloads,
serialized formats, TypeScript/Python types, Convex schemas, GraphQL/OpenAPI
contracts, or other data-shape changes when present.

### Contract / Code Shape Changes

```text
[Function, endpoint, service, or type]
Input:
- ...

Output:
- ...

Behavior:
- old behavior -> new behavior
```

Use compact summaries, not long code dumps. Show only the function signatures,
endpoint shapes, edited-code structure, or before/after snippets that help the
reviewer understand the diff quickly.

### Reviewer Mental Model

- Reviewer entry point:
- Core contract/invariant:
- Downstream effects:
- Tests encode:
- Likeliest places bugs hide:

## Review Guide

### 1. [Unit name]

[What this unit does and why to read it here.]

| Order | File | Attention | Focus |
|---|---|---|---|
| 1 | `path/to/types.ts` | Must Review | Defines the contract consumed below. |
| 2 | `path/to/handler.ts` | Must Review | Main behavior and failure modes. |

Tests in this unit:

`path/to/handler.test.ts`

Input validation:
- Rejects requests with a missing required field.
- Accepts a valid request and passes normalized input to the handler.

Error handling:
- Converts service failures into the expected error response.
- Preserves the existing cache when refresh fails.

## Risk And Invariant Callouts

- **[Risk/invariant]**: [Specific file/flow and what to verify.]

## Skim / Skip

- `path/to/config` — Skim: [why]
- `generated/file.ts` — Skip: [why]

## Completeness Check

- [Missing tests, migrations, caller updates, docs, or "No obvious gaps detected."]
```

For very small diffs, collapse sections while preserving the same substance. For
very large diffs, lead with the map, then group aggressively so the user can
review one coherent unit at a time.

## Style principles

- Be concise, but not shallow. Optimize for faster understanding.
- Prefer concrete file-specific guidance over generic advice.
- Explain tests as an explicit per-case behavior inventory inside the review
  group they validate.
- Keep review order and grouping in one Review Guide.
- Use flexible, PR-specific group names; the call-stack flow is a guide, not a
  fixed section template.
- Do not replace the human reviewer, but do surface obvious suspected gaps that
  affect review attention.
- Keep generated, lockfile, and mechanical churn out of the user's critical path.
