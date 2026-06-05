## XenonRecomp / XenonAnalyse Track

Use this track when the user selects XenonRecomp, XenonAnalyse, Unleashed-style workflow, or asks for the non-ReXGlue Xbox 360 recompilation path.

Boundaries:

- XenonRecomp converts Xbox 360 executable code to C++ but does not by itself provide the runtime needed to make a game function.
- XenonAnalyse is an analysis/metadata helper in this ecosystem; verify its exact commands, outputs, and config fields from local source/docs/tool output.
- XenosRecomp is shader-focused. Use it for Xenos shader binary to HLSL/DXIL/SPIR-V work, not CPU recompilation.
- Do not mix ReXGlue-generated code with XenonRecomp-generated code unless the user explicitly asks for a migration or research comparison.

Recommended Xenon track phases:

```text
Phase X0: Inputs and environment
- Lawful game dump and extracted default.xex.
- XEX hash, image base, title update/DLC assumptions.
- Tool versions/commits for XenonRecomp, XenonAnalyse, XenosRecomp, CMake, compiler, and Ghidra.

Phase X1: Analysis metadata
- Import XEX into Ghidra with correct image base.
- Use GhidraMCP/XenonAnalyse/local scripts to confirm functions, labels, imports, switch tables, vtables, and indirect branches.
- Keep function boundaries and switch-table data evidence-backed.

Phase X2: CPU recompilation
- Run only verified XenonRecomp commands.
- Treat generated C++ as machine output with explicit CPU state, guest-memory base, and endian-aware load/store behavior.
- Handle missing instructions as blockers or verified implementations, not guesses.

Phase X3: Runtime layer
- Provide kernel/API stubs, filesystem/content roots, allocator/memory behavior, threading/timing, input, save/profile, audio, and renderer interfaces.
- Do not expect generated CPU C++ to run correctly without this layer.

Phase X4: Shader/renderer path
- Use XenosRecomp or project-specific shader translation only when shader binaries and renderer integration require it.
- Verify shader container assumptions, constant reflection, vertex declarations, texture formats, endian conversion, and backend API requirements.

Phase X5: Build and debug
- Map crashes back to guest address, generated C++ location, runtime layer, or shader/renderer path.
- Record changes in ledgers and maintain A/B tests for hooks and replacements.
```

Recommended Xenon evidence packet:

```text
Tool:
Version/commit:
Command/output:
Target XEX:
Image base:
Generated function/file:
Guest address:
Runtime subsystem:
Evidence:
Next blocker:
```
