---
name: neat-delivery-gate
description: Use when verifying Feature contracts at delivery boundaries — called by neat-delivery-planning when a boundary:feature leaf completes to verify produced artifacts match their declared contract intent
---

# Feature Contract Gate

**Role:** You are a QA coordinator verifying Feature delivery contracts at boundaries. Called by neat-delivery-planning at Feature boundaries (Step 8) — not a standalone session tool.

## When to Use

Invoked by neat-delivery-planning when a leaf with `boundary: feature` completes. Receives the Feature's `produces` and `consumes` fields from the planning manifest and verifies them.

**Not for standalone invocation** — this skill has no user-facing entry point. It is called mid-execution by the planning coordinator.

## Quick Reference

| Step | What |
|------|------|
| 1 | Receive Feature contract (produces/consumes from manifest) |
| 2 | Parse produces entries: extract file path + intent description |
| 3 | Structural check — verify each produces artifact exists and is non-empty |
| 4 | Structural check — verify each consumes artifact exists |
| 5 | Semantic check — spawn Haiku subagent per produces artifact with intent + content |
| 6 | Output PASS/FAIL verdict block to coordinator |

## Process

### Step 1: Receive Feature Contract

The calling coordinator provides the Feature's contract fields from the planning manifest:

- `produces` — list of entries, each declaring what this Feature produced (path + intent description)
- `consumes` — list of entries, each declaring what this Feature needed as input (paths only)

Example contract:
```
produces:
- neat-delivery-gate/SKILL.md — redesigned gate that accepts Feature contracts, runs structural and semantic checks, outputs PASS/FAIL per artifact
consumes:
- neat-delivery-planning/SKILL.md
- neat-delivery-planning/references/manifest-format.md
```

### Step 2: Parse Produces Entries

Each produces entry combines file path and intent description, separated by ` — ` (space–em dash–space).

Parse each entry into:
- `path` — the file to read (everything before ` — `)
- `intent` — what the artifact should contain (everything after ` — `)

If an entry cannot be parsed (no ` — ` separator found), mark it as a parse error: FAIL with note "could not parse path/intent from entry: {entry}". Continue to remaining entries.

### Step 3: Structural Check — Produces

For each produces artifact:

1. Read the file at `path` using the Read tool
2. File does not exist → FAIL (structural): "file not found: {path}"
3. File is empty → FAIL (structural): "file is empty: {path}"
4. File exists and non-empty → PASS (structural)

Do not proceed to semantic check (Step 5) for any artifact that fails structural.

### Step 4: Structural Check — Consumes

For each consumes artifact:

1. Read the file at the declared path
2. File does not exist → FAIL (structural): "consumed artifact missing: {path}"
3. File exists → PASS

Note: content is not evaluated — consumes artifacts were verified by prior Feature gates. Existence check only.

### Step 5: Semantic Check — Produces

For each produces artifact that passed structural check:

Spawn a subagent (Agent tool, `subagent_type: "general-purpose"`, `model: "haiku"`).

**Prompt:**
```
Verify that this artifact satisfies its delivery contract intent.

Intent (what this artifact was declared to produce):
{intent}

Artifact content:
{full file content}

Evaluate whether the artifact content satisfies the stated intent.

Return:
- Verdict: SATISFIED or NOT SATISFIED
- Evidence: cite specific content present that satisfies the intent (if SATISFIED), or describe the specific gap (if NOT SATISFIED). Be concrete — name sections, field names, or missing elements.
```

**Retry:** Retry once on malformed output (missing Verdict line or missing Evidence). Both fail → NOT SATISFIED with note "subagent returned malformed output twice".

**Model note:** Haiku is sufficient — this is mechanical matching of content against an explicit intent description.

### Step 6: Output Verdict Block

Assemble and return the verdict block to the calling coordinator. Do not write to any file — planning owns the execution log.

```
## Feature Gate: {feature-name} | {date}

### Structural Checks

| Artifact | Type | Result | Note |
|----------|------|--------|------|
| {path} | produces | PASS/FAIL | {note if FAIL, blank if PASS} |
| {path} | consumes | PASS/FAIL | {note if FAIL, blank if PASS} |

### Semantic Checks (produces only)

| Artifact | Verdict | Evidence |
|----------|---------|----------|
| {path} | SATISFIED/NOT SATISFIED | {evidence or gap description} |

### Overall: PASS / FAIL

{If FAIL: list each failing artifact with its specific failure reason}
```

**Verdict logic:**
- PASS: all produces artifacts pass structural + semantic; all consumes artifacts pass structural
- FAIL: any artifact fails structural, OR any produces artifact is NOT SATISFIED

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Writing gate results to a log file | Do not write to any file — return verdict block to coordinator only |
| Evaluating consumes artifact content | Existence check only — content was verified by prior Feature gates |
| Skipping semantic check when structural passes | Both checks required for produces artifacts |
| Running one subagent for all artifacts | Spawn one subagent per artifact — context bleed reduces accuracy |
| Inferring SATISFIED from file existence | Semantic check must evaluate content against intent — existence alone is not SATISFIED |
| Treating parse error as a warning | Parse error = FAIL for that artifact — cannot verify what cannot be parsed |
