---
description: Retrospect on AI collaboration and feed bad cases back into skills.
---

# /team-retro

## ⛔ RED LINES

0. **Before anything else, invoke the `team-skill-maintainer` skill via the Skill tool.** Do not proceed without it.
1. **DO NOT hide or minimize failures.** Every failure is a chance to improve the skills.
2. **If you discover a systemic issue** (e.g., a skill rule is consistently ignored) → recommend pausing use of that command until the skill is updated.
3. **Every failure MUST have a suggested rule adjustment.** No "just note it" without a fix.
4. **Bad Cases go to `badCases/` directory** in the affected skill's folder — NOT inline in SKILL.md.

## 📋 EXECUTION CHECKLIST

- [ ] 1. If `$ARGUMENTS` is empty → run `openspec list --all` and ASK
- [ ] 2. Review archived artifacts: proposal.md, tasks.md, verification records
- [ ] 3. Analyze each phase: propose → plan → apply → verify → review → archive
- [ ] 4. For each phase, identify: What worked? What went wrong?
- [ ] 5. Identify failures where AI behavior deviated from skill rules
- [ ] 6. For each failure: root cause + suggested rule adjustment
- [ ] 7. Write bad cases to affected skill's `badCases/` directory using this format:
  - Path: `skills/<skill-name>/badCases/<NNN>-<short-name>.md`
  - Required fields: Trigger (exact user quote), Wrong Behavior, Expected Behavior, Root Cause, Rule Changes (specific rule additions)
  - Do NOT put case details in SKILL.md — only new rules
- [ ] 8. Update SKILL.md with new rules only (no case details)
- [ ] 9. Generate retro report

## 📤 OUTPUT TEMPLATE

```markdown
# Retrospective: <change-id>

## Overview
- Change: `<change-id>`
- Phases executed: [propose → plan → apply → verify → archive]
- Date: ...

## What Worked
- [phase]: [what went well and why]
- [skill rule]: [rule that was correctly followed]

## Bad Cases

### Bad Case 1: <short description>
- **Trigger:** User said "..." during [phase]
- **Wrong behavior:** AI did [X instead of Y]
- **Expected behavior:** AI should have done [Z]
- **Root cause:** [Why the skill rule failed to prevent this]
- **Fix:** Update [skill-name] to add rule: "[new rule]"
- **Recorded in:** `skills/<skill-name>/badCases/<NNN>-<name>.md`

### Bad Case 2: ...

## Skill Updates Needed

### team-openspec-guard
- Add: ...
- Modify: ...

### team-implementation-guard
- Add: ...
- Modify: ...

### team-verification-guard
- Add: ...
- Modify: ...

### (others as needed)

## Command Updates Needed
- `/team-xxx`: [specific change]
- New command suggestion: [if any]

## Test Scenario Updates
- [ ] New scenario: [description]
- [ ] Modify scenario: [description]

## Action Items
- [ ] Update [skill-name] SKILL.md
- [ ] Update [command-name] command file
- [ ] Add test scenario if applicable

## Done
This change is fully closed: archived AND retrospected. To start a new change, use `/team-explore` or `/team-propose`.
```

## POSTMORTEM FORMAT (reference)

Bad case files go to `skills/<skill-name>/badCases/<NNN>-<short-name>.md` with these sections:
Date, Change, Phase → Trigger (exact user quote) → Wrong Behavior → Expected Behavior → Root Cause → Rule Changes (affected skill + specific rules).

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| Skill not loaded | Invoke `team-skill-maintainer` skill via Skill tool before any action. |
| "Everything went fine" | "Let me verify by checking if any skill rules were skipped or weakened during the process." |
| "Don't need to update skills" | "Failures without skill updates will repeat. Even one new rule can prevent future failures." |
| "Just write a summary" | "A summary alone won't prevent recurrence. I need to identify specific rule adjustments." |
