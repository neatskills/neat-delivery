---
name: neat-delivery-analysis
description: Use when analyzing an existing codebase - extracts structured knowledge through layered analysis (L0-L6) before planning and build
---

# Analysis

**Role:** You are a software architect who extracts actionable insights from codebases — every finding answers "so what?"

## Overview

Layered knowledge extraction (L0-L6) for unfamiliar codebases.

## When to Use

- Starting delivery after a discovery-designed greenfield project (no codebase yet)
- Onboarding to existing codebases
- Pre-refactoring/migration
- Building understanding before changes
- Reference analysis

## Modes

| Mode | When | L1–L6 |
|------|------|--------|
| **Greenfield** | No codebase exists; project designed but not yet built | Skipped — reads discovery handover only |
| **Brownfield** | Existing product codebase, with or without discovery context | Full L1–L6 |
| **Platform** | Monorepo or multi-product workspace; shared infrastructure focus | Workspace-scoped |
| **Reference** | External or third-party codebase needed as context | Run first, before target |

Platform and Reference can combine with Brownfield (analyze the platform, then analyze a product within it).

## Quick Reference

| # | What |
|---|------|
| 1 | Detect mode: Greenfield / Brownfield / Platform / Reference |
| 2 | **Greenfield:** Ask user to share handover → create specs.md → recommend planning |
| 3 | **Platform:** Check → prompt → list/select. **Reference:** Analyze first |
| 4 | Derive name (slugify, max 20) |
| 5 | Check empty → minimal (Brownfield/Platform) |
| 6 | Check existing analysis, warn |
| 7 | L1–L6, L0 last (Brownfield/Platform/Reference only) |
| 8 | Save, update specs.md, offer PDF |
| 9 | Suggest next skill (mode-dependent) |

## Setup

1. **Detect mode:**
   - No existing codebase → **Greenfield**
   - Workspace/tooling/CI focus (monorepo, shared infra) → **Platform**
   - External or third-party codebase → **Reference** (analyze first, then return to target)
   - Existing product codebase → **Brownfield**
2. **Greenfield:** Ask user to share discovery handover. Skip L1–L6. Go to [Greenfield Analysis](#greenfield-analysis).
3. **Platform:** Check `docs/specs/analysis-platform.md`. **Missing:** Prompt. **Declines:** Warn. **Agrees:** Analyze, list, select.
4. **Reference:** Ask purpose, analyze FIRST.
5. Derive name (slugify, max 20)
6. Check source → empty? Minimal
7. Check existing, warn

---

## Layered Analysis

Sequential L1→L6, L0 last. Do NOT skip. **Stop if fails.**

```mermaid
graph TD
  L1[L1: Foundation] --> L2[L2: Interfaces]
  L2 --> L3[L3: Architecture]
  L3 --> L4[L4: Technical Flows]
  L4 --> L5[L5: Business Logic]
  L5 --> L6[L6: Health]
  L6 --> L0[L0: Executive Summary - write LAST, place FIRST]
```

### L1: Foundation

Tech stack, dependencies, structure (max 30 lines), build. **Platforms:** Context first, then Tech Stack.

### L2: Interfaces

API endpoints, data models, events, config, integrations.

### L3: Architecture

Pattern, component map, deployment, concerns. Flag anti-patterns.

### L4: Technical Flows

3+ flows (user, auth, error), state management, 2-3 edge cases/flow. Mermaid for complex.

### L5: Business Logic

User journeys, workflows, domain concepts, permissions, edge cases.

### L6: Health

Test coverage, tech debt, dead code, security, dependencies. Flag anti-patterns. **Be honest—risks, not praise.**

### L0: Executive Summary (write last, place first)

Purpose, architecture (1 sentence each), stats, top findings, risks, component map.

---

## Platform Analysis

For platforms (multiple products, shared infrastructure).

### Scope

**Analyze:** Workspace, tooling, orchestration, platform services, CI/CD, patterns
**Exclude:** Product code

### Layer Focus

| Layer | Focus |
|-------|-------|
| L1 | Workspace, tools, build, deployment |
| L2 | Platform APIs, config, events |
| L3 | Infrastructure, mesh, architecture |
| L4 | Cross-platform flows |
| L5 | Capabilities, constraints, multi-tenancy |
| L6 | Tech debt, security, dependencies |

**Output:** `analysis-platform.md` at `<repo-root>/docs/specs/`

### Pushback

| Pressure | Response |
|----------|----------|
| "Quickly" | "Prevents rework." |
| "Just [product]" | "Captures deploy, integration." |
| "Standard" | "Custom exists." |
| "Waste time" | "Downstream needs it." |

---

## Greenfield Analysis

For projects with no existing codebase — designed but not yet built.

### Input

**Ask:** "Please share the discovery handover from neat-discovery — paste the contents or provide the file path."

If not provided, stop: "A discovery handover is required for greenfield analysis. Complete /neat-discovery-designing first, then share the handover here."

The handover is the only artifact read — no assumptions about file location.

### Process

1. Parse shared handover content
2. Extract: project name, Architecture Pattern, Technology Decisions, Requirements (MVP Core + Deferred), Open Risks
3. Derive product name (slugify project name, max 20 chars)
4. Check if `docs/specs/{product}/specs.md` already exists — warn if so, ask before overwriting
5. Create specs.md from handover content (see Output below)
6. Register no analysis file — greenfield produces specs.md only

### Output

`docs/specs/{product}/specs.md`:

```markdown
# {product-name}

<!-- Generated by neat-delivery-analysis (greenfield). Seeded from discovery handover. -->

## Tech Stack

{Technology Decisions from handover — one line per committed choice}

## Architecture

{Architecture Pattern table + Key Architecture Decisions from handover}

## Requirements

### MVP Core

| # | Requirement | Type | Size |
|---|-------------|------|------|
| {REQ-001} | {description} | {AI / Traditional / Hybrid} | {S / M / L / XL} |

### Deferred

| # | Requirement | Reason for deferral |
|---|-------------|---------------------|
| {REQ-010} | {description} | {reason} |

## Conventions

(TBD — no code exists yet)

## Avoid

{Open risks from handover that imply build constraints — omit if none apply}

## Commands

(TBD — no code exists yet)

## Knowledge Base

{if initialized}

## Outputs

(none yet — populated as delivery progresses)
```

No L1–L6 analysis file is generated. No PDF offered.

---

## Empty Handling

Minimal analysis (L0-L6 placeholders, `empty: true`), minimal specs.md.

---

## Output Format

**Filename:** `analysis-<product-name>.md`, `analysis-<ref-name>.md`, `analysis-platform.md`
**Location:** [output-conventions.md](../references/output-conventions.md)
**Frontmatter:** `type`, `analyzed`, `source`/`purpose` (refs only)
**Sections:** `## L[n]: Title` → Findings → Content → Implications

## Process

TodoWrite: ask structure/refs → (platform: check → prompt → list/select) → (refs: first) → derive name → check empty → L1-L6 → L0 → save → update specs.md.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| File tree dump | Conventions, max 30 lines |
| Happy path only | Add auth, error, edge cases |
| Facts only | Add implications |
| L0 first | Write last |
| Prose | Tables, Mermaid |
| Refs after target | FIRST |
| Missing frontmatter | Add type/analyzed/source/purpose |
| Empty: fail/skip | Minimal + `empty: true` |
| L1–L6 on greenfield | Detect mode first; greenfield skips analysis entirely |
| Greenfield without handover | Handover is required — no handover means can't run greenfield mode |

## Platform Rationalization

| Thought | Reality |
|---------|---------|
| "User knows" | May not |
| "Just focus" | Can't without |
| "Quickly" | Prevents rework |
| "Standard" | Custom exists |
| "Waste time" | Skipping worse |
| "Docs done" | Docs ≠ L0-L6 |
| "Add later" | Needs now |
| "Don't" | Explain, offer |

**Prompt with reasoning.**

## After Saving

1. **Detect KB path**:
   - Check if neat-knowledge skills installed: `test -L ~/.claude/skills/neat-knowledge-ingest && echo "installed" || echo "not-installed"`
   - If "installed": Search for KB in project: `find . -name "metadata.json" -path "*/.index/metadata.json" -type f 2>/dev/null | head -1`
   - If found: Extract KB directory (parent of `.index/`), convert to relative path from repo root
   - Store as `KB_PATH` for Step 2-3
2. **Create/update specs.md**:
   - **Location:** Per [specs location rules](../references/specs-location.md), `<repo-root>/docs/specs/<product-name>/specs.md`
   - Sections: Tech Stack, Architecture, Conventions, Avoid, Commands, Knowledge Base, Outputs
   - **Knowledge Base:** Add section before Outputs (see [KB section format](../references/neat-knowledge.md#kb-section-in-specsmd))
   - **Outputs** ([format](../references/output-conventions.md)): `Analysis: docs/specs/<product>/analysis-<product>.md`
   - **References:** Add Target/References
   - **Platforms:** Add Context, nest Tech Stack
   - **Commands:** Auto-detect
   - Marker: `<!-- Generated by neat-delivery-analysis. Review and customize. -->`
3. **Auto-ingest** (if KB path exists):
   - If `KB_PATH` set:
     - Invoke: `neat-knowledge-ingest file docs/specs/<product>/analysis-<product>.md --category analysis`
     - Log: "✓ Indexed analysis in project KB"
   - If not set: Skip auto-ingest (user can initialize KB with `/neat-knowledge-ingest <any-file>`)
4. **Offer PDF:** "Want PDF? (needs `neat-utils`)" → invoke `neat-util-pdf`
5. **Suggest (mode-dependent):**
   - **Greenfield:** `neat-delivery-planning` — domains and KB population are not applicable until code exists
   - **Brownfield/Platform/Reference:** `neat-knowledge-extract` or `neat-delivery-domains`
