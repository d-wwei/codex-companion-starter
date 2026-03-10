# Codex Companion Starter

[中文说明](./README.zh-CN.md)

> **Looking for other CLI assistants?** 
> Check out the sister extensions: [Claude Code Companion Starter](https://github.com/d-wwei/claude-code-companion-starter) & [Gemini Companion Starter](https://github.com/d-wwei/gemini-companion-starter)

Turn Codex CLI into a directly installable, workspace-aware, long-term personal assistant skill.

Prompt templates for people who want Codex CLI to do more than answer one-off questions. These prompts help Codex:

- keep a stable collaboration style
- initialize local project memory structure
- avoid unnecessary rewrites
- bootstrap user preferences in a lightweight way

The pack is intentionally split into three variants, so you can choose how cautious or proactive Codex should be in your environment.

## Why This Exists

Raw prompts often overfit to an idealized agent model. In practice, Codex CLI behaves better when instructions are:

- explicit about file operations
- careful about existing content
- realistic about what global config can and cannot control
- lightweight enough to avoid blocking normal work

These prompts are written around that reality.

## Key Features

This prompt transforms Codex CLI from a powerful but amnesic one-shot tool into a long-term collaborator that builds rapport with you.

- **Long-term Memory Setup:** Automatically establishes persistent global rules (`~/.codex/`) and project-specific memory (`.assistant/`).
- **Preference Accumulation:** Remembers how you like things done—your role, coding style, workflow, and recurring instructions.
- **Global Memory Promotion:** As it discovers your habits and reusable knowledge during active projects, it can automatically promote them to your global profile.
- **Project-Specific Customization:** Every project can have its own tailored rules and context that override the global defaults.
- **Continuous Collaboration:** You never have to explain your background, structure, or current progress from scratch again when re-entering a workspace.
- **Layered Bootstrap Interview:** The unified prompt now uses a compact 3-step interview that captures naming, style, assistant role, ambiguity handling, work types, and memory boundaries without blocking real work.
- **Global Quick Mode:** When run from `$HOME`, Codex updates only `~/.codex/AGENTS.md` and does not ask whether globally written information should be synced to global memory.
- **Historical Project Scan:** First-time setup can now scan older `.assistant/` workspaces, extract project/session summaries, and register them into the global projects index.
- **Task-Level Resume Checkpoints:** For non-trivial implementation work, Codex can maintain a module-local `PROGRESS.md` so a new process can quickly resume from the exact acceptance step instead of inferring progress from scratch.

## Why OpenClaw-Inspired Memory For Codex CLI

Codex CLI and OpenClaw are strong in different ways.

Codex CLI is naturally good at instruction layering and execution. Its `AGENTS.md` model is useful for defining behavior by scope: global defaults, repository rules, and local overrides. That makes it strong at policy and workflow control, but not inherently strong at long-lived structured memory. [Codex AGENTS.md guide](https://developers.openai.com/codex/guides/agents-md)

OpenClaw takes a different approach. Its memory model treats Markdown files as durable working memory, with daily logs and longer-term memory living in the workspace as explicit artifacts. That makes it much better at preserving collaboration context over time. [OpenClaw memory docs](https://docs.openclaw.ai/concepts/memory)

This repository borrows that memory philosophy and adapts it to Codex CLI.

The goal is not to pretend Codex CLI has native persistent memory in the same sense. The goal is to use the file system as an external memory layer, so Codex can behave less like a stateless tool and more like a repeatable long-term collaborator.

### What This Changes

- Rules and preferences stop living only in chat history.
- Project decisions can survive across sessions.
- Temporary notes and stable preferences can be separated cleanly.
- Codex can re-enter a workspace with better continuity.
- **NEW**: Global preferences, style, and memory are now tracked cross-project via `~/.codex/AGENTS.md`.
- **NEW**: A Global Projects Index tracks your active workspaces and session history.
- **NEW**: The unified prompt includes a "Memory Promotion Flow" to seamlessly elevate recurring project habits into global defaults.

### Why It Feels More Human

The improvement is not about making Codex sound warmer. It is about making collaboration feel continuous.

With only raw prompts, you often have to restate:

- how you want answers structured
- what role you play in the workspace
- which tradeoffs the project already chose
- where the last session stopped

With a memory layout such as `.assistant/STYLE.md`, `.assistant/WORKFLOW.md`, `.assistant/MEMORY.md`, and `memory/projects/*.md`, those details become editable project assets instead of fragile conversational leftovers.

For active implementation work, a nearby `PROGRESS.md` adds one more layer: a very small task-local checkpoint file that says what was already finished, what is currently in progress, and what should happen next.

That makes Codex more likely to:

- respond in your preferred format
- avoid repeating already-settled project debates
- preserve the way you like work to be pushed forward

### Why It Can Grow With The User

`AGENTS.md` is best for stable rules, but real collaboration also includes evolving preferences, recurring habits, project-specific decisions, and temporary context. 

The enhanced **Unified Prompt** splits `AGENTS.md` into four distinct global zones:
1. `<GLOBAL_USER_PROFILE>`: Your identity across all projects.
2. `<GLOBAL_STYLE>`: Your default communication preferences.
3. `<GLOBAL_MEMORY>`: Cross-project reusable technical knowledge.
4. `<GLOBAL_PROJECTS_INDEX>`: A registry of all active projects and recent sessions.

A good external memory layer lets Codex grow with the user by gradually capturing:

- stable style preferences
- recurring workflows
- project constraints and decisions
- unfinished work from recent sessions

This is a better match for long-term use than trying to stuff everything into one unstructured rules file.

### Concrete Examples

1. **Style becomes stable across sessions.**  
   Instead of repeating "be concise, give the conclusion first, then risks", those preferences can live in `STYLE.md`.

2. **Project decisions stop resetting.**  
   If a project already decided not to split a service yet, that can live in `memory/projects/architecture.md` instead of being re-debated from scratch.

3. **Workflow becomes personalized.**  
   If you prefer "inspect first, explain the edit briefly, then change files, then report verification", that can live in `WORKFLOW.md`.

4. **Temporary context stops polluting long-term rules.**  
   Short-lived notes such as "this API doc is still unverified" belong in `memory/daily/YYYY-MM-DD.md`, not in permanent memory.

5. **Cross-Project Synergy (New).**  
   If Codex notices you always prefer ESLint over Prettier in 3 different projects, it will proactively ask: *"Should I promote this preference to your global AGENTS.md?"*

6. **Lazy Loading (New).**  
   Codex now intentionally *lazy-loads* memory. It reads `SYSTEM.md` and your `inbox.md` first, only fetching deeper archival memory when the context actually demands it, saving tokens and improving speed.

7. **Task Resumption (New).**  
   If work stops halfway through a feature, a module-local `PROGRESS.md` can tell the next Codex process which acceptance items are done and what the next concrete step is.

The recommended `PROGRESS.md` shape is:

```md
status: in_progress
task: Add task-level progress recovery
module_path: packages/assistant-memory/

# 开发进度

## 已完成
- [x] 明确 `PROGRESS.md` 使用模块局部文件而不是 `.assistant/`
- [x] 加入恢复口令：继续上次进度 / 恢复进度
- [x] 定义恢复时的候选定位顺序

## 进行中
- [ ] 把恢复逻辑接入当前模块的初始化流程

## 待做
- [ ] 补充恢复话术模板
- [ ] 增加多候选 `PROGRESS.md` 的确认逻辑
- [ ] 完成一次中断恢复流程验证

## 关键决策
- `PROGRESS.md` 放在实际模块目录，避免所有任务共用一份中心状态文件
- 恢复时只读取最相关的 1-2 个候选，减少 token 消耗

## 已知问题
- 当前模块还没有验证“多个候选进度文件”时的选择行为
```

Update it every time an acceptance item is completed. For explicit recovery, the user can say `继续上次进度`, `恢复进度`, `resume progress`, or `continue from progress`.

When recovering, the assistant should locate the most relevant `PROGRESS.md` in this order: current working directory, most recently modified module, user-named module, then best keyword-matching module. It should read only the top 1-2 candidates instead of scanning every progress file in the repo.

The recommended resume wording is:

```text
我找到了这份进度记录：
- 已完成：...
- 进行中：...
- 下一步：...

要我按这份进度继续吗？
```

### The Practical Meaning

This approach does not replace Codex CLI's core behavior.

It adds a structured external memory layer on top of what Codex already does well:

- Codex provides instruction loading and execution
- the file system provides durable collaboration memory
- this prompt pack connects the two

That is the real point of the project: not just making Codex act smarter, but making it easier to work with over time.

## What's Included

- [SKILL.md](./SKILL.md)  
  The installable root skill for Codex.

- [agents/openai.yaml](./agents/openai.yaml)  
  UI metadata for Codex skill installation and invocation.


## Install

You can install this repository directly as a Codex skill from GitHub.

Example:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo d-wwei/codex-companion-starter \
  --path .
```

After installation, restart Codex.

## Quick Start


1. Install the skill.
2. Restart Codex.
3. In a target workspace, invoke it explicitly, for example:
   - `Use $codex-companion-starter to initialize this workspace for long-term collaboration.`
4. Let Codex inspect the workspace, update config, and start lightweight bootstrap if appropriate.

## Recent Improvements

- stronger first-round interview for long-term user profiling
- explicit global quick mode with no redundant global-sync question
- fuller historical project and session scanning flow
- better alignment between bootstrap questions, memory promotion, and delivery reporting
- clearer guidance for incremental updates instead of overwriting existing config
- task-level `PROGRESS.md` checkpoints for interrupted implementation work

## Installable Skill Layout

```text
SKILL.md
agents/
  openai.yaml
```

This repository now follows the standard Codex skill shape, so the repo itself can be installed instead of requiring a nested `skill/` path.

## Design Principles

- Default preference layer, not hard control layer
- Incremental edits over destructive rewrites
- Project-local memory over fake global omniscience
- Lightweight bootstrap over long onboarding forms
- Realistic compatibility with Codex CLI behavior

## Important Boundaries

- `~/.codex/AGENTS.md` should be treated as a default preference layer, not a permanent hard control mechanism.
- `.assistant/` is a local project convention for memory and collaboration context, not a built-in Codex CLI protocol.
- Actual behavior still depends on runtime instructions, tool permissions, and higher-priority context.

## FAQ

### Why not just use a single `AGENTS.md`?

Because `AGENTS.md` is best at behavior rules, not at storing evolving collaboration context.

If you put everything into one file, it becomes a mix of:

- hard rules
- style preferences
- project history
- temporary notes

That is harder to maintain and easier to stale out. A small memory structure keeps those concerns separated.

### Doesn't `.assistant/` make the workspace too heavy?

It can, if overdesigned.

That is why this prompt pack keeps the structure intentionally small and encourages incremental filling instead of full upfront documentation. The goal is not bureaucracy. The goal is just enough local memory to reduce repeated friction.

### Who is this for?

It is a good fit for people who use Codex CLI repeatedly in the same projects and want more continuity across sessions.

It is less useful if you only use Codex for quick one-off prompts and do not want any local workflow structure.

### Is this trying to turn Codex into OpenClaw?

No.

This project does not replace Codex CLI's native behavior. It borrows one useful idea from OpenClaw: treating the file system as a durable memory layer. The result is still Codex CLI, just with a more structured collaboration surface.

## Who This Is For

This project is useful if you want Codex CLI to behave more like:

- a repeatable project assistant
- a workspace-aware collaborator
- a tool that can preserve lightweight context across tasks

It is probably not useful if you only want quick one-off prompts with no local structure.

## Notes

If Codex behaves too aggressively in your environment, fall back to `safe`.

If it is too conservative and does not make enough progress, move to `strong`.
