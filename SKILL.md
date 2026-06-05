---
name: xbox360-decomp
description: |
  Xbox 360/XBLA RE and static-recomp (A ReXGlue, B XenonRecomp, C matching decomp, D 360toolsUpdated+ReXGlue). Use for default.xex, STFS/LIVE, Xenon PPC, guest vs host VA, ReXGlue hooks/config, Ghidra/XEXLoaderWV/GhidraMCP, switch/bctrl indirect, Xenos, decomp.me, ledgers, bring-up (unregistered VA, invalid/unregistered function, DEVICE_HUNG, endian, same crash twice)—Guardian Heroes, After Burner, Daytona, XBLA ports—even if they say 360 static recomp or PC port. PPC/Ghidra/code/ evidence; SDK file:line for hooks; stuck loop before repeat patches. Track D: DohmBoy64Bit/360toolsUpdated extract→rexglue init/codegen, templates/advanced, SDK patches 0001-0005. Prefer 360tools skill for extract-only without Ghidra/stuck. NOT for OG .xbe (xboxrecomp), Win Unity/Unreal PE, Xenia, N64, homework/C++, malware, DRM bypass.

metadata:
  mcpmarket-version: 1.3.1
---

# Xbox 360 Reverse Engineering, Recompilation, and Full Decompilation

Lawful Xbox 360 / XBLA workflow hub: pick a track, verify evidence, open one reference file at a time.

> **Install path:** copy or link this folder to `.cursor/skills/xbox360-decomp/` (or your agent’s skills directory). Keep title-specific hashes, commands, and paths in root `AGENTS.md`; update both when project facts change.

## Related skills

| Skill | Use when |
| --- | --- |
| **360tools** | Track D extraction + `rexglue init` quick path — user wants 360toolsUpdated scripts only, not full RE methodology |
| **xboxrecomp** | Original Xbox `default.xbe`, x86, NV2A — not Xenon/XEX |
| **windows-game-matching-decomp** | Win32/Unity/Unreal PE matching with no Xbox 360 context |

## Quick navigation

Open the track or topic file before giving deep steps — the hub alone is not enough for codegen, Ghidra setup, or 360tools ordering.

## Bundled references

Read when the task needs depth. Do not load all files up front.

| File | When to read |
|------|----------------|
| [references/xbla-stfs.md](references/xbla-stfs.md) | XBLA, STFS, LIVE package, title ID, extraction, `game_data_root` |
| [references/assets-legal.md](references/assets-legal.md) | `.gitignore`, proprietary assets, commit boundaries |
| [references/agents-md.md](references/agents-md.md) | Creating or updating project `AGENTS.md` |
| [references/frontend-rules.md](references/frontend-rules.md) | Cursor/Codex/Claude MCP rule file setup |
| [references/project-templates.md](references/project-templates.md) | Public ReXGlue/Xenon repo layout references |
| [references/dev-environment.md](references/dev-environment.md) | Installing Git, CMake, Ghidra, Python, SDK audit |
| [references/project-setup.md](references/project-setup.md) | New workspace folder layout |
| [references/ghidra-mcp.md](references/ghidra-mcp.md) | XEXLoaderWV, GhidraMCP install, MCP config merge |
| [references/ghidra-evidence.md](references/ghidra-evidence.md) | GhidraMCP evidence protocol, offline fallbacks |
| [references/function-discovery.md](references/function-discovery.md) | Function inventory, classification pass |
| [references/track-rexglue.md](references/track-rexglue.md) | Track A phases R0-R6, codegen, launch |
| [references/track-xenon.md](references/track-xenon.md) | Track B XenonRecomp / XenonAnalyse |
| [references/track-full-decomp.md](references/track-full-decomp.md) | Track C matching / decomp.me |
| [references/track-360tools.md](references/track-360tools.md) | Track D: 360toolsUpdated — XBLA/ISO extract → ReXGlue init/codegen → templates/advanced + SDK patches |
| [references/rexglue-phases.md](references/rexglue-phases.md) | ReXGlue phases 1A-3, hook discipline |
| [references/runtime-hooks.md](references/runtime-hooks.md) | Runtime boundary classification |
| [references/rexglue-config.md](references/rexglue-config.md) | Config/manifest edit safety |
| [references/debug-triage.md](references/debug-triage.md) | Crashes, TDR, endian, filesystem debug |
| [references/stuck-cross-recomp.md](references/stuck-cross-recomp.md) | Same crash twice, no new evidence, blocked boot/GPU — mandatory external research before retry |
| [references/original-game-evidence.md](references/original-game-evidence.md) | PPC/Ghidra/code/ vs generated/ — never assume guest behavior |

### Ghidra quick gate

Before game analysis, confirm setup (full steps in [references/ghidra-mcp.md](references/ghidra-mcp.md)):

```text
[ ] XEXLoaderWV import (or documented raw fallback)
[ ] Expected image base in Ghidra
[ ] GhidraMCP plugin enabled and server started
[ ] Bridge transport matches frontend MCP config
[ ] Health check / tool list OK
```


## Core Rules

Apply across XEX analysis, ReXGlue/Xenon config, generated code, hooks, runtime glue, host builds, and debug:

- **Pick the track first** (A–D below). If the user only needs 360tools script order, open [references/track-360tools.md](references/track-360tools.md) or defer to the **360tools** skill.
- Do not combine ReXGlue and XenonRecomp as sequential workflow stages unless the user explicitly asks to migrate or compare artifacts.
- Do not invent ReXGlue, XenonRecomp, XenonAnalyse, XenosRecomp, Ghidra, decomp.me, compiler, linker, hook, manifest/config, generated path, runtime API, helper wrapper, or instruction-semantics details.
- Before recommending a tool command, config field, hook macro, generated path, compiler preset, Ghidra extension, MCP frontend config, decomp.me setup, or runtime API, verify it from local source/docs/tool output or user-provided output. For ReXGlue APIs, quote **file:line** under the active SDK (`thirdparty/rexglue-sdk`, `REXSDK`, or project-documented path).
- For guest behavior, crashes, and hook placement, read [references/original-game-evidence.md](references/original-game-evidence.md): PPC and Ghidra first, then the project's original-analysis tree (`code/` or path in `AGENTS.md`), then `generated/` and `src/` hooks — never invent semantics.
- When blocked on the same failure without new proof, read [references/stuck-cross-recomp.md](references/stuck-cross-recomp.md) before the next patch. Honor project `.cursor/rules/*` stuck rules when present.
- When the task is environment setup, first audit installed tools, then install or configure only missing pieces. Prefer automatic setup when shell/file access is available, but back up config files before editing and preserve the user's existing frontend configuration style.
- Do not treat guest virtual addresses as host pointers without an explicit runtime address translation layer.
- Do not hand-edit massive generated output as the primary fix. Prefer source metadata, selected-tool config, hook definitions, runtime glue, shim code, or the project's documented patch system (never edit decompiled/generated functions in place when patches exist).
- Do not provide, request, commit, or redistribute copyrighted game binaries, decrypted assets, keys, leaked SDK headers, proprietary SDK libraries, or other proprietary copyrighted game content.
- Prefer user-provided evidence: XEX hash, XBLA/STFS package hash and metadata, logs, raw PowerPC disassembly, Ghidra/GhidraMCP exports, generated C/C++, manifest/config files, CMake output, hook files, address ledgers, asset ledgers, regression logs, and runtime output.
- Every config edit, hook, shim, runtime replacement, script, or build-system patch should explain why it is required, which guest/host boundary it affects, which registers or memory ranges it touches, and how to verify it.
- Keep generated code, handwritten runtime glue, game-specific hooks, scripts, extracted assets, legal-source notes, and documentation separated.
- If evidence is insufficient, state exactly what is unknown and name the narrow artifact that would prove it.

## Track Selection

Before project-specific steps, pick A–D. If the user named a tool or goal, use that track without re-asking. Open the matching reference for phases and commands: A → [track-rexglue.md](references/track-rexglue.md), B → [track-xenon.md](references/track-xenon.md), C → [track-full-decomp.md](references/track-full-decomp.md), D → [track-360tools.md](references/track-360tools.md).

```text
Track A: ReXGlue recompilation
default.xex / extracted game files
  -> ReXGlue analysis/config/generation
  -> generated C++ + ReXGlue runtime/hook layer
  -> Windows/Linux/macOS native executable

Track B: XenonRecomp / XenonAnalyse recompilation
default.xex / extracted game files
  -> XenonAnalyse evidence/config/function metadata
  -> XenonRecomp generated C++
  -> project-owned runtime/kernel/filesystem/renderer/audio/input layer
  -> native executable

Track C: Matching or full decompilation
default.xex / extracted code/data
  -> Ghidra/GhidraMCP + PPC disassembly + function/symbol recovery
  -> compiler/ABI exploration, including decomp.me for small functions when suitable
  -> handwritten C/C++ source that is checked against disassembly, traces, and behavior
  -> optional replacement modules or long-term source port

Track D: 360toolsUpdated pipeline (ReXGlue-native)
XBLA package or Xbox 360 ISO
  -> 360toolsUpdated extract_stfs / extract_iso (+ xex_info, parse_xex_imports)
  -> ReXGlue SDK (+ optional patches/ 0001-0005)
  -> rexglue init -> assets/default.xex -> rexglue codegen
  -> cmake build; optional templates/advanced/
  -> native executable (see references/track-360tools.md)
```

Ask for track selection only when the user's goal is ambiguous. Otherwise infer it:

```text
Mentions ReXGlue, reNut, TiP-Recomp, reDAHM, The Outfit ReXGlue project -> Track A.
Mentions XenonRecomp, XenonAnalyse, Unleashed Recompiled, Sonic Unleashed, XenosRecomp -> Track B.
Mentions matching, full decomp, source recreation, decomp.me, compiler matching, clean C/C++ source -> Track C.
Mentions 360toolsUpdated, 360tools, extract_stfs, rexglue init, templates/advanced, simpsonsarcade/vig8/gh2/ctxbla -> Track D.
Legacy sp00nznet tree (extract_pe, XenonRecomp patches, post_codegen) -> Track D legacy section in track-360tools.md only after confirming their clone.
Mentions "PC port" without a tool -> explain tracks A–D; for new XBLA titles, recommend D (360toolsUpdated) or A.
```

Keep these boundaries:

- ReXGlue-only (A) and 360toolsUpdated (D) both use ReXGlue codegen/runtime; D adds extraction scripts, `templates/advanced/`, and SDK `patches/`.
- Track D (current fork) does **not** use XenonRecomp. Legacy sp00nznet/360tools XenonRecomp path is documented separately in track-360tools.md.
- ReXGlue and XenonRecomp are not sequential stages unless the user is on the legacy 360tools tree or explicitly on Track B.
- XenonAnalyse may assist a XenonRecomp workflow by producing or validating analysis metadata, but do not claim exact outputs or config fields without local evidence.
- XenosRecomp is shader-focused and belongs with renderer/shader work, especially in a XenonRecomp-style project; do not confuse it with CPU recompilation.
- Full decompilation is not the same as static recompilation. It prioritizes readable/rebuildable source and verified compiler behavior over immediate host runtime bring-up.
- A project may use findings from one track as research evidence in another, but never import generated code, hooks, or config across stacks without proving compatibility.

## Reference Priority

Prefer evidence in this order:

1. Current local project files, generated files, configs, hook files, scripts, logs, ledgers, and checked-out tool source for the selected track.
2. Current tool output from ReXGlue, XenonRecomp, XenonAnalyse, XenosRecomp, CMake, compiler/linker, runtime logs, Ghidra MCP, decomp.me attempts, and local helper scripts.
3. User-provided artifacts: raw PPC disassembly, Ghidra exports, crash logs, generated code, manifest/config files, hook files, source snippets, compiler output, map files, and build output.
4. Official source/docs/wiki/release notes for the selected tool: ReXGlue, XenonRecomp, XenonAnalyse, XenosRecomp, decomp.me, Ghidra, CMake, LLVM/Clang, Windows SDK, Git, Python, and PowerPC references.
5. Public Xbox 360 recompilation/decompilation repositories strictly as research-only references for layout, documentation style, build hygiene, hook organization, asset-boundary patterns, renderer organization, installer/data integrity patterns, and runtime-layer separation.
6. Carefully labeled inference.

Always separate:

- What the evidence proves.
- What the source says.
- What is only inferred.
- What remains unverified.

## Address-Space and Memory Discipline

Always distinguish between:

- Guest virtual address
- Original XEX image base
- Original XEX image offset
- File offset inside the encrypted/decrypted/extracted XEX container, when known
- Guest stack address
- Guest heap address
- Guest global/static data address
- Generated C++ CPU state field or register accessor
- ReXGlue guest-memory translation layer
- Runtime asset pointer
- Host process pointer
- Host graphics/audio/input/runtime object pointer

Never cast a guest address directly to a host pointer unless the current ReXGlue runtime explicitly documents that translation.

Useful address formula:

```text
guest_offset = guest_address - xex_image_base
```

Only use this formula after confirming the loaded XEX image base. If a crash log reports both a guest address and a host instruction pointer, keep those mappings separate.

### XEX / Guest / Host Mapping Table

Track major regions as evidence is discovered:

```text
Name              Guest Start  Guest End    XEX/Image Offset  Host/Runtime Mapping  Notes
main_text         0x82A00000   0x82B23456   0x00000000        generated code        executable PPC text
main_rdata        0x82B24000   0x82C00000   0x00124000        guest memory          constants/vtables/strings
main_data         0x82C00000   0x82D00000   0x00200000        guest memory          writable globals
stack             dynamic      dynamic      n/a               guest memory          r1-based stack frames
asset_root        n/a          n/a          n/a               host filesystem       extracted game files
```

Do not assume another Xbox 360 title uses the same image base, region layout, loader behavior, section names, filesystem paths, or asset format.

## PowerPC / Xenon CPU Discipline

Xbox 360 code runs on the Xenon PowerPC CPU and is big-endian from the guest program's perspective.

When analyzing or translating behavior, explicitly consider:

- GPR state and signedness.
- FPR state, rounding, NaN behavior, and conversion instructions.
- VMX/VMX128 vector lane order and saturation behavior.
- CR fields and conditional branches.
- XER carry/overflow behavior.
- LR and CTR flow, especially `bl`, `bctrl`, `mtctr`, `bctr`, tail calls, and virtual dispatch.
- `r1` stack frame layout.
- Nonvolatile register save/restore behavior.
- Guest memory byte order when reading/writing multi-byte values on little-endian Windows.
- Alignment assumptions and unaligned access behavior.
- Atomic/interlocked behavior.
- Compiler-generated thunks, prologues, epilogues, and import stubs.

Do not "simplify" a PPC instruction to C++ unless carry, overflow, condition-code, endian, and vector-lane effects are either modeled or proven irrelevant.

## Xbox 360 Runtime Subsystem Checklist

When an issue touches platform behavior, classify the subsystem before proposing a fix:

- **XEX loader / image layout**: image base, sections, imports, exports, relocations, TLS, entrypoint, and module metadata.
- **Xenon CPU / PPC execution**: control flow, register state, stack, branch targets, instruction semantics.
- **Guest memory**: global data, stack, heap, allocator behavior, endian conversion, address translation.
- **Kernel/API stubs**: thread creation, synchronization, events, timers, files, memory allocation, module imports, title APIs.
- **Filesystem / assets**: path mapping, case sensitivity, loose files, packaged archives, DVD layout, mounted roots.
- **Input**: controller polling, XInput-style host mapping, deadzones, vibration, device status.
- **Audio**: XMA or title-specific audio paths, streaming, buffer queues, timing, sample conversion, host output.
- **GPU / Xenos-facing behavior**: command submission, shaders, textures, render targets, synchronization, and host renderer mapping.
- **Host graphics backend**: D3D12/Vulkan/OpenGL/other backend behavior, device loss, resource lifetime, TDR/DEVICE_HUNG diagnosis.
- **Timing / scheduler**: frame pacing, sleep/spin waits, host thread scheduling, guest timers.
- **Networking / services**: usually stubbed or disabled unless the port explicitly supports them.
- **Save data / user profile**: profile paths, save containers, endian conversion, completion semantics.
- **DLC / title updates**: optional content roots, content IDs, version assumptions.

Important terminology:

- Xbox 360 GPU: **Xenos**.
- Original Xbox GPU: **NV2A**.
- Do not call Xbox 360 GPU work "NV2A" unless discussing original Xbox, not Xbox 360.

Common ReXGlue-port failures are often runtime/hook/mapping issues rather than raw instruction translation bugs: wrong image base, wrong guest-to-host pointer conversion, wrong endian reads, wrong virtual dispatch target, missing import stub, missing asset path, bad filesystem root, skipped GPU synchronization, stale resource lifetime, overbroad hook, or a host graphics backend crash.

## Phase 0: XEX Reconnaissance

Before generating or patching anything, record:

- Game title and region/revision if known.
- Legal source of the user-owned dump, without requesting copyrighted material.
- Original ISO/source path, extracted folder path, or XBLA/STFS/LIVE package path if relevant to local workflow.
- Source content type: disc/ISO dump, extracted loose files, XBLA STFS/LIVE package, GOD container, title update, DLC package, or unknown.
- Extracted `default.xex` path.
- XEX file size.
- SHA-256 hash of the XEX.
- XEX image base, if known.
- Entry point, if known.
- Required extracted game data root.
- Known title update/DLC/XBLA package assumptions, if any.
- Package metadata when applicable: content type, title ID, media ID, package display name, package size, package hash, and package extraction tool/output.
- Whether the current project uses loose files, packed archives, XBLA package extraction, or a mix.
- ReXGlue SDK version/commit, if known.
- Ghidra project/program name and import base, if known.
- Current run status and active blocker.

Useful Windows commands:

```cmd
certutil -hashfile assets\game_files\default.xex SHA256
dir assets\game_files\default.xex
```

Useful PowerShell commands:

```powershell
Get-FileHash .\assets\game_files\default.xex -Algorithm SHA256
Get-Item .\assets\game_files\default.xex | Select-Object FullName,Length,LastWriteTime
```

Do not continue with guessed XEX identity if the hash, path, or executable revision is unclear and the task depends on exact addresses.

## Symbol, Function, and Hook Confidence

Use confidence labels when adding symbols, naming functions, documenting hook points, exporting metadata, or feeding function boundaries into ReXGlue configuration:

```text
Known       backed by ReXGlue output, Ghidra function boundary, clean xref proof, symbol/export/import evidence, or runtime trace
Likely      supported by prologue/control-flow/xref evidence
Tentative   inferred from decompiler output, pattern matching, or heuristic script output only
Unknown     needs raw PPC assembly, Ghidra evidence, runtime trace, or better mapping
```

Never commit tentative function boundaries or hook points as if they are proven. Check raw disassembly, branch targets, register state, nearby data boundaries, image mapping, and runtime behavior first.

Preferred export columns:

```text
Name, Guest Start, Guest End, Size, XEX/Image Offset, Type, Confidence, Evidence
```

## Indirect Control Flow Rules

Indirect control flow is one of the highest-risk areas in Xbox 360 static recompilation.

When you see:

```asm
mtctr rX
bctr
```

or:

```asm
mtctr rX
bctrl
```

classify the source of `rX`:

```text
Source of CTR:
- switch table
- virtual table slot
- function pointer field
- import thunk
- callback array
- tail-call target
- hand-written dispatcher
- unresolved
```

Evidence to collect:

```text
Target address:
Containing function:
Instructions that compute CTR:
Data references feeding CTR:
Possible table base:
Entry size:
Bounds check:
Known targets:
Fallthrough/continuation:
Ghidra switch/vtable result:
ReXGlue config impact:
Confidence:
```

Do not add switch-table or indirect-call configuration from a guess. Verify with GhidraMCP, raw PPC, local helper output, or ReXGlue logs.

## Documentation Ledgers

Column shapes and bootstrap: [references/project-setup.md](references/project-setup.md). Maintain these when the project uses them:

```text
docs/address_ledger.md
docs/asset_ledger.md
docs/xbla_package_ledger.md
docs/function_ledger.md
docs/runtime_boundaries.md
docs/regression_log.md
docs/toolchain.md
docs/compiler_matching.md
docs/ghidra_mcp_setup.md
docs/frontend_agent_rules.md
```

Suggested `address_ledger.md` columns:

```text
Guest Address, Function/Symbol, Type, Evidence, Hook/Config Impact, Status, Notes
```

Suggested `asset_ledger.md` columns:

```text
Guest Path, Host Path, Source Archive/Folder, Required?, Status, Evidence, Notes
```

Suggested `regression_log.md` entry shape:

```text
Date:
Build/commit:
Change:
Evidence before:
Result after:
Regression risk:
Next action:
```

Do not let ledgers become fiction. If evidence is missing, mark TODO with the exact artifact needed.

## When to Ask the User for More Data

Ask for the smallest useful artifact when evidence is missing. Examples:

```text
Please paste the ReXGlue error line plus 30 lines above it.
Please paste the current manifest/config section that defines this hook.
Please paste the generated hook macro/declaration from the ReXGlue output.
Please paste the raw PPC disassembly from 0x82XXXXXX to 0x82XXXXXX.
Please paste the GhidraMCP output for callers/callees of 0x82XXXXXX.
Please paste the runtime log from process start through the first crash.
Please paste the `rexglue --help` or subcommand help output for this command.
Please paste the Ghidra version and installed extension list if XEXLoaderWV or GhidraMCP setup fails.
Please paste the MCP frontend name and config path if automatic MCP configuration cannot locate it.
```

Do not ask for broad project dumps when a narrow artifact would answer the question.

## Removed or Retargeted N64 Concepts

This Xbox 360 skill intentionally excludes N64-specific hardware and toolchain workflows, but preserves the original high-level split between **static recompilation** and **full/matching decompilation**.

Removed as N64-only:

```text
splat64 project generation
N64 ROM byte-order conversion
baserom.z64 workflow
MIPS assembly matching
libultra identification
n64sym
IDO-specific N64 compiler setup
RSP/RDP/VI/AI/PI/SI/PIF/MI/RI hardware checklists
RDRAM/Expansion Pak/save-type workflows
N64Recomp/N64ModernRuntime/RT64 workflow
RSP microcode extraction
ROM/VROM/VRAM overlay tables specific to N64
```

Retargeted to Xbox 360:

```text
decomp.me compiler matching -> Xbox 360 PPC compiler/ABI exploration
N64Recomp static recomp track -> ReXGlue track and XenonRecomp/XenonAnalyse track
ROM/VRAM/RDRAM discipline -> XEX image base, guest virtual address, image offset, guest memory, host pointer discipline
RSP/RDP renderer hazards -> Xenos GPU, shader translation, host renderer, D3D12/Vulkan, resource lifetime hazards
libultra/custom-runtime boundary finding -> Xbox kernel/API/filesystem/audio/input/renderer/runtime boundary classification
```

If the user actually asks for an N64 project, use an N64-specific skill instead of this one.

## Debugging

For failure-mode checklists, layer mapping, TDR/endian/VFS triage, and the stuck loop, read [references/debug-triage.md](references/debug-triage.md). Patch the owning layer, not the symptom.
