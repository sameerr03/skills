---
name: guided-pr-review
description: Explore a pull request conversationally, one file or coherent file section at a time, so the user can review code without manually chasing callers, parameter origins, types, nested helpers, downstream effects, and tests across the repository. Use when the user wants a second pair of eyes, a risk-ordered review route, function-level context and test cases presented together, exact high-value lines to inspect, or tailored manual product checks for behavior that automated tests cannot establish. Especially use for large or stacked PRs and code involving databases, schemas, financial math, auth, concurrency, integrations, or performance.
---

# Guided PR Review

Act as a just-in-time code navigator and second pair of eyes. Fill in the
surrounding knowledge the user would otherwise gather by jumping among callers,
types, helpers, tests, and consumers. Keep the real code visible and let the
user control the questions, judgment, pacing, and GitHub review actions.

## Operating assumptions

- Expect a fresh chat in the same repository worktree and branch as the PR.
- Read `AGENTS.md` and apply its repository-specific risk guidance.
- Discover the current branch's PR and verify its actual base and head. For a
  stacked PR, compare it with its parent rather than the default branch.
- Remain read-only unless the user requests a concrete change. Never merge.
- The user decides when a file is understood and clicks GitHub's viewed button.
  Do not spend tokens managing review status, ledgers, or process bookkeeping.

## Optimize for reduced context reconstruction

- Use conversation as the interface and code as evidence.
- Reconstruct the smallest useful dependency slice around the current target.
  Do not explain the entire repository or every helper implementation.
- Pair production behavior with its tests. Do not make the user review tests as
  a separate body of code.
- Let strong tests establish straightforward capability. Direct detailed code
  reading toward the branches, queries, calculations, side effects, or failure
  paths where implementation details can still violate the contract.
- Distinguish facts, suspected defects, and design questions. The user will ask
  follow-ups; do not manufacture a ceremony of mandatory questions.

## Establish the review route

Before starting individual files:

1. Read the PR intent, linked issue or Atlas leaf when present, changed files,
   commit history, checks, and complete diff.
2. Build a compact mental model of the entry points, data flow, authoritative
   state, public contracts, side effects, and main failure modes.
3. Order files by conceptual dependency and review value. Put schema, types,
   APIs, database behavior, financial logic, auth, and state transitions early.
   Keep each test file beside the production behavior it verifies.
4. Split a very large file into coherent targets such as validation, query
   construction, core algorithm, persistence, error handling, or rendering.
5. Present the route briefly, identify the highest-value manual-review targets,
   and begin the first target after the user accepts or adjusts the order.

## Build a context packet for each target

At the start of a file or file section, inspect the full relevant code, its base
version, callers, callees, types, helpers, consumers, and relevant test groups.
Then provide a concise context packet containing the useful parts below.

### Role and contract

- Explain what the target does in the larger flow and old behavior -> new
  behavior.
- Enumerate the changed functions, components, queries, mutations, or types and
  summarize their inputs, outputs, side effects, and failure behavior.

### Upstream context

- Show where important parameters and state originate.
- Resolve their concrete types, validation, defaults, normalization, and whether
  values are trusted, user-controlled, derived, cached, or authoritative.
- Summarize the relevant caller chain so the user does not need to chase it.

### Internal shape

- Collapse nested helpers into a compact staged explanation of the larger
  function: for example validate -> load -> transform -> persist -> return.
- Explain a helper body only when it contains an important assumption, branch,
  side effect, performance cost, or suspected defect.

### Downstream effects

- Explain who consumes the return value and what state, database records,
  external services, UI, caches, jobs, or later calculations can change.
- Call out cross-file assumptions that must agree for the flow to be correct.

### Tested behavior

For each changed function or observable behavior:

- list every relevant test case in plain English beside that function;
- state what the assertion or oracle actually proves;
- distinguish tests that merely exist from tests or CI checks known to have run
  on the reviewed head;
- explain material mocks, fixtures, shared helpers, or weak oracles;
- identify meaningful behavior with no automated coverage.

Tests can often provide the clearest statement of a function's capability and
save exhaustive body reading. They are not sufficient when the oracle is weak,
the mock omits reality, or correctness depends on query shape, authorization,
financial calculation, concurrency, performance, or external side effects.

### Highest-value code to inspect

- Point to exact lines or the smallest useful ranges.
- Explain why each location deserves human judgment and what failure would look
  like.
- Prefer a few consequential locations over asking the user to understand every
  line equally.

### Second-pair-of-eyes findings

- Surface concrete suspicious behavior, missing cases, inconsistent contracts,
  or surprising assumptions found while reconstructing the context.
- Label uncertainty and evidence. Do not invent findings to appear thorough.

Walk through a large packet in conversational pieces rather than dumping it all
at once. Stay on the target while the user asks questions or requests changes;
move on when the user chooses.

## Manual product verification

When automated tests cannot establish integrated or visual behavior, give the
user a tailored product walkthrough rather than a generic instruction to test
the UI. Respect `AGENTS.md` rules about who starts the development server.

- State the setup, exact action, expected result, and failure signal for each
  important scenario.
- Derive edge cases from the actual change: relevant auth/role states,
  empty/populated data, loading/error/retry/reconnect, navigation and refresh,
  repeated actions, desktop/mobile, and blank, malformed, unusual, long, stale,
  or duplicate input.
- Apply repository context. For example, consider relevant signed-out/signed-in,
  region, broker, and partial-import states in Finbelief; consider relevant
  unusual messages, partial streams, cancellation, tool failure, retry,
  compaction, archive/unread, and anonymous/authenticated states in Fermion.
- Suggest the highest-value scenarios in manageable batches and let the user
  perform them. Never claim a manual check that was not actually performed.

No model can enumerate every possible UI path. Seek strong coverage from the
changed state space, input boundaries, and failure model, and identify remaining
uncertainty without turning the review into process administration.

## Mandatory close implementation inspection

Do not let tests replace direct inspection of consequential:

- database schemas, migrations, indexes, query cardinality, mutations, and
  backfills;
- financial formulas, time-series behavior, percentiles, prices, order
  execution, and risk controls;
- auth, authorization, tenancy, secrets, and trust boundaries;
- concurrency, retries, cancellation, streaming, and stale writes;
- external integrations, irreversible effects, and operational recovery;
- N+1 access, whole-table scans, unbounded work, caching, and request-path
  performance.

For these surfaces, explain the invariant and test coverage, then point the user
to the exact implementation details that determine correctness.

## Changes and finishing

If the user requests a change, make the smallest in-scope edit, run focused
validation, and return to the affected target with updated context. Do not add a
separate review-management workflow.

When the user reaches the end, provide only a concise synthesis if useful:
confirmed concerns, unresolved cross-file assumptions, verification gaps, and
the remaining highest-value manual checks. The purpose is not to certify or
administer the PR; it is to maximize the user's understanding and consequential
risk coverage per minute.
