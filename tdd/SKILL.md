---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** exercise real code paths through public APIs. They describe *what* the system does, not *how*. A good test survives refactors because it doesn't care about internal structure.

**Bad tests** are coupled to implementation — they mock internal collaborators, test private methods, or break when you rename an internal function without changing behavior.

## Workflow

### 1. Planning

Before writing any code:
- [ ] Confirm with user what interface changes are needed
- [ ] Confirm which behaviors to test (prioritize critical paths)
- [ ] List the behaviors to test — not implementation steps
- [ ] Get user approval on the plan

**You can't test everything.** Focus on critical paths and complex logic.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing:
```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

### 3. Incremental Loop

For each remaining behavior:
```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:
- One test at a time
- Only enough code to pass the current test
- Don't anticipate future tests

### 4. Refactor

After all tests pass:
- [ ] Extract duplication
- [ ] Simplify interfaces
- [ ] Run tests after each refactor step

**Never refactor while RED.**

## Anti-Pattern: Horizontal Slices

Do NOT write all tests first, then all implementation. This produces tests that verify imagined behavior and break on valid refactors.

Correct: `RED→GREEN: test1→impl1`, `RED→GREEN: test2→impl2`, …

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```
