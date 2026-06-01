---
description: Incrementally implement an approved OpenSpec change with TDD and parallel execution.
---

# /team-apply

## ⛔ RED LINES — READ BEFORE EVERY TASK

0. **Before anything else, invoke the `team-implementation-guard` skill via the Skill tool.** Do not proceed without it.
1. **WORKTREE PATH IS MANDATORY:** ALL Read/Edit/Write/Glob/Grep operations MUST use `WORKTREE_DIR` absolute paths. Relative paths resolve against the main project, NOT the worktree.
2. **ONE TASK AT A TIME.** Only implement what's in tasks.md. No "while I'm here" refactors. If user says "skip testing" → REFUSE.
3. **No evidence = not done.** Run verification, record the result. DO NOT weaken test assertions to make tests pass.
4. **STOP on unknowns:** Design issue → STOP and update OpenSpec. Bug → investigate root cause first, do NOT guess-and-fix.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list` and ASK
- [ ] 1.1. Check if worktree exists for this change-id:
  - Run: `git worktree list | grep <change-id>`
  - If exists → run `cd .worktrees/<change-id> && pwd` to anchor WORKTREE_DIR, then continue to step 1.4
- [ ] 1.2. Check if currently in a worktree:
  - Run: `GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P) && GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P) && [ "$GIT_DIR" != "$GIT_COMMON" ] && echo "in-worktree" || echo "not-in-worktree"`
  - If in-worktree (different change) → ASK: "You are in another worktree. Switch to <change-id>'s worktree?"
    - Yes → switch to .worktrees/<change-id>
    - No → STOP. Finish current work first.
- [ ] 1.3. Create worktree:
  - Ensure .worktrees/ in .gitignore: `git check-ignore -q .worktrees 2>/dev/null || echo ".worktrees/" >> .gitignore` (append only, do NOT stage or commit)
  - Create: `git worktree add .worktrees/<change-id> -b <change-id>`
  - Change directory: `cd .worktrees/<change-id>`
- [ ] 1.4. Anchor worktree absolute path (CRITICAL):
  - Run: `cd .worktrees/<change-id> && pwd` → capture the absolute path as `WORKTREE_DIR`
  - ALL subsequent Read/Edit/Write/Glob/Grep operations MUST use paths under `WORKTREE_DIR`
  - Example: to read `src/app.py`, use `Read <WORKTREE_DIR>/src/app.py` (NOT `src/app.py` from main project)
  - Announce: "Working in worktree: <WORKTREE_DIR>"
  - **Do NOT use relative paths** for file tools — they resolve against the main project directory, not the worktree
- [ ] 2. Read: proposal.md, design.md (if exists), tasks.md, all spec deltas (all via WORKTREE_DIR paths)
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
- [ ] 7. For each task (ALL paths MUST be under WORKTREE_DIR):
  - [ ] 7a. Read the task and its requirements (from WORKTREE_DIR, NOT main project)
  - [ ] 7b. Define the verification path BEFORE editing code
  - [ ] 7c. Write tests FIRST (TDD)
  - [ ] 7d. Implement the minimal change to pass
  - [ ] 7e. Run the targeted verification
  - [ ] 7f. If shared code was changed → run broader verification
  - [ ] 7g. Update tasks.md: `- [x]` with verification result
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

## Worktree
- Path: `.worktrees/<change-id>/`
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
| "I'm already in a worktree" | "You are in another worktree. Switch to <change-id>'s worktree?" |
| "Don't use worktree" | "Worktrees provide isolation and prevent cross-change interference. Recommended." |
| No plan review found | Plan skipped → change is simple. Default to **Inline** mode. Note: "Plan review skipped, using Inline mode." |
