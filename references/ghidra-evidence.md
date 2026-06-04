## Ghidra / GhidraMCP Evidence Protocol

Use Ghidra or GhidraMCP as the primary evidence-gathering tool when function boundaries, XREFs, vtables, imports, jump targets, switch tables, crash addresses, or guest data references are unclear.

Do not trust Ghidra decompiler output by itself. Use raw PPC disassembly, xrefs, branch targets, control-flow context, data references, and runtime traces for final decisions.

When Ghidra evidence is needed, use narrow requests:

```text
GhidraMCP Evidence Needed:
Target:
Tool(s):
Expected evidence:
Why it matters:
Fallback if MCP unavailable:
```

Prefer evidence such as:

- Program name and image base.
- Containing function name, start address, and size.
- Raw PPC disassembly around the target.
- Decompiled pseudocode for structure only.
- Callers, callees, XREFs, labels, and comments.
- Raw bytes around an address when needed.
- Import/export references.
- Vtable and function-pointer references.
- Switch/jump-table candidates.
- Struct field offsets inferred by Ghidra.
- Crash-address context.
- Memory map entries.

Useful targets include boot/init code, kernel import wrappers, allocator calls, thread/event/timer functions, file I/O, asset loaders, audio submission, render submission, input polling, virtual dispatch sites, and functions touching suspected guest/host boundaries.

Common Ghidra issues in Xbox 360 PPC work:

```text
Wrong image base
Wrong function boundaries
Bad signedness
Bad endian interpretation
Fake structs
Missed CR/XER semantics
VMX/VMX128 decompiler confusion
Code/data confusion
Jump tables mistaken for pointers
Vtables mistaken for ordinary data
Import thunks misidentified as user code
bctr/bctrl targets left unresolved
```


## Fallback Analysis Without GhidraMCP

If GhidraMCP is unavailable, disconnected, too heavy for the current step, or not yet worth setting up, use narrower offline evidence tools.

Use these fallbacks as appropriate:

- Existing project helper: `scripts/xref_ppc_in_xex.py` for read-only PPC xref/disassembly/heuristic `bctr` scans.
- `capstone`: scripted PPC disassembly from XEX ranges or extracted binary chunks.
- `lief`, `construct`, or project-local parsers: XEX metadata and binary structure inspection when supported by current scripts.
- `rg`, `xxd`, PowerShell, and small Python scripts: fast searches for byte patterns, strings, paths, constants, and logs.
- Compiler/linker output: generated symbol names, object lists, and source locations.
- Runtime logs: host crash, guest PC, register dumps, hook traces, resource lifetime errors.

Recommended fallback output shape:

```text
Target:
Tool used:
Raw PPC disassembly:
Function start/end if known:
Callers/callees or XREF substitute:
Branch targets:
Import/data/vtable references:
Guest image mapping:
Confidence:
What still needs Ghidra or runtime trace:
```

Do not treat heuristic script results as stronger evidence than GhidraMCP/user-provided Ghidra output or ReXGlue output.
