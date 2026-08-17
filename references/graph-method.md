# Knowledge and Evidence Graph Method

## Purpose

Use the graph to decide what the learner should do next, not merely to display a topic taxonomy. Relationships explain why a node comes now, what it unlocks, or which distinction prevents a mistake; do not build a large ontology just to name every edge.

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

Use only relationships that change teaching: dependency, part-whole, explanation/mechanism, contrast/confusion, alternate representation, application bridge, generalization/extension, or demonstrated-by evidence. Name them plainly (for example, `prerequisite-of`, `part-of`, `explains`, `often-confused-with`, `another-view-of`, `applies-to`, `extends-to`, and `demonstrated-by`).

## Interaction Rules

Keep the graph provisional and reveal the smallest useful level: a **local neighborhood** (completed node, current transition, and destination), a **stage path** (the current milestone with relevant prerequisites and outcomes), or the **full active map** (core path, branches, and confusions). Start local. Expand when the learner enters a bridge or consequential junction, is confused about fit, must choose a branch, returns after a handoff or pause, changes goal or constraints, asks for orientation, or needs a coherent-cluster review. Never expand by turn count. Update state only from observed learner output.

Do not ask for evidence from several unseen nodes at once. Review one node's output, then decide whether to advance, repair a prerequisite, contrast a confusion, or test transfer. At a consequential junction, do not advance to `secure` from a familiar isolated exercise if its important relationship is still misunderstood; ask for the smallest relationship output instead, such as “Why does the limit step enable the derivative?”, “Place this node before or after its prerequisite and say why,” or “Which representation would you use for this application?” Ordinary leaf nodes need no such ritual.

## Graph Artifact

When a full graph is requested, show dependency order, required versus optional branches, and common confusions. Add a compact checkpoint table:

| Core node | Learner output | Advance criterion | Pause |
|---|---|---|---|

Use Mermaid only when visual structure materially helps. A plain outline is better when the graph is small.

## Compression Rules

- Merge synonyms and ornamental leaves.
- Keep the core trunk finishable.
- Separate `understand`, `perform`, and `transfer`.
- Remove any node or edge that does not change sequence, explanation, evidence, or transfer.
