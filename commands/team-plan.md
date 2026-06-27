---
description: Review and strengthen task breakdown before implementation.
---

# /team-plan

## ⛔ RED LINES

- **Before anything else, invoke the `team-planning-guard` skill via the Skill tool.** Do not proceed without it.
- **DO NOT implement anything.** This phase is review only.
- **DO NOT proceed to /team-apply** if there are unresolved hard-blocking open questions.
- **DO NOT accept tasks without verification methods.** Every task MUST have a verifiable check.

## 🔍 PARAMETER EXTRACTION (do this FIRST)

The `change-id` does NOT arrive via `$ARGUMENTS` — OpenCode performs pure string substitution that breaks conditional logic. Find the change-id by checking, in priority order:

1. **`**User Arguments**` field** at the top of this message (most reliable)
2. **`## User Request` section** near the bottom of this message
3. **The user's raw command text** (the token immediately after the command name)

Rules:
- If a change-id is found → capture it as `<change-id>` and **DO NOT run `openspec list`**. Proceed to the EXECUTION CHECKLIST.
- If NO change-id is found → run `openspec list`, present the active changes, and ASK the user to pick one.

## 📋 EXECUTION CHECKLIST

- [ ] 1. Use the `<change-id>` from PARAMETER EXTRACTION above
- [ ] 2. Read all artifacts: proposal.md, design.md (if exists), tasks.md, spec deltas
- [ ] 3. Check task granularity: each task ≤ 1 hour
- [ ] 4. Check task dependencies: no circular dependencies, correct ordering
- [ ] 5. Check verification: every task has a specific command or scenario
- [ ] 6. Check requirement linkage: every task links to a requirement
- [ ] 7. Check open questions: all hard blockers resolved?
- [ ] 8. Recommend execution mode (Inline/Subagent/Parallel) based on dependency graph
- [ ] 9. If tasks need splitting/merging → update tasks.md
- [ ] 10. Output review summary

## 📤 OUTPUT TEMPLATE

```markdown
# Plan Review: <change-id>

## Task Quality

| Check | Status | Notes |
|-------|--------|-------|
| Granularity (≤1h) | ✅/❌ | ... |
| Dependency order | ✅/❌ | ... |
| Verification methods | ✅/❌ | ... |
| Requirement linkage | ✅/❌ | ... |

## Execution Mode
**Recommended:** [Inline / Subagent / Parallel]
**Reason:** [e.g., 5 tasks, 2 independent → Parallel]

## Open Questions Status
- [x] Resolved: ...
- [ ] Still open (🟡 soft): ...

## Changes Made to tasks.md
- ...

## Verdict
- [ ] Ready
- [ ] Needs fixes before implementation (see above)

## Next Command
If ready → **`/team-apply <change-id>`**
If not ready → fix issues above, then re-run **`/team-plan <change-id>`**
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| "Tasks look fine, let's start" | "Let me verify: each task has a verifiable check and takes ≤1 hour? [re-check]" |
| Skill not loaded | Invoke `team-planning-guard` skill via Skill tool before any action. |
| "Skip the review" | "I cannot skip plan review. Bad task breakdown causes rework during implementation." |
| Task is "Implement X" (too big) | "That task is too large. Let me break it into 2-3 subtasks with clear verification steps." |
