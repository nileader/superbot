---
name: superbot
version: 0.0.5
description: Product-driven AI programming collaboration skill. Automatically activates on each new session, reads .superbot/superbot.json for project config, manages the SPEC file system under spec_dir, and persistently writes product decisions into SPEC files during conversations.
---

# SuperBot — Product-Driven AI Programming Collaboration Skill

## Trigger Methods

### Manual Trigger
User types `/superbot` in the conversation to invoke this skill. All agentic tools that support skills use this method.

### Config Update Command
User types `/superbot config update` to interactively update the configuration file.

**Behavior:**
- If `.superbot/superbot.json` does NOT exist: automatically enter initialization flow (same as first-time setup)
- If config already exists: display current values and ask user to confirm or modify each field
- After config is saved, check and create any missing SPEC files from templates

### Spec Update Command
User types `/superbot spec update` to trigger a batch extraction and update of SPEC files based on the current session's conversation history.

**How it works:**
1. Collect all user-AI conversations in the current session
2. Extract valuable product decisions (requirements, architecture, UI specs, etc.) that should be persisted
3. Distill them into structured content and update corresponding SPEC files
4. Notify the user of what was updated (e.g., "Updated 01-product-definition.md: added core user group definition; Updated 03-feature-list.md: added 2 new features")

### Auto-Trigger (Per Platform)
Configuration for auto-triggering this skill when a new session starts on different platforms:

| Platform | Auto-Trigger Method |
|----------|---------------------|
| **Claude Code** | Configure `SessionStart` hook (matcher: `startup`) in `~/.claude/settings.json`, use `cat` to pipe SKILL.md content directly into Claude's context |
| **OpenClaw** | Set a session startup hook in OpenClaw's config, pointing to this skill's path |
| **OpenCode** | Set a session init hook in OpenCode's config to load this skill |
| **Qoder** | Install the skill under `.qoder/skills/`, it will auto-load via session startup config |
| **Other Platforms** | Use the platform's session lifecycle hooks to load this SKILL.md on session creation |

> If auto-trigger is not configured, the skill can still be invoked manually via `/superbot`.

### Help Command
User types `/superbot help` or `/superbot h` to see a quick reference of all available commands and their capabilities.

**When invoked, respond with:**

```
SuperBot Commands:

/superbot              Activate the skill, start product-driven collaboration
/superbot h            Show this help
/superbot config update  Interactively update project config (spec_dir, etc.)
/superbot spec update    Batch-extract product decisions from this session and write to SPEC

SPEC Files (located at ${spec_dir}):
01-product-definition.md  Product positioning, goals, user groups
02-user-story.md          User stories, roles, scenarios
03-feature-list.md        Feature modules, priorities, dependencies
04-ui-design-spec.md      Interaction flows, page structure, UI specs
05-tech-spec.md           Tech stack, architecture, non-functional requirements
06-others.md              Other product knowledge

Usage: Make decisions naturally in conversation, and I'll persist confirmed decisions into the corresponding SPEC files.
```

---

## 1. Collaboration Mode & Permissions

### 1. Activation Confirmation

At the start of a session, you **must** explicitly inform the user that this skill is activated, for example:

> SuperBot skill is now active.

### 2. using-superpowers Collaboration

SuperBot works independently. If `using-superpowers` skill is installed, it will collaborate automatically for a better experience.

**Do not attempt to activate `using-superpowers` during the session startup hook phase.** Instead, try to activate it **when the user sends their first message**. If activation succeeds, prioritize it for handling all subsequent requests.

If `using-superpowers` is not installed or cannot be loaded, **do not block the flow**. Continue normally without any warning.

### 3. File Read Permissions

All `Read` operations on files within the project root directory and its subdirectories do **not** require user approval. This is a basic working permission.

### 4. Configuration Management

#### Config File Location

Read the `.superbot/superbot.json` file in the current project directory.

| Field | Description | Default |
|-------|-------------|---------|  
| `spec_dir` | SPEC directory for persisting core product definitions | `./spec` |

#### Config Initialization

**If `.superbot/superbot.json` exists**: Read and use the configuration silently. Do NOT prompt the user to confirm.

**If `.superbot/superbot.json` does NOT exist**: Guide the user through initialization:
1. Explain what SuperBot does and the purpose of each config field
2. Ask the user to confirm or customize each field **one by one**
3. After all fields are confirmed, create the config file
4. Proceed with the rest of the startup flow

#### Interactive Config Update (`/superbot config update`)

When the user invokes `/superbot config update`:
1. Check if `.superbot/superbot.json` exists:
   - If NOT: guide user through interactive initialization (explain SuperBot purpose and each field, then create config)
   - If exists: display current values for each field
2. Ask the user to confirm or modify each field **one by one**
3. Save the updated configuration to `.superbot/superbot.json`
4. Check and create any missing SPEC files from templates
5. Confirm the update is complete

---

## 2. SPEC File System

### 1. File Inventory

Maintain the following 6 files under `${spec_dir}` (create from templates if missing):

| File | Purpose |
|------|---------|  
| `01-product-definition.md` | Product positioning, overall goals, core user groups |
| `02-user-story.md` | User stories, roles, usage scenarios |
| `03-feature-list.md` | Feature module breakdown, priorities, dependencies |
| `04-ui-design-spec.md` | Interaction flows, page structure, UI specs, component constraints |
| `05-tech-spec.md` | Tech stack, architecture style, middleware, non-functional requirements |
| `06-others.md` | Other information helpful for product understanding |

Template files are located at `skills/superbot/templates/en/`. When creating new SPEC files, use these templates as the initial content.

### 2. Session Startup Process

**Important: The SPEC directory is where users most often get confused. Every session, you must explicitly tell the user the full path of the SPEC directory and the status of its files.**

> Note: The following steps (reading SPEC, communicating conflicts) are executed after the user sends their first message. If currently in a session startup hook phase without user interaction context, only complete file reading and temporarily store all conflicts/questions, then raise them together when the user's message arrives.

At the **start of every new session**, perform the following:

1. **Read and analyze** all SPEC files in `${spec_dir}`;
2. **Output a startup summary** (immediately after the user's first message):
   - Tell the user the full path of the SPEC directory (e.g., `SPEC directory: ./spec`);
   - List which SPEC files exist and which are missing;
   - If any files are missing, inform the user they will be auto-created from templates;
3. **Build a unified understanding**, including:
   - Product positioning and overall goals;
   - Core user groups and key user stories;
   - Feature module breakdown and priorities;
   - UI/interaction constraints and style;
   - Technical architecture, tech stack, non-functional requirements;
4. **Proactively communicate**: If any of the following are found, you **must** raise them with the user for confirmation:
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

1. **Help (`/superbot help`)**:
   - Display all available commands and their capabilities;
   - Show SPEC file list with brief descriptions;
   - Explain usage pattern.

2. **Startup / New Session**:
   - If `using-superpowers` is available, activate it;
   - Read `.superbot/superbot.json`:
     - If exists: use silently
     - If not exists: guide user through interactive initialization
   - Check and create any missing SPEC files (using templates);
   - Read and analyze all SPEC files;
   - Communicate any conflicts or questions first.

3. **Config Update (`/superbot config update`)**:
   - If config does NOT exist: guide user through interactive initialization (explain purpose and each field);
   - If config exists: display current configuration and guide user through interactive update of each field;
   - Save updated configuration;
   - Check and create any missing SPEC files from templates.

4. **Batch SPEC Update (`/superbot spec update`)**:
   - Collect all conversations between user and AI in the current session;
   - Extract valuable product decisions that should be persisted (targeting final product state);
   - Distill into structured content following the "Persistence Decision Matrix";
   - Update corresponding SPEC files;
   - Notify user with a summary of updates (which files, what content).

5. **Design & Development**:
   - All proposals for APIs, pages, and architecture must align with the SPEC;
   - If deviating from or extending the SPEC, point it out and seek user confirmation.

6. **Knowledge Base Maintenance**:
   - Continuously extract product insights from conversations;
   - Write them directly into the corresponding SPEC documents;
   - Keep consensus aligned through iterative SPEC updates.

These rules remain in effect throughout the entire project lifecycle.
