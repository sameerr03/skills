---
name: create-pr
description: Use whenever the user asks to create, open, or publish a pull request.
---

# Create PR

Read the request, issue, and final diff before writing the pull request.

Format titles as `<type>(<scope>): <intent>`. Use `feat`, `fix`, `research`, `docs`, `refactor`, `perf`, `test`, `ci`, `data`, or `chore`. Use `research` for exploratory work and `data` for evidence or dataset changes.

Choose the smallest stable owning service or product surface. Prefer repository instructions and established scopes rather than inventing a new one.

- Finbelief: `beliefs`, `principles`, `portfolio`, `stock`, `research`, `support`, `marketing`, `platform`, `admin`, `auth`, `convex`, `infra`, `broker`.
- Finbelief Engine: `principles`, `beliefs`, `financials`, `ingestion`, `database`, `research`, `platform`, `infra`.

Use the product scope when a change crosses frontend and backend. Reserve technical scopes such as `convex` or `database` for genuinely cross-cutting infrastructure. Describe the intended outcome, not files or implementation details.

Bad: `Refactor product mode resolver`

Good: `feat(platform): split product surfaces by hostname`

Keep the body focused on why the change exists, what behavior changed, and how it was verified. Use the user's initial request and discussion to understand the intent behind the change when writing the title and description. Include UI evidence, exclusions, or residual risks only when relevant. Do not restate the diff. Use `Closes #N` only when the pull request fully closes an issue.

Open it as a draft unless the user requests to create it as open. Do not merge.
