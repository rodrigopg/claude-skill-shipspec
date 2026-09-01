# Legacy impact — SHIP GATE closure (issue #2)

**Feature dir:** `001-shipspec-ship-gate-closure`
**Delivered:** 2026-09-01
**Scope:** `skills/shipspec/SKILL.md` only (doc/skill-definition patch, no application code).

## What changed

- Step 6 (SHIP GATE): added closure check #5 — refuse SHIP while any `actions.md` task is `[ ]` without a documented de-scope reason.
- Step 7 (Spec writeback): added sub-step #4 — advance `.reversa/active-requirements.json` to `current-stage: "shipped"` and append to `shipped-features`.
- Step 7: added guidance for multi-PR deliveries — writeback commit rides in the same PR as the last delivery commit.

## Confidence

- 🟢 CONFIRMADO — the three additions are copied near-verbatim from issue #2's proposed change (A/B/C), which the repo owner authored and offered to PR themselves.
- 🔴 LACUNA — this repo (`claude-skill-shipspec`) has no `.reversa/state.json` / `_reversa_sdd/` baseline of its own (it's the skill's *source*, not a project consuming the skill). `active-requirements.json` transition described in the new Step 7.4 cannot be exercised or verified against a real state file here. First real project that runs W1 end-to-end through a multi-PR delivery is the actual test of this change.
