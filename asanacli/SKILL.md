---
name: asanacli
description: Asana CLI for managing tasks, viewing task details, posting comments, marking tasks complete, and downloading attachments.
---

# Asana CLI

Command-line interface for Asana project management. Supports listing tasks, viewing task details with custom fields, posting comments, marking tasks complete, setting due dates, and downloading attachments.

## Installation

`
go install github.com/ttk/asana@bbb68ec3aaa319031f6dfe8ced9a2395ec226976
`
## Setup

### Configure asana

First check if already configured:

```bash
asana workspaces
```

If no results or returns a 404, then guide the user through the configuration setup and wizard.

1. Ask if they have an Asana account, if not as them to create one or ask to be invited to a workspace.
2. Tell the user to create a personal access token from the accounts "My Apps" page (https://app.asana.com/0/my-apps) 
3. Tell the user to run `asana config` in a separate terminal to enter the token, and select the workspace.
4. Once the user confirmed the config is complete, double check by running `asana workspaces`.

Your settings will be saved in `~/.asana.yml`.

## Usage

Run `asana --help` to see all available commands.

### Common Operations

**List all tasks:**
```bash
asana tasks
asana tasks --refresh       # update cache
asana tasks --no-cache      # bypass cache
asana tasks --json          # Render as JSON
```

**View task details:**
```bash
asana task <task-id>
asana task --with-comments <task-id>        # includes comments
asana task --json <task-id>    # JSON output for scripting
```

Task ids are shown in the list output. The tool automatically displays custom fields like Urgency, Dev Severity, and Environment.

**Mark task complete:**
```bash
asana done <task-id>
```

**Set due date:**
```bash
asana due <task-id> <date>
# Examples:
asana due 5 2026-01-15
asana due 5 tomorrow
asana due 5 "next friday"
```

**Set task section:**
```bash
asana set-sec -p eng <task-id> <section-name>   # set section for engineering tasks
# Examples:
asana set-sec -p eng 5 done  # mark task as done in engineering section
asana set-sec -p eng 5 "in progress"  # set task to "in progress" in engineering section
```
Note 1: `-p` flag is required if the task is part of two projects.  Always use `-p eng` when using this skill.
Note 2: Section names are case-insensitive and can be a keyword.  Valid values in the "eng" project are: "blocked", "up next", "in progress", "submitted pr", "deployed", and "done."

**Post comment:**
```bash
asana comment -c "Your comment here" <task-id> 
```

**Open task in browser:**
```bash
asana browse <task-id>
```

**Download attachments:**
```bash
asana download <attachment-gid> # download by Global ID
asana download <task-id> <attachment-index> # download specific attachment by index
```

**List workspaces:**
```bash
asana workspaces
```

### JSON Output and Scripting

Use `--json` flag with `asana task` for machine-readable output:

```bash
# Extract specific fields with jq
asana task --json 5  | jq '.task.name'
asana task --json 5 --json | jq '.task.custom_fields[] | select(.name == "Type") | .display_value'
```

Example JSON Structure

```json
{
  "task": {
    "gid": "1212293030279497",
    "name": "[Customer-Reported] KI Vorschläge",
    "custom_fields": [
      {
        "gid": "1212278811468834",
        "name": "Urgency",
        "display_value": "Very High",
        "type": "enum"
      }
    ],
    "notes": "...",
    "assignee": {...}
  },
  "stories": [...],
  "attachments": [...]
}

```


### Global Options

- `--help, -h`: Show help

## Data Storage

Configuration and cache files are stored in:
- `~/.asana.yml` - Authentication credentials and workspace settings
- Task cache (location depends on implementation)

## Notes

- The CLI uses Asana's modern API with string-based Global IDs (GIDs) instead of legacy integer identifiers
- Task caching speeds up repeated operations; use `--refresh` to update or `--no-cache` to bypass
- Custom fields are automatically displayed without additional configuration
- Verbose mode (`-v`) shows complete task history including all comments and activity, but usually `--with-comments` is sufficient.
