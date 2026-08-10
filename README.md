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

Each new learning session gets its own archive under:

```text
<current-workspace>/guided-sessions/<session-id>/
```

The archive preserves learning progress across long contexts and agent handoffs. The active session may update its own archive; other session archives are read-only references. If progress cannot be saved, the skill says so explicitly.

See [references/session-archive.md](references/session-archive.md) for the agent-facing protocol.

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
```

`$deep-research` is an optional companion skill when current, multi-source research is required.

## 中文简介

该 Skill 将 AI 从“一次性学习资料生成器”转变为自适应学习导师：先获取学生的真实输出，再围绕关键知识节点诊断、反馈和推进；资源与多模态内容只在解决当前学习障碍时提供。每个学习 Session 默认建立独立档案，以便跨越长上下文和 Agent 交接继续学习；其他 Session 档案仅作只读参考。

## License

MIT
