# Memory Protocol — Rationale and Worked Examples

Companion to `skills/memory-layers/SKILL.md`. This document explains *why* each
rule exists, using anonymized, composite examples drawn from production
multi-agent operations.

## Why three layers and not one memory file?

A single unbounded memory file fails in a predictable sequence:

1. Week 1–2: works fine (small file, everything relevant).
2. Week 3–6: contradictions appear (a preference changed; both versions still in file).
3. Month 2+: retrieval degrades (relevant facts buried under event logs), and the
   agent starts repeating questions — the classic "I already told you this".

The failure is not discipline; it is **missing write-ownership**. The three-layer
model assigns every piece of information a home, a writer, and a lifespan.

## Worked example: the changed preference

Scenario: the user preferred table-formatted reports, then switched to prose.

- **Wrong (single file):** both preferences coexist; the agent picks whichever it
  reads first. Silent inconsistency.
- **Right (layered):** Layer-2 (user-level) rule is updated *in place* — the old
  rule is gone, because Layer 2 supports only explicit, in-place edits. Project
  archives in Layer 3 may still mention the old preference, but as history.

## Worked example: the stale cache

Scenario: the server-injected profile says the user works in city A; they moved
to city B months ago and told the agent, which "noted it down" locally.

- The agent kept editing its local file while the injected profile kept
  overwriting the truth at every session start. Both sides "remembered", neither
  agreed.
- **Rule:** injected profiles are read-only. To change them, correct the source
  of truth on the server side. Editing the cache locally produces a fight you
  lose every session.

## Worked example: the lost audit trail

Scenario: an agent "cleaned up" its daily workspace logs by rewriting them into
a tidy summary — and destroyed the only record of when a bad decision was made.

- **Rule:** daily logs are append-only. Summaries go into the curated note; raw
  logs are archived, not rewritten. Drift investigations (see `soul-audit`)
  depend on the raw trail.

## Distillation, concretely

When a layer hits its cap:

1. Group entries by topic (rules / decisions / preferences / events).
2. Keep: rules, decisions *with their reasons*, standing preferences.
3. Drop: transient events, search results, temp paths, tool errors.
4. Anything possibly needed verbatim later → archive outside the hot layer.
5. Record the distillation itself (one line: what was distilled, what was dropped).

## Search-order rule of thumb

Search narrow → wide: workspace (project) → user-level (machine) → injected
profile. The narrowest layer that contains an answer is the most current one.
