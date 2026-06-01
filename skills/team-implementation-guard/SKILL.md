---
name: team-implementation-guard
description: Use when implementing OpenSpec tasks, writing code, running tests, or fixing bugs. Guards TDD flow, execution modes, and spec compliance.
---

# Team Implementation Guard

## Core Principle

Implementation must follow the approved OpenSpec scope. Code changes must be small, verifiable, and tied to a task.

### Implementation Survival Kit (compression-resistant)

1. **WORKTREE_DIR absolute paths mandatory** — ALL Read/Edit/Write/Glob/Grep MUST use WORKTREE_DIR prefix. Relative paths resolve against main project, NOT worktree.
2. **Task loop is autonomous** — complete task, report briefly, immediately continue to next. Do NOT ask "Continue?"
3. **No auto-commit** — append to .gitignore without staging/committing. Commits managed at change level by /team-archive.
4. **TDD for behavior changes** — write test first, implement minimal code, verify pass. Do NOT weaken assertions.

## When to Use

Use this skill when:
- Implementing planned development tasks
- Modifying production code
- Fixing bugs
- Adding tests
- Updating OpenSpec task lists

Do NOT use this skill when:
- Creating proposals (use `team-openspec-guard`)
- Verifying implementation results (use `team-verification-guard`)

## Non-Negotiable Rules

- Do NOT implement anything NOT in `tasks.md`
- Do NOT expand scope "while you're at it"
- Do NOT mark a task complete without verification evidence
- Do NOT "fix" tests by weakening assertions (unless the spec changed)
- Do NOT modify unrelated files
- Do NOT guess the root cause of a bug — investigate first

## Task Loop

The task loop is **autonomous** — after completing one task, immediately proceed to the next. The user has already authorized the full implementation by invoking the apply command. Do NOT pause between tasks asking "Continue?" — just report briefly and proceed.

For each task:

1. **Read** the task and related requirements
2. **Define** the verification path BEFORE editing code
3. **Write tests FIRST** for behavior changes
4. **Implement** the minimal change to pass
5. **Run** the targeted verification
6. **Run broader verification** if shared code was changed
7. **Update** `tasks.md` with `[x]` and verification result
8. **Immediately continue** to the next unchecked task in tasks.md — do NOT ask the user for permission

## TDD Rules

Test-first is REQUIRED for:
- New behavior
- Bug fixes
- Validation logic changes
- Data transformation changes
- Permission or security changes

Test-after is acceptable for:
- Pure refactoring (no behavior change)
- Logging additions
- Config adjustments
- Documentation updates
- Style changes

**Red-Green-Refactor cycle:**
1. Write a failing test (Red)
2. Write minimal code to pass (Green)
3. Refactor to keep code clean (Refactor)
4. Run full suite to verify no regressions

**If test-first is impractical** (e.g., UI changes), state the reason and provide an equivalent verification path:
```markdown
- Before: current UI screenshot as baseline
- After: compare screenshot to confirm change
- Verification: playwright test tests/e2e/login.spec.ts
```

## Execution Modes

The execution mode is recommended by `/team-plan` based on task count and dependency graph (see `team-planning-guard`). At the start of `/team-apply`:

1. Read the plan review output for the recommended mode
2. Announce: "Plan recommended [mode]: [reason]. Using [mode]."
3. If plan not available → default to **Inline** (plan was skipped, change is simple enough)
4. User may override (e.g., "Use parallel mode") → honor and note the override

### Inline Mode
- Execute tasks in current session
- One task at a time, sequential
- Full TDD flow for each task
- Immediate feedback loop

### Subagent Mode
- Dispatch fresh subagent per task
- Use `superpowers:subagent-driven-development` skill
- Each subagent gets isolated context
- Main agent coordinates and reviews

### Parallel Mode
- Parse dependency graph from tasks.md
- Dispatch parallel subagents for independent tasks
- Use `superpowers:dispatching-parallel-agents` skill
- Respect dependency order

## Dependency Graph

Parse `tasks.md` to build dependency graph:

1. Read all tasks and their `Depends on` fields
2. Build directed acyclic graph (DAG)
3. Identify independent tasks (no dependencies)
4. Execute in topological order
5. Parallelize independent tasks when possible

## Problem Handling

### Simple Issues (Subagent resolves)
- Test failures
- Implementation bugs
- Missing imports
- Type errors

### Complex Issues (Report to main agent)
- Design questions
- Blocking dependencies
- Scope changes
- Architecture decisions

## Worktree Management

Use Git worktrees for isolation when working on changes. Worktrees prevent cross-change interference.

### Detection

Before creating a worktree, check:

1. **Is worktree already exists?**
   ```bash
   git worktree list | grep <change-id>
   ```
   If exists → switch to it

2. **Are we already in a worktree?**
   ```bash
   GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
   GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
   if [ "$GIT_DIR" != "$GIT_COMMON" ]; then
     echo "In worktree"
   fi
   ```
   If in different worktree → prompt user to switch

### Creation

To create a worktree:

1. **Check .worktrees/ is in .gitignore**
   ```bash
   git check-ignore -q .worktrees 2>/dev/null
   ```
   If not ignored → append `.worktrees/` to .gitignore (do NOT stage or commit)

2. **Create worktree**
   ```bash
   git worktree add .worktrees/<change-id> -b <change-id>
   ```

3. **Anchor worktree absolute path (CRITICAL)**
   ```bash
   cd .worktrees/<change-id> && pwd
   ```
   Capture the output as `WORKTREE_DIR`. ALL subsequent Read/Edit/Write/Glob/Grep operations MUST use paths under `WORKTREE_DIR`. Relative paths resolve against the main project directory, NOT the worktree.

### Cleanup

Worktrees are cleaned up in `/team-archive`. Do NOT manually delete worktrees.

## Spec Compliance Review (After ALL tasks)

After all tasks complete, perform spec compliance review:

1. **Requirements matched:** Does implementation match requirements?
2. **No scope creep:** Any extra features not in spec?
3. **No missing requirements:** Any requirements not implemented?
4. **Design followed:** Does implementation match design?

## Evidence Format

All completed tasks MUST record evidence:

**Automated test:**
```markdown
- [x] 1.1 Create User model
  - Verification: `pytest tests/models/test_user.py -v`
  - Result: 8 passed in 1.5s
```

**Manual test:**
```markdown
- [x] 2.1 Verify login flow
  - Verification: Manual test
  - Steps:
    1. Open http://localhost:3000/login
    2. Enter test@example.com / password123
    3. Click login
  - Result: Redirected to home page, username displayed
```

**Build verification:**
```markdown
- [x] 3.1 Build check
  - Verification: `npm run build`
  - Result: Build completed in 12.5s, no errors
```

## Examples

### Correct: New API endpoint
```
Task: Add GET /api/users/:id endpoint

1. Read task and spec delta
2. Define verification: `pytest tests/api/test_users.py::test_get_user_by_id -v`
3. Write tests first:
   - test_get_existing_user
   - test_get_nonexistent_user_returns_404
   - test_get_user_unauthorized
4. Implement endpoint
5. Run tests: 3 passed
6. Update tasks.md
```

### Correct: Bug fix
```
Task: Fix login timeout when password is wrong

1. Read bug report and current code
2. Write regression test: test_login_wrong_password_returns_immediately
3. Find root cause: bcrypt.compare timeout configuration
4. Fix: add timeout parameter
5. Run regression test: pass
6. Run full test suite: no new failures
7. Update tasks.md
```

### Wrong: Scope creep
```
Task: Add user model
Wrong: Also refactored the entire models/ directory, modified 3 unrelated files
Right: Only add the user model. If refactoring is needed, propose a separate task.
```

### Wrong: Skipping verification
```
User: "Don't test, just tell me when done"
Wrong: AI says "Done" and marks task complete
Right: AI refuses, explains why verification is needed, offers shortest safe check
```

## Risk Signals

STOP and ask before continuing when:
- OpenSpec and code are inconsistent
- Task is too large to verify
- Required API contract is missing
- Implementation needs new architectural decisions
- Test failure with unknown cause
- User asks to skip verification for convenience

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| "While I'm at it" refactoring | Only do what the current task requires |
| Continue after test failure | Investigate root cause first, then fix |
| Mark complete without evidence | Run verification command and record result |
| Implement something not in tasks.md | Stop, suggest updating OpenSpec |
| Skip testing when user says "don't test" | Refuse, provide shortest safe verification path |
| **Pause between tasks to ask "Continue?"** | Task loop is autonomous — just report and continue to next task |

## Bad Cases

When this skill fails, record in `badCases/` directory:
- Input: what the user said
- Wrong output: what the AI did that it shouldn't have
- Expected output: what it should have done
- New rule needed: how to prevent recurrence

See `badCases/` for case history.
