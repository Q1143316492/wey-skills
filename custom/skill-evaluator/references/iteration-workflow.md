# Iteration Workflow

How to organize a multi-iteration evaluation workspace.

## Directory structure

```
.github/skills/<skill-name>/evals/
├── test-cases.json
├── baselines/                    # optional; known-good outputs to diff against
│   └── tc-001-expected.md
├── runs/
│   ├── iteration-1/
│   │   ├── _skill-snapshot/      # SKILL.md as it was when this iteration ran
│   │   │   └── SKILL.md
│   │   ├── tc-001/
│   │   │   ├── with_skill/
│   │   │   │   ├── run-1/
│   │   │   │   │   ├── output.md
│   │   │   │   │   └── transcript.md
│   │   │   │   ├── run-2/
│   │   │   │   ├── run-3/
│   │   │   │   └── grading.json  # aggregated grading across the 3 runs
│   │   │   └── without_skill/    # or old_skill/ when improving an existing skill
│   │   │       └── run-1/ ...
│   │   ├── tc-002/
│   │   └── benchmark.json        # iteration-level aggregate (see schemas.md)
│   └── iteration-2/
│       └── ...
└── history/
    ├── 2026-06-04-v1.0.md
    └── 2026-06-05-v1.1.md
```

## Per-iteration steps

1. **Snapshot the skill** before iterating. Copy current SKILL.md (and `references/`, `scripts/` if used) into `runs/iteration-N/_skill-snapshot/`. This makes the iteration reproducible later.
2. **Run all (test_case × configuration × run_number) combinations** in the same subagent batch when possible. Don't run with_skill first and baselines later — same-batch ensures comparable conditions.
3. **Grade each run** (inline or with the grader subagent). Save `grading.json` per run, plus an aggregated `grading.json` per `tc-XXX/<config>/`.
4. **Aggregate** into `runs/iteration-N/benchmark.json` matching the schema in [schemas.md](./schemas.md).
5. **Write the iteration report** to `history/<date>-v<version>.md` with diff vs the previous iteration.

## Choosing number of runs per case

| Runs | Use for | Variance signal |
|---|---|---|
| 1 | Quick smoke test, throwaway experimentation | None |
| 3 | **Default for iteration runs** | Detects obvious flaky cases |
| 5+ | Investigating a suspected flaky case, or before declaring a regression | Stronger variance estimates |

3 is a good default. Don't go lower — a single run can't distinguish "regressed" from "got unlucky once".

## What changes between iterations

Across iterations, **keep stable**:
- Test case prompts (changing them invalidates comparisons)
- Baseline configuration
- Grading rubric

Across iterations, **expect to change**:
- The skill's SKILL.md and bundled resources
- The expected behaviors (only when the grader's `eval_feedback` flags genuinely weak assertions)
- Adding new test cases (from real bugs); never removing them

If you change a test case mid-iteration, mark it (`"changed_in_iteration": N`) so the diff in the next report accounts for it.

## When to stop iterating

Stop when any of:
- The user says they're satisfied
- The user's review feedback is empty across cases ("looks good" / no comments)
- Two consecutive iterations show no meaningful pass-rate improvement
- You're making changes that improve one test case at the cost of another (overfitting — widen the suite instead)

Don't stop because "the numbers are pretty good" if the user hasn't reviewed the actual outputs. Numbers are a proxy.

## Cross-iteration tracking

The report in `history/<date>-v<version>.md` should always include a diff section against the previous iteration:

| Case | Prev | This | Change |
|---|---|---|---|
| tc-001 | PASS | PASS | stable |
| tc-002 | FAIL | PASS | **fixed** |
| tc-003 | PASS | FAIL | **regression** |
| tc-004 | 67% (2/3) | 100% (3/3) | reduced flakiness |

Regressions block the iteration — investigate before declaring an improvement.
