## Xbox 360 Function Discovery and Game-Function Classification Pass

Use this pass before broad hook work, handwritten replacement work, or full-decomp source recovery. The goal is to build a high-confidence inventory of functions and classify which functions are game logic versus compiler/runtime/platform/recompilation support code.

Do not restart an already-started project. Preserve existing structure, labels, scripts, configs, generated output, and ledgers unless evidence proves they are wrong. Do not rename functions blindly. Use conservative names until multiple evidence sources agree.

### Function Boundary Evidence Priority

Start from the strongest Xbox 360-specific boundary evidence available:

```text
1. PDB/map/export/symbol data when lawfully available and version-matched.
2. XEXLoaderWV/Ghidra function starts imported with the correct XEX image base.
3. ReXGlue, XenonAnalyse, or XenonRecomp discovered function metadata from the selected local toolchain.
4. Entry point, startup path, TLS/static initializer paths, and global constructor/destructor patterns when identifiable.
5. Direct PPC call/branch targets, especially `bl` targets inside executable ranges.
6. Indirect call/branch targets recovered from `mtctr` + `bctr` / `bctrl` patterns.
7. Switch-table targets and jump-table entries.
8. Vtable entries, callback arrays, import thunk tables, and function-pointer tables in read-only/data regions.
9. Kernel import wrappers, SDK/runtime thunks, and generated guest/host boundary stubs.
10. Runtime evidence from crash PCs, trace logs, hook logs, register dumps, or debugger sessions.
11. Manual recovery only when all higher-confidence sources are insufficient.
```

Do not trust one source alone when the function boundary is unusual. Verify tail calls, thunk chains, code/data boundaries, alignment padding, branch islands, hand-written PPC/VMX code, generated-runtime stubs, and functions adjacent to jump tables or embedded data.

### Xbox 360 Function Classification

Classify every discovered function before calling it game logic:

```text
xex_entry_startup
crt_compiler_runtime
xbox_sdk_kernel_import
import_thunk
recomp_generated_stub
runtime_glue_or_host_boundary
allocator_memory_helper
string_math_helper
threading_timer_sync
filesystem_asset_loader
save_profile_storage
audio
input
renderer_xenos_submission
host_renderer_bridge
network_or_service_stub
ui_menu_frontend
gameplay_state_machine
entity_actor_logic
physics_collision
script_or_mission_logic
resource_decompression
middleware
unknown_internal
```

Use categories as evidence labels, not permanent names. A function can have multiple roles, but record the primary reason it matters to the project.

### Required Function Inventory Fields

Create or update `docs/function_ledger.md` and cross-link to `docs/address_ledger.md`, `docs/runtime_boundaries.md`, and the selected tool config when relevant.

```text
Function:
Guest start:
Guest end:
Size:
XEX image offset:
File offset if known:
Current name:
Proposed name:
Discovery source:
Containing region/section:
Callers:
Callees:
Direct branch/call evidence:
Indirect branch evidence:
Imports / kernel calls referenced:
Strings/assets referenced:
Globals/data references:
Vtable/callback/switch-table involvement:
VMX/FPU/CR/XER involvement:
Likely category:
Selected track impact: ReXGlue / XenonRecomp / full decomp / runtime glue / unknown
Evidence:
Confidence: Known / Likely / Tentative / Unknown
Next proof needed:
```

### Conservative Naming Rule

Do not use confident names unless raw PPC evidence, xrefs, data references, strings, runtime traces, or known symbols support them. Prefer candidate names until proven:

```text
kernel_file_open_wrapper_candidate
asset_stream_load_candidate
xenos_submit_candidate
audio_submit_candidate
input_poll_candidate
save_mount_candidate
entity_update_candidate
mission_state_candidate
vtable_dispatch_candidate
switch_dispatch_candidate
```

If a name came from a generated tool, generated SDK metadata, or a heuristic script, record that source explicitly and keep confidence separate from readability.

### Cross-Check Function Identity

Before promoting a function to `Known` or using it as a hook/replacement boundary, cross-check with:

```text
Call graph position
Raw PPC prologue/epilogue and nonvolatile register saves
Branch targets and branch-delay-free PPC control flow
Imports / Xbox kernel calls
Strings, asset paths, hashes, or file extensions
Guest data references and struct offsets
Vtable membership or callback-array membership
Switch-table bounds and known cases
VMX/FPU/CR/XER behavior
Generated ReXGlue/XenonRecomp metadata if using a recomp track
Runtime traces, crash PCs, register dumps, or hook logs
Original-vs-rebuilt behavior when replacing or shimming code
```

### Output Groups

After the first pass, report results in groups:

```text
Definitely not game logic:
Platform/runtime/middleware support:
Likely game systems:
Unknown internal functions:
High-priority functions for decompilation:
High-priority functions for runtime tracing:
High-risk indirect-control-flow functions:
Functions missing from Ghidra or selected tool metadata:
```

### Report Format

Use this report format when summarizing the pass:

```text
Evidence proves:
Likely:
Tentative:
Unknown:
Functions discovered:
Functions missing from Ghidra/tool metadata:
Likely game functions:
Likely runtime/platform functions:
High-risk indirect control-flow sites:
Recommended next pass:
```

This pass is for evidence, classification, and planning. Do not rewrite code, add hooks, patch generated output, or modify runtime behavior unless the user explicitly asks for implementation after the inventory is complete.
