## Common failure modes (ReXGlue / XenonRecomp / 360tools)

```text
Wrong XEX image base
Wrong guest address / image offset mapping
Bad function boundary
Unresolved bctr/bctrl target
Missing switch-table config (Track D: add [[switch_tables]] in TOML; fallback extract_switch_tables.py)
Vtable slot misidentified
Import thunk treated as game logic
Guest pointer treated as host pointer
Host hook reads big-endian data as little-endian
Vector lane order mismatch
Hook placed after unsafe instruction
Overbroad hook skips required initialization
Missing kernel/API stub
Wrong filesystem root
Missing extracted asset
Optional file probe mistaken for fatal error
Audio queue/timing mismatch
GPU-facing data not converted correctly
D3D12 DEVICE_HUNG / TDR after bad command/resource state
Generated source manually patched instead of fixing config
CMake/clang-cl environment mismatch
Submodule or ReXGlue / XenonRecomp version mismatch
Ghidra XEXLoaderWV missing or incompatible with installed Ghidra version
GhidraMCP installed but plugin/server/bridge not running
MCP frontend config path guessed incorrectly or overwritten without backup
Wrong direct-CLI vs CMake-codegen-target assumption
Built exe cannot locate assets because runtime expects a different working directory or asset argument
Extract failed (Track D: re-run extract_stfs.py / extract_xex_direct.py / xex_info.py)
Missing ReXGlue SDK patch (Track D: see patches/rexglue_patches_audit.md — setjmp, switch discovery, D3D12 UAV)
VdSwap half-speed (Track D: docs/speed-fix.md QPC limiter — not legacy __rdtsc scaling)
```

Map each failure to one layer:

```text
Input asset / XBLA extract (360toolsUpdated)
default.xex path / assets layout
ReXGlue SDK + patches/ applied
mygame_config.toml / codegen
Generated code
Hook layer
Runtime/platform shim (templates/advanced, stubs)
Host build system
Host graphics/audio/input backend
Documentation/test process
```

## Stuck on the same failure

If the same crash, unregistered VA, or GPU/boot blocker appears twice without new evidence, stop patching locally. Follow [stuck-cross-recomp.md](stuck-cross-recomp.md) (repomix/SDK/web, then Ghidra + original game tree + `generated/` + hooks). Cite the source that justified the next change.

## Build System

Use the build system generated or recommended by ReXGlue. If using CMake presets or Ninja, keep the commands tied to the actual generated project.

Generic checks:

```cmd
cmake --list-presets
cmake --build <build_dir> --config Release
```

Do not invent exact preset names, target names, or paths. Use generated project files, local docs, or user-provided output as evidence.

When editing `CMakeLists.txt` or build scripts:

- Use `clang-cl` when the project supports it.
- Keep generated source files grouped separately from hooks/runtime code.
- Prefer target-specific flags over global flags.
- Use response files or source grouping if command lines become too long.
- Avoid manually refactoring generated translation units.
- Keep hooks and runtime glue separate from generated code when practical.
- Document why each build flag or source-list patch is needed.

Common build failures:

```text
Compiler not on PATH
Wrong Developer Command Prompt
Missing Windows SDK
Missing generated files
Bad hook declaration
Unresolved runtime symbols
Command line too long
Out-of-memory compile of huge generated translation units
CMake preset mismatch
Submodules not initialized
ReXGlue SDK version mismatch
```


## Runtime Debugging Workflow

When analyzing a crash or hang:

1. Identify exception type or failure class.
2. Record host instruction pointer.
3. Record guest PPC address if available.
4. Record guest registers when available.
5. Compute `guest_offset = guest_address - xex_image_base` only after confirming image base.
6. Map the guest address to generated C++, original PPC disassembly, GhidraMCP function evidence, or hook code.
7. Determine whether the failure belongs to translation, memory mapping, endian handling, runtime scaffolding, asset pathing, GPU/audio/input handling, hook logic, or build/link configuration.
8. Patch the narrowest layer that owns the failure.
9. Rebuild and test.
10. Record the result in `docs/regression_log.md`.

Debug response format:

```text
Observed failure:
Evidence:
Guest address / function:
Host location:
Likely layer:
Why this layer:
Next proof needed:
Proposed narrow fix:
Verification:
Rollback/A/B test:
```

Never call a change correct merely because a crash moved or disappeared.


## D3D12 DEVICE_HUNG / TDR Triage

When the active blocker is `D3D12 DEVICE_HUNG`, TDR, GPU timeout, or renderer device loss, do not immediately blame the recompiler.

Classify possible causes:

- Invalid command buffer or draw parameters.
- Bad resource lifetime.
- Wrong texture/buffer format.
- Wrong endian conversion for GPU-facing data.
- Guest pointer passed as host resource pointer.
- Missing synchronization.
- Use-after-free in host renderer state.
- Infinite GPU work or shader/compiler issue.
- CPU hook skipped initialization that renderer depended on.
- Host driver/backend issue.

Evidence to collect:

```text
Runtime log around device loss:
Last guest function before GPU submission:
Last hook fired:
Resource creation/destruction logs:
Guest pointers and translated host pointers:
Draw/dispatch parameters:
Render target/depth/texture formats:
Backend validation/debug-layer output:
A/B result with suspected hook disabled:
```

Use graphics debug layers or backend validation when available, but do not claim validation results without logs.


## Endianness Bug Triage

When data "looks right in guest memory but wrong in host hooks," suspect host-side probing before changing generated guest stores.

Check:

- Whether the generated code already byte-swaps guest memory stores.
- Whether the hook reads guest memory with the correct endian helper.
- Whether a struct was copied into a host-native C++ struct without conversion.
- Whether the field offset is correct.
- Whether vector or packed data has lane-order differences.
- Whether the object is guest memory, host memory, or a mixed wrapper.

Rule:

```text
Fix the boundary that reads/writes incorrectly.
Do not change guest stores unless raw PPC and generated code prove the store itself is wrong.
```


## Filesystem and Asset Debugging

When logs show missing files, blank paths, VFS warnings, or asset lookup failures:

1. Identify whether the path is guest-provided, ReXGlue-mapped, or host-generated.
2. Log both the original guest string and final host path.
3. Check case sensitivity and slash direction.
4. Verify the asset root.
5. Verify whether the file is loose, packed, compressed, or generated.
6. Do not silence missing-file warnings until you know whether the game treats them as fatal, optional, or probing behavior.

Recommended evidence:

```text
Guest path string:
Host mapped path:
Asset root:
File exists:
Caller function:
Fallback behavior:
Fatal or optional:
```
