## ReXGlue Track: Concrete Project Workflow

Use this section when Track A is selected. It is based on patterns seen across actual ReXGlue projects, but every command and path still needs local verification.

### ReXGlue Track Phases

```text
Phase R0: SDK and repository audit
- Locate ReXGlue SDK or `REXSDK`.
- Record SDK version/commit/nightly.
- Run or inspect `rexglue --help`.
- Identify whether this project uses direct CLI codegen or a CMake codegen target.
- Optional SDK patches: `25-rexglue-sdk-patches.md` + skill `patches/` — apply per symptom/upstream check; record in `docs/toolchain.md`.

Phase R1: Asset placement
- Create ignored `assets/`.
- Place lawful user-provided `default.xex` and extracted game files under the expected asset root.
- Hash `default.xex` and record it.
- Do not commit retail data, disc images, keys, or title updates.

Phase R2: Config audit
- Identify `<project>_config.toml` and included TOML files.
- Confirm `file_path`, output directory, functions, hooks, switch tables, and imports from local config.
- Do not copy config fields from another project without active SDK proof.

Phase R3: Codegen
- Run `rexglue migrate --app_root .` only when the active SDK/project uses it.
- Run `rexglue codegen <project_config>.toml` only when the project uses direct CLI codegen.
- If the project uses CMake, run the project-specific codegen utility target instead.
- Keep generated output in the configured generated directory.

Phase R4: Build
- Use the project's CMake preset or Visual Studio CMake configuration.
- Prefer the project's documented preset names over guessed names.
- Keep build output, generated files, and handwritten runtime code separated.

Phase R5: Runtime launch
- Verify whether the exe must sit beside assets, assets must be copied beside the exe, or an asset path argument is required.
- Log asset root, current working directory, and `default.xex` discovery on launch.

Phase R6: Iteration
- Map failures to config, generated code, hook layer, runtime shim, renderer, audio, input, asset path, or build system.
- Update ledgers after each meaningful change.
```

### ReXGlue Commands: Safe Presentation Rule

It is acceptable to mention common observed command shapes as examples:

```cmd
rexglue migrate --app_root .
rexglue codegen <project_config>.toml
cmake --preset <preset>
cmake --build --preset <preset>
```

But do not present them as final commands until the current project proves:

```text
[ ] active SDK supports the command
[ ] config path exists
[ ] project uses direct CLI rather than CMake target
[ ] expected generated output path is known
[ ] CMake preset or VS configuration exists
[ ] asset placement requirement is known
```

### ReXGlue Project Reference Decision Tree

```text
Need official command/config truth?
  -> Check active ReXGlue SDK source/docs first.

Need common project layout?
  -> Compare reNut, reDAHM, Re-Cherry, NaughtyBear_ReStuff, RB2, AC6_recomp, or demo-iruka.

Need renderer/runtime separation ideas?
  -> Prefer AC6_recomp-style source-only/runtime-renderer separation and Xenia/Xenos-derived docs.

Need CMake codegen target pattern?
  -> Compare RB2-style REXSDK + CMake codegen target flow.

Need asset/legal policy wording?
  -> Compare AC6_recomp and other source-only projects.

Repo has no-AI / no-AI-assisted-analysis rule?
  -> Do not inspect or use it as an AI analysis reference. Choose another repository.
```
