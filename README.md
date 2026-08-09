# Guided Learning

An adaptive, evidence-driven tutoring skill for Codex-compatible AI agents.

Instead of generating a complete study pack up front, the skill:

- diagnoses the learner with one small, path-determining task;
- advances through a provisional knowledge structure one node at a time;
- requires observable learner output at core nodes;
- adapts explanations, resources, and activities to that evidence;
- maintains durable session state across long contexts, models, and agents;
- uses modalities only when they solve a real learning bottleneck;
- embeds assessment throughout learning instead of relying on one final quiz.

## Install

```powershell
git clone https://github.com/RAYaRayaRay/guided-learning-skill.git "$env:USERPROFILE\.codex\skills\guided-learning"
```

Restart Codex if the skill is not discovered immediately.

## Use

```text
Use $guided-learning to help me learn attention mechanisms through an interactive session.
```

It can also be selected automatically for personalized understanding, practice, troubleshooting, self-assessment, study planning, interviews, projects, and research preparation.

## Modes

- **Guided session:** adaptive dialogue driven by learner evidence.
- **Artifact mode:** sequential guides, graphs, notebooks, diagnostics, or plans when explicitly requested.
- **Direct answer:** bounded factual questions without a forced tutoring ritual.

## Session archives

On the first invocation for a new learning session, the skill creates and maintains:

```text
<current-workspace>/guided-sessions/<session-id>/
```

The learner is told the resolved path and immutable Session UID; the default does not require confirmation. When the host provides a stable task/thread ID, a versioned pointer under `guided-sessions/.bindings/` lets a fresh agent resolve and verify the active archive. Otherwise the UID and path form the resume handle. Agents may write only when the Host ID, pointer path/revision, UID, and canonical root agree with the archive's active binding record; an archive cannot grant itself write permission.

Archives for other learning sessions are read-only references: relevant prior evidence may inform the current path, but it keeps its provenance and is revalidated when stale or context-dependent. Same-session agents are all authorized to update the current archive, while a small exclusive lock serializes actual concurrent writes.

Each archive contains one learning session only, with compact entry files for session identity, the learner model, knowledge map, evidence, resources, and the next handoff checkpoint. If storage is unavailable, the skill reports the failure and returns an exportable checkpoint instead of claiming progress was saved.

## Repository structure

```text
SKILL.md
agents/
  openai.yaml
references/
  diagnostic-planning.md
  graph-method.md
  output-blueprints.md
  resource-strategy.md
  session-archive.md
docs/plans/
  2026-08-10-session-archive-design.md
  2026-08-10-session-archive-implementation.md
```

`$deep-research` is an optional companion skill when current, multi-source research is required.

## 中文简介

该 Skill 将 AI 从“一次性学习资料生成器”转变为自适应学习导师：先获取学生的真实输出，再围绕关键知识节点诊断、反馈和推进；资源与多模态内容只在解决当前学习障碍时提供。每个学习 Session 默认建立独立档案库，使学习地图、学习者状态、证据和下一步行动能够跨越超长上下文、模型与 Agent 延续；其他 Session 档案只读参考，不会被当前 Session 改写。

## License

MIT
