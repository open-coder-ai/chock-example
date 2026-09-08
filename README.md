<div align="center">

<img src=".github/logo.svg" alt="chock-example: a working chock adoption you can read end to end — one policy per artifact layer: git hook, agent rule and skill. The mark is chock's: a wheel held by a chock wedge." width="110">

# chock-example

**A working adoption: one policy per layer (hook, rule, skill).**

[the framework →](https://github.com/open-coder-ai/chock) ·
[the full catalog →](https://github.com/open-coder-ai/chock-catalog) ·
[just the bare scaffold →](https://github.com/open-coder-ai/chock-quickstart)

</div>

> **Demo repository.** Three policies instead of the full catalog's twenty-eight, so every
> moving part fits in one sitting. Questions and issues belong on the
> [framework repo](https://github.com/open-coder-ai/chock/issues). Click **Use this
> template** to start your own.

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/demo.gif" alt="chock's demo" width="760">
</p>

## One policy per layer

| Layer | Policy | Enforcement | Mechanism |
| :--- | :--- | :--- | :--- |
| Hook | `protect-main-branch` | Enforced — blocks | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`, wired into `.git/hooks`: commits, merges and pushes to `main` fail (pre-commit, pre-merge-commit, pre-push, plus a CI gate step) |
| Rule | `block-no-verify` | Advisory | Ambient text compiled into `AGENTS.md` and every agent wrapper, plus a compiled PreToolUse guard blocking `--no-verify` at tool time in Claude Code |
| Skill | `commit-message-style` | Invoked, not gated | Runs when the agent's task matches; `evals/suite.yaml` defines what "working" means |

Full detail per policy: [`.agents/policies/`](.agents/policies/) and
[`.agents/skills/commit-message-style/`](.agents/skills/commit-message-style/). More on how
this repo was built, and where to go next: [`docs/README.md`](docs/README.md).

## Part of open-coder-ai

| | |
|---|---|
| [agentseam](https://github.com/open-coder-ai/agentseam) | the primitives — one handler API and a verified capability matrix across 16 agents |
| [chock](https://github.com/open-coder-ai/chock) | the compiler — one policy into git hooks, CI gates and native pre-tool hooks |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | the policies — 39, each labelled enforced or advisory, with replayed evals |
| [context-report](https://github.com/open-coder-ai/context-report) | the evidence — a signed report of whether an agent artifact actually works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | the threat ledger the catalog's policies answer to |
| chock-{claude,cursor,copilot,codex}-plugins | the catalog, packaged for each agent's plugin format (generated) |
| chock-quickstart · chock-example | template repos: what `chock init` leaves behind, and a full adoption |

## License

Apache-2.0 — see [LICENSE](LICENSE).
