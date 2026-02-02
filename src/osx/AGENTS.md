# src/osx/ - Apple Silicon Monitoring

## Overview

macOS-specific metrics collection. IOReport framework for GPU/Power/ANE, IOHIDSensors for temperature, SMC for fans.

## Structure

```
osx/
├── mbtop_collect.cpp    # Main collector: Cpu, Mem, Net, Proc, Disk data
├── apple_silicon_gpu.cpp/hpp  # IOReport GPU/Power/ANE metrics (1.5K lines)
├── sensors.cpp/hpp      # IOHIDSensors temperature monitoring
└── smc.cpp/hpp          # SMC interface for fans/temps (Intel fallback)
```

## Where to Look

| Task | File | Key Functions |
|------|------|---------------|
| GPU utilization | apple_silicon_gpu.cpp | `AppleSiliconGpu::collect()` |
| Power readings | apple_silicon_gpu.cpp | IOReport "Energy Model" channel |
| ANE activity | apple_silicon_gpu.cpp | IOReport "H11ANE" channel |
| CPU/GPU temp | sensors.cpp | `IOHIDSensors::getTemperature()` |
| Per-process GPU | apple_silicon_gpu.cpp | `collect_gpu_processes()` |
| GPU memory | apple_silicon_gpu.cpp | IORegistry AGXAccelerator query |
| Fan RPM | smc.cpp | SMC key "F0Ac", "F1Ac" |

## IOReport Channels

| Channel Group | Subgroup | Data |
|---------------|----------|------|
| "GPU Stats" | "GPUPH" | P-state residency → frequency |
| "Energy Model" | "GPU Energy" | GPU power (watts) |
| "Energy Model" | "CPU Energy" | CPU power (watts) |
| "Energy Model" | "ANE*" | Neural Engine power |
| "H11ANE" | "ANECPU Commands Sent" | ANE activity (cmds/sec) |

## Key Patterns

**Dynamic library loading** (no public headers):
```cpp
dlopen("/usr/lib/libIOReport.dylib", RTLD_NOW);
IOReportCopyChannelsInGroup = dlsym(handle, "IOReportCopyChannelsInGroup");
```

**Async sampling with timeout** (IOReport can hang):
```cpp
dispatch_async(..., ^{ sample = IOReportCreateSamples(...); });
dispatch_semaphore_wait(sem, 2*NSEC_PER_SEC);  // 2s timeout
```

**Atomic updates** (cross-thread):
```cpp
Shared::gpuPower.store(watts, std::memory_order_release);
Shared::cpuTemp.store(temp, std::memory_order_release);
```

## Anti-Patterns

| Don't | Why |
|-------|-----|
| Call IOReport synchronously | Can block 10+ seconds |
| Skip sensor fallbacks | Different chips have different sensor names |
| Hardcode frequency tables | Query from IORegistry "voltage-states9" |
| Assume unified memory size | Query via sysctl "iogpu.wired_limit_mb" |

## Chip Variations

| Sensor | M1/M2/M3 | M4+ |
|--------|----------|-----|
| GPU temp | "GPU MTR Temp Sensor" | "PMU tdie" |
| CPU temp | "PMU TP*" | "eACC*", "pACC*" |
| ANE channel | "H11ANE" | "H11ANE" (same) |

## Notes

- **No sudo required**: Uses public IOKit/IOReport APIs
- **Unified memory**: GPU shares system RAM, not discrete VRAM
- **E/P cores**: Detected via `hw.perflevel0/1.physicalcpu` sysctl
- **GPU freq table**: Parsed from IORegistry pmgr "voltage-states9"
