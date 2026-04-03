---
name: project-context-builder
description: Automatically maintain a PROJECT_CONTEXT.md file that maps plain-English nicknames to n8n workflow IDs, Monday board IDs, and Slack channel IDs. Triggers when the user adds, creates, or references an n8n workflow, Monday board, or Slack channel — or asks to "update project context", "add this to the brain", "register this flow", "remember this board", "add this channel", "set up project context", "initialize project brain", or "add to project context". Enriches each resource via MCP tools and keeps a Cursor rule and CLAUDE.md snippet pointing agents to the central document.
user-invocable: true
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion, mcp__plugin_n8n-mcp_n8n__n8n_get_workflow, mcp__plugin_n8n-mcp_n8n__n8n_list_workflows, mcp__plugin_slack_slack__slack_read_channel, mcp__plugin_slack_slack__slack_search_channels, mcp__plugin_kremer-mcp_kremer-mcp__knowledge-base-query]
---

# Project Context Auto-Builder

Maintains a central `PROJECT_CONTEXT.md` file that maps human-friendly nicknames to resource IDs (n8n workflows, Monday boards, Slack channels). Any agent in the project — Cursor or Claude Code — can look up the real ID from a nickname.

## When to Activate

- User adds, creates, or references an **n8n workflow** (via MCP tools or by mentioning a workflow ID).
- User adds, creates, or references a **Monday.com board** (via Monday MCP tools or by pasting a board URL like `monday.monday.com/boards/<ID>`).
- User adds or references a **Slack channel** (via Slack MCP tools or by providing a channel ID like `C0xxxxxxxx`).
- User says: "add this to the project context", "remember this board", "register this flow", "add this channel to the brain", "update project context", "set up project context", "initialize the project brain".

## Workflow

### Step 1: Check for PROJECT_CONTEXT.md

Read `PROJECT_CONTEXT.md` in the project root.

- If it **does not exist**, create it from [PROJECT_CONTEXT.md.template](references/PROJECT_CONTEXT.md.template).
  - Ask the user for a **project name** to use as the heading.
- If it **already exists**, read its current contents so you know what's already registered.

### Step 2: Identify the resource type and ID

Parse the user's input to determine:

| Input Pattern | Resource Type | How to Extract ID |
|---|---|---|
| n8n workflow ID (alphanumeric, ~16 chars) or MCP workflow call | n8n Workflow | Use the ID directly |
| `monday.monday.com/boards/<ID>` or board ID (numeric) | Monday Board | Extract numeric ID from URL or input |
| Slack channel ID (`C0xxxxxxxxx`) or `#channel-name` | Slack Channel | Use the ID directly, or search by name |

### Step 3: Enrich via MCP

Pull details about the resource using MCP tools. If MCP tools are unavailable, skip to Step 4 and ask the user to provide details manually.

| Resource Type | MCP Enrichment |
|---|---|
| **n8n Workflow** | Call `n8n_get_workflow` with the workflow ID. Extract the workflow name and inspect nodes/connections to write a 1-2 sentence description of what it does. |
| **Monday Board** | Call `knowledge-base-query` or the Monday MCP `get_board_info` with the board ID. Extract the board name, description, workspace, and item count. |
| **Slack Channel** | Call `slack_read_channel` with the channel ID and limit 1. Extract the channel name from the response header. |

### Step 4: Ask for a nickname

Ask the user:

> "What nickname should I use for this resource? I'd suggest '[auto-generated suggestion based on name]'."

If the user doesn't provide one, use the auto-generated suggestion.

### Step 5: Check for duplicates

Before adding, check if this resource ID already exists in `PROJECT_CONTEXT.md`.

- If it **already exists**, ask the user if they want to **update** the existing entry's nickname or description.
- If it **does not exist**, proceed to add it.

### Step 6: Update PROJECT_CONTEXT.md

Add the new resource to the correct section using the table format:

**n8n Workflows:**
```
| **[Nickname]** | [Workflow Name] | `[ID]` | [1-2 sentence description] |
```

**Monday Boards:**
```
| **[Nickname]** | [Board Name] | `[ID]` | [Link](https://monday.monday.com/boards/[ID]) | [1-2 sentence description] |
```

**Slack Channels:**
```
| **[Nickname]** | `#[channel-name]` | `[ID]` | [1-2 sentence description] |
```

Rules:
- **Never remove existing entries.** Only add new ones or update existing descriptions/nicknames.
- Keep descriptions to 1-2 sentences. Focus on *what it does* and *why it matters*.
- Preserve the existing table formatting.

### Step 7: Ensure agent rules exist

Check for and create the appropriate rule files so that all agents in the project know about `PROJECT_CONTEXT.md`:

**For Cursor:** Check if `.cursor/rules/project-context.mdc` exists. If not, create it from [project-context.mdc.template](references/project-context.mdc.template).

**For Claude Code:** Check if `CLAUDE.md` exists in the project root.
- If it exists, check whether it already references `PROJECT_CONTEXT.md`. If not, append the snippet from [CLAUDE_SNIPPET.md.template](references/CLAUDE_SNIPPET.md.template).
- If it doesn't exist, create a new `CLAUDE.md` with the snippet.

### Step 8: Confirm

Tell the user what was added:

> "Done — added '[Nickname]' ([Resource Type]) to your project context. ID: `[ID]`."

## Manual Mode (No MCP)

If MCP tools are not available, the skill still works. Ask the user to provide:

1. **Resource type** (n8n workflow, Monday board, or Slack channel)
2. **ID**
3. **Name** (the official name of the resource)
4. **Nickname** (what they want to call it)
5. **Description** (1-2 sentences about what it does)

Then proceed from Step 5.

## Batch Mode

If the user provides multiple resources at once (e.g., "add these 3 workflows"), process each one sequentially through the full workflow. Ask for nicknames in batch:

> "I found 3 workflows. What nicknames should I use?
> 1. [Name 1] — I'd suggest '[Suggestion 1]'
> 2. [Name 2] — I'd suggest '[Suggestion 2]'
> 3. [Name 3] — I'd suggest '[Suggestion 3]'"

## Initialization Mode

When the user says "set up project context" or "initialize project brain":

1. Create `PROJECT_CONTEXT.md` from template.
2. Create `.cursor/rules/project-context.mdc` from template.
3. Add `PROJECT_CONTEXT.md` reference to `CLAUDE.md`.
4. Ask the user if they have any resources to register now.

## Reference Files

- [PROJECT_CONTEXT.md.template](references/PROJECT_CONTEXT.md.template) — template for the central context document
- [project-context.mdc.template](references/project-context.mdc.template) — Cursor rule template
- [CLAUDE_SNIPPET.md.template](references/CLAUDE_SNIPPET.md.template) — Claude Code CLAUDE.md snippet
