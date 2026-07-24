---
name: neat-delivery-planning
description: Use when user has a high-level goal or objective that needs to be broken down into discrete features for build
---

# Planning

**Role:** You are a product owner who clarifies ambiguous goals and decomposes them into discrete, build-ready features.

**Core principle:** Clarify before decomposing. Detailed acceptance criteria derived during build's design phase.

## Overview

Decomposes goals into build-ready features via KB-guided clarification, architecture validation, and automated requirements derivation.

## When to Use

Goals need decomposition into build-ready features. Not for design or implementation.

## Quick Reference

| Step | What |
|------|------|
| 1 | Query KB for context overview (structured index) |
| 2 | Generate clarifying questions → query KB for factual answers → ask user for decisions |
| 3 | Synthesize capabilities from goal (5-15 features) |
| 4 | Cross-check against architecture → identify components, type, risks |
| 5 | Present features, iterate on feedback |
| 6 | For each feature: derive goal statement, auto-detect dependencies, extract risks |
| 7 | Derive goal identifier, save as `feature-{goal}-{nn}-{slug}.md` with `state: planned`, update specs.md KB |

## Setup

1. Locate specs.md ([procedure](../references/specs-location.md))
2. Construct output path ([rules](../references/output-conventions.md))
3. Plan in KB? Ask "Update or fresh?"

## Process

1. Query KB overview per [pattern](../references/output-access.md)
2. Generate questions → query KB factual → ask user decisions
3. Synthesize capabilities, cross-check architecture
4. Present features with components/risks, iterate
5. Per feature: derive goal, auto-detect dependencies, extract risks
6. Save `features/<slug>.md`, update specs.md KB

### Step 1: Load Context

Query KB per [pattern](../references/output-access.md):

```markdown
Invoke: neat-knowledge-extract "What is tech stack, integrations, components, workflows, business logic?"
```

Agent evaluates matches, decides depth (summary/sections/full). Parse JSON: extract tech_stack, integrations, components, workflows, business_logic.

Fails → fallback to direct reads, log "neat-knowledge not available, using direct reads"

**Fallback:** Read specs.md, parse KB, read analysis.

KB minimal → use goal only; factual → decision questions.

### Step 2: Clarify (REQUIRED Before Step 3)

**Generate:** 2-5 questions (scope, users, integration, constraints, priorities). Categorize: **Factual** (KB) or **Decision** (user).

**Query KB factual:** Per [pattern](../references/output-access.md) with citations. Example: "Real-time support?" → "What workflows/patterns support real-time?" Agent loads analysis + domain knowledge, synthesizes.

Fails → fallback, log "neat-knowledge not available, using direct reads"

**Ask user:** Only decisions KB can't answer (priorities, permissions, strategy).

### Step 3: Decompose & Cross-Check

**Synthesize:** Functional capabilities (user-centric, independent, clear value). 5-15 features. Avoid layers/vague goals.

**Cross-check:** Per feature: components (from KB), type (Incremental/Transformative), risks (blast radius, conflicts, ordering). Query KB for relationships/patterns.

### Step 4: Present & Iterate

Show: Name, description, components, type, risks. Iterate until approved.

### Step 5: Derive Requirements

Per approved feature:

**Goal:** One-sentence outcome. Pattern: "Users can {action} via {mechanism}" or "{System} enables {capability}".

**Dependencies:** Parse components from all features. Per feature: Query KB "What infrastructure does {component} require?" → cross-reference features → create depends_on list.

**Risks:** Query "What are known risks for {components}?" Extract from analysis. None → "None identified from KB."

### Step 6: Save & Update

**Goal identifier:** 1-3 key terms (lowercase, hyphens, max 20 chars).

Examples: "Implement OAuth" → `auth`, "Real-time editing" → `realtime-collab`, "API v2 GraphQL" → `api-v2`, "Migrate microservices" → `microservices`

**Save:** `docs/specs/<product>/features/feature-{goal}-{nn}-{slug}.md` (two-digit numbers, scoped per goal).

**Update specs.md:** `- Features: docs/specs/<product>/features/ (8 features)` per [format](../references/output-conventions.md)

**Recommend:** `neat-delivery-build` for design/implementation. Build derives acceptance criteria during design.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping clarification | Always clarify before decomposing |
| Asking factual questions | Query KB for facts, ask user for decisions |
| Wrong granularity | 5-15 capabilities with user value |
| Skipping architecture cross-check | Must identify components, type, risks |
| Not deriving goal identifier | Derive from user's goal in Step 6 |
| Skipping requirements derivation | Must derive goal statements, dependencies, risks in Step 5 |

## Output

Save to `docs/specs/<product>/features/feature-{goal}-{nn}-{slug}.md` per [output path rules](../references/output-conventions.md).

**Format:**

```markdown
---
name: Feature Name
goal: goal-identifier
state: planned
created: YYYY-MM-DD
depends_on: [feature-id-1, feature-id-2]  # If dependencies detected
---

# Feature Name

Brief 1-2 sentence description.

## Goal

One-sentence outcome statement derived from feature description.

## Components Affected

**Components affected:** component-a, component-b

**Type:** Incremental | Transformative

**Cross-repo format (if applicable):** `[repo-name] component-name`

## Acceptance Criteria

(Derived during design phase - see Build skill Step 5)

## Risks

[Risks extracted from KB, or "None identified from KB"]
```

**Naming:**

- **Goal identifier:** 1-3 key terms from goal (lowercase, hyphens, max 20 chars)
- **Number:** Scoped per goal (01, 02, etc.)
- **Slug:** Feature name (lowercase, hyphens only, e.g., "Real-Time Editing!" → `realtime-editing`)

**Terminology standard:**

- **Section heading:** `## Components Affected` (always capitalized)
- **Field label:** `**Components affected:**` (bold with colon)
- **Inline reference:** "components" or "blast area" (lowercase)

**Note:** Build skill adds `designed: YYYY-MM-DD` and `spec_doc: path` to frontmatter during Step 5.
