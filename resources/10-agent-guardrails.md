# Agent Guardrails & Self-Correction — Xbox 360 / XBLA

> Load when you repeat a failure, hit the stuck loop (same crash twice), or need crash/build triage before patching.

## 0. Core discipline (all sessions)

- **Stuck loop:** same failure without new proof → `12-stuck-cross-recomp.md` first. Honor project `.cursor/rules/*` stuck rules when present (`20-frontend-rules.md`).
- **Evidence:** prefer user/local artifacts — XEX hash, package metadata, logs, raw PPC, Ghidra exports, config/hooks, ledgers, build output — before inference.
- **Edits:** every config change, hook, shim, or build patch states *why*, which guest/host boundary it touches, which registers or memory ranges it affects, and how to verify.
- **Separation:** keep generated output, handwritten hooks/runtime, extracted assets, and docs in distinct paths; do not commit proprietary game content.
- **Environment:** audit installed tools first (`01-dev-environment.md`); install only missing pieces; backup and **merge** MCP/frontend configs — preserve the user's existing style.
- **Unknowns:** state what is unverified and name the smallest artifact that would prove it (`22-decisional-brain.md` § Ask packet).

## 1. Agent mistake taxonomy

| Your mistake | Prevention |
|-------------|------------|
| Patching `generated/` as the first fix | Use config/TOML, hooks, runtime glue, or regen — see §3 Fix Taxonomy |
| Inventing ReXGlue/Xenon API or hook macro names | Quote **SDK file:line** from local `thirdparty/rexglue-sdk` / `REXSDK` |
| Guest VA cast to host pointer | Guest memory layer or documented translation only |
| Recommending legacy XenonRecomp steps on **360toolsUpdated** fork | Read `06-track-360tools.md` — current Track D is ReXGlue-native |
| Blind-overwriting `.cursor/mcp.json` | Backup + merge; see `07-ghidra-mcp.md` |
| Same patch twice without new evidence | **STOP** — load `12-stuck-cross-recomp.md` |
| Confident without build/log proof | Run command, read full output, verify exit code |
| Treating decompiler output as PPC truth | `09-original-game-evidence.md` — raw PPC / Ghidra first |

## 2. Fix taxonomy — four tools (no fifth)

| # | Tool | When | Files |
|---|------|------|-------|
| 1 | **Config / TOML / manifest** | Function boundaries, switches, hooks metadata | `*_config.toml`, manifest, ReXGlue project config |
| 2 | **Runtime / hooks / stubs** | Kernel, GPU, audio, VFS, timing | `src/`, `hooks/`, `templates/advanced/stubs.cpp` |
| 3 | **Handwritten shim** | Host boundary only — compare PPC first | Project `src/`, not `generated/` |
| 4 | **Regenerate** | After config proof | `rexglue codegen`, XenonRecomp (Track B only), CMake |

**Never** hand-edit massive `generated/*.cpp` as the primary fix.

## 3. Crash decision flowchart

```
Crash / unregistered VA / TDR / hang
  → Same symptom twice? → YES → 12-stuck-cross-recomp.md (mandatory)
  → Classify layer → 13-debug-triage.md subsystem map
  → Guest PC known? → GhidraMCP / raw PPC at PC (08-ghidra-evidence.md)
  → Indirect dispatch? → mtctr/bctrl evidence → 23-xenon-execution-discipline.md
  → Pick fix tool (§2) → cite evidence packet → verify with log/build
```

## 4. Evidence packet (required for every fix)

```text
Source: SDK:<path>:<line> | Ghidra:<addr> | repomix:<file> | web:<url>
Hypothesis:
Narrow change:
Verification:
```

## 5. Related deep references

| Topic | File |
|-------|------|
| Stuck loop order | `12-stuck-cross-recomp.md` |
| Failure modes + layers | `13-debug-triage.md` |
| Guest truth / PPC | `09-original-game-evidence.md` |
| Hook placement | `16-runtime-hooks.md` |
| Config safety | `15-rexglue-config.md` |
