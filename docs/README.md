# Chock setup

This repo uses Chock for agent policy engineering. This folder contains human-readable documentation only. Agents must not read files here.

## Structure

- `.agents/skills/` — your business skills
- `.agents/policies/` — your rules, hooks, and policies
- `docs/` — human documentation
- `AGENTS.md` — agent-readable rules

## Next steps

1. Create your first policy using the `/policy-init` skill installed in your agent's skill directory.
2. Validate with `chock check`.

## Try the enforcement

Moved here from the root `README.md`, verbatim, so the landing page can stay to one screen.

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
