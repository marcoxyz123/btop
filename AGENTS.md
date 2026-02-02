# mbtop Knowledge Base

**Generated:** 2026-02-02
**Commit:** 7b4f3b2
**Branch:** main

## Overview

macOS-focused terminal system monitor with comprehensive Apple Silicon support. Forked from btop, now independent with GPU/Power/ANE monitoring unique to mbtop.

## Structure

```
mbtop/
├── src/                    # Core application (see src/AGENTS.md)
│   ├── osx/                # Apple Silicon specifics (see src/osx/AGENTS.md)
│   ├── linux/              # Linux + GPU vendors (NVML/RSMI/Intel)
│   └── {freebsd,netbsd,openbsd}/  # BSD platform collectors
├── include/                # Third-party: fmt, toml.hpp (DO NOT EDIT)
├── mcp-server/             # Python MCP server (see mcp-server/AGENTS.md)
├── themes/                 # 39 color themes (.theme files)
├── tests/                  # ANE/GPU stress tests (Swift/C++)
├── packaging/{deb,rpm,macos}/  # Package build scripts
└── Makefile                # Primary build system
```

## Where to Look

| Task | Location | Notes |
|------|----------|-------|
| Add UI panel | `src/mbtop_draw.cpp` | 7K lines - find similar panel, copy pattern |
| Add menu option | `src/mbtop_menu.cpp` | 6.6K lines - V2 menu system |
| Handle keyboard | `src/mbtop_input.cpp` | Check BOTH keyboard AND mouse handlers |
| Add config option | `src/mbtop_config.cpp` | TOML format, auto-migration from INI |
| Apple Silicon metrics | `src/osx/apple_silicon_gpu.cpp` | IOReport framework |
| Process tagging (AI) | `mcp-server/src/mbtop_mcp/` | Python FastMCP server |
| Platform collector | `src/{platform}/mbtop_collect.cpp` | One per OS |

## Code Map

| Namespace | File | Purpose |
|-----------|------|---------|
| `Global` | mbtop_shared.hpp | Version, quitting, resized flags |
| `Runner` | mbtop_shared.hpp | Main loop control, threading |
| `Shared` | mbtop_shared.hpp | Core counts, power readings, temps |
| `Cpu` | mbtop_shared.hpp | CPU panel data + draw |
| `Mem` | mbtop_shared.hpp | Memory/disk panel |
| `Net` | mbtop_shared.hpp | Network panel |
| `Proc` | mbtop_shared.hpp | Process list + tree view |
| `Gpu` | mbtop_shared.hpp | GPU panel (Apple Silicon/NVIDIA/AMD/Intel) |
| `Pwr` | mbtop_shared.hpp | Power panel (CPU/GPU/ANE watts) |
| `Logs` | mbtop_shared.hpp | macOS unified logging panel |

## Conventions

- **Tabs** for indentation (4-space width)
- **Alternative operators**: `and`, `or`, `not` instead of `&&`, `||`, `!`
- **Braces**: Opening brace on same line as statement
- **Heavy STL usage**: Prefer algorithms over raw loops
- **fmt library**: Use for string formatting (included, header-only)
- **Platform isolation**: `src/{platform}/` directories, not `#ifdef`

## Anti-Patterns

| Forbidden | Reason |
|-----------|--------|
| Edit `include/toml.hpp` | Auto-generated, will be overwritten |
| Apply theme during menu | Causes UI corruption - defer to menu exit |
| Fix keyboard without mouse | ALWAYS update BOTH input handlers |
| Build packages locally | Use GitLab CI pipeline only |
| Fast-forward merge | Use `--no-ff` to preserve branch history |
| Commit to main directly | Use feature branches |

## Commands

```bash
# Build
make                        # Standard build
make DEBUG=true             # Debug build (-O0 -g)
make clean && make          # Clean rebuild

# Run
./bin/mbtop                 # Launch mbtop
./bin/mbtop --help          # Show options

# Git workflow
git checkout -b feature/x   # New feature branch
git commit -m "type(scope): description"
git checkout main && git merge feature/x --no-ff -m "Merge feature/x: description"
```

## Notes

- **Dual repos**: GitLab (dev) → GitHub (public releases)
- **Dual build systems**: Makefile (primary) + CMake (secondary)
- **C++23 required**: Uses `std::expected`, `std::ranges::to`, etc.
- **GPU support**: macOS ARM64 only (via IOReport), Linux x86_64 (NVML/RSMI/Intel)
- **Multi-instance**: Secondary instances get `(S)` marker, won't overwrite config
- **MCP server**: AI assistants can tag/filter processes via Unix socket
