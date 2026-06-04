---
name: diagnose
description: Disciplined diagnosis loop for hard bugs and performance regressions. Reproduce → minimise → hypothesise → instrument → fix → regression-test. Use when user says "diagnose this" / "debug this", reports a bug, says something is broken/throwing/failing, or describes a performance regression.
---

# Diagnose

A discipline for hard bugs. Skip phases only when explicitly justified.

## Phase 1 — Build a feedback loop

**This is the skill.** If you have a fast, deterministic, pass/fail signal for the bug, you will find the cause. If you don't, no amount of staring at code will save you.

Spend disproportionate effort here. Be aggressive. Be creative. Refuse to give up.

Ways to construct one (try in order):
1. Failing test at whatever seam reaches the bug
2. CLI invocation with a fixture input, diffing output against known-good snapshot
3. Throwaway harness: minimal subset of the system that exercises the bug code path
4. Property/fuzz loop: if the bug is "sometimes wrong output", run many random inputs
5. Bisection harness: if bug appeared between two known states, automate "check at state X"

If you genuinely cannot build a loop — stop and say so. List what you tried. Ask the user for access to the reproducing environment or a captured artifact. **Do not proceed to Phase 2 without a loop.**

## Phase 2 — Reproduce

Run the loop. Confirm:
- [ ] The loop produces the failure the **user** described — not a different nearby failure
- [ ] The failure is reproducible (or at a high enough rate for non-deterministic bugs)
- [ ] You have captured the exact symptom

## Phase 3 — Hypothesise

Generate **3–5 ranked hypotheses** before testing any of them. Each must be falsifiable:

> "If X is the cause, then changing Y will make the bug disappear."

Show the ranked list to the user before testing. They often have domain knowledge that re-ranks instantly. Don't block on it — proceed if user is AFK.

## Phase 4 — Instrument

Each probe must map to a specific prediction from Phase 3. **Change one variable at a time.**

- Debugger/breakpoint if env supports it — one breakpoint beats ten logs
- Targeted logs at boundaries that distinguish hypotheses
- **Tag every debug log** with a unique prefix e.g. `[DEBUG-a4f2]` — cleanup becomes a single search

For performance regressions: establish a baseline measurement first, then bisect. Measure first, fix second.

## Phase 5 — Fix + regression test

Write the regression test **before the fix** — but only if there is a correct seam for it.

If a correct seam exists:
1. Turn the minimised repro into a failing test
2. Watch it fail
3. Apply the fix
4. Watch it pass
5. Re-run the Phase 1 loop against the original scenario

## Phase 6 — Cleanup + post-mortem

- [ ] Original repro no longer reproduces
- [ ] Regression test passes (or absence of seam is documented)
- [ ] All `[DEBUG-...]` instrumentation removed
- [ ] Throwaway prototypes deleted
- [ ] The correct hypothesis is stated in the commit message

Then ask: what would have prevented this bug?
