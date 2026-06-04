## Development Environment Check and Dependency Setup

Before starting a selected-tool generation pass, GhidraMCP session, host build, full-decomp matching pass, or crash-debug pass, check whether the needed tools are already installed. Prefer verifying tools before installing them. Do not install unrelated packages or change global configuration unless the tool is needed for the current workflow.

### Core Tool Checks

Windows PowerShell / Developer PowerShell:

```powershell
git --version
cmake --version
ninja --version
python --version
py --version
java -version
mvn -version
clang-cl --version
where git
where cmake
where ninja
where python
where py
where java
where mvn
where clang-cl
where cl
where ghidraRun
```

If `clang-cl.exe` exists but is not on `PATH`, document the actual path instead of assuming the shell can call it.

### Windows Dependency Setup

Use a Visual Studio x64 Developer PowerShell or Developer Command Prompt when building C/C++ targets with MSVC-compatible tooling.

Install missing tools through the user's preferred package manager or official installers. Example `winget` commands, if available:

```powershell
winget install --id Git.Git -e
winget install --id Kitware.CMake -e
winget install --id Ninja-build.Ninja -e
winget install --id Python.Python.3.11 -e
winget install --id EclipseAdoptium.Temurin.21.JDK -e
winget install --id Apache.Maven -e
winget install --id LLVM.LLVM -e
winget install --id Microsoft.VisualStudio.2022.BuildTools -e
winget install --id NSA.Ghidra -e
```

After installing Visual Studio Build Tools, ensure the C++ workload is installed:

```text
Desktop development with C++
MSVC v143 or newer
Windows 10/11 SDK
CMake tools for Windows, if desired
```

For Ghidra, keep the install path explicit, for example:

```text
C:\ghidra_12.1_PUBLIC
```

GhidraMCP setup commands usually need an exact Ghidra path.

### Python Analysis Dependencies

Prefer a project-local virtual environment:

```cmd
py -3.11 -m venv workspace_env\python
workspace_env\python\Scripts\activate
python -m pip install --upgrade pip
python -m pip install capstone tomlkit construct lief
```

Use extra Python dependencies only when a script actually needs them. Document the script, expected input, and output format before relying on it.

### Selected Tool Stack Checks

Before running ReXGlue generation, XenonRecomp/XenonAnalyse generation, full-decomp tooling, or building the host app, verify:

```cmd
git submodule status
cmake --version
ninja --version
python --version
clang-cl --version
```

Verify selected tool availability from the local checkout or installed SDK, and verify Ghidra extensions when analysis will be used:

```cmd
rexglue --help
XenonRecomp --help
XenonAnalyse --help
XenosRecomp --help
# In Ghidra UI or extension install directory: verify XEXLoaderWV and GhidraMCP are installed/enabled
```

Run only the commands relevant to the selected track. If a command is unavailable, inspect that tool's local repository docs/build output rather than inventing an invocation.

### Dependency Documentation Rule

When the user has a working setup, document the verified environment in `AGENTS.md` or `docs/toolchain.md`:

```text
OS and shell
Tool versions
Compiler and generator
Ghidra version and install path
XEXLoaderWV repository/release path and installed extension version
GhidraMCP repository path and plugin version/commit
ReXGlue SDK version/commit
Python environment command
Generation/build/run commands
Known missing PATH entries
```
