# Guided Learning Session Archive Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Make every new Guided Learning session initialize and maintain an isolated, durable, agent-readable archive that survives long contexts and cross-agent handoffs.

**Architecture:** Keep the mandatory behavior short and prominent in `SKILL.md`, with the full Markdown archive schema and update/resume protocols in one progressive-disclosure reference. Store one session per writable archive; consult other session archives read-only with provenance and proportional revalidation.

**Tech Stack:** Markdown skill instructions, YAML agent metadata, Codex skill validator, subagent pressure tests, Git/GitHub.

---

### Task 1: Capture the Old-Skill Baseline

**Files:**
- Create outside repository: `E:/learn/AI_infra/tmp/guided-learning-evals/red/*.md`

**Step 1: Run failing pressure scenarios**

Run fresh agents without the proposed persistence rules against: first invocation, context-limit handoff, cross-session reference, and unwritable-filesystem scenarios.

**Step 2: Verify RED**

Expected: at least one agent relies on chat summary, omits default archive initialization, mixes or overwrites foreign state, or claims continuity without a durable write.

**Step 3: Record exact behavior**

Save verbatim responses and summarize the observed loopholes. Do not edit the Skill before the failures are captured.

### Task 2: Add the Durable Archive Contract

**Files:**
- Modify: `SKILL.md`
- Create: `references/session-archive.md`

**Step 1: Add the minimal mandatory rules**

Add first-invocation initialization, immutable Session UID, host-binding pointer or resume handle, default-path notification, verified current-root read/write ownership, foreign-root read-only boundaries, checkpoint timing, and truthful fallback behavior to `SKILL.md`.

**Step 2: Add the detailed archive reference**

Define file purposes, templates, initialization, UID/root binding verification, write ordering, resume reads, cross-session provenance, exclusive single-writer locking, conflict handling, and no-filesystem fallback in `references/session-archive.md`.

**Step 3: Check progressive disclosure**

Confirm `SKILL.md` contains every non-negotiable behavior while delegating only detailed schemas and examples to the reference.

### Task 3: Update User-Facing Metadata

**Files:**
- Modify: `agents/openai.yaml`
- Modify: `README.md`

**Step 1: Update metadata**

Keep the short description within 25–64 characters and update the one-sentence default prompt to mention maintaining the session archive.

**Step 2: Update README**

Explain the default archive path, isolation rule, cross-session read-only references, and the new reference file in the repository tree.

### Task 4: Verify GREEN and Close Loopholes

**Files:**
- Create outside repository: `E:/learn/AI_infra/tmp/guided-learning-evals/green/*.md`

**Step 1: Run the same pressure scenarios with the revised Skill**

Expected: every agent initializes or resumes only a UID/root-verified archive, never guesses among active archives, writes only the current root while holding the exclusive lock, uses compact checkpoints, and treats foreign archives as read-only prior evidence.

**Step 2: Meta-test any failure**

Ask the failing agent what wording allowed the violation. Add only the smallest explicit counter, then rerun that scenario.

**Step 3: Run structural validation**

Run:

```powershell
D:\apps\miniforge3\envs\ENV\python.exe -X utf8 C:\Users\Administrator\.codex\skills\.system\skill-creator\scripts\quick_validate.py E:\learn\AI_infra\tmp\guided-learning-skill
```

Expected: `Skill is valid!`

**Step 4: Check references and forbidden ambiguity**

Run targeted `rg` checks for the default path, write boundary, reference link, and archive fallback. Inspect `git diff --check` and the complete diff.

### Task 5: Deploy and Publish

**Files:**
- Replace matching files under: `C:/Users/Administrator/.codex/skills/guided-learning/`
- Publish matching repository files to: `RAYaRayaRay/guided-learning-skill`

**Step 1: Deploy the verified files locally**

Copy only the changed Skill files after verification, preserving unrelated local files.

**Step 2: Validate the installed path**

Run `quick_validate.py` against the installed directory and compare hashes with the verified worktree.

**Step 3: Publish intentionally**

Commit the reviewed changes with a focused message and update GitHub without force-pushing or modifying unrelated files.

**Step 4: Verify remote state**

Fetch the published files through the GitHub connector and compare their content or SHA with the verified local versions.
