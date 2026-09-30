<div align="center">

<img src=".github/logo.svg" alt="chock-example: a working chock adoption you can read end to end — one policy per artifact layer: git hook, agent rule and skill. The mark is chock's: a wheel held by a chock wedge." width="110">

# chock-example

**A working adoption of agent security guardrails: one policy per layer (hook, rule, skill), readable end to end.**

[![security policies](https://img.shields.io/badge/security_policies-48-D9B45C?labelColor=0D1626)](https://github.com/open-coder-ai/chock-catalog)
[![eval cases](https://img.shields.io/badge/eval_cases-1%2C179-D9B45C?labelColor=0D1626)](https://github.com/open-coder-ai/chock-catalog)
[![OWASP Agentic Top 10](https://img.shields.io/badge/OWASP_Agentic_Top_10-10%2F10-D9B45C?labelColor=0D1626)](https://github.com/open-coder-ai/chock-catalog)
[![template repo](https://img.shields.io/badge/repo-template-D9B45C?labelColor=0D1626)](https://github.com/open-coder-ai/chock-example/generate)
[![license](https://img.shields.io/badge/license-Apache--2.0-D9B45C?labelColor=0D1626)](LICENSE)

[the framework →](https://github.com/open-coder-ai/chock) ·
[the full catalog →](https://github.com/open-coder-ai/chock-catalog) ·
[just the bare scaffold →](https://github.com/open-coder-ai/chock-quickstart)

</div>

> **Demo repository.** Three policies instead of the full catalog's 48, so every moving part
> fits in one sitting. Questions and issues belong on the
> [framework repo](https://github.com/open-coder-ai/chock/issues).

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/demo.gif" alt="Terminal: five security guards adopted from the catalog; a hard-coded AWS key, an MCP server at @latest, a wildcard IAM grant, model output piped into os.system and a Trojan Source bidi override are each refused at commit; the fixed file commits cleanly." width="760">
</p>

## Why guardrails

Your coding agent has a shell, your git history and your cloud credentials. Agents push
straight to `main`, skip failing hooks with `--no-verify`, and forget the rule in their prompt
once the context fills. Chock refuses the dangerous action before it lands — as a git hook, a
CI gate, or the agent's own pre-tool hook. *A rule an agent reads is advice. A hook that exits
non-zero is a control.* This repo shows both, side by side, so you can see the difference.

## Use this template

Click **Use this template** above to start your own adoption, then wire the hooks in:

```bash
git clone <your-new-repo-url> && cd <your-new-repo>
pip install chock && chock sync --repo .   # git never clones hooks — this wires them in
```

## One policy per layer

| Layer | Policy | Enforcement | Mechanism |
| :--- | :--- | :--- | :--- |
| Hook | `protect-main-branch` | Enforced at commit — blocks | A compiled gate in `.chock/compiled/protect-main-branch/git-hook/`, wired into `.git/hooks` by `chock sync`: commits, merges and pushes to `main` fail (pre-commit, pre-merge-commit, pre-push), plus a CI gate step in `ci-gate/step.yaml` |
| Rule | `block-no-verify` | Advisory, plus a best-effort in-agent guard | Rule text compiled into `.agents/policies/INDEX.md`, which `AGENTS.md` and every agent wrapper send the agent to before any work; plus a compiled pre-tool guard blocking `--no-verify` / `-n` at tool time in Claude Code, Gemini CLI and VS Code Copilot |
| Skill | `commit-message-style` | Invoked, not gated | Runs when the agent's task matches; `evals/suite.yaml` defines what "working" means |

**Hook.** Try it: `git checkout main && echo x >> README.md && git add . && git commit -m
"direct to main"` fails outright — `Direct commits/pushes to a protected branch
(main|master) are blocked. Create a feature branch and open a pull request.` The commit
never lands.

**Rule.** An agent reading `AGENTS.md` is pointed at `.agents/policies/INDEX.md` and sees the
instruction there before it acts. In Claude Code (`.claude/settings.json`), Gemini CLI
(`.gemini/settings.json`) and VS Code Copilot (`.github/hooks/chock.json`), a compiled
pre-tool guard also blocks the `--no-verify`/`-n` flag at tool-call time, so skipping the
hook isn't an option even if the agent tries it. That guard is best-effort: it fails open if
the hook itself crashes, and aliases or wrapper scripts are known bypasses.

**Skill.** `commit-message-style` doesn't block anything — it's invoked when the task
matches (writing a commit message), and `evals/suite.yaml` is the test of whether it did
that well.

Full detail per policy: [`.agents/policies/`](.agents/policies/) and
[`.agents/skills/commit-message-style/`](.agents/skills/commit-message-style/). More on how
this repo was built, and where to go next: [`docs/README.md`](docs/README.md).

## How it works

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/architecture.svg" alt="Author a policy once, compile it, and enforce it on every agent's native surface." width="820">
</p>

Everything under `.chock/compiled/` is generated from the policy sources; `chock check` fails
if it drifts. No LLM calls and no network access at enforcement time.

## Grow from here

Every area below is one `chock add <id>` away in the
[catalog](https://github.com/open-coder-ai/chock-catalog) (48 policies, 1,179 eval cases,
1,019 replayed deterministically in CI).

| Security area | Start with | What it refuses | Tier |
| :--- | :--- | :--- | :--- |
| Secrets & data leakage | `scan-secrets` | Vendor key prefixes, private-key blocks, key/token/password assignments | enforced-at-commit |
| Destructive commands | `block-destructive-commands` | `rm -rf` on absolute/home paths, `git push --force`, `reset --hard`, `terraform destroy`, `kubectl delete` | enforced-at-commit |
| Agent self-protection | `protect-agent-config`, `verify-mcp-allowlist` | An agent editing its own guardrail config; MCP servers not on your allowlist | in-agent · enforced-at-commit |
| Supply chain | `block-unpinned-agent-components`, `pin-github-actions` | MCP servers at `@latest`, `:latest` images, unpinned GitHub Actions | enforced-at-commit |
| Prompt injection | `block-invisible-unicode` | Trojan Source bidi overrides (CVE-2021-42574), Unicode tag smuggling | enforced-at-commit |
| Secure code: Java / Kotlin | `java-security` | 129 rules in 16 packs: injection, XXE, SSRF, unsafe deserialization, weak crypto | enforced-at-commit |
| Secure code: agent code | `agentic-code-security`, `block-unsafe-code-execution`, `block-wildcard-iam` | `eval`/`exec`, `shell=True`, `pickle`; `Action: *` and admin grants | enforced-at-commit |
| Test integrity | `protect-test-integrity`, `block-test-skips` | Deleted tests, net assertion loss, `assert True`, new skips | enforced-at-commit |
| Accessibility (ADA / 508 / WCAG) | `no-a11y-regression` | Retracting an accessible name an element already had | enforced-at-commit |
| OWASP Agentic Top 10 | `owasp-asi01` … `owasp-asi10` | One policy per risk; advisory text with enforced slices for ASI03–05 | advisory |

**Tiers, honestly.** `enforced-at-commit` is a hook that exits non-zero before the change
lands; `in-agent` is a best-effort pre-tool hook that fails open if it crashes; `advisory` is
text the agent reads. The top tier, `enforced`, is reached by no agent today. Guardrails, not
guarantees.

## Part of open-coder-ai

| | |
|---|---|
| [agentseam](https://github.com/open-coder-ai/agentseam) | the primitives — one handler API and a verified capability matrix across 16 agents |
| [chock](https://github.com/open-coder-ai/chock) | the compiler — one policy into git hooks, CI gates and native pre-tool hooks |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | the policies — 48, each labelled by what it actually enforces, with replayed evals |
| [context-report](https://github.com/open-coder-ai/context-report) | the evidence — a signed report of whether an agent artifact actually works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | the threat ledger the catalog's policies answer to |
| chock-{claude,cursor,copilot,codex,devin}-plugins | the catalog, packaged for each agent's plugin format (generated) |
| [chock-quickstart](https://github.com/open-coder-ai/chock-quickstart) · **chock-example** | template repos: what `chock init` leaves behind, and a full adoption |

## Contribute

This tree is the output of the framework's own commands, so fixes belong upstream. Issues go
to the [framework repo](https://github.com/open-coder-ai/chock/issues/new/choose).

| Good first contribution | Where |
| :--- | :--- |
| Found a bypass? Add an eval case that reproduces it | [chock-catalog](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md) |
| Verify an agent row in the capability matrix with a live run | [agentseam](https://github.com/open-coder-ai/agentseam/blob/main/CONTRIBUTING.md) |
| Ship a new policy with `chock new policy <id>` — the `policy wanted` entries in the [threat ledger](https://github.com/open-coder-ai/chock-threat-intel/blob/main/reference/agentic-threat-ledger.md) are the open work list | [chock-catalog](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md) |
| Add an agent adapter | [chock](https://github.com/open-coder-ai/chock/blob/main/CONTRIBUTING.md) |

Commits are signed off under the DCO (`git commit -s`). Read
[CONTRIBUTING](https://github.com/open-coder-ai/chock/blob/main/CONTRIBUTING.md) first;
report vulnerabilities privately per [SECURITY](https://github.com/open-coder-ai/chock/blob/main/SECURITY.md).

## License

Apache-2.0 — see [LICENSE](LICENSE).
