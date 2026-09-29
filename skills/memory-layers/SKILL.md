---
name: memory-layers
description: Three-layer memory protocol for AI agents — what to store where, who may write, when to distill, and which layer to search first. Use when deciding how to persist information, when memory files grow unbounded, or when the agent repeats questions it should already know.
---

# memory-layers — Three layers, clear ownership

## When this skill activates

- The agent must decide where to persist a piece of information (or whether to persist it at all)
- Memory files are growing unbounded or contradicting each other
- The user complains: "I already told you this" (search-order failure) or "why do you remember that?" (write-scope failure)

## The three layers

| # | Layer | Scope | Agent may write? | Discipline |
|---|---|---|---|---|
| 1 | **Injected / server profile** (cloud profile auto-loaded at session start) | cross-everything stable profile | ❌ read-only — server-owned, cached, overwritten externally | Never edit locally; it will be overwritten. If it's wrong, correct the source of truth, not the cache |
| 2 | **User-level local memory** (one file per user, machine-wide) | cross-project preferences, standing rules | ✅ only on explicit user instruction ("remember this") | Hard size cap. When exceeded: distill, don't grow. In-place edits only (no untracked copies) |
| 3 | **Workspace memory** (per project) | project state, decisions, daily logs | ✅ yes, with rules | Daily logs are **append-only**. Curated long-term note has a cap. Secrets only if user explicitly asks |

## Retrieval order (the part everyone gets wrong)

1. Current conversation context — free, always first
2. Workspace memory (project-specific, most specific)
3. User-level memory (cross-project, less specific)
4. Injected profile (stable but possibly stale)

**Rule of specificity**: search the *narrowest* layer that could contain the answer first. Searching wide-first produces stale answers.

## Write decision table

| Information type | Goes to | Example |
|---|---|---|
| "I prefer X in all projects" | Layer 2 (explicit ask) | "永远用表格给我汇报" |
| A decision made in this project | Layer 3 curated note | framework choice + why |
| What happened today | Layer 3 daily log (append) | delivered X, hit error Y |
| Facts about the user the server hasn't learned | Layer 2, only if user says "remember" | — |
| Anything the injected profile already covers | **nowhere** — don't duplicate | — |
| Secrets | **nowhere** unless the user explicitly asks, and then only in a dedicated vault, never in plain files | — |

## Distillation policy

- When Layer 2 or Layer 3 curated notes hit their cap: **distill by topic, keep rules/decisions/preferences, drop transient events** (search results, temp paths, tool errors).
- Daily logs older than ~30 days: distill into the curated note, then remove the raw files.
- Distillation is **lossy by design**. If something might be needed verbatim later, archive it — don't keep it in the hot layer.

## Anti-patterns (memory rot)

1. **The everything-file** — one unbounded file mixing rules, events, and transient data.
2. **Duplicate truths** — the same fact stored in two layers; they will disagree, and you won't know which is live.
3. **Cache editing** — editing the injected profile locally instead of the source of truth.
4. **Silent rewrites** — overwriting append-only logs (you lose the audit trail that `soul-audit` needs).
5. **Memory without write rules** — any agent in any session can write anything anywhere; within a month the memory is a lie nobody can debug.

See `references/memory-protocol.md` for the full rationale and worked examples.
