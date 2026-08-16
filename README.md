<img src=".github/logo.svg" alt="Chock logo" width="90">

# chock-example

> **Demo repository.** This repo exists so you can *see* a working [Chock](https://github.com/open-coder-ai/chock)
> adoption — one policy per artifact layer, three total, instead of the full
> [catalog](https://github.com/open-coder-ai/chock-catalog)'s twenty-eight. Questions and
> issues belong on the [framework repo](https://github.com/open-coder-ai/chock/issues).
> Click **Use this template** to start your own.

## One policy per layer

| Layer | Policy here | What enforces it | Where to look |
| :--- | :--- | :--- | :--- |
| **Hook** (blocks) | `protect-main-branch` | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`, wired into `.git/hooks` — commits, merges and pushes to `main` **fail** (pre-commit, pre-merge-commit and pre-push dispatchers, plus a CI gate step) | [`​.agents/policies/protect-main-branch/`](.agents/policies/protect-main-branch/) |
| **Rule** (advises) | `block-no-verify` | Ambient text compiled into `AGENTS.md` and every agent wrapper — plus a compiled PreToolUse guard that blocks `--no-verify` at tool time in Claude Code | [`​.agents/policies/block-no-verify/`](.agents/policies/block-no-verify/) |
| **Skill** (does) | `commit-message-style` | Invoked by the agent when the task matches; evals in `evals/suite.yaml` define what "working" means | [`​.agents/skills/commit-message-style/`](.agents/skills/commit-message-style/) |

The three layers answer three different questions: what the agent **cannot do** (hook),
what it **should know** (rule), and what it **can be asked to do well** (skill).

Beyond the three demo policies, `chock init` also leaves the bundled authoring skills in
`.agents/skills/`, per-directory guardrail files stating the provenance-and-editing
contract, a Claude Code skills bridge under `.claude/skills/`, and a `.gitattributes`
pinning generated content to LF.

## How this repo was made

```bash
chock init .                        # wiring (see chock-quickstart for just this)
chock add protect-main-branch       # hook, from the catalog
chock add block-no-verify           # rule, from the catalog
chock new skill commit-message-style  # skill, authored here (SKILL.md + evals)
chock sync
```

## Try the enforcement

```bash
git checkout main
echo x >> README.md && git add . && git commit -m "direct to main"
# Direct commits/pushes to a protected branch (main|master) are blocked. Create a feature branch and open a pull request.
#   - main

chock status          # what's installed, what each layer claims
chock check           # validate + verify + evals, all green
```

Everything under `.chock/compiled/` is generated — `chock sync` rebuilds it, and
`chock check` fails if it ever drifts from the policy sources. That claims-match-mechanism
loop is the point of the tool.
