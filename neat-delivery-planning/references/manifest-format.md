# Manifest Format

Field-level spec for the decomposition manifest produced at Step 6 of planning.

**Location:** `docs/specs/<product>/decompositions/decomposition-{goal}-{nn}.md`

## File Structure

```markdown
---
goal: <requirement text, verbatim>
product: <product name>
created: YYYY-MM-DD
knowledge_sources: [<doc paths read, or "none">]
global_constraints:
  implementation_affecting:
    - <constraint>
  post_hoc_verifiable:
    - <constraint>
conventions:
  - <convention>
---

# Decomposition: <goal>

## Coordinator

**Who:** The planning session itself (Claude Code). No separate coordinator agent is spawned.

**At phase boundaries (`boundary: phase`):** You read the leaf's declared outputs directly with Claude Code tools and run:
1. Structural check — verify each declared output exists and is non-empty
2. Semantic check — whole-diff read of the leaf's declared scope, looking for stale references, drift, contradictions

**At feature boundaries (`boundary: feature`):** Invoke `neat-util-gate` with the Feature's `produces`/`consumes` contract.

**Failure handling:** See recovery path declared per Workflow below.

## Recovery Path

<Per-Workflow recovery path — override defaults per workflow if needed>

| Failure type | Action |
|---|---|
| Structural gate failure | Retry the responsible leaf once with specific finding |
| Semantic gate failure | Targeted fix request, retry once |
| Two consecutive failures | Escalate to user: retry / re-decompose / abort |

## Tree Summary

<ASCII tree of the decomposition, showing Feature/Workflow nodes and leaves>

## Leaves

<One section per leaf — see Leaf Fields below>
```

## Leaf Fields

Each leaf is one fenced block:

```markdown
### <leaf-id>

**name:** <human-readable name>
**tool:** direct | inline | parallel | reviewed-loop
**parallel_group:** <group-name> (omit if not concurrent with siblings)
**dependencies:** [<leaf-id>, ...] (empty = no dependencies)
**boundary:** phase | feature | none

#### Brief

**scope:**
- <file or artifact path>

**inputs:**
- <what this leaf needs>

**outputs:**
- <what this leaf must produce>

**constraints:**
- <implementation-affecting constraints relevant to this leaf>

**primary_source:** Verify against <doc or file name>, not plan draft values.
```

## Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `goal` | string | yes | The requirement verbatim — unchanged from Step 1 |
| `product` | string | yes | Product/project name |
| `created` | date | yes | YYYY-MM-DD |
| `knowledge_sources` | list | yes | Paths read in Step 2, or `["none"]` |
| `global_constraints.implementation_affecting` | list | yes | Constraints that affect HOW to implement. Empty list if none. |
| `global_constraints.post_hoc_verifiable` | list | yes | Constraints verifiable after the fact. Empty list if none. |
| `conventions` | list | yes | Naming, file location, style conventions. Empty list if none. |
| `leaf-id` | string | yes | Unique. Format: `<phase-or-feature>-<nn>`, e.g. `phase-auth-01` |
| `name` | string | yes | Human-readable description of this leaf |
| `tool` | enum | yes | `direct` \| `inline` \| `parallel` \| `reviewed-loop` |
| `parallel_group` | string | no | Leaves in the same group run concurrently. Omit if sequential. |
| `dependencies` | list | yes | Leaf IDs this leaf depends on. Empty list = can start immediately. |
| `boundary` | enum | yes | Gate type after this leaf: `phase` \| `feature` \| `none` |
| `brief.scope` | list | yes | Files/artifacts this leaf will touch. Paths must resolve or be declared new. |
| `brief.inputs` | list | yes | What this leaf needs (from prior contract or Feature contract). |
| `brief.outputs` | list | yes | What this leaf must produce. |
| `brief.constraints` | list | yes | Distilled from global constraints — only what's relevant to this leaf. |
| `brief.primary_source` | string | yes | Exact instruction to verify against a named doc or file, not plan draft values. |

## Validation Checklist

Before presenting the manifest for approval:

- [ ] Coordinator section present — who, phase boundary actions, feature boundary actions
- [ ] Recovery path declared per Workflow
- [ ] Every leaf has all 5 brief fields present and non-empty
- [ ] `scope` file paths either resolve to existing files or are declared as new with intent
- [ ] `tool` for each leaf matches exactly one row in [tooling-selection.md](tooling-selection.md)
- [ ] `dependencies` form a DAG — no cycles
- [ ] `boundary` is set: `feature` for Feature-level leaves, `phase` for Workflow phase leaves, `none` for intermediate steps within a Workflow phase
- [ ] `global_constraints` are present (empty list if none, not omitted)
- [ ] Tree summary matches the leaf list — every internal node visible in the tree has corresponding leaves

## Example

```markdown
---
goal: Add rate limiting to the public API
product: neat-api
created: 2026-08-02
knowledge_sources:
  - docs/architecture/api-design.md
global_constraints:
  implementation_affecting:
    - Rate limit headers must follow RFC 6585 (X-RateLimit-*)
    - Redis is the shared state store — no in-memory state
  post_hoc_verifiable:
    - All endpoints return 429 on limit exceeded
    - X-RateLimit-Limit and X-RateLimit-Remaining headers present on every response
conventions:
  - Middleware files in src/middleware/
  - Tests co-located with source in __tests__/
---

# Decomposition: Add rate limiting to the public API

## Tree Summary

Workflow: Add rate limiting to the public API
├── phase-01: Implement rate limit middleware (inline)
├── phase-02: Apply to endpoints (parallel, 3 endpoint groups)
│   ├── phase-02-a: Apply to /auth endpoints
│   ├── phase-02-b: Apply to /data endpoints
│   └── phase-02-c: Apply to /admin endpoints
└── phase-03: Add integration tests (inline)

## Leaves

### phase-01

**name:** Implement rate limit middleware
**tool:** inline
**parallel_group:** (none)
**dependencies:** []
**boundary:** phase

#### Brief

**scope:**
- src/middleware/rate-limit.ts (new)
- src/middleware/index.ts

**inputs:**
- Architecture decisions from docs/architecture/api-design.md

**outputs:**
- src/middleware/rate-limit.ts — Redis-backed middleware with RFC 6585 headers

**constraints:**
- Redis is the shared state store — no in-memory state
- Headers must follow RFC 6585 (X-RateLimit-*)

**primary_source:** Verify against docs/architecture/api-design.md, not plan draft values.

---

### phase-02-a

**name:** Apply rate limit middleware to /auth endpoints
**tool:** parallel
**parallel_group:** phase-02
**dependencies:** [phase-01]
**boundary:** none
...
```
