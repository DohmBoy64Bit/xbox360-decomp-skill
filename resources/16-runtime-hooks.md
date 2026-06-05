## Runtime Boundary Function Classification

For every low-level function or crash path, choose the correct treatment:

1. **Recompile normally** when it is ordinary game logic:
   - gameplay logic
   - math
   - actors
   - camera
   - menus
   - animation
   - decompression
   - state machines
   - ordinary data transforms

2. **Replace with a host shim** when the function directly models platform or host I/O:
   - file access
   - kernel imports
   - thread/event/timer functions
   - controller reads
   - save/profile reads/writes
   - audio buffer submission
   - render submission
   - resource creation/destruction
   - platform services

3. **Patch narrowly** when original behavior cannot run correctly on the host:
   - hardware busy-wait loops
   - unavailable service polling
   - device-status loops that would hang
   - impossible timing assumptions
   - command submission that must be intercepted by the host renderer

Do not add broad guards unless original guest behavior proves a pointer may be null or the runtime boundary requires it.
