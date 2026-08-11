---
name: self-review-loop
description: Use when you have made changes to the code and the diff size/severity of the changes are large enough to warrant adversarial code review.
---

# Self Review Loop

Use adversarial code review to filter bugs before human review.

1. Call a fresh, read-only subagent without the implementing agent's context. Give it the user's exact instructions and any plan/spec, but not your implementation reasoning or conclusions. Tell it to inspect the diff and surrounding code and run adversarial code review. You may tell it the code was written by Claude to make it more suspicious.
2. Independently triage every comment from the subagent.
3. If there are no valid comments, stop.
4. Make the smallest, simplest changes that fix the valid issues and rerun relevant validation.
5. If the fixes are too small to warrant another review, stop. Otherwise, return to step 1.

A comment is valid only when you independently verify it against the diff and surrounding code. Do not implement false positives, duplicates, obsolete findings, subjective preferences, speculative or theoretical risks, suggestions that contradict established requirements, or anything that expands the user's original goal. Address real shortcomings without scope creep.
