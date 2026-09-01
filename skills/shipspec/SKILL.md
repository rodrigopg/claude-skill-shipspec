---
name: shipspec
description: Ship fast, keep the spec ("W1") — spec-anchored feature delivery built on top of the Reversa framework. Scopes a feature with Reversa's forward pipeline (requirements → plan → to-do), measures blast radius before any code is written, delivers with a parallel Claude Code agent team (coder + reviewer + devil's advocate), then writes the spec and regression-watch back so future Reversa re-extractions detect drift. Use when the user says "/shipspec", "shipspec", "W1", "spec-anchored delivery", "deliver feature with spec", "ship but keep the spec updated", or wants fast parallel delivery without losing the living spec. Requires the Reversa skill collection (npx reversa install).
license: MIT
---

You are the orchestrator of **shipspec (W1)** — a hybrid between Reversa (scope + living spec) and Claude Code agent teams (parallel delivery).

Core idea: Reversa scopes the work into `actions.md`, an agent team delivers it in parallel, then the spec + `regression-watch.md` are written back so the next `/reversa` re-extraction detects drift. Deliver fast, never lose the spec.

## Hard contract

- **Spec artifacts are append-only / create-if-absent.** shipspec's contract for everything under `.reversa/`, `_reversa_sdd/`, `_reversa_forward/` (stricter than Reversa's own agents): never overwrite, never delete. Mark checkboxes, append history sections — nothing destructive. **Exception:** state-machine fields inside `.reversa/active-requirements.json` (`current-stage`, `stages-completed`, `shipped-features`) are advanced in place as part of Step 7 — that file tracks pipeline progress, not spec content, so updating its state fields is not an overwrite of spec material.
- **shipspec DOES write application source code** (steps 5–6). That is the delivery and is the one place it departs from pure Reversa. It only applies to the project's own working tree, never to `_reversa_sdd/` spec material.
- **Human checkpoints are blocking.** Same as Reversa — scope decisions wait for the user.
- **Graceful degradation.** If the `Agent` tool (agent teams) is unavailable this session, fall back to inline `/reversa-coding` — the original single-agent path. If no code-intelligence MCP is connected, fall back to `Grep`/`Read` for the scope check and say so.

## Prerequisites check (run first)

0. **Reversa installed?** Check that the Reversa skills exist (`.claude/skills/reversa*/` or `.agents/skills/reversa*/`). If absent, STOP and tell the user:

   > This workflow requires the [Reversa framework](https://github.com/sandeco/reversa). Install it with:
   > ```
   > npx reversa install
   > ```

1. Read `.reversa/state.json` → resolve `output_folder` (default `_reversa_sdd`), `forward_folder` (default `_reversa_forward`), `user_name`.
2. Detect spec baseline: does `output_folder/` hold ≥1 `.md`?
   - Yes → **legacy mode**: forward pipeline anchors decisions in the extracted specs.
   - No → **greenfield mode**: still valid, just no legacy anchor. Tell the user they can run `/reversa` first to build a baseline.
3. Detect agent teams: is the `Agent` tool (with `SendMessage` / shared task list) available this session? Record `team_capable = true|false`.
4. Detect code intelligence: is a code-graph MCP (e.g. [jCodemunch](https://jcodemunch.com)) connected? Record `code_intel = true|false`.

Report the resolved mode + tool availability to the user in one line, then proceed.

## The pipeline

```
requirements → clarify? → plan → to-do → [SCOPE GATE] → deliver(agent team | inline) → [SHIP GATE] → spec-writeback → re-extract?
```

### Step 1–4 — Scope via Reversa forward (delegated)

Run Reversa's forward skills — a deliberate subset of the full upstream chain (`/reversa-quality` and `/reversa-sync` are covered by shipspec's own gates and writeback). Do NOT reimplement them — delegate and obey their human checkpoints:

1. `/reversa-requirements <idea>` → `requirements.md` (anchored to `_reversa_sdd/` in legacy mode).
2. If `requirements.md` has `[DÚVIDA]` markers (Reversa's open-question marker) → `/reversa-clarify` (max 5 questions) before planning.
3. `/reversa-plan` → `roadmap.md`, investigation, data-delta, interfaces (delta over legacy).
4. `/reversa-to-do` → `actions.md`: atomic tasks with sequential IDs, dependencies, and parallel markers.

Output of this phase: `_reversa_forward/<NNN-feature>/actions.md`, the scoped contract the delivery team will execute.

### Step 4.5 — SCOPE GATE (blocking, the anti-creep guard)

Before any code, make the scope **measured, not guessed** — this is what keeps the delivery team from over-reaching.

For each symbol/file `actions.md` says it will touch:
- `code_intel` → use the code-graph MCP (blast radius / find references / changed symbols). Surface anything the radius hits that is NOT in `actions.md`.
- else → `Grep`/`Read` the references manually and say the estimate is unverified.

Optionally run `/reversa-audit` (cross-check actions vs requirements/roadmap) if you see inconsistencies.

Present to the user:

> `<user_name>`, scope check for **`<feature>`**:
> - Tasks: `<N>` (`<P>` parallelizable)
> - Files in scope: `<list>`
> - Blast radius hits OUTSIDE the plan: `<list or "none">`
>
> 1. **Proceed** — deliver as scoped
> 2. **Re-plan** — radius found surprises, go back to `/reversa-plan`
>
> Type CONTINUAR for 1, or REPLAN for 2.

(`CONTINUAR` is Reversa's confirmation token — keep it verbatim for consistency with the rest of the framework.)

Block until answered.

### Step 5 — Deliver

**Deliver with an agent team** via the `Agent` tool (requires `team_capable`).

1. Map `actions.md` into the shared task list. Each task → one item; respect parallel markers (independent → parallel teammates, dependent → task dependencies so blocked tasks unblock automatically). Carry each task's spec anchor (the `_reversa_sdd/` reference) into the task/spawn prompt so teammates stay anchored.
2. Spawn the team with named roles:
   - **`coder`** (lead implementer) + extra coders per independent file set — use a fast model for mechanical implementation, a stronger model for non-trivial work. Each owns distinct files (no conflicts).
   - **`reviewer`** — audits diffs against `actions.md` + spec anchors.
   - **`devil`** (devil's advocate) — adversarial; challenges the coders' assumptions and the reviewer's verdicts via `SendMessage`.
   - For risky tasks: require plan approval before teammates implement.
3. Monitor natively: shared task list + teammate messages. Steer via direct messages.
4. Use live-docs MCPs during delivery where relevant (e.g. `context7`) for current library APIs.

**If agent teams are unavailable** this session: fall back to `/reversa-coding` — the inline path that executes `actions.md` one task at a time. Same end artifacts.

As tasks complete, mark their `actions.md` checkbox `[X]` (append-safe edit, never rewrite the table structure).

### Step 6 — SHIP GATE

1. Run the quality gate: the `reviewer` teammate (or `/code-review`). Returns SHIP / FIX-FIRST.
2. On FIX-FIRST → assign fixes back to a `coder` teammate, max 2 rounds, then escalate to the user.
3. If the feature has UI/behavior: E2E via a browser MCP (`playwright` / `chrome-devtools`) before declaring SHIP.
4. If an `actions.md` exists for this feature: verify closure before declaring SHIP — every task is `[X]`. A remaining `[ ]` is either unfinished work or a checkbox marked in a discarded tree — investigate, don't ignore. A de-scoped task is also marked `[X]`, with its one-line reason recorded in `legacy-impact.md` (Step 7.1), never left `[ ]`. No `actions.md` (e.g. an inline single-file delivery with no formal plan) → this check is satisfied trivially, skip to 5.
5. Only on SHIP proceed to writeback (Step 7) — commit to doing it now, in this same PR, not as a follow-up.

### Step 7 — Spec writeback (the reason W1 exists)

This is the "update the specs for future work" half — what upstream Reversa calls convergence (`/reversa-sync`). Invoke `/reversa-coding`'s tail behavior (or do it directly if the team did the coding):

1. `legacy-impact.md` in the feature dir — what existing behavior this delivery touched, with Reversa's confidence scale 🟢 CONFIRMADO (confirmed) / 🟡 INFERIDO (inferred) / 🔴 LACUNA (gap).
2. `regression-watch.md` in the feature dir — the watch items linking the new code back to `_reversa_sdd/` specs. THIS is what a future `/reversa` re-extraction (regression check) reads to assign 🟢/🟡/🔴 drift verdicts.
3. `progress.jsonl` — append the delivery record.
4. `.reversa/active-requirements.json` — set `current-stage: "shipped"`; add any pipeline stage this delivery passed through (requirements/plan/to-do/coding) that is still missing from `stages-completed`; append the feature to `shipped-features` with `{ feature-dir, feature-id, short-name, shipped-at, pr, note }`. This is the transition the next `/reversa` reads to know the feature is done, not in-flight — see the Hard contract exception above.

Append-only. Never rewrite the main watch table; only add to its history section.

When a delivery spans multiple PRs, do the writeback in the **same PR as the last delivery commit** — not a separate docs PR you have to remember to open. A trailing `docs(reversa): writeback` commit alongside the final code change is the reliable pattern; a deferred writeback PR is exactly what gets forgotten.

Close by telling the user: delivered + spec written back — the next `/reversa` re-extraction will compare `regression-watch.md` against freshly extracted specs and flag drift; run it after the next big change to keep the spec honest.
