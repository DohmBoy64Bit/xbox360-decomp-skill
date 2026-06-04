## XBLA / STFS Title Detection and Workflow

Use this section when the source title is Xbox Live Arcade, Games on Demand, marketplace DLC-like content, or another Xbox 360 package/container workflow instead of a simple disc-root `default.xex` layout.

XBLA titles are still Xbox 360 titles running Xenon PPC code, but the project input is often a **STFS / LIVE content package** rather than a DVD-style extracted filesystem. Do not assume a disc ISO layout, a loose `assets/game_files/` tree, or a directly available root `default.xex` until package extraction proves it.

### XBLA / STFS Detection Checklist

Record evidence for:

```text
Original package path:
Package SHA-256:
Package size:
Detected container type: STFS / LIVE / CON / PIRS / GOD / title update / DLC / unknown
Content type: Arcade Title / Game Demo / Title Update / DLC / Games on Demand / unknown
Title ID:
Media ID:
Package display name:
Package internal file count:
Root executable path after extraction:
Extraction output root:
Extraction tool and version/commit:
Extracted default.xex SHA-256:
Extracted default.xex image base:
Extracted default.xex entry point:
```

Useful checks, depending on available tools:

```powershell
Get-FileHash .\game\<package_file> -Algorithm SHA256
Get-Item .\game\<package_file> | Select-Object FullName,Length,LastWriteTime
Get-ChildItem .\extracted -Recurse | Select-Object FullName,Length | Select-Object -First 50
```

```bash
sha256sum game/<package_file>
find extracted -maxdepth 3 -type f | sort | head -100
file extracted/default.xex assets/default.xex 2>/dev/null || true
```

If the project includes a package extractor script, treat that script as project-specific until reviewed. Verify what it reads, what it writes, and whether it copies or transforms `default.xex`, package metadata, or asset files.

### XBLA Extraction and Asset Root Rules

A lawful XBLA workflow usually looks like:

```text
user-owned XBLA/STFS package
  -> local package hash and metadata ledger
  -> extractor produces extracted/ package filesystem
  -> root or nested default.xex is copied/referenced as assets/default.xex or equivalent
  -> extracted/ is used as game_data_root / asset root by the host runtime
  -> package, extracted files, default.xex, generated PE/image dumps, and media archives stay ignored/private
```

Do not assume the executable lives at the package root. Some package/container layouts may include nested paths, title-update content, DLC content, multiple executables, or auxiliary metadata. Find the executable by package metadata, extractor output, file type, and XEX loader evidence.

For XBLA projects, maintain an `xbla_package_ledger.md` or add a dedicated section to `docs/asset_ledger.md`:

```text
Package path:
Package hash:
Package/container type:
Content type:
Title ID:
Media ID:
Package display name:
Extractor/tool:
Extraction command:
Extraction output path:
Extracted file count:
Root default.xex path:
default.xex hash:
Files copied into assets/:
Files used as runtime game_data_root:
Ignored/private paths:
Verification command:
```

### XBLA Runtime Boundary Considerations

XBLA titles may rely on Xbox Live Arcade and content-package behaviors that are not visible from raw PPC code alone. Classify these separately from ordinary disc filesystem behavior:

```text
XContent / package mounting behavior
Xam/XLive title and user APIs
profile/user index assumptions
trial/demo/full-unlock state checks, without bypassing DRM or licensing
save-container paths and title storage
leaderboard/achievement/network stubs
package-relative asset paths
case sensitivity and path separator behavior after extraction
media archive loading from extracted package roots
```

Legal/safety rule:

```text
Allowed: identify whether the title expects XBLA package/content APIs, document how package extraction feeds the recomp runtime, stub offline-only service calls when lawful, and ask for legally usable package/extraction evidence.
Not allowed: bypassing trial/full unlock checks, license checks, activation, marketplace entitlement, online authentication, anti-cheat, multiplayer integrity, or redistributing extracted package contents.
```

### XBLA Recomp Project Reference Pattern

Use public XBLA recomp projects only as research-only layout references. For example, `Subarasheese/daytona-xbla-recomp` describes itself as a static recompilation of **Daytona USA (Xbox 360 / XBLA, 2011)** using ReXGlue. Its README shows an XBLA-specific layout where copyrighted game files are excluded, the user provides a lawful local package under `game/`, a script extracts the STFS package into `extracted/`, copies the required executable to `assets/default.xex`, and keeps `game/`, `extracted/`, and `assets/` ignored. It also shows that a project may require a specific ReXGlue SDK branch, CMake codegen targets, generated-code patches, and a `--game_data_root` argument.

Treat that as a pattern only:

```text
Useful as reference:
- package-ledger discipline
- extractor-script placement
- ignored package/extracted/asset paths
- copied assets/default.xex convention
- runtime --game_data_root convention
- documenting tool branches, patches, and build/run commands

Not proof for the current project:
- ReXGlue branch name
- manifest schema
- codegen target name
- patch policy
- file count
- package title/media IDs
- runtime warnings
- asset paths
- host renderer behavior
```

Before copying any Daytona-style convention, verify the current project's local package, extractor, SDK branch, manifest/config schema, generated output, and runtime command.
