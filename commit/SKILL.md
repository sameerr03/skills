---
name: commit
description: >
  Create a git commit for the user based on current staged and unstaged changes.
  Use this skill whenever the user says "commit", "commit this", "commit my changes",
  "make a commit", "save my work", or any variation of wanting to create a git commit.
  Also trigger when the user types /commit or /c. Accepts an optional "push" argument
  (e.g., "/commit push") — when provided, push to remote after committing. Without
  the push argument, never push.
---

# Commit

Create a well-crafted git commit from the user's current changes.

## Workflow

1. **Understand the changes** — run `git status` and `git diff` (both staged and unstaged) to see everything that's changed. Also run `git log --oneline -5` to pick up the repo's commit message style.

2. **Stage selectively** — add relevant changed files by name. Avoid `git add -A` or `git add .` which can accidentally include sensitive files (.env, credentials) or large binaries. If there are untracked files that look intentional, include them. If unsure, ask.

3. **Write the commit message** — summarize the *why*, not the *what*. Keep the title under 72 characters. Add a body only if the change needs context that isn't obvious from the diff. Match the style of recent commits in the repo.

4. **Commit** — create the commit and run `git status` afterward to confirm success.

5. **Push (only if `push` argument is provided)** — if the user invoked this skill with the `push` argument (e.g., `/commit push`), run `git push` after the commit succeeds. Otherwise, do not push.

## Rules

- Only push to remote when the `push` argument is explicitly provided
- Never amend previous commits — always create new ones
- Never skip hooks (no `--no-verify`)
- Do not commit files that likely contain secrets (.env, credentials.json, etc.) — warn the user if they ask to
- If a pre-commit hook fails, fix the issue and create a new commit (do not amend)
- Use a HEREDOC for the commit message to preserve formatting
