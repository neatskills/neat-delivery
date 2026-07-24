# Accessing Outputs

**neat-delivery works independently** - reads output files from docs/specs/ directly. Optional: Install [neat-knowledge](https://github.com/neatskills/neat-knowledge) for 80-90% context savings (see [auto KB pattern](neat-knowledge.md)).

## Core Principle

| Mode | When | Result |
|------|------|--------|
| **Direct reads** | Always works | Read specs.md + docs/specs/ files. Higher context usage. |
| **Agent-driven** | .index/metadata.json exists in docs/knowledge/ | Natural language queries, intelligent ROI-based loading. 80-90% context savings. |

Both produce identical results. Agent-driven discovery is a performance optimization.

## Extract Mode (Agent-Driven Discovery)

Command: `/neat-knowledge-extract "<natural language query>"`

Returns structured JSON for automation.

| Mode | Use For |
|------|---------|
| **Extract** | Automated workflows, programmatic extraction, non-interactive queries |
| **Ask** | Manual research, exploratory investigation, synthesis |

### Interactive KB Queries During Workflows

Skills query KB during execution to answer questions before prompting user:

**Pattern:**

1. Skill has clarifying question
2. Query: `neat-knowledge-extract "relevant query"`
3. Parse JSON: extract content from documents[].content
4. If KB has answer → Use it, inform user "Based on {source}: ..."
5. If KB silent → Ask user

**How it works:**

1. Search returns 20-30 keyword matches with metadata (summaries, sections, token costs, tags)
2. Agent evaluates relevance and optimal loading depth (summary/sections/full)
3. Returns JSON with loaded content based on ROI

**Key benefits:** Semantic relevance filtering, ROI optimization, adaptive depth loading, no hard-coded section names.

### Examples

| Query | Loaded | Savings |
|-------|--------|---------|
| "What is the tech stack, architectural components, integrations, and main workflows?" | Summary (2.5K) | 70% vs full file (8K+) |
| "What are the architectural components and identified risks?" | Sections (1.8K) | Combined with domain query: 76% vs direct reads |
| "What domain knowledge is available?" | Summaries (600 tokens) | |

### Response Structure

**Key fields:**

- `documents[]` - Array of loaded documents
  - `filename` - Document filename
  - `title` - Document title
  - `category` - Category (analysis/domains/features/adrs)
  - `loaded` - Depth loaded (summary/sections/full)
  - `content` - Actual content based on agent decision
  - `tokens` - Token costs for ROI tracking
  - `tags` - Document tags
- `loading_strategy` - Overall strategy (summaries/sections/full/mixed)
- `tokens_loaded` - Total tokens loaded
- `total` - Number of documents returned

## Ask Mode (For Humans)

Command: `/neat-knowledge-ask <question>`

Returns natural language answer with source citations. For manual research and exploratory investigation.

## KB State Definitions

| State | Definition |
|-------|------------|
| **Minimal** | specs.md exists, no Analysis or Domain Knowledge entries |
| **Partial** | specs.md has Analysis, no Domain Knowledge |
| **Full** | specs.md has both Analysis and Domain Knowledge |

Skills check KB state in specs.md Outputs section to determine behavior.

## Agent-Driven Discovery Benefits

1. Skills ask natural language questions
2. Search returns 20-30 matches with metadata
3. Agent evaluates relevance and loading depth (summary/sections/full)
4. Content cached in conversation
5. Future queries reuse cache (0 tokens)

**Savings:** 70-85% vs direct reads. Multi-query conversations reach 90%.

## Usage in Skills

**Standard pattern:**

Check .index/metadata.json exists in docs/knowledge/?

- **YES:** Invoke `neat-knowledge-extract "<query>"` - returns JSON (80-90% savings)
- **NO:** Read specs.md, parse KB entries, read docs/specs/ files directly

Both paths produce same data structure.

## Anti-Patterns

| Don't | Do |
|-------|-----|
| Require neat-knowledge for functionality | Make optional, always support direct reads |
| Fail if neat-knowledge unavailable | Check availability, fall back to direct reads |
| Hard-code section names (L1, L2, L3) | Use natural language queries |
| Pre-filter results or prescribe depth | Trust agent to evaluate relevance and optimize depth |
| Load files multiple times | Trust conversation caching |

## Formulating Effective Queries

| Good (Specific) | Avoid (Vague/Prescriptive) |
|-----------------|----------------------------|
| "What is the tech stack, architectural components, integrations, and main workflows?" | "Tell me about the system" (too broad) |
| "What authentication patterns, token handling, and security decisions exist?" | "Load everything" (defeats optimization) |
| "What architectural decisions have been made?" | "Get analysis L1-L6" (hard-coded sections) |

**Multiple focused queries > one broad query.** Agent optimizes each, caches results, avoids loading unneeded content.
