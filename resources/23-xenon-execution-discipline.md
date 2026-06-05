# Xenon execution discipline — guest memory, PPC, subsystems

> Load for address bugs, endian issues, indirect calls, or platform classification. Hook documentation template → `14-rexglue-phases.md` § Hook Discipline.

## Address-space discipline

Always distinguish:

- Guest virtual address
- Original XEX image base and image offset
- File offset inside the XEX container, when known
- Guest stack / heap / global-static addresses
- Generated C++ register accessors
- ReXGlue guest-memory translation layer
- Runtime asset pointers vs host process / graphics / audio pointers

```text
guest_offset = guest_address - xex_image_base
```

Only after **confirmed** image base. Crash guest PC ≠ host RIP — keep separate. Never cast guest VA to host pointers unless the runtime documents that translation.

### Region table (example — verify per title)

```text
Name         Guest Start   Guest End    XEX/Image Offset  Host/Runtime Mapping  Notes
main_text    0x82A00000    …            0x00000000        generated code        PPC text
main_rdata   …             …            …                 guest memory          vtables, strings
main_data    …             …            …                 guest memory          writable globals
stack        dynamic       dynamic      n/a               guest memory          r1-based frames
asset_root   n/a           n/a          n/a               host filesystem       extracted files
```

Do not assume another title shares the same image base, region layout, loader behavior, section names, paths, or asset format.

## PowerPC / Xenon CPU

Xenon PPC is **big-endian** from the guest program's perspective. When analyzing or translating, consider:

- GPR signedness; FPR rounding, NaN, conversions
- VMX/VMX128 lane order and saturation
- CR fields; XER carry/overflow
- LR/CTR: `bl`, `bctrl`, `mtctr`, `bctr`, tail calls, virtual dispatch
- `r1` stack frames; nonvolatile save/restore
- Guest endian on little-endian Windows hosts
- Alignment and unaligned access; atomics/interlocked behavior
- Compiler thunks, prologues, epilogues, import stubs

Do not simplify PPC to C++ unless carry, overflow, condition-code, endian, and vector-lane effects are modeled or proven irrelevant.

## Indirect control flow (high risk)

When you see:

```asm
mtctr rX
bctr
```

or `mtctr rX` + `bctrl`, classify the CTR source:

```text
switch table | vtable slot | function pointer | import thunk
callback array | tail-call target | hand-written dispatcher | unresolved
```

Evidence to collect before config changes:

```text
Target address:
Containing function:
Instructions that compute CTR:
Data references feeding CTR:
Possible table base / entry size / bounds check:
Known targets / fallthrough:
Ghidra switch or vtable result:
ReXGlue config impact:
Confidence:
```

No switch-table or indirect-call config from guess — GhidraMCP, raw PPC, or tool output required.

## Runtime subsystem checklist

Classify the owning layer before proposing a fix (`13-debug-triage.md`):

- **XEX loader**: base, sections, imports, exports, relocations, TLS, entrypoint
- **Xenon CPU / PPC**: control flow, registers, stack, branch targets, semantics
- **Guest memory**: globals, stack, heap, endian, address translation
- **Kernel/API stubs**: threads, sync, timers, files, allocation, title APIs
- **Filesystem / assets**: path mapping, case sensitivity, archives, `game_data_root`
- **Input**: controller polling, XInput mapping, deadzones, vibration
- **Audio**: XMA or title-specific paths, streaming, buffers, host output
- **GPU / Xenos**: command submission, shaders, textures, RTs, sync (not NV2A — OG Xbox)
- **Host graphics backend**: D3D12/Vulkan/OpenGL, device loss, resource lifetime, TDR/DEVICE_HUNG
- **Timing / scheduler**: frame pacing, VdSwap (`06-track-360tools.md` speed-fix), guest timers
- **Networking / services**, **save/profile**, **DLC / title updates**: usually stubbed unless the port targets them

Common ReXGlue failures are runtime/mapping issues, not raw instruction bugs: wrong image base, guest/host pointer mix-up, endian, bad indirect target, missing stub or asset path, GPU sync, overbroad hook, host renderer crash.
