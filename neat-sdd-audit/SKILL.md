---
name: neat-delivery-audit
description: Use when multiple features are implemented and need cross-feature verification - checks dependency integration, blast area coordination, implementation gaps, and pattern consistency - requires implemented features
---

# neat-delivery Cross-Feature Audit

**Role:** You are a QA engineer who verifies cross-feature integration, coordination, and consistency after implementation.

**Usage:** `neat-delivery-audit <product>` or `neat-delivery-audit` (will prompt for product)

**Requires:** Multiple features with `state: implemented` in `docs/specs/<product>/features/`

**Not for:** Single feature verification (use `neat-delivery-gate`), pre-implementation planning (use `neat-delivery-planning`)

## Overview

Cross-feature integration verification: dependency integration, blast area coordination, implementation gaps, pattern consistency. Severity-ranked findings (ERROR/WARNING/INFO).

**Scope:** Single feature → `neat-delivery-gate`; Multi-feature → `neat-delivery-audit`

## When to Use

After implementation when: features have `depends_on`, overlapping blast areas, batch completes, before milestones.

**Not for:** Single feature (`neat-delivery-gate`), pre-implementation (`neat-delivery-planning`).

## Quick Reference

| Step | What |
|------|------|
| 1 | Setup: locate specs, load features, determine which checks apply |
| 2 | Run applicable checks (1-4) |
| 3 | Present audit report with severity-ranked findings |
| 4 | Handle user choice: fix (recommend actions), accept (log rationale), done |
| 5 | Save audit report to `docs/specs/<product>/audit.md` |

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| ERROR | Integration broken, gaps, or conflicts | Should address before merge/release |
| WARNING | Inconsistencies or weak coordination | Review recommended |
| INFO | Verification passed or awareness only | No action needed |

All severities are advisory - user decides whether to address.

## Setup

1. Locate specs.md ([procedure](../references/specs-location.md)), read Outputs for features path
2. Glob `feature-*.md`. If no `state: implemented` → STOP: "No implemented features found"
3. Load per feature: frontmatter (name, goal, state, depends_on), blast area (components, precision), acceptance criteria
4. Determine applicable checks (see Checks)
5. Construct output path ([rules](../references/output-conventions.md))

## Checks

Run all checks with available inputs. Skip others, note in report.

### Check 1: Dependency Integration Verification

**Inputs:** Features with `depends_on` (≥1 pair) | **Skip:** No `depends_on`

**Verifies:** Dependent features integrate with dependencies in code

**Algorithm:** Build pairs → query KB for exports → grep imports/usage → verify API patterns → classify

| Severity | Condition |
|----------|-----------|
| ERROR | No imports or wrong API usage |
| WARNING | Weak coupling |
| INFO | Correct integration |

### Check 2: Blast Area Overlap Coordination

**Inputs:** Overlapping blast areas (≥1) | **Skip:** No overlaps

**Algorithm:** Build map → find overlaps → query KB approaches → detect conflicts vs coordination

| Severity | Condition |
|----------|-----------|
| ERROR | Conflicting approaches |
| WARNING | Different modifications without coordination |
| INFO | Coordinated (referenced in dependencies/risks) |

### Check 3: Implementation Gap Detection

**Inputs:** Acceptance criteria, implementations (≥2 features) | **Skip:** < 2 features

**Algorithm:** Grep criteria for integration keywords → verify via KB queries

| Severity | Condition |
|----------|-----------|
| ERROR | Criteria requires integration but code has no connection |
| WARNING | Related features without integration |
| INFO | Integration points match spec |

### Check 4: Cross-Feature Pattern Consistency

**Inputs:** Multiple features per domain (≥2) | **Skip:** < 2 per domain

**Algorithm:** Group by domain → query KB patterns → check divergence intentionality

| Severity | Condition |
|----------|-----------|
| WARNING | Patterns differ without rationale |
| INFO | Intentional or consistent patterns |

## Process

### Step 1: Run Checks

Execute checks 1-4 per input availability. Collect findings. Note skipped checks with reasons.

### Step 2: Present Report (BLOCKING)

**Format:**

```markdown
# Audit Report: <product>
Date: YYYY-MM-DD HH:MM
Features Audited: N features (M implemented)

## Summary
- Errors: N
- Warnings: N
- Info: N
- Checks Skipped: [list with reasons]

## Check 1: Dependency Integration
[Table if run, or "SKIPPED: No features with depends_on"]

## Check 2: Blast Area Overlaps
[Table if run, or "SKIPPED: No overlapping blast areas"]

## Check 3: Implementation Gaps
[Table if run, or "SKIPPED: < 2 implemented features"]

## Check 4: Pattern Consistency
[Table if run, or "SKIPPED: < 2 features per domain"]

## Verdict
[PASS | FAIL] - [Summary of ERRORs if FAIL]
```

Present findings, ask: "Proceed? **Fix** | **Accept** | **Done**"

### Step 3: Handle Choice

**Fix:** Recommend actions:

| Finding Type | Recommended Action |
|--------------|-------------------|
| Dependency integration broken | Fix integration in dependent feature, re-run `neat-delivery-gate` |
| Blast area conflict | Coordinate features - may need refactoring, re-run `neat-delivery-gate` for affected features |
| Implementation gap | Implement missing integration, update acceptance criteria verification |
| Pattern inconsistency | Align patterns or document intentional divergence in Technical Decisions |

**Accept:** Get one-line rationale per finding, log in report.

**Done:** Save as-is.

### Step 4: Save Report

Save to `docs/specs/<product>/audit.md` (overwrite). Register in specs.md Outputs ([format](../references/output-conventions.md)): `- Audit: docs/specs/<product>/audit.md`

## Red Flags

Stop rationalizing - run full checks:

- "Gates passed → integration fine"
- "Tests pass → integrates correctly"
- "Time pressure → skip checks"
- "Reviewed → no verification needed"
- "Just check docs consistency"
- "Quick sanity check enough"
- "Tired → rubber-stamp"

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Checking only docs/tests | Verify implementation integration in code |
| Skipping code reading | Read implementations, don't infer from specs |
| Running before implementation | Requires `state: implemented` |
| Using for single feature | Use `neat-delivery-gate` instead |
| Inferring from proximity | Verify imports/usage explicitly |

## KB Registration

Register per [standard format](../references/output-conventions.md): `- Audit: docs/specs/<product>/audit.md`

## Output

`docs/specs/<product>/audit.md`
