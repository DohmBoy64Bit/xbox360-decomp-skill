## Track D: 360tools pipeline (ReXGlue-native)

**Primary upstream:** [DohmBoy64Bit/360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) — modernized, **ReXGlue-native** fork of [sp00nznet/360tools](https://github.com/sp00nznet/360tools). Read the checked-out repo’s `README.md`, `docs/rexglue-workflow.md`, `docs/speed-fix.md`, and `patches/rexglue_patches_audit.md` before recommending commands or patch names.

If the user only needs extraction script usage with no Ghidra/ledger methodology, the dedicated **360tools** agent skill may suffice when installed. This file is the full Track D chapter inside **xbox360-decomp**.

### Trigger phrases

Use Track D when the user mentions **360toolsUpdated**, **360tools** (current fork), `extract_stfs.py`, `extract_iso.py`, `extract_xex_direct.py`, `rexglue init`, `rexglue codegen`, `templates/advanced/`, ReXGlue SDK **patches/** (`0001`–`0005`), `docs/rexglue-workflow.md`, or reference ports (simpsonsarcade, vig8, gh2, ctxbla).

### Pipeline (current — no XenonRecomp step)

360toolsUpdated is **not** a CPU engine. It orchestrates **extraction → ReXGlue SDK → native exe**:

```text
XBLA STFS/LIVE/PIRS/CON  or  Xbox 360 ISO (XDVDFS)
  -> 360toolsUpdated Python extractors (tools/)
  -> default.xex + loose assets in output_dir/
  -> ReXGlue SDK (clone/build; apply patches/ if needed)
  -> rexglue init --app_name <name> --app_root <project>/
  -> copy default.xex -> <project>/assets/
  -> rexglue codegen <name>_config.toml
  -> cmake build (optionally drop in templates/advanced/)
  -> native x86-64 executable
```

**Removed from this fork** (legacy sp00nznet/360tools only — do not recommend unless user explicitly uses old tree):

```text
extract_pe.py, find_abi_addrs.py, find_missing_vtable_funcs.py, post_codegen.py
patches/xenonrecomp-*.patch, templates/project/, docs/xenonrecomp-workflow.md
hedge-dev/XenonRecomp as Track D CPU codegen
```

### Relationship to Tracks A–C

| Track | Role |
|-------|------|
| **A (ReXGlue-only)** | `rexglue init` + `rexglue codegen` from XEX you already have; no 360tools extract bundle |
| **B (XenonRecomp)** | Separate stack — XenonRecomp/XenonAnalyse CPU lift + project runtime; **not** Track D |
| **D (360toolsUpdated)** | End-to-end: STFS/ISO extract → ReXGlue init/codegen → build; optional `templates/advanced/` + SDK `patches/` |
| **C (full decomp)** | Handwritten C++; may use `xex_info.py` / `parse_xex_imports.py` / `extract_switch_tables.py` for analysis only |

Track D and Track A both use **ReXGlue for CPU codegen and runtime**. Track D adds the **360toolsUpdated** extraction scripts, workflow docs, advanced templates, and battle-tested **ReXGlue SDK patches**. Do not tell users to run XenonRecomp between extract and ReXGlue unless they are on the **legacy** sp00nznet tree.

### Prerequisites (verify locally)

From [360toolsUpdated/requirements.txt](https://github.com/DohmBoy64Bit/360toolsUpdated/blob/master/requirements.txt):

```text
Python 3.8+  (pip install -r requirements.txt  ->  pycryptodome>=3.15)
CMake 3.20+, Ninja, Clang 18+ (clang-cl on Windows)
Git
ReXGlue SDK: git clone --recursive https://github.com/rexglue/rexglue-sdk.git
```

Build the SDK per `tools/rexglue-sdk/README.md` (or use a prebuilt `rexglue` binary). Record SDK commit and patch set in `docs/toolchain.md`.

### Phase D0: Locate 360toolsUpdated

```text
[ ] Clone: https://github.com/DohmBoy64Bit/360toolsUpdated
[ ] Record commit hash in docs/toolchain.md
[ ] pip install -r requirements.txt at repo root
[ ] Read docs/rexglue-workflow.md, docs/speed-fix.md, patches/rexglue_patches_audit.md
```

### Phase D1: Extract game files

**XBLA / LIVE package:**

```cmd
python tools/extract_stfs.py path\to\XBLA_PACKAGE output_dir\
```

**Disc ISO:**

```cmd
python tools/extract_iso.py path\to\game.iso output_dir\
```

**Fallback** when STFS layout is unusual:

```cmd
python tools/extract_xex_direct.py
```

(Run with paths per script `--help` / usage block in the local file — do not invent flags.)

Encrypted ISOs: follow `extract_iso.py` message (e.g. [extract-xiso](https://github.com/XboxDev/extract-xiso)) — do not invent decrypt steps.

**Triage** (recommended before codegen):

```cmd
python tools/xex_info.py output_dir\default.xex
python tools/parse_xex_imports.py output_dir\default.xex
```

`parse_xex_imports.py` informs which kernel/XAM stubs you need in `stubs.cpp` (see `templates/advanced/stubs.cpp`).

### Phase D2: ReXGlue SDK + patches

```cmd
git clone --recursive https://github.com/rexglue/rexglue-sdk.git tools\rexglue-sdk
```

Build/install per SDK README. **Optional** ReXGlue SDK patches — skill-bundled `patches/` or **360toolsUpdated/patches/** (same files). Read `25-rexglue-sdk-patches.md` for source-of-truth gating; full audit in `patches/rexglue_patches_audit.md` (SDK v0.8.0 baseline):

| Patch | Purpose | Audit conclusion |
|-------|---------|------------------|
| `0001-use-manual-switch-tables-during-block-discovery.patch` | Manual `[[switch_tables]]` guide block discovery | Needed |
| `0002-tolerate-modifier-only-physical-protection.patch` | Kernel alloc tolerance | Needed |
| `0003-defer-d3d12-primary-submission-with-pending-uav-work.patch` | D3D12 UAV defer submit | Recommended; test per title |
| `0004-fix-d3d12-missing-uav-barriers.patch` | UAV barrier on same-state transitions | Needed for EDRAM-heavy titles |
| `0005-ppc-setjmp-non-volatiles.patch` | `ppc_setjmp`/`ppc_longjmp` save PPC non-volatiles | Critically needed if game uses setjmp |

Apply with `git apply --check` then `git apply` from SDK root **only for patches evidence supports** — do not apply all five on every title. Re-read audit + `25-rexglue-sdk-patches.md` for target commit `e8ce24fa73cd7c1ede80262c06f34893b7963dbe` or your actual checkout.

### Phase D3: Initialize project

```cmd
tools\rexglue-sdk\out\install\win-amd64\bin\rexglue init --app_name mygame --app_root my_project\
cd my_project
```

Generates `CMakeLists.txt`, `src/main.cpp`, `CMakePresets.json`, `mygame_config.toml`.

Copy extracted XEX:

```cmd
mkdir assets
copy ..\output_dir\default.xex assets\
```

Confirm TOML (from `docs/rexglue-workflow.md`):

```toml
file_path = "assets/default.xex"
```

### Phase D4: Codegen

```cmd
rexglue codegen mygame_config.toml
```

Single command handles vtable discovery, switch detection, missing-instruction handling, and C++ generation into `generated/`.

If validation reports missing switch tables, add `[[switch_tables]]` overrides in `mygame_config.toml`. **Fallback helper** (not primary — ReXGlue handles switches natively):

```cmd
python tools/extract_switch_tables.py path\to\default.xex
```

Paste script output into TOML only with PPC proof — do not invent table entries.

**Do not** run legacy `post_codegen.py` or hand-patch `*_init.h` for `PPC_CALL_INDIRECT_FUNC` on this fork; modern ReXGlue handles dispatch natively per upstream README.

### Phase D5: Optional advanced scaffold

`rexglue init` provides the base project. Copy battle-tested pieces from **templates/advanced/** when needed:

| File | Use |
|------|-----|
| `menu.cpp` / `menu.h` | Win32 menu + ImGui dialogs (Graphics, Game, Debug, Controls) |
| `settings.cpp` / `settings.h` | TOML persistence via toml++ |
| `stubs.cpp` | Kernel/XAM stub overrides (license, sign-in, UI) — adapt from `parse_xex_imports.py` |
| `keyboard_driver.cpp` / `.h` | Keyboard + XInput merged input |
| `test_boot.cpp` | Console harness for crash isolation |

Rename `mygame_config.h` includes in templates to match your app name.

### Phase D6: Build and run

```cmd
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
.\build\mygame.exe
```

Next work: game-specific stubs, input, GPU quirks, audio — see ReXGlue wiki (Mid-ASM hooks, ReXCRT).

### Phase D7: Battle-tested runtime fixes

From `docs/speed-fix.md` and README:

**VdSwap frame limiter (still needed):** Windows `Sleep(16)` ≈ 31ms → half-speed feel. Replace with QPC spin-loop (~16667µs target). See `docs/speed-fix.md` for stub pattern.

**Timebase / `__rdtsc` (no longer manual):** Modern ReXGlue maps `mftb` → `QueryGuestTickCount()` at guest 50 MHz. Do **not** recommend legacy `__rdtsc` scaling or `post_codegen` timebase hacks for this fork.

**ROV vs RTV:** White screens with `k_2_10_10_10_FLOAT` + MSAA → prefer ROV (pixel shader interlock) path for EDRAM emulation.

### Phase D8: Bring-up and ledgers

Same discipline as Track A: `docs/address_ledger.md`, `docs/regression_log.md`, [13-debug-triage.md](13-debug-triage.md), [12-stuck-cross-recomp.md](12-stuck-cross-recomp.md).

Failure layer map (current fork):

```text
360toolsUpdated extract script
default.xex path / assets layout
ReXGlue SDK version + patches/ applied?
mygame_config.toml (file_path, switch_tables overrides)
rexglue codegen / generated/
templates/advanced stubs, input, VdSwap
Host CMake/clang-cl build
D3D12 backend (patches 0003/0004 if GPU sync issues)
```

### Reference ports (research-only)

| Game | Repo | Notes |
|------|------|-------|
| The Simpsons Arcade XBLA | [sp00nznet/simpsonsarcade](https://github.com/sp00nznet/simpsonsarcade) | Playable reference |
| Vigilante 8 Arcade | [sp00nznet/vig8](https://github.com/sp00nznet/vig8) | High FPS, many shaders |
| Guitar Hero II | [sp00nznet/gh2](https://github.com/sp00nznet/gh2) | Disc-based |
| Crazy Taxi XBLA | [sp00nznet/ctxbla](https://github.com/sp00nznet/ctxbla) | D3D12, XMA in progress |

Ports may predate 360toolsUpdated; cite repo + file when copying layout. Verify SDK branch, TOML, and patches locally.

### Stack (licenses separate)

- **[ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)** — recompiler + runtime (kernel, D3D12, XMA, input)
- **[Xenia](https://github.com/xenia-project/xenia)** — GPU/kernel lineage in ReXGlue
- **[SIMDE](https://github.com/simd-everywhere/simde)** — Altivec/VMX → host SIMD (handled in ReXGlue codegen)
- **toml++**, **Dear ImGui** — advanced template settings/UI

### Legacy sp00nznet/360tools (XenonRecomp path)

If the user’s tree still has `extract_pe.py`, `find_abi_addrs.py`, XenonRecomp patches, and `post_codegen.py`, they are on **legacy** Track D. Treat that as a **different pipeline** (PE → patched XenonRecomp → ReXGlue `templates/project`). Verify which repo/commit they cloned before giving steps. Default recommendation for new XBLA ports: **360toolsUpdated + ReXGlue-native workflow** above.
