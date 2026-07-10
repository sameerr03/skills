---
name: pr
description: Create a ready pull request using the GitHub CLI. Use whenever the user says "create a PR", "make a PR", "open a PR", "pull request", or types /pr. Build a comprehensive description from the full branch and conversation scope, link the exact implementation issue with a closing keyword when the PR fully completes it, reference related epic or grouping issues without closing them, push the branch, and open the PR ready for review unless the user explicitly asks for a draft.
---

# Create Pull Request

Create a ready, non-draft PR whose description covers the complete intended scope of the branch, not only its latest commit.

## Workflow

1. **Inspect the branch.** In parallel, check `git status`, staged and unstaged changes, branch tracking state, commits against the intended base, and the complete base-to-head diff.
2. **Determine scope and base.** Use the branch history, current conversation, plans, and issue context. Ask only when the base or intended PR boundary cannot be inferred safely.
3. **Resolve issue links.** Identify the single implementation issue this PR fully completes, plus any related epic, parent, grouping, research, or dependency issues. Never guess an issue relationship.
4. **Handle uncommitted work.** If relevant changes are uncommitted, ask whether the user wants them committed before creating the PR.
5. **Push and create.** Push with upstream tracking when needed, then create the PR ready for review. Use draft mode only when the user explicitly requests it.
6. **Verify.** Read the created PR back to confirm its title, base, head, draft state, body, and issue links, then report the URL.

## Description

Use this structure unless the repository provides another template:

```markdown
## Summary
- Complete behavior and outcome of this PR

## Changes
- Related implementation groups

## Test plan
- [ ] Relevant validation

Closes #123
Related to #100
```

Include `Closes #N` only for the implementation-leaf issue the PR fully satisfies. GitHub will link it and close it when the PR is merged. If the PR only partially addresses an issue, use `Related to #N` instead. Reference the epic, parent, grouping, research, or contextual issues without closing them. Omit either line when no verified matching issue exists.

## Rules

- Default to ready/non-draft; respect an explicit request for a draft.
- Keep the title under 70 characters and put detail in the body.
- Describe the full PR boundary and group related changes clearly.
- Do not claim unfinished work, unrun validation, screenshots, or review results.
- For stacked PRs, use the intended predecessor branch as the base and preserve one closing issue per implementation PR.
