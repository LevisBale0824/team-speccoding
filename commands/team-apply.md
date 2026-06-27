---
description: Incrementally implement an approved OpenSpec change with TDD and parallel execution.
---

# /team-apply

## ⛔ RED LINES — READ BEFORE EVERY TASK

0. **Before anything else, invoke the `team-implementation-guard` skill via the Skill tool.** Do not proceed without it.
1. **ONE TASK AT A TIME.** Only implement what's in tasks.md. No "while I'm here" refactors. If user says "skip testing" → REFUSE.
2. **No evidence = not done.** Run verification, record the result. DO NOT weaken test assertions to make tests pass.
3. **STOP on unknowns:** Design issue → STOP and update OpenSpec. Bug → investigate root cause first, do NOT guess-and-fix.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list` and ASK
  - Capture project root: `pwd` → `PROJECT_DIR`. ALL file operations (code AND openspec) use paths under `PROJECT_DIR`.
- [ ] 1.1. Create or switch to the change branch (isolation without worktree):
  - Run: `git switch <change-id> 2>/dev/null || git switch -c <change-id>`
  - Announce: "Working on branch: <change-id>"
- [ ] 2. Read: proposal.md, design.md (if exists), tasks.md, all spec deltas (all via PROJECT_DIR paths)
- [ ] 3. Invoke `superpowers:test-driven-development` skill via Skill tool
- [ ] 4. Parse tasks.md → build dependency graph
- [ ] 5. Read execution mode from plan review (see plan output):
  - Announce: "Plan recommended [mode]: [reason]. Using [mode]."
  - If plan not available → fall back to Inline, note "Plan review skipped, defaulting to Inline"
  - If user overrides mode → honor it and note the override
- [ ] 6. Execute tasks based on mode:
  - **Inline:** Execute one by one with TDD flow
  - **Subagent:** Dispatch subagent per task (use `superpowers:subagent-driven-development`)
  - **Parallel:** Dispatch parallel subagents for independent tasks (use `superpowers:dispatching-parallel-agents`)
- [ ] 7. For each task:
  - [ ] 7a. Read the task and its requirements
  - [ ] 7b. Define the verification path BEFORE editing code
  - [ ] 7c. Write tests FIRST (TDD)
  - [ ] 7d. Implement the minimal change to pass
  - [ ] 7e. Run the targeted verification
  - [ ] 7f. If shared code was changed → run broader verification
  - [ ] 7g. Update tasks.md: `Edit <PROJECT_DIR>/openspec/changes/<change-id>/tasks.md` → mark `- [x]` with verification result
- [ ] 8. After ALL tasks complete → Spec Compliance Review:
  - [ ] 8a. Check: does implementation match requirements?
  - [ ] 8b. Check: any extra features (scope creep)?
  - [ ] 8c. Check: any missing requirements?
  - [ ] 8d. Check: does implementation match design?
- [ ] 9. Report and suggest next step

## 📤 OUTPUT TEMPLATE

```markdown
# Implementation Complete: <change-id>

## Execution Summary
- Mode: Inline / Subagent / Parallel
- Tasks completed: X / Y
- TDD: Yes (tests written first)

## Task Results
| Task | Status | Verification | Evidence |
|------|--------|--------------|----------|
| 1.1 | ✅ | `pytest tests/...` | 8 passed |
| 1.2 | ✅ | `npm test` | All green |

## Spec Compliance Review
| Check | Status | Notes |
|-------|--------|-------|
| Requirements matched | ✅/❌ | ... |
| No scope creep | ✅/❌ | ... |
| No missing requirements | ✅/❌ | ... |
| Design followed | ✅/❌ | ... |

## Branch
- Branch: `<change-id>`

## Next Command
→ **`/team-verify <change-id>`**
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| Skill not loaded | Invoke `team-implementation-guard` skill via Skill tool before any action. |
| "Skip testing / don't test" | "I cannot mark work complete without verification. The shortest safe check I can run is: [suggestion]. Should I proceed?" |
| "Also fix X while you're there" | "X is not in tasks.md. I'll finish the current task first. If X is needed, update the OpenSpec tasks." |
| "Just mark it done" | "I need evidence. Let me run the verification command first." |
| Test fails unexpectedly | "Test failed at [location]. Let me investigate the root cause before making code changes." |
| "Use subagent mode" | "Switching to subagent mode. Each task will be dispatched to a fresh subagent." |
| "Use parallel mode" | "Switching to parallel mode. Independent tasks will run in parallel." |
| No plan review found | Plan skipped → change is simple. Default to **Inline** mode. Note: "Plan review skipped, using Inline mode." |
