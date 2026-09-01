# Legacy impact — SHIP GATE closure (issue #2)

**Feature dir:** `001-shipspec-ship-gate-closure`
**Delivered:** 2026-09-01
**Scope:** `skills/shipspec/SKILL.md` only (doc/skill-definition patch, no application code).

## What changed

- Step 6 (SHIP GATE): added closure check #4 — refuse SHIP while any `actions.md` task is `[ ]` without a documented de-scope reason (reason recorded in this file).
- Step 7 (Spec writeback): added sub-step #4 — advance `.reversa/active-requirements.json` to `current-stage: "shipped"` and append to `shipped-features`.
- Hard contract: carved an explicit exception for `active-requirements.json` state fields, since the pre-existing wording read as forbidding step 7.4's own in-place update.
- Step 7: added guidance for multi-PR deliveries — writeback commit rides in the same PR as the last delivery commit.

## Confidence

- 🟢 CONFIRMADO — the wording added matches issue #2's proposed change (A/B/C) in substance (the repo owner authored and offered to PR it), refined during review to resolve internal contradictions (Hard contract exception, gate ordering, undefined terms) the original proposal's prose did not address.
- 🔴 LACUNA — this repo (`claude-skill-shipspec`) has no `.reversa/state.json` / `_reversa_sdd/` baseline of its own (it's the skill's *source*, not a project consuming the skill). `active-requirements.json` transition described in the new Step 7.4 cannot be exercised or verified against a real state file here. First real project that runs W1 end-to-end through a multi-PR delivery is the actual test of this change.
