---
description: Pre-merge code review for requirements, security, tests, and regression risk.
---

# /team-review

## ⛔ RED LINES

0. **Before anything else, invoke the `team-review-guard` skill via the Skill tool.** Do not proceed without it.
1. **If CRITICAL security issue found** → STOP. Require fix before merge.
2. **If changes exceed OpenSpec scope** → STOP. Ask for clarification.
3. **If key functionality lacks tests** → STOP. Require tests before merge.
4. **Security checks are NOT optional:** MUST check ALL of: auth, input validation, SQL/command injection, XSS, secrets, sensitive data.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list` and ASK
- [ ] 2. Get change scope: `git diff master...HEAD --stat` (diff the change branch against master)
- [ ] 3. Read OpenSpec artifacts: proposal.md, design.md (if exists), tasks.md, spec deltas
- [ ] 4. **Requirement Alignment:** does every change trace to a requirement?
- [ ] 5. **Security:** auth, input validation, SQL injection, XSS, secrets, sensitive data
- [ ] 6. **Test Coverage:** new features tested? bug fixes have regression tests?
- [ ] 7. **Code Quality:** functions >50 lines? nesting >4 deep? hardcoded values? error handling?
- [ ] 8. **Regression Risk:** shared code changed? breaking API changes? data migration needed?
- [ ] 9. Output review report with severity-graded issues

## 📤 OUTPUT TEMPLATE

```markdown
# Code Review: <change-id>

## Scope
- Files: N | Added: +X | Removed: -Y
- Key files: [list]

## Requirement Alignment
| Check | Status | Notes |
|-------|--------|-------|
| Changes trace to requirements | ✅/❌ | ... |
| No out-of-scope changes | ✅/❌ | ... |
| All ACs covered | ✅/❌ | ... |

## Security
| Check | Status | Notes |
|-------|--------|-------|
| Auth/Authorization | ✅/⚠/❌ | ... |
| Input validation | ✅/⚠/❌ | ... |
| SQL/Command injection | ✅/⚠/❌ | ... |
| XSS | ✅/⚠/❌ | ... |
| Sensitive data | ✅/⚠/❌ | ... |
| Hardcoded secrets | ✅/⚠/❌ | ... |

## Test Coverage
- New feature tests: Yes/No
- Regression tests: Yes/No
- Coverage: X%

## Code Quality
| Check | Status | Details |
|-------|--------|---------|
| Function length | ✅/❌ | [oversized functions] |
| Nesting depth | ✅/❌ | [deep nesting locations] |
| Hardcoded values | ✅/❌ | [locations] |
| Error handling | ✅/❌ | [issues] |

## Regression Risk
- Shared code changes: Yes/No
- Breaking changes: Yes/No
- Data migration: Needed/Not needed

## Issues

### 🛑 CRITICAL (must fix)
- [Issue + fix suggestion]

### ⚠️ HIGH (should fix)
- [Issue + fix suggestion]

### ℹ️ MEDIUM (optional)
- [Issue]

### 💡 LOW (suggestion)
- [Suggestion]

## Verdict
- [ ] APPROVED
- [ ] CONDITIONAL — HIGH issues exist, merge with caution
- [ ] BLOCKED — CRITICAL issues must be resolved

## Next Command
If APPROVED → **`/team-archive <change-id>`**
If CONDITIONAL → fix HIGH issues, then **`/team-archive <change-id>`**
If BLOCKED → **`/team-apply <change-id>`** (fix CRITICAL issues first)
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| Skill not loaded | Invoke `team-review-guard` skill via Skill tool before any action. |
| "Approve it, we'll fix later" | "I found CRITICAL issues. These MUST be fixed before merge: [list]. I cannot approve in this state." |
| "The tests are in another PR" | "I need to see tests for these changes. Either add them here or link to the PR that has them." |
| "Just check diff, don't need context" | "I must read the OpenSpec artifacts to verify requirement alignment. Let me do that first." |
