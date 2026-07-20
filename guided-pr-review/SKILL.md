---
name: guided-pr-review
description: >
  Teach and review a pull request in either full guided mode or quick mode. Full
  mode provides background, intuition, a logically ordered file-by-file review,
  and an understanding quiz. Quick mode provides one high-level,
  explain-diff-style review response followed by a shorter quiz. Use when the
  user wants to
  understand a PR without chasing callers, types, nested helpers, downstream
  effects, and tests; wants exact high-value lines to inspect; asks for a quick,
  fast, miniature, or high-level guided review; or needs tailored manual product
  checks. Especially use for large or stacked PRs and changes involving
  databases, schemas, financial math, auth, concurrency, integrations, or
  performance.
---

# Guided PR Review

Teach the relevant system, build intuition for the change, and then act as a
just-in-time code navigator and second pair of eyes. Fill in the knowledge the
user would otherwise gather by jumping among callers, types, helpers, tests,
and consumers. Keep the real code visible and let the user control the
questions, judgment, pacing, and GitHub review actions.

## Operating assumptions

- Expect a fresh chat in the same repository worktree and branch as the PR.
- Read `AGENTS.md` and apply its repository-specific risk guidance.
- Discover the current branch's PR and verify its actual base and head. For a
  stacked PR, compare it with its parent rather than the default branch.
- Remain read-only unless the user requests a concrete change. Never merge.
- The user decides when a file is understood and clicks GitHub's viewed button.
  Do not spend tokens managing review status, ledgers, or process bookkeeping.

## Choose the review mode

Use **quick mode** when the user explicitly asks for `quick`, `fast`,
`miniature`, `high-level`, or one-response review. Otherwise default to **full
mode**. Treat the mode as a per-PR choice rather than a permanent repository
classification: a side project can contain a consequential migration, while a
production repository can contain a mechanical change.

Both modes require inspecting the complete PR, its checks, and enough
surrounding code to verify the explanation. Quick mode compresses what is
presented to the user; it does not permit shallow agent-side inspection.

Before using quick mode, apply a risk gate. Look for the consequential surfaces
listed under Mandatory close implementation inspection. If they are present,
identify them prominently, require their exact implementation under `Read this
code`, and recommend full mode when a one-response overview would not provide
enough scrutiny. Do not call a PR ready merely because automated reviewers
approved it.

## Full mode: use four review phases

Run the review as **Background -> Intuition -> Code -> Quiz**. Background and
intuition prepare the user for the code rather than becoming detached reports.
The code phase is the complete conversational review. Start the quiz only after
every changed file has been covered and the user is ready for the understanding
gate.

### Follow this turn-by-turn protocol

1. Inspect the complete PR and surrounding system before responding.
2. In the first substantive review response, provide **Background**,
   **Intuition**, and only the **Code review route**. The route must list the
   logically grouped files or coherent file segments in review order, pair tests
   with the behavior they verify, and name the first target.
3. Stop after the route and ask whether to begin the first named target. Do not
   include its context packet, start reviewing its code, or show the quiz in
   that response.
4. After the user agrees, review exactly one current file or coherent file
   segment conversationally. Stay on it while the user reads, asks questions,
   requests changes, or performs verification. Do not pre-generate context
   packets for later targets.
5. Move to the next target only when the user chooses. Repeat until every file
   and segment in the route has been reviewed.
6. Start **Quiz** only after the final target is complete and the user is ready
   to leave the code-review phase.

## Quick mode: one response -> Quiz

Quick mode is a compact knowledge-transfer and assurance workflow, not a
miniature file-by-file review.

1. Inspect the complete PR and surrounding system before responding.
2. Produce one integrated review response with these sections in order:
   - **Risk gate**: state whether quick mode is appropriate and identify any
     surface requiring close implementation inspection.
   - **Background**: explain only the prior system needed for this change.
   - **Intuition**: show the goal, old behavior -> new behavior, central
     invariants, and a concrete example when useful.
   - **High-level code walkthrough**: group the implementation by execution,
     dependency, or data flow rather than file order. Explain how the important
     pieces realize the new behavior without dumping the diff.
   - **Verification and findings**: summarize test behavior, checks known to
     have run on the reviewed head, adversarial-review evidence if available,
     weak or missing oracles, and concrete second-pair-of-eyes findings.
   - **Read this code**: assign zero to three exact, consequential snippets using
     the reading-assignment rules below. Say explicitly when no manual code
     reading is recommended.
   - **Manual checks**: give only the integrated or product checks that remain
     valuable after automation.
3. Keep this overview in the conversation. Do not generate HTML merely to make
   quick mode resemble an explain-diff artifact; create a visual artifact only
   when interaction materially improves understanding.
4. Stop after the overview and ask whether to begin the quiz.
5. Ask exactly three interactive questions, one at a time, using the quiz
   quality rules below. Test the main flow, the most consequential invariant or
   trade-off, and what remains unproved or where debugging should begin.
6. After the quiz, give a compact synthesis and one evidence-based merge
   posture: automated evidence sufficient, manual verification still required,
   or escalate to full review. Leave the merge decision to the user.

## Optimize for reduced context reconstruction

- Use conversation as the interface and code as evidence.
- Reconstruct the smallest useful dependency slice around the current target.
  Do not explain the entire repository or every helper implementation.
- Pair production behavior with its tests. Do not make the user review tests as
  a separate body of code.
- Let strong tests establish straightforward capability. Direct detailed code
  reading toward the branches, queries, calculations, side effects, or failure
  paths where implementation details can still violate the contract.
- Distinguish links used for explanation or navigation from explicit human
  reading assignments. Do not make the user infer which linked code they are
  actually expected to inspect.
- Distinguish facts, suspected defects, and design questions. The user will ask
  follow-ups; do not manufacture a ceremony of mandatory questions.

## 1. Background

Before explaining the change, inspect the PR intent, linked issue or Atlas leaf,
changed files, commit history, checks, complete diff, relevant existing code,
and `AGENTS.md`. Explain the existing system relevant to the change and broadly
explore the surrounding code rather than inferring it from the diff alone.

Do not assume how much the user already knows. Start with deep,
beginner-friendly background and clearly note that it can be skipped when the
user is already familiar with it. Then narrow to the background directly
required for this PR:

- the prior behavior and why it existed;
- the relevant entry points, data flow, authoritative state, schemas, public
  contracts, side effects, and failure modes;
- the concepts or repository conventions the user would otherwise have to
  rediscover while reading the diff.

Do not wander into unrelated architecture. Highlight important schema, type,
endpoint, mutation, and data-shape changes early because they frame the rest of
the review. In quick mode, compress this into the smallest useful mental model
inside the one-response overview.

## 2. Intuition

Explain the core intuition for the code change. Focus on the essence, not the
full details. Use concrete examples with toy data. Use figures and diagrams
liberally.

- state the user-visible or system-level goal;
- contrast old behavior -> new behavior;
- explain the central invariants, trade-offs, and surprising consequences;
- use flows, tables, before/after comparisons, and state walkthroughs to make
  important transformations easier to predict.

When interaction would materially improve understanding, create an ephemeral,
self-contained HTML artifact outside the repository. Use it for behavior such
as financial calculations, migrations, reconciliation, query scaling, or
concurrent state transitions that benefits from manipulating toy inputs or
stepping through time. Ground the artifact in inspected code, label
simplifications and assumptions, keep production data and secrets out of it,
and verify that it works before sharing the path. Do not build an artifact for
straightforward behavior or let it delay the actual review.

## 3. Code

### Quick-mode high-level walkthrough

In quick mode, do not establish a file-by-file route or wait for approval
between targets. Cover the complete implementation internally, then explain the
change once in conceptual groups ordered by execution, dependency, or data
flow. Include precise links for supporting evidence and reserve `Read this
code` for the few locations that actually require human judgment.

### Full-mode logical review route

Cover every changed file, but do not use alphabetical file order. Group and
order production code, supporting types or schemas, and its tests by conceptual
dependency, execution flow, and review value. Put schema, types, APIs, database
behavior, financial logic, auth, and state transitions early. Keep each test
file beside the production behavior it verifies rather than reviewing tests as
a separate block.

Split a very large file into coherent targets such as validation, query
construction, core algorithm, persistence, error handling, or rendering.
Present the route briefly, identify the highest-value manual-review targets,
name the first target, then stop and ask whether to begin it. Label which route
targets are likely to require direct human code reading, but defer exact line
assignments until that target is current. The initial route is an agenda, not
the file review itself.

## Build a context packet for each full-mode target

At the start of a file or file section, inspect the full relevant code, its base
version, callers, callees, types, helpers, consumers, and relevant test groups.
Then provide a concise context packet for that current target only, containing
the useful parts below. Never batch packets for every file into one response.

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

### Explicit human reading assignment

Every target packet must clearly separate code the user should read from links
provided only as supporting context. Include a visually distinct `Read this
code` block; ordinary file and line links elsewhere in the explanation do not
count as a reading assignment.

Use only the parts that apply:

- **Read directly:** point to the exact function, branch, diff hunk, or smallest
  useful line range. Explain why it deserves human judgment, what question the
  user should answer while reading, and what failure would look like.
- **Read the assertions:** point to critical assertions, mocks, fixtures, or
  expected values when the test oracle itself deserves human validation.
- **Agent-covered:** identify code whose behavior is sufficiently established
  by the context packet, strong tests, deterministic checks, or mechanical
  inspection and therefore does not need manual source reading.
- **Optional drill-down:** link code that becomes worth reading only if the user
  disputes an assumption, test result, or explanation.

Keep the assignment sparse and risk-based. Prefer a few consequential snippets
over broad file links or asking the user to understand every line equally. If a
target genuinely requires no manual source reading, say `No manual code reading
recommended for this target` and explain the evidence supporting that choice.
Never label a location as read, understood, or verified merely because it was
linked or summarized.

Walk through assigned snippets conversationally rather than dumping a list.
Make the exact judgment explicit, let the user inspect or question the code, and
remain on the current target until the user chooses to continue.

### Second-pair-of-eyes findings

- Surface concrete suspicious behavior, missing cases, inconsistent contracts,
  or surprising assumptions found while reconstructing the context.
- Label uncertainty and evidence. Do not invent findings to appear thorough.

Walk through a large packet in conversational pieces rather than dumping it all
at once. Stay on the target while the user asks questions or requests changes;
move on when the user chooses. Relate each completed target back to the evolving
system mental model and preserve cross-file invariants so files are not
understood in isolation.

### Manual product verification

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

### Mandatory close implementation inspection

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
to the exact implementation details that determine correctness and include
them under `Read directly`; a contextual link alone is not sufficient.

### Changes during review

If the user requests a change, make the smallest in-scope edit, run focused
validation, and return to the affected target with updated context. Do not add a
separate review-management workflow.

## 4. Quiz

In full mode, start the PR-level quiz after every changed file has been reviewed
and create five interactive multiple-choice questions. In quick mode, start
after the one-response overview and create exactly three questions. In both
modes, make them medium difficulty: answering should require understanding the
substance of the PR, but the questions must not be gotchas. Ask them one at a
time and test the user's ability to:

- trace the main input -> transformation -> state or output flow;
- explain a consequential invariant or contract;
- predict a realistic edge case, failure path, or counterfactual;
- state what the tests prove and what remains unproved;
- explain a downstream effect, debugging entry point, or recovery path.

Test behavior, causality, and system understanding rather than line trivia.
Use plausible, similarly worded options tied to real misunderstandings, vary
the correct-answer position, and avoid answer cues such as one conspicuously
long or precise option. Do not reveal the answer before the user responds.

When Plan mode's structured question tool is available, use it so the user can
select an answer interactively. Otherwise present the choices conversationally
and wait for the user's selection. Immediately after each answer, state whether
it was correct and give useful feedback explaining the relevant behavior and
why the selected answer was right or wrong.

Evaluate each answer against the inspected code. If an answer is wrong,
incomplete, or uncertain, identify the missing mental model, revisit only the
relevant background, intuition, file, tests, or exact lines, and then ask a
different question that proves the gap has closed. Do not treat guessing a
multiple-choice answer as understanding.

Passing the quiz establishes that the user can reason about the changed system;
it does not override unresolved defects, weak evidence, failed checks, or
missing manual verification. End with a concise synthesis of confirmed
concerns, unresolved cross-file assumptions, verification gaps, and remaining
manual checks. Leave the merge decision to the user.
