# Knowledge and Evidence Graph Method

## Purpose

Use the graph to decide what the learner should do next, not merely to display a topic taxonomy.

## Build Order

1. Define the observable target capability.
2. Identify only prerequisites that can block it.
3. Extract the minimum core trunk.
4. Add theory-to-application bridge nodes.
5. Mark common confusions and optional branches.
6. Attach one proportionate evidence task and advance criterion to every core node.

## Node Record

For each core node, track:

- concept or capability
- prerequisites
- why it matters to the target
- likely misconception
- learner state: `unseen`, `developing`, `secure`, or `transfer-ready`
- smallest useful learner output
- criterion for advancing

Useful outputs include teach-back, comparison, prediction, calculation, code modification, measurement, diagnosis, design choice, or claim-evidence analysis.

## Relationships

- `prerequisite-of`
- `supports`
- `often-confused-with`
- `leads-to`
- `optional-extension-of`
- `demonstrated-by`

## Interaction Rules

Keep the full graph provisional and internal during a guided session. Reveal the current path and next node; show the whole graph only when it helps orientation or the learner asks for it. Update state only from observed learner output.

Do not ask for evidence from several unseen nodes at once. Review one node's output, then decide whether to advance, repair a prerequisite, contrast a confusion, or test transfer.

## Graph Artifact

When a full graph is requested, show dependency order, required versus optional branches, and common confusions. Add a compact checkpoint table:

| Core node | Learner output | Advance criterion |
|---|---|---|

Use Mermaid only when visual structure materially helps. A plain outline is better when the graph is small.

## Compression Rules

- Merge synonyms and ornamental leaves.
- Keep the core trunk finishable.
- Separate `understand`, `perform`, and `transfer`.
- Remove any node that does not change sequence, evidence, or decisions.
