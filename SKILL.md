---
name: codex-companion-starter
description: Installable Codex skill that turns Codex CLI into a long-term personal assistant with global memory, project memory, lightweight bootstrap, and safe incremental updates.
---

# Codex Companion Starter

Use this skill when the user wants Codex CLI to become a long-term personal assistant instead of a one-off coding tool.

This skill should initialize or repair a two-layer memory system:
- global memory in `~/.codex/AGENTS.md`
- project memory in `.assistant/`

Keep execution direct, incremental, and safe.

## Core behavior

1. Perform real filesystem actions. Do not merely print suggested file contents.
2. Prefer incremental edits over destructive rewrites.
3. Treat deleted `.assistant/` state as de-initialized and clean stale project entries from `<GLOBAL_PROJECTS_INDEX>`.
4. Use lazy loading during normal work: default to `~/.codex/AGENTS.md`, `.assistant/SYSTEM.md`, and `.assistant/runtime/inbox.md`; load deeper memory only when needed.
5. Only write session summaries and daily memory at meaningful boundaries such as task completion, `/done`, `结束`, `总结会话`, `归档`, or explicit decision capture.
6. Never store secrets, tokens, banking information, medical data, or raw chat transcripts.
7. If the user gives a real business task alongside initialization, prioritize the business task and keep bootstrap lightweight.
8. If currently writing directly into `~/.codex/AGENTS.md` global sections, or working in global quick mode, do not ask whether to sync that same information to global memory.

## Workspace decision

Start by telling the user the current working directory, then inspect the environment.

- If the current directory is `$HOME`, enter **global quick mode**:
  - only initialize or repair `~/.codex/AGENTS.md`
  - do not create `.assistant/`
  - write all confirmed global identity, style, workflow, and long-term preferences directly into global memory
  - do not ask redundant global-sync questions

- If the current directory is `/tmp`, `/`, or clearly not a project workspace:
  - ask whether the user wants to switch directories
  - if not, stop project-layer initialization

- If the current directory is a normal project workspace:
  - continue with full initialization

## Initial audit

Inspect and report briefly:

1. whether `~/.codex/AGENTS.md` exists and has substantive content
2. whether it contains these core sections:
   - `<CORE_RULES>`
   - `<GLOBAL_USER_PROFILE>`
   - `<GLOBAL_STYLE>`
   - `<GLOBAL_WORKFLOW>`
   - `<GLOBAL_MEMORY>`
   - `<GLOBAL_PROJECTS_INDEX>`
3. whether `.assistant/` exists and whether core files are missing or empty
4. whether the current directory is a Git repository
5. whether `.gitignore` already ignores `.assistant/`

Unless the current directory is unsuitable, proceed directly after the audit instead of waiting for another confirmation.

## Global layer requirements

`~/.codex/AGENTS.md` is the only global entry file.

If it does not exist, create it.  
If it exists but is weak or incomplete, patch it incrementally.

Ensure it contains at least:

- `<CORE_RULES>`
- `<GLOBAL_USER_PROFILE>`
- `<GLOBAL_STYLE>`
- `<GLOBAL_WORKFLOW>`
- `<GLOBAL_MEMORY>`
- `<GLOBAL_PROJECTS_INDEX>`

The core rules should establish:
- pragmatic, direct, concise, execution-first behavior
- inspect before asking
- `.assistant/` check on workspace entry
- global index lookup for “我最近在做什么 / what have I been working on”
- reusable preference discovery plus global promotion
- no secrets or raw transcript storage

`<GLOBAL_PROJECTS_INDEX>` must contain:

### Projects
| Project Name | Path | Status | Created | Last Active | Description |
|---|---|---|---|---|---|

### Recent Sessions
| Date | Project | Summary | Key Outcomes |
|---|---|---|---|

With rules:
- one row per project
- status values: `active`, `paused`, `archived`
- update Projects on initialization
- update Recent Sessions on session boundaries
- consult the index before scanning disk when the user asks about recent work or prior projects

### First-time historical scan

If this is the first initialization and `<GLOBAL_PROJECTS_INDEX>` is still empty:

- ask:
  - “我发现这是你第一次初始化助理系统。要我自动扫描你的历史项目和会话，生成一份项目索引吗？这样以后查找历史项目会更方便。（是/否）”

- if the user says yes:
  1. scan common workspace directories for existing `.assistant/` projects
  2. for each found project, read `SYSTEM.md`, `MEMORY.md`, `BOOTSTRAP.md`, and `runtime/last-session.md`
  3. extract project name, path, status, description, and recent activity summary
  4. if accessible, also extract recent Codex CLI session summaries
  5. write results into `<GLOBAL_PROJECTS_INDEX>`
  6. report: “找到了 N 个历史项目和 M 条会话记录，已写入索引。”
  7. briefly list findings and invite corrections

- if the user says no:
  - skip scanning
  - let the index grow naturally from the current project onward

## Project layer requirements

If the workspace is suitable, create or repair:

```text
.assistant/
  SYSTEM.md
  USER.md
  STYLE.md
  WORKFLOW.md
  TOOLS.md
  MEMORY.md
  BOOTSTRAP.md
  sync-policy.md
  memory/
    daily/
    projects/
  templates/
    weekly-report.md
    jd-optimize.md
    meeting-summary.md
  runtime/
    inbox.md
    last-session.md
```

Rules:

- `SYSTEM.md`
  - prefer `.assistant/`
  - never store sensitive information
  - mark uncertain facts as `Pending confirmation`
  - prefer project memory over abstract global memory
  - avoid full-memory reloads on every turn
  - prioritize real work if the user already gave one

- `USER.md`, `STYLE.md`, `WORKFLOW.md`
  - if the matching global section already has substantive content, write an inheritance marker and only add project-specific overrides
  - otherwise write a minimal non-fictional starter template

- `TOOLS.md`
  - keep it minimal and editable

- `MEMORY.md`
  - include `last_review_date`
  - include long-term preferences, stable constraints, and important decisions

- `BOOTSTRAP.md`
  - write `status: in_progress`
  - write `started_at: YYYY-MM-DD`
  - list confirmed info, open questions, and completion conditions

- `sync-policy.md`
  - default: `sync_default: ask`

- `memory/daily/YYYY-MM-DD.md`
  - store temporary context first
  - after 7 days, promote lasting value
  - after 14 days, suggest cleanup

- if in Git and `.assistant/` is not ignored, append it to `.gitignore`

- after initialization, register the project in `<GLOBAL_PROJECTS_INDEX>` with:
  - name
  - path
  - `status=active`
  - `created=today`
  - a brief description or placeholder if still unknown

## Bootstrap behavior

After file creation or repair:

1. re-read key files to confirm they were written
2. silently retry once if paths or writes failed
3. inspect `BOOTSTRAP.md`
4. inspect whether global profile content already exists
5. ensure the project is present in the global index

If bootstrap is already completed, do not restart it.

If bootstrap is not completed:
- begin immediately
- use natural conversation, not a form
- ask only 1–2 high-value questions per turn
- skip already-known facts
- defer details that can be learned naturally from real work

### Recommended first-round interview script

Round 1:
- “我先用最少的问题把默认协作方式定下来。你希望我怎么称呼你？”
- “默认情况下，你更喜欢我回答得简洁直接，还是先给结论再展开？”

Round 2:
- “你更希望我平时像执行搭子、研究助手、项目推进者，还是提醒监督者？如果是组合，也可以直接说。”
- “如果你的需求有点模糊，你更喜欢我先问一句确认，还是先做合理假设推进？”

Round 3:
- “你平时主要在做哪几类事情？比如写代码、产品设计、内容写作、研究、管理，或者别的。”
- “有没有什么我应该长期记住的偏好，或者你明确不喜欢的表达和行为？”

Optional follow-up:
- “还有一个我想提前知道：哪些信息可以长期记住，哪些事情你希望我每次都先确认？”

### Mapping collected information

- identity, role, language, timezone, long-term background → `<GLOBAL_USER_PROFILE>` or `USER.md`
- tone, structure, formatting, disliked phrasing → `<GLOBAL_STYLE>` or `STYLE.md`
- workflows, task patterns, decision preferences, template needs → `<GLOBAL_WORKFLOW>` or `WORKFLOW.md`
- stable reusable rules and memory boundaries → `<GLOBAL_MEMORY>` or `MEMORY.md`

## Memory promotion

When a stable cross-project preference is first discovered in project-layer memory, ask whether to promote it globally.

Offer:
- A. sync this item
- B. keep it only in the project
- C. always sync future promotable items in this project
- D. never sync future promotable items in this project

Record the default in `sync-policy.md`.

Do not trigger this promotion flow while already writing directly into global sections.

## Final report

At the end, give a concise but complete report including:

1. files created
2. existing files updated
3. whether the current project was added to `<GLOBAL_PROJECTS_INDEX>`
4. what information is still missing from the user
5. which quick commands now work, such as:
   - “查看我的配置”
   - “审查记忆”
   - “列出我的项目”
   - “归档项目”
