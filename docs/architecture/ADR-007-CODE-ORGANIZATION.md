# ADR-007: Code Organization - Human-Readable File Sizes

**Status:** Accepted
**Date:** 2025-12-06
**Author:** Claude Opus 4.5 (Principal Autonomous AI)

---

## Context

Forge follows Rust official guidelines strictly. The Rust Book states:

> "As a project grows, you **should** organize code by splitting it into multiple modules and then multiple files."
> — [The Rust Book, Chapter 7](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html)

### RFC 2119 Interpretation

In technical documentation ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)):

| Term | Meaning |
|------|---------|
| **MUST** | Absolute requirement |
| **SHOULD** | Best practice - recommended, valid reasons may exist to ignore |
| **MAY** | Optional - no opinion, take it or leave it |

### RoyalBit Interpretation

For RoyalBit projects, we interpret RFC 2119 as:

| Term | RoyalBit Action |
|------|-----------------|
| **MUST** | We follow (obviously) |
| **SHOULD** | We follow (best practice = we do it) |
| **MAY** | We don't care (no opinion needed, zero bikeshedding) |

**Why?** Best practices exist for a reason. Experts recommend them. We follow expert recommendations.

### Project Philosophy

RoyalBit has an explicit requirement for **human-readable, beautiful, well-formatted code**. This is not about LLM convenience - it's about:

1. **Developer experience** - Humans read and maintain this code
2. **Code review quality** - Smaller files are easier to review
3. **Onboarding** - New developers can understand modules quickly
4. **Maintainability** - Focused files have clearer responsibilities

### Current State (v5.2.0)

| File | Lines | Status |
|------|-------|--------|
| `evaluator.rs` | 2,918 | Exceeds threshold |
| `commands.rs` | 2,445 | Exceeds threshold |
| `parser/mod.rs` | 2,059 | Exceeds threshold |
| `mcp/server.rs` | 1,219 | Exceeds threshold |
| `excel/exporter.rs` | 1,107 | Exceeds threshold |

## Decision

**For this project, "should" becomes "must".**

We adopt a **1,000-line soft limit** per file as a project standard, following the Rust recommendation strictly.

### Thresholds

| Threshold | Action |
|-----------|--------|
| **< 500 lines** | Ideal - no action needed |
| **500-1,000 lines** | Acceptable - monitor for growth |
| **1,000-1,500 lines** | Evaluate - consider splitting if logical boundaries exist |
| **> 1,500 lines** | Split required - find natural module boundaries |

### Splitting Criteria

Files should be split when:

1. **Logical boundaries exist** - Distinct sections with clear responsibilities
2. **Navigation is impaired** - Hard to find code within the file
3. **Section comments exist** - `// ═══ SECTION NAME ═══` indicates natural split points
4. **Test organization** - Tests for a section could live with that section

Files should NOT be split when:

1. **Tight coupling** - Code is genuinely interdependent (e.g., single match dispatch)
2. **Splitting adds complexity** - Traits/fn pointers needed just for organization
3. **No logical boundaries** - The code is one cohesive unit

### Test Organization

Per Rust convention, unit tests stay with code:

```rust
// In the same file as the code being tested
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_something() { ... }
}
```

When splitting a file, tests move WITH their associated code to the new module.

## Implementation

### Phase 1: commands.rs (2,445 lines)

Each command handler is independent - ideal for splitting:

```
cli/commands/
├── mod.rs         (~200 lines) - shared utilities, re-exports
├── calculate.rs   - calculate() + tests
├── validate.rs    - validate() + tests
├── audit.rs       - audit() + tests
├── export.rs      - export() + tests
├── import.rs      - import() + tests
├── compare.rs     - compare() + tests
├── variance.rs    - variance() + tests
├── sensitivity.rs - sensitivity() + tests
├── goal_seek.rs   - goal_seek() + tests
├── break_even.rs  - break_even() + tests
├── functions.rs   - functions() + tests
├── upgrade.rs     - upgrade() + tests
└── watch.rs       - watch() + tests
```

### Phase 2: parser/mod.rs (2,059 lines)

Split by parsing concern:

```
parser/
├── mod.rs        (~200 lines) - entry point, dispatch
├── single.rs     - single document parsing + tests
├── multi_doc.rs  - multi-document YAML + tests
├── includes.rs   - file includes resolution + tests
├── tables.rs     - table/column parsing + tests
├── scalars.rs    - scalar variable parsing + tests
├── scenarios.rs  - scenario parsing + tests
└── validation.rs - JSON schema validation + tests
```

### Phase 3: evaluator.rs (2,918 lines)

**DEFERRED** - The match dispatch pattern is idiomatic Rust. Splitting would require:
- Complex trait abstractions
- Function pointer indirection
- Loss of compiler optimizations

The file has 16 section comments that provide sufficient navigation. Revisit if a cleaner splitting pattern emerges.

### Enforcement

```yaml
# .asimov/project.yaml
standards:
  file_size:
    soft_limit: 1000
    hard_limit: 1500
    action: "Split into modules when exceeded"
```

## Rationale

### Why Follow "Should" Strictly?

1. **Consistency** - Arbitrary exceptions lead to inconsistent codebase
2. **Quality culture** - High standards attract quality contributors
3. **Future-proofing** - Smaller files are easier to refactor
4. **Code review** - PRs touching smaller files are easier to review

### Why Not Ignore It?

The Rust ecosystem has large files, but:

1. **We're not the Rust compiler** - Different constraints
2. **Human readability matters** - Our explicit project value
3. **Recommendation exists for a reason** - The Rust team knows best practices

### Exception: evaluator.rs

The evaluator's match dispatch is a **valid reason to ignore** the recommendation because:

1. Splitting requires complex traits or fn pointers
2. Single match is more efficient (compiler optimization)
3. Section comments provide adequate navigation
4. The pattern is idiomatic Rust

This is the RFC 2119 use case: "valid reasons may exist to ignore".

## Consequences

### Positive

1. **Human-readable codebase** - Easy to navigate and understand
2. **Better code reviews** - Smaller, focused changes
3. **Clear responsibilities** - Each file has one job
4. **Easier testing** - Tests colocated with focused code

### Negative

1. **More files** - Directory navigation required
2. **Import management** - More `use` statements
3. **Initial refactoring effort** - One-time cost

### Metrics

| Metric | Before | After (Target) |
|--------|--------|----------------|
| Files > 1,000 lines | 5 | 1 (evaluator.rs, excepted) |
| Avg file size | ~800 lines | ~400 lines |
| Max file size | 2,918 lines | 1,500 lines |

## Related

- [ADR-004: 100% Test Coverage](ADR-004-100-PERCENT-TEST-COVERAGE.md) - Tests move with code
- [ADR-006: Coverage Exclusions](ADR-006-COVERAGE-EXCLUSIONS.md) - watch() exclusion
- [The Rust Book - Managing Growing Projects](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html)
- [The Rust Book - Separating Modules](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html)
- [Clippy Lint Configuration](https://doc.rust-lang.org/nightly/clippy/lint_configuration.html)

---

**Previous:** [ADR-006](ADR-006-COVERAGE-EXCLUSIONS.md)
**Next:** [ADR-008](ADR-008-*.md)
