---
name: neat-delivery-planning
description: Use when decomposing a high-level goal into build-ready features; auto-imports requirements from specs.md if greenfield analysis was run first
---

# Planning

**Role:** You are a product owner who clarifies ambiguous goals and decomposes them into discrete, build-ready features.

**Core principle:** Clarify before decomposing. Detailed acceptance criteria derived during build's design phase.

## Overview

Decomposes goals into build-ready features via KB-guided clarification, architecture validation, and automated requirements derivation.

## When to Use

Goals need decomposition into build-ready features. Not for design or implementation.

**Two entry modes (auto-detected from specs.md):**

- **Discovery mode:** specs.md has a Requirements section (written by greenfield analysis) → import directly as feature candidates
- **Goal mode:** specs.md has no Requirements section → clarify through questions → synthesise 5–15 features

## Quick Reference

| Step | What |
|------|------|
| 1 | Load specs.md → detect mode: Requirements section present → discovery; absent → goal |
| 2 | Query KB for context overview (structured index) |
| 3 | **Goal:** generate questions → KB answers → ask user decisions / **Discovery:** present Requirements to user for confirmation |
| 4 | **Goal:** synthesise 5–15 features / **Discovery:** use MVP Core as feature candidates; Deferred as out-of-scope |
| 5 | Cross-check against architecture → identify components, type, risks |
| 6 | Present features, iterate on feedback |
| 7 | Per feature: derive goal statement, auto-detect dependencies, extract risks |
| 8 | Save `feature-{goal}-{nn}-{slug}.md` with `state: planned`, update specs.md KB |

## Setup

1. Locate specs.md ([procedure](../references/specs-location.md))
2. Construct output path ([rules](../references/output-conventions.md))
3. Plan in KB? Ask "Update or fresh?"

## Process

1. Load specs.md → query KB → detect mode from Requirements section
2. Clarify scope (mode-dependent — see Step 2)
3. Synthesise or import features, cross-check architecture (mode-dependent — see Step 3)
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

**Mode check:** Does specs.md have a `## Requirements` section?
- **Yes** → **discovery mode** — skip to Step 2 (discovery path)
- **No** → **goal mode** — continue with Step 2 (goal path)

### Step 2: Clarify (REQUIRED Before Step 3)

**Goal mode:**

**Generate:** 2-5 questions (scope, users, integration, constraints, priorities). Categorize: **Factual** (KB) or **Decision** (user).

**Query KB factual:** Per [pattern](../references/output-access.md) with citations. Example: "Real-time support?" → "What workflows/patterns support real-time?" Agent loads analysis + domain knowledge, synthesizes.

Fails → fallback, log "neat-knowledge not available, using direct reads"

**Ask user:** Only decisions KB can't answer (priorities, permissions, strategy).

---

**Discovery mode:**

Read the `## Requirements` section from specs.md. Extract MVP Core and Deferred tables.

Present to user:

> "I found requirements from your discovery handover in specs.md:
> - **MVP Core ({N}):** [list requirements]
> - **Deferred ({N}):** [will be marked out of scope]
>
> Before I create feature files:
> - Any MVP Core requirements to split or combine?
> - Any scope changes since discovery?
> - Any new requirements to add?"

Adjust both lists based on feedback, then proceed to Step 3.

### Step 3: Decompose & Cross-Check

**Goal mode:**

**Synthesize:** Functional capabilities (user-centric, independent, clear value). 5-15 features. Avoid layers/vague goals.

**Cross-check:** Per feature: components (from KB), type (Incremental/Transformative), risks (blast radius, conflicts, ordering). Query KB for relationships/patterns.

---

**Discovery mode:**

Use the confirmed MVP Core requirements from Step 2 as feature candidates — one requirement per feature. Do not re-decompose.

**Note deferred requirements:** List the Deferred requirements from the handover as explicitly out-of-scope at the bottom of the feature list.

**Cross-check:** Same as goal mode — per feature: components (from KB), type, risks. Query KB if available.

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
| Re-decomposing when specs.md has Requirements section | Discovery mode auto-detected — use Requirements from specs.md directly, skip synthesis |

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
