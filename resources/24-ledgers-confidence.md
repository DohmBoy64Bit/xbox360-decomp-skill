# Ledgers & symbol confidence

> Load when naming functions, hooks, or maintaining `docs/` ledgers.

## Confidence labels

```text
Known     ReXGlue output, Ghidra boundary, xref proof, export/import, trace
Likely    prologue/CF/xref evidence
Tentative decompiler/heuristic only
Unknown   needs PPC, Ghidra, or trace
```

Never commit **Tentative** as **Known**. Before promoting a boundary or hook point, check raw disassembly, branch targets, register state, nearby data boundaries, image mapping, and runtime behavior.

Export columns:

```text
Name, Guest Start, Guest End, Size, XEX/Image Offset, Type, Confidence, Evidence
```

## Ledger files

```text
docs/address_ledger.md
docs/asset_ledger.md
docs/xbla_package_ledger.md
docs/function_ledger.md
docs/runtime_boundaries.md
docs/regression_log.md
docs/toolchain.md
docs/compiler_matching.md
docs/ghidra_mcp_setup.md
docs/frontend_agent_rules.md
```

### address_ledger.md

```text
Guest Address, Function/Symbol, Type, Evidence, Hook/Config Impact, Status, Notes
```

### asset_ledger.md

```text
Guest Path, Host Path, Source Archive/Folder, Required?, Status, Evidence, Notes
```

### regression_log.md entry

```text
Date, Build/commit, Change, Evidence before, Result after, Regression risk, Next action
```

Mark TODO with exact missing artifact — no fiction.
