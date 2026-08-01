# spec-anchored-delivery (W1)

Spec-anchored feature delivery for Claude Code: scope with [Reversa](https://github.com/sandeco/reversa), measure blast radius before writing code, deliver with a parallel agent team, then write the spec back so future Reversa runs detect drift.

**The problem it solves:** fast parallel delivery usually ends with "spec amnesia" — the code ships, the spec rots. W1 keeps Reversa's living spec as the anchor on both ends of the delivery.

## Pipeline

```
requirements → clarify? → plan → to-do → [SCOPE GATE] → deliver (agent team) → [SHIP GATE] → spec writeback → re-extract
```

- **Scope** — Reversa's forward pipeline (`/reversa-requirements` → `/reversa-plan` → `/reversa-to-do`) produces `actions.md`, an atomic, spec-anchored task list.
- **Scope gate** — blast radius is *measured* (code-graph MCP, or `Grep` fallback), not guessed. Anything the radius hits outside the plan is surfaced before a single line of code. Blocking human checkpoint.
- **Deliver** — a Claude Code agent team (coder + reviewer + devil's advocate) executes `actions.md` in parallel, each teammate anchored to its spec reference. Falls back to inline `/reversa-coding` when agent teams are unavailable.
- **Ship gate** — review against `actions.md` + spec anchors; E2E via browser MCP when there is UI.
- **Writeback** — `legacy-impact.md`, `regression-watch.md`, and `progress.jsonl` are appended so the next `/reversa` re-extraction flags drift between spec and shipped code.

Spec material (`.reversa/`, `_reversa_sdd/`, `_reversa_forward/`) is append-only — the workflow never overwrites or deletes it.

## Requirements

| Dependency | Status | Install |
|---|---|---|
| [Reversa framework](https://github.com/sandeco/reversa) | **Required** | `npx reversa install` (in the target project) |
| Agent teams (`Agent` tool) | Recommended | Built into Claude Code; falls back to inline `/reversa-coding` |
| Code-graph MCP (e.g. [jCodemunch](https://jcodemunch.com)) | Optional | Sharper scope gate; falls back to `Grep`/`Read` |
| Docs MCP (e.g. context7), browser MCP (playwright) | Optional | Live library docs during delivery, E2E at the ship gate |

The skill checks for Reversa at startup and stops with install instructions if it is missing.

## Install

```
/plugin marketplace add rodrigopg/claude-plugins
/plugin install spec-anchored-delivery@rodrigopg
```

## Usage

```
/spec-anchored-delivery <feature idea>
```

Also triggers on "W1", "deliver feature with spec", "ship but keep the spec updated".

## Credits

Original W1 workflow by **Caio**. Built on the [Reversa](https://github.com/sandeco/reversa) framework by [@sandeco](https://github.com/sandeco).

## License

MIT
