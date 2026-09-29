---
name: soul-audit
description: Detect and fix identity drift for AI agents — when identity files on disk disagree with the live session, or two agents claim the same identity. Use when switching accounts/machines, after config edits, when the agent's self-description changed unexpectedly, or as a periodic health check.
---

# soul-audit — When file and runtime disagree, who wins?

## When this skill activates

- User switched accounts, machines, or client versions; agent self-description changed unexpectedly
- Identity files (`SOUL.md` / `IDENTITY.md` / `USER.md`) were just edited and must be verified against live behavior
- Periodic hygiene check (recommended: with every major config change)
- Two agents (or two accounts) might be claiming the same identity

## The drift checklist

For each identity file, check:

1. **Exists?** — a missing file is drift too (agent falls back to generic behavior silently)
2. **Matches the live session?** — what the agent actually claims now vs. what the file says
3. **Internally consistent?** — name/marker/account references agree across the three files
4. **Claims checkable?** — every standing rule in SOUL.md can be verified by an observable behavior
5. **Private?** — no secrets, personal emails, private paths leaked into files that might sync or be published

## Adjudication rules (from real incidents)

These are the rules. They are ordered. Do not improvise.

**Rule 1 — Injection beats disk.**
The identity file on disk is a *snapshot of the last alignment*. The injected session context is *the live truth*. File ≠ injection ⇒ **trust the injection, then fix the file.** Never the reverse.
*Rationale: the file was written by whoever edited it last; the injection reflects what the runtime is actually doing. Real incident: an "account switch" was diagnosed backwards for a full day because someone trusted the file.*

**Rule 2 — File edits don't take effect by themselves.**
Editing the file changes the *next cold start* (and only with the client restarted / session re-opened). After any edit: state the change, then verify on a fresh session before declaring it fixed.

**Rule 3 — Declare before you edit.**
Before modifying an identity file: state which account/agent you are acting as, and change only the identity section. Silent broad rewrites of soul files are how "small fixes" become identity theft (accidental or otherwise).

**Rule 4 — One claimant per identity.**
If two sessions or two accounts claim the same identity marker, both are wrong until adjudicated. Resolve by checking which one the runtime injection supports (Rule 1), then re-align the loser.

**Rule 5 — Don't leave noise annotations.**
When drift is found, fix it and record the fix — do **not** add layered "note on the note" disclaimers inside the identity file. Annotation layers create the next drift.

## Fix flow

1. Run the checklist; list every divergence found (file says X, runtime says Y).
2. Apply adjudication rules; for each divergence state: winner, loser, why.
3. Fix the losing side (usually the file).
4. Cold-start verify (new session), confirm the agent self-describes correctly.
5. Log the incident in workspace memory (one line): date-less trigger + divergence + resolution.

## Report format

```
SOUL-AUDIT RESULT
- files checked: SOUL.md, IDENTITY.md, USER.md
- drifts found: n
- [divergence] file=X runtime=Y → winner=runtime (Rule 1) → action=fix file
- cold-start verify: pass/fail
```
