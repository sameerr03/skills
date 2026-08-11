---
name: execute-plan
description: Use when the user wants one long-running task to execute an agreed large plan as a stack of review-ready pull requests.
---

# Execute Plan

For each planned PR:

1. Create the next GitHub stack layer.
2. Implement against the plan's validation contract. Where practical, establish the failing test or reproduction first, then iterate until all relevant validation passes.
3. Run `$self-review-loop`.
4. Run Computer Use-based validation for smoke testing, UI/design review, real user flows, and judging whether the UX is the right direction when required by the plan.
5. Submit the layer, apply `$create-pr`, then run `$auto-pr`.
6. Continue with the next PR.

Use `gh stack init <branch>` for the first layer and `gh stack add <branch>` from the top for later layers. Submit with `gh stack submit --auto --open`, apply the `$create-pr` title and description with `gh pr edit`, and verify with `gh stack view --json`.

When a lower layer changes, edit its branch, run `gh stack rebase --upstack`, revalidate affected layers, then run `gh stack push`. Use `--continue` or `--abort` for conflicts. Pass `--remote <name>` when the repository has multiple remotes.

Finish when the plan is satisfied and every PR is ready to merge. Report the native stack order and validation. Never merge.
