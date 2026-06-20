---
name: issue
description: >
  Create a GitHub issue from the current conversation context. Use this skill whenever
  the user says "issue", "create an issue", "file an issue", "open an issue",
  "make an issue", "log this as an issue", or types /issue. Also trigger when the user
  has been discussing a bug, feature, or task and says something like "let's track this",
  "we should file this", or "add this to the board". The skill synthesizes the
  conversation context into a well-structured issue, so the user should not need to
  re-explain what the issue is about.
---

# Create GitHub Issue

Create a GitHub issue in the repository relevant to the current conversation. Do not use
hard-coded repository names, owners, project boards, project IDs, field IDs, or personal
account details from examples or old local state. If the target repo is not obvious from
the current working directory, Git remote, or conversation, ask the user before creating
the issue.

## How it works

The user has been working with you in conversation — discussing a bug, feature, refactor, or task. When they invoke this skill, your job is to synthesize that conversation context into a well-structured GitHub issue. They should NOT need to repeat themselves or pass the issue content as an argument.

## Step 1: Synthesize from conversation context

Read back through the conversation to understand what the issue is about. Identify:
- What the problem or feature is
- Why it matters
- What needs to be done
- Any technical details, file paths, or code references discussed

## Step 2: Draft the issue

Present the user with a draft before creating it. Include:

**Title** — concise, action-oriented (e.g. "Add rate limiting to support chat", "Fix hydration mismatch in portfolio tabs")

**Label** — pick ONE from: `bug`, `enhancement`, `refactor`, `epic`, `tests`, `documentation`, `question`, `needs repro`. Choose based on the nature of the work, not what the user called it.

**Body** — use this structure:

```markdown
## Context

[1-3 paragraphs explaining the background, current behavior, and why this matters]

## Action items

- [ ] First task
- [ ] Second task
- [ ] ...
```

For bugs, add a `## Current behavior` and `## Expected behavior` section before Action items. For epics, use `## Requirements` with subsections instead of Action items.

Include relevant code snippets, file paths, or references from the conversation only when
they help future readers understand the issue. Do not include private paths, secrets,
tokens, personal account identifiers, or unrelated local-machine details.

**Priority** — P0 (urgent/blocking), P1 (important, do soon), P2 (nice to have, backlog). If the user specified a priority, use it. Otherwise, infer from context: bugs in production paths are P0-P1, new features are P1-P2, tech debt is P2.

**Size** — XS (< 1 hour, trivial change), S (a few hours, straightforward), M (half a day to a day, some complexity), L (multi-day, significant scope), XL (multi-day, major feature or refactor). If the user passed `--size` or mentioned a size, use it. Otherwise, estimate based on the number and complexity of action items.

## Step 3: Get confirmation

Show the draft to the user and ask if they want to adjust anything. Wait for confirmation before creating.

## Step 4: Create the issue

Once confirmed, execute these steps:

### Resolve the repository

Prefer the current GitHub remote:

```bash
gh repo view --json nameWithOwner --jq .nameWithOwner
```

If that fails, infer the repository from conversation context. If still unclear, ask the
user for the target owner/repo.

### Create the issue

Use a temporary body file or heredoc to preserve formatting. Prefer `--repo "$REPO"` only
after resolving it explicitly.

```bash
REPO="$(gh repo view --json nameWithOwner --jq .nameWithOwner)"
gh issue create \
  --repo "$REPO" \
  --title "<title>" \
  --label "<label>" \
  --body-file /path/to/issue-body.md
```

Capture the issue number from the output.

### Report back

Tell the user the issue was created with a link to it. Report the label, priority, and
size that were chosen in the issue body. Do not add the issue to a project board unless
the user explicitly asks for that in the current conversation.

## Optional arguments

The user can pass these as arguments when invoking the skill:
- `--priority P0|P1|P2` — override the inferred priority
- `--size XS|S|M|L|XL` — override the inferred size
- `--label <label>` — override the inferred label

If not provided, infer all three from context.
