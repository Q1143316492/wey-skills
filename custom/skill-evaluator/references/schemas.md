# JSON Schemas

Reference for the JSON files produced and consumed by the evaluation workflow.

---

## test-cases.json

Location: `<skill>/evals/test-cases.json`. Defines the test set for the skill.

```json
{
  "skill_name": "openspec-propose",
  "version": "1.0",
  "last_updated": "2026-06-04",
  "notes": "Start with 2–3 real prompts. Grow from bugs.",
  "test_cases": [
    {
      "id": "tc-001",
      "name": "short descriptive name",
      "priority": "P0",
      "prompt": "The exact text a real user would type",
      "expected_behaviors": [
        "Concrete, checkable statement",
        "Another checkable statement (file exists, regex matches, ...)"
      ],
      "files": ["evals/inputs/example.csv"],
      "baseline_file": "baselines/tc-001-expected.md",
      "tags": ["core", "chinese"]
    }
  ]
}
```

| Field | Required | Description |
|---|---|---|
| `skill_name` | yes | Must match the skill's frontmatter `name` |
| `version` | yes | Free-form version label for this test set |
| `last_updated` | yes | ISO date |
| `test_cases[].id` | yes | Stable identifier, e.g. `tc-001`. Never rename. |
| `test_cases[].priority` | yes | `P0` / `P1` / `P2`. See [test-case-design.md](./test-case-design.md). |
| `test_cases[].prompt` | yes | The literal user text |
| `test_cases[].expected_behaviors` | yes | At least 1, ideally 3–5. Each must be checkable from output. |
| `test_cases[].files` | no | Input files to provide to the skill (paths relative to skill root) |
| `test_cases[].baseline_file` | no | Path to a known-good output for diffing |
| `test_cases[].tags` | no | Free-form labels for slicing the suite |

---

## grading.json

Location: `<skill>/evals/runs/iteration-N/tc-XXX/<config>/grading.json`. Written by the grader (inline or subagent).

```json
{
  "expectations": [
    {
      "text": "Generates a single change directory, not multiple",
      "passed": true,
      "evidence": "Only one directory 'add-user-auth/' was created in openspec/changes/"
    },
    {
      "text": "Directory name uses kebab-case and ASCII only",
      "passed": false,
      "evidence": "Directory was named '添加-user-auth' — contains non-ASCII chars"
    }
  ],
  "summary": {
    "passed": 1,
    "failed": 1,
    "total": 2,
    "pass_rate": 0.50
  },
  "claims": [
    {
      "claim": "Used the openspec CLI to create the change",
      "type": "process",
      "verified": true,
      "evidence": "Transcript step 2 shows: 'Running: openspec new change ...'"
    }
  ],
  "eval_feedback": {
    "suggestions": [
      {
        "assertion": "Generates a single change directory",
        "reason": "Passes even for empty/wrong directory — also check directory contains at least proposal.md and tasks.md"
      }
    ],
    "overall": "Assertions check structure but not content. Consider adding content-validity checks."
  },
  "timing": {
    "duration_seconds": 23.3
  }
}
```

**Critical field-name rules:**
- `expectations[]` must use `text`, `passed`, `evidence` — not `name`/`met`/`details` or other variants
- `passed` is boolean, never a string or number
- `summary.pass_rate` is a fraction 0.0–1.0, never a percentage

---

## benchmark.json

Location: `<skill>/evals/runs/iteration-N/benchmark.json`. Aggregates all runs in an iteration.

```json
{
  "metadata": {
    "skill_name": "openspec-propose",
    "iteration": 1,
    "timestamp": "2026-06-04T10:30:00Z",
    "test_cases_run": ["tc-001", "tc-002", "tc-003"],
    "runs_per_configuration": 3,
    "configurations": ["with_skill", "without_skill"]
  },
  "runs": [
    {
      "test_case_id": "tc-001",
      "test_case_name": "chinese basic input",
      "configuration": "with_skill",
      "run_number": 1,
      "result": {
        "pass_rate": 0.85,
        "passed": 6,
        "failed": 1,
        "total": 7,
        "duration_seconds": 42.5,
        "errors": 0
      },
      "expectations": [
        { "text": "...", "passed": true, "evidence": "..." }
      ],
      "notes": ["Used fallback for missing field"]
    }
  ],
  "run_summary": {
    "with_skill": {
      "pass_rate":         { "mean": 0.85, "stddev": 0.05, "min": 0.80, "max": 0.90 },
      "duration_seconds":  { "mean": 45.0, "stddev": 12.0, "min": 32.0, "max": 58.0 }
    },
    "without_skill": {
      "pass_rate":         { "mean": 0.35, "stddev": 0.08, "min": 0.28, "max": 0.45 },
      "duration_seconds":  { "mean": 32.0, "stddev": 8.0,  "min": 24.0, "max": 42.0 }
    },
    "delta": {
      "pass_rate": "+0.50",
      "duration_seconds": "+13.0"
    }
  },
  "notes": [
    "Assertion X passes 100% in both configurations — not discriminating",
    "tc-003 has 33% ± 47% variance — investigate or mark flaky"
  ]
}
```

| Field | Notes |
|---|---|
| `metadata.configurations` | Always include `with_skill`. The other is `without_skill` (new skill) or `old_skill` (skill being improved). |
| `runs[].configuration` | Must match a string in `metadata.configurations`. |
| `run_summary.<config>.<metric>` | Always `{ mean, stddev, min, max }`. Even with 1 run, stddev = 0. |
| `notes` | Free-form observations from the analyzer (see [analyzer.md](./analyzer.md) part 2). |

---

## history.json (optional)

If you want a single rollup across all iterations of a skill, write it to `<skill>/evals/history/history.json`.

```json
{
  "skill_name": "openspec-propose",
  "started_at": "2026-06-04T10:30:00Z",
  "current_best": "v1.1",
  "iterations": [
    {
      "version": "v1.0",
      "parent": null,
      "pass_rate_with_skill": 0.65,
      "pass_rate_without_skill": 0.30,
      "delta": 0.35,
      "is_current_best": false,
      "report_file": "2026-06-04-v1.0.md"
    },
    {
      "version": "v1.1",
      "parent": "v1.0",
      "pass_rate_with_skill": 0.85,
      "pass_rate_without_skill": 0.30,
      "delta": 0.55,
      "is_current_best": true,
      "report_file": "2026-06-05-v1.1.md"
    }
  ]
}
```

This is optional — the per-iteration reports under `history/` already capture the same info. Use `history.json` only if you want machine-readable rollups (e.g. for trend charts).

---

## trigger-eval.json

Location: `<skill>/evals/trigger-eval.json`. Used by [description-optimization.md](./description-optimization.md).

```json
[
  {
    "query": "ok so my boss just sent me this xlsx file ...",
    "should_trigger": true,
    "trigger_results": {
      "current_description": { "rate": 0.67, "runs": 3 },
      "candidate_v2":        { "rate": 1.00, "runs": 3 }
    }
  }
]
```

`trigger_results` is filled in after running each candidate description through the eval. Track multiple candidates to pick the best.
