---
name: team-brainstorm-guard
description: Use when exploring requirements, doing deep design exploration, or brainstorming before proposing. Guards OpenSpec environment and brainstorm output location.
---

# Team Brainstorm Guard

## Core Principle

Exploration phase creates design artifacts only (brainstorm.md). No implementation, no proposal.

## When to Use

Use this skill when:
- Requirements are vague and need deep exploration
- Need to understand existing system implementation
- Need to compare multiple approaches
- Need structured brainstorming before /team-propose
- Problem is complex enough to warrant design exploration

Do NOT use this skill when:
- Problem is already clear, change-id is decided → go directly to `/team-propose`
- Creating proposals or OpenSpec artifacts (use `team-openspec-guard`)
- Implementing code (use `team-implementation-guard`)

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

## Gate 1: OpenSpec Directory Access (Mandatory)

Run `openspec list` to verify the openspec directory is accessible.

If the command fails:

1. **STOP** — the openspec directory is not accessible
2. **Tell the user:**
   > OpenSpec directory is not accessible from the project root. Initialize it first:
   > ```bash
   > openspec init --tools none .
   > ```
   >
   > Then verify with `openspec list`.

If the command succeeds, proceed to Gate 2.

## Gate 2: Brainstorm Output Location (Mandatory)

All brainstorm output MUST write to `openspec/changes/<change-id>/brainstorm.md`.

**DO NOT** write to:
- `docs/superpowers/specs/`
- Any other location

This ensures OpenSpec can track and validate the design artifact.

## Brainstorming Process

After gates pass, invoke `superpowers:brainstorming` skill via Skill tool.

The brainstorming skill will:
1. Explore project context (files, docs, recent commits)
2. Offer visual companion (if topic involves visual questions)
3. Ask clarifying questions — one at a time
4. Propose 2-3 approaches with trade-offs and recommendation
5. Present design sections, get approval after each section

**IMPORTANT output redirection:**
- Do NOT write to `docs/superpowers/specs/`. Instead, write the brainstorming design output directly to `openspec/changes/<change-id>/brainstorm.md`.

## Spec Self-Review

After writing brainstorm.md, perform self-review:

1. **Placeholder scan:** Any "TBD", "TODO", incomplete sections? Fix them.
2. **Internal consistency:** Do any sections contradict each other?
3. **Scope check:** Is this focused enough for a single implementation plan?
4. **Ambiguity check:** Could any requirement be interpreted two different ways? If so, pick one and make it explicit.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Writing code during exploration | Design only, no implementation |
| Skipping openspec environment check | Must verify CLI and directory first |
| Writing to docs/superpowers/specs/ | Write to openspec/changes/<change-id>/brainstorm.md |
| Not following brainstorming skill process | Invoke superpowers:brainstorming and follow its checklist |
| Finalizing with unresolved hard blockers | Keep hard blockers in Open Questions section |

## Bad Cases

When this skill fails, record in `badCases/` directory:
- Input: what the user said
- Wrong output: what the AI did that it shouldn't have
- Expected output: what it should have done
- New rule needed: how to prevent recurrence

See `badCases/` for case history.
