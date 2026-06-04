---
name: write-a-skill
description: Create new agent skills with proper structure, progressive disclosure, and bundled resources. Use when user wants to create, write, or build a new skill.
---

# Writing Skills

## Process

1. **Gather requirements** — ask user:
   - What task/domain does the skill cover?
   - What specific use cases should it handle?
   - Does it need scripts or just instructions?

2. **Draft the skill** — create:
   - `SKILL.md` with concise instructions
   - Additional reference files if content exceeds 100 lines

3. **Review with user** — present draft and ask:
   - Does this cover your use cases?
   - Anything missing or unclear?

## Structure

```
skill-name/
└── SKILL.md        # Required
```

## SKILL.md Template

```md
---
name: skill-name
description: Brief description. Use when [specific triggers].
---

# Skill Name

## Quick start
[Minimal working example]

## Workflow
[Step-by-step process]
```

## Description Requirements

The description is the only thing the agent sees when deciding to load a skill.

- Max 1024 chars
- First sentence: what it does
- Second sentence: "Use when [specific triggers]"

Good: `Extract text from PDF files. Use when working with PDFs or user mentions document extraction.`
Bad: `Helps with documents.`

## Review Checklist

- [ ] Description includes "Use when..." triggers
- [ ] SKILL.md under 100 lines
- [ ] No time-sensitive info
- [ ] Concrete and actionable instructions
