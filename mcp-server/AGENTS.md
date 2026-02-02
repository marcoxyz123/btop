# mcp-server/ - AI Control Interface

## Overview

Python MCP (Model Context Protocol) server for AI assistant control of mbtop. Allows process tagging, filtering, and log panel control via Unix socket.

## Structure

```
mcp-server/
├── src/mbtop_mcp/__init__.py  # FastMCP server (700 lines)
├── pyproject.toml             # Python package config (uv/pip)
├── uv.lock                    # Dependency lock file
└── README.md                  # Usage documentation
```

## Tools Provided

| Tool | Purpose |
|------|---------|
| `tag_process(name, command, color)` | Tag process with Nord Aurora color |
| `untag_process(name, command)` | Remove tag from process |
| `set_filter_tagged(enabled)` | Show only tagged processes |
| `list_tagged_processes()` | List all tagged processes |
| `show_logs_panel(name, pid)` | Open Logs panel for process |
| `hide_logs_panel()` | Close Logs panel |
| `select_process(name, command, pid)` | Select process in list |
| `set_log_level_filter(level)` | Filter logs by level |
| `get_mbtop_live_state()` | Query mbtop state via socket |

## Communication

**Config-based** (persisted):
- Writes to `~/.config/mbtop/mbtop.toml`
- mbtop auto-reloads within ~2 seconds
- Process tags stored in `[logging.processes]` array

**Socket-based** (real-time):
- Unix socket at `~/.config/mbtop/mbtop.sock`
- JSON command/response protocol
- Commands: `show_logs`, `hide_logs`, `select_process`, `get_state`

## Colors (Nord Aurora)

| Name | Use Case |
|------|----------|
| red | Alerts, errors, critical |
| orange | Warnings, attention |
| yellow | Active work, in progress |
| green | Healthy, OK, success |
| violet | Info, background |
| blue | Calm, secondary |

## Installation

```bash
cd mcp-server
uv sync          # or: pip install -e .
```

## Claude Desktop Config

```json
{
  "mcpServers": {
    "mbtop": {
      "command": "uv",
      "args": ["--directory", "/path/to/mbtop/mcp-server", "run", "mbtop-mcp"]
    }
  }
}
```

## Notes

- Requires mbtop v1.7.0+ with TOML config
- `command` parameter REQUIRED for process matching (prevents ambiguity)
- Socket commands require mbtop to be running
- Config changes work even if mbtop not running (applied on next start)
