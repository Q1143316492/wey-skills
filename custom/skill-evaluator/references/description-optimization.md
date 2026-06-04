# Description Optimization

The `description` field in SKILL.md frontmatter is the **only thing** the model sees when deciding whether to invoke your skill. Bad description = skill never triggers = skill is useless even if its body is perfect.

This is a separate playbook because it tests **triggering**, not output quality.

## When to run this

- A skill is "good" by output evaluation but rarely gets triggered in real use
- After significant changes to what the skill does (description may be out of sync)
- You see the user manually invoking it with `/<skill-name>` instead of natural language

Skip if: the skill is barely used yet (collect real-use signals first), or the skill is only ever invoked explicitly by slash command.

## How skill triggering works

The model sees a list of available skills (name + description, ~100 tokens each). For each user request, it decides whether to consult any skill based on the description.

Critical caveats:
- **Simple one-step requests rarely trigger any skill**, even if the description matches perfectly — the model handles them directly with basic tools.
- **Substantive, multi-step, or specialized requests** trigger reliably when the description matches.
- Models tend to **under-trigger** — they err on the side of not consulting a skill when in doubt.

This means your eval queries must be **substantive enough** that the model would benefit from consulting the skill. "Read this file" is not a useful trigger test.

## Procedure

### Step 1: Build a trigger-eval set

Create ~20 queries split into two groups:

**should-trigger (8–12)**:
- Varied phrasings of the skill's actual use cases
- Some formal, some casual, some with typos
- Include cases where the user doesn't explicitly name the skill — they describe the *problem*, not the *tool*
- Some edge cases where the skill competes with another but should win

**should-not-trigger (8–12)**:
- The most valuable are **near-misses**: queries that share keywords with the skill but actually need something different
- Adjacent domains, ambiguous phrasing where a naive keyword match would falsely trigger
- Cases where the user touches on something the skill does, but in a context where something else is more appropriate

> Don't pad should-not-trigger with obviously irrelevant queries. "Write a fibonacci function" is not a useful negative for a PDF skill — it doesn't test anything. Negatives must be genuinely tricky.

Save to `evals/trigger-eval.json`:

```json
[
  { "query": "我有个 q4 sales 的 xlsx，老板让我加一列算利润率%，revenue 在 C 列 cost 在 D 列", "should_trigger": true },
  { "query": "帮我把这个 csv 文件读成 dataframe", "should_trigger": false },
  ...
]
```

Queries must be **realistic**:
- Bad: "Format this data" / "Create a chart" / "Extract text from PDF"
- Good: "ok so my boss just sent me an xlsx (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a profit margin % column. Revenue is in column C and costs are in column D i think"

Include file paths, personal context, casual speech, abbreviations, typos. That's what real users type.

### Step 2: Review with the user

Walk the user through the eval set:
- "Does each should-trigger query genuinely warrant this skill?"
- "Are the should-not-trigger queries tricky enough?"

Bad eval queries lead to bad descriptions. This review step is not optional.

### Step 3: Run the trigger eval

Without Claude CLI (Anthropic's `claude -p` isn't available here), you have two options:

**Option A — Subagent vote (preferred):**

For each query, spawn an `Explore` subagent with this prompt:

```
You are about to receive a user request. You have access to these skills:
<paste the current available_skills list as the chat shows them>

Decide whether you would consult any skill, and if so which one. Return JSON:
{ "would_consult_skill": true|false, "skill_name": "<name or null>", "reasoning": "<one line>" }

USER REQUEST:
<query>
```

Run each query **3 times** (LLM decisions are non-deterministic). Compute trigger rate per query.

**Option B — Manual:**

Paste each query into a fresh chat and observe whether the skill is invoked. Tedious but works for small eval sets.

Aggregate:
- For should-trigger queries: trigger rate ≥ 67% (≥ 2 out of 3) counts as PASS
- For should-not-trigger queries: trigger rate ≤ 33% counts as PASS
- Overall score = (passed should-trigger + passed should-not-trigger) / total

### Step 4: Iterate the description

If the score is low, look at *which queries failed*:
- Should-trigger that didn't fire → description is missing relevant keywords or "use when" cues
- Should-not-trigger that did fire → description is too broad

Common fixes:
- Add **"Use when ..."** with concrete trigger contexts ("when user mentions X, Y, even if they don't explicitly say 'Z'")
- Be a bit **"pushy"** — models tend to under-trigger. "Make sure to use this skill whenever ..." is fine.
- Remove ambiguous phrasing that overlaps with adjacent domains
- Add explicit DO-NOT-USE-FOR clauses for the near-misses that wrongly triggered

Then re-run the eval. Iterate up to ~5 rounds. Stop when the score plateaus.

### Step 5: Hold out a test split

If you only iterate on the same 20 queries, you'll overfit the description to them. **Split 60% train / 40% held-out test**. Use train to iterate, then verify the final description on the test set. If the test score is much lower than train, you overfit — go back and use simpler description changes.

### Step 6: Apply

Update the SKILL.md `description` with the best-performing version. Show the user before/after and the scores.

## Description writing patterns

**What to include:**
- One sentence on *what* the skill does
- "Use when ..." with specific trigger contexts
- Keywords the user is likely to say (with synonyms / related terms)
- For under-triggering issues: an explicit "even if they don't say X" cue
- For over-triggering issues: an explicit "DO NOT USE FOR ..." clause

**What to avoid:**
- Generic descriptions: "A helpful skill that does X" — gives the model no trigger signal
- Description so long the model loses the trigger cue in detail
- Implementation details (the model triggers on *intent*, not *how* the skill works internally)
- Hedging: "May be useful for ..." reads as "probably skip this"

**Length:** Most good descriptions are 100–400 characters. The hard cap is 1024.

## Common failure modes

| Symptom | Likely cause | Fix |
|---|---|---|
| Skill rarely triggers in real use | Description too generic, or trigger words missing | Add "Use when ..." with concrete contexts |
| Skill triggers when it shouldn't | Description too broad, overlaps with adjacent skills | Add DO-NOT-USE-FOR clause |
| Skill triggers on explicit requests but not natural language | Description matches the skill's name but not user phrasing | Add user-style trigger phrases |
| Trigger rate varies wildly across re-runs | Description is borderline-relevant — model is genuinely uncertain | Either commit harder (more pushy) or accept it as an explicit-invocation skill |
