---
name: file-pr
description: File a concise pull request. Use when the user asks to file, open, or create a PR. For user-requested visual evidence on user-visible UI changes, conditionally invoke $file-upload after the PR exists; never invoke it for backend-only work or ordinary PR creation.
---

# File PR

Before filing, check whether a PR for this branch already exists. Review the diff locally against `origin/dev` to make sure its contents match the goal.

PR titles usually become commit messages, so follow the repository's title conventions. Look at recently merged PRs and Git history for examples.

Prefer a concise, human-readable title that explains why the change matters:

BAD
> test(ci): gate the complete Phase 3 journey

GOOD
> test(ci): validate the Phase 3 Agency Admin to HRMO workflow

Open the description with a simple explanation of the problem based on the user's original prompt or related ticket, then briefly explain the solution. Use ASD-STE100 Simplified Technical English.

Use this exact Markdown structure for every PR description:

```markdown
## Issue report

[Issue description]

## Solution

[Solution description]

## Related work

[Related issue or work]

## Validation

[Validation description]
```


Open a real PR rather than a draft so review bots run if the repository has AI reviewers.

After opening the PR, watch its CI checks until they finish. Filing is complete only when every CI check is green. If a check fails, inspect it, make and push any in-scope fix, then watch the new run. Report an externally blocked check as a blocker and the PR as incomplete.

## Optional UI evidence

After the PR exists, invoke `$file-upload` only when every gate passes:

1. The PR contains a user-visible UI change.
2. The user explicitly asked to upload, attach, or publish screenshots, recordings, or other visual evidence, or explicitly named `$file-upload`.
3. The evidence files are available and safe to publish.

Pass the created PR and exact evidence paths to `$file-upload`, then wait for its publication verification before reporting the PR complete. Use it for before/after images when requested and for a short video when the requested UI behavior depends on motion or timing.

For backend-only work, skip `$file-upload` unconditionally. For UI work without an explicit upload request, create the PR and report validation without attaching files.
