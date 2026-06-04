## Project Setup

When creating a new Xbox 360 project workspace for any track:

1. Ask whether the user wants to initialize a git repository unless they already gave the answer.
2. Confirm the selected project root.
3. Add `.gitignore` before copying proprietary assets or generating large outputs.
4. Keep the original XEX untouched.
5. Place extracted assets under an ignored asset root.
6. Keep configuration, hooks, scripts, source, and docs under version control.
7. Verify selected-tool commands before presenting final generation steps. For ReXGlue projects, determine whether the project uses direct CLI codegen, a CMake codegen utility target, or a generated project-specific wrapper before giving commands.

Suggested layout:

```text
Workspace/
├-- tools/
|   └-- rexglue-sdk/
├-- workspace_env/
|   └-- python/
└-- ProjectName_Port/
    ├-- source/
    |   ├-- generated/
    |   ├-- hooks/
    |   └-- shared/
    ├-- assets/
    |   ├-- default.xex
    |   └-- game_files/
    ├-- config/
    |   ├-- manifest.toml
    |   ├-- config.toml
    |   └-- hooks.toml
    ├-- scripts/
    ├-- docs/
    |   ├-- address_ledger.md
    |   ├-- asset_ledger.md
    |   ├-- regression_log.md
    |   └-- toolchain.md
    ├-- ghidra/
    |   ├-- exports/
    |   ├-- function_lists/
    |   └-- notes/
    └-- CMakeLists.txt
```

This layout is a suggestion, not proof of ReXGlue's required structure. If ReXGlue generates a different structure, follow the generated project and document the difference.
