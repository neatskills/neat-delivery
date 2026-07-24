# Output Conventions

Standards for writing and registering skill outputs in neat-delivery.

## Path Construction

All skills writing to `docs/specs/` must follow these path construction rules.

### Path Resolution

1. Locate specs.md per [standard procedure](specs-location.md)
2. Extract product name from specs.md
3. Use standard paths (below)

### Standard Paths

All specs are written to `<repo-root>/docs/specs/<product-name>/` where:

- `<repo-root>` is the git repository root
- `<product-name>` is the product name from specs.md

**Platform analysis:** `<repo-root>/docs/specs/analysis-platform.md` (root level, not in a product folder)

All paths below are relative to `docs/specs/<product-name>/`:

| Type | Path |
|------|------|
| Specs | `specs.md` |
| Analysis | `analysis-<product-name>.md` or `analysis-<ref-name>.md` |
| Domain Knowledge | `domains/` |
| ADRs | `adrs/` |
| Features / Gate Logs | `features/` |
| Audit | `audit.md` |
| Changes | `changes/` |
| FAQ | `faq.md` |

### Path Examples

**Single product "myapp":** `docs/specs/myapp/features/feature-auth-01-oauth-integration.md`

**Platform with "web" product:** `docs/specs/web/features/feature-checkout-01-payment-flow.md`

**Platform with "mobile" product:** `docs/specs/mobile/features/feature-auth-01-biometric-login.md`

## Registration Format

Format for specs.md Outputs section entries: `<Type>: <path>`

### Entry Types

| Type | Path Pattern |
|------|--------------|
| Analysis | `docs/specs/<product>/analysis-<product>.md` |
| ADRs | `docs/specs/<product>/adrs/` |
| Features | `docs/specs/<product>/features/` |
| Domain Knowledge | `docs/specs/<product>/domains/domain-knowledge-<NN>-<name>.md` |
| FAQ | `docs/specs/<product>/faq.md` |
| Gate Logs | `docs/specs/<product>/features/` |

### Format Rules

- Plain text only — no bold, no markdown links
- Relative paths from product root
- Collections: add count suffix `(N features)`
- Domain Knowledge: use parentheses `(NN-name)` after type
- Directories: end with `/`

### Registration Examples

```markdown
Analysis: docs/specs/myapp/analysis-myapp.md
ADRs: docs/specs/myapp/adrs/ (3 ADRs)
Features: docs/specs/myapp/features/ (8 features)
Domain Knowledge (03-auth): docs/specs/myapp/domains/domain-knowledge-03-auth.md
FAQ: docs/specs/myapp/faq.md
```
