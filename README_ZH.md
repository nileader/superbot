# SuperBot

产品驱动的 AI 编程协作技能 — 让 AI 自动管理产品 SPEC 文件，实时沉淀产品决策。

[English](./README.md)

## 它能帮你做什么？

在与 AI 协作开发的过程中，产品决策往往散落在对话历史里，难以追溯。SuperBot 解决这个问题：

- **自动识别产品决策**：AI 在对话中识别你确认的需求、架构、UI 规范等关键决策
- **实时写入 SPEC 文件**：决策自动沉淀到结构化的产品文档中
- **统一产品认知**：任何人（或 AI）阅读 SPEC 就能快速理解产品应该是什么样

## 安装

- 如果安装superbot整个插件，则依次执行：
```bash
/plugin marketplace add nileader/superbot

/plugin install superbot

/reload-plugins
```

- 如果仅安装某些skill，则选择性执行下述命令
```bash
npx skills add https://github.com/nileader/superbot --skill superbot

npx skills add https://github.com/nileader/superbot --skill prd
```


## 功能特性

- 新 session 自动激活，读取 `.superbot/superbot.json` 配置
- 管理 `spec_dir` 下的 6 个 SPEC 文件（产品定义、用户故事、功能清单、UI规范、技术架构、其他）
- 对话中实时将产品决策沉淀到 SPEC 文件
- 支持手动 `/superbot` 调用
- 支持 `/superbot init` 强制初始化 — 适用于首次使用或重新初始化
- 支持 `/superbot config` 交互式修改配置
- 支持 `/superbot update spec` 批量更新 SPEC — 从当前 session 提取所有有价值的决策并更新到 SPEC 文件

## 快速开始

### 1. 安装技能

```bash
npx skills add https://github.com/nileader/superbot --skill superbot
```

### 2. 配置项目（可选）

在项目根目录创建 `.superbot/superbot.json`：

```json
{
  "spec_dir": "./spec"
}
```

> 如果不创建配置文件，首次使用时 AI 会引导你完成初始化。

### 3. 开始使用

- 手动调用：在对话中输入 `/superbot`
- 或配置自动触发（见下方）

## 配置说明

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `spec_dir` | SPEC 文件存放目录 | `./spec` |

运行时可通过 `/superbot config` 交互式修改配置。

## SPEC 文件体系

SuperBot 在 `spec_dir` 目录下维护以下 6 个文件：

| 文件 | 用途 |
|------|------|
| `01-产品定义.md` | 产品定位、整体目标、核心用户群 |
| `02-用户故事.md` | 用户故事、角色、使用场景 |
| `03-功能清单.md` | 功能模块划分、优先级、依赖关系 |
| `04-UI设计规范.md` | 交互流程、页面结构、UI 规范 |
| `05-技术架构规范.md` | 技术选型、架构风格、非功能性要求 |
| `06-其他.md` | 其他对产品认知有帮助的信息 |

> 英文版文件名：`01-product-definition.md`, `02-user-story.md`, `03-feature-list.md`, `04-ui-design-spec.md`, `05-tech-spec.md`, `06-others.md`

## 自动触发配置

### Claude Code

在 `~/.claude/settings.json` 中配置 `SessionStart` 钩子，通过 `cat` 直接将 SKILL.md 内容注入 Claude 上下文：

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

> 注意：需要先将 `skills/superbot/SKILL.md` 复制到 `~/.claude/skills/superbot/` 目录下。
> `matcher: "startup"` 表示只在全新会话启动时触发，不在 resume / compact 时重复注入。

### OpenClaw / OpenCode

在各平台配置文件中设置 session init hook，指向本技能的 `SKILL.md` 路径。

### Qoder

在 `.qoder/skills/` 目录下安装技能后，通过 session 启动配置自动加载。

## 协作技能

SuperBot 可独立使用。如果安装了 `using-superpowers` 技能，会自动协同工作以获得更好的体验。

## License

MIT
