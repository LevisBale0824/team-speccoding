---
description: Verify implementation against OpenSpec and team quality gates.
---

# /team-verify

## ⛔ RED LINES

0. **Before anything else, invoke the `team-verification-guard` skill via the Skill tool.** Do not proceed without it.
1. **DO NOT claim "ready" if any required task is incomplete.**
2. **DO NOT claim "ready" if `openspec validate --strict` fails.**
3. **DO NOT claim "ready" if you have not run tests** (or explained why you cannot).
4. **DO NOT infer "passing" from the absence of errors in chat.** Run the actual commands.
5. **DO NOT hide skipped checks.** List them explicitly with reasons.
6. **ALL 4 verification layers are mandatory:** (1) OpenSpec Validity → (2) Task Completion → (3) Project Checks → (4) Acceptance Criteria. Do not skip any layer.

## 🔍 PARAMETER EXTRACTION (do this FIRST)

The `change-id` does NOT arrive via `$ARGUMENTS` — OpenCode performs pure string substitution that breaks conditional logic. Find the change-id by checking, in priority order:

1. **`**User Arguments**` field** at the top of this message (most reliable)
2. **`## User Request` section** near the bottom of this message
3. **The user's raw command text** (the token immediately after the command name)

Rules:
- If a change-id is found → capture it as `<change-id>` and **DO NOT run `openspec list`**. Proceed to the EXECUTION CHECKLIST.
- If NO change-id is found → run `openspec list`, present the active changes, and ASK the user to pick one.

## 📋 EXECUTION CHECKLIST

Execute all 4 layers. Do not skip.

- [ ] 1. Use the `<change-id>` from PARAMETER EXTRACTION above
- [ ] 2. Read: proposal.md, tasks.md, design.md (if exists), spec deltas (all via PROJECT_DIR paths)
- [ ] 3. **Layer 1 — OpenSpec Validity:**
  - Run `openspec validate <change-id> --strict`
  - If FAIL → STOP. Fix before proceeding.
- [ ] 4. **Layer 2 — Task Completion:**
  - Count completed vs total tasks
  - Check each `- [x]` has evidence
  - Check deferred tasks have reasons
- [ ] 5. **Layer 3 — Project Checks:**
  - Run lint, unit tests, build, typecheck
  - Record the actual command output
- [ ] 6. **Layer 4 — Acceptance Criteria:**
  - Map every AC to a test or manual evidence
  - Mark each: ✅ / ❌ / ⚠ (missing evidence)
- [ ] 7. Output verification report using template below

## 📤 OUTPUT TEMPLATE

```markdown
# Verification Report: <change-id>

## Verdict
- Status: ✅ READY / ⚠ CONDITIONAL / ❌ NOT READY
- Reason: [if not ready]

## Layer 1: OpenSpec Validation
- Command: `openspec validate <change-id> --strict`
- Result: ✅ PASS / ❌ FAIL
- Errors: [list if any]

## Layer 2: Task Completion
- Completed: X / Y
- Deferred: Z (reasons: ...)

| Task | Status | Evidence |
|------|--------|----------|
| 1.1 | ✅ | `pytest tests/...` — 8 passed |
| 1.2 | ❌ | No evidence recorded |

## Layer 3: Project Checks
| Check | Command | Result |
|-------|---------|--------|
| Lint | `...` | ✅/❌ |
| Unit Tests | `...` | ✅/❌ |
| Build | `...` | ✅/❌ |

## Layer 4: Acceptance Criteria
| AC | Evidence | Status |
|----|----------|--------|
| AC-1: ... | `pytest tests/...` | ✅ |
| AC-2: ... | Manual test: ... | ⚠ |

## Remaining Risks
- ...

## Next Command
If ready → **`/team-review <change-id>`** (then `/team-archive <change-id>`)
If NOT ready → **`/team-apply <change-id>`** (fix remaining issues)
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| Skill not loaded | Invoke `team-verification-guard` skill via Skill tool before any action. |
| "Looks good, approve it" | "I need to run the checks first. Let me execute the verification layers." |
| "Don't need all 4 layers" | "All 4 layers are required. I can run the fastest checks first, but all must pass." |
| "Build is broken but it's not my change" | "I'll note the pre-existing build issue and verify only my changes. But I cannot claim full readiness." |
| A check cannot be run | Explain why AND provide an alternative verification path before skipping. |
