---
name: skill-evaluator
description: 'Evaluate and iterate on agent skills using a lightweight test-case + baseline comparison workflow. Use when the user wants to test a skill, compare two versions of a skill, score a skill''s outputs, track skill quality over time, design test cases for a skill, or investigate why a skill regressed. Works fully manually — no external dependencies beyond VS Code Copilot and markdown/JSON files.'
---

# Skill Evaluator

A zero-dependency workflow for evaluating VS Code Copilot skills. Adapted from Anthropic's [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) — same ideas, no Python / CLI / browser required.

## When to use

- Testing a newly created or modified skill
- Comparing version A vs version B of the same skill (regression check)
- Tracking skill quality across iterations
- Investigating why a skill regressed
- Designing test cases for a skill (see [references/test-case-design.md](./references/test-case-design.md))
- Optimizing a skill's `description` for trigger accuracy (see [references/description-optimization.md](./references/description-optimization.md))

## Core loop

```
draft / edit skill
   ↓
run test cases  (with-skill + baseline, ideally 3 runs each for variance)
   ↓
grade outputs  (separate "grader" view — see references/grader.md)
   ↓
write iteration report  (evals/history/<date>-<version>.md)
   ↓
diff vs previous report — regressions? fixes? flakiness changes?
   ↓
improve skill, repeat
```

Manual but **structured**. The structure is what separates this from "just try it a few times".

## Layout per skill being evaluated

```
.github/skills/<skill-name>/
├── SKILL.md
└── evals/
    ├── test-cases.json         # Test case definitions (assets/test-cases.template.json)
    ├── baselines/              # Known-good outputs for diffing (optional)
    │   └── tc-001-expected.md
    ├── runs/                   # Raw outputs, organized by iteration
    │   ├── iteration-1/
    │   │   ├── _skill-snapshot/      # SKILL.md as it was for this iteration
    │   │   ├── tc-001/
    │   │   │   ├── with_skill/
    │   │   │   │   ├── run-1/  (output.md, transcript.md)
    │   │   │   │   ├── run-2/
    │   │   │   │   └── run-3/
    │   │   │   └── without_skill/    # or old_skill/ when improving an existing skill
    │   │   └── benchmark.json        # iteration-level aggregate
    │   └── iteration-2/
    └── history/                # One report per iteration
        ├── 2026-06-04-v1.0.md
        └── 2026-06-05-v1.1.md
```

See [references/iteration-workflow.md](./references/iteration-workflow.md) for full directory rationale.

## Procedure

### 1. Load or create test cases

Read [references/test-case-design.md](./references/test-case-design.md) before designing cases — bad cases give false confidence and are the single biggest cause of useless evaluations.

Use the schema in [references/schemas.md](./references/schemas.md#test-casesjson) (template at [templates/test-case-template.json](./templates/test-case-template.json)).

If `evals/test-cases.json` doesn't exist:
- Ask the user for 2–3 realistic prompts they would actually type
- For each, write 3–5 concrete **expected behaviors** (checkable statements — not "looks good")
- Start small. Grow from real bugs, not imagined edge cases

If it exists, confirm with the user whether to use it as-is or extend it.

### 2. Identify what you're testing

Ask the user:
- **Single version** — establish a baseline (no comparison)
- **Comparison** — current vs prior, A vs B, or with-skill vs without-skill

For comparisons against the **prior version**, snapshot the old SKILL.md first into `runs/iteration-<N>/_skill-snapshot/` *before* editing. Otherwise the baseline can't reproduce the old behavior.

Decide how many **repeat runs** per case:
- **1 run** — fast smoke test, no variance signal
- **3 runs** — recommended minimum to detect flaky cases
- **5+ runs** — when you suspect non-determinism is hiding regressions

### 3. Run each test case

For each `(test_case × configuration × run_number)`:

**Option A — Subagent run (preferred when available)**

Use `runSubagent` with the `Explore` agent. Spawn all runs for an iteration **in the same turn** so they execute under comparable conditions.

```
Prompt template:

You will execute one skill evaluation run.

1. Read the skill at <abs path to SKILL.md>. Treat its instructions as your operating procedure.
2. Complete this task as if a real user asked it:

   <test case prompt>

3. Save any files you produce under: <abs path to runs/iteration-N/tc-XXX/with_skill/run-K/>
4. Write a brief transcript of your steps to transcript.md in the same folder.
5. Return a one-paragraph summary.
```

For the **baseline** run, use the same prompt but either:
- Omit step 1 entirely (`without_skill` baseline)
- Point step 1 at the snapshotted old skill (`old_skill` baseline when iterating)

**Option B — Manual run**

Ask the user to open a fresh chat with the relevant skill loaded, paste the prompt, and copy the output back to the run directory. Use when subagents can't faithfully reproduce the user-facing flow.

### 4. Grade each output

Two grading modes:

- **Inline grading** — you walk through each expected behavior in the main chat. Use for 1–3 cases when you wrote the assertions and outputs are plainly right or wrong. Apply the rules in [templates/grading-rubric.md](./templates/grading-rubric.md).
- **Grader subagent** — spawn an `Explore` agent with the prompt in [references/grader.md](./references/grader.md). **Preferred for comparisons** and whenever you wrote the skill yourself (self-grading bias).

Either way, the grader has **two jobs**:
1. PASS / FAIL each expected behavior with cited evidence
2. **Critique the assertions themselves** — flag trivially-satisfied ones, and important outcomes no assertion covers

Save `grading.json` per run matching [references/schemas.md#gradingjson](./references/schemas.md#gradingjson). A PASS on a weak assertion creates false confidence — the meta-critique is what keeps the suite honest.

### 5. Write the iteration report

Create `evals/history/<YYYY-MM-DD>-v<X.Y>.md` from [templates/report-template.md](./templates/report-template.md).

Must include:
- Date, skill version, # test cases, # runs per case
- Per-test-case PASS rate (and variance across runs if > 1 run)
- Aggregate metrics: overall pass rate, P0 pass rate, mean duration
- New issues, each with a stable ID (`#001`, `#002`...) that survives across reports
- **Diff vs previous report**: cases that flipped PASS↔FAIL (regressions vs fixes)
- Cross-iteration patterns from the analyzer (optional, see [references/analyzer.md](./references/analyzer.md))

### 6. Report findings to the user

Summarize:
- Overall pass rate vs last run (better / same / regressed)
- Each case that flipped (PASS → FAIL is a **regression**, FAIL → PASS is a **fix**)
- Top 1–3 issues to address next
- Whether baselines need updating (**only with explicit user confirmation** that the new output is genuinely better)

## Advanced patterns

These are optional. Load the relevant reference only when needed:

| Need | Reference |
|------|-----------|
| Design test cases that don't give false confidence | [references/test-case-design.md](./references/test-case-design.md) |
| Grade outputs with an unbiased subagent | [references/grader.md](./references/grader.md) |
| Rigorous A/B between two skill versions | [references/comparator.md](./references/comparator.md) |
| Understand *why* one version beat another | [references/analyzer.md](./references/analyzer.md) |
| Tune `description` for better trigger accuracy | [references/description-optimization.md](./references/description-optimization.md) |
| Organize multi-iteration workspace | [references/iteration-workflow.md](./references/iteration-workflow.md) |
| JSON schemas (test-cases, grading, benchmark) | [references/schemas.md](./references/schemas.md) |

## Adding test cases from bugs

When a real-use bug surfaces (not during evaluation):
1. Reproduce the failing prompt
2. Add it to `test-cases.json` with `tags: ["bug", "regression"]`
3. Link to the issue ID from the most recent report

This is how the test suite grows organically. Don't try to enumerate everything up front.

## Improvement principles

When applying feedback from a grading or comparison run:

- **Generalize from the feedback.** A fix for one test case should improve the skill broadly, not just patch that case. If you find yourself adding "if the prompt mentions X, do Y" — step back and ask whether the underlying instruction is unclear.
- **Keep the prompt lean.** Delete sections that aren't pulling their weight. A longer SKILL.md is not a better SKILL.md.
- **Explain the why, briefly.** A one-line rationale for a non-obvious instruction helps the model follow it. Avoid all-caps "MUST" / "ALWAYS" — they read as desperation, not authority.
- **Bundle repeated work into bundled resources.** If you find the skill always references the same procedure, move it to a reference file the skill links to.
- **Snapshot before iterating.** Always copy the current SKILL.md into `_skill-snapshot/` before editing — otherwise you can't reproduce the baseline.

## Stop conditions

Stop iterating when any of:
- The user is satisfied with current results
- Review feedback is consistently empty across cases
- Two consecutive iterations show no meaningful pass-rate movement
- You're making changes that improve one test case at the cost of another (overfitting — widen the suite instead)

## Anti-patterns

- **Designing 20 test cases on day one.** Start with 2–3. Add from bugs.
- **Vague expected behaviors** ("output is good"). Use checkable statements: "directory name matches `/^[a-z-]+$/`".
- **Single run per case.** No variance signal — a flaky case looks like a regression.
- **Skipping the report.** A run with no recorded result can't show change over time.
- **Auto-updating baselines.** Only update a baseline after the user confirms the new output is genuinely better.
- **Self-grading without distance.** The author of the skill is biased. Prefer the grader subagent for important comparisons.
- **Mixing skills in one evals folder.** Each skill owns its own `evals/`. Don't share cases across skills.
- **Optimizing for the test set.** If a skill works only on the 3 cases you've been iterating on, it's overfit. Periodically widen the suite.

## Dependencies

None beyond what VS Code Copilot already provides:
- `read_file` / `create_file` for managing `evals/` files
- `runSubagent` (optional) for automated runs and grading
- Markdown + JSON only — no Python scripts, no CLI tools, no external services

## Related

- [templates/test-case-template.json](./templates/test-case-template.json) — test case schema
- [templates/report-template.md](./templates/report-template.md) — iteration report skeleton
- [templates/grading-rubric.md](./templates/grading-rubric.md) — PASS / FAIL evidence rules
- [references/](./references/) — deeper guides for each advanced pattern
