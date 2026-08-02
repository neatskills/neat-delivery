---
name: neat-util-adr
description: Use when creating or extracting architectural decisions - standalone conversational mode or extraction from design specs - generates MADR format ADRs
neatskills:
  self_contained: true
---

# ADR Management

**Role:** You are a software architect who documents architectural decisions in MADR format.

**Usage:**

- Standalone: `neat-util-adr`
- Extraction: `neat-util-adr <design-spec-path> <feature-doc-path> <mode>`

## Overview

- **Modes:** Standalone (conversational) or Extraction (from "Key Decisions")
- **Architecture:** Shared utilities. Standalone in main, extraction spawns parallel sub-agents
- **Output:** MADR format in `docs/specs/<product>/adrs/`, shared numbering

## When to Use

| Mode | Use Cases |
|------|-----------|
| Standalone | Document without specs, record past decisions, explore trade-offs |
| Extraction | neat-delivery-build integration, backfill from specs, design review |

## Quick Reference

### Mode Detection

| Arguments | Mode | Workflow |
|-----------|------|----------|
| None | Standalone | Converse → Generate → Review → Save |
| 2-3 args | Extraction | Parse → Filter → Confirm → Extract → Save |

### Standalone Steps

| Step | What |
|------|------|
| 1 | Setup: paths, specs.md |
| 2 | Context: query KB |
| 3 | Converse: ask questions |
| 4 | Generate: assign #, format MADR |
| 5 | Review: present, edit |
| 6 | Save: write, update |

### Extraction Steps

| Step | What |
|------|------|
| 1 | Setup: mode, paths |
| 2 | Parse: extract Key Decisions |
| 3 | Filter: check significance |
| 4 | Confirm: present, approve |
| 5 | Spawn: parallel (1 per) |
| 6 | Extract: check, ask, generate |
| 7 | Save: collect, update |

## Setup & Mode Detection

**Detect:** No args → Standalone | 2+ args → Extraction

**Setup:** Locate specs.md ([standard procedure](../references/specs-location.md)), extract product. Path: `<repo-root>/docs/specs/<product>/adrs/`.

## Shared Utilities

[Shared utilities](references/utilities.md)

## Standalone Mode

### Phase 0: Context

Ask topic. Query KB per [knowledge query pattern](../references/output-access.md): "What decisions, components, constraints, risks for [topic]?" Load relevant context.

### Phase 1: Conversation

Ask: "What decision?" "What problem?" Check: decision, context, alternatives, rationale, consequences. Prompt if missing. Suggest alternatives, identify conflicts. See [examples](references/examples.md).

### Phase 2: Generation

Assign date (`YYYYMMDD`), filename `adr-{YYYYMMDD}-{slug}.md`. Status: "Accepted"/"Proposed". Format per [template](references/template.md).

### Phase 3: Review

Present for approval (yes/edit/no). Edit: ask → update → loop. No: cancel.

### Phase 4: Save

Write file. Update index.md (sort by date), specs.md. Auto-ingest per [auto KB pattern](../references/neat-knowledge.md): `neat-knowledge-ingest file <adr-path> --category adrs`.

**Errors:** Can't create dir → exit | Write fails → no updates | Corrupt index → backup + fresh

## Extraction Mode

Extract from `## Key Decisions` (H3 subsections). Input: `neat-util-adr <design-spec-path> <feature-doc-path> <mode>`. May yield zero ADRs (valid).

### Step 1: Parse & Filter

Parse H3 → extract number, title, content. Filter ADR-worthy (multi-component, hard to reverse, trade-offs, non-functionals). Skip local/trivial. Confirm: "Detected {N}: Significant ({X}), Uncertain ({Y}), Skipped ({Z}). Generate {X}?" → yes/no/add/remove. None = valid.

### Step 2: Assign Numbers

Use today's date (`YYYYMMDD`). All ADRs share date (slugs differentiate). Extract feature from `feature-<nn>-<slug>.md`.

### Step 3: Spawn Sub-Agents

**Dispatch:** Extract, generate MADR, save, return metadata. Inputs: title, content, date-number, paths, names. Steps: (1) Check completeness → ask if 2+ missing or 1 + <100 words (2) Filename: `adr-{date-number}-{slug}.md` (3) Generate per [template](references/template.md) (4) Write (5) Return metadata.

Spawn ALL (`Agent`, `subagent_type: "general-purpose"`). Collect.

### Step 4: Collect & Save

Receive metadata. Update index.md, specs.md. Auto-ingest per [auto KB pattern](../references/neat-knowledge.md): `neat-knowledge-ingest directory docs/specs/<product>/adrs/ --category adrs`. Benefit: ~30 tokens/ADR vs 800-1,200.

### Error Handling

| Error | Message |
|-------|---------|
| Spec not found | "Design spec not found at {path}" |
| No Key Decisions | "No 'Key Decisions' section found" |
| User rejects | "Canceled. No files created." |
| Sub-agent failure | "Generated {N} of {M}. Failed: {title}: {error}" |

### Success

| Outcome | Message |
|---------|---------|
| With ADRs | "SUCCESS: Extraction triggered: Analyzed {N} decisions, filtered to {M} significant<br>SUCCESS: Generated {M} ADRs: docs/specs/{product}/adrs/adr-{YYYYMMDD}-*.md<br>SUCCESS: Index and specs.md updated<br>SUCCESS: Indexed in project KB (if available)" |
| No ADRs | "SUCCESS: Extraction triggered: Analyzed {N} decisions, found 0 architecturally significant<br>INFO: No ADRs generated. All decisions were implementation-specific or trivial." |

## Output

ADRs saved to `docs/specs/<product>/adrs/` per [output conventions](../references/output-conventions.md). Updated in specs.md and KB.

## Common Mistakes

[Common mistakes](references/common-mistakes.md)
