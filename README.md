<div align="center">

<img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/readme/cover-chock-example.png" alt="chock-example: a small working adoption of chock, with one policy per layer: a git hook, an agent rule and a skill." width="760">

# Teach your AI agent what not to do.

**Open-source guardrails for AI coding agents: rules the agent reads, checks that run as it writes, and gates at commit and in CI. This template is a working adoption you can read end to end: one policy per layer.**

[the framework →](https://github.com/open-coder-ai/chock) ·
[the catalog →](https://github.com/open-coder-ai/chock-catalog) ·
[the bare scaffold →](https://github.com/open-coder-ai/chock-quickstart)

</div>

**What this is.** chock-example is a GitHub template repository: a repo where Chock is already adopted, with three artifacts installed so every moving part fits in one sitting. Chock is open-source application security for code written by AI coding agents. Each check is a deterministic local script, with no model and no upload, and it refuses known vulnerability classes before they are committed.

> **Demo repository.** Questions and issues belong on the [framework repo](https://github.com/open-coder-ai/chock/issues).

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/demo.gif" alt="Terminal recording of chock refusing unsafe commits and accepting the fixed ones." width="760">
</p>

## Application security for the code your agents write

Coding agents already ask before they run a shell command. What they do not check is the code they write. Chock refuses known classes of insecure code and unsafe actions while the agent writes, at commit and in CI. This repo installs two guardrails and one skill so you can see each layer work. The [catalog](https://github.com/open-coder-ai/chock-catalog) holds the application-security policies you would add next.

## Install

Chock is not on PyPI. Install the frozen engine from its commit (Python 3.11 or newer):

```bash
pip install "chock @ git+https://github.com/open-coder-ai/chock@992711af4cf8d4fd9c4c861f10ef6e53374d75d7"
```

Click **Use this template**, clone your copy, then run `chock sync --repo .` once. Git never clones hooks, so every clone does this.

## Two ways to adopt

<img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/readme/adopt.png" alt="The two adoption routes: policies installed in your repository, or Chock plugins installed in your coding agent." width="760">

| | In your repository (for teams) | In your coding agent, as plugins |
| :--- | :--- | :--- |
| Steps | `chock init .`, then `chock add <id> --ref <catalog commit> --verify-sha <sha256> --skip-compile` for each policy, then `chock sync --repo . --ci` | Install the plugin for your client from its repo (below) |
| Where it runs | Your agent's hook where its client has one, at commit, and in CI | The client's pre-tool hook only |
| Strength | The commit gates are enforced at commit and in CI. Commit the result | Best-effort: the client's hook fails open, and it does not run in CI |

This repo is a result of the repository route. Plugin repos, with per-client install lines in each README: [Claude Code](https://github.com/open-coder-ai/chock-claude-plugins), [Copilot](https://github.com/open-coder-ai/chock-copilot-plugins), [Cursor](https://github.com/open-coder-ai/chock-cursor-plugins), [Codex](https://github.com/open-coder-ai/chock-codex-plugins), [Devin](https://github.com/open-coder-ai/chock-devin-plugins). A third route, one Claude Code plugin from a selection of policies, comes from the chock.sh builder (launching soon).

## How it works: one policy per layer

| Layer | Policy | Tier in the catalog | Mechanism here |
| :--- | :--- | :--- | :--- |
| Hook | `protect-main-branch` | Enforced at commit | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`: commits, merges and pushes to `main` fail (pre-commit, pre-merge-commit, pre-push), plus a CI gate step |
| Rule | `block-no-verify` | Best-effort (in the agent, fails open) | Ambient text in `AGENTS.md` and every agent wrapper, plus a compiled pre-tool guard that refuses `--no-verify` and `-n` where the client has a hook: wired here for Claude Code, Gemini CLI and VS Code Copilot |
| Skill | `commit-message-style` | Invoked, not gated | Runs when the agent's task matches; its `evals/suite.yaml` defines what "working" means |

Tiers are from the catalog's `registry.yaml` at commit `9a64623`. The two policies here are local copies (`chock.lock` records `source: local`) and may be older than the catalog's versions. No agent reaches "enforced" today.

**Hook.** Try it: `git checkout main && echo x >> README.md && git add . && git commit -m "direct to main"` fails with `Direct commits/pushes to a protected branch (main|master) are blocked. Create a feature branch and open a pull request.` The commit never lands.

**Rule.** An agent reading `AGENTS.md` sees the instruction before it acts. Where its client has a pre-tool hook, the guard also refuses the flag at tool-call time. `git commit --no-verify` skips every git hook, which is why the rule exists.

**Skill.** `commit-message-style` blocks nothing. It runs when the task matches, and its eval suite is the test of whether it did that well.

A check costs no tokens: each is a script, not a model. A passing check adds nothing to the agent's context, and a refusal adds one short reason. The known classes are fixed in the agent's turn, not in review. Chock adds no new place your code goes; the agent still sends context to its own model provider. More: [`docs/README.md`](docs/README.md) and [`.agents/policies/`](.agents/policies/).

## What it stops

Only what is installed here: direct commits and pushes to `main`, and `--no-verify` at the agent's tool call. To grow from here, `chock add` the catalog policies that match how your team gets hurt, such as `scan-secrets` and `block-destructive-commands` (both enforced at commit) or `java-security`. The catalog has 71 policies: 35 enforced at commit, 11 in the agent, 25 advisory. Area table: [chock-catalog](https://github.com/open-coder-ai/chock-catalog#the-policies).

## FAQ for people and agents

**Does Chock use an LLM?** No. Each check is a deterministic script. A check costs no tokens; a refusal adds one short reason.

**Does my code leave my machine?** Chock adds no new place your code goes. The agent still sends context to its own model provider. `chock add` fetches policies from the catalog once.

**Which agents does it work with?** Any agent that reads `AGENTS.md`, plus native hooks where the client has them. Plugins cover Claude Code, Copilot, Cursor, Codex and Devin.

**How do I install it?** The repository route or the plugin route above.

**What does it cost?** Free and open source (Apache-2.0).

**Does it replace SAST or code review?** No. It refuses known classes while the agent writes, so they are fixed before review; keep SAST and review.

**Which OWASP items does it cover?** Every OWASP Agentic (ASI01–ASI10) risk has at least one catalog policy mapped to it, and every mapping is partial: [`docs/coverage.md`](https://github.com/open-coder-ai/chock-catalog/blob/main/docs/coverage.md). The two policies in this repo are not application-security gates.

## For tools and agents

- [`registry.yaml`](https://github.com/open-coder-ai/chock-catalog/blob/main/registry.yaml): every policy, its tier and eval counts
- Policy manifests, with `compliance` mappings: [`base/*/manifest.yaml`](https://github.com/open-coder-ai/chock-catalog/tree/main/base)
- [`docs/coverage.md`](https://github.com/open-coder-ai/chock-catalog/blob/main/docs/coverage.md): OWASP coverage
- Plugin `marketplace.json` in each plugin repo above
- This repo's own index: [`.agents/policies/INDEX.md`](.agents/policies/INDEX.md)
- chock.sh `/llms.txt` and `/api/index.json`: launching soon

## Contribute

Issues about the scaffold go to the [framework repo](https://github.com/open-coder-ai/chock/issues/new/choose). Policies go to the [catalog](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md). Sign commits with `git commit -s`.

## Part of open-coder-ai

| Repository | What it is |
| :--- | :--- |
| [chock](https://github.com/open-coder-ai/chock) | The framework: write a policy once, enforce it on git hooks, CI and every agent |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | The policies, each graded by what it actually enforces |
| [agentseam](https://github.com/open-coder-ai/agentseam) | One handler API over every coding agent's hooks, instruction files and plugin packaging |
| [context-report](https://github.com/open-coder-ai/context-report) | A signed report format for whether a plugin, hook, skill or `AGENTS.md` works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | A weekly, human-reviewed threat ledger scored against the catalog |
| [chock-claude-plugins](https://github.com/open-coder-ai/chock-claude-plugins) · [copilot](https://github.com/open-coder-ai/chock-copilot-plugins) · [cursor](https://github.com/open-coder-ai/chock-cursor-plugins) · [codex](https://github.com/open-coder-ai/chock-codex-plugins) · [devin](https://github.com/open-coder-ai/chock-devin-plugins) | The catalog compiled into each client's plugin format |
| [chock-quickstart](https://github.com/open-coder-ai/chock-quickstart) | The bare scaffold `chock init` leaves behind |

Apache-2.0, see [LICENSE](LICENSE).
