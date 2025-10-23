# SuperBot

A product-driven AI programming collaboration skill — automatically manages product SPEC files and persistently captures product decisions in real time.

[中文](./README_ZH.md)

## What does it do?

When collaborating with AI on development, product decisions often get scattered across conversation history and become hard to trace. SuperBot solves this:

- **Auto-identifies product decisions**: AI recognizes confirmed requirements, architecture choices, UI specs, and other key decisions during conversation
- **Writes to SPEC files in real time**: Decisions are automatically distilled into structured product documents
- **Unified product understanding**: Anyone (or any AI) can read the SPEC files to quickly understand what the product should be

## Installation

```bash
npx skills add https://github.com/nileader/superbot --skill superbot
```

## Features

- Auto-activates on new sessions, reads `.superbot/superbot.json` config
- Manages 6 SPEC files under `spec_dir` (product definition, user story, feature list, UI spec, tech spec, others)
- Captures product decisions into SPEC files in real time during conversation
- Manual invocation via `/superbot`
- Force initialization via `/superbot init` — for first-time setup or re-initialization
- Interactive config update via `/superbot config`
- Batch SPEC update via `/superbot update spec` — extracts all valuable decisions from current session and updates SPEC files

## PRD Skill — Workflow

**Background**: 产品经理已通过 Coding Agent 完成系统 Demo 实现，但当前研发流程规范要求产品经理只交付 PRD，具体实现由研发团队完成。本 skill 将已实现的 Demo 转化为标准 PRD 交付物。

```
┌─────────────────────────────────────────────────────────────┐
│  第一步：确认功能模块                                         │
│  ─────────────────                                          │
│  询问用户"针对哪个功能模块？"                                 │
│  ┌─ URL → 提取路径 → 在 src/pages/views/app 中查找           │
│  └─ 文件路径 → 检查是否存在                                 │
│  失败 → 告知用户，不得继续推测                                │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第二步：读取并分析功能模块                                   │
│  ─────────────────                                          │
│  深度阅读源码：页面结构、交互逻辑、数据流、接口调用、          │
│  边界处理、路由配置                                          │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第三步：确认 PRD 输出路径                                    │
│  ─────────────────                                          │
│  询问用户"PRD 文件输出到哪里？"                               │
│  已存在 → 更新；不存在 → 新建                                 │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第四步：重新读取 PRD 与代码                                  │
│  ─────────────────                                          │
│  1. 如果 PRD 文件存在 → 重新读取全文                          │
│  2. 重新分析最新源码 → 代码是唯一事实来源，PRD 仅供参考        │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第五步：生成或更新 PRD                                       │
│  ─────────────────                                          │
│  新建：读取模板 → 填写可从代码推导的内容                       │
│  更新：逐节对比 PRD vs 代码 → 只改不一致处，不重写全文         │
│  无法推导的内容先预留，不得留空抛给用户                        │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第六步：逐一确认待推导内容                                   │
│  ─────────────────                                          │
│  按顺序逐一确认（每次一个字段，给出推荐值）：                    │
│    ① 功能名称                                               │
│    ② 所属模块                                               │
│    ③ 作者（默认 $USER）                                     │
│    ④ 1.1 背景                                               │
│    ⑤ 1.2 目标                                               │
│    ⑥ 1.3 Non-Goals                                          │
│    ⑦ 每条用户故事（n 条 = n 次独立确认）                      │
│    ⑧ 6. 非功能性要求                                        │
│    ⑨ 8. 待确认问题                                          │
│  格式："接下来确认【字段名】，我推荐的值是 XX"                 │
│  确认/修改 → 立即写入 → 继续下一项                            │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  第七步：完成确认                                             │
│  ─────────────────                                          │
│  输出：已操作文件路径 + 变更摘要 + 留空项清单（如有）           │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start

### 1. Install the skill

```bash
npx skills add https://github.com/nileader/superbot --skill superbot
```

### 2. Configure your project (optional)

Create `.superbot/superbot.json` in your project root:

```json
{
  "spec_dir": "./spec"
}
```

> If the config file doesn't exist, the AI will guide you through initialization on first use.

### 3. Start using

- Manual: type `/superbot` in the conversation
- Or configure auto-trigger (see below)

## Configuration

| Field | Description | Default |
|-------|-------------|--------|
| `spec_dir` | Directory to store SPEC files | `./spec` |

Run `/superbot config` at any time for interactive configuration updates.

## SPEC File System

SuperBot maintains the following 6 files under `spec_dir`:

| File | Purpose |
|------|---------|
| `01-product-definition.md` | Product positioning, overall goals, core user groups |
| `02-user-story.md` | User stories, roles, usage scenarios |
| `03-feature-list.md` | Feature module breakdown, priorities, dependencies |
| `04-ui-design-spec.md` | Interaction flows, page structure, UI specs |
| `05-tech-spec.md` | Tech stack, architecture style, non-functional requirements |
| `06-others.md` | Other information helpful for product understanding |

> Chinese filename variants: `01-产品定义.md`, `02-用户故事.md`, `03-功能清单.md`, `04-UI设计规范.md`, `05-技术架构规范.md`, `06-其他.md`

## Auto-Trigger Configuration

### Claude Code

Add a `SessionStart` hook to `~/.claude/settings.json`. Using `cat` pipes the full SKILL.md content directly into Claude's context:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "cat ~/.claude/skills/superbot/SKILL.md",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

> Note: Copy `skills/superbot/SKILL.md` to `~/.claude/skills/superbot/` first.
> `matcher: "startup"` ensures the hook only fires on fresh sessions, not on resume or compact.

### OpenClaw / OpenCode

Set a session init hook in each platform's config file, pointing to this skill's `SKILL.md` path.

### Qoder

Install the skill under `.qoder/skills/` and it will auto-load via the session startup configuration.

## Companion Skill

SuperBot works standalone. If `using-superpowers` skill is also installed, they collaborate automatically for an enhanced experience.

## License

MIT
