# Olympus Skills

Agent skills for the [Olympus](https://github.com/evg-agentic) agentic runtime —
an operator whitepaper plus general-purpose knowledge skills. Works with any
Markdown-skill agent (Claude Code, Cursor, Codex, …).

**[中文](./README.zh-CN.md)**

## Skills

| Skill | What it does |
|---|---|
| [`olympus-operator`](./skills/olympus-operator/SKILL.md) | Operate the whole Olympus system through `@evg-olympus/cli` — the module-linkage model, an 18-point hidden-dependency checklist, and five end-to-end maintenance playbooks. The flagship skill. |
| [`image-prompt-craft`](./skills/image-prompt-craft/SKILL.md) | Photoreal image-prompt engineering with cross-shot character/scene/staging consistency. |
| [`emotional-narrative`](./skills/emotional-narrative/SKILL.md) | Convey feeling through animation — emotional beats the audience cares about. |
| [`filmmaker`](./skills/filmmaker/SKILL.md) | Cinematic sequencing + animation principles for video storytelling. |

General-purpose knowledge skills (prompt craft, narrative, filmmaking, …) land
incrementally — each is self-contained Markdown, no Olympus runtime required.

## Install

Pick one — all of them clone this repo into your agent's skills directory:

```bash
npx skills add evg-agentic/olympus-skills      # recommended, cross-agent
gh skill install evg-agentic/olympus-skills     # GitHub CLI 2.90+
```

Or inside Claude Code:

```
/plugin marketplace add evg-agentic/olympus-skills
/plugin install olympus@olympus-operator
```

## Using `olympus-operator`

The whitepaper drives the Olympus CLI over the gated Admin API:

```bash
npm install -g @evg-olympus/cli
olympus login        # browser device-flow; you must be on the operator allowlist
```

Then ask your agent to inspect, assemble, or modify the Olympus system — it
follows the linkage model and hidden-dependency checklist inside the skill.
Runnable bundles (sandbox scripts) are NOT published here; they live in the
Olympus platform and are managed via the CLI.

## Repository layout

```
skills/<name>/SKILL.md     one directory per skill (the agent loads this)
```

## License

MIT — see [LICENSE](./LICENSE).
