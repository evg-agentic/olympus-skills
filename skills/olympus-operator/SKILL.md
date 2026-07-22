---
name: olympus-operator
description: Drive the @evg-olympus/cli to inspect, assemble, and modify the Olympus system — knowing how the modules LINK, so a change in one triggers the right changes in others. Use when the task touches skills, agents, models, specialists, channels, integrations, MCP, operators, or approvals.
---

# Olympus Operator

You operate the Olympus system through the `@evg-olympus/cli` (`olympus …`) over
the gated Admin API. Your value is not running commands — it is knowing what a
change in one module **forces** in others, and saying plainly when a need cannot
be met without an administrator.

## Setup

```bash
npm install -g @evg-olympus/cli
olympus login          # browser device-flow; must be on the operator allowlist
```

Every write command is two-step (preview → confirm), supports `--dry-run` and
`-y`, and is gated by the operator allowlist on the backend — you never decide
access. Config is DB-resident (ADR-0017); after import-level changes the
in-memory snapshot must be reloaded.

## The mental model — AgentProfile is the hub

A **Primary Agent** is a profile that references almost every other module:

```
AgentProfile
  ├─ brain.models        → model catalog (ModelEntry → credential_ref → vault)
  ├─ tool_groups         → tool catalog (run_skill / exec / skills_read / …)
  ├─ skills              → skill catalog (knowledge OR runnable bundles)
  ├─ integrations        → central credential groups (ADR-0018) — only with exec
  ├─ mcp_servers         → MCP registry (global enabled ∩ per-agent opt-in)
  ├─ a2a.allowed_agents  → specialists (A2A tool-agents)
  ├─ rules               → shared/private soul rules
  └─ data_isolation / shared_memory / confirmation
channels (Lark bots) bind app_id → agent_id (connect only on gateway restart)
```

The backend `_validate` cross-checks every reference on save and collects a
single 422 — so an invalid combination is refused, but the **runtime** gates
below are NOT checked at save time. You must check them yourself.

## Hidden dependencies — verify before you claim "done"

These are the things the save will NOT catch. Each one is a "changed A, must
also do B" rule:

| # | Rule | If missed |
|---|---|---|
| H1 | A runnable skill is unusable until `olympus approve <skill>` | agent sees it but `run_skill` silently no-ops |
| H2 | An agent with a runnable skill must hold a group containing `run_skill` (e.g. `code`) | save 422 "skills.runnable" |
| H3 | `exec` group is off by default; granting `integrations` requires exec | credentials granted but can't run |
| H4 | A newly registered Lark bot connects **only on gateway restart** | "bot doesn't respond" right after register |
| H5 | A model's credential lives in the vault (`set-credential`), not the catalog | route selects it → 401/no key |
| H6 | GPT/Codex use a global OAuth session (`codex.session`); Claude is per-user OAuth | new GPT model can't auth |
| H7 | MCP needs BOTH global enabled AND listed in the agent's mcp_servers | server configured but agent sees no tools |
| H8 | Remote MCP url/token are secrets; rotated values need a reload | old connection keeps using old token |
| H9 | Libraries need the `library_read` tool (in the `archive` group) | bundle materialized but unreadable |
| H10 | DB edits need a snapshot reload to take effect at runtime | "changed it but nothing happened" |
| H11 | YAML is only a bootstrap seed; editing it changes nothing live | edit file, rebuild, no effect |
| H12 | A specialist with `model: None` inherits the caller's route + per-user OAuth | image specialist runs on a non-image model |
| H13 | An integration is "maintained once, injected many places": skill declares `uses`, agent grants via `integrations` | missing embedding key → mid-run break |
| H14 | File credentials (e.g. google-sa.json) are declared as `files:` and materialized to a path at exec | tool can't find the credential file |
| H15 | A tracker's methodology must be an existing skill or saved workflow | create-tracker fails |
| H16 | Autonomous tracker Runs pre-authorize the risk gate; supervised ones stall unattended | background high-risk tools never execute |
| H17 | data_isolation / shared_memory / attachment_policy default to strictest | new agent: PERSON isolation, no auto-ingest, no sharing |
| H18 | Approving a paused runtime call does NOT resume the turn | operator thinks it continues seamlessly; must re-send |

## Playbooks (end-to-end flows, each with its H checkpoints)

### Publish a runnable skill
`skills lint <dir>` → `skills push <dir>` → **H1** `approve <skill>` → set its
integration secrets (`integrations set-secret`) **H13/H14** → `agent set-skills
<id> <skill>` (backend enforces **H2**) → reload snapshot **H10**.

Knowledge-only skill: skip approve/secrets; just push + set-skills.

### Assemble a new agent
`agent new <id> --model <ref> --tools <groups> --skills <skills>` →
**H5** model credential must be set → verify every runnable skill is approved
(**H1**) and a `run_skill` group is granted (**H2**) → optional: integrations
(need exec, **H3**), mcp (two-layer, **H7**), specialists (**H12**) → `channels
set <app-id> --agent <id>` (**H4** restart) → reload (**H10**).

### Onboard a new model
`models upsert <ref> …` → `models set-credential <cred-ref> --value …` (**H5**;
GPT/Codex use the global session, **H6**) → it's now selectable by any agent
whose `brain.models` lists it → reload (**H10**).

### Create a specialist
`specialists options` (pick model/mode) → write a payload JSON → `specialists new
<id> --file <payload.json>` (**H12**: set `model` explicitly for capability-bound
work like images) → add to a primary agent: `agent set-…` (a2a.allowed_agents).

### Add an operator
`operators add <email> --name <…>` → the allowlist is the single auth source;
their `olympus login` token is gated by it (non-operators get 403). You cannot
deactivate/remove yourself.

## Command map

| Region | Commands |
|---|---|
| auth | `login` `whoami` `logout` |
| skills | `list` `show` `lint` `pack` `push` `new` `delete` `file-get` `file-put` `file-rm` |
| capability trust | `approve <skill> [--standing]` `revoke <skill>` `capabilities <skill>` |
| runtime approvals | `approvals list` `approvals decide <id>` |
| agents | `list` `show` `new` `set-skills` `add-skill` `set-tools` `set-model` `archive` `unarchive` `duplicate` `rules` |
| specialists (a2a) | `list` `show` `options` `new` `update` `delete` |
| models (brain) | `list` `upsert` `delete` `set-credential` |
| channels (lark) | `list` `set` `delete` |
| integrations | `list` `set` `delete` `set-secret` |
| tools | `list` |
| mcp | `list` `test` |
| operators | `list` `add` `activate` `deactivate` `remove` |

Conversations, knowledge, memory, and collections are **out of scope** (privacy)
— no CLI surface for them.

## Iron rules

- **Never claim a module exists without seeing it in the live catalog this
  session** (`list`/`show`). Treat unproven as non-existent.
- **Never assemble a combination you have not validated against the H table.**
- **When a need cannot be met, say so explicitly** — "this requires an
  administrator to set up X first" — never soften an impossibility into a maybe.
- **Production writes are conservative**: confirm twice, never mass-overwrite;
  prefer `--dry-run` first.
- **Every change is auditable and gated**; you are the selector/validator, not
  the authorization.

This SKILL.md is self-contained. Per-module deep-dives (field semantics, ADR
references) land incrementally as the system evolves.
