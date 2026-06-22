---
name: team-openspec-guard
description: Use when creating, reviewing, or updating OpenSpec proposals, designs, tasks, spec deltas, acceptance criteria, or archiving changes.
---

# Team OpenSpec Guard

## Core Principle

OpenSpec artifacts are executable contracts: specific enough to drive implementation, review, and regression verification. **All artifacts MUST follow OpenSpec CLI standard format, or `openspec validate --strict` will fail.**

### Spec Format Survival Kit (compression-resistant)

These 3 rules are the minimum to pass validation. Even if Gate 2 details are compressed, these MUST be followed:
1. **Headers are EXACT:** `## ADDED Requirements` / `### Requirement: <name>` / `#### Scenario: <name>` (the word "Requirement:" is mandatory, 4 hashes for Scenario)
2. **Keywords:** Requirement text MUST contain `SHALL` or `MUST` (not "should"/"will"). Scenario body uses ONLY `- **WHEN**` / `- **THEN**` / `- **AND**`
3. **Every ADDED/MODIFIED requirement MUST have ≥ 1 scenario** with WHEN/THEN structure

## When to Use

Use this skill when:
- Creating `openspec/changes/<id>/proposal.md` (for medium/high complexity features)
- Writing `design.md`
- Writing spec deltas under `openspec/changes/<id>/specs/`
- Validating or archiving OpenSpec changes
- Turning vague requirements into verifiable specs

For bug fixes, security patches, and small enhancements, use `/team-repair` (with `team-repair-guard`) instead of the full OpenSpec pipeline.

Do NOT use this skill when:
- Fixing bugs or applying security patches (use `team-repair-guard` with `/team-repair`)
- Implementing code (use `team-implementation-guard`)
- Verifying results (use `team-verification-guard`)

## Gate 0: OpenSpec CLI Availability (Mandatory — Check First)

Before any OpenSpec operation, verify the CLI is installed and accessible:

```bash
openspec --version
```

If the command fails (not found / not recognized):

1. **STOP** — do not proceed with any OpenSpec operations
2. **Tell the user:**
   > OpenSpec CLI is not installed or not in PATH. Install it first:
   > ```bash
   > npm install -g @fission-ai/openspec
   > ```
   > Then verify with `openspec --version`.

If the command succeeds, proceed to Gate 1.

## Gate 1: Proposal Format (Mandatory — English headers)

proposal.md **MUST** use these sections (OpenSpec CLI hard-codes English header matching):

```markdown
## Why
<!-- 1-2 sentences. Minimum 50 characters. -->

## What Changes
<!-- Bullet list. Mark breaking changes with **BREAKING**. -->

## Capabilities
### New Capabilities
- `<name>`: <brief description>  <!-- kebab-case -->

### Modified Capabilities
- `<existing-name>`: <what requirement is changing>

## Impact
<!-- Affected code, APIs, dependencies, systems -->
```

Checklist:
- [ ] `## Why` header correct, content ≥ 50 chars
- [ ] `## What Changes` non-empty with specific changes
- [ ] `## Capabilities` lists all new/modified capabilities
- [ ] `## Impact` describes affected scope

## Gate 2: Spec Delta Format (Mandatory — English headers + specific format)

Every spec delta (`specs/<capability>/spec.md`) **MUST** follow this format:

```markdown
## ADDED Requirements

### Requirement: User can reset password

The system SHALL allow registered users to request a password reset email.

#### Scenario: Valid registered email
- **WHEN** user submits the reset form with a registered email
- **THEN** system sends a password reset email

#### Scenario: Unregistered email
- **WHEN** user submits the reset form with an unregistered email
- **THEN** system displays "If the email exists, a reset link will be sent"

## MODIFIED Requirements

### Requirement: Login rate limiting

The system SHALL limit login attempts to 5 per minute per account.

#### Scenario: Rate limit exceeded
- **WHEN** user exceeds 5 login attempts within 1 minute
- **THEN** system returns HTTP 429 and locks the account for 15 minutes

## REMOVED Requirements

### Requirement: Legacy password policy
**Reason**: Replaced by new password policy in ADDED section
**Migration**: Existing passwords will be re-hashed on next login

## RENAMED Requirements
- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**Critical rules** (violating any causes `--strict` validation failure):

1. Section headers MUST use: `## ADDED Requirements` / `## MODIFIED Requirements` / `## REMOVED Requirements` / `## RENAMED Requirements`
2. Requirement header MUST use: `### Requirement: <name>` (the `Requirement:` prefix is hard-coded in the parser)
3. Requirement text MUST contain `SHALL` or `MUST`
4. Scenario header MUST use 4 hashes: `#### Scenario: <name>` (3 hashes won't be recognized as a scenario)
5. Scenario body: `- **WHEN** ...` / `- **THEN** ...` / `- **AND** ...`
6. Every ADDED/MODIFIED requirement MUST have at least one scenario
7. REMOVED requirements only need the name (no scenarios needed)
8. RENAMED uses FROM:/TO: format

**Quick reference:**

| Element | Correct | Wrong |
|---------|---------|-------|
| Delta section | `## ADDED Requirements` | `## 新增需求`, `## Added` |
| Requirement | `### Requirement: Name` | `### 需求 1：名称` |
| Scenario | `#### Scenario: Name` | `#### 场景：名称` |
| Steps | `- **WHEN** / - **THEN**` | `- **假设**：/ - **当**：` |
| Keywords | Contains `SHALL` or `MUST` | No normative keyword |

## Gate 2.5: Plan and Tasks (Mandatory — writing-plans integration)

The implementation plan (`plan.md`) is created by invoking `superpowers:writing-plans` skill. The `tasks.md` is extracted from `plan.md`.

**Team workflow commit override:** After `writing-plans` generates the plan, REMOVE all per-task "Commit" steps (e.g., `Step N: Commit` with `git add/commit`). In team workflow, commits are managed at the change level by `/team-apply`, NOT per-task.

### plan.md format:

plan.md is generated by `superpowers:writing-plans` and follows its format:

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** ...

**Goal:** ...
**Architecture:** ...
**Tech Stack:** ...

---

### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file`
- Modify: `exact/path/to/file`

- [ ] **Step 1: Description**
- [ ] **Step 2: Description**
```

### tasks.md extraction from plan.md:

Extract each `- [ ]` step from plan.md and renumber:

**Input (plan.md):**
```markdown
### Task 1: Create user model
- [ ] Step 1: Write failing test
- [ ] Step 2: Run test to verify it fails
- [ ] Step 3: Implement User model
```

**Output (tasks.md):**
```markdown
> **Working Directory:** When executing with `/team-apply`, all file paths below are relative to `.worktrees/<change-id>/` (WORKTREE_DIR), NOT the main project directory. Read/Edit/Write/Glob/Grep MUST use WORKTREE_DIR absolute paths.

## 1. Create user model
- [ ] 1.1 Write failing test
- [ ] 1.2 Run test to verify it fails
- [ ] 1.3 Implement User model
```

**Extraction rules:**
1. Each `### Task N: <name>` becomes `## N. <name>`
2. Each `- [ ] Step N: <description>` becomes `- [ ] N.M <description>`
3. Steps are numbered sequentially within each task group
4. Preserve the exact description text from plan.md
5. **Prepend working directory header** (the blockquote above) to every tasks.md

## Gate 3: Open Questions

Open questions are **hard blockers** (must resolve before implementation) when they affect:
- API fields
- Permissions
- Data migration
- Payment or security behavior
- UX branching
- Backward compatibility

**If hard blockers exist, do NOT proceed to implementation.**

Format:
```markdown
## Open Questions

### [Q1] API field naming
- Impact: API contract, database model
- Options:
  - A: `user_email`
  - B: `email_address`
- Recommendation: A
- Blocking: 🔴 Hard (must resolve before implementation)

### [Q2] Error message wording
- Impact: UX
- Options: [list]
- Blocking: 🟡 Soft (can resolve during implementation)
```

## Gate 4: Task Quality

Every task in `tasks.md` MUST:
- Be small enough to review (MUST be ≤ 1 hour)
- Link to a requirement or design decision
- Be verifiable by test, command, or manual scenario
- Be ordered by dependency

**Wrong:**
```markdown
- [ ] Implement authentication
```

**Right:**
```markdown
## 1. Data Layer
- [ ] 1.1 Add reset-token persistence model
  - Requirement: password-reset
  - Verification: Write unit tests for token expiry and single-use
  - Estimate: 30 min

- [ ] 1.2 Create password reset API endpoint
  - Requirement: password-reset
  - Verification: Integration test for reset flow
  - Estimate: 45 min
  - Depends on: 1.1
```

Required task format:
```markdown
- [ ] X.Y <Task title>
  - Requirement: <requirement name or ID>
  - Verification: `<command or manual scenario>`
  - Estimate: <minutes>
  - Depends on: <preceding task, if any>
```

## Required Validation

Before reporting proposal work as done:
```bash
openspec validate <change-id> --strict
```

If validation fails, fix artifacts before reporting done. Common failures:
- `## Why` less than 50 characters
- Spec delta missing delta header (`## ADDED Requirements` etc.)
- Requirement missing `SHALL`/`MUST`
- Requirement has no `#### Scenario:` block
- Same requirement in two conflicting sections (e.g., both ADDED and REMOVED)

## Design Template

design.md is **REQUIRED** when ANY of these apply: new components/modules, multiple implementation approaches, data schema or API changes, cross-subsystem impact, migration concerns, non-trivial error handling/security decisions. Only skip for simple changes (add field, fix bug, update config).

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

## Quick Checklists

### Before submitting proposal:
- [ ] `## Why` in English, ≥ 50 chars
- [ ] `## What Changes` has specific bullets
- [ ] `## Capabilities` lists kebab-case names
- [ ] `## Impact` describes scope
- [ ] `openspec validate <change-id> --strict` passes

### Before submitting spec delta:
- [ ] Standard delta headers used (`## ADDED Requirements` etc.)
- [ ] Every requirement uses `### Requirement: <name>`
- [ ] Every requirement contains `SHALL` or `MUST`
- [ ] Every scenario uses `#### Scenario: <name>` (4 hashes)
- [ ] Scenarios use `WHEN`/`THEN` format
- [ ] Every ADDED/MODIFIED requirement has ≥ 1 scenario

### Before archiving:
- [ ] All required tasks completed
- [ ] Deferred tasks have reasons
- [ ] All acceptance criteria have evidence
- [ ] `openspec validate <change-id> --strict` passes
- [ ] Global `openspec validate --strict` passes

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Chinese headers like "## 问题" | Use "## Why" |
| Requirement as "### 需求 1：..." | Use "### Requirement: ..." |
| Scenario as "#### 场景：..." | Use "#### Scenario: ..." |
| Scenario uses 假设/当/那么 | Use WHEN/THEN/AND |
| Requirement text has no SHALL/MUST | Explicitly add SHALL or MUST |
| Acceptance criteria says "works correctly" | Replace with specific scenarios |
| Task too broad | Split by user-visible vertical slices |
| Open questions buried in paragraphs | Put in explicit list |
| Missing design trade-offs | Add `design.md` before implementation |
| Non-goals not listed | Explicitly state "NOT doing X, Y, Z" |

## Bad Cases

When this skill fails, record in `badCases/` directory:
- Input: what the user said
- Wrong output: what the AI did that it shouldn't have
- Expected output: what it should have done
- New rule needed: how to prevent recurrence

See `badCases/` for case history.
