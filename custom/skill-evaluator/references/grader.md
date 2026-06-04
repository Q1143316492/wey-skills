# Grader Subagent

Spawn an unbiased grader to score test outputs and critique the test cases themselves.

## When to use a grader subagent vs inline grading

**Inline grading** (you, in the main chat):
- Few cases (1–3)
- You wrote the assertions and the output is plainly correct/incorrect
- Fast iteration on the skill, not on the eval system

**Grader subagent** (preferred for comparisons):
- More than one configuration to grade (with_skill vs without_skill, A vs B)
- You wrote the skill being evaluated (self-grading bias)
- You want the grader to *also* critique your assertions

## Why a separate grader

The author of a skill is biased toward seeing what they expected to see. A subagent that hasn't seen the skill instructions evaluates the output on its merits.

The grader has **two jobs**:
1. PASS/FAIL each expected behavior with cited evidence
2. Critique the assertions — flag ones that are too lenient, or important outcomes no assertion covers

A PASS on a weak assertion creates false confidence. The meta-critique is what keeps the suite honest.

## Spawning the grader

Use `runSubagent` with the `Explore` agent. Pass the prompt below, filling in the bracketed slots.

```
You are grading a skill evaluation run. You have NOT been told which version of the skill produced this output — do not try to guess.

INPUTS
- Test case prompt:
  <eval prompt>

- Expected behaviors (grade each one PASS or FAIL):
  1. <behavior 1>
  2. <behavior 2>
  3. ...

- Output directory: <abs path to runs/iteration-N/tc-XXX/<config>/>
- Transcript file: <abs path to .../transcript.md> (if present)

PROCESS
1. List and read the output files. If any are binary, describe them by structure (size, type) rather than guessing content.
2. Read the transcript if present.
3. For each expected behavior:
   - Search for concrete evidence in the outputs and transcript
   - Verdict: PASS only if evidence is specific and reflects genuine completion; FAIL if missing, contradicted, or superficial (e.g. correct filename, empty content)
   - Cite the evidence (quote a line, name a file, describe a tool call)
4. Extract any implicit claims the output makes (factual / process / quality) and verify them. Flag unverifiable claims.
5. Critique the assertions themselves: any that are trivially satisfied? Any important outcome no assertion covers?

OUTPUT
Write a JSON file to <abs path to .../grading.json> with this exact schema:

{
  "expectations": [
    { "text": "<original behavior text>", "passed": true|false, "evidence": "<specific quote or description>" }
  ],
  "summary": { "passed": N, "failed": N, "total": N, "pass_rate": 0.00 },
  "claims": [
    { "claim": "<implicit claim>", "type": "factual|process|quality", "verified": true|false, "evidence": "<...>" }
  ],
  "eval_feedback": {
    "suggestions": [
      { "assertion": "<text or null>", "reason": "<why it's weak or what's missing>" }
    ],
    "overall": "<one-line assessment, or 'No suggestions, evals look solid'>"
  }
}

Field rules:
- Use the exact field names above (expectations, passed, evidence). Other variants break the report generator.
- expectations[].text must match the input behavior text exactly.
- No partial credit. Each expectation is true or false.
- Burden of proof is on PASS. When uncertain, FAIL.

Return a one-paragraph summary at the end describing the overall verdict and any notable issues.
```

## Grading criteria

**PASS when all hold:**
- The output or transcript clearly demonstrates the behavior
- Specific evidence can be cited (a file, a line, a tool call)
- The evidence reflects substance, not surface (a file exists *and* contains correct content)

**FAIL when any of:**
- No evidence found
- Evidence contradicts the behavior
- The behavior cannot be verified from the available output
- The output meets the assertion by coincidence rather than by doing the work

**No partial credit.** Each expectation is binary. If you'd give "0.5", FAIL and note the partial completion in the evidence.

## Strong vs weak evidence

| Evidence | Strength |
|---|---|
| "File `output/spec.md` exists with 47 non-empty lines including the headers '## Why', '## What changes'" | strong |
| "Output contains the literal string 'kebab-case' in the directory name" | strong |
| "Transcript shows tool call `create_file` with path matching the pattern" | strong |
| "The output looks reasonable" | weak |
| "Model reported success" | weak |
| "No errors were raised" | weak (absence ≠ success) |

## Critiquing assertions

After grading, the grader should flag assertions that:
- Passed but would also pass for a clearly wrong output (e.g. checks a filename but not content)
- No assertion covers an important outcome the grader observed
- Can't actually be verified from what the skill produces

These show up in `eval_feedback.suggestions` and feed into the next iteration's test case revision.

Don't nitpick. The bar is: "would the eval author say *good catch*?"
