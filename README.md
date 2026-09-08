<div align="center">

<img src=".github/logo.svg" alt="chock-example: a working chock adoption you can read end to end — one policy per artifact layer: git hook, agent rule and skill. The mark is chock's: a wheel held by a chock wedge." width="90">

# chock-example

[![Demo repository](https://img.shields.io/badge/demo-repository-lightgrey)](https://github.com/open-coder-ai/chock)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Issues and PRs](https://img.shields.io/badge/issues%20%26%20PRs-chock-8957e5)](https://github.com/open-coder-ai/chock/issues)

**A working [Chock](https://github.com/open-coder-ai/chock) adoption you can read — one policy per artifact layer.**

[the framework →](https://github.com/open-coder-ai/chock) ·
[the full catalog →](https://github.com/open-coder-ai/chock-catalog) ·
[just the bare scaffold →](https://github.com/open-coder-ai/chock-quickstart)

</div>

> **Demo repository.** Three policies instead of the full catalog's twenty-eight, so every
> moving part fits in one sitting. Questions and issues belong on the
> [framework repo](https://github.com/open-coder-ai/chock/issues).
> Click **Use this template** to start your own.

## One policy per layer

| Layer | Policy here | What enforces it | Where to look |
| :--- | :--- | :--- | :--- |
| **Hook** (blocks) | `protect-main-branch` | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`, wired into `.git/hooks` — commits, merges and pushes to `main` **fail** (pre-commit, pre-merge-commit and pre-push dispatchers, plus a CI gate step) | [`​.agents/policies/protect-main-branch/`](.agents/policies/protect-main-branch/) |
| **Rule** (advises) | `block-no-verify` | Ambient text compiled into `AGENTS.md` and every agent wrapper — plus a compiled PreToolUse guard that blocks `--no-verify` at tool time in Claude Code | [`​.agents/policies/block-no-verify/`](.agents/policies/block-no-verify/) |
| **Skill** (does) | `commit-message-style` | Invoked by the agent when the task matches; evals in `evals/suite.yaml` define what "working" means | [`​.agents/skills/commit-message-style/`](.agents/skills/commit-message-style/) |

The three layers answer three different questions: what the agent **cannot do** (hook),
what it **should know** (rule), and what it **can be asked to do well** (skill).

## Try the enforcement

```bash
git checkout main
echo x >> README.md && git add . && git commit -m "direct to main"
# Direct commits/pushes to a protected branch (main|master) are blocked. Create a feature branch and open a pull request.
#   - main

chock status          # what's installed, what each layer claims
chock check           # validate + verify + evals, all green
```

## How this repo was made

```bash
chock init .                        # wiring (see chock-quickstart for just this)
chock add protect-main-branch       # hook, from the catalog
chock add block-no-verify           # rule, from the catalog
chock new skill commit-message-style  # skill, authored here (SKILL.md + evals)
chock sync
```

Beyond the three demo policies, `chock init` also leaves the bundled authoring skills in
`.agents/skills/`, per-directory guardrail files stating the provenance-and-editing
contract, a Claude Code skills bridge under `.claude/skills/`, and a `.gitattributes`
pinning generated content to LF.

Everything under `.chock/compiled/` is generated — `chock sync` rebuilds it, and
`chock check` fails if it ever drifts from the policy sources. That claims-match-mechanism
loop is the point of the tool.

## Where to go from here

- **Adopt it in your own repository.** `pip install chock && chock init .`, then `chock add`
  the policies that match how your team gets hurt; the
  [catalog](https://github.com/open-coder-ai/chock-catalog) labels each one with what it actually enforces.
- **Contribute a policy.** The [catalog's contributing guide](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md) is short
  and its rules are mechanical: a policy claims only what it can do, and evals are the
  argument. The `policy wanted` entries in the [threat ledger](https://github.com/open-coder-ai/chock-threat-intel/blob/main/reference/agentic-threat-ledger.md) are the
  open work list.
- **Found something wrong in this exhibit?** This tree is the output of the framework's own commands, so the fix belongs there. Issues go to the
  [framework repo](https://github.com/open-coder-ai/chock/issues/new/choose).

## Part of the open-coder-ai family

Everything under [open-coder-ai](https://github.com/open-coder-ai) is built on one rule: a claim must match a
mechanism. Where this repository sits among the others:

| Repository | What it is |
| :--- | :--- |
| [chock](https://github.com/open-coder-ai/chock) | The framework: write a policy once, enforce it on git hooks, CI, and every agent |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | The policies, each graded by what it actually enforces |
| [agentseam](https://github.com/open-coder-ai/agentseam) | The primitives layer under chock: one handler API over every agent's hooks, with a capability matrix that carries its provenance |
| [context-report](https://github.com/open-coder-ai/context-report) | A signed report format for whether a plugin, hook, skill or `AGENTS.md` actually works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | A weekly, human-reviewed threat digest scored against the catalog |
| [chock-claude-plugins](https://github.com/open-coder-ai/chock-claude-plugins) · [copilot](https://github.com/open-coder-ai/chock-copilot-plugins) · [cursor](https://github.com/open-coder-ai/chock-cursor-plugins) · [codex](https://github.com/open-coder-ai/chock-codex-plugins) | The catalog compiled into each client's native plugin format; generated only, rebuilt and diffed in CI |
| [chock-quickstart](https://github.com/open-coder-ai/chock-quickstart) | The bare scaffold: exactly what `chock init` leaves behind, no policies |
