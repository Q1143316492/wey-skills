# Analyzer

Two jobs:
1. **Post-hoc analysis** of a blind A/B verdict: *why* did the winner win, and what should change in the loser?
2. **Cross-run pattern analysis** of a benchmark: anomalies, flaky cases, non-discriminating assertions that aggregate stats hide.

## Part 1 — Post-hoc analysis of a blind comparison

Run after [comparator.md](./comparator.md) returns a verdict. The comparator was blind; the analyzer is **not** — it knows which skill is which and reads the skill files plus transcripts to extract *why*.

### Inputs

- Winner side (A or B) from the comparator verdict
- Winner skill path + transcript
- Loser skill path + transcript
- Comparator verdict JSON

### Spawning prompt

```
You are analyzing why the winning skill beat the losing skill in a blind comparison. The comparator's verdict is final — your job is to explain it.

INPUTS
- Winner verdict: <paste relevant fields from verdict.json>
- Winner skill: <abs path to winner SKILL.md>
- Winner transcript: <abs path>
- Loser skill: <abs path to loser SKILL.md>
- Loser transcript: <abs path>

PROCESS
1. Read both skills. Note structural differences: instruction clarity, scripts/tools used, examples covered, edge case handling.
2. Read both transcripts. Compare execution patterns: did each agent follow its skill's instructions? did the loser diverge from optimal? did either recover from errors?
3. Score instruction-following 1–10 for each, with specific issues noted.
4. Identify what made the winner better — be specific, quote skill text or transcript lines.
5. Identify what held the loser back — same.
6. Generate prioritized improvement suggestions for the LOSING skill. Categories: instructions, tools, examples, error_handling, structure, references. Priority: high (would have changed the outcome), medium (improves quality), low (nice-to-have).

OUTPUT
Write JSON to <abs path>:

{
  "winner_strengths": ["specific, quoted reasons"],
  "loser_weaknesses": ["specific, quoted reasons"],
  "instruction_following": {
    "winner": { "score": N, "issues": ["..."] },
    "loser":  { "score": N, "issues": ["..."] }
  },
  "improvement_suggestions": [
    {
      "priority": "high|medium|low",
      "category": "instructions|tools|examples|error_handling|structure|references",
      "suggestion": "concrete change to make",
      "expected_impact": "why this would have changed the outcome"
    }
  ],
  "execution_patterns": {
    "winner": "1-line summary of how winner executed",
    "loser":  "1-line summary of how loser executed"
  }
}

RULES
- Be specific. Quote the skill or transcript, don't say "instructions were unclear".
- Focus on changes to the LOSING SKILL, not on critiquing the agent that ran it.
- Prioritize by impact. Which changes would most likely change the outcome on the next iteration?
- Consider generalization: would this suggestion help on other evals too, or only this one?
```

## Part 2 — Cross-run pattern analysis

Run after grading a full iteration with multiple repeats per case. Surfaces patterns aggregate stats hide.

### What to look for

| Pattern | Why it matters | What to do |
|---|---|---|
| Assertion always PASSes in both with-skill and without-skill | Non-discriminating — the skill isn't adding value on this assertion | Strengthen the assertion or drop it |
| Assertion always FAILs in both | Beyond current capability, or assertion is wrong | Verify the assertion is achievable; if so, the skill needs that capability |
| Always PASS with-skill, FAIL without | The skill is clearly the cause of success here | Good signal — preserve this in iterations |
| Always FAIL with-skill, PASS without | The skill is *hurting* | High-priority investigation |
| High variance (e.g. 50% ± 40%) across runs of one config | Flaky — either the assertion is ambiguous or the skill is non-deterministic | Tag the case flaky or tighten the assertion |
| One run is an outlier (e.g. takes 5× longer) | Possible LLM retry, tool failure, or context-window issue | Read the outlier's transcript to confirm or rule out |

### Cross-eval patterns

- Are certain *categories* of evals (by tag) consistently harder? (e.g. all `chinese-input` cases pass < 60%)
- Are some evals stable across iterations while others swing wildly?
- Does total token / duration cost track pass-rate improvements, or is the skill getting more expensive without getting better?

### Output

A short list of notes appended to the iteration report. Each note states one specific observation grounded in the data. Examples:

- "Assertion 'Output is a markdown file' passes 100% in both with_skill and without_skill — drop or replace, it doesn't differentiate."
- "tc-003 (long Chinese input) has 33% ± 47% variance across 3 runs — investigate or mark flaky."
- "without_skill runs consistently fail on tasks.md format (0/3 pass) — strong evidence the skill is the cause of success here."
- "With-skill runs cost 80% more tokens for a 15-point pass-rate gain — acceptable tradeoff or worth optimizing?"

### What NOT to do in this part

- Don't suggest skill improvements here (that's part 1's job)
- Don't editorialize ("the output was good/bad") — describe what the data shows
- Don't speculate about causes without evidence — flag the pattern and let the human decide
