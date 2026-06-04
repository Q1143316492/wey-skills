# <skill-name> evaluation — iteration <N> — <YYYY-MM-DD> — v<X.Y>

**Skill version**: <git ref or SKILL.md version>
**Iteration**: <N>
**Test cases run**: <M>
**Runs per case**: <K> (per configuration)
**Configurations**: with_skill, without_skill (or old_skill)
**Previous report**: [<prev-date>](../history/<prev-date>-v<prev-version>.md) (or "none — first run")

---

## Summary

| Metric | This run | Previous | Delta |
|---|---|---|---|
| Pass rate (with_skill) | X/N (XX%) | Y/N (YY%) | +/- |
| Pass rate (baseline) | X/N (XX%) | Y/N (YY%) | +/- |
| **Delta (skill value)** | +Z pts | +W pts | +/- |
| P0 pass rate | X/N | Y/N | +/- |
| Mean duration (with_skill) | Xs | Ys | +/- |

**Verdict**: improved / unchanged / regressed

---

## Per-test-case results

### tc-001 — <name> — PASS / FAIL / FLAKY

**Prompt**: `<the user prompt>`
**Priority**: P0
**Pass rate**: with_skill 3/3 (100%) — without_skill 1/3 (33%)
**Variance**: stable / flaky (variance > 30%)

| Expected behavior | with_skill | baseline | Evidence |
|---|---|---|---|
| Behavior 1 | 3/3 PASS | 0/3 PASS | `output/foo.md` exists with N lines |
| Behavior 2 | 2/3 PASS | 0/3 PASS | Run 2 created `BadName/`, expected kebab-case |
| Behavior 3 | 3/3 PASS | 3/3 PASS | Quoted: "..." — **non-discriminating, both configs pass** |

**Mean duration**: Xs (with_skill) vs Ys (baseline)
**Notes**: Anything noteworthy that no assertion captured.

---

### tc-002 — <name> — PASS / FAIL

(same structure)

---

## Issues

### #00X — <one-line title> — NEW / OPEN / FIXED
- **Severity**: P0 / P1 / P2
- **First seen**: <date>
- **Test case(s)**: tc-XXX
- **Description**: What goes wrong, why it matters
- **Suggested fix**: Where in the skill to change, or what to investigate

---

## Changes vs previous iteration

- tc-XXX: PASS → FAIL (**regression**) — <hypothesis why>
- tc-YYY: FAIL → PASS (**fixed**) — <what changed>
- tc-ZZZ: 1/3 → 3/3 (**flakiness reduced**)
- tc-AAA: PASS → PASS (stable)

---

## Cross-iteration patterns (from analyzer)

- Assertion "<X>" passes 100% in both configurations — **non-discriminating**, consider strengthening or dropping
- tc-003 has 33% ± 47% variance across 3 runs — investigate or mark `flaky`
- without_skill runs consistently fail on tasks.md format (0/3 pass) — strong evidence skill is the cause of success here

---

## Eval-set feedback (from grader meta-critique)

- tc-001 behavior 3 — Trivially satisfied even on wrong outputs. Suggest replacing with "<stronger assertion>".
- tc-002 — Observed skill produced output not covered by any assertion (e.g. <observation>). Suggest adding "<new assertion>".

---

## Next actions

1. Fix #00X (P0) — apply suggestion from analyzer
2. Strengthen weak assertions flagged in grader feedback
3. Add a new test case covering <gap discovered during this run>
4. Consider updating baseline for tc-YYY (improvement confirmed by user)

