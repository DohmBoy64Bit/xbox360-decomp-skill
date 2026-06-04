## Phase 1A: ReXGlue Reconnaissance and Ghidra-Assisted Analysis

Use this phase to identify function boundaries, indirect branches, switch tables, runtime calls, unsafe control flow, data references, and likely hook points before or during ReXGlue generation.

Tasks:

1. Confirm the user has a lawful game dump and extracted `default.xex`.
2. Keep the original `.xex` unmodified.
3. Verify the exact ReXGlue command set with `rexglue --help` and, when needed, subcommand help or local docs.
4. Import or inspect the `.xex` in Ghidra and confirm GhidraMCP can access the target program when possible.
5. Use GhidraMCP to confirm function starts, branch targets, data references, jump tables, crash addresses, xrefs, and decompiler context.
6. Identify indirect branch patterns and classify them using the **Indirect Control Flow Rules** section.
7. Patch ReXGlue configuration only with evidence from ReXGlue output, GhidraMCP/user-provided Ghidra output, disassembly, local helper script output, or logs.

Do not guess function boundaries. If analysis is ambiguous, query GhidraMCP for targeted disassembly, decompilation, xrefs, and jump targets. Request manual lookup only if MCP cannot provide the artifact.

Manual fallback request shape:

```text
Please open Ghidra and go to 0x82XXXXXX.
Paste:
1. The containing function name and start address.
2. 20 PPC instructions before and after the address.
3. XREFs to the function.
4. Any computed branch, jump table, vtable, or switch table Ghidra detected.
```


## Phase 2A: ReXGlue Project Generation and Configuration

Use ReXGlue as the selected end-to-end pipeline for project generation, translation/recompilation configuration, runtime binding, hook registration, and generated code production.

Tasks:

1. Run only verified ReXGlue commands.
2. Point ReXGlue at the verified `.xex` and project assets only after confirming required CLI options or project configuration.
3. Keep ReXGlue manifests/config files under version control.
4. Make translation, mapping, hook, switch-table, and runtime changes through ReXGlue-supported configuration whenever possible.
5. Do not manually refactor massive generated C++ files.
6. Do not import XenonRecomp-generated C++ into the ReXGlue project unless the user explicitly abandons the ReXGlue-only constraint.

When adjusting ReXGlue configuration:

- Preserve guest address semantics.
- Preserve PowerPC big-endian data behavior.
- Preserve signedness, overflow behavior, saturation, rounding, NaN behavior, and vector lane order.
- Mark uncertain instructions or runtime calls as unresolved until evidence is available.
- Prefer hook/config changes over editing generated code.
- Verify function boundaries, call sites, vtables, branch targets, imports, and data references before writing hooks.

Generated C++ should be treated as machine output. It may be massive and structured around explicit guest-state/runtime models.

Custom logic belongs in ReXGlue hook files, helpers, runtime glue, or configuration-driven patches.


## Phase 3: Runtime Scaffolding, Hooks, and Subsystem Replacement

Use this phase to implement or stub missing Xbox 360 runtime behavior through ReXGlue-supported mechanisms.

Runtime responsibilities may include:

- Xbox kernel/API stubs or replacements.
- Filesystem path mapping.
- Timing, threading, events, and synchronization.
- Input mapping.
- Audio/XMA/streaming replacement paths.
- GPU-facing command handling or high-level rendering replacements.
- Host renderer resource lifetime.
- Guest-to-host memory translation.
- Runtime asset lookup.
- Hook registration and dispatch.
- Save/profile handling.
- Logging that preserves both host and guest context.

The ReXGlue project should own both the generated/recompiled code path and the runtime glue path for this workflow.

Keep the original `default.xex` in assets only as a reference, runtime asset source, base-memory source, or dispatch-table source when the selected ReXGlue template/project architecture requires it. Verify this behavior from project files instead of assuming it.

Do not silently patch or overwrite the original `.xex`.

### Hook Discipline

When creating or changing a hook, document:

```text
Hook name:
Guest address:
Containing function:
Original PPC instruction window:
Registers read:
Registers written:
Guest memory read/write ranges:
Host systems touched:
Condition for taking the hook:
Fallthrough/continuation behavior:
Why this exact address is safe:
How to disable for A/B testing:
How to verify it fired:
Regression risk:
Evidence source:
```

Prefer hooks that are:

- Narrow.
- Evidence-backed.
- Placed before the first unsafe guest load/store/call.
- Easy to disable.
- Logged with enough guest context to debug regressions.

Avoid hooks that are:

- Broad null guards without proof.
- Patches that skip large unknown regions.
- Fixes that only make a crash disappear without proving correctness.
- Placed after the instruction that already performed the unsafe load/store.
- Mixed directly into generated code when a ReXGlue-supported hook/config path exists.
