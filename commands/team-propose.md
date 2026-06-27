---
description: Create an OpenSpec change proposal with team guardrails.
---

# /team-propose

## ⛔ RED LINES — READ FIRST

These are non-negotiable. You MUST comply.

0. **Before anything else, invoke the `team-openspec-guard` skill via the Skill tool.** Do not proceed without it.
1. **DO NOT implement any code.** This phase is planning only.
2. **DO NOT skip `openspec validate --strict`.** If it fails, fix the artifacts before reporting done.
3. **DO NOT proceed if hard-blocking open questions exist** (API fields, permissions, data migration, payments, security, UX branching, backward compatibility).
4. **If change-id already exists** → ASK whether to continue or create a new one.
5. **STOP after creating artifacts.** Wait for human review before implementation.
6. **If `openspec list` fails** → STOP. The openspec directory is not accessible. Remind the user: "Run `openspec init --tools none .` first to initialize OpenSpec in the project root."
7. **Spec format is validation-critical.** These rules MUST be followed or `openspec validate --strict` WILL fail:
   - Requirement text MUST contain `SHALL` or `MUST` (not "should", not "will")
   - Headers are EXACT: `## ADDED Requirements` / `### Requirement: <name>` / `#### Scenario: <name>` (the word "Requirement:" is mandatory)
   - Scenario body uses ONLY: `- **WHEN**` / `- **THEN**` / `- **AND**` (not "GIVEN/WHEN/THEN", not "IF/THEN")

## 🔍 PARAMETER EXTRACTION (do this FIRST)

The `description` does NOT arrive via `$ARGUMENTS` — OpenCode performs pure string substitution that breaks conditional logic. Find the description by checking, in priority order:

1. **`**User Arguments**` field** at the top of this message (most reliable)
2. **`## User Request` section** near the bottom of this message
3. **The user's raw command text** (the text after the command name)

Rules:
- If a description is found → capture it and proceed directly to the EXECUTION CHECKLIST. Do NOT re-ask the user.
- If NO description is found → ASK the user "What change do you want to propose?".

## 📋 EXECUTION CHECKLIST

- [ ] 1. Run `openspec --version` — if command not found → STOP. Tell user: "OpenSpec CLI is not installed. Run `npm install -g @fission-ai/openspec` first, then verify with `openspec --version`."
- [ ] 2. Use the `<description>` from PARAMETER EXTRACTION above
- [ ] 2.5. **Complexity routing check** — analyze the user's description against the routing matrix:
  - **Low complexity signals** (bug fix, crash, error, regression, security patch, small UI tweak, config typo, single-sentence description) →
    → Suggest redirect: "This appears to be a low-complexity change. `/team-repair` is designed for this — it will diagnose, fix, verify, and handle dual-track closure in one command without creating proposal, spec delta, or design artifacts. Suggest switching to `/team-repair <description>`. Proceed with `/team-propose` anyway? (yes/no)"
    - If user says no → STOP. User will invoke `/team-repair` separately.
    - If user says yes → continue with full propose.
  - **Medium complexity signals** (single-module feature, small refactor, adding a field/endpoint, one-file change) →
    → Note: "Medium complexity detected. I will skip spec delta and design.md unless architecture decisions are needed. Flow: propose → apply → verify → archive (4 commands, skip plan/review)."
  - **High complexity signals** (cross-module, API contracts, architecture change, data migration, auth/permission changes) →
    → Note: "High complexity detected. Using full pipeline: propose → plan → apply → verify → review → archive."
- [ ] 3. Run `openspec list` — if it fails → STOP (see RED LINE #6). This validates the junction/symlink and init were done.
- [ ] 3.1. Read context: `openspec/project.md`, `openspec/AGENTS.md` (if exists)
- [ ] 3.2. Check for existing brainstorm.md: `openspec/changes/<change-id>/brainstorm.md`
  - If exists → Read brainstorm.md as context for proposal generation
  - If not exists → Continue without it (user may have skipped /team-explore)
- [ ] 4. Run `openspec list --specs`
- [ ] 5. Determine change-id in kebab-case (verb-noun, e.g., `add-user-auth`)
- [ ] 6. Create directory: `openspec/changes/<change-id>/`
- [ ] 7. Generate proposal.md using EXACTLY the format below
- [ ] 8. Generate spec deltas using EXACTLY the format below
  - **Medium complexity:** Skip spec deltas. Note: "spec delta not needed: single-module change, no API contract or cross-module impact."
- [ ] 9. Generate design.md — REQUIRED when ANY of these conditions apply:
  - New components, modules, or services created
  - Multiple implementation approaches exist (need to document trade-offs)
  - Data schema or API contract changes
  - Cross-subsystem changes (affects > 1 module)
  - Migration or backward compatibility concerns
  - Non-trivial error handling or security decisions
  - If NONE of these apply (simple change: add field, fix bug, update config) → skip design.md and note "design.md not needed: simple change, no architectural decisions"
  - **Medium complexity:** Skip design.md automatically. Note: "design.md not needed: medium complexity, no architectural decisions."
  - If brainstorm.md exists from `/team-explore` → use it as input, formalize the decisions into design.md
- [ ] 10. Invoke `superpowers:writing-plans` skill via Skill tool
  - **Medium complexity:** Skip plan.md/tasks.md generation via writing-plans. Generate a simple tasks.md directly with 3-5 tasks.
  - Write output to `openspec/changes/<change-id>/plan.md`
  - Do NOT write to `docs/superpowers/plans/`
  - **Team workflow commit override:** After writing-plans generates the plan, REMOVE all per-task "Commit" steps (e.g., `Step N: Commit` with `git add/commit`). In team workflow, commits are managed at the change level by `/team-apply`, NOT per-task. Keep TDD steps (write test → verify fail → implement → verify pass), only remove commit steps.
  - **Add working directory header** after the plan header (Goal/Architecture/Tech Stack):
    ```
    > **Working Directory:** All file paths in this plan are relative to the project root (PROJECT_DIR).
    ```
- [ ] 11. Extract tasks.md from plan.md (after commit steps removed):
  - Each `### Task N: <name>` → `## N. <name>`
  - Each `- [ ] Step N: <description>` → `- [ ] N.M <description>`
  - **Prepend tasks.md header:**
    ```
    > **Working Directory:** All file paths below are relative to the project root (PROJECT_DIR).
    ```
  - Write to `openspec/changes/<change-id>/tasks.md`
  - **Medium complexity:** If plan.md was skipped, ensure tasks.md was created directly in step 10.
- [ ] 12. Run `openspec validate <change-id> --strict`
  - **Medium complexity (no spec deltas):** Skip validation — nothing to validate. Note: "Validation skipped: no spec deltas to validate."
- [ ] 13. If validation fails → FIX first, then re-validate
- [ ] 14. Report and STOP for human review

## 📐 FORMAT REQUIREMENTS — DEVIATION = VALIDATION FAILURE

### proposal.md MUST use these headers (English only):

```markdown
## Why
<!-- 1-2 sentences. Minimum 50 characters. -->

## What Changes
<!-- Bullet list of specific changes. Mark breaking changes with **BREAKING**. -->

## Capabilities
### New Capabilities
- `<name>`: <brief description>  <!-- kebab-case -->

### Modified Capabilities
- `<existing-name>`: <what requirement is changing>

## Impact
<!-- Affected code, APIs, dependencies, systems -->
```

### spec delta (specs/<capability>/spec.md) MUST use this format:

```markdown
## ADDED Requirements

### Requirement: <name>
<description MUST contain SHALL or MUST>

#### Scenario: <name>
- **WHEN** <condition>
- **THEN** <expected outcome>

#### Scenario: <name>
- **WHEN** <condition>
- **THEN** <expected outcome>
```

**Critical validation rules:**
- Section headers: `## ADDED Requirements` / `## MODIFIED Requirements` / `## REMOVED Requirements` / `## RENAMED Requirements`
- Requirement header: `### Requirement: <name>` — the word "Requirement:" is mandatory
- Requirement text MUST contain `SHALL` or `MUST`
- Scenario header: `#### Scenario: <name>` — exactly 4 hashtags
- Scenario body: `- **WHEN**` / `- **THEN**` / `- **AND**`
- Every ADDED/MODIFIED requirement MUST have at least one scenario

### design.md format (when needed):

```markdown
# Design: <change-id>

## Context
<!-- Background, current state, constraints, stakeholders -->

## Goals / Non-Goals
**Goals:** ...
**Non-Goals:** ...

## Decisions
### Decision 1: [Title]
- Background: [why needed]
- Alternatives considered: [options]
- Choice: [what was chosen]
- Rationale: [why]

## Risks / Trade-offs
| Risk | Mitigation |
|------|------------|
| ... | ... |

## Migration Plan
<!-- Steps to deploy, rollback strategy -->

## Open Questions
<!-- Outstanding decisions or unknowns -->
```

## 📤 OUTPUT TEMPLATE

### High Complexity Output

```markdown
# Proposal Created (High Complexity)

- Change ID: `<change-id>`
- Directory: `openspec/changes/<change-id>/`

## Artifacts
- [x] proposal.md
- [x] design.md
- [x] plan.md (via superpowers:writing-plans)
- [x] tasks.md (extracted from plan.md)
- [x] specs/<capability>/spec.md (N deltas)

## Validation
`openspec validate <change-id> --strict` → PASS

## Open Questions
| Q# | Question | Impact | Blocking |
|----|----------|--------|----------|
| Q1 | ... | ... | 🔴/🟡 |

## Next Command
Human review required. After approval:
→ **`/team-plan <change-id>`** (review task breakdown)
→ Then **`/team-apply <change-id>`**
```

### Medium Complexity Output

```markdown
# Proposal Created (Medium Complexity)

- Change ID: `<change-id>`
- Directory: `openspec/changes/<change-id>/`

## Artifacts
- [x] proposal.md
- [x] tasks.md (simplified, 3-5 tasks)
- [ ] design.md — skipped (medium complexity, no architectural decisions)
- [ ] spec deltas — skipped (single-module change)
- [ ] plan.md — skipped (simplified flow)

## Validation
Skipped — no spec deltas to validate.

## Open Questions
| Q# | Question | Impact | Blocking |
|----|----------|--------|----------|
| Q1 | ... | ... | 🔴/🟡 |

## Next Command
Human review required. After approval:
→ **`/team-apply <change-id>`** (plan/review are skipped for medium complexity)
```

## IF-THIS-THEN-THAT

| User says... | You MUST respond... |
|---|---|
| "Just skip validation" | "I cannot. Validation ensures the artifacts are well-formed. Let me fix any issues and re-run." |
| "Don't need design.md" | "OK, I'll note that design.md is not needed for this change." |
| Skill not loaded | Invoke `team-openspec-guard` skill via Skill tool before any action. |
| `openspec list` fails | "OpenSpec is not accessible from the project root. Run `openspec init --tools none .` first to initialize OpenSpec." |
| "Add feature X too" (scope creep) | "That is out of scope. Let's finish this proposal first. You can create a separate change for X." |
| User describes a bug fix / small change | Suggest redirect to `/team-repair`. "This appears to be a low-complexity change. `/team-repair` is designed for this — it will diagnose, fix, verify, and handle dual-track closure in one command. Suggest switching to `/team-repair <description>`." |
| "Skip the redirect, I want full propose" | "OK, proceeding with full `/team-propose` pipeline." |
```

**Status reporting:**
- Report DONE when complete
- Report DONE_WITH_CONCERNS if you have doubts
- Report NEEDS_CONTEXT if you need more information
- Report BLOCKED if you cannot proceed
