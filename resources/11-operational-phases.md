# Operational Phases — Xbox 360 / XBLA Projects

> Load at **session start** or when bootstrapping a new port. Works with `XBOX360_PROJECT_STATE.md` in project root.

## Phase 0 — Orientation & XEX reconnaissance

Before codegen or patching, record in state file + `docs/`:

- Game title, region/revision (if known)
- Lawful source of dump (no requesting copyrighted binaries)
- Package path: ISO, STFS/LIVE/XBLA, GOD, title update, DLC
- Source content type: disc/ISO, loose files, XBLA STFS/LIVE, GOD, title update, DLC, or unknown
- `default.xex` path, size, **SHA-256**
- XEX image base, entry point (verify — do not guess)
- `game_data_root` / asset layout; loose vs packed vs XBLA extraction mix
- Package metadata when applicable: content type, title ID, media ID, display name, package size, package hash, extraction tool/output
- Selected **track** (A–D) — see `22-decisional-brain.md`
- ReXGlue / XenonRecomp / 360toolsUpdated commits
- Ghidra program name + import base
- Active blocker

```powershell
Get-FileHash .\assets\game_files\default.xex -Algorithm SHA256
Get-Item .\assets\game_files\default.xex | Select-Object FullName,Length,LastWriteTime
```

```cmd
certutil -hashfile assets\game_files\default.xex SHA256
dir assets\game_files\default.xex
```

Do not continue address-dependent work without proven XEX identity.

## Phase 1 — Environment & tooling

- Audit tools: `01-dev-environment.md` (install/configure only missing pieces; backup configs before merge)
- XBLA extract ledger: `02-xbla-stfs.md`
- **Ghidra quick gate** (before game analysis — full setup in `07-ghidra-mcp.md`):

```text
[ ] XEXLoaderWV import (or documented raw fallback)
[ ] Expected image base in Ghidra matches project/XEX evidence
[ ] GhidraMCP plugin enabled and server started
[ ] Bridge transport matches frontend MCP config
[ ] Health check / tool list OK in the agent frontend
```

- Frontend rules: `20-frontend-rules.md`, `19-agents-md.md`

## Phase 2 — Track workflow (pick one)

| Track | Phase file |
|-------|------------|
| A ReXGlue | `03-track-rexglue.md`, `14-rexglue-phases.md` |
| B XenonRecomp | `04-track-xenon.md` |
| C Matching / full decomp | `05-track-full-decomp.md` |
| D 360toolsUpdated | `06-track-360tools.md` |

## Phase 3 — Analysis & evidence

- Function inventory: `17-function-discovery.md`
- Ghidra protocol: `08-ghidra-evidence.md`
- PPC / guest memory: `23-xenon-execution-discipline.md`

## Phase 4 — Config, hooks, bring-up

- ReXGlue config: `15-rexglue-config.md`
- Runtime boundaries: `16-runtime-hooks.md`
- Debug triage: `13-debug-triage.md`

## Phase 5 — Ledgers & regression

- `24-ledgers-confidence.md`
- Update `docs/regression_log.md` after every verified change

## Project bootstrap (new workspace)

When creating a port tree for any track:

1. Ask whether to `git init` unless the user already decided.
2. Confirm project root path.
3. Add `.gitignore` before copying proprietary assets or large generated output (`21-assets-legal.md`).
4. Keep the original XEX untouched in assets; do not silently overwrite it.
5. Place extracted game data under an ignored asset root.
6. Keep config, hooks, scripts, handwritten source, and docs under version control — not retail binaries.
7. Before final codegen commands: verify the project's ReXGlue mode (direct CLI `rexglue codegen`, CMake codegen utility target, or project wrapper) from local `CMakeLists.txt` / docs — do not assume.

## Project layout (suggested)

Minimal port tree:

```text
ProjectName_Port/
├-- assets/          (default.xex, game_files — gitignored)
├-- config/          (TOML, manifest)
├-- generated/       (tool output — regen overwrites)
├-- src/             (hooks, stubs, handwritten)
├-- docs/            (ledgers, toolchain)
├-- ghidra/          (exports, notes)
└-- CMakeLists.txt
```

Multi-repo / SDK-at-side layout (common for ReXGlue bring-up):

```text
Workspace/
├-- tools/
│   └-- rexglue-sdk/
├-- workspace_env/
│   └-- python/
└-- ProjectName_Port/
    ├-- source/ or src/
    │   ├-- generated/
    │   ├-- hooks/
    │   └-- shared/
    ├-- assets/ (default.xex, game_files/)
    ├-- config/ (manifest.toml, config.toml, hooks.toml)
    ├-- scripts/
    ├-- docs/ (address_ledger.md, regression_log.md, toolchain.md, …)
    ├-- ghidra/ (exports/, function_lists/, notes/)
    └-- CMakeLists.txt
```

Follow the structure your ReXGlue init or template actually generated; document differences in `docs/toolchain.md`. See `18-project-templates.md` for public port layouts — research only.

## When to ask the user

Smallest useful artifact only — examples in `22-decisional-brain.md` § Ask packet.
