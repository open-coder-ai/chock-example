---
name: block-no-verify
description: "Best-effort guard against bypassing git hooks via git commit/push --no-verify or -n. Known bypass classes include aliases, wrapper scripts, and non-standard clients. Fix the underlying hook failure instead of skipping validation."
metadata:
  chock.artifact: rule
  chock.enforcement: advise
  chock.coverage_without_chock: advisory
---

# Block No-Verify

Best-effort guard against bypassing git hooks via git commit/push --no-verify or -n. Known bypass classes include aliases, wrapper scripts, and non-standard clients. Fix the underlying hook failure instead of skipping validation.

```
never(commit|push): --no-verify|-n
if(hook_fails): fix_issue; never(skip_hook)
```

This skill is advisory: the client reading it has no mechanism to enforce it. The same policy compiled by `chock` can refuse an agent's shell command before it runs. See https://github.com/open-coder-ai/chock
