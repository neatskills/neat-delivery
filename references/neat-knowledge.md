# Automatic Knowledge Management

When neat-knowledge skills are installed, neat-delivery automatically manages the project knowledge base.

## Skill Installation Check

```bash
test -L ~/.claude/skills/neat-knowledge-ingest && \
test -L ~/.claude/skills/neat-knowledge-extract && \
echo "installed" || echo "not-installed"
```

Result: `installed` → enable auto KB management; `not-installed` → skip KB operations

## KB Detection

Auto-ingest requires initialized KB. User initializes once by running `/neat-knowledge-ingest <any-file>`, which prompts for location and creates:

- `{KB_PATH}/.index/metadata.json` (metadata)
- `{KB_PATH}/.index/index.json` (search index)
- `{KB_PATH}/.index/summaries/` (category summaries)

**Check sequence:**

1. Are neat-knowledge skills installed?
2. Does KB path exist in specs.md?
3. If yes to both: Use for auto-ingestion
4. If no to either: Skip KB operations

**Ownership:** Analysis skill detects KB and writes path to specs.md. All downstream skills read path from specs.md (single source of truth).

## KB Section in specs.md

Single source of truth for KB path. Located after `## Commands`, before `## Outputs`.

**Format:**

| State | Content |
|-------|---------|
| Initialized | `**Path:** docs/knowledge/`<br>`**Status:** Initialized (42 documents)` |
| Not initialized | `**Status:** Not initialized` |

### Detection & Usage

**Detection commands:**

```bash
# Find KB
find . -name "metadata.json" -path "*/.index/metadata.json" -type f 2>/dev/null | head -1

# Count documents
cat <kb-path>/.index/index.json | grep -c '"file_path"'
```

**Analysis skill workflow:**

1. Run find command to detect KB
2. If found: Extract KB directory (parent of `.index/`), convert to relative path, count documents
3. Write specs.md section: `**Path:** {KB_PATH}` + `**Status:** Initialized ({count} documents)`
4. If not found: Write `**Status:** Not initialized`

**Downstream skills workflow:**

1. Read specs.md, locate `## Knowledge Base` section
2. Look for `**Path:**` line, extract value
3. If path exists: Use for ingestion operations
4. If no path: Skip KB operations, continue normally

## Automatic Ingestion Points

| Skill | Trigger | Path | Category | Notes |
|-------|---------|------|----------|-------|
| **neat-delivery-analysis** | After saving analysis + specs | `analysis-<product>.md` | analysis | Detects KB, writes path to specs.md |
| **neat-delivery-domains** | After domain investigation | `domain-knowledge-{NN}-{name}.md` | domains | Reads KB path from specs.md |
| **neat-delivery-adr** | After ADR creation | `docs/specs/<product>/adrs/` (directory) | adrs | |
| **neat-delivery-build** | Feature `state: implemented` | `feature-{goal}-{nn}-{slug}.md` | features | Only implemented features ingested |

### Standard Workflow

**For Analysis skill:**

1. Check installation: `test -L ~/.claude/skills/neat-knowledge-ingest && echo "installed" || echo "not-installed"`
2. If not installed: Skip auto-ingest, write `**Status:** Not initialized` to specs.md
3. If installed: Detect KB path via `find` command
4. If KB found: Write path + count to specs.md, invoke ingest, log success
5. If KB not found: Write `**Status:** Not initialized`, log initialization hint

**For downstream skills (domains, adr, build):**

1. Check installation
2. If not installed: Skip
3. If installed: Read KB path from specs.md `## Knowledge Base` section
4. If path found: Invoke `neat-knowledge-ingest` with appropriate args, log success
5. If path not found: Skip

## Benefits & Fallback

**With KB:** Searchable analysis/domains/ADRs, 80-90% context savings in downstream skills.  
**Without KB:** neat-delivery reads files directly, no optimization. Can install/ingest later.

## Implementation Pattern

All content-generating skills follow this pattern:

1. **Save:** Write file to `docs/specs/<product>/<path>`
2. **Register:** Add to specs.md Outputs section
3. **Auto-ingest:**
   - Check if neat-knowledge skills installed
   - Read KB path from specs.md (or detect for analysis skill)
   - If path exists: Invoke `neat-knowledge-ingest` with file/directory + category
   - Log: "Indexed in project KB" or skip reason

### Error Handling

**KB initialization fails:** Log warning, continue without KB, suggest manual installation.

**Ingestion fails:** Log warning, continue. File already saved/registered in specs.md. User can ingest manually later.

**Skills not installed:** Silent skip. neat-delivery operates independently.

### User Experience

**With neat-knowledge installed:**

- User runs `/neat-knowledge-ingest <any-file>` once to initialize KB
- All subsequent neat-delivery operations auto-index content
- Transparent logging confirms operations
- 80-90% context savings in downstream skills

**Without neat-knowledge:**

- neat-delivery reads files directly
- No KB optimization
- Can install neat-knowledge later and manually ingest existing content
- No failures or blockers
