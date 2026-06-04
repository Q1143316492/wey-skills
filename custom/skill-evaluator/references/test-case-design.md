# Designing Test Cases

The test suite is the hardest part of evaluation. Bad test cases give **false confidence** — a 100% pass rate on weak cases is worse than no evaluation at all. This guide is how to avoid that trap.

## The fundamental problem

Skill outputs are open-ended. There is no `assert result == 42`. So:
- You cannot enumerate all valid inputs
- "Quality" of output is partially subjective
- The same input under different context can have different correct answers

The trick is not to solve this — it's to **bound it** with a small set of cases that cover the parts that matter.

## Three sources of test cases

### 1. Real usage (best)

Use the skill yourself for a week. Log every prompt you actually type. Pick the 3–5 most representative ones.

> "Imagined" cases are too clean. Real prompts have typos, casual phrasing, mid-sentence pivots, and assumed context.

### 2. Bugs (always add these)

Every time a real user (including you) hits a bug, that prompt becomes a permanent test case. Tag it `regression`. The suite grows in the direction of past pain — exactly where it should.

### 3. Adversarial / edge (sparingly)

A small number of intentionally hard cases. Don't pad with these; they're cheap to add but don't reflect real usage.

## Anatomy of a good test case

```json
{
  "id": "tc-003",
  "name": "long Chinese description with mixed technical terms",
  "priority": "P1",
  "prompt": "我需要做一个完整的用户认证系统，包括 JWT、session 管理、忘记密码流程，最好支持 OAuth2.0 集成 GitHub 和 Google",
  "expected_behaviors": [
    "Generates a single change directory, not multiple",
    "Directory name uses kebab-case and ASCII only (e.g. add-user-auth)",
    "tasks.md contains discrete tasks for JWT, OAuth, password reset",
    "design.md acknowledges the OAuth dependency choice"
  ],
  "tags": ["chinese", "long-input", "multi-feature"]
}
```

What makes this good:
- **Realistic** — the kind of thing a developer actually says
- **Specific** behaviors that can be checked from the output
- **Tags** that let you slice the suite (e.g. "show me all Chinese cases")
- **One concept per behavior** — easier to identify *which* part broke

## Anatomy of a bad test case

```json
{
  "id": "tc-bad",
  "name": "basic test",
  "prompt": "make a thing",
  "expected_behaviors": [
    "output is reasonable",
    "no errors",
    "files are created"
  ]
}
```

What's wrong:
- Prompt is too short / generic to trigger anything meaningful
- "Reasonable" is not checkable
- "No errors" is the absence of failure, not the presence of success
- "Files are created" — *which* files? With what content?

## Writing expected behaviors

**Strong** (verifiable from the output):
- "A file `proposal.md` exists with at least 30 non-empty lines"
- "The directory name matches `/^[a-z][a-z0-9-]+$/`"
- "tasks.md contains a numbered list of at least 5 items"
- "The output does not contain the phrase 'I cannot' or 'I'm not able'"
- "Completes within 30 seconds"

**Weak** (avoid as the sole assertion):
- "Looks reasonable"
- "Follows the spec" (which spec? which clauses?)
- "Is high quality"
- "The model says it did X" — verify X actually happened

If you can't write a strong assertion, the test case isn't ready. Either drop it or refine until you can.

## Priority and tiering

| Priority | Meaning | Should-pass threshold |
|---|---|---|
| **P0** | Core functionality. If this fails, the skill is broken. | 100% |
| **P1** | Common variants. Failures degrade UX but skill is usable. | ≥ 80% |
| **P2** | Edge cases, adversarial inputs. | Best-effort |

When making changes, P0 regressions must block the change. P1/P2 can be triaged.

## When to add vs. when to refuse

**Add when:**
- A real user hit it
- It exposes a class of behavior not yet covered
- It tests a known prior fix (regression guard)

**Refuse when:**
- It's a synthetic edge case you can't imagine a real user hitting
- It's testing the same thing as an existing case
- You can't write a strong assertion for it

A small, high-quality suite beats a large, redundant one.

## When the suite is "good enough"

Never. But practical signs you have enough:
- Last 3 user-reported bugs were already covered by an existing case (or a near-variant)
- New iterations show measurable movement (cases flip), not always-100% or always-50%
- You'd be embarrassed to ship if it failed any P0 case

If the suite is always at 100% across iterations, either the skill is genuinely solid **or** the suite is too easy. Investigate before celebrating.

## Maintenance

- **Quarterly**: re-read the suite. Remove cases that no longer exercise anything meaningful.
- **After major skill rewrites**: re-baseline. Old baselines may reflect old expectations.
- **When a case becomes flaky** (different result across runs): either tighten the assertion to remove ambiguity, or accept it as a "known-flaky" case (tag it `flaky` and report separately).

## Quick checklist before saving a test case

- [ ] A real user could plausibly type this
- [ ] At least 3 expected behaviors, each independently checkable
- [ ] No "looks good" / "reasonable" / "high quality"
- [ ] Priority is set (P0/P1/P2)
- [ ] At least one tag for slicing
- [ ] If it's a regression: link to the original issue ID
