# Neat Delivery

Markdown-based skills for spec-driven development. Optimised for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), but all output is plain Markdown — so it works with any AI coding agent or human dev team.

## What Makes Neat Delivery Different

**1. Strict Guardrails** — Audit queries the knowledge base to verify cross-cutting consistency across planning, features, domain knowledge, and coverage after changes. Gate enforces spec alignment at build transitions (brainstorming → plan → execute) using risk-based independent AI review. Low-risk features skip gates; medium/high-risk features get verified. Both prevent drift and catch errors humans miss.

**2. Self-improving Knowledge Base** — specs.md indexes analysis reports, domain knowledge, planning, and features as queryable knowledge sources. The knowledge skill's Q&A mode synthesizes answers from KB, code, internet, and library docs — then saves knowledge-generating conversations as FAQ documents that become part of the KB. Every question that uncovers new insights grows the knowledge base. Build phases query this KB to enrich design decisions with architectural patterns and domain insights. Code remains the source of truth; KB provides guidance.

**3. Context Bloat Control** — Contextual extraction keeps token usage predictable as projects scale. The knowledge skill acts as a smart reader: when other skills need data, it reads full reports and extracts only what's requested (acceptance criteria, risks, architecture, etc.). Each extraction is tailored to the caller's needs. A 50-document KB stays manageable because skills never load full reports directly — they request focused extracts through the knowledge skill.

## Workflow

```text
           Neat Delivery - shape                         Neat Delivery - build
┌────────────────────────────────────────┐    ┌──────────────────────────────┐
│                                        │    │                              │
│  analysis → domains → planning ────────┼────┼─→ feature docs → any agents  │
│     │           │         │            │    │         │                    │
│     ▼           ▼         ▼            │    │         ▼                    │
│ Understand   Deepen   Decompose        │    │    design      [gate]        │
│ the product  domain   into features    │    │         │                    │
│              knowledge with goal,      │    │         ▼                    │
│                       deps, risks      │    │    plan        [gate]        │
│                                        │    │         │                    │
│                                        │    │         ▼                    │
│                                        │    │    execute     [gate]        │
│                                        │    │         │                    │
│                                        │    │         ▼                    │
│                                        │    │    audit (after 2+ features) │
│                                        │    │                              │
└────────────────────────────────────────┘    └──────────────────────────────┘

                     Neat Delivery - context
              ┌──────────────────────────────────────────────┐
              │                                              │
              │      knowledge         adr         changes   │
              │          │              │             │      │
              │          ▼              ▼             ▼      │
              │      Q&A for        Record        Change     │
              │        any           arch.         notes     │
              │      audience      decisions        git      │
              │                                              │
              └──────────────────────────────────────────────┘
```

The pipeline ensures you **understand before you plan, and plan before you build**.

## Skills & Roles

Each skill operates with a specific role persona that guides its behavior and outputs:

| Skill | Role |
|-------|------|
| **neat-delivery-analysis** | Software architect who extracts actionable insights from codebases |
| **neat-delivery-planning** | Product owner who clarifies ambiguous goals and decomposes them into discrete, build-ready features |
| **neat-util-domains** | Domain expert who builds domain knowledge through code investigation and focused analysis |
| **neat-delivery-build** | Tech lead who orchestrates feature builds from design through verified code |
| **neat-delivery-gate** | QA engineer who verifies implementation alignment against feature specifications |
| **neat-delivery-audit** | QA engineer who verifies cross-feature integration, coordination, and consistency |
| **neat-util-adr** | Software architect who documents architectural decisions in MADR format |
| **neat-util-changes** | Project manager who distills complex git history into clear, audience-appropriate change notes |
| **neat-util-debugging** | Software engineer who diagnoses and fixes bugs through ranked hypothesis testing and rapid iteration in isolated worktrees |

### Shape Phase (requirements → feature docs)

A human-in-the-loop funnel — each step produces progressively deeper understanding, and you review and steer at every step. No automated gates here; the protection comes from the sequential flow itself, where weak output from one step is naturally surfaced by the next.

1. **Analysis** (`neat-delivery-analysis`) — Starting point for any product. Analyzes **existing codebases** through multi-level (L0–L6) analysis to extract structured knowledge about architecture, tech stack, patterns, and conventions. Outputs an analysis report and a `specs.md` (product metadata and knowledge base path map). Can analyze reference repositories for learning.

2. **Domains** (`neat-util-domains`) — Builds domain knowledge through focused investigations of specific topics within technical or business domains. Investigates topics through code exploration, analysis synthesis, and domain-specific research that merge into living domain knowledge files. Each domain gets one growing file with Overview, Investigations, and Change Log.

3. **Planning** (`neat-delivery-planning`) — Takes high-level goals or objectives and breaks them down into discrete, build-ready features. Clarifies ambiguous goals via KB-guided questions, synthesizes capabilities from product analysis, cross-checks against architecture, and automatically derives goal statements, dependencies, and risks for each feature. Produces feature files with `state: planned` ready for build.

### Build Phase (feature docs → working code)

This is where spec gates kick in. The feature doc from planning, enriched with acceptance criteria during design, becomes the contract, and every transition is verified against it — preventing drift between intent and execution.

**Build** (`neat-delivery-build`) — Orchestrates end-to-end feature implementation from design through verified code with risk-based gate verification. Requires existing feature docs from planning. Queries the knowledge base to enrich brainstorming with architectural patterns and domain insights.

- **Readiness check** → evaluates doc quality (components, risks, goal), discovers blast area files
- **Brainstorming** → queries KB for patterns and domain knowledge, reads code in blast area, produces design spec AND derives acceptance criteria from chosen design approach
- **Update feature doc** → adds designed date, spec_doc path, and acceptance criteria to feature file
- **Writing Plans** → breaks design into TDD implementation tasks with exact file paths, splits if >15 tasks
- **Risk Assessment + Plan Gate** → analyzes design complexity (task count, keywords, dependencies, blast area size) and runs gate if medium/high risk to verify design spec + task plan together
- **Execute** → produces working code via grouped parallel, subagent-driven, or inline execution
- **Risk Assessment + Execute Gate** → analyzes implementation (diff size, modified files, critical paths) and runs gate if medium/high risk to verify code against acceptance criteria
- **Status update** → updates feature doc to `state: implemented`
- **Audit prompt** → after 2+ features with relationships, prompts to run cross-feature verification

Gates use independent AI review (Haiku-powered) for fast, cost-effective verification against acceptance criteria. Risk assessment prevents unnecessary gates for simple features while ensuring thorough verification of complex or critical changes.

**Audit** (`neat-delivery-audit`) — Final cross-feature verification after multiple features are implemented. Checks dependency integration, blast area coordination, implementation gaps, and pattern consistency across features. Should be run after gates complete for individual features.

### Context Phase

Independent skills usable at any point — no fixed order, no build dependency. All require analysis output (specs.md).

**Knowledge** (`neat-knowledge-extract`, `neat-knowledge-ingest`) — Query and build your knowledge base. **Search mode** for fast keyword lookups across metadata. **Ask mode** for interactive research with progressive source loading and synthesis. **Ingest** converts web pages, PDFs, Office documents, ZIP archives, images, and text into structured markdown with automatic analysis and indexing. When installed, neat-delivery automatically manages the project KB (auto-ingests analysis, domains, ADRs, implemented features, and change notes). User must initialize KB once via `/neat-knowledge-ingest <any-file>`. Separate sibling project at [neat-knowledge](https://github.com/neatskills/neat-knowledge).

**Changes** (`neat-util-changes`) — Generates curated, audience-appropriate change notes from git history. Categorizes commits using conventional commit prefixes or AI, summarizes in the right tone, outputs markdown and terminal-ready text.

**ADR** (`neat-util-adr`) — Two modes: **Standalone Mode** for conversational ADR creation. **Extraction Mode** for pulling architectural decisions from design specs' "Key Decisions" sections. Generates formal ADRs in MADR format with shared sequential numbering. Can be invoked manually or called by other skills.

## Standalone Skills

Independent utility skills that can be called standalone or as part of other workflows:

| Skill | Purpose | Usage |
| ----- | ------- | ----- |
| `neat-knowledge-extract` | Search or research knowledge base with progressive loading and synthesis | Standalone for Q&A or called by other skills for KB access |
| `neat-knowledge-ingest` | Convert web/PDF/Office/ZIP/images to structured markdown with indexing | Standalone for adding content to KB |
| `neat-util-adr` | Create or extract architectural decisions as formal ADRs in MADR format | Standalone conversational mode or extraction mode (can be called by other skills) |
| `neat-delivery-gate` | Independent AI review against feature doc acceptance criteria | Called by `neat-delivery-build` to verify design/plan/code |
| `neat-delivery-audit` | Final cross-feature verification after multiple features implemented | Run after 2+ features with relationships to verify integration, coordination, consistency |
| `neat-util-changes` | Generate audience-appropriate change notes from git commit history | Standalone for release notes, changelogs |
| `neat-util-debugging` | AI-powered debugging with ranked hypotheses, worktree isolation, and mandatory user alignment before code changes | Standalone for any bug, error, test failure, or unexpected behavior |

## Install / Uninstall

Neat Delivery is installed as a complete skill set:

```bash
./scripts/manage-skills.sh              # Install all skills
./scripts/manage-skills.sh uninstall    # Uninstall all skills
```

The scripts automatically install all skills found in `neat-*` folders.

**Note:** Knowledge base skills (`neat-knowledge-extract`, `neat-knowledge-ingest`, `neat-knowledge-rebuild`) are installed separately from the [neat-knowledge](https://github.com/neatskills/neat-knowledge) project. When installed, neat-delivery automatically manages the project KB — auto-ingests analysis, domains, ADRs, implemented features, and change notes. User must initialize KB once via `/neat-knowledge-ingest <any-file>`.

## Output

All skill output goes to `docs/specs/<product>/`. For local targets, this is relative to the target product directory. For URL repos (learning/context only, cloned to `/tmp`), output is saved relative to the neat-delivery product root.

For target repos, the analysis skill generates `specs.md` (repos, tech stack, architecture, conventions, commands, knowledge base path map) at `docs/specs/`. All downstream skills read `specs.md` for product context and file locations.

## Project Structure

```text
neat-delivery/
  CLAUDE.md                          # Project rules and conventions
  README.md                          # This file
  scripts/
    manage-skills.sh                 # Manage skill installation (install/uninstall)
  neat-delivery-<skill>/                  # Skills (pipeline and standalone)

Each skill folder contains:
  SKILL.md, README.md, scripts/     (scripts optional)
```
