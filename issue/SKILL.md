---
name: issue
description: >
  Create a GitHub issue in the repository currently being worked in. Use this skill
  whenever the user says "issue", "create an issue", "file an issue", "open an issue",
  "make an issue", "log this as an issue", or types /issue. Also trigger when the user
  has been discussing a bug, feature, or task and says something like "let's track this",
  "we should file this", or "add this as an issue". The skill synthesizes the
  conversation context into a well-structured issue so the user does not need to
  re-explain it.
---

# Create GitHub Issue

Create a GitHub issue in the repository currently being worked in.

## Workflow

1. Determine the target repository from the current working tree. Prefer
   `gh repo view --json nameWithOwner --jq .nameWithOwner`; if needed, infer it
   from `git remote get-url origin`.
2. Synthesize the issue from the conversation context. Capture the problem or
   feature, why it matters, what needs to be done, and any relevant file paths,
   logs, commands, or code references already discussed.
3. Draft the issue and show it to the user before creating it. Wait for explicit
   confirmation.
4. Create the issue with `gh issue create --repo <owner/repo>`.
5. Report back with the issue link.

## Issue Draft

Include:

- **Title**: concise and action-oriented.
- **Label**: pick one if the repository has an appropriate existing label, or
  omit labels if unsure.
- **Body**: clear, actionable markdown.

Use this body structure by default:

```markdown
## Context

[Brief background and why this matters.]

## Action items

- [ ] First task
- [ ] Second task
```

For bugs, include `## Current behavior` and `## Expected behavior` before action
items. Include relevant reproduction steps, logs, file paths, or code references
when available.

Add `Generated with [Codex](https://codex.com)` at the end of the body.

## Create Command

Use a heredoc or temporary file so markdown formatting is preserved:

```bash
gh issue create --repo <owner/repo> --title "<title>" --body-file <body-file>
```

If using a label, add `--label "<label>"`.

## Optional Arguments

- `--repo owner/name`: override the repository inferred from the current working
  tree.
- `--label <label>`: use a specific label.

If an optional argument is not provided, infer only what is safe from the current
repository and conversation context.
