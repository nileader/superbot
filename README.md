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
- Interactive config update via `/superbot config`
- Batch SPEC update via `/superbot update spec` — extracts all valuable decisions from current session and updates SPEC files

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
