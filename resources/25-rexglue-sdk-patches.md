# ReXGlue SDK optional patches (0001–0005)

> **Not every title needs every patch.** Apply only when **source of truth** proves the upstream SDK at your commit still lacks the fix. Bundled files: skill `patches/` (same set as [360toolsUpdated/patches](https://github.com/DohmBoy64Bit/360toolsUpdated/tree/master/patches)).

## Source-of-truth order (before `git apply`)

1. **Your SDK checkout** — read `function_scanner.cpp`, `phase_discover.cpp`, `xboxkrnl_memory.cpp`, D3D12 backend, `init_h.inja` at the pinned commit. If upstream already contains the fix, **skip** the patch and record “upstream absorbed” in `docs/toolchain.md`.
2. **Symptom evidence** — crash log, GPU artifact, codegen gap, or PPC proof (`09-original-game-evidence.md`, `08-ghidra-evidence.md`).
3. **`patches/rexglue_patches_audit.md`** — what each patch changes and audit conclusion (evaluated vs SDK `v0.8.0` / `e8ce24fa`).
4. **Title / port docs** — `docs/toolchain.md`, public port README (research only, `18-project-templates.md`).
5. **Labeled inference** — last resort; note in regression log.

Record in `XBOX360_PROJECT_STATE.md` / `docs/toolchain.md`:

```text
SDK commit:
Patches applied: 0001 / 0002 / 0003 / 0004 / 0005 / none
Evidence for each:
Upstream check: file:line still missing fix? yes/no
```

## Apply workflow

```text
cd <rexglue-sdk-root>
git status   # clean or branched patch branch
git apply --check ../path/to/patches/0001-....patch
git apply ../path/to/patches/0001-....patch
# rebuild SDK; rexglue codegen; cmake build; verify symptom
```

**0005 caveat:** the bundled `0005-ppc-setjmp-non-volatiles.patch` is a **cumulative rollup** (includes hunks from earlier patches). On a clean upstream checkout, `git apply --check` for **0005 alone often fails** — apply **0001–0004 individually** first, or copy the `init_h.inja` non-volatile save/restore hunks manually per `rexglue_patches_audit.md` when setjmp/longjmp is proven in PPC.

Patch paths (skill install):

```text
<prefix>/xbox360-decomp/patches/0001-use-manual-switch-tables-during-block-discovery.patch
… through 0005-ppc-setjmp-non-volatiles.patch
<prefix>/xbox360-decomp/patches/rexglue_patches_audit.md
```

Or from **360toolsUpdated** clone: `tools/../patches/` at repo root.

## Per-patch decision guide

| Patch | Layer | Apply when evidence shows… | Skip when… |
|-------|-------|---------------------------|------------|
| **0001** manual switch tables in block discovery | Codegen Phase 1 | `[[switch_tables]]` in TOML but codegen misses blocks behind `bctr`; incomplete functions after discovery | Upstream `discoverBlocks` already merges `manualSwitchTables`; auto-detection + TOML emission sufficient for this title |
| **0002** modifier-only physical protection | Kernel / `MmAllocatePhysicalMemoryEx` | Kernel error on alloc with NOCACHE/WRITECOMBINE but no RW bit; early-title sloppy protect flags | Upstream tolerates modifier-only requests; no such crash in trace |
| **0003** defer D3D12 submit with pending UAV | GPU / D3D12 pacing | Flicker, missing draws, sync crashes with pending UAV work; audit symptom matches | Stable frames after 0004 only; or upstream already defers submit; latency regression on test |
| **0004** UAV barriers on same-state transition | GPU / EDRAM | EDRAM/compute RAW/WAW artifacts, flicker, DEVICE_HUNG on UAV-heavy paths | Upstream `PushUAVBarrier` on modified UAV→UAV; title has no EDRAM-heavy passes |
| **0005** `ppc_setjmp` / `ppc_longjmp` non-volatiles | Codegen runtime template | Game uses setjmp/longjmp; guest `r14–r31` / FPR corruption after longjmp; PPC proof at crash | Upstream `init_h.inja` already saves/restores `PPCContext` non-volatiles; no setjmp in title |

### Default tiers (audit summary — verify per SDK commit)

```text
Proven / apply first when symptom matches: 0001, 0002, 0005
GPU experimental — test per title: 0003, 0004
```

**Do not** apply all five blindly on every XBLA port. Guardian Heroes–class bring-up may need **0001** (switch discovery) + **0005** (setjmp) only; a 2D XBLA may need **0002** kernel tolerance only; shader-heavy titles may need **0003/0004** after GPU triage (`13-debug-triage.md`).

## Symptom → patch quick map

| Symptom | Check first | Patch candidate |
|---------|-------------|-----------------|
| Unregistered VA / incomplete codegen at switch `bctr` | TOML `[[switch_tables]]`, `17-function-discovery.md` | **0001** |
| `MmAllocatePhysicalMemoryEx` / kernel protect error | Import trace, kernel log | **0002** |
| White screen / EDRAM flicker / TDR | ROV path, `06-track-360tools.md` D7 | **0003**, **0004** |
| Corruption after longjmp / exception path | PPC at setjmp site | **0005** |

## Config before patch

For switch issues, prefer **`[[switch_tables]]` in `*_config.toml`** (`15-rexglue-config.md`) with Ghidra/PPC proof. Patch **0001** makes manual tables visible during **discovery** — still requires correct TOML entries.

## Related

- Track D pipeline: `06-track-360tools.md` Phase D2
- Track A SDK audit: `03-track-rexglue.md` Phase R0
- Debug layers: `13-debug-triage.md`
- Full audit prose: `patches/rexglue_patches_audit.md`
