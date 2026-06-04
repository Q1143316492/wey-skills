# Blind A/B Comparator

When the user asks "is the new version actually better?", a side-by-side blind comparison removes confirmation bias.

## When to use

- Comparing two versions of the same skill (before/after a non-trivial change)
- The expected-behaviors PASS rate is similar but you suspect quality differs
- The skill produces subjective output (formatting, structure, prose) that assertions can't fully capture

Skip this for: pure regression checking (the grader is enough), or when one version obviously beats the other on hard metrics.

## How it works

1. Both versions run all test cases (same prompts, same baselines if relevant)
2. For each test case, you take the two outputs and **strip identifying labels**
3. A comparator subagent receives them labeled only "A" and "B" — it does not know which is the new version
4. The comparator scores each output on a rubric and picks a winner
5. After the verdict, an analyzer subagent reads both skills and explains *why* the winner won

The blindness is what gives the result credibility. If you skip it, the comparator is just rationalizing your preferred version.

## Spawning the comparator

For each test case, copy both outputs to a neutral location:

```
comparison/tc-XXX/
├── output_a/       # one version's output (you remember the mapping privately)
├── output_b/       # the other version's output
└── verdict.json    # comparator writes here
```

**Randomize the A/B assignment per test case** so the comparator can't infer the pattern across cases.

Spawn the comparator with `runSubagent` (Explore agent):

```
You are a blind comparator. Two skill outputs have been provided. You do NOT know which version produced which — do not speculate.

INPUTS
- Original task prompt:
  <eval prompt>

- Output A: <abs path to comparison/tc-XXX/output_a/>
- Output B: <abs path to comparison/tc-XXX/output_b/>
- Optional expectations (treat as secondary signal, not decisive):
  <list of expected behaviors>

PROCESS
1. Read both outputs in full.
2. Derive a rubric from the task. Pick 3–6 criteria across two dimensions:
   - Content: correctness, completeness, accuracy
   - Structure: organization, formatting, usability
3. Score each output on each criterion (1–5 scale, where 3 = acceptable).
4. Compute content_score (mean of content criteria), structure_score (mean of structure), overall_score = (content_score + structure_score) * 1.0 (1–10 scale).
5. Check optional expectations against each output if provided — record pass counts but do not let them override the rubric.
6. Pick a winner. TIE only if scores are truly identical and you cannot articulate a difference.

OUTPUT
Write JSON to <abs path to comparison/tc-XXX/verdict.json>:

{
  "winner": "A" | "B" | "TIE",
  "reasoning": "<2–4 sentences citing specific differences>",
  "rubric": {
    "A": { "content": {...}, "structure": {...}, "content_score": N, "structure_score": N, "overall_score": N },
    "B": { "content": {...}, "structure": {...}, "content_score": N, "structure_score": N, "overall_score": N }
  },
  "strengths": { "A": ["..."], "B": ["..."] },
  "weaknesses": { "A": ["..."], "B": ["..."] },
  "expectation_results": { "A": { "passed": N, "total": N }, "B": { "passed": N, "total": N } }   // omit if no expectations
}

RULES
- Do NOT try to infer which version is which. Do not mention "the newer" or "the original" anywhere.
- Cite specific evidence. "B is better organized" without an example is not useful.
- Be decisive. TIE should be rare.
- If both outputs fail, pick the one that fails less badly.
```

## Reading the results

After all cases have verdicts, aggregate:

| Test case | Winner | Margin | A score | B score |
|-----------|--------|--------|---------|---------|
| tc-001 | A | clear | 8.5 | 5.0 |
| tc-002 | B | narrow | 7.0 | 7.5 |
| tc-003 | TIE | — | 8.0 | 8.0 |

Then **un-blind** the mapping:
- A = old version, B = new version → tally wins for each
- The new version "wins" overall if it wins more cases (and especially the wider margins)

A 4–1 win with three narrow margins is much weaker evidence than a 3–2 win where all 3 are landslide. Look at the distribution.

## When to follow with the analyzer

If the new version wins clearly, run [analyzer.md](./analyzer.md) to extract *why* — those reasons are reusable patterns for future skill revisions.

If the new version loses or ties unexpectedly, the analyzer is also useful: it tells you what regressed so you can roll back the offending change.

## Caveats

- Comparators are LLMs and have their own biases (verbosity, structure, etc.). Don't treat a single comparator verdict as ground truth — re-run with a fresh subagent if a verdict looks suspicious.
- Comparator verdicts on **subjective** outputs (writing style, design) are stronger signal than on outputs that should match a hard spec — use the grader's assertion-based PASS rate for those.
- If both versions are very close, the comparator may pick the more verbose one. Add a "concision" criterion to the rubric if length is a real concern.
