# Tooling Selection

Decision lookup for assigning a tool to a leaf node. Apply after confirming the node is right-sized (fits one row exactly). If it fits multiple rows, the node is not yet right-sized — decompose further.

## The Table

| Row | Situation | Tool | Gate |
|-----|-----------|------|------|
| 1 | Single atomic action. One file or one config change. No setup, no state dependency. | **Direct** | `none` |
| 2 | Sequential steps, ≤5 steps, bounded context (one component or tightly coupled files). Correct approach is known. | **Inline** | `phase` boundary |
| 3 | N independent tasks (2–8), correctness fully specifiable in a brief upfront. Each task has no dependency on siblings. | **Parallel agents** | `phase` boundary |
| 4 | Correct approach cannot be specified upfront. Requires discovery, exploration, or expert judgment mid-task. | **Reviewed task loop** | `feature` boundary |

## Fit Test

A node fits a row when you can answer YES to every qualifier in that row's Situation column without ambiguity.

**Fits multiple rows?** Not right-sized — decompose further.

**Fits no rows?** Requirements are underspecified — clarify before proceeding.

## Heuristics

### Direct

- The action is one git operation, one file write, one config edit, or one command.
- You could describe it in one sentence and a competent engineer could do it without asking questions.
- Examples: "Add environment variable X to .env.example", "Update version field in package.json".

### Inline

- Up to 5 sequential steps where each step informs the next.
- All steps stay in bounded context (one module, one service, one feature file).
- No step spawns parallel branches.
- Examples: "Refactor auth middleware to extract token validation", "Add logging to request handler".

### Parallel agents

- Each task is independent — starting task B does not require output from task A.
- You can write a complete brief for each task before any agent starts.
- The correctness standard is explicit: "compare against [source], do not use plan draft values."
- Keep N ≤ 8. Above 8, consider whether tasks share enough context to merge, or whether the node needs further decomposition.
- Examples: "Migrate 6 API endpoint tests to new fixture format", "Add type annotations to 4 independent service modules".

### Reviewed task loop

- The task requires judgment about approach that cannot be resolved upfront.
- Output quality depends on iteration — a single pass is unlikely to be correct.
- The reviewer needs to see intermediate output to guide the next step.
- Examples: "Design the caching strategy for the recommendation engine", "Write the onboarding copy for the empty state".

## Size Hints from Scoping

| Scoping signal | Likely tool |
|---------------|-------------|
| Effort: trivial, 1 file | Direct |
| Effort: small, 2–5 steps, one component | Inline |
| Effort: medium, N independent mechanical tasks | Parallel agents |
| Effort: large or unknown, correctness unclear | Reviewed task loop |
| Risk: high (no established pattern) | Reviewed task loop |
| Risk: low (established pattern, verifiable brief) | Parallel agents or Inline |

## Superpowers Usage

Only **Reviewed task loop** invokes superpowers:

1. `brainstorming` with the leaf brief
2. `writing-plans` with brainstorming output
3. `subagent-driven-development` or `executing-plans`

Direct, Inline, and Parallel agents do NOT invoke superpowers. This is the performance fix — most leaves are not reviewed task loops.
