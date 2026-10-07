<div align="center">

<p><img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/readme/cover-chock-example.png" alt="Chock mark on a dusk-blue background." width="100%"></p>

</div>

# Teach your AI agent what not to do.

Open-source guardrails for AI coding agents: rules the agent reads, checks that run as it writes, and gates at commit and in CI. This template is a working adoption you can read end to end: one policy per layer.

[chock](https://github.com/open-coder-ai/chock) · [chock-catalog](https://github.com/open-coder-ai/chock-catalog) · [chock-quickstart](https://github.com/open-coder-ai/chock-quickstart) · chock.sh (launching soon)

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

chock-example is a GitHub template repository: a repo where Chock is already adopted, with three artifacts installed so every moving part fits in one sitting. Chock is open-source application security for code written by AI coding agents. Each check is a deterministic local script, with no model and no upload, and it refuses known vulnerability classes before they are committed. This is a demo repository; questions and issues belong on the [framework repo](https://github.com/open-coder-ai/chock/issues).

## Application security for the code your agents write

Coding agents already ask before they run a shell command. What they do not check is the code they write: SQL injection in a Spring repository, an IAM grant on `*`, an MCP server at `@latest`, a bidi override hiding in a source file, a secret written into agent memory. Chock checks that code as the agent writes it, at commit and in CI. This repo installs two guardrails and one skill so you can see each layer work. The [catalog](https://github.com/open-coder-ai/chock-catalog) holds the application-security policies you would add next.

| Area | What gets refused | Policy | Tier |
| :--- | :--- | :--- | :--- |
| Java & Kotlin | injection, XXE, SSRF, unsafe deserialization, weak crypto, dependencies below a known fix | `java-security` | commit |
| Unsafe code, IAM | `eval`, `shell=True`, `os.system`, `pickle`; IAM `Action: *` | `block-unsafe-code-execution`, `block-wildcard-iam` | commit |
| Supply chain | dependencies off an allowlist, Actions on a mutable tag, MCP servers and images at `@latest` | `verify-dependency-exists`, `pin-github-actions`, `block-unpinned-agent-components` | commit |
| Agent code | host execution, approvals switched off, credential leaks | `agentic-code-security` | commit |
| Accessibility | a stripped `alt`, `aria-label`, label or `lang` | `no-a11y-regression` | commit |
| Prompt injection, memory | bidi and tag characters; secrets written into agent memory | `block-invisible-unicode`, `guard-memory-writes` | commit |
| Test integrity | deleted tests, lost assertions, new skips | `protect-test-integrity` | commit |
| Also included | secrets, destructive commands, agent self-protection | `scan-secrets`, `block-destructive-commands`, `protect-agent-config` | commit, in-agent |

Tiers: `commit` is a git hook or CI gate that exits non-zero. `in-agent` is the agent's pre-tool hook: best-effort, and it fails open. `advisory` is rule text the agent reads. No agent reaches `enforced` today.

## Install

chock is on PyPI, but the release there (0.15.2, 30 Sep 2026) is older than the engine this page describes. Install the frozen engine from its commit (Python 3.11 or newer):

```bash
pip install "chock @ git+https://github.com/open-coder-ai/chock@992711af4cf8d4fd9c4c861f10ef6e53374d75d7"
```

### Two ways to adopt it

1. **In your repository, for teams.** Run `chock init .`, then `chock add <id> --ref <catalog commit> --verify-sha <sha256> --skip-compile` for each policy, then `chock sync --repo . --ci`. Commit the result. Every clone runs `chock sync --repo .` once, because git never clones hooks. The commit gates are enforced at commit and in CI.
2. **In your coding agent, as plugins.** Best-effort, and they fail open. One repo per client: [Claude Code](https://github.com/open-coder-ai/chock-claude-plugins), [Cursor](https://github.com/open-coder-ai/chock-cursor-plugins), [Copilot](https://github.com/open-coder-ai/chock-copilot-plugins), [Codex](https://github.com/open-coder-ai/chock-codex-plugins), [Devin](https://github.com/open-coder-ai/chock-devin-plugins). Each README has the install line for its client.
3. **One Claude Code plugin from a selection.** The chock.sh builder (launching soon) gives a `chock install --selection '…' --apply` command.

| | In your repository (for teams) | In your coding agent, as plugins |
| :--- | :--- | :--- |
| Where it runs | Your agent's hook where its client has one, at commit, and in CI | The client's pre-tool hook only |
| Strength | The commit gates are enforced at commit and in CI | Best-effort: the client's hook fails open, and it does not run in CI |

This repo is a result of the repository route.

### Use this template

Click **Use this template**, clone your copy, then run `chock sync --repo .` once. Git never clones hooks, so every clone does this.

## How it works

### One policy per layer

| Layer | Policy | Tier in the catalog | Mechanism here |
| :--- | :--- | :--- | :--- |
| Hook | `protect-main-branch` | Enforced at commit | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`: commits, merges and pushes to `main` fail (pre-commit, pre-merge-commit, pre-push), plus a CI gate step |
| Rule | `block-no-verify` | Best-effort (in the agent, fails open) | Ambient text in `AGENTS.md` and every agent wrapper, plus a compiled pre-tool guard that refuses `--no-verify` and `-n` where the client has a hook: wired here for Claude Code, Gemini CLI and VS Code Copilot |
| Skill | `commit-message-style` | Invoked, not gated | Runs when the agent's task matches; its `evals/suite.yaml` defines what "working" means |

Tiers are from the catalog's `registry.yaml` at commit `f25f5a3` (policies unchanged since `9a64623`). The two policies here are local copies (`chock.lock` records `source: local`) and may be older than the catalog's versions. No agent reaches "enforced" today.

The recording below shows chock refusing unsafe commits and accepting the fixed ones. This repo installs only the policies in the table.

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/demo.gif" alt="Terminal recording of chock refusing unsafe commits and accepting the fixed ones." width="760">
</p>

**Hook.** Try it: `git checkout main && echo x >> README.md && git add . && git commit -m "direct to main"` fails with `Direct commits/pushes to a protected branch (main|master) are blocked. Create a feature branch and open a pull request.` The commit never lands.

**Rule.** An agent reading `AGENTS.md` sees the instruction before it acts. Where its client has a pre-tool hook, the guard also refuses the flag at tool-call time. `git commit --no-verify` skips every git hook, which is why the rule exists.

**Skill.** `commit-message-style` blocks nothing. It runs when the task matches, and its eval suite is the test of whether it did that well.

A check costs no tokens: each is a script, not a model. A passing check adds nothing to the agent's context, and a refusal adds one short reason. The known classes are fixed in the agent's turn, not in review. Chock adds no new place your code goes; the agent still sends context to its own model provider. More: [`docs/README.md`](docs/README.md) and [`.agents/policies/`](.agents/policies/).

## What it stops

Only what is installed here: direct commits and pushes to `main`, and `--no-verify` at the agent's tool call. To grow from here, `chock add` the catalog policies that match how your team gets hurt, such as `scan-secrets` and `block-destructive-commands` (both enforced at commit) or `java-security`. The catalog has 71 policies: 35 enforced at commit, 11 in the agent, 25 advisory (`registry.yaml` at `f25f5a3`). Area table: [chock-catalog](https://github.com/open-coder-ai/chock-catalog#the-policies).

## Guardrails, not guarantees

Tiers: `commit` is a git hook or CI gate that exits non-zero. `in-agent` is the agent's pre-tool hook: best-effort, and it fails open. `advisory` is rule text the agent reads. No agent reaches `enforced` today. OWASP mappings are partial and the engine is frozen at the commit above. Chock does not stop every attack: it closes common, known entry points before they ship.

## FAQ for people and agents

**Does Chock use an LLM?** No. Each check is a deterministic script. A check costs no tokens; a refusal adds one short reason to the agent's context.

**Does my code leave my machine?** Chock adds no new place your code goes. The agent still sends context to its own model provider. `chock add` fetches policies from the catalog once.

**Which agents does it work with?** Any agent that reads `AGENTS.md`, plus native hooks where the client has them. Plugins cover Claude Code, Copilot, Cursor, Codex and Devin.

**How do I install it?** The repository route or the plugin route, both in [Install](#install).

**What does it cost?** Free and open source (Apache-2.0).

**Does it replace SAST or code review?** No. It refuses known classes while the agent writes, so they are fixed before review; keep SAST and review.

**Which OWASP items does it cover?** Every OWASP Agentic (ASI01–ASI10) risk has at least one catalog policy mapped to it, and every mapping is partial: [`docs/coverage.md`](https://github.com/open-coder-ai/chock-catalog/blob/main/docs/coverage.md). The two policies in this repo are not application-security gates.

## For tools and agents

Machine-readable sources:
- [`registry.yaml`](https://github.com/open-coder-ai/chock-catalog/blob/main/registry.yaml): every policy, its tier and eval counts
- Policy manifests, with `compliance` mappings: [`base/*/manifest.yaml`](https://github.com/open-coder-ai/chock-catalog/tree/main/base)
- [`docs/coverage.md`](https://github.com/open-coder-ai/chock-catalog/blob/main/docs/coverage.md): OWASP coverage
- Plugin `marketplace.json` in each plugin repo above
- This repo's own index: [`.agents/policies/INDEX.md`](.agents/policies/INDEX.md)
- chock.sh `/llms.txt` and `/api/index.json`: launching soon

## Part of open-coder-ai

The 13 public repositories:

| Repository | What it is |
| :--- | :--- |
| [agentseam](https://github.com/open-coder-ai/agentseam) | Core: One handler API over every coding agent. |
| [chock](https://github.com/open-coder-ai/chock) | Core: Author a policy once, enforce it on every agent. |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | Policies: The policies, each labelled by what it enforces, with replayed evals. |
| [context-report](https://github.com/open-coder-ai/context-report) | Evidence: A signed report of whether an agent artifact works. |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | Evidence: A weekly threat ledger, each entry scored against the catalog. |
| [chock-claude-plugins](https://github.com/open-coder-ai/chock-claude-plugins) | Plugins: The catalog as Claude Code plugins (generated). |
| [chock-copilot-plugins](https://github.com/open-coder-ai/chock-copilot-plugins) | Plugins: The catalog as Copilot CLI and VS Code plugins (generated). |
| [chock-cursor-plugins](https://github.com/open-coder-ai/chock-cursor-plugins) | Plugins: The catalog as Cursor plugins (generated). |
| [chock-codex-plugins](https://github.com/open-coder-ai/chock-codex-plugins) | Plugins: The catalog as Codex plugins (generated). |
| [chock-devin-plugins](https://github.com/open-coder-ai/chock-devin-plugins) | Plugins: The catalog as Devin plugins (generated). |
| [chock-quickstart](https://github.com/open-coder-ai/chock-quickstart) | Template: What chock init leaves behind. |
| [chock-example](https://github.com/open-coder-ai/chock-example) | Template: A working adoption, one policy per layer. |
| [.github](https://github.com/open-coder-ai/.github) | Community: Org profile and community health files. |

## Contributing

Issues about the scaffold go to the [framework repo](https://github.com/open-coder-ai/chock/issues/new/choose). Policies go to the [catalog](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md). Sign commits with `git commit -s`.

Apache-2.0, see [LICENSE](LICENSE).

