---
name: neat-delivery-build
description: Superseded by neat-delivery-planning, which handles decomposition and execution. This skill is no longer invoked.
---

**Deprecated.** Use `neat-delivery-planning` instead — it handles requirement decomposition, tool selection, execution, and gating in a single skill.

# Build

**Role:** You are a tech lead who orchestrates feature builds from spec through verified code, using spec gates for alignment at every transition.

## Overview

Orchestrates continuous parallel builds: readiness → brainstorming → ADR extraction → writing-plans → risk assessment + spec gate → execution → risk assessment + spec gate. Dynamic parallel queuing automatically prepares and spawns independent features as others complete.

**Requires:** Feature docs in `docs/specs/<product>/features/`, superpowers: `brainstorming`, `writing-plans`, `subagent-driven-development`/`executing-plans`.

## When to Use

After `neat-delivery-planning` for end-to-end execution with verification. Implementation traces to feature doc.

**Not for:** Standalone implementation (use superpowers)

## Quick Reference

| Step | What |
|------|------|
| 1 | Pick feature(s), validate independence |
| 2 | Automatic state detection - resume from Status/plan/design or fresh |
| 3 | Check entry criteria, doc quality (per feature) |
| 4 | Discover blast area (per feature) |
| 5 | Design - query KB (per feature) |
| 6 | Extract ADRs (per feature) |
| 7 | Write TDD tasks (per feature) |
| 8 | Analyze dependencies (per feature) |
| 9 | Risk assessment → gate design + plan if needed (per feature) |
| 10 | Spawn all features in background (isolated worktrees) |
| 11 | Monitor completion, merge as finished |
| 12 | Risk assessment → gate code if needed (per feature) |
| 13 | Update feature doc status, auto-ingest to KB (per feature) |
| 14 | Prompt for audit if 2+ features with relationships |
| 15 | Continue building |

## Setup

Locate specs.md ([procedure](../references/specs-location.md)), construct output path ([rules](../references/output-conventions.md)), read features with `state: planned` (no `## Status`).

**Feature state:** `state: implemented` = code complete + verified. Build updates frontmatter + appends `## Status`.

## Process

### Step 1: Pick Feature(s) (BLOCKING)

Present features with entry criteria. User selects 1+ features.

**Independence Validation:** Check `depends_on`, blast areas, task plans. Conflicts → present, ask adjust. Independent → batch workflow (worktrees).

### Step 2: Automatic State Detection

Per [State Detection Algorithm](references/state-detection.md). `## Status` → "Build again?". Plan exists → resume Step 10. Design + ADRs → resume Step 7.

### Step 3: Readiness Check

**Prerequisites:** NO `## Status`. Validate `depends_on` features exist. Missing → STOP, recommend audit.

**Doc Quality:** Score components, risks, goal. 3/3=High, 2/3=Medium, 0-1/3=Low. High/Medium → Step 4. Low → planning.

### Step 4: Discover Blast Area Files

Parse components → keywords → search → rank → confirm.

### Step 5: Brainstorming

**Load KB:** Read specs.md KB. Found → invoke `neat-knowledge-extract` with blast area questions (fallback: direct reads). NO KB → read specs.md, parse, read files.

**Invoke:** `/brainstorming` with: "Use neat-knowledge-extract for queries. Check KB BEFORE asking. Derive detailed acceptance criteria from design decisions." Inputs: feature doc (goal, components, risks), specs.md, KB, blast area. Output: `docs/superpowers/specs/`.

**After:** Update feature doc: `designed: YYYY-MM-DD`, `spec_doc: path`, acceptance criteria section. → Step 6.

### Step 6: Extract ADRs

Invoke `neat-delivery-adr {design-spec} {feature-doc} integrated`. Outcomes: SUCCESS → Step 7 | MINOR → auto-fix | MAJOR → re-brainstorm.

### Step 7: Writing Plans

Invoke `/writing-plans` with design spec and feature doc. Output: `docs/superpowers/plans/`.

### Step 8: Dependency Analysis

Per [algorithm](references/dependency-analysis.md): Count tasks, build graph, identify layers, present breakdown.

### Step 9: Risk Assessment + Gate — Design + Plan

Analyze complexity per [algorithm](references/risk-assessment.md#design-phase-assessment). Gate runs → ensure artifacts, invoke `neat-delivery-gate <product>` (auto-detects design).

### Step 10: Spawn Agents

Per feature: spawn with worktree isolation (`run_in_background: true`), pass tasks + docs, track feature→agent→worktree. After all spawned → Step 11. See [reference](references/parallel-execution.md).

### Step 11: Monitor & Queue

Wait for agents. Per completion:

1. Retrieve status, worktree path
2. **BLOCKING:** `/simplify` in worktree, fix, commit. MUST run before merge. No exceptions (code looks clean, agent reviewed, simple changes). Catches: reuse, quality, efficiency.
3. Merge to main, run tests
4. Tests pass → Step 12 (Risk + Gate) → Step 13 (state: implemented). State MUST update BEFORE next.
5. Tests fail → mark, log, continue

**After Step 13:**

1. Check `state: planned` features
2. Validate independence (depends_on, blast areas, plans)
3. Independent → Steps 3-9, spawn Step 10
4. Conflicts → wait

**Until:** No planned OR all conflict.

### Step 12: Risk Assessment + Gate — Execute

Analyze per [algorithm](references/risk-assessment.md#execute-phase-assessment). Gate → ensure artifacts, invoke `neat-delivery-gate <product>` (auto-detects execute).

### Step 13: Update Doc (CRITICAL - BLOCKING)

MUST complete before next spawn.

1. Set `state: implemented`, append `## Status`
2. Auto-ingest: if installed + KB exists, invoke, log
3. Announce

**Red Flag:** Next check before state update - STOP.

### Step 14: Audit Prompt

2+ implemented AND (`depends_on` OR overlaps) → prompt "Run audit? Y/n". Y → invoke `neat-delivery-audit`.

### Step 15: Completion

"All features built."

## Gate Handling

Gates run if medium/high risk. Low-risk → skip, log. Issues → manually invoke `neat-delivery-gate <product>`.

**Failure:** "Fix plan | Fix design | Accept | Abort". Max 3. Wrong criteria → surface, update if approved, re-run.

## Common Mistakes

See [Common Mistakes Reference](references/common-mistakes.md).

## Output

```text
docs/superpowers/
  specs/YYYY-MM-DD-{goal}-{slug}-design.md
  plans/YYYY-MM-DD-{goal}-{slug}-plan.md

docs/specs/<product>/
  adrs/adr-YYYYMMDD-<decision>.md, index.md
  features/feature-{goal}-{nn}-{slug}.md              # state: implemented, Status
  features/feature-{goal}-{nn}-{slug}-gates.md        # by neat-delivery-gate
```
