# mbtop MCP Test Runbook

**Trigger:** "Hey Claude start the MCP test runbook"

This runbook demonstrates ALL mbtop MCP server functionalities for a feature presentation video (~30 seconds).

---

## Pre-flight Check

First, verify clean state - no existing process configs:

```
mcp_mbtop_get_mbtop_status()
```

Expected: `Process configs: 0, Tagged processes: 0`

If configs exist, clean them first with UI "Clear All" button (press `c` on any process).

---

## Step 1: Tag 6 processes with all 6 Aurora colors

**Comment:** "Tagging 6 visible processes with all 6 Nord Aurora colors - red, orange, yellow, green, violet, and blue"

```
mcp_mbtop_tag_process(name="mbtop", command="mbtop", color="red", display_name="mbtop")
mcp_mbtop_tag_process(name="Finder", command="/System/Library/CoreServices/Finder.app/Contents/MacOS/Finder", color="orange", display_name="Finder")
mcp_mbtop_tag_process(name="Adguard", command="/Applications/Adguard.app/Contents/MacOS/Adguard", color="yellow", display_name="Adguard")
mcp_mbtop_tag_process(name="AltTab", command="/Applications/AltTab.app/Contents/MacOS/AltTab", color="green", display_name="AltTab")
mcp_mbtop_tag_process(name="BetterDisplay", command="/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay", color="violet", display_name="BetterDisplay")
mcp_mbtop_tag_process(name="Tabby", command="/Applications/Tabby.app/Contents/MacOS/Tabby", color="blue", display_name="Tabby")
```

**Comment:** "Verifying all 6 processes are tagged"

```
mcp_mbtop_list_tagged_processes()
```

---

## Step 2: Rename a tagged process

**Comment:** "Renaming 'mbtop' to 'System Monitor' - watch the display name change in the process list"

```
mcp_mbtop_tag_process(name="mbtop", command="mbtop", color="red", display_name="System Monitor")
```

**Comment:** "Verifying the rename - mbtop now shows as 'System Monitor'"

```
mcp_mbtop_list_tagged_processes()
```

---

## Step 3: Enable tagged filter

**Comment:** "Enabling the tagged filter - now only the 6 tagged processes are visible"

```
mcp_mbtop_set_filter_tagged(enabled=true)
```

---

## Step 4: Configure application log for mbtop

**Comment:** "Setting up an application log file path for the mbtop process"

```
mcp_mbtop_set_process_log(name="mbtop", command="mbtop", log_path="~/.config/mbtop/mbtop.log")
```

---

## Step 5: Show Logs panel and select mbtop

**Comment:** "Activating the Logs panel for mbtop - this selects the process and shows its logs"

```
mcp_mbtop_activate_process_log(name="mbtop")
```

---

## Step 6: Verify current state

**Comment:** "Checking the live state - logs panel should be shown with system source, app log available"

```
mcp_mbtop_get_mbtop_live_state()
```

---

## Step 7: Switch to application logs

**Comment:** "Switching from macOS unified system logs to the application's own log file"

```
mcp_mbtop_set_log_source(source="application")
```

---

## Step 8: Change log level filter

**Comment:** "Filtering logs to show only ERROR level - watch the log entries change"

```
mcp_mbtop_set_log_level_filter(level="error")
```

---

## Step 9: Reset log level and switch back to system

**Comment:** "Resetting log level back to INFO and switching back to system logs"

```
mcp_mbtop_set_log_level_filter(level="info")
mcp_mbtop_set_log_source(source="system")
```

---

## Step 10: Hide Logs panel

**Comment:** "Hiding the Logs panel"

```
mcp_mbtop_hide_logs_panel()
```

---

## Step 11: Remove processes one by one (in filtered view)

**Comment:** "Now removing each tagged process one by one - watch them disappear from the filtered view"

```
mcp_mbtop_remove_process_config(name="Tabby", command="/Applications/Tabby.app/Contents/MacOS/Tabby")
```

**Comment:** "5 remaining..."

```
mcp_mbtop_remove_process_config(name="BetterDisplay", command="/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay")
```

**Comment:** "4 remaining..."

```
mcp_mbtop_remove_process_config(name="AltTab", command="/Applications/AltTab.app/Contents/MacOS/AltTab")
```

**Comment:** "3 remaining..."

```
mcp_mbtop_remove_process_config(name="Adguard", command="/Applications/Adguard.app/Contents/MacOS/Adguard")
```

**Comment:** "2 remaining..."

```
mcp_mbtop_remove_process_config(name="Finder", command="/System/Library/CoreServices/Finder.app/Contents/MacOS/Finder")
```

**Comment:** "1 remaining..."

```
mcp_mbtop_remove_process_config(name="mbtop", command="mbtop")
```

**Comment:** "All gone! The list is now empty."

---

## Step 12: Switch to clean PROC view

**Comment:** "Disabling the tagged filter to show all processes - back to normal view"

```
mcp_mbtop_set_filter_tagged(enabled=false)
```

---

## Step 13: Final verification

**Comment:** "Verifying clean state - no configs, no tags, filter disabled"

```
mcp_mbtop_get_mbtop_status()
```

Expected: `Process configs: 0, Tagged processes: 0, Tagged filter: disabled`

---

## Summary of MCP Tools Demonstrated

| Tool | Purpose |
|------|---------|
| `tag_process` | Tag process with color and display name (also used for renaming) |
| `list_tagged_processes` | List all tagged processes |
| `set_filter_tagged` | Enable/disable tagged-only filter |
| `set_process_log` | Configure application log path |
| `activate_process_log` | Select process and show logs |
| `get_mbtop_live_state` | Get real-time UI state via socket |
| `set_log_source` | Switch between system/application logs |
| `set_log_level_filter` | Filter by log level |
| `hide_logs_panel` | Hide the Logs panel |
| `remove_process_config` | Remove a process config |
| `get_mbtop_status` | Get config file status |

---

## Notes for Recording

- Ensure mbtop is running with PROC panel visible
- All 6 processes (mbtop, Finder, Adguard, AltTab, BetterDisplay, Tabby) must be running
- Execute commands with ~1 second pause between for visual effect
- Total runtime: ~35 seconds
