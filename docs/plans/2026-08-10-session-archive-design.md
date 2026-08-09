# Guided Learning Session Archive Design

## Context

Guided Learning already adapts one learning node at a time from observable learner output. A long-running session, however, will eventually outgrow one model context and may be resumed by another model or agent. Chat history alone cannot be the source of truth for the learning map, learner state, evidence, or next teaching move.

The session therefore needs a durable, human-inspectable archive that is optimized for selective agent reading rather than full transcript replay.

## Goals

- Create and maintain an archive on the first invocation of a new Guided Learning session.
- Use a predictable default location without making the learner approve routine file placement.
- Preserve the learning map, learner constraints, evidence, decisions, and next step across context compaction, model changes, and agent handoffs.
- Keep exactly one learning session in each archive.
- Allow relevant knowledge from another archive for the same learner to be consulted without modifying that archive or silently treating old mastery as current mastery.
- Keep the protocol useful with plain Markdown files; avoid requiring a database, daemon, or orchestration framework.

## Non-goals

- Storing a verbatim transcript by default.
- Creating one global writable learner profile shared by every session.
- Replacing learner evidence with agent summaries.
- Turning every turn into a large documentation operation.
- Copying the full documentation harness from a software repository.

## Chosen Approach

Use a small Markdown archive rooted at:

```text
<current-workspace>/guided-sessions/<session-id>/
```

If the learner explicitly supplies another path before initialization, use it. Otherwise initialize the default path and tell the learner where it is; do not ask for confirmation. If the default cannot be written, state that persistence is unavailable and ask only for a writable location, while returning an exportable checkpoint rather than pretending anything was saved.

The Session UID is immutable and globally unique for the life of the archive. A practical directory name is `<date>-<goal-slug>-<short-uid>`, but the readable name never substitutes for UID verification.

When a stable host task/thread/session ID is available, a versioned pointer under `guided-sessions/.bindings/` maps it to the Session UID and canonical archive root. Write authority requires a four-way match among the current Host ID, pointer path/revision, UID/root, and the archive's active binding row. Otherwise the UID and root are returned as a resume handle. A successor never selects an archive merely because it is recent or marked `active`.

## Archive Layout

```text
guided-sessions/<session-id>/
  SESSION.md
  CHECKPOINT.md
  LEARNER.md
  MAP.md
  EVIDENCE.md
  RESOURCES.md
  evidence/                 # optional larger learner outputs or artifacts
```

- `SESSION.md` is the identity and boundary record: session ID, goal, scope, archive root, creation time, permissions, and any read-only reference archives.
- `CHECKPOINT.md` is the compact handoff entry point: current node, current gap, last learner evidence, immediate next task, unresolved decisions, and pointers to only the files needed next.
- `LEARNER.md` stores session-relevant facts such as prerequisites, available time, goals, project requirements, interests, tools, and accessibility or safety constraints. Each changeable fact includes provenance and last-confirmed time.
- `MAP.md` is the authoritative node graph and learner overlay: node capability, status, relationships, evidence references, and next candidate moves.
- `EVIDENCE.md` is an append-oriented evidence index. It records learner outputs, tutor judgments, node-state transitions, and links to larger files under `evidence/`.
- `RESOURCES.md` records only resources actually offered or used, why they were selected, the requested learner action, and what happened. It is not a link dump.

## Ownership and Isolation

An agent may read and write the current archive only after an explicit path/handle or a four-way verified host-binding pointer establishes the current Session UID and canonical root. An archive's own metadata never grants write permission. Host-bound writes use the fixed order `binding lock -> archive lock`; rebinding between existing archives locks both archives in canonical-path order. This prevents pointer rebinding, explicit-handle writers, and archive mutation from racing. Stale locks are recovered only after confirmed ownership loss by atomically renaming them to unique diagnostic records.

Archives from other sessions are foreign archives. They may be read selectively as references, but are never edited, reorganized, or used as a destination for new notes. The current archive may record a provenance-bearing pointer to foreign evidence; it must not copy foreign documents wholesale or merge their maps and logs into the current archive.

Reading another archive does not by itself make its learner state authoritative. Old mastery, goals, time availability, and project requirements may be stale or context-specific. Imported claims are marked `prior evidence` and are rechecked proportionately before driving a consequential teaching decision.

## Write Protocol

Initialize the archive before or alongside the first learner-facing step. Archive maintenance then happens at meaningful state changes, not after every conversational sentence:

- after the learner establishes or changes the goal, constraints, or project requirements;
- after evidence changes a node status or reveals a misconception;
- after choosing or evaluating a resource;
- before likely context compaction, model change, or agent handoff;
- at a natural pause or end of turn when the next agent would otherwise have to reconstruct state.

For each update:

1. Preserve the learner output or evidence reference.
2. Append the evidence judgment and state transition.
3. Update the learner model or map only where the evidence warrants it.
4. Rewrite `CHECKPOINT.md` last so it points to a consistent state.

Never mark a node secure merely because an agent summary says it was taught. The evidence ID remains the audit trail.

## Resume Protocol

A fresh model or agent begins with `SESSION.md` and `CHECKPOINT.md`, then reads only the referenced sections of `LEARNER.md`, `MAP.md`, `EVIDENCE.md`, or `RESOURCES.md`. It does not replay the whole chat or load the whole archive unless necessary.

Before continuing, it verifies the archive identity, confirms the current root is writable, and treats drift-prone learner facts as provisional when their last-confirmed time or context is no longer adequate.

## Validation Scenarios

The revised Skill must handle at least these cases:

1. First invocation with no path: create the default archive, announce its path, and still obey the one-question/one-task first-reply invariant.
2. Context limit and handoff pressure: persist evidence and update the checkpoint before handing off instead of relying on a prose chat summary.
3. Fresh agent resume: read the compact entry files first and continue from the current node without reconstructing everything.
4. Cross-session reference pressure: consult a relevant foreign archive read-only, retain provenance, and revalidate context-sensitive mastery.
5. Unwritable filesystem: disclose the limitation and emit an exportable checkpoint without claiming a successful save.

## Files to Change

- Add the mandatory lifecycle and boundaries to `SKILL.md`.
- Add detailed schemas and templates in `references/session-archive.md`.
- Update `agents/openai.yaml` so the default invocation mentions durable session state.
- Update `README.md` to document archive behavior and repository structure.
