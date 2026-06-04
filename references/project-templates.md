## Existing Xbox 360 Recomp Project Templates and Documentation

Use public Xbox 360 recompilation projects only as research-only layout references. They can tighten workflow discipline, but they are not proof that a config field, hook API, generated path, compiler flag, or runtime behavior applies to the current project.

Known references to inspect only when their license and contribution rules allow it:

| Track | Project / Repository | Research-only use | Handling rule |
|---|---|---|---|
| ReXGlue | `rexglue/rexglue-sdk` | Official ReXGlue SDK, quickstart/wiki entry point, release/channel expectations, Xenia-rooted runtime context, legal disclaimer, CLI/config docs to verify locally | Authoritative only for the checked-out SDK version or cited docs/source path |
| ReXGlue | `rexglue/demo-iruka` | Minimal SDK usage shape, demo app layout, starter CMake/codegen organization | Treat as minimal demo, not proof that production title setup is complete |
| ReXGlue | `masterspike52/reNut` | Real ReXGlue title workflow: ignored `assets/default.xex`, `rexglue migrate`, `rexglue codegen renut_config.toml`, `generated/`, CMake/VS presets, exe near assets | Use only for workflow/layout patterns, never game-specific hooks/config |
| ReXGlue | `masterspike52/redahm` | Real ReXGlue title workflow: project config, generated folder, Visual Studio/CMake build, copied assets beside output | Use only for workflow/layout and asset-placement patterns |
| ReXGlue / XBLA | `Subarasheese/daytona-xbla-recomp` | XBLA/STFS package workflow reference: package in `game/`, local extractor script, extracted filesystem, copied `assets/default.xex`, ignored copyrighted package/extracted/assets paths, CMake codegen/build/run commands, `--game_data_root` pattern | Research-only layout reference; verify the active package, extractor, SDK branch, config schema, patches, and runtime arguments locally |
| ReXGlue | `MaxDeadBear/Re-Cherry` | Real ReXGlue title workflow: ISO extraction to `assets`, `rexglue codegen re_cherry_config.toml`, VS CMake target, copied assets | Use only for common ReXGlue build hygiene |
| ReXGlue | `MaxDeadBear/NaughtyBear_ReStuff` | Real ReXGlue title workflow: ReXGlue SDK install, asset/default.xex placement, `rexglue codegen restuffed_config.toml`, VS CMake build | Use only for common ReXGlue build hygiene |
| ReXGlue | `sal063/AC6_recomp` | Source-only policy, native D3D12/Vulkan renderer replacement, `assets/default.xex` input, generated/runtime separation, in-game-but-WIP status reporting | Good reference for renderer/runtime separation and legal repository policy |
| ReXGlue | `WistfulHopes/RB2` | CMake-driven ReXGlue flow: `REXSDK` environment, `assets`, CMake configure, codegen utility target, build target, launch with asset path | Good reference for CMake codegen target pattern instead of direct CLI-only flow |
| ReXGlue | `rexglue/reblue` | ReXGlue-based Blue Dragon recompilation and modding-support goal | Use only for high-level scope/layout ideas after checking repo rules |
| ReXGlue | `flashfire199/Armored-core-ReAnswered` | Community ReXGlue workspace: `assets/`, `generated/`, `src/`, project config TOML, CMake presets | Treat as lower-confidence/community reference; verify everything locally |
| ReXGlue | `testdriveupgrade/TDURE` | ReXGlue community project mentioning SDK version and normal X360 project build | Treat as lower-confidence/community reference; verify everything locally |
| ReXGlue, restricted | `SolarCookies/TiP-Recomp` | Existence and policy reference only: confirms ReXGlue use and no-AI/no-AI-assisted-analysis rule | Do not inspect, imitate, or contribute AI-assisted analysis/code/assets when a repo forbids it |
| XenonRecomp | `hedge-dev/XenonRecomp` | CPU recompiler behavior, generated CPU-state model, endian load/store discipline, missing-instruction handling, and the fact that it does not provide a runtime | Track B; also CPU engine inside Track D |
| 360tools | `sp00nznet/360tools` | XBLA/ISO extraction, XEX→PE, ABI/switch/vtable/import scripts, XenonRecomp patches, `templates/project`, `post_codegen.py`, workflow docs | Track D orchestration; verify commit, `requirements.txt`, and `docs/xenonrecomp-workflow.md` locally |
| 360tools ports | `sp00nznet/simpsonsarcade`, `vig8`, `gh2`, `ctxbla`, `comixzone`, `voot`, `saintsrow` | Shipped or in-progress layouts using the same stack | Research-only; do not copy game stubs or assets |
| XenonRecomp/XenosRecomp | `hedge-dev/UnleashedRecomp` | Full port organization, legal asset installer rules, XenosRecomp shader path, custom renderer/runtime/UI/mod-loader patterns | Track B/Xenos reference only; do not mix generated output into ReXGlue |
| XenosRecomp | `hedge-dev/XenosRecomp` | Shader binary to HLSL workflow, renderer/shader-specific limitations, and expected project-specific modification burden | Shader/renderer reference, not CPU recompilation |
| Historical | `rexdex/recompiler` and LLVM360-style projects | Research-only historical context for Xbox 360 static recompilation | Do not assume current ReXGlue or XenonRecomp compatibility |

Rules for reference repos:

- Cite the exact repository and file/path inspected when using a reference in an answer.
- Do not treat another project's tool version, config schema, commands, hooks, runtime stubs, generated layout, renderer strategy, or generated-code style as authoritative.
- Do not copy game-specific hooks, generated code, copyrighted assets, proprietary data, binary-derived content, project-specific legal/install flows, or generated function metadata.
- Use them only for general organization, documentation style, legal asset handling, build hygiene, runtime-layer separation, codegen/build orchestration, and workflow comparison.
- Verify any command, config field, hook macro, runtime API, generated path, compiler flag, or decompilation helper against the current local source/docs before recommending it.
- Respect project-specific contribution rules and licenses. If a repository bans AI-assisted code, disassembly, or analysis, do not use it as an AI analysis source; at most note that such a policy exists and move to another permitted reference.

### Verified ReXGlue Workflow Patterns From Existing Projects

Use these patterns to tighten Track A answers, while still verifying every command and config field against the user's active ReXGlue SDK checkout.

**Pattern 1: SDK-first setup**

```text
Install or build ReXGlue SDK.
Record SDK version/commit or nightly tag.
Make `rexglue`/`rexglue.exe` or `REXSDK` location explicit.
Run `rexglue --help` and inspect local docs before using commands.
```

**Pattern 2: Asset boundary is always explicit**

```text
Create ignored `assets/` directory.
Place the lawful user-provided `default.xex` and extracted game files under `assets/`.
Never commit `default.xex`, ISO/DVD images, title updates, keys, firmware, or retail data.
Document the local asset root and required folder layout.
```

**Pattern 3: ReXGlue config drives codegen**

Common project configs are title-specific TOML files such as:

```text
renut_config.toml
redahm_config.toml
re_cherry_config.toml
restuffed_config.toml
<project>_config.toml
```

Known fields from real projects include patterns like `includes`, `project_name`, `file_path = "assets/default.xex"`, and `out_directory_path = "generated"`, but do not assume those fields exist or have the same meaning until confirmed in the active SDK/docs and current project file.

**Pattern 4: Two common codegen launch styles**

```text
Direct CLI style:
  rexglue migrate --app_root .      # only if supported/needed by active SDK
  rexglue codegen <project_config>.toml

CMake target style:
  set or verify REXSDK / SDK path
  cmake configure
  run the project-specific codegen utility target
  build the game target
```

Do not force one style. Follow the repository/project convention already present.

**Pattern 5: Generated output and handwritten runtime stay separated**

```text
generated/      ReXGlue PPC-generated C++ output
src/ or source/ handwritten runtime, hooks, renderer, app, launcher, patches
config/         TOML manifests, functions, hooks, switch tables, imports
docs/           ledgers, XBLA/STFS package notes, regression notes, toolchain notes
assets/         ignored retail game data, extracted XBLA data, and `default.xex`
```

Do not patch `generated/` first. Fix config, hook definitions, runtime glue, or source-side replacements unless the active project explicitly has a generated-code patch policy.

**Pattern 6: Build output usually needs asset placement**

Many ReXGlue projects require either:

```text
- built executable in/near the asset directory, or
- copied `assets/` beside the build output, or
- launch argument pointing to the assets folder.
```

When debugging launch failures, verify the exact runtime asset discovery path before changing code.

**Pattern 7: Renderer/runtime strategy is project-specific**

Some projects rely on Xenia-derived GPU/runtime behavior; others replace major renderer paths with native D3D12/Vulkan systems. Treat GPU failures as renderer/runtime-boundary bugs until evidence proves CPU codegen is wrong. Track:

```text
Xenos command/format source
host renderer backend
resource lifetime
shader translation path
texture/buffer endian conversion
asset-derived shader/material metadata
last guest function before submission
```

**Pattern 8: Real project status language should be conservative**

Use status labels like:

```text
codegen-ready
builds
boots
menus
in-game
playable-with-known-bugs
renderer-blocked
audio-blocked
asset-path-blocked
crash-regression
```

Do not describe a port as correct or complete just because it builds, boots, or reaches gameplay.

### ReXGlue Project Audit Checklist

Before modifying or advising a ReXGlue project, collect:

```text
[ ] ReXGlue SDK path, version/commit/nightly tag.
[ ] Whether project uses direct `rexglue codegen` or CMake codegen target.
[ ] Main config TOML path.
[ ] Included config TOML files.
[ ] `default.xex` path and SHA-256.
[ ] Generated output directory.
[ ] Handwritten source/runtime/hook directory.
[ ] Asset discovery rule at runtime.
[ ] Required CMake preset or Visual Studio configuration.
[ ] Whether `rexglue migrate --app_root .` is used, supported, or intentionally avoided.
[ ] Current status: codegen/build/boot/menu/in-game/blocker.
[ ] Known local docs: `AGENTS.md`, `docs/toolchain.md`, `docs/regression_log.md`, `docs/address_ledger.md`, `docs/asset_ledger.md`, `docs/xbla_package_ledger.md`.
```

Xbox 360-specific documentation/reference buckets to consult when needed:

```text
PowerPC/Xenon: PPC instruction set, ABI/calling convention, CR/XER/LR/CTR behavior, VMX/VMX128 semantics.
XEX: image base, sections, imports/exports, TLS, compression/encryption state after legal extraction.
Xenia-derived knowledge: kernel object behavior, filesystem/content roots, Xenos GPU command/format behavior, audio/input/timing semantics.
Microsoft/Windows host docs: D3D12/Vulkan/Win32 threading/filesystem/audio/debug-layer behavior for native port runtime.
Tool docs: selected tool wiki/README/source, Ghidra/GhidraMCP docs, decomp.me and compiler preset docs.
```
