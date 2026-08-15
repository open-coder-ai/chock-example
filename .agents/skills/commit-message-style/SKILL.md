---
name: commit-message-style
description: Write a Conventional Commit message for staged changes. args(diff_summary) returns(commit_message) invoke(commit, message, changelog) exclude(code_review, push)
metadata:
  owner: chock-example
  version: "0.0.1"
  status: draft
---

# Commit Message Style

Produce one Conventional Commit message for the staged diff.

## Format

```
<type>(<scope>): <imperative summary, <=72 chars>
```

- type: `feat` | `fix` | `docs` | `test` | `chore` | `refactor`
- scope: the subsystem touched (one word)
- body (optional): the WHY, wrapped at 72; never restate the diff

## Rules

- One logical change per commit; if the diff mixes concerns, say so instead of inventing an umbrella summary.
- Never claim a fix works without naming the check that proves it.
- End with `Signed-off-by` when the repo requires DCO.
