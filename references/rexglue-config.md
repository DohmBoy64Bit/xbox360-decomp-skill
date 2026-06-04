## ReXGlue Config / Manifest Safety

Because ReXGlue schema and CLI details can change, treat all examples as patterns until verified against the current local version.

Before changing config:

```text
[ ] Which ReXGlue version/commit is active?
[ ] Which config/manifest file is being edited?
[ ] Does local source/docs confirm this field?
[ ] Is the guest address correct?
[ ] Is the function boundary proven?
[ ] Is this a switch table, hook, runtime import, or generated-code setting?
[ ] How will this be verified?
```

When writing config comments, include:

```text
Evidence:
Guest address:
Original PPC context:
Reason:
Expected effect:
Rollback:
```
