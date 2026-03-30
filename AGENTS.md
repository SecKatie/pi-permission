# pi-permission

Layered permission control extension for [pi-coding-agent](https://github.com/mariozechner/pi-coding-agent). Implements security levels to protect users from unintended operations.

## What This Project Is

A TypeScript extension that adds permission-based command filtering to the pi coding agent. It classifies shell commands into 5 security levels:

| Level | Description | Allowed Operations |
|-------|-------------|-------------------|
| `minimal` | Read-only (default) | `ls`, `grep`, `cat`, `git status/diff/log`, `npm list`, etc. |
| `low` | File operations | Create/edit files, mkdir, cp, mv |
| `medium` | Development operations | `npm install`, `npm build`, `npm test`, `git commit/pull`, builds |
| `high` | Full operations | `git push`, deployments, curl, docker push, kubectl |
| `bypassed` | All checks disabled | Everything (dangerous for CI only) |

## Key Features

- **Command classification**: Automatically classifies shell commands by required permission level
- **Dangerous command detection**: Special handling for `rm -rf`, `sudo`, `chmod 777`, etc.
- **MCP tool permissioning**: Controls access to MCP tools (search, connect, etc.)
- **Shell trick detection**: Prevents bypass via `$(cmd)`, backticks, `eval`, etc.
- **Configurable overrides**: Users can customize classification in `~/.pi/agent/settings.json`
- **Two permission modes**: `ask` (prompt) or `block` (deny without asking)

## Major Files

```
src/
├── permission-core.ts      # Core logic: command classification, config, settings
├── permission.ts          # Extension entry point, handlers, UI prompts

tests/
├── permission.test.ts     # Command classification tests (~1400 lines)
├── permission-prompt.test.ts  # UI prompt behavior tests
```

### permission-core.ts

Pure functions for:
- `classifyCommand()` - Determines permission level for any shell command
- `parseCommand()` - Shell parsing with operator detection
- Config/cache management functions

### permission.ts

Extension hooks and handlers:
- `handleBashToolCall()` - Bash command permission checks
- `handleWriteToolCall()` - File write/edit permission checks
- `handleMcpToolCall()` - MCP tool call permission checks
- `handlePermissionCommand()` - `/permission` slash command
- `handlePermissionModeCommand()` - `/permission-mode` slash command

## Testing

Run tests with:

```bash
npm test
```

Or individually:

```bash
npx tsx tests/permission.test.ts
npx tsx tests/permission-prompt.test.ts
```

### Test Structure

- **permission.test.ts**: Tests `classifyCommand()` directly
  - Covers all 5 permission levels
  - Tests command parsing, pipelines, redirections
  - Tests shell tricks (`$()`, backticks, `eval`)
  - Tests config overrides and prefix mappings

- **permission-prompt.test.ts**: Tests UI handler functions
  - Tests prompt messages and options
  - Tests Allow/Cancel/Block behavior
  - Tests block mode vs ask mode

## Requirements

- **New features MUST be covered by tests**
- All command classification changes require test updates
- New handlers need corresponding prompt behavior tests
- Run `npm test` before committing

## Building

```bash
npm run build    # TypeScript compilation
npm test        # Run tests
```

Output goes to `dist/`.

## Configuration

Settings are stored in `~/.pi/agent/settings.json`:

```json
{
  "permissionLevel": "medium",
  "permissionMode": "ask",
  "permissionConfig": {
    "overrides": {
      "minimal": ["tmux list-*"],
      "medium": ["tmux *"]
    },
    "prefixMappings": [
      { "from": "fvm flutter", "to": "flutter" }
    ]
  }
}
```

## Architecture Notes

- Uses `shell-quote` for command parsing
- Caches compiled regex patterns for performance
- Handles tmux/screen terminal detection for notifications
- Supports both interactive and print mode (`-p`) execution
