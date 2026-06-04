## Original game evidence (do not assume)

Static recompilation mistakes often come from naming or behavior invented without checking the retail binary. Treat **guest truth** as: Ghidra at the correct image base, raw PPC, and any project-local original-analysis tree — not decompiler guesses alone.

### Evidence order for “what does the game do?”

1. **Raw PPC** at the guest address (Ghidra listing or export).
2. **GhidraMCP** function boundary, xrefs, callers/callees, switch/vtable targets (see [ghidra-evidence.md](ghidra-evidence.md)).
3. **Project original-analysis tree** when present — commonly named `code/`, `disasm/`, `asm/`, or documented in root `AGENTS.md`. Read the path from project docs; do not assume a folder exists.
4. **Generated recomp output** (`generated/` or project-equivalent) — shows what the toolchain emitted, not necessarily what PPC intended.
5. **Handwritten hooks** (`src/`, `hooks/`) — host boundary; compare against PPC before changing guest semantics.

### When recommending a hook, config field, or patch

State explicitly:

```text
Guest PC:
PPC evidence (instruction or table):
Config/hook field (from local TOML/manifest, quoted):
SDK proof (file:line under thirdparty/rexglue-sdk or active REXSDK):
Why host layer needs change:
How to verify:
```

If any row is missing, label it **Unknown** and name the smallest artifact that would fill it (disassembly range, MCP xref dump, config section).

### Common failure mode

Patching `generated/` or guessing API behavior from another title’s port without SDK line proof and without PPC at the crash PC. That violates project evidence rules and usually reproduces the same crash.
