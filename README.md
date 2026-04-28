# go-style-uber

Executable Go coding standard extracted from the [Uber Go Style Guide](https://github.com/uber-go/guide) for automated code review, static analysis, and AI-assisted refactoring.

## What This Is

This project converts the prose-based Uber Go Style Guide into a **machine-enforceable rule system**. Each rule is:

- **Actionable** — can be checked by a program or AI
- **Unambiguous** — uses "MUST" / "MUST NOT", not "should" / "prefer"
- **Fixable** — every rule includes a concrete fix strategy

## Quick Start

### Files

| File | Purpose |
|---|---|
| `SKILL.md` | Complete rule definitions (72 rules across 9 categories) with meta rules |
| `manifest.json` | Machine-readable metadata (version, categories, tags) |

### Rule Format

Each rule in `SKILL.md` follows a uniform structure:

```
- ID: RULE-XXX-NNN
- Name: short rule name
- Description: what the rule enforces
- Bad: anti-pattern (optional)
- Good: correct pattern (optional)
- Fix Strategy: how to remediate (required)
```

### Using with AI Code Review

Feed `SKILL.md` into your AI tool or linter pipeline as a rule reference. The rule IDs (e.g., `RULE-ERR-001`) can be used to map violations to specific remediation strategies.

### Using with Static Analysis

Map rule IDs to linter configuration. Example mappings:

| Rule ID | Applicable Linter |
|---|---|
| RULE-ERR-001 | `revive` (unhandled-error), `errcheck` |
| RULE-CONC-002 | custom `govet` / `staticcheck` rule |
| RULE-STYLE-006 | `goimports` |
| RULE-STYLE-008 | `go vet` (fieldalignment, composites) |
| RULE-STYLE-012 | `go vet` (printf) |
| RULE-PERF-001 | `prealloc` |
| RULE-PERF-002 | `staticcheck` (S1025) |

## Rule Summary

**72 rules** organized into 9 categories:

| Category | Prefix | Count | Key Topics |
|---|---|---|---|
| Error Handling | `RULE-ERR-` | 10 | no panic, error-once, wrapping, sentinels, naming, exit control |
| Interface Design | `RULE-IFACE-` | 4 | no pointer-to-interface, compile-time checks, no public embedding |
| Concurrency | `RULE-CONC-` | 8 | atomics, channel sizing, goroutine lifecycle, mutex handling |
| Memory Safety | `RULE-MEM-` | 4 | defensive copy, no mutable globals, byte conversion caching |
| API Design | `RULE-API-` | 6 | functional options, time types, struct tags |
| Code Style | `RULE-STYLE-` | 29 | naming, imports, declarations, struct init, slices/maps, nesting |
| Testing | `RULE-TEST-` | 3 | table-driven tests, test simplicity, parallel capture |
| Performance | `RULE-PERF-` | 2 | container capacity, strconv |
| Structural Patterns | `RULE-STRUCT-` | 2 | init avoidance, defer cleanup |

## Design Philosophy (Meta Rules)

1. **Explicit over implicit** — no hidden side effects
2. **Control resource lifetimes** — every resource has a deterministic cleanup path
3. **Avoid shared mutable state** — inject state, don't share it globally
4. **Errors are values** — return them, don't panic; handle exactly once
5. **Interfaces belong to consumers** — define where used, keep minimal
6. **Consistency above all** — one style per package, no exceptions
7. **Reduce cognitive load** — minimize nesting, reduce scope, early returns
8. **Defensive at boundaries** — copy data at API edges, never expose internals
9. **Zero values must be useful** — types work without initialization
10. **Test readability matters** — simple tables, no branching logic

## Metadata

| Field | Value |
|---|---|
| Name | `go-style-uber` |
| Version | 1.0.0 |
| Author | kaiiyvwu |
| Language | Go |
| Source | [uber-go/guide](https://github.com/uber-go/guide) |
| License | MIT |
| Categories | error-handling, interface-design, concurrency, memory-safety, api-design, code-style, testing, performance |
| Tags | go, style-guide, code-review, linting, uber |

## License

MIT
