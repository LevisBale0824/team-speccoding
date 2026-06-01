---
description: Deep design exploration with brainstorming. Clarify requirements, explore approaches, create design.
---

# /team-explore

## ⛔ RED LINES

- **DO NOT write any implementation code.**
- **DO NOT create proposal.md, specs, or tasks.md.** Those belong to /team-propose.
- **DO NOT skip openspec environment check.**
- **DO NOT proceed if brainstorm.md would have unresolved hard blockers.**

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → ASK "What do you want to explore?"
- [ ] 2. Invoke the `team-brainstorm-guard` skill via the Skill tool. Do not proceed without it.
- [ ] 3. Read context: `openspec/project.md`, `openspec/AGENTS.md` (if exists)
- [ ] 4. Run `openspec list --specs` to understand existing capabilities
- [ ] 5. Determine change-id (kebab-case, verb-noun)
- [ ] 6. Create directory: `openspec/changes/<change-id>/`
- [ ] 7. Invoke `superpowers:brainstorming` skill via Skill tool
  - IMPORTANT: Write output to `openspec/changes/<change-id>/brainstorm.md`
  - Do NOT write to `docs/superpowers/specs/`
- [ ] 8. Brainstorming process (follow skill checklist):
  - [ ] 8a. Explore project context (files, docs, recent commits)
  - [ ] 8b. Offer visual companion (if topic involves visual questions)
  - [ ] 8c. Ask clarifying questions — one at a time
  - [ ] 8d. Propose 2-3 approaches with trade-offs and recommendation
  - [ ] 8e. Present design sections, get approval after each section
- [ ] 9. Write brainstorm.md to `openspec/changes/<change-id>/`
- [ ] 10. Spec self-review (from brainstorming skill):
  - [ ] Placeholder scan: any TBD/TODO?
  - [ ] Internal consistency: sections contradict each other?
  - [ ] Scope check: focused enough for single implementation?
  - [ ] Ambiguity check: requirements interpretable two ways?
- [ ] 11. Report and suggest next step (do NOT auto-commit brainstorm.md — let user decide)

## 📤 OUTPUT TEMPLATE

```markdown
# Exploration Complete: <topic>

- Change ID: `<change-id>`
- Directory: `openspec/changes/<change-id>/`

## Artifacts
- [x] brainstorm.md

## Design Summary
<!-- Brief summary of the agreed design -->

## Open Questions
| Q# | Question | Impact | Blocking |
|----|----------|--------|----------|
| Q1 | ... | ... | 🔴/🟡 |

## Next Command
→ **`/team-propose <change-id>`**
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| "Just start coding" | "Exploration is design only. I will NOT write code. Run /team-propose after design is complete." |
| "Skip to propose" | "Let me at least clarify the problem and check the codebase first." |
| Vague request | Ask: "What specific problem are we solving?" |
| openspec not installed | Guide to install: `npm install -g @fission-ai/openspec` |
| openspec list fails | Guide to create junction/symlink |
