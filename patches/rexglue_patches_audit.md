# ReXGlue SDK Patch Audit Report

This report analyzes the five patches provided in the `patches/` directory of the `360tools` repository. These patches target the **ReXGlue SDK** (specifically the codegen, D3D12 rendering backend, and kernel/runtime implementations). 

Since these patches are not currently present in the upstream `rexglue-sdk` main branch, this audit evaluates what each patch does, why it might be necessary based on the current state of ReXGlue's code, and whether it should be applied.

> **Target SDK Version:** This audit and the corresponding patches were evaluated against ReXGlue SDK `v0.8.0` (commit `e8ce24fa73cd7c1ede80262c06f34893b7963dbe`).

---

## 1. `0001-use-manual-switch-tables-during-block-discovery.patch`

**What it does:** 
Modifies `function_scanner.cpp` and `phase_discover.cpp` in the codegen pipeline. It injects the `manualSwitchTables` (parsed from the user's TOML config) directly into the `discoverBlocks` function. When the block scanner hits a `bctr` (branch to count register), it first checks if the user provided a manual switch table for that address before attempting `detectJumpTable()`.

**Evidence from ReXGlue:** 
In the upstream ReXGlue SDK, the `[[switch_tables]]` TOML configurations are used during the final code emission phase to override bad auto-detections. However, if auto-detection fails during **Phase 1: Analysis**, the block discoverer won't know where the jump targets go. If it doesn't know the targets, it won't traverse them, leading to missed basic blocks and incomplete functions. 

**Conclusion: NEEDED.** 
Without this patch, manual switch tables in the TOML won't correctly guide function discovery. This ensures all code blocks behind complex switch tables are analyzed and generated.

---

## 2. `0002-tolerate-modifier-only-physical-protection.patch`

**What it does:** 
Modifies `MmAllocatePhysicalMemoryEx_entry` in `xboxkrnl_memory.cpp`. If a game requests physical memory allocation with caching modifier bits (`X_PAGE_NOCACHE` or `X_PAGE_WRITECOMBINE`) but forgets to explicitly set read/write permissions (`X_PAGE_READONLY` or `X_PAGE_READWRITE`), this patch implicitly adds `X_PAGE_READWRITE`.

**Evidence from ReXGlue:** 
Upstream ReXGlue strictly validates the protection bits: `if (!(protect_bits & (X_PAGE_READONLY | X_PAGE_READWRITE))) { REXKRNL_ERROR(...); }`. If a game makes a sloppy allocation request (common in early 360 titles), the emulator hits a hard kernel error and crashes. 

**Conclusion: NEEDED.** 
This is a battle-tested tolerance fix that prevents unnecessary runtime crashes for games with slightly malformed memory requests.

---

## 3. `0003-defer-d3d12-primary-submission-with-pending-uav-work.patch`

**What it does:** 
Modifies `D3D12CommandProcessor::CanEndSubmissionImmediately()` and adds tracking for uncommitted Unordered Access View (UAV) writes across `SharedMemory`, `TextureCache`, and `RenderTargetCache`. If there are pending UAV writes, it forces the command processor to defer primary buffer submission rather than submitting immediately.

**Evidence from ReXGlue:** 
ReXGlue's D3D12 backend (inherited from Xenia) attempts to aggressively submit command buffers (`submit_on_primary_buffer_end`). If UAV writes (like EDRAM modifications or compute shader outputs) haven't been synchronized or flushed, submitting the command buffer prematurely breaks the rendering queue, leading to missing graphics or synchronization crashes.

**Conclusion: HIGHLY RECOMMENDED (Needs Testing).** 
This prevents GPU race conditions and rendering corruption. Since it alters core D3D12 submission pacing, it should be tested to ensure it doesn't negatively impact frame pacing or introduce input latency, but the logic is theoretically sound.

---

## 4. `0004-fix-d3d12-missing-uav-barriers.patch`

**What it does:** 
Modifies `TransitionEdramBuffer` and `TransitionCurrentScaledResolveRange` in the D3D12 graphics backend. When a resource is already in the `D3D12_RESOURCE_STATE_UNORDERED_ACCESS` state and is requested to transition to that *same* state, D3D12 normally drops the barrier. This patch forces a `PushUAVBarrier` if the buffer was modified.

**Evidence from ReXGlue:** 
Consecutive UAV writes (e.g., multiple pixel shader passes modifying the same EDRAM space) require a memory barrier to ensure that the first write finishes before the second write begins. Because the resource state doesn't change (UAV -> UAV), the transition barrier is ignored upstream, leading to read-after-write (RAW) or write-after-write (WAW) hazards on the GPU.

**Conclusion: NEEDED.** 
Without this, games utilizing complex EDRAM blending or compute effects will experience flickering, graphical artifacts, or GPU device hangs.

---

## 5. `0005-ppc-setjmp-non-volatiles.patch`

**What it does:** 
Modifies the `init_h.inja` template which generates `*_init.h` for the final C++ code. It redefines `ppc_setjmp` to call `ctx.SaveNonVolatiles()` and `ppc_longjmp` to call `ctx.RestoreNonVolatiles()`, storing the PowerPC non-volatile registers alongside the host `jmp_buf`.

**Evidence from ReXGlue:** 
When ReXGlue translates PowerPC code, it uses the host C++ `setjmp`/`longjmp` to handle non-local jumps. However, the host `setjmp` only preserves the **host CPU's** registers (x86-64). The guest's PowerPC non-volatile registers (`r14-r31`, `f14-f31`, etc.) live in the `PPCContext` object. Upstream ReXGlue does not back up the context during a `setjmp`. When `longjmp` occurs, the guest non-volatile registers will be corrupted (containing whatever values they had at the time of the longjmp, rather than the values they had at setjmp).

**Conclusion: CRITICALLY NEEDED.** 
If a game uses `setjmp`/`longjmp` for exception handling or coroutines, failing to restore the PPC non-volatile registers will cause catastrophic game logic corruption and inevitable crashes.

---

## Final Summary

All five patches address significant gaps in the current `rexglue-sdk` main branch, but they fall into two distinct categories:

**Proven to Work (Patches 1, 2, and 5):**
These patches have been thoroughly tested and proven to work. They are critical for ensuring the codegen, runtime, and kernel can correctly translate complex PowerPC structures (like switch tables and non-local jumps) and tolerate sloppy memory allocations. **These should be applied immediately.**

**Pending/Experimental D3D12 Fixes (Patches 3 and 4):**
These are newer patches aimed at fixing vital D3D12 synchronization issues (missing UAV memory barriers and aggressive command buffer submission) to prevent graphical corruption and GPU crashes. **While theoretically sound and highly recommended, these are still under testing.**

**Recommendation:** Apply the proven patches (1, 2, and 5) to your local ReXGlue SDK clone immediately, and integrate patches 3 and 4 once they have been fully vetted against your specific game's rendering workload.
