<!-- ⛔ Re-read QUICK RULES every session — mirrored in SKILL.md §3 -->

# ⛔ QUICK RULES

1. **Generated code:** Do not patch `generated/` first — config, hooks, regen (`10-agent-guardrails.md`).
2. **Evidence:** SDK **file:line** for hooks; PPC/Ghidra before guest semantics (`09-original-game-evidence.md`).
3. **Stuck:** Same crash twice → `12-stuck-cross-recomp.md` before another patch.
4. **MCP config:** Backup `.cursor/mcp.json` before merge (`07-ghidra-mcp.md`).
5. **Track D:** 360toolsUpdated = ReXGlue-native — no `extract_pe`/XenonRecomp unless legacy tree (`06-track-360tools.md`).

---

# Xbox 360 Project State

> Auto-maintained by agent. Read at session start; update after major actions.

## Boot status

- [ ] Located skill `resources/` directory
- [ ] Read `11-operational-phases.md` Phase 0 checklist
- [ ] Track selected: A / B / C / D

## Game info

- **Title**:
- **Region / package type**: XBLA / disc / LIVE / other
- **Title ID** (if XBLA):

## Workspace paths

- **Project root**:
- **default.xex**:
- **game_data_root**:
- **ReXGlue SDK** (`thirdparty/rexglue-sdk` / `REXSDK`):
- **SDK commit**:
- **Patches applied** (0001–0005 / none — see `25-rexglue-sdk-patches.md`):
- **360toolsUpdated clone** (Track D):
- **Ghidra install**:

## XEX identity

- **SHA-256**:
- **Image base**:
- **Entry point**:
- **Size**:

## Current phase

<!-- PHASE_0_RECON | PHASE_ENV | PHASE_TRACK_WORK | PHASE_ANALYSIS | PHASE_BRINGUP | PHASE_REGRESSION -->

## Active blocker

<!-- guest PC, log line, layer from 13-debug-triage.md -->

## Active commands (verified)

```text
Build:
Run:
Codegen:
```

## Crash / regression table

| Date | Guest PC | Symptom | Layer | Change tried | Result |
|------|----------|---------|-------|--------------|--------|

## Learned patterns

<!-- Format: "X causes Y; fix with Z (source: …)" -->
