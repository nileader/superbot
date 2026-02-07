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

在 `.claude/settings.json` 中配置 `SessionStart` 钩子：

```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "echo '{\"systemMessage\":\"SuperBot 技能已激活\",\"additionalContext\":\"SuperBot 技能已激活。请按照 SKILL.md 中的指令执行。\"}'",
        "timeout": 5
      }]
    }]
  }
}
```

### OpenClaw / OpenCode

参考各平台的 session 启动钩子文档，加载本技能的 `SKILL.md` 文件。
