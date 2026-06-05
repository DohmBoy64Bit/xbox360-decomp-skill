## Stuck on the same blocker

If progress stalls — same crash twice, no new evidence, unregistered guest VA, GPU draw/present failures, boot not advancing — do **not** retry the same local-only patch. Gather external evidence first, then re-check local game artifacts. The parent skill’s [13-debug-triage.md](13-debug-triage.md) maps symptoms to layers; this file is the mandatory research order before the next patch.

Why this order matters: other ReXGlue/XBLA ports and the active SDK often already solved the same symptom (VFS, hooks, PM4, shader dump, present). Repeating a guess wastes time and can desync generated code from config.

### 1. Cross-recomp research (project-local paths first)

Use whatever exists in the current repo; do not invent paths.

| Artifact | Typical location | What to search for |
|----------|------------------|-------------------|
| Port-specific ReXGlue notes | `docs/reference_*_rexglue.md`, `docs/reference_daytona_rexglue.md` | Hook patterns, known blockers, image base, asset roots |
| Repomix / reference dumps | `docs/repomix/repomix-*-recomp-*.xml` | Exact log tokens: `ShaderDump`, `VFS`, `REX_HOOK_RAW`, `present`, `PM4`, guest addresses |
| Active SDK | `thirdparty/rexglue-sdk`, `REXSDK`, `docs/rexwiki/` | Macro/API semantics, config schema, hook registration |
| Public layout reference | See [18-project-templates.md](18-project-templates.md) | Filesystem and XBLA extraction patterns only — verify locally |

Grep repomix and SDK for the **exact** symptom string from the log, not a paraphrase.

### 2. Web search

Query the exact guest address, log line, or ReXGlue/Xenia error. Include title context (e.g. Xbox 360 XBLA static recompilation) when the port is XBLA.

### 3. Local game evidence (after external research)

Only then:

- Ghidra / GhidraMCP at the guest PC (see [08-ghidra-evidence.md](08-ghidra-evidence.md))
- Original or extracted game analysis tree (project `code/`, dumps, or disassembly exports — whatever this repo documents)
- `generated/` output and handwritten hooks in `src/`

### Reporting fixes

Every change after a stuck loop must cite what drove it:

```text
Source: repomix:<file> | SDK:<path>:<line> | web:<query/URL> | Ghidra:<addr>
Hypothesis:
Narrow change:
Verification:
```

If the workspace has `.cursor/rules/stuck-use-external-recomps.mdc` (or equivalent), treat it as mandatory for that project — this reference is the portable skill version.
