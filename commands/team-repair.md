---
description: Diagnose, fix, and verify bugs/patches/small enhancements with dual-track closure. One command for low-complexity changes — no OpenSpec artifacts needed.
---

# /team-repair

## ⛔ RED LINES — READ BEFORE EVERY TASK

0. **Before anything else, invoke the `team-repair-guard` skill via the Skill tool.** Do not proceed without it.
1. **DIAGNOSE FIRST.** No fixes without confirmed root cause. DIVE protocol is mandatory.
2. **No evidence = not done.** Run verification, record the result. DO NOT weaken test assertions to make tests pass.
3. **Dual-track closure is mandatory.** Every repair MUST produce Repair Track + Retirement Track in tasks.md.
4. **STOP if scope exceeds repair.** If the fix requires new architecture, cross-module changes, or API contract changes → escalate to `/team-propose`.

## 🔍 PARAMETER EXTRACTION (do this FIRST)

The `description` does NOT arrive via `$ARGUMENTS` — OpenCode performs pure string substitution that breaks conditional logic. Find the description by checking, in priority order:

1. **`**User Arguments**` field** at the top of this message (most reliable)
2. **`## User Request` section** near the bottom of this message
3. **The user's raw command text** (the text after the command name)

Rules:
- If a description is found → capture it and proceed directly to the EXECUTION CHECKLIST. Do NOT re-ask the user.
- If NO description is found → ASK the user "What do you want to fix? Describe the bug or issue.".

## 📋 EXECUTION CHECKLIST

- [ ] 1. Use the `<description>` from PARAMETER EXTRACTION above
  - Capture project root: `pwd` → `PROJECT_DIR`. ALL file operations (code AND openspec) use paths under `PROJECT_DIR`.
  - Generate change-id from description: `fix-<short-slug>` (e.g., `fix-login-timeout`)
- [ ] 2. Create or switch to the change branch (isolation without worktree):
  - Run: `git switch <change-id> 2>/dev/null || git switch -c <change-id>`
  - Announce: "Working on branch: <change-id>"
- [ ] 3. **DIVE Phase 1 — Symptom:**
  - [ ] 3a. Understand the user's description of the problem
  - [ ] 3b. Search codebase for relevant files: Grep/Glob for related code
  - [ ] 3c. Read the affected source files to understand current behavior
- [ ] 4. **DIVE Phase 2 — Investigation:**
  - [ ] 4a. Reproduce the issue: write a failing test or reproduction script
  - [ ] 4b. Check recent changes: `git log --oneline -10` and `git log --grep` for related commits
  - [ ] 4c. Trace data flow from symptom to source
  - [ ] 4d. Identify the exact line/condition causing the bug
- [ ] 5. **DIVE Phase 3 — Root Cause:**
  - [ ] 5a. Confirm root cause with evidence (log output, test failure, stack trace)
  - [ ] 5b. Identify the canonical owner (which file/module should own this fix?)
  - [ ] 5c. If root cause crosses module boundaries or requires architectural changes → STOP, escalate to `/team-propose`
- [ ] 6. **DIVE Phase 4 — Evidence:**
  - [ ] 6a. Record reproduction evidence (test output, log, screenshot)
  - [ ] 6b. Check if same bug pattern exists elsewhere: `grep -r "<pattern>" src/` → record count
- [ ] 7. **Patch-Shape Triage** (BEFORE editing code):
  - Check candidate fix against H1-H8 signals (see team-repair-guard)
  - If H3 (not canonical owner) → trace up to correct owner first
  - If H8 (adds fallback) → record as bounded mitigation with Retirement Track trigger
  - Record findings: which H-signals fired, if any
- [ ] 8. **Regression Test** (TDD Red):
  - [ ] 8a. Write the simplest failing test that reproduces the bug
  - [ ] 8b. Run the test → confirm it FAILS (Red)
  - [ ] 8c. If full H4 (same pattern elsewhere) → write tests for all occurrences
- [ ] 9. **Minimal Fix** (TDD Green):
  - [ ] 9a. Implement the minimal change at the canonical owner
  - [ ] 9b. ONE change at a time — no "while I'm here" refactors
  - [ ] 9c. Run the regression test → confirm it PASSES (Green)
  - [ ] 9d. Run full test suite: `npm test` or equivalent → confirm no regressions
  - [ ] 9e. If 3+ fix attempts failed → STOP, question architecture, do not attempt Fix #4
- [ ] 10. **Dual-Track Closure** → update `<PROJECT_DIR>/openspec/changes/<change-id>/tasks.md`:
  - [ ] 10a. **Repair Track:**
    ```markdown
    ## Repair Track
    - [x] RP.1 Root cause: [description]
    - [x] RP.2 Regression test: [command + result]
    - [x] RP.3 Minimal fix: [file:line + change description]
    - [x] RP.4 Compat boundary: [verified / concern noted]
    ```
  - [ ] 10b. **Retirement Track:**
    ```markdown
    ## Retirement Track
    - [x] RT.1 Legacy path checked: [found / none]
    - [x] RT.2 [If found] Retained/Scheduled/Deleted: [decision + trigger]
    ```
  - [ ] 10c. If old code was deleted → verify no lingering references: `grep -r "<old-path>" src/`
- [ ] 11. **Patch-Shape Re-Check** (AFTER fix):
  - [ ] 11a. Count code paths before vs after → D0: paths decreased?
  - [ ] 11b. Count conditional branches → D1: branches decreased (not new fallback)?
  - [ ] 11c. Verify fix is at canonical owner → D2 satisfied?
  - [ ] 11d. Run original reproduction → D3: no anomaly?
  - [ ] 11e. Verify no same-pattern occurrences remain → D4 satisfied?
- [ ] 12. **Verification:**
  - [ ] 12a. Lint: run project linter → record result
  - [ ] 12b. Tests: run full test suite → record pass/fail + count
  - [ ] 12c. Build: run build command if applicable → record result
- [ ] 13. Report and suggest next step

## 📤 OUTPUT TEMPLATE

```markdown
# Repair Complete: <change-id>

## DIVE Summary
- Symptom: <description>
- Root Cause: <explanation>
- Canonical Owner: <file:line>
- Evidence: <test/log output>

## Patch-Shape Triage
| Signal | Status |
|--------|--------|
| H1 (new branch) | no / yes (bounded) |
| H3 (correct owner) | yes |
| H8 (new fallback) | no |

## Repair Track
| Item | Status | Detail |
|------|--------|--------|
| Root cause confirmed | ✅ | ... |
| Regression test | ✅ | `pytest ...` — X passed |
| Minimal fix | ✅ | `<file:line>` — <change> |
| Compat boundary | ✅ | <no API changes / note> |

## Retirement Track
| Item | Status | Detail |
|------|--------|--------|
| Legacy path checked | ✅ | <found / none> |
| Old code retired | ✅/⚠️ | <deleted / scheduled / n/a> |

## Verification
- Lint: ✅ PASS / ❌ FAIL
- Tests: ✅ X/Y passed / ❌ FAIL
- Build: ✅ PASS / ❌ FAIL / N/A

## Branch
- Branch: `<change-id>`

## Next Command
→ **`/team-archive <change-id>`**
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| Skill not loaded | Invoke `team-repair-guard` skill via Skill tool before any action. |
| "Skip testing / don't test" | "I cannot mark a repair complete without regression evidence. Let me write the simplest failing test first." |
| "Also fix X while you're there" | "X is outside this repair scope. I'll finish the current fix first. If X is a separate issue, run `/team-repair X` afterwards." |
| "Just mark it done" | "I need evidence. Let me run the regression test and dual-track checks first." |
| "It's not a bug, it's a feature request" | "This exceeds repair scope. Use `/team-propose <description>` for new features and architecture changes." |
| Test fails unexpectedly | "Test failed at [location]. Let me investigate the root cause before making code changes." |
| "The fix needs architectural discussion" | "This exceeds repair scope. Let me summarize findings and we'll switch to `/team-propose` for the full design pipeline." |
| 3+ fix attempts failed | "I've tried [N] fixes. Let me stop and question the architecture before attempting more." |
