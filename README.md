# Project Context Auto-Builder

A Claude Code skill that automatically maintains a `PROJECT_CONTEXT.md` file — a central registry mapping plain-English nicknames to resource IDs for n8n workflows, Monday.com boards, and Slack channels.

Works with both **Claude Code** and **Cursor**.

## The Problem

When working across n8n, Monday.com, and Slack, you constantly juggle IDs like `aBcDeFgH12345678`, `1234567890`, or `C012ABC3DEF`. You know them as "Main Flow", "Results Board", and "Escalations" — but your AI agents don't.

## The Solution

This skill creates and maintains a `PROJECT_CONTEXT.md` file that serves as a shared brain for your project. Every agent opened in the project — whether in Cursor or Claude Code — immediately knows your resources by their nicknames.

```
User: "Run the Main Flow"
Agent: (looks up PROJECT_CONTEXT.md) → calls n8n workflow aBcDeFgH12345678
```

## How It Works

1. **Detects** when you add, create, or reference a resource (n8n workflow, Monday board, Slack channel)
2. **Enriches** it by pulling details from MCP tools (name, description, structure)
3. **Asks** you for a nickname
4. **Updates** `PROJECT_CONTEXT.md` with the nickname, ID, and a short description
5. **Creates** lightweight rules for Cursor (`.cursor/rules/project-context.mdc`) and Claude Code (`CLAUDE.md`) that point agents to the central document

## Installation

### Claude Code

Copy or symlink the `project-context-builder/` directory into your Claude Code skills folder:

```bash
# Option A: Symlink (recommended — stays in sync with updates)
ln -s /path/to/project-context-builder ~/.claude/skills/project-context-builder

# Option B: Copy
cp -r /path/to/project-context-builder ~/.claude/skills/project-context-builder
```

### Cursor

The skill automatically creates `.cursor/rules/project-context.mdc` in your project when it runs. No separate Cursor installation needed — just use Claude Code with this skill installed and it handles both tools.

## Usage

### Initialize a new project

```
> set up project context
```

or

```
> /project-context-builder
```

This creates `PROJECT_CONTEXT.md`, the Cursor rule, and the Claude Code rule in your project.

### Add a resource

**By URL or ID:**
```
> Add this Monday board: https://acme.monday.com/boards/1234567890
> Register workflow aBcDeFgH12345678
> Add Slack channel C012ABC3DEF
```

**By reference:**
```
> Remember this board as "Campaign Tracker"
> Add this channel to the project context
```

**Batch:**
```
> Add these 3 workflows: aBcDeFgH12345678, xYzWvUtS98765432, mNoPqRsT11223344
```

### Trigger phrases

The skill activates on phrases like:
- "add this to the project context"
- "remember this board"
- "register this flow"
- "add this channel to the brain"
- "update project context"
- "set up project context"
- "initialize project brain"

### Manual mode (no MCP)

If MCP tools aren't available, the skill asks you to provide the name, ID, and description manually. It still maintains the same `PROJECT_CONTEXT.md` format.

## What Gets Created

### `PROJECT_CONTEXT.md`

The central document. Contains tables for each resource type:

```markdown
## n8n Workflows

| Nickname | Workflow Name | ID | Description |
| :--- | :--- | :--- | :--- |
| **Main Flow** | Daily Report Pipeline | `aBcDeFgH12345678` | Fetches data from API, transforms records, and sends summary notifications. |

## Monday Boards

| Nickname | Board Name | ID | Link | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Results Board** | Task Tracker | `1234567890` | [Link](https://acme.monday.com/boards/1234567890) | Tracks task status and completion metrics. |

## Slack Channels

| Nickname | Channel Name | ID | Description |
| :--- | :--- | :--- | :--- |
| **Playground** | `#dev-sandbox` | `C034XYZ5GHI` | Sandbox channel for testing integrations. |
```

### `.cursor/rules/project-context.mdc`

A lightweight Cursor rule that tells Cursor agents to look up nicknames in `PROJECT_CONTEXT.md`.

### `CLAUDE.md` snippet

A section appended to your project's `CLAUDE.md` that tells Claude Code agents to resolve nicknames via `PROJECT_CONTEXT.md`.

## Design Principles

- **Never removes entries.** Only adds or updates.
- **Always enriches via MCP.** Pulls real names and writes useful descriptions — doesn't just log bare IDs.
- **Short descriptions.** 1-2 sentences max. What it does and why it matters.
- **Cross-tool compatible.** Works in both Cursor and Claude Code from a single source of truth.
- **Graceful degradation.** Works without MCP tools via manual input.

## File Structure

```
project-context-builder/
├── SKILL.md                                  # Skill definition
├── README.md                                 # This file
├── LICENSE                                   # MIT License
└── references/
    ├── PROJECT_CONTEXT.md.template            # Template for the context doc
    ├── project-context.mdc.template           # Template for Cursor rule
    └── CLAUDE_SNIPPET.md.template             # Template for CLAUDE.md snippet
```

## Requirements

- Claude Code CLI or VS Code extension
- Optional: n8n MCP server, Monday MCP server, Slack MCP server (for auto-enrichment)

## License

MIT
