---
name: pr
description: >
  Create a draft pull request using the GitHub CLI. Use this skill whenever the user
  says "create a PR", "make a PR", "open a PR", "draft PR", "pull request", or any
  variation of wanting to create a pull request. Also trigger when the user types /pr.
  This skill creates draft PRs with comprehensive descriptions that cover the full
  planned scope of the PR — not just what has been committed so far.
---

# Draft Pull Request

Create a draft PR with a description that covers the **full intended scope** of the changes, not just what's been committed so far.

## Why draft PRs with full-scope descriptions

The user works in draft mode — they create the PR early and continue pushing changes. The PR description should reflect everything that's planned for this PR based on prior discussions, plans, or todo files in the conversation. This way reviewers and collaborators can understand the full picture from the start, and the description doesn't need constant rewriting as more commits land.

## Workflow

1. **Gather context** — run these in parallel:
   - `git status` to check for uncommitted changes
   - `git diff` for staged/unstaged changes
   - `git log main...HEAD --oneline` (or appropriate base branch) to see all commits on this branch
   - `git diff main...HEAD` to see the full diff from the base branch
   - Check if the branch tracks a remote and is up to date

2. **Determine the full scope** — look at:
   - All commits on the branch (not just the latest one)
   - Any plans, todo files, or discussion context from the current conversation
   - The user may have discussed features or changes that haven't been implemented yet but are part of this PR's scope — include those in the description

3. **Commit if needed** — if there are uncommitted changes, ask the user if they want to commit first.

4. **Push and create** — push with `-u` flag if needed, then create the draft PR:

```bash
gh pr create --draft --title "short title under 70 chars" --body "$(cat <<'EOF'
## Summary
<bullet points covering the FULL scope of the PR — both what's done and what's planned>

## Changes
<group related changes together, describe what each group accomplishes>

## Test plan
- [ ] checklist of testing items
EOF
)"
```

5. **Open in browser** — run `gh pr view <number> --web` to open it for the user.

## Rules

- Always create as **draft** (`--draft` flag)
- The description must cover the entire planned scope, not just current commits
- Keep the title short (under 70 characters) — details go in the body
- Group related changes in the description for readability
- If the base branch isn't obvious, ask the user
