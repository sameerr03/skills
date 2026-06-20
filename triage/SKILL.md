---
name: triage
description: Read-only triage of pull request review comments, GitHub comments, Greptile/Copilot/code-review findings, or pasted reviewer feedback. Use when Codex should inspect the relevant codebase, files, conventions, tests, diffs, and assumptions thoroughly, search for false positives, and recommend whether each issue should be fixed, deferred, or rejected without making any code changes.
---

# Triage

## Overview

Use this skill to evaluate reviewer comments as a careful, read-only code reviewer. The goal is not to implement fixes. The goal is to determine whether each comment is valid, whether it matters, and whether the user should fix it.

## Hard Rules

- Do not edit files, stage changes, commit, push, run formatters that write files, or apply patches.
- Treat every reviewer comment as a hypothesis, not a fact.
- Inspect the current code and surrounding conventions before deciding.
- Search for evidence that a comment is a false positive, already handled elsewhere, intentionally excluded, covered by an invariant, or inconsistent with local style.
- If a command might mutate state, do not run it. Prefer read-only commands such as `rg`, `sed`, `git diff`, `git show`, `git blame`, test listing commands, and type or test commands only when they are known not to write.

## Workflow

1. Capture the comments.
   - Preserve the reviewer, file, line, quoted code, and exact claim when available.
   - If comments are pasted without context, infer likely target files, then verify by searching.
   - If a PR or branch is referenced, inspect the diff against the target branch, usually `main` or `origin/main`.

2. Build codebase context.
   - Read the target file and nearby code.
   - Search for similar patterns, helper APIs, tests, naming conventions, error handling, and established exceptions.
   - Inspect relevant call sites and data flow instead of judging only the commented line.
   - Check tests or docs that define the expected behavior when they are nearby or clearly relevant.

3. Triage each comment.
   - Decide one of: `Fix`, `Do not fix`, `Defer`, or `Needs clarification`.
   - Mark likely false positives explicitly.
   - Separate correctness/security/data-loss issues from style preferences, speculative maintainability concerns, and reviewer taste.
   - Prefer fixing comments that identify real behavioral bugs, broken contracts, user-visible regressions, security/privacy risk, missing validation at trust boundaries, or test gaps around changed behavior.
   - Prefer not fixing comments contradicted by existing conventions, covered by upstream validation, irrelevant to the changed code, too speculative, or worse than the current design tradeoff.

4. Report clearly.
   - Lead with the recommendation summary.
   - For each comment, cite the evidence checked: files, functions, related patterns, tests, or diffs.
   - Explain why the recommendation follows from the evidence.
   - Include the smallest plausible fix direction for comments marked `Fix`, but do not write code.
   - Call out residual uncertainty and what evidence would resolve it.

## Output Format

Use this shape unless the user asks for another format:

```markdown
**Recommendation**
- Fix: N
- Do not fix: N
- Defer: N
- Needs clarification: N

**Comments**
1. [Fix|Do not fix|Defer|Needs clarification] Short title
   Comment: concise restatement of the reviewer claim.
   Evidence checked: files/functions/patterns/tests inspected.
   Reasoning: why the claim is valid or invalid.
   Suggested response: what to tell the reviewer or what fix direction to take.
```

Keep the answer concise, but include enough evidence that the user can trust the call without redoing the investigation.
