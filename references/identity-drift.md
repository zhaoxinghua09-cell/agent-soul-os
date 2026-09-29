# Identity Drift — Anonymized Incident Cases

Companion to `skills/soul-audit/SKILL.md`. All cases are anonymized and
composed from real production incidents across multi-account, multi-machine
agent operations. Details are generalized; the mechanisms are exact.

## Case 1: The backwards diagnosis (file vs. runtime)

**Symptom:** after an account switch, the agent kept addressing the user with the
wrong persona. The team concluded "the switch didn't take" and spent a day
re-configuring the account.

**Reality:** the runtime injection had switched correctly from the first minute.
The identity *file* on disk still described the old persona — and the team had
treated the file as truth.

**Lesson — Rule 1 (injection beats disk):** the file is a snapshot of the last
alignment; the injection is live truth. Diagnose from the injection outward.

## Case 2: The edit that never took effect

**Symptom:** an identity file was corrected, but a fresh session still showed the
old self-description. The editor concluded the "fix didn't work" and re-edited,
adding increasingly frantic annotations inside the file.

**Reality:** file edits take effect on the next cold start. The session being
observed had never restarted. Worse, the stacked annotations became their own
drift surface.

**Lesson — Rule 2 + Rule 5:** verify fixes in a fresh session; fix once, record
once, and never layer notes inside identity files.

## Case 3: Two agents, one name

**Symptom:** two sessions — run under two different accounts — both signed work
with the same agent name and marker. Trust broke down: which output was whose?

**Reality:** neither claimant was "right". Both were told their identity by
files that had drifted in opposite directions.

**Lesson — Rule 4:** when two claimants appear, both are wrong until adjudicated
against the runtime injection; then re-align the loser's files.

## Case 4: The silent broad rewrite

**Symptom:** a "small fix" to an identity file rewrote the whole persona section,
including boundaries. Downstream behavior changed for days before anyone noticed.

**Lesson — Rule 3:** declare which identity you are acting as, change only the
identity section, and log the change. Identity files are the most
high-leverage files in the workspace; they deserve the strictest write
discipline.

## The general pattern

Every drift incident shares one shape: **a snapshot was mistaken for the live
state** (or vice versa). The adjudication rules exist so the next occurrence is
a 30-second check, not a day of reconfiguration.
