# Grading Rubric

How to decide whether an expected behavior PASSes or FAILs.

## Core rule

Each expected behavior is **binary**: PASS or FAIL. No partial credit.

## PASS criteria

All of these must hold:
- There is **specific evidence** in the output (a file, a quoted line, an observed tool call).
- The evidence reflects **genuine completion**, not just surface-level compliance.
- The behavior would still be observable if you re-ran the test.

## FAIL criteria

Any of these triggers FAIL:
- No evidence found.
- Evidence contradicts the expected behavior.
- The behavior cannot be verified from the available output.
- The output appears to satisfy the assertion **by coincidence** rather than by actually doing the work (e.g., right filename but empty file).

## Evidence types

**Strong evidence** (prefer these when writing expected behaviors):
- A file exists at a specific path with non-trivial content
- A specific string appears in the output
- A regex matches a generated value (e.g., directory name)
- A numeric threshold is met (e.g., duration < 10s, file count >= 3)

**Weak evidence** (avoid as the sole basis for PASS):
- "The output looks reasonable"
- "The model said it did X" (without verifying X actually happened)
- "No error was reported" (absence of failure ≠ success)

## When uncertain

Default to FAIL. The burden of proof is on the expected behavior.
Then note the uncertainty in the report so the assertion can be refined for next time.

## Critiquing the assertions themselves

While grading, flag assertions that turned out to be weak:
- Passed but would also pass for clearly wrong output → too lenient
- Important outcome the run revealed but no assertion covered → coverage gap
- Couldn't be verified from the output the skill produces → unverifiable

Record these in the report's "Next actions" section as test-case improvements.
