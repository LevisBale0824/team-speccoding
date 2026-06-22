---
name: team-repair-guard
description: Use when diagnosing and fixing bugs, applying security patches, or making small enhancements with /team-repair. Guards DIVE protocol, dual-track closure, and evidence-driven repair.
---

# Team Repair Guard

## Core Principle

Repair is diagnosis-first. Fix only after root cause is confirmed with evidence. Every repair must close both tracks: what was fixed (Repair) and what old code was retired (Retirement).

### Repair Survival Kit (compression-resistant)

1. **WORKTREE_DIR for code, PROJECT_DIR for tasks.md** — Source code operations MUST use `WORKTREE_DIR` prefix. `tasks.md` lives in `openspec/` which is NOT git-tracked. Update it via `<PROJECT_DIR>/openspec/changes/<change-id>/tasks.md`.
2. **Diagnose before fixing** — DIVE protocol is mandatory. No guessing, no trial-and-error fixes.
3. **Dual-track closure is NOT optional** — every repair must produce Repair Track + Retirement Track.
4. **No auto-commit** — commits managed by /team-archive.
5. **TDD for behavior changes** — regression test first, minimal fix, verify.

## When to Use

Use this skill when:
- Diagnosing and fixing bugs
- Applying security patches
- Making small enhancements to existing behavior
- Executing the `/team-repair` command

Do NOT use this skill when:
- Creating new features or modules (use `team-openspec-guard` with `/team-propose`)
- Verifying implementation results (use `team-verification-guard`)
- Reviewing code before merge (use `team-review-guard`)

## Non-Negotiable Rules

- Do NOT fix before confirming root cause
- Do NOT mark complete without verification evidence
- Do NOT expand scope — if the fix requires architectural changes, escalate to `/team-propose`
- Do NOT skip the Retirement Track — always check if old code should be deleted
- Do NOT "fix" tests by weakening assertions
- Do NOT guess the root cause — investigate first

## DIVE Protocol

### Phase 1: Symptom → Investigation → Root Cause → Evidence

1. **Symptom:** Record what the user observes. What fails? Exact error message? Reproduction steps?
2. **Investigation:** Read relevant code. Trace data flow. Check recent changes (`git log`, `git diff`). Use `git log --grep` to check if this symptom was "fixed" before.
3. **Root Cause:** Identify the exact line/condition causing the bug. Keep asking "why" until no deeper cause remains.
4. **Evidence:** Record reproduction output. Take a failing test or a log snippet that proves the bug exists.

### Phase 2: Patch-Shape Triage (BEFORE editing code)

Before making any code change, check the candidate fix shape against these signals:

**H-class signals (ANY hit = fix is a local patch, NOT sufficient repair):**
- **H1** — fix adds a conditional branch (`if` / `switch` / `catch` / `try`)
- **H2** — fix touches multiple sites but only one covered by failing test
- **H3** — fix is at consumer/caller, not canonical owner
- **H4** — same bug pattern exists elsewhere in repo (grep for it)
- **H5** — original reproduction still produces any anomaly
- **H6** — `git log --grep` shows this symptom was "fixed" before
- **H7** — candidate fix adds keyword/phrase/regex/sample-text exception
- **H8** — candidate fix adds fallback, adapter, compatibility branch, or legacy path expansion

**If any H-signal fires:** The fix is a mitigation, not a repair. Record the signal and proceed with the fix IF:
- The canonical owner fix is blocked by external constraints (record as T1-T4), OR
- The fix is explicitly bounded with a Retirement Track trigger

**D-class signals (ALL must pass before claiming done):**
- **D0** — fix eliminated >= 1 code path (paths after <= paths before)
- **D1** — fix eliminated >= 1 conditional branch (not added a fallback)
- **D2** — fix is at canonical owner
- **D3** — original reproduction steps no longer trigger any anomaly
- **D4** — no same-pattern occurrences remain unaddressed in repo

### Phase 3: Regression Test + Minimal Fix

1. **Write failing regression test** (TDD Red) — simplest possible reproduction
2. **Implement minimal fix** (TDD Green) — ONE change, at the canonical owner
3. **Run full test suite** — confirm no regressions
4. **3+ failed fixes** → STOP. Question the architecture. Do not attempt Fix #4.

### Phase 4: Dual-Track Closure

**Repair Track** (must answer):
1. True root cause
2. Canonical owner
3. Minimum necessary change
4. Compatibility boundary
5. Verification method

**Retirement Track** (must answer):
1. Where is the old owner/fallback/legacy path?
2. Is it still active on the main path?
3. If inactive → delete directly
4. If active → schedule retirement with trigger and observation metric
5. If deletion blocked by external dependency → record: retained object, retention reason, observation metrics, retirement timing

**Default: delete-first.** Internal code retirement defaults to deletion. Retention requires explicit justification.

## Task Loop

The task loop is **autonomous** — after completing one task, immediately proceed to the next.

For each repair task:

1. **Read** the symptom and investigation results
2. **Define** the verification path BEFORE editing code
3. **Write tests FIRST** for behavior changes
4. **Implement** the minimal change to pass
5. **Run** the targeted verification
6. **Run** full test suite to verify no regressions
7. **Update** `<PROJECT_DIR>/openspec/changes/<change-id>/tasks.md` with Repair Track + Retirement Track evidence
8. **Immediately continue** to the next task

## TDD Rules

Test-first is REQUIRED for all repair changes. Regression test must exist before the fix.

**Red-Green-Refactor cycle:**
1. Write a failing regression test (Red)
2. Write minimal code to pass (Green)
3. Run full suite to verify no regressions

## Worktree Management

Same as team-implementation-guard: create `.worktrees/<change-id>/`, anchor WORKTREE_DIR, all code operations use absolute paths under WORKTREE_DIR. Cleanup delegated to `/team-archive`.

## Evidence Format

All completed tasks MUST record evidence:

**Automated test:**
```markdown
## Repair Track
- [x] RP.1 Root cause: bcrypt.compare() had no timeout parameter
  - Evidence: reproduction log shows 30s delay on wrong password
- [x] RP.2 Regression test: `pytest tests/auth/test_login.py::test_wrong_password_timeout -v`
  - Result: FAIL (before fix) → PASS (after fix, <2s response)
- [x] RP.3 Minimal fix: auth/login.ts:53 — added timeout: 2000 parameter
- [x] RP.4 Compat boundary: no API change, same error response format

## Retirement Track
- [x] RT.1 Legacy path checked: no fallback/duplicate owner found — nothing to retire
  - Verified: grep for duplicate compare() calls → 0 results
```

## Spec Compliance Review (After ALL tasks)

- Check: does fix match root cause?
- Check: any scope creep?
- Check: any regressions introduced?

## Risk Signals

STOP and ask before continuing when:
- Root cause crosses module boundaries without clear owner
- Fix requires new architectural decisions
- 3+ fix attempts have failed
- Fix would change a published API contract
- User asks to skip verification
- H-signal fires and the canonical owner fix is within reach

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Fixing at consumer instead of owner | Trace up to canonical owner first |
| Adding fallback instead of fixing root cause | H8 signal — record and add retirement trigger |
| Skipping regression test | Write the simplest failing test before touching code |
| Marking done without Retirement Track | Always check: is there old code to delete? |
| Guessing root cause without investigation | DIVE protocol: evidence before diagnosis |
| Expanding scope to nearby refactors | Only fix the root cause; propose separately for refactors |

## Bad Cases

When this skill fails, record in `badCases/` directory:
- Input: what the user said
- Wrong output: what the AI did that it shouldn't have
- Expected output: what it should have done
- New rule needed: how to prevent recurrence

See `badCases/` for case history.
