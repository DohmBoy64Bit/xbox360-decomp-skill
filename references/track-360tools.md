## Track D: 360tools pipeline (XenonRecomp + ReXGlue)

If the user only needs script order, patches, and `post_codegen` (no Ghidra/ledger methodology), the dedicated **360tools** agent skill is narrower and sufficient when installed. This file is the full Track D chapter inside **xbox360-decomp**.

Use when the user mentions **[360tools](https://github.com/sp00nznet/360tools)**, `sp00nznet/360tools`, `extract_stfs.py`, `extract_pe.py`, `find_abi_addrs.py`, `extract_switch_tables.py`, `find_missing_vtable_funcs.py`, `post_codegen.py`, XenonRecomp **patches** from that repo, or shipped ports in the README (simpsonsarcade, vig8, gh2, ctxbla, comixzone, voot, saintsrow).

360tools is **not** a fourth CPU engine. It is an orchestration layer:

```text
XBLA STFS/LIVE/PIRS/CON  or  Xbox 360 ISO (XDVDFS)
  -> 360tools Python extractors (tools/)
  -> decrypted/decompressed PE image (extract_pe.py; XenonRecomp input)
  -> analysis scripts (ABI, switch tables, vtables, XEX imports)
  -> XenonRecomp (hedge-dev/XenonRecomp + 360tools patches/)
  -> generated C++
  -> ReXGlue SDK runtime + templates/project scaffold
  -> native x86-64 executable
```

### Relationship to Tracks A–C

| Track | Role |
|-------|------|
| **A (ReXGlue-only)** | `rexglue codegen` from XEX; no 360tools PE pipeline required |
| **B (XenonRecomp)** | CPU codegen only; 360tools can **feed** patched XenonRecomp + TOML helpers |
| **D (360tools)** | End-to-end: extraction → PE → XenonRecomp (patched) → ReXGlue template runtime |
| **C (full decomp)** | Handwritten C++; may use 360tools scripts for binary analysis only |

Do not tell users to run Track A and Track D codegen in parallel on the same title without proving compatibility. Typical 360tools ports use **XenonRecomp for CPU** and **ReXGlue for OS/GPU/audio/input**.

### Prerequisites (verify locally)

From upstream README — confirm versions in the checked-out `360tools/requirements.txt` and project docs:

```text
Python 3.8+  (pip install -r requirements.txt)
CMake 3.20+, Ninja, Clang 18+ (clang-cl on Windows)
MSVC 2022 (Windows SDK)
Git clones: XenonRecomp (recursive), ReXGlue SDK
```

### Phase D0: Locate 360tools

```text
[ ] Clone or vendor: https://github.com/sp00nznet/360tools
[ ] Record commit hash in docs/toolchain.md
[ ] pip install -r requirements.txt inside 360tools root
[ ] Read docs/xenonrecomp-workflow.md, docs/speed-fix.md, docs/binary-analysis.md
```

### Phase D1: Extract game files

**XBLA / LIVE package:**

```cmd
python tools/extract_stfs.py path\to\XBLA_PACKAGE output_dir\
```

Fallback when STFS layout is unusual: `extract_xex_direct.py`.

**Disc ISO:**

```cmd
python tools/extract_iso.py path\to\game.iso output_dir\
```

Encrypted ISOs: follow tool message (e.g. extract-xiso) — do not invent decrypt steps.

Triage before full pipeline: `python tools/xex_info.py path\to\default.xex`

### Phase D2: XEX → PE image

```cmd
python tools/extract_pe.py output_dir\default.xex pe_image.bin
```

`extract_pe.py` handles AES-128, basic block and LZX compression, raw COFF edge cases. This PE is what **XenonRecomp** consumes (not the encrypted XEX2 container).

Optional faster path if XenonUtils already built: `dump_pe.cpp` (see 360tools README).

### Phase D3: XenonRecomp config helpers

Run from 360tools root; paste outputs into project TOML (see `config/` examples):

```cmd
python tools/find_abi_addrs.py pe_image.bin
python tools/extract_switch_tables.py pe_image.bin
python tools/find_missing_vtable_funcs.py pe_image.bin generated\<game>_init.cpp
python tools/parse_xex_imports.py path\to\default.xex
```

Re-run XenonRecomp after each TOML update. Do not invent `[[switch]]` or function entries without script output or PPC proof.

### Phase D4: Build patched XenonRecomp

Verify patch paths exist in the local 360tools checkout:

```text
patches/xenonrecomp-altivec-vmx.patch
patches/xenonrecomp-missing-instructions.patch
```

```cmd
git clone --recursive https://github.com/hedge-dev/XenonRecomp.git
cd XenonRecomp
git apply <path-to-360tools>\patches\xenonrecomp-altivec-vmx.patch
git apply <path-to-360tools>\patches\xenonrecomp-missing-instructions.patch
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release ^
  -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl
cmake --build build --config Release
```

Exact binary name and TOML schema: read local XenonRecomp README and `config/example.toml` — do not guess CLI flags.

### Phase D5: Codegen iteration

```text
XenonRecomp pe_image.bin <project>.toml  -> generated C++
Append switch/vtable/ABI outputs -> re-run until clean or documented gaps
```

After **ReXGlue** codegen passes in the game repo:

```cmd
python tools/post_codegen.py
```

Re-applies `PPC_CALL_INDIRECT_FUNC` / `PPC_UNIMPLEMENTED` overrides in `*_init.h` when rexglue regenerates — run after every rexglue codegen per upstream docs.

### Phase D6: Project scaffold (ReXGlue runtime)

```text
cp -r templates/project <your_game>/project
```

Customize from local template only:

- `ppc_config.h` — `__builtin_trap` override, scaled `__rdtsc`, safe `PPC_CALL_INDIRECT_FUNC`
- `CMakeLists.txt` — ReXGlue SDK, WHOLEARCHIVE linking
- `src/main.cpp` — VEH, guest demand paging, null page handler
- `src/stubs.cpp` — game-specific kernel/XAM stubs (use `parse_xex_imports.py` list)
- `docs/speed-fix.md` — VdSwap `Sleep(16)` → QPC spin; timebase scaling

Battle-tested runtime topics (verify in template + speed-fix doc):

```text
VdSwap frame limiter (Windows Sleep granularity)
Guest timebase vs host TSC (__rdtsc scaling)
PPC_CALL_INDIRECT_FUNC (NULL, import thunk, in-range)
ROV vs RTV for k_2_10_10_10_FLOAT + MSAA white-screen class
```

### Phase D7: Bring-up and ledgers

Same discipline as Track A/B: `docs/address_ledger.md`, `docs/regression_log.md`, [debug-triage.md](debug-triage.md), [stuck-cross-recomp.md](stuck-cross-recomp.md).

Map failures to layer:

```text
360tools extract script
PE image / XenonRecomp TOML
XenonRecomp translation + patches
ReXGlue config/codegen
templates/ runtime (stubs, GPU, audio, input)
Host build (CMake/clang-cl)
```

### Reference ports (research-only)

| Game | Repo | Notes |
|------|------|-------|
| The Simpsons Arcade XBLA | sp00nznet/simpsonsarcade | Playable reference |
| Vigilante 8 Arcade | sp00nznet/vig8 | High FPS, many shaders |
| Guitar Hero II | sp00nznet/gh2 | Disc-based |
| Crazy Taxi XBLA | sp00nznet/ctxbla | D3D12, XMA in progress |
| Comix Zone XBLA | sp00nznet/comixzone | Large codegen, scaffold WIP |

Cite repo + file when copying layout; verify SDK branch, TOML, and patches locally.

### Upstream stack (licenses separate)

- **XenonRecomp** (hedge-dev) — PPC → C++
- **ReXGlue SDK** (hedge-dev) — kernel, D3D12 GPU, XMA, input
- **Xenia** — GPU lineage in ReXGlue
- **SIMDE** — VMX → SSE/AVX in XenonRecomp
