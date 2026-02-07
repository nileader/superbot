# SuperBot

产品驱动的 AI 编程协作技能。

## 安装

```bash
npx skills add <this-repo-url> --skill superbot
```

## 功能

- 新 session 自动激活，读取 `.superbot/superbot.json` 配置
- 管理 `spec_dir` 下的 6 个 SPEC 文件体系
- 对话中实时将产品决策沉淀到 SPEC 文件
- 支持手动 `/superbot` 调用

## 配置

在项目根目录创建 `.superbot/superbot.json`：

```json
{
  "workspacedir": ".",
  "spec_dir": "./spec"
}
```

## 自动触发

### Claude Code

在 `.claude/settings.json` 中配置 `on_session_start` 钩子，通过 `Skill` 工具激活 `superbot`：

```json
{
  "hooks": {
    "on_session_start": [{
      "hooks": [{
        "type": "skill",
        "skill": "superbot"
      }]
    }]
  }
}
```

> 注意：`using-superpowers` 技能会在用户发送第一条消息时尝试激活，不会在 hook 阶段调用 Skill tool，避免启动报错。

### OpenClaw / OpenCode

参考各平台的 session 启动钩子文档，加载本技能的 `SKILL.md` 文件。
