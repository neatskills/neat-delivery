---
name: neat-delivery-planning
description: Use when planning the execution of a single requirement — applies the delivery framework to decompose it into an executable manifest and guide implementation
---

# Planning

**Role:** You are a delivery engineer who applies the delivery framework to decompose one requirement into an executable plan. During execution you also act as **coordinator** — the role that verifies outputs at contract boundaries, manages phase handoffs, and decides whether to proceed or escalate. Both roles are you; no separate coordinator agent is spawned.

## Overview

Takes one requirement, reads documentation to extract constraints and conventions, applies recursive decomposition (Feature vs Workflow at every node until every leaf is directly executable by one tool), produces a manifest, and drives execution.

Codebase exploration is done natively with Claude Code tools as needed during decomposition. Documentation (architecture docs, design decisions, conventions) is the knowledge input.

## When to Use

One requirement is ready to plan. One requirement per session.

**Not for:** requirement selection, discovery, or goal clarification — those happen upstream.

## Quick Reference

| Step | What |
|------|------|
| 1 | Receive the requirement |
| 2 | Ask for documentation (architecture, design, conventions) |
| 3 | Extract global constraints and conventions from docs |
| 4 | Recursive decomposition — Feature vs Workflow at every node |
| 5 | Per leaf: select tool, write and validate brief |
| 6 | Output manifest, get approval |
| 7 | Execute in dependency order |
| 8 | Gate at contract boundaries (structural + semantic) |

## Process

### Step 1: Receive Requirement

Ask:

> "What is the requirement to plan?"

Record it exactly as stated. This is the root node for decomposition — do not interpret or expand.

---

### Step 2: Ask for Documentation

Ask:

> "Is there documentation I should read before planning? This could include architecture design docs, technical decision records, convention guides, or any reference material relevant to this requirement. Share folder paths or individual file paths."

If provided:
- Read the files
- Extract: global constraints, conventions, architecture decisions, technical decisions

If nothing provided: continue without documentation; note in manifest.

---

### Step 3: Extract and Confirm Planning Context

From the documentation read in Step 2, consolidate:

**Global constraints (implementation-affecting):** rules that affect HOW to implement — naming, structure, format, patterns. Goes into every agent brief.

**Global constraints (post-hoc verifiable):** rules verifiable after the fact — counts, grep patterns, file structure checks. Goes to coordinator lens only.

**Note:** a constraint can belong to both buckets. A naming convention an agent must apply AND that can be grep-verified afterward goes into agent briefs (to guide implementation) AND the coordinator lens (to catch violations).

**Conventions:** naming patterns, file locations, code style.

**Architecture / technical decisions:** committed choices that bound the implementation.

Present to the user:

> "Here is what I've extracted as planning context:
>
> **Global constraints (implementation-affecting):** [list or "none found"]
> **Global constraints (post-hoc verifiable):** [list or "none found"]
> **Conventions:** [list or "none found"]
> **Architecture decisions:** [list or "none found"]
>
> Does anything need to be corrected or added before I decompose?"

Wait for confirmation or corrections before proceeding.

---

### Step 4: Recursive Decomposition

Apply the delivery framework's decision flow. Start at the root node (the requirement). Explore the codebase as needed using Claude Code tools. Recurse until every leaf is directly executable.

**Minimum structure principle:** decompose only as deep as the structure requires. Skip any Feature or Workflow layer that does not reflect real structure in the work. A single atomic action with no meaningful phases or independent subjects needs no decomposition at all.

**At every node, ask two questions in order:**

**Q1 — Is this right-sized?**

Can this node be placed unambiguously in exactly one row of the tooling table? (See [tooling-selection.md](references/tooling-selection.md))

**Guard:** reviewed task loop is phase-level only. If the node spans multiple phases or multiple independent subjects, answer No regardless of whether you could assign it to reviewed task loop — decompose further first.

- Yes → leaf. Proceed to Step 5 for this node.
- No → Q2.

**Q2 — What varies?**

| What varies | Decomposition | Define |
|-------------|---------------|--------|
| The subject — same type of work on N different things | Feature split (parallelize across subjects) | Feature contracts: `produces` / `consumes` declared per Feature, specifying dependencies on other Features |
| The type of work — N phases on one subject | Workflow split (sequence phases) | Phase contracts: `input` / `output` per phase |

Recurse on each resulting node from the top.

**Common compositions:**

| Pattern | Structure |
|---------|-----------|
| One subject, multiple phases | Workflow only |
| N independent subjects, each with phases | Feature outer → Workflow inner |
| One workflow, N parallel items within a phase | Workflow outer → Feature inner |
| Single atomic action | Direct (no decomposition) |

**Mixed work:** Default to Workflow outer (phases). Apply Feature decomposition within a phase when multiple items of the same type are independent.

**At every node, also:**
- Define contracts (Feature: `produces`/`consumes`; Phase: `input`/`output`)
- Identify which nodes have no contract dependency → those can run in parallel
- Place quality gates at boundaries

**Phase handoff mechanism:** when defining a Workflow's phase contracts, also define the physical mechanism for passing output to the next phase — file artifact, structured summary, or coordinator-injected context. Declare it in the manifest before execution begins.

---

### Step 5: Per Leaf — Select Tool and Write Brief

For each leaf node:

**1. Select tool** using [tooling-selection.md](references/tooling-selection.md):

| Situation | Tool |
|-----------|------|
| Single atomic action, ≤1 file, no state dependencies | Direct |
| Sequential steps, ≤5 steps, bounded context | Inline |
| N independent tasks (≤8), correctness fully specifiable upfront | Parallel agents |
| Correctness of output cannot be fully specified upfront — requires judgment or review | Reviewed task loop |

**2. Write brief** — all five fields required:

| Field | Content |
|-------|---------|
| `scope` | Files/components this leaf will touch (explore codebase to confirm). For Inline leaves, also include a numbered step sequence (≤5 steps). |
| `inputs` | What this leaf needs (from prior phase contract or Feature contract) |
| `outputs` | What this leaf must produce |
| `constraints` | Distilled from Step 3 — only constraints relevant to this leaf |
| `primary_source` | The canonical reference to verify against (named doc or file). For all leaf types: used to anchor implementation. For parallel agents: also instruct the agent to report which source they actually used. |

**3. For parallel agents:** add two explicit instructions to `constraints` in every brief:
- Git scoping: "Scope all git operations to your assigned files — never use a catch-all add. Prefer worktree isolation if available."
- Source reporting: "Report which source you used — name the primary source (doc or file), or state that you used plan draft values." Agents see their brief; if it is not in the brief, they will not report it.

**4. Validate brief:** All 5 fields present and non-empty. File paths in `scope` must resolve to existing files, or be new files with declared intent. Do not spawn any agent before validation passes.

**5. Set boundary type:** `phase` | `feature` | `none`

---

### Step 6: Output Manifest and Get Approval

Write manifest to `docs/specs/<product>/decompositions/decomposition-{goal}-{nn}.md`

See [manifest-format.md](references/manifest-format.md) for the full field spec.

**Before presenting for approval, declare the recovery path for each Workflow in the manifest:**

| Failure type | Default action |
|-------------|----------------|
| Structural gate failure | Retry the responsible leaf once with the specific finding |
| Semantic gate failure | Targeted fix request, retry once |
| Two consecutive failures | Escalate to user: retry / re-decompose / abort |

If a specific Workflow needs a different escalation path, override it in the manifest. Define this before execution — not when a gate fails.

Present to the user:

> "Decomposition complete: [N leaves, tool breakdown].
> [Leaf list with tool, dependencies, boundary per leaf]
> Shall I proceed with execution?"

Do not execute until the user approves.

---

### Step 7: Execute in Dependency Order

Execute leaves in dependency order. Run leaves in the same parallel group concurrently.

**Direct:** Execute inline. No agents, no planning overhead.

**Inline:** Execute sequentially inline following the numbered step sequence in the brief's `scope`.

**Parallel agents:**
1. Validate each brief (all 5 fields) before spawning
2. Spawn agents concurrently within the parallel group
3. Each agent must report: primary source used or plan draft values
4. You (as coordinator) verify source usage — surface uneven confidence explicitly (do not flatten to a uniform PASS; an agent that used plan draft values while others used primary sources is a risk, not a detail)

**Reviewed task loop:**
1. Invoke `brainstorming` with the leaf brief
2. Invoke `writing-plans` with the brainstorming output
3. Execute via `subagent-driven-development` or `executing-plans`

This is the only path that invokes superpowers.

---

### Step 8: Gate at Contract Boundaries

You run these checks yourself — no subagent is spawned for gating. Read the outputs directly using Claude Code tools and apply the checks below.

**Phase boundary** — after each phase leaf completes:

Both layers required — structural alone is not sufficient:

1. **Structural:** grep / counts / file existence checks against declared `outputs`
2. **Semantic:** whole-diff read scoped to leaf's declared `scope` — stale references, drift, anything structural misses

**Failure recovery:**

| Failure | Action |
|---------|--------|
| Structural | Retry the responsible leaf once with the specific finding |
| Semantic | Targeted fix request, retry once |
| Two consecutive failures | Escalate to user: retry / re-decompose leaf / abort |

**Feature boundary** — after all phases of a Feature complete:

Invoke `neat-util-gate` against Feature contracts (`produces` / `consumes`).

**Cross-feature** — after 2+ Features with dependencies or overlapping scope:

Invoke `neat-util-audit`.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping documentation read | Step 2 before Step 4 — always |
| Using "one subject" as right-sized test | Right-sized = fits one tooling row. One subject can still be too large. |
| Checking tooling row fit after Feature/Workflow split | Q1 (right-sized?) before Q2 (what varies?) at every node |
| Spawning before brief validation | Validate all 5 brief fields first — no exceptions |
| Using superpowers for every leaf | Only reviewed task loop uses brainstorming + writing-plans |
| Running semantic coordinator pass without structural | Both required; structural first |
| Executing before manifest approval | Present manifest, wait for approval |
| Assigning reviewed task loop to an entire requirement | Reviewed task loop is phase-level — decompose further first |
| Parallel brief missing git scoping instruction | Every parallel brief must explicitly state: scope git ops to assigned files, never catch-all add |
| Coordinator reports uniform PASS across parallel agents | Surface uneven confidence — an agent that used draft values is a risk, not a pass |
| Recovery path left undefined until a gate fails | Declare it in the manifest before execution begins (Step 6) |

## Output

`docs/specs/<product>/decompositions/decomposition-{goal}-{nn}.md`

See [manifest-format.md](references/manifest-format.md).
