---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up. Use when switching sessions, handing work to another agent, or resuming later.
argument-hint: "What will the next session be used for?"
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save to the OS temp directory — not the current workspace.

Include a "suggested skills" section that suggests skills the next agent should invoke.

Do not duplicate content already captured in other artifacts (plans, commits, diffs). Reference them by path instead.

Redact any sensitive information (API keys, passwords, personal data).

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.
