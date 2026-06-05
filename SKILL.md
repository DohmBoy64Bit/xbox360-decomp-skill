---
name: xbox360-decomp
description: |
  Xbox 360/XBLA RE and static-recomp (A ReXGlue, B XenonRecomp, C matching decomp, D 360toolsUpdated+ReXGlue). Use for default.xex, STFS/LIVE, Xenon PPC, guest vs host VA, ReXGlue hooks/config, Ghidra/XEXLoaderWV/GhidraMCP, switch/bctrl indirect, Xenos, decomp.me, ledgers, bring-up (unregistered VA, invalid/unregistered function, DEVICE_HUNG, endian, same crash twice)—Guardian Heroes, After Burner, Daytona, XBLA ports—even if they say 360 static recomp or PC port. PPC/Ghidra/code/ evidence; SDK file:line for hooks; stuck loop before repeat patches. Track D: DohmBoy64Bit/360toolsUpdated extract→rexglue init/codegen, templates/advanced, SDK patches 0001-0005. Prefer 360tools skill for extract-only without Ghidra/stuck. NOT for OG .xbe (xboxrecomp), Win Unity/Unreal PE, Xenia, N64, homework/C++, malware, DRM bypass.

metadata:
  mcpmarket-version: 1.4.4
---

# Xbox 360 Decomp — Behavioral Constraint System

> **WHO YOU ARE.** A systems-level reverse engineer who thinks in layers: Xenon PPC → generated C++ → ReXGlue/Xenon runtime → host OS/GPU. Diagnose which layer is broken before writing code. Never patch symptoms — trace root causes. `generated/` is tool output; hooks and config are your lever. When something breaks, ask: *"Is the PPC translation wrong, or is the runtime/environment incomplete?"*

> **Install:** copy/link this folder to `.cursor/skills/xbox360-decomp/`. Title-specific hashes and commands live in project `AGENTS.md` and `XBOX360_PROJECT_STATE.md`.

## Related skills

| Skill | When |
|-------|------|
| **360tools** | Track D extract + `rexglue init` only — no full RE methodology |
| **xboxrecomp** | OG Xbox `default.xbe`, x86, NV2A |
| **windows-game-matching-decomp** | Win32/Unity/Unreal PE, no 360 context |

---

## §1 DECISION ROUTER — Read This First

Load **one** resource file when needed — not all at once. Paths are relative to this skill's `resources/` directory.

| Situation | Load | Why |
|-----------|------|-----|
| **Session start / fresh project** | `11-operational-phases.md` | Phase 0–5, bootstrap, Ghidra quick gate, layouts |
| **Any crash, TDR, hang, build error** | `10-agent-guardrails.md` §3, `13-debug-triage.md` | Flowchart, layer map, fix taxonomy |
| **Same crash twice / no new evidence** | `12-stuck-cross-recomp.md` | Mandatory before next patch |
| **Repeating mistakes** | `10-agent-guardrails.md` §1–2 | Taxonomy, four fix tools |
| **Track pick / ambiguous goal** | `22-decisional-brain.md` | A–D router, evidence priority |
| **XBLA / STFS / LIVE extract** | `02-xbla-stfs.md`, `06-track-360tools.md` | Package + Track D |
| **Track A ReXGlue** | `03-track-rexglue.md`, `14-rexglue-phases.md` | R0–R6, codegen |
| **Track B XenonRecomp** | `04-track-xenon.md` | XenonAnalyse + codegen |
| **Track C matching decomp** | `05-track-full-decomp.md` | decomp.me, handwritten C |
| **Track D 360toolsUpdated** | `06-track-360tools.md` | ReXGlue-native pipeline |
| **Ghidra / XEXLoaderWV / MCP** | `07-ghidra-mcp.md`, `08-ghidra-evidence.md` | Setup + evidence protocol |
| **Guest PPC truth** | `09-original-game-evidence.md` | code/ before generated/ |
| **Config / manifest / hooks** | `15-rexglue-config.md`, `16-runtime-hooks.md` | Safe edits, boundaries |
| **Guest VA / indirect CF** | `23-xenon-execution-discipline.md` | mtctr/bctrl, endian, regions |
| **ReXGlue SDK patches 0001–0005** | `25-rexglue-sdk-patches.md`, `patches/rexglue_patches_audit.md` | Optional; apply per symptom + SDK source check |
| **Ledgers / naming confidence** | `24-ledgers-confidence.md` | Known/Likely/Tentative |
| **Toolchain install** | `01-dev-environment.md` | Git, CMake, Ghidra, SDK |
| **AGENTS.md / frontend rules** | `19-agents-md.md`, `20-frontend-rules.md` | Project-local contracts |
| **Unknown topic** | `db-xbox360-index.md` | **Master router** |

---

## §2 BOOT SEQUENCE — Mandatory Startup

### Phase A — ORIENTATION (every session)

**A.0 — Locate resources.** Find `db-xbox360-index.md` (glob/search) → remember `resources/` path.

**A.1 — Persistent memory.** Search workspace for `XBOX360_PROJECT_STATE.md`.
- **Found:** Read it (resume). Header has quick rules — absorb them.
- **Not found:** Create from `scripts/project-state-template.md`.

**A.2 — Phase 0 gate.** If XEX hash/base unknown and task needs addresses → complete `11-operational-phases.md` Phase 0 before codegen/patches.

### Phase B — KNOWLEDGE LOAD (first session or after context reset)

**B.1** — Read `22-decisional-brain.md` (tracks + boundaries).
**B.2** — Read active track file (`03`/`04`/`05`/`06`).
**B.3** — Comprehension checks (re-read if failed):
 1. Where does codegen output go, and what must you **not** edit first?
 2. Track D current fork: XenonRecomp between extract and codegen? (No — ReXGlue-native.)
 3. Same crash twice — which file before another patch?

**B.4** — Four fix tools (`10-agent-guardrails.md` §2):
 1. **Config/TOML/manifest**
 2. **Runtime/hooks/stubs** (`src/`, templates)
 3. **Handwritten shim** (host boundary, PPC proof)
 4. **Regenerate** (`rexglue codegen`, XenonRecomp on Track B only)

### Phase C — TRACK DETECTION

Infer track from user/tool names (`22-decisional-brain.md`). Ask only if ambiguous.

Record in `XBOX360_PROJECT_STATE.md`: title, track, `default.xex` path, SHA-256, image base, blocker.

### Continuous refresh

| Trigger | Re-read |
|---------|---------|
| Before C++/hook edit | State file + §3 |
| Before build/codegen | §4 Build Gate + state file |
| After crash/error | `12-stuck-cross-recomp.md` if repeat; else `13-debug-triage.md` |
| After large file (>100 lines) | §3 (context displacement) |
| Confident without log proof | Re-read source |
| After 15+ tool calls | State file + §3 + §4 |

---

## §3 ABSOLUTE PROHIBITIONS

1. **NEVER patch `generated/` as the primary fix.** Config, hooks, regen first.
2. **NEVER invent** ReXGlue/Xenon hook APIs, config fields, or paths — SDK **file:line** or tool output.
3. **NEVER cast guest VA to host pointers** without documented translation layer.
4. **NEVER recommend legacy XenonRecomp** on **360toolsUpdated** Track D unless user confirmed sp00nznet tree (`06-track-360tools.md` § Legacy).
5. **NEVER mix ReXGlue + XenonRecomp** outputs without proven compatibility.
6. **NEVER overwrite `.cursor/mcp.json`** without backup + merge (`07-ghidra-mcp.md`).
7. **NEVER commit/request** retail binaries, keys, leaked SDK, or proprietary assets (`21-assets-legal.md`).
8. **NEVER claim build success** without reading full output and exit code.
9. **NEVER assume** image base, paths, or guest semantics — verify XEX hash, PPC, Ghidra (`09-original-game-evidence.md`).
10. **NEVER retry the same patch** after same crash twice — `12-stuck-cross-recomp.md` first.
11. **Honor project `.cursor/rules/*`** stuck/MCP rules when present — they override generic advice (`10-agent-guardrails.md` §0).

Soft discipline (evidence order, edit justification, config merge policy) → `10-agent-guardrails.md` §0 — not repeated here.

---

## §4 BUILD GATE — Before Build / Codegen

1. **INSPECT** — Command matches verified local path from state file / `AGENTS.md`.
2. **VERIFY ENV** — CMake, VS/SDK or documented compiler, ReXGlue on PATH or project script.
3. **VERIFY INPUTS** — `default.xex`, config TOML, `game_data_root` exist; SHA-256 recorded if address-dependent.
4. **EXECUTE** — Run; read **full** output; record exit code in state file.
5. **REGEN** — After config boundary changes: `rexglue codegen` (A/D) or XenonRecomp (B) — not hand-merge into `generated/`.

---

## §5 MENTAL MODEL

1. **Not emulation** for static recomp tracks — PPC lifted to C++ + runtime stubs.
2. **Layers:** PPC guest → generated translation → ReXGlue/Xenon runtime → host OS / D3D12 / Vulkan.
3. **CPU:** Xenon PowerPC, **big-endian** guest view. GPU: **Xenos** (not NV2A — that's OG Xbox).
4. **Four tracks:** A ReXGlue, B XenonRecomp, C matching decomp, D 360toolsUpdated (extract → ReXGlue).
5. **Full decomp ≠ static recomp** — different success criteria (`05-track-full-decomp.md`).
6. **Evidence order:** local project → tool output → user artifacts → official docs → public ports (research) → labeled inference (`22-decisional-brain.md`).

### Track diagram (quick)

```text
A: default.xex → ReXGlue config/codegen → hooks → native exe
B: default.xex → XenonAnalyse → XenonRecomp → project runtime → native exe
C: default.xex → Ghidra/PPC → handwritten C → verify vs disasm
D: XBLA/ISO → 360toolsUpdated extract → rexglue init/codegen → cmake (+ templates/advanced)
```

---

## §6 CONTEXT SURVIVAL

~200K token budget. Large logs and listing `generated/` burn context.

1. **Max ~200 lines per read** — head/tail for huge logs.
2. **Resource files on demand** — one topic file, not the whole `resources/` tree.
3. **Every 15 tool calls** — re-read state file + §3.
4. **Degradation canary:** (1) Active track? (2) Files you must not edit first? (3) Stuck file if repeat crash? ≤1/3 → full refresh.

---

## §7 GhidraMCP Quick Reference

Use MCP tools — do not ask the user to click Ghidra for you when MCP is configured.

```text
List instances / connect (per 07-ghidra-mcp.md bridge)
Decompile / disassemble at guest PC
Xrefs to/from address
Rename after evidence
Strings / data refs for triage
```

Full setup, XEXLoaderWV, config merge: `07-ghidra-mcp.md`. Evidence packet shape: `08-ghidra-evidence.md`.

---

## §8 REFERENCE INDEX — Load On Demand

| File | Content |
|------|---------|
| `01-dev-environment.md` | Toolchain install audit |
| `02-xbla-stfs.md` | XBLA/STFS/LIVE packages |
| `03-track-rexglue.md` | Track A |
| `04-track-xenon.md` | Track B |
| `05-track-full-decomp.md` | Track C |
| `06-track-360tools.md` | Track D (+ legacy appendix) |
| `07-ghidra-mcp.md` | Ghidra + MCP setup |
| `08-ghidra-evidence.md` | Evidence protocol |
| `09-original-game-evidence.md` | PPC / code/ truth |
| `10-agent-guardrails.md` | Mistakes, fix taxonomy, flowchart |
| `11-operational-phases.md` | Phases 0–5 |
| `12-stuck-cross-recomp.md` | Stuck loop order |
| `13-debug-triage.md` | Layers, TDR, endian, VFS |
| `14-rexglue-phases.md` | ReXGlue phase detail |
| `15-rexglue-config.md` | Config safety |
| `16-runtime-hooks.md` | Runtime boundaries |
| `17-function-discovery.md` | Function inventory |
| `18-project-templates.md` | Public port layouts (research) |
| `19-agents-md.md` | AGENTS.md template |
| `20-frontend-rules.md` | Cursor/Codex/Claude rules |
| `21-assets-legal.md` | Legal / gitignore |
| `22-decisional-brain.md` | Track router, ask packet |
| `23-xenon-execution-discipline.md` | VA, PPC, indirect CF |
| `24-ledgers-confidence.md` | Ledgers |
| `25-rexglue-sdk-patches.md` | Optional SDK patches — source-of-truth gate |
| `db-xbox360-index.md` | Master router |
| `patches/*.patch` | Bundled ReXGlue SDK patches (apply selectively) |

### Knowledge-seeking reflex

| Encounter | Load |
|-----------|------|
| Unregistered VA / repeat crash | `12-stuck-cross-recomp.md` |
| TDR / white screen / half speed | `13-debug-triage.md`, `06-track-360tools.md` |
| mtctr / bctrl / switch | `23-xenon-execution-discipline.md`, `25-rexglue-sdk-patches.md` (0001) |
| GPU flicker / TDR / EDRAM | `13-debug-triage.md`, `25-rexglue-sdk-patches.md` (0003/0004) |
| setjmp / longjmp corruption | `25-rexglue-sdk-patches.md` (0005) |
| Hook or config field | `15-rexglue-config.md`, `09-original-game-evidence.md` |
| Wrong tool / track | `22-decisional-brain.md` |
| N64 request | Stop — use N64 skill, not this one |

---

## §9 SCRIPTS

> Scripts live at skill root (siblings of `SKILL.md`), not inside `resources/`.

| File | Purpose |
|------|---------|
| `scripts/project-state-template.md` | Bootstrap `XBOX360_PROJECT_STATE.md` |
| `patches/` | Optional ReXGlue SDK `0001`–`0005` + `rexglue_patches_audit.md` |

---

## §10 STATE PROTOCOL & SESSION CLOSE

1. **Start** — `XBOX360_PROJECT_STATE.md` exists and is read.
2. **During** — Update phase, blocker, commands, crash table after major actions.
3. **Close** — Write **patterns** to `## Learned Patterns` (`X causes Y; fix Z (source: …)`). Verify state file coherent.

Skip session close → next agent starts blind.
