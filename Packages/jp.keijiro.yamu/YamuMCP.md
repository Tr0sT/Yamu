# Yamu MCP - Guide

Yamu (Yet Another Minimal MCP server for Unity) is an MCP server for integrating AI agents with Unity.

## Core commands

| Command | Description |
|---------|-------------|
| `compile_and_wait` | Starts compilation and waits for the result |
| `compile_status` | Checks compilation status without starting it |
| `refresh_assets` | Refreshes the AssetDatabase (after creating/deleting files) |
| `get_console_logs` | Retrieves Unity console logs |
| `run_tests` | Runs tests (EditMode/PlayMode) |
| `editor_status` | Editor status (compilation, tests, play mode) |

## Workflow: Compile until success

### After editing an existing file:
```
1. compile_and_wait (timeout: 120)
2. editor_status — ensure compilation is finished
3. get_console_logs (type: Error) — check for console errors
4. If there are errors → fix code → repeat from step 1
```

### After creating/deleting/moving a file:
```
1. refresh_assets (force: true for deletions, false for creations)
2. editor_status — ensure refresh is finished
3. compile_and_wait (timeout: 120)
4. editor_status — ensure compilation is finished
5. get_console_logs (type: Error) — check for console errors
6. If there are errors → fix code → repeat from step 3
```

## Common errors and fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Error -32603` + "Unity HTTP server restarting" | Normal during compilation | Wait 3–5 sec, retry |
| `CS2001: Source file could not be found` | File deleted, Unity unaware | `refresh_assets(force: true)` |
| `Unity Editor HTTP server unavailable` | Unity not running | Ensure Unity is open with the project |

## Example error-fix loop

```
loop:
  result = compile_and_wait(timeout: 120)
  editor_status()
  logs = get_console_logs(type: Error)
  
  if result.success and logs.empty:
    break
  
  for error in result.errors + logs:
    fix_error(error.file, error.line, error.message)
  
  continue loop
```

## compile_and_wait parameters

- **timeout** (default: 30) — timeout in seconds. 120 recommended for reliability.

## refresh_assets parameters

- **force** (default: false) — use `ImportAssetOptions.ForceUpdate`.
  - `true` — when deleting files
  - `false` — when creating new files

## get_console_logs parameters

- **type** — filter: `Log`, `Warning`, `Error`, `Assert`, `Exception`
- **limit** (default: 100, max: 1000) — number of entries
- **clear** (default: false) — clear logs after reading
