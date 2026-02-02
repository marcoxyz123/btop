# src/ - Core Application

## Overview

Main application source. 35+ files, 113K lines. Platform-agnostic code + platform directories.

## Structure

```
src/
├── main.cpp                 # Entry point (delegates to btop_main)
├── mbtop.cpp/hpp            # Main app logic, signal handlers, runner thread
├── mbtop_shared.cpp/hpp     # Namespaces: Global, Runner, Shared, Cpu, Mem, Net, Proc, Gpu, Pwr, Logs
├── mbtop_draw.cpp/hpp       # Panel rendering (7K lines) - all draw functions
├── mbtop_menu.cpp/hpp       # V2 menu system (6.6K lines) - settings, presets
├── mbtop_input.cpp/hpp      # Keyboard + mouse handlers
├── mbtop_config.cpp/hpp     # TOML config, INI migration, instance locking
├── mbtop_theme.cpp/hpp      # Theme loading, color management
├── mbtop_tools.cpp/hpp      # Utilities: string ops, formatting, timing
├── mbtop_socket.cpp/hpp     # Unix socket for MCP communication
├── mbtop_log.cpp/hpp        # Logging utilities
├── mbtop_cli.cpp/hpp        # CLI argument parsing
└── {osx,linux,freebsd,netbsd,openbsd}/  # Platform collectors
```

## Where to Look

| Task | File | Key Functions |
|------|------|---------------|
| Add panel | mbtop_draw.cpp | `Cpu::draw()`, `Gpu::draw()`, `Logs::draw()` patterns |
| Add menu tab | mbtop_menu.cpp | `Menu::show()`, tab rendering sections |
| Handle key | mbtop_input.cpp | `Input::process()` - keyboard section |
| Handle click | mbtop_input.cpp | `Input::process()` - mouse section |
| Add config | mbtop_config.cpp | `Config::conf`, `Config::ints`, `Config::bools` |
| Add namespace | mbtop_shared.hpp | Follow Cpu/Mem/Net/Proc/Gpu/Pwr/Logs pattern |

## Large Files (>1K lines)

| File | Lines | Complexity |
|------|-------|------------|
| mbtop_draw.cpp | 7,040 | Panel rendering, graphs, box drawing |
| mbtop_menu.cpp | 6,624 | V2 settings menu, preset editor |
| osx/mbtop_collect.cpp | 3,357 | macOS metrics collection |
| linux/mbtop_collect.cpp | 3,363 | Linux metrics + GPU vendors |
| mbtop_config.cpp | 2,020 | Config I/O, migration, validation |
| mbtop.cpp | 1,535 | Init sequence, main loop, signals |
| osx/apple_silicon_gpu.cpp | 1,466 | IOReport GPU/Power/ANE metrics |
| mbtop_input.cpp | 1,444 | Keyboard + mouse dispatch |

## Initialization Order (btop_main)

1. Drop SUID privileges
2. Parse CLI args
3. Setup config paths + logger
4. Detect binary path (for themes)
5. Load config (TOML/INI migration)
6. Init locale (UTF-8)
7. Init terminal (raw mode)
8. `Shared::init()` - platform-specific (core counts, GPU init)
9. Load themes
10. Setup signal handlers
11. Start Runner thread
12. Start Socket server
13. Enter main loop

## Conventions

- Namespace per panel: `Cpu`, `Mem`, `Net`, `Proc`, `Gpu`, `Pwr`, `Logs`
- Each has: `collect()` (gather data), `draw()` (render), data structs
- Platform code in `src/{platform}/mbtop_collect.cpp`
- Shared atomics for cross-thread data: `Shared::cpuPower`, `Shared::gpuTemp`, etc.

## Anti-Patterns

| Don't | Why |
|-------|-----|
| Block in collect() | Runs in Runner thread - blocks UI |
| Skip mouse handler | Keyboard + mouse MUST be in sync |
| Use raw `cout` | Use `Term::` or fmt for output |
| Add new #ifdef platforms | Use platform directories instead |
