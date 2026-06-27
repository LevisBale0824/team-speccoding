---
description: Archive a completed change after verification passes.
---

# /team-archive

## ⛔ RED LINES

1. **DO NOT archive if tasks are incomplete** without explicit user confirmation.
2. **DO NOT archive if `openspec validate <change-id> --strict` fails** (standard OpenSpec changes only; repair changes skip validation).
3. **DO NOT manually move directories for standard changes.** Use `openspec archive` command. Exception: repair changes (no proposal.md) use manual directory move since they have no openspec artifacts to sync.
4. **DO NOT hide incomplete tasks.** Report them transparently and ask for confirmation.
5. **DO NOT delete the change branch without merging it first.** If a branch exists, its commits MUST be merged to the base branch before deletion.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list` and ASK
- [ ] 2. Run `openspec list` to confirm change exists and is active
- [ ] 2.1. **Change type detection:** Check if `openspec/changes/<change-id>/proposal.md` exists
  - **proposal.md exists → Standard OpenSpec change.** Use `openspec archive` workflow (steps 5-8).
  - **proposal.md does NOT exist → Repair change.** Skip openspec validation/archive (steps 5-8). Use manual directory move with `mv openspec/changes/<change-id> openspec/changes/archive/<change-id>`.
- [ ] 3. Read `tasks.md` — count incomplete tasks
- [ ] 4. If incomplete tasks exist → WARN:
  ```
  ⚠️ WARNING: X tasks remain incomplete:
  - [ ] Task N.M: <description>

  Do you still want to archive? (yes/no)
  ```
  Wait for explicit confirmation before proceeding.
- [ ] 5. **Standard change only:** Run `openspec validate <change-id> --strict`
  - Repair change → skip to step 7.1
- [ ] 6. If validation fails → STOP. Fix before archiving.
- [ ] 7. **Standard change only:** Run `openspec archive <change-id> --yes`
  - Repair change → skip to step 7.1
- [ ] 7.1. **Repair change only:** Manually archive by moving directory:
  - Ensure target exists: `ls openspec/changes/<change-id>/tasks.md`
  - Create archive dir: `mkdir -p openspec/changes/archive`
  - Move: `mv openspec/changes/<change-id> openspec/changes/archive/<change-id>`
  - Verify: `ls openspec/changes/archive/<change-id>/tasks.md`
- [ ] 8. **Standard change only:** Run `openspec validate --strict` (global validation)
  - Repair change → skip
- [ ] 9. **Branch merge & cleanup (BEFORE final report):**
  - Run: `git branch --list <change-id>`
  - If no branch → skip to step 10
  - Check for uncommitted changes on the change branch: `git status --porcelain`
    - If has changes → STOP. Warn: "Branch <change-id> has uncommitted changes. Please commit or stash them before running /team-archive."
  - **Merge branch to base branch:**
    - Run: `git switch master && git merge <change-id> --no-ff -m "merge: <change-id> implementation complete"`
    - If merge conflict → STOP. Warn: "Merge conflict detected. Resolve conflicts first, then re-run /team-archive."
    - If no branch existed (changes were on current branch already) → skip merge
  - ASK user: "Branch <change-id> is merged. Delete it?"
    - Yes → `git branch -d <change-id>`
    - No → keep, note in report
- [ ] 10. Output final report

## 📤 OUTPUT TEMPLATE

### Standard OpenSpec Change

```markdown
# Archive Complete

- Change: `<change-id>`
- Type: Standard (OpenSpec)
- Archive path: `openspec/changes/archive/<change-id>/`
- Method: `openspec archive`
- Specs updated: Yes / No
- Global validation: ✅ PASS / ❌ FAIL

## Archived Artifacts
- proposal.md → archived
- design.md → archived
- tasks.md → archived
- specs/<capability>/spec.md → merged to openspec/specs/<capability>/spec.md

## Warnings
- [any issues encountered]

## Maintenance References
- Long-term spec: `openspec/specs/<capability>/spec.md`
- Change history: `openspec/changes/archive/<change-id>/`

## Branch Merge
- Source branch: `<change-id>`
- Target branch: `master`
- Merge result: ✅ Merged / ⚠ Conflicts / N/A (no branch)
- Branch deleted: Yes / No / N/A

## Next Command
Optional → **`/team-retro <change-id>`** (review the collaboration)
```

### Repair Change

```markdown
# Archive Complete (Repair)

- Change: `<change-id>`
- Type: Repair (no OpenSpec artifacts)
- Archive path: `openspec/changes/archive/<change-id>/`
- Method: Manual directory move

## Archived Artifacts
- tasks.md → archived (Repair Track + Retirement Track)

## Warnings
- [any issues encountered]

## Maintenance References
- Change history: `openspec/changes/archive/<change-id>/tasks.md`

## Branch Merge
- Source branch: `<change-id>`
- Target branch: `master`
- Merge result: ✅ Merged / ⚠ Conflicts / N/A (no branch)
- Branch deleted: Yes / No / N/A

## Next Command
Optional → **`/team-retro <change-id>`** (review the collaboration)
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| "Archive it, I'll finish later" | "⚠️ X tasks are incomplete. Are you sure? This will be recorded in the archive." |
| "Move the files manually" | "I must use `openspec archive` to ensure specs are properly synced. Let me run the command." |
| "Ignore the validation error" | "I cannot archive with validation errors. Let me check what's wrong first." |
| Archive fails | Show the error output. Suggest: "Check if the change is still active. Run `openspec list` to verify." |
| "Keep the branch" | "OK, branch <change-id> will be kept. Remove manually later: git branch -d <change-id>" |
| "Branch has uncommitted changes" | "Branch <change-id> has uncommitted changes. Please commit or stash them before running /team-archive." |
| Merge conflict | "Merge conflict detected. Resolve conflicts first, then re-run /team-archive." |
| "Don't merge, I'll do it manually" | "OK. The branch <change-id> still has your changes. You can merge manually: git merge <change-id>." |
| Repair change — no proposal.md | "This is a repair change. Using manual directory move instead of `openspec archive`. No spec sync needed." |
