## Matching / Full Decompilation Track

Use this track when the user wants readable source recovery, matching functions, compiler identification, long-term source reconstruction, or asks for the Xbox 360 equivalent of the original N64 decomp/full decomp path.

Do not promise a clean matching build early. Xbox 360 titles commonly use proprietary Microsoft/Xbox 360-era compiler toolchains, linkers, SDK libraries, profile-guided optimizations, LTCG-like behavior, custom build flags, and VMX/VMX128 codegen. Treat matching as an evidence-driven research track.

### Full Decomp Track Goals

```text
Goal A: Behavioral source recovery
- Produce readable C/C++ that preserves behavior and can replace functions/modules in a port.
- Exact binary matching is nice but not required.

Goal B: Matching decompilation
- Recover C/C++ and build settings that reproduce PPC assembly for selected functions or modules.
- Requires compiler/toolchain identification, ABI proof, data layout proof, and repeated asm comparison.

Goal C: Hybrid source-port support
- Use decompiled functions as handwritten replacements for generated recomp code or runtime shims.
- Verify each replacement through traces, tests, or side-by-side guest behavior.
```

### Compiler and ABI Reconnaissance

Record evidence before using decomp.me or writing source:

```text
XEX hash and image base:
Containing function:
Function start/end:
Raw PPC disassembly:
Callers/callees:
Imports/SDK calls:
Stack frame size:
Saved nonvolatile registers:
CR/XER behavior:
FPR/VMX usage:
Small-data/global access pattern:
RTTI/vtable/string references:
Candidate compiler:
Confidence:
```

Xbox 360 compiler matching should focus on:

- PPC calling convention and stack frame layout.
- `r1` stack discipline and backchain behavior.
- Nonvolatile register save/restore order.
- TOC/global access patterns if present.
- CR field usage and compare/branch idioms.
- `rlwinm`, `clrlwi`, `extsw`, `cntlzw`, rotate/mask idioms.
- Floating-point conversion/rounding.
- VMX/VMX128 instruction selection and lane order.
- Inlining and tail-call behavior.
- Switch lowering and jump-table format.
- Import thunk or SDK wrapper patterns.

### Using decomp.me for Xbox 360 PPC

Use decomp.me as a compiler-exploration and function-matching aid, not as proof by itself.

1. Pick a small, linear function with:
   - Clear function boundaries.
   - Few branches.
   - No unresolved `bctr`/`bctrl`.
   - Minimal VMX/VMX128.
   - Known or stubbed callees.
   - Simple stack frame.
2. Export raw PPC assembly from Ghidra/GhidraMCP or the selected analysis tool.
3. Create a minimal C/C++ scratch with:
   - Fixed-width typedefs: `u8/u16/u32/u64/s8/s16/s32/s64`.
   - Explicit big-endian boundary helpers only when modeling memory access, not ordinary local arithmetic.
   - Function prototypes for callees.
   - Struct placeholders with named offsets.
   - Volatile or noalias annotations only when evidence supports them.
4. Try relevant PowerPC/Xbox 360/compiler presets if decomp.me provides them. If no exact Xbox 360 preset exists, label the attempt as exploratory and use local compiler output as stronger evidence.
5. Compare instruction-by-instruction:
   - Prologue/epilogue.
   - Register allocation.
   - Stack offsets.
   - Branch shape.
   - Constant materialization.
   - CR/XER side effects.
   - Floating/vector instructions.
6. Record every attempt in `docs/compiler_matching.md` or equivalent.

Recommended decomp.me packet:

```text
Target:
Guest address:
Function size:
Source of assembly:
Known callees/signatures:
Known globals/structs:
Candidate compiler/preset:
Flags tried:
Match result:
Remaining mismatches:
Next hypothesis:
```

Do not use decomp.me for large functions first. Do not force C to match by introducing undefined behavior, wrong signedness, or fake volatile accesses unless the PPC evidence requires it.

### Converting Decompiled Functions to Source

When moving a function from pseudocode/scratch into the project:

1. Keep the raw PPC assembly and Ghidra pseudocode in notes.
2. Define all structs by offset evidence, not by wishful names.
3. Use explicit endian helpers only at guest-memory boundaries.
4. Preserve signedness and overflow assumptions.
5. Add tests, runtime traces, or A/B hook checks when possible.
6. Build the function in isolation before replacing generated/recompiled behavior.
7. Record status:

```text
Status:
- raw_pseudocode
- compiles
- behavior_checked
- asm_close
- asm_matching
- integrated
- retired
```

### Full Decomp vs Recomp Decision

Choose full decomp when:

- The user wants understandable source.
- A generated function is too hard to debug and a handwritten replacement is more practical.
- A subsystem is better represented as native code than generated PPC.
- The project has long-term source-port goals.

Choose recomp when:

- The priority is booting/running quickly.
- The codebase is too large for manual source recovery.
- The function is ordinary game logic that generated code already handles.
- Runtime/platform boundaries, not source readability, are the blocker.

The best practical workflow is often hybrid: use static recompilation for broad coverage, then full-decompile or rewrite the functions/subsystems where correctness, maintainability, or renderer/runtime integration demands it.
