# Decisional Brain — Track selection & evidence

> Load when the goal is ambiguous, tracks compete, or you need reasoning before a large change.

## Track router

| User signals | Track | Open |
|--------------|-------|------|
| ReXGlue, reNut, TiP-Recomp, codegen from XEX | **A** | `03-track-rexglue.md` |
| XenonRecomp, XenonAnalyse, XenosRecomp shaders+CPU | **B** | `04-track-xenon.md` |
| matching, decomp.me, handwritten C++, full decomp | **C** | `05-track-full-decomp.md` |
| 360toolsUpdated, extract_stfs, rexglue init, simpsonsarcade layout | **D** | `06-track-360tools.md` |
| Legacy extract_pe, post_codegen, XenonRecomp patches | **D legacy** | `06-track-360tools.md` § Legacy only |

**PC port** without naming a tool → explain A–D; new XBLA → prefer **D** (360toolsUpdated) or **A**.

### Inference keywords (when track not stated)

```text
Track A: ReXGlue, reNut, TiP-Recomp, reDAHM, The Outfit ReXGlue project
Track B: XenonRecomp, XenonAnalyse, Unleashed Recompiled, Sonic Unleashed, XenosRecomp
Track C: matching, full decomp, source recreation, decomp.me, compiler matching, clean C/C++ source
Track D: 360toolsUpdated, 360tools, extract_stfs, rexglue init, templates/advanced, simpsonsarcade/vig8/gh2/ctxbla
Legacy D: sp00nznet tree — extract_pe, XenonRecomp patches, post_codegen (confirm clone first)
```

### Boundaries

- Tracks **A** and **D** (current) both use ReXGlue codegen/runtime; **D** adds extraction scripts, `templates/advanced/`, SDK `patches/`.
- Track D (current) = ReXGlue CPU+runtime; **no** XenonRecomp between extract and codegen.
- ReXGlue and XenonRecomp are not sequential stages unless legacy 360tools or explicit Track B.
- **XenonAnalyse** may assist XenonRecomp by producing/validating metadata — do not claim outputs without local evidence.
- Do not mix ReXGlue and XenonRecomp generated output, hooks, or config without proven compatibility.
- **XenosRecomp** = shaders/renderer (often with Track B), not CPU recomp.
- Full decomp ≠ static recomp — different success criteria.
- Findings from one track may inform another as **research**; never import generated code or hooks across stacks without compatibility proof.

## Evidence priority

1. Local project: generated, config, hooks, scripts, logs, ledgers, checked-out SDK/tool source
2. Tool output: ReXGlue, XenonRecomp, XenonAnalyse, XenosRecomp, CMake, compiler/linker, runtime logs, GhidraMCP, decomp.me
3. User artifacts: XEX hash, package metadata, PPC disasm, Ghidra exports, crash logs, manifest/config, hook files, maps, build output
4. Official docs/wiki for active tool versions
5. Public ports — **research only** (`18-project-templates.md`)
6. Labeled inference

Separate: **proves** / **source says** / **inferred** / **unknown**.

## Ask packet (smallest artifact)

```text
ReXGlue error + 30 lines above
Config section for this hook
Generated hook macro/declaration from ReXGlue output
Raw PPC 0x82XXXXXX–0x82XXXXXX
GhidraMCP xrefs at guest PC
Runtime log start → first crash
rexglue --help for subcommand in question
Ghidra version + installed extension list (if XEXLoaderWV/GhidraMCP setup fails)
MCP frontend name + config path (if auto MCP config cannot locate it)
```

No broad dumps.

## Related skills

| Skill | When |
|-------|------|
| **360tools** | Extract-only, no Ghidra/stuck methodology |
| **xboxrecomp** | OG Xbox `.xbe` |
| **windows-game-matching-decomp** | Win PE, no 360 context |

## N64 exclusion

If the user wants an **N64** project → use an N64-specific skill, not this one.

Removed as N64-only (do not apply to Xbox 360):

```text
splat64, baserom.z64, MIPS matching, libultra, n64sym, IDO N64 compiler setup
RSP/RDP/VI/AI/PI/SI/PIF/MI/RI hardware checklists, RDRAM/Expansion Pak workflows
N64Recomp, N64ModernRuntime, RT64, RSP microcode extraction, N64 overlay tables
```

Retargeted Xbox 360 analogues:

```text
decomp.me matching        → PPC compiler/ABI exploration (Track C)
N64Recomp static recomp   → ReXGlue (A) / XenonRecomp (B)
ROM/VRAM/RDRAM discipline → XEX base, guest VA, image offset, guest memory, host pointers
RSP/RDP renderer hazards  → Xenos GPU, shader translation, host renderer, D3D12/Vulkan, TDR
libultra/runtime boundary → kernel/API/filesystem/audio/input/renderer stubs (16-runtime-hooks.md)
```
