---
description: Archive a completed change after verification passes.
---

# /team-archive

## ⛔ RED LINES

1. **DO NOT archive if tasks are incomplete** without explicit user confirmation.
2. **DO NOT archive if `openspec validate <change-id> --strict` fails.**
3. **DO NOT manually move directories.** Use `openspec archive` command only.
4. **DO NOT hide incomplete tasks.** Report them transparently and ask for confirmation.
5. **DO NOT delete worktree without merging the branch first.** If a worktree exists, its code changes MUST be merged to the current branch before cleanup.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list` and ASK
- [ ] 2. Run `openspec list` to confirm change exists and is active
- [ ] 3. Read `tasks.md` — count incomplete tasks
- [ ] 4. If incomplete tasks exist → WARN:
  ```
  ⚠️ WARNING: X tasks remain incomplete:
  - [ ] Task N.M: <description>

  Do you still want to archive? (yes/no)
  ```
  Wait for explicit confirmation before proceeding.
- [ ] 5. Run `openspec validate <change-id> --strict`
- [ ] 6. If validation fails → STOP. Fix before archiving.
- [ ] 7. Run `openspec archive <change-id> --yes`
- [ ] 8. Run `openspec validate --strict` (global validation)
- [ ] 9. **Worktree merge & cleanup (BEFORE final report):**
  - Run: `git worktree list | grep <change-id>`
  - If no worktree → skip to step 10
  - Check for uncommitted changes: `cd .worktrees/<change-id> && git status --porcelain`
  - If has changes → STOP. Warn: "Worktree has uncommitted changes. Please commit or stash them before running /team-archive."
  - **Merge branch to current branch:**
    - Run: `git merge <change-id> --no-ff -m "merge: <change-id> implementation complete"`
    - If merge conflict → STOP. Warn: "Merge conflict detected. Resolve conflicts in the worktree first, then re-run /team-archive."
    - If no worktree existed (changes were on current branch already) → skip merge
  - ASK user: "Worktree .worktrees/<change-id>/ still exists. Delete it?"
    - Yes → `git worktree remove .worktrees/<change-id>`, verify with `git worktree list | grep <change-id>`
    - No → keep, note in report
- [ ] 10. Output final report

## 📤 OUTPUT TEMPLATE

```markdown
# Archive Complete

- Change: `<change-id>`
- Archive path: `openspec/changes/archive/<change-id>/`
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
- Merge result: ✅ Merged / ⚠ Conflicts / N/A (no worktree)

## Worktree Cleanup
- Worktree existed: Yes / No
- Uncommitted changes: Yes / No
- Cleanup: Deleted / Kept / N/A

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
| "Keep the worktree" | "OK, worktree .worktrees/<change-id>/ will be kept. Remove manually later: git worktree remove .worktrees/<change-id>" |
| "Worktree has uncommitted changes" | "Worktree has uncommitted changes. Please commit or stash them before running /team-archive." |
| Merge conflict | "Merge conflict detected. Resolve conflicts first: `cd .worktrees/<change-id>`, fix conflicts, commit, then re-run /team-archive." |
| "Don't merge, I'll do it manually" | "OK. The branch `<change-id>` still has your changes. You can merge manually: `git merge <change-id>`. Worktree will be kept." |
