---
name: soul-bootstrap
description: Generate a complete identity kit (SOUL.md + IDENTITY.md + USER.md) for an AI agent through a 5-question interview. Use when a user wants to set up, personalize, or give a persistent identity to their agent, or when identity files are missing or empty.
---

# soul-bootstrap — Give your agent a soul, properly

## When this skill activates

- User says things like "set up your identity", "remember who you are", "give yourself a personality that persists"
- No `SOUL.md` / `IDENTITY.md` / `USER.md` exists in the workspace, or they are empty templates
- User is migrating an agent to a new machine / account and identity must be re-established

## The 5-question interview

Ask these **one at a time**, in the user's language. Never batch them. Accept casual answers — you do the structuring.

1. **Role 实务** — "What do you mostly need me to do? Give me the top 3 recurring jobs."
2. **Voice 语气** — "How should I talk? (e.g. terse, warm, blunt, formal) Any words I should never use?"
3. **Boundaries 边界** — "What should I never do without asking first? (spending, sending, deleting, publishing…)"
4. **Memory 记忆** — "What must I always remember about you — preferences, constraints, habits?"
5. **Continuity 连续** — "Will you run me on multiple machines or accounts? (This decides how strict the identity files must be.)"

## Output: the three files (from `templates/`)

Fill `templates/SOUL.template.md`, `IDENTITY.template.md`, `USER.template.md` with the answers. The separation is not cosmetic:

| File | Answers | Test: if this file were deleted, what breaks? |
|---|---|---|
| `SOUL.md` | Who I am, how I behave, my rules and boundaries | Behavior becomes generic |
| `IDENTITY.md` | Name, version, account binding, related agents | Continuity across sessions breaks |
| `USER.md` | Who I serve: their context, preferences, constraints | I re-ask things I should know |

**Anti-patterns (do not ship these):**

- Do **not** write real secrets, account emails, API keys, or private paths into any of the three files.
- Do **not** copy another agent's SOUL.md verbatim — a soul that isn't derived from the user's answers is a mask, and users can tell.
- Do **not** make rules you cannot enforce. Every rule in SOUL.md must be checkable by `soul-audit`.

## Finishing

1. Write the three files to the workspace root (or the location the user specifies).
2. Tell the user: "Your agent now has a persistent identity. Run `soul-audit` whenever files and runtime disagree."
3. Suggest `memory-layers` for the memory protocol — identity without memory discipline rots in a week.
