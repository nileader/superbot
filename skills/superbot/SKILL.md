---
name: superbot
description: Product-driven AI programming collaboration skill. Automatically activates on each new session, reads .superbot/superbot.json for project config, manages the SPEC file system under spec_dir, and persistently writes product decisions into SPEC files during conversations.
---

# SuperBot — Product-Driven AI Programming Collaboration Skill

## Trigger Methods

### Manual Trigger
User types `/superbot` in the conversation to invoke this skill. All agentic tools that support skills use this method.

### Auto-Trigger (Per Platform)
Configuration for auto-triggering this skill when a new session starts on different platforms:

| Platform | Auto-Trigger Method |
|----------|---------------------|
| **Claude Code** | Configure hooks in `.claude/settings.json`, use the `on_session_start` hook to call the `Skill` tool to activate `superbot` |
| **OpenClaw** | Set a session startup hook in OpenClaw's config, pointing to this skill's path |
| **OpenCode** | Set a session init hook in OpenCode's config to load this skill |
| **Other Platforms** | Use the platform's session lifecycle hooks to load this SKILL.md on session creation |

> If auto-trigger is not configured, the skill can still be invoked manually via `/superbot`.

---

## 1. Collaboration Mode & Permissions

### 1. Activation Confirmation

At the start of a session, you **must** explicitly inform the user that this skill is activated, for example:

> SuperBot skill is now active.

### 2. using-superpowers Collaboration

`using-superpowers` is the core collaboration skill for this session. **Do not attempt to activate it during the session startup hook phase.** Instead, try to activate it **when the user sends their first message**. If activation succeeds, prioritize it for handling all subsequent requests.

If `using-superpowers` is not properly installed or cannot be loaded, **do not block the flow**. Briefly remind the user in your response: "It is recommended to install the using-superpowers skill for the full experience," then continue normally.

### 3. File Read Permissions

All `Read` operations on files within the project root directory and its subdirectories do **not** require user approval. This is a basic working permission.

### 4. Reading the Config File

Read the `.superbot/superbot.json` file in the current project directory to obtain the following configuration:

| Field | Description |
|-------|-------------|
| `workspacedir` | Working root directory |
| `spec_dir` | SPEC directory, used to persist core product definitions |

If `.superbot/superbot.json` does not exist or any field is missing from the config, fill in the default values first without blocking the flow. The default configuration is:

```json
{
  "workspacedir": ".",
  "spec_dir": "./spec"
}
```

After the user sends their first message, print the current configuration and ask the user to confirm or modify each field one by one.

---

## 2. SPEC File System

### 1. File Inventory

Maintain the following 6 files under `${spec_dir}` (create empty files if missing):

| File | Purpose |
|------|---------|
| `01-产品定义.md` | Product positioning, overall goals, core user groups |
| `02-用户故事.md` | User stories, roles, usage scenarios |
| `03-功能清单.md` | Feature module breakdown, priorities, dependencies |
| `04-UI设计规范.md` | Interaction flows, page structure, UI specs, component constraints |
| `05-技术架构规范.md` | Tech stack, architecture style, middleware, non-functional requirements |
| `06-其他.md` | Other information helpful for product understanding |

### 2. Session Startup Process

> Note: The following steps (reading SPEC, communicating conflicts) are executed after the user sends their first message. If currently in a session startup hook phase without user interaction context, only complete file reading and temporarily store all conflicts/questions, then raise them together when the user's message arrives.

At the **start of every new session**, perform the following:

1. **Read and analyze** all SPEC files in `${spec_dir}`;
2. **Build a unified understanding**, including:
   - Product positioning and overall goals;
   - Core user groups and key user stories;
   - Feature module breakdown and priorities;
   - UI/interaction constraints and style;
   - Technical architecture, tech stack, non-functional requirements;
3. **Proactively communicate**: If any of the following are found, you **must** raise them with the user for confirmation:
   - Content is unclear or has multiple possible interpretations;
   - Internal contradictions within a single document;
   - Conflicting definitions across different documents for the same concept/feature/flow;
   - SPEC content is clearly inconsistent with the current code implementation;

   When communicating, specify the exact location and wording of the conflict, provide 1-2 possible interpretations, and ask the user to choose or clarify.

**Only proceed to concrete design and coding suggestions after forming a clear, consistent understanding of the SPEC.**

---

## 3. SPEC Real-Time Persistence Specification

The SPEC directory is a product knowledge base that requires **continuous iterative maintenance**. Goal:
> Enable any new team member or Agent to quickly align on consensus and reproduce the final product state by reading only this set of SPEC files.

### Real-Time Persistence Principles

**In every round of conversation, you must proactively identify, distill, and write key decisions into SPEC files.**

Process:

1. **Listen**: Identify new decisions, confirmations, changes, or constraints in user responses;
2. **Judge**: Use the "Persistence Decision Matrix" below to determine if it's worth writing;
3. **Distill**: Abstract conversational discussion into structured conclusions;
4. **Write**: Directly Edit/Write the corresponding SPEC file;
5. **Notify**: Briefly state at the end of your response: "Updated XX.md: added YY entry."

#### Persistence Decision Matrix

| Must Write | Do Not Write |
|------------|--------------|
| Explicitly confirmed product decisions ("let's do it", "yes", "correct") | Tentative ideas during discussion ("what if we try...", "how about...") |
| New/modified/deleted feature requirements | Trial-and-error on implementation details (unless affecting architecture) |
| Confirmed changes to user stories, roles, or scenarios | Pure debugging, troubleshooting, or code fix processes |
| Confirmed tech stack or architecture decisions | Temporary workarounds explicitly marked as "do this for now, fix later" |
| Confirmed UI/interaction specifications | Proposals explicitly rejected by the user |
| Non-functional constraints raised by the user (performance, security, compliance) | Casual chat or technical discussions unrelated to the product |

**Core criterion**: Would this information affect how a newcomer understands "what the product should be"? If yes, write it.

#### Write Timing

- **Immediate write**: When the user gives a clear decision/confirmation, complete the SPEC update directly
- **Batch write**: When a single conversation round contains multiple related decisions, merge into one update
- **End-of-session review**: When the conversation is winding down, proactively check for any missed decisions to backfill

### Content Requirements for Updates

1. Directly Edit/Write the corresponding SPEC file;
2. Provide structured update content (new section/subsection titles, feature entries, etc.);
3. Write from a final-state perspective, avoiding:
   - Heavy conversational language;
   - Expressions tightly coupled to time or conversation context;
   - Vague or ambiguous wording.

### Handling SPEC vs. Implementation Deviations

When you discover any of the following:
- Actual code implementation differs from the SPEC;
- New requirements conflict with the current SPEC;
- The SPEC is outdated or no longer reasonable;

You should:
1. Clearly mark the point of deviation;
2. Ask the user: should the new conversation conclusion take precedence, or should the SPEC be the source of truth?
3. After the user decides:
   - If conversation wins: update the SPEC to align with the new product direction;
   - If SPEC wins: adjust the implementation suggestion accordingly.

---

## 4. Comprehensive Workflow

1. **Startup / New Session**:
   - Activate the `using-superpowers` skill;
   - Read `.superbot/superbot.json`, confirm configuration;
   - Check and create any missing SPEC files;
   - Read and analyze all SPEC files;
   - Communicate any conflicts or questions first.

2. **Design & Development**:
   - All proposals for APIs, pages, and architecture must align with the SPEC;
   - If deviating from or extending the SPEC, point it out and seek user confirmation.

3. **Knowledge Base Maintenance**:
   - Continuously extract product insights from conversations;
   - Write them directly into the corresponding SPEC documents;
   - Keep consensus aligned through iterative SPEC updates.

These rules remain in effect throughout the entire project lifecycle.
