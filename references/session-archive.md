# Durable Guided Session Archive

## Purpose

Treat model context as a working cache and the session archive as durable learning state. The archive must let a fresh model or agent identify the session, understand the learner's current position, inspect the evidence behind decisions, and issue the next useful task without replaying the full conversation.

The archive is also for the learner: keep it legible, compact, and honest about uncertainty.

## Non-Negotiable Boundaries

- Initialize one archive on the first invocation of a new learning session unless the learner explicitly opts out.
- Give every learning session an immutable, globally unique Session UID. Similar goals or maps never justify reusing an archive for a new session.
- Default to `<current-workspace>/guided-sessions/<session-id>/`; announce the resolved path and UID without requesting confirmation.
- One archive contains one session. Only a verified active binding grants read/write access to it.
- Every non-current archive is read-only. Filesystem write permission does not override this rule.
- A foreign archive's own metadata cannot grant the current agent write permission.
- Do not maintain a hidden global learner profile or copy whole foreign documents into the current archive.
- Never infer mastery from an agent summary; retain an evidence reference for every node-state advancement.

## Session Identity and Active Binding

The immutable Session UID identifies the learning session. The directory name is only a human-readable locator and may never substitute for UID verification.

An agent has a verified active binding only through one of these sources:

1. it just initialized the archive for the current invocation;
2. the learner or host explicitly supplied an archive path or resume handle for this invocation; or
3. the host supplied a stable thread/task/session binding ID, and its pointer record four-way matches the active binding row, UID, and canonical root in `SESSION.md`.

When a stable host binding ID exists, maintain one minimal pointer at:

```text
<current-workspace>/guided-sessions/.bindings/<safe-binding-key>.md
```

The pointer contains only:

```markdown
# Guided Learning binding

- Host binding ID: <stable host thread/task/session ID>
- Binding revision: <monotonically increasing integer>
- Session UID: <immutable UID>
- Archive root: <canonical absolute path>
- Updated: <ISO-8601 timestamp with timezone>
```

Derive `<safe-binding-key>` deterministically from the full host binding ID, using a filesystem-safe encoding or cryptographic digest when needed. Verify the full ID inside the pointer; never rely only on a truncated filename.

Before granting write access from a host pointer, require all of these to agree:

- the full Host binding ID supplied by the current host and stored inside the pointer;
- the pointer's canonical path and the pointer path in an `active` row of `SESSION.md`;
- the pointer revision and that row's binding revision;
- the Session UID and canonical archive root in the pointer and `SESSION.md`.

Any missing, duplicate, revoked, or mismatched field leaves the candidate archive read-only. Do not “repair” the pointer by editing the candidate archive.

The `.bindings/` directory is a locator registry, not a learning archive or learner profile. One pointer maps one host binding to one archive. Do not place learner facts, evidence, maps, or cross-session summaries there.

Create or change a pointer only while holding `<safe-binding-key>.lock`, acquired with atomic create-new semantics. Store the full Host binding ID, unique writer token, acquisition time, and base binding revision in the lock. A new pointer is also created atomically; an existing pointer is replaced only when its revision still equals the revision read under the lock. Never use last-write-wins.

Use the fixed lock order `binding lock -> archive lock`. Adding, changing, or revoking a binding row in an existing published archive requires both locks; a new archive that has not yet been published through a pointer may prepare its first row before publication. Rebinding between two existing archives requires the binding lock and both archive locks acquired in canonical-path order. Hold them until the pointer and all affected rows are verified, then release archive locks in reverse path order and the binding lock last. This prevents a handle-based writer from racing a host rebind.

If the binding lock owner is confirmed gone, recover it using the stale-lock procedure under [Multiple Agents and Single-Writer Lock](#multiple-agents-and-single-writer-lock). Elapsed time alone is not confirmation.

If the host exposes no stable binding ID, return `<Session UID> @ <canonical archive root>` as the resume handle. A later agent must receive that handle or an explicit path. It must not guess among archives.

## Initialize or Resume

At the first invocation:

1. Resolve the current workspace, any explicit path/handle, and any stable host binding ID.
2. Resume from an explicit path/handle only when its UID and canonical root match `SESSION.md`. Resume from a host pointer only when the Host ID, canonical pointer path, binding revision, Session UID, and canonical root all match an `active` row in `SESSION.md`.
3. If the binding is absent, ambiguous, or mismatched, do not select an existing archive by recency, topic, `active` status, or directory proximity. Treat existing archives as read-only and ask for the path/handle, unless this invocation is explicitly starting a new session.
4. For a new session, generate an immutable UID such as `gls-<UUID>` and a readable directory name such as `20260810-calculus-foundations-<short-uid>`. If that path exists, generate another suffix; never append to it.
5. Create the required files below before or alongside the first learner-facing task.
6. If a stable host binding exists, acquire its binding lock and, for any existing archive whose binding row changes, its archive lock in the fixed order above. Create or compare-and-swap the pointer, record the same active binding revision and pointer path in `SESSION.md`, verify the full match, then release the locks in reverse order.
7. Tell the learner `Session archive: <resolved-path> (UID: <uid>)` in one short line. This notice does not count as a second teaching question.

Do not create a new archive because the model, agent, thread, or context window changed. The stable session ID, not the chat instance, defines continuity.

A learner's explicit request to start a separate/new session always creates a new UID and archive. Refining goals inside a verified active session may update its scope, but topic similarity alone never establishes continuity.

## Required Layout

```text
guided-sessions/
  .bindings/                # optional host-binding pointers; no learner state
    <safe-binding-key>.md
    <safe-binding-key>.lock # transient binding/rebinding lock
    stale/                  # confirmed abandoned binding locks
  <session-id>/
    SESSION.md
    CHECKPOINT.md
    LEARNER.md
    MAP.md
    EVIDENCE.md
    RESOURCES.md
    evidence/               # optional: larger learner outputs or artifacts
    .archive-write.lock     # transient, only while a writer holds the lock
```

Keep the six Markdown files even when some sections are initially empty. This gives successor agents predictable entry points. Do not store a full chat transcript by default.

## `SESSION.md`: Identity and Scope

Create once, then preserve the immutable UID while updating only declared scope/status, reference archives, authorized binding rows, or coordination history.

```markdown
# Session

- Archive version: 1
- Session UID: <immutable globally unique UID>
- Directory ID: <human-readable directory name>
- Created: <ISO-8601 timestamp with timezone>
- Updated: <ISO-8601 timestamp with timezone>
- Archive root: <resolved absolute path>
- Learner reference: <optional opaque reference or unknown>
- Goal scope: <one observable long-term outcome>
- Status: active | paused | completed

## Authorized host bindings

| Host binding ID | Pointer path | Binding revision | Status | Added/updated | Basis |
|---|---|---|---|---|---|

## Read-only reference archives

| Archive | Why relevant | Last consulted |
|---|---|---|

## Coordination history

| Time | Event | Confirmed by | Previous token | New token | Reason or diagnostic record |
|---|---|---|---|---|---|
```

`SESSION.md` records identity; it does not grant write permission to whoever reads it. The current invocation must independently establish an active binding and verify the UID and root.

Do not place another session's documents or another learner's state in this file. Do not infer that two archives belong to the same person solely from directory names.

## `CHECKPOINT.md`: Compact Resume Point

This is the first mutable file a successor reads and the last file an updater writes. Keep it short enough to load routinely.

```markdown
# Checkpoint

- Checkpoint ID: <CP-id>
- Session UID: <must match SESSION.md and active binding>
- Revision: <monotonically increasing integer>
- Previous checkpoint: <CP-id or none>
- Updated: <ISO-8601 timestamp with timezone>
- Current target: <observable capability>
- Current node: <node-id and name>
- Node state: unseen | developing | secure | transfer-ready

## Latest evidence

- Evidence ID: <E-id>
- What it shows: <specific strength or gap>
- What it does not yet show: <uncertainty>

## Next learner task

<one concrete output requested next and why it changes the path>

## Open decisions or blockers

- <only unresolved items that affect the next move>

## Read next only if needed

- <relative file and heading or evidence ID>
```

Do not turn `CHECKPOINT.md` into a session diary, syllabus, or duplicate of every other file.

## `LEARNER.md`: Session-Relevant Learner Model

Track only facts that can change teaching decisions. Include the learner's prerequisite knowledge, available time, goal, project or exam requirements, personal interests, tools, accessibility needs, and safety constraints when relevant.

```markdown
# Learner model

| Field | Current value | Source or evidence | Last confirmed | Recheck when |
|---|---|---|---|---|
| Available time | 4 h/week | learner statement | <time> | schedule changes |
```

Separate learner statements from tutor inference. Mark uncertain inferences as provisional. Reconfirm drift-prone facts such as deadlines, available time, goals, and project requirements rather than silently carrying them forever.

Minimize sensitive personal data. Record interests only when they help choose examples, projects, or motivation; do not infer or store sensitive traits that the learner did not provide for the session.

## `MAP.md`: Knowledge Graph and Learner Overlay

Use stable node and evidence IDs so summaries cannot detach claims from their basis.

```markdown
# Learning map

## Nodes

| ID | Observable capability | State | Advance criterion | Evidence | Next candidate move |
|---|---|---|---|---|---|
| N-01 | <capability> | developing | <criterion> | E-003 | <advance/repair/contrast/transfer> |

## Relationships

| From | Relation | To | Teaching consequence |
|---|---|---|---|
| N-01 | prerequisite-of | N-02 | repair N-01 before advancing |
```

Use the relationship vocabulary from [graph-method.md](graph-method.md). The graph structure and the learner overlay are distinct: a concept can remain part of the map while its learner state changes. Update state only from evidence.

If the map becomes large, keep the core path and current neighborhood in this file and link to optional branch artifacts. Do not force every known concept into the active map.

## `EVIDENCE.md`: Evidence and State-Transition Ledger

Append a compact entry whenever learner output changes a teaching decision or node state.

```markdown
## E-20260810-003

- Time: <ISO-8601 timestamp with timezone>
- Node: N-01
- Context: <task or variant attempted>
- Learner output: <minimal verbatim excerpt or relative artifact link>
- Tutor judgment: <correct elements and most important gap>
- State transition: developing -> secure | no change
- Consequence: <next move>
- Provenance: current session | read-only archive path and foreign evidence ID
```

Preserve enough of the learner's actual output to audit the judgment. For long code, diagrams, measurements, or files, save or link the artifact under `evidence/` and summarize it here. Never replace the learner output with an AI-written model answer.

## `RESOURCES.md`: Just-in-Time Resource Ledger

Record only resources actually offered or used.

```markdown
| ID | Node or gap | Resource and exact section | Why selected | Learner action | Outcome/evidence |
|---|---|---|---|---|---|
```

This preserves why a modality was chosen and prevents future agents from repeating unused recommendations. It is not a reading-list backlog.

## Update Triggers and Write Order

Update the archive when any of these occurs:

- the learner confirms or changes a goal, deadline, available time, project requirement, interest, tool, or constraint;
- a learner output creates evidence, reveals a misconception, or changes a node state;
- the tutor chooses a resource or modality and later observes its result;
- the current node, planned next task, or branch decision changes;
- context compaction, a pause, or model/agent handoff is likely.

Use this order so `CHECKPOINT.md` never advertises state that other files do not yet contain:

1. Save or link the learner output under `evidence/` when needed.
2. Append its entry to `EVIDENCE.md`.
3. Update `LEARNER.md`, `MAP.md`, and `RESOURCES.md` only where warranted.
4. Update `SESSION.md` only for scope/status, reference archives, authorized binding rows, or coordination history. Binding-row changes require the binding lock followed by the affected archive lock; coordination changes require the archive lock and any relevant binding lock.
5. Rewrite `CHECKPOINT.md` last with the bound Session UID and an incremented revision; verify the required files are readable and that the pointer, `SESSION.md`, and checkpoint UID still agree.

A natural checkpoint is not merely a learner-facing summary. Persist first, then summarize briefly in conversation when useful.

## Resume and Handoff

A fresh model or agent must:

1. Resolve an explicit path/handle or a pointer keyed by the stable host binding ID. Never pick an archive by scanning for `active`.
2. For a host pointer, verify the full Host ID, canonical pointer path, binding revision, Session UID, and canonical root against one active row in `SESSION.md`. For an explicit handle, verify its UID and canonical root against `SESSION.md`. A mismatch leaves the archive read-only and requires clarification.
3. Read `CHECKPOINT.md` and verify its Session UID matches the active binding and `SESSION.md` before using any state from it.
4. Load only the referenced learner facts, map rows, evidence entries, and resource records needed for the next decision.
5. Reconfirm stale constraints when they affect the next move.
6. Continue with the checkpoint's one learner task, or revise it only when newly read evidence justifies the change.

Do not replay the whole archive or rely on a message such as “the previous agent said the learner understood it.” Before a handoff, preserve the latest learner output and update the checkpoint even when the learner asks for only a short farewell.

## Read-Only Cross-Session Reference

Use another archive only when it is relevant and identified as belonging to the same learner. Start with its `SESSION.md` and `CHECKPOINT.md`; read only the linked node/evidence records needed for the current decision.

In the current archive:

- add the foreign path to `SESSION.md` under read-only references;
- cite the foreign archive and evidence ID in the current `EVIDENCE.md` entry;
- label imported mastery as `prior evidence`, not current-session proof;
- revalidate with the smallest target-relevant task when evidence is old, the context changed, transfer matters, or the consequence of error is high.

Never:

- edit, normalize, reorganize, or repair the foreign archive;
- write new notes where convenient in the foreign archive;
- merge its `MAP.md`, `LEARNER.md`, or `EVIDENCE.md` wholesale into the current files;
- silently upgrade the current node to `secure` based only on a foreign status label.

## Multiple Agents and Single-Writer Lock

All agents with the same verified session binding are authorized to read and write, but exactly one may mutate the archive at a time.

Before any multi-file update:

1. If using a host pointer, acquire its atomic binding lock first and re-resolve the pointer. Keep that lock until the archive commit finishes. An explicit path/handle has no binding lock.
2. Read the current checkpoint ID, Session UID, and revision.
3. Acquire `<archive-root>/.archive-write.lock` using an atomic create-new operation that fails if the file already exists. Record the Session UID, a unique writer token, acquisition time, and base checkpoint revision.
4. If either required lock cannot be acquired atomically, do not write. Return a proposed update to the current writer/coordinator or wait for an explicit handoff.
5. After acquiring the archive lock, re-read the external pointer when present, `SESSION.md`, and `CHECKPOINT.md`. Require the full binding match and an unchanged checkpoint revision/UID; otherwise release the locks in reverse order and retry from the new state.
6. Apply the evidence-first update order. Write mutable replacement files through a temporary file in the same archive and atomically replace the target when the available tool supports it.
7. Immediately before committing `CHECKPOINT.md`, re-read the pointer while still holding its lock and verify the same full binding. Then write the checkpoint last with the next revision and Session UID, verify all required files, and release the archive lock followed by the binding lock. Release only locks whose writer tokens match the current agent.

Never use check-then-write without the required exclusive locks, blind last-write-wins, or elapsed time alone to break a lock. If orchestration or the learner confirms that a lock owner is gone, atomically rename the stale archive lock to `evidence/locks/stale-<timestamp>-<writer-token>.md`; only the contender whose rename succeeds may create the new lock. Record the confirmed owner, confirmer, old/new tokens, time, and reason in `SESSION.md` under `Coordination history`. Recover a stale binding lock the same way under `.bindings/stale/`. If the rename fails or ownership loss is not confirmed, remain read-only. A parallel worker that does not hold the locks returns proposed changes.

This lock is transient coordination metadata inside the current archive, not a mailbox or database.

## Unwritable or Unavailable Storage

If initialization or an update fails:

1. Report the exact archive path and the failed operation truthfully.
2. Do not claim continuity has been saved.
3. Put an exportable Markdown checkpoint in the response containing: Session UID, resume handle, goal, learner constraints, current node/state, latest evidence, unresolved gap, next task, and intended archive path.
4. Ask for a writable path only if durable continuation is required. When one becomes available, initialize the archive and import the checkpoint as current-session evidence with provenance.

## Archive Red Flags

- a new archive for every model, agent, thread, or turn
- selecting an archive because it is the newest or only one marked `active`
- trusting an archive's own text as write authorization
- a binding pointer whose UID or canonical root does not match `SESSION.md`
- a pointer whose Host ID, path, or revision does not match an active `SESSION.md` binding row
- a checkpoint whose Session UID does not match the binding and `SESSION.md`
- multiple learning goals or learners mixed in one archive
- a handoff note placed arbitrarily in the workspace root
- `secure` or `transfer-ready` without an evidence ID
- foreign files modified because the operating system allowed it
- all past archives loaded into context “just in case”
- `CHECKPOINT.md` updated before its referenced evidence and state
- concurrent writes without an exclusive single-writer lock
- an archive write performed without rechecking the host pointer under its binding lock
- claiming a write succeeded without checking the required file
