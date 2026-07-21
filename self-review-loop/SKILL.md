---
name: self-review-loop
description: Run one bounded independent read-only review after a substantial implementation. Use when Codex should check a completed change against its explicit task or issue, direct regressions, and code simplicity before PR acceptance, without starting an open-ended multi-agent review loop.
---

# Self Review Loop

Run one independent read-only reviewer by default. The calling agent owns triage and coordinates any edits; the reviewer only provides feedback.

## Review standard

- Review the explicit task or GitHub issue, its stated failure cases, and direct regressions.
- Treat a finding as actionable only when it identifies a violated requirement or realistic failure, gives exact code evidence or a reproduction, and proposes the smallest fix.
- Prefer deletion, reuse, or simplification. Do not add abstractions, state models, schemas, recovery protocols, dependencies, or broad defensive handling unless the task requires them.
- Treat theoretical risks, unstated behaviors, subjective preferences, and out-of-scope hardening as observations, not fixes.
- Keep the reviewer read-only and forbid state-changing tools because agents share the workspace.

## Workflow

1. Inspect the task, diff, status, and existing validation.
2. Launch one independent reviewer via a subagent. Add another only when the user or repository explicitly requires a genuinely separate review method.
3. Verify each finding and classify it as fix, simplify, reject, or needs-user-decision.
4. Make the smallest validated correction and rerun only relevant validation.
5. Run another review only if a confirmed fix substantially changed behavior or meaningful uncertainty remains.

Stop before editing when a finding would change an unapproved product contract, public API, persistence shape, permissions, security posture, dependency set, migration strategy, generated data, or user-visible workflow. Also stop when the proposed fix would materially grow the implementation or introduce a concept absent from the task.

Report the reviewer, accepted and rejected findings, resulting validation, and residual risks.
