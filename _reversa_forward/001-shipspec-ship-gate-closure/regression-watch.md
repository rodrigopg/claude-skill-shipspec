# Regression watch — SHIP GATE closure (issue #2)

| Watch item | Anchor | Verdict |
|---|---|---|
| SHIP GATE must block on incomplete `actions.md` closure | `skills/shipspec/SKILL.md` Step 6, item 5 | 🟢 CONFIRMADO — text present, matches issue #2 clause B |
| Step 7 writeback advances `active-requirements.json` to `shipped` | `skills/shipspec/SKILL.md` Step 7, item 4 | 🔴 LACUNA — no consuming project's `active-requirements.json` exercised this yet; verify shape (`feature-dir`, `feature-id`, `short-name`, `shipped-at`, `pr`, `note`) against the real Reversa schema on first real use |
| Multi-PR writeback rides the final delivery PR | `skills/shipspec/SKILL.md` Step 7, closing paragraph | 🟡 INFERIDO — reasonable convention, not yet tested against an actual multi-PR W1 run |

## History

- 2026-09-01 — Initial watch entries created alongside the fix for issue #2 (rodrigopg/claude-skill-shipspec).
