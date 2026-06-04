## Xbox 360 Ghidra Loader, GhidraMCP, and Frontend Auto-Setup

Use this section when the user asks to set up analysis tooling, when an Xbox 360 XEX needs to be imported into Ghidra, or when an MCP-capable editor/agent should be wired to Ghidra. The goal is to make setup repeatable without overwriting user configuration.

### Setup Automation Policy

When shell and file access are available, the agent should perform the setup steps directly. When access is not available, provide exact commands and config snippets.

Always follow this order:

```text
1. Audit existing dev dependencies.
2. Locate or install Ghidra.
3. Detect Ghidra version and install path.
4. Check/install the Xbox 360 XEX loader extension, preferably `zeroKilo/XEXLoaderWV` or an explicitly selected maintained fork.
5. Check/install `bethington/ghidra-mcp`.
6. Start or guide starting Ghidra and the GhidraMCP server.
7. Detect the user's MCP frontend: Cursor, Antigravity, Codex, OpenCode, Claude Desktop, Warp, or other.
8. Locate the frontend's existing MCP config through the frontend UI or known config path.
9. Back up the config file before editing.
10. Merge a single `ghidra` MCP entry without deleting existing servers.
11. Verify both Ghidra-side health checks and frontend-side tool availability.
12. Record the final paths/versions in `docs/toolchain.md` or `AGENTS.md`.
```

Do not blindly overwrite config files. If a config file already contains a `ghidra`, `ghidra-mcp`, or similar server entry, update it only after comparing the existing command, args, transport, and path.

### Dev Dependency Audit Script Shape

Prefer using a small audit script or commands that report tools rather than failing immediately on the first missing dependency.

Windows PowerShell audit shape:

```powershell
$tools = @(
  'git', 'cmake', 'ninja', 'python', 'py', 'java', 'mvn',
  'clang-cl', 'cl', 'ghidraRun', 'curl'
)
foreach ($tool in $tools) {
  $cmd = Get-Command $tool -ErrorAction SilentlyContinue
  if ($cmd) { Write-Host "FOUND $tool -> $($cmd.Source)" }
  else { Write-Host "MISSING $tool" }
}

if (Get-Command winget -ErrorAction SilentlyContinue) { winget --version }
if (Get-Command scoop -ErrorAction SilentlyContinue) { scoop --version }
```

Linux / WSL audit shape:

```bash
for tool in git cmake ninja python3 java mvn clang curl ghidraRun; do
  if command -v "$tool" >/dev/null 2>&1; then
    echo "FOUND $tool -> $(command -v "$tool")"
  else
    echo "MISSING $tool"
  fi
done
```

### Automatic Dependency Installation Rules

Install only missing dependencies needed for the selected track and current task. Use the **Windows Dependency Setup** commands above as the canonical dependency-install examples, then return here for Ghidra/XEXLoaderWV/GhidraMCP-specific setup.

After Visual Studio Build Tools installation, verify the C++ workload and Windows SDK. If `clang-cl.exe` exists but is not on `PATH`, record the actual path and either launch from a Developer PowerShell or set project-local toolchain variables.

### XEXLoaderWV Ghidra Extension Setup

For Xbox 360 `.xex` analysis, prefer importing through a dedicated Ghidra XEX loader extension rather than treating the XEX as a raw binary when an appropriate extension is available.

Primary extension:

```text
https://github.com/zeroKilo/XEXLoaderWV
```

Use a maintained fork only when the user chooses it or the primary extension is incompatible with the installed Ghidra version. Known maintained-fork candidates should be verified from the repository/release page before use.

Checks:

```text
[ ] Ghidra version and install path are known.
[ ] Installed extension list checked for XEXLoaderWV / X360 XEX loader.
[ ] The XEXLoaderWV release ZIP matches the installed Ghidra major/minor version when releases are version-specific.
[ ] Java requirement is satisfied. XEXLoaderWV requires at least a compatible JDK level; verify against the current README/release notes.
[ ] Extension install path is recorded.
[ ] Ghidra was restarted after installation.
[ ] A test import of a lawful `default.xex` shows an Xbox 360/XEX loader option.
```

Manual install flow:

```text
1. Download the XEXLoaderWV release ZIP that matches the installed Ghidra version.
2. In Ghidra: File > Install Extensions > Add.
3. Select the XEXLoaderWV ZIP.
4. Restart Ghidra.
5. Import the legally extracted/decrypted `default.xex` and confirm the XEX loader is selected.
6. Confirm image base, sections, imports, exports, and entrypoint before analysis.
```

Automation-safe install flow:

```text
1. Query installed Ghidra version/path.
2. Query XEXLoaderWV releases or local downloaded ZIPs.
3. Select only a compatible ZIP.
4. Back up any existing extension ZIP/version notes if replacing.
5. Install via Ghidra extension mechanism or documented headless-compatible path.
6. Start Ghidra once to verify the extension appears.
```

Do not force raw-binary import unless XEXLoaderWV or a compatible loader is unavailable. If raw import is the fallback, document that image base, section boundaries, imports, and relocation behavior are lower-confidence.

### Ghidra Import Checklist for Xbox 360 XEX

After installing XEXLoaderWV and before relying on Ghidra evidence:

```text
[ ] Program imported as Xbox 360/XEX, not generic raw PPC blob unless intentionally fallback.
[ ] Correct PowerPC big-endian language/processor selected.
[ ] Image base matches XEX metadata or tool evidence.
[ ] Entrypoint identified.
[ ] Sections/memory blocks look plausible.
[ ] Imports/exports recognized when available.
[ ] Function discovery has run.
[ ] Decompiler output is treated as structural hint only.
[ ] Raw PPC disassembly and xrefs are used for final decisions.
```

### GhidraMCP Installation and Health Checks

Preferred MCP bridge/plugin:

```text
https://github.com/bethington/ghidra-mcp
```

Before installation:

```text
[ ] Ghidra is installed.
[ ] Java 21 LTS and Maven are installed for source builds.
[ ] Python 3.10+ is available.
[ ] The target XEX can be imported/opened in Ghidra.
[ ] XEXLoaderWV is installed or the raw-import fallback is documented.
```

Preferred source install, adjusting the Ghidra path:

```powershell
git clone https://github.com/bethington/ghidra-mcp.git external/ghidra-mcp
cd external/ghidra-mcp
python -m tools.setup preflight --ghidra-path "C:\ghidra_12.1_PUBLIC"
python -m tools.setup ensure-prereqs --ghidra-path "C:\ghidra_12.1_PUBLIC"
python -m tools.setup build
python -m tools.setup deploy --ghidra-path "C:\ghidra_12.1_PUBLIC"
```

Manual release ZIP install:

```text
1. Download the GhidraMCP release ZIP matching the installed Ghidra version.
2. In Ghidra: File > Install Extensions > Add.
3. Restart Ghidra.
4. Enable plugin: File > Configure > Configure All Plugins > GhidraMCP.
5. Start server: Tools > GhidraMCP > Start MCP Server.
6. Install Python bridge requirements: `python -m pip install -r requirements.txt`.
7. Run bridge: `python bridge_mcp_ghidra.py`.
```

Health checks:

```cmd
curl http://127.0.0.1:8089/check_connection
curl http://127.0.0.1:8089/get_version
```

Only proceed to agent-side MCP configuration after the Ghidra-side health check passes or a specific failure is documented.

### MCP Frontend Auto-Detection and Config Merge

When the user has Cursor, Antigravity, Codex, OpenCode, Claude Desktop, Warp, or another MCP frontend, detect the frontend before writing config.

Detection sources:

```text
[ ] User-named frontend in the prompt.
[ ] Existing repo files: `.cursor/`, `.gemini/`, `.codex/`, `opencode.json`, `opencode.jsonc`, `CLAUDE.md`, `.warp/`, `.vscode/mcp.json`.
[ ] User profile config paths.
[ ] Frontend CLI commands, if installed.
[ ] Frontend UI: "MCP", "Tools & MCP", "Manage MCP Servers", "View raw config", or "Edit Config".
```

Config merge rules:

```text
1. Prefer project-local config when the MCP server should only apply to this reverse-engineering workspace.
2. Prefer global config only when the user explicitly wants Ghidra MCP available across projects.
3. Read the existing config first.
4. Create a timestamped backup beside the file before editing.
5. Preserve JSON vs JSONC vs TOML syntax.
6. Preserve existing MCP servers.
7. Add or update one `ghidra` entry.
8. Prefer stdio bridge unless the frontend requires HTTP/SSE/streamable HTTP.
9. Use absolute paths for the Python executable and `bridge_mcp_ghidra.py` on Windows unless the frontend documents working-directory behavior.
10. Restart or reload the frontend and verify the tools appear.
```

Canonical stdio entry shape for JSON-like clients:

```json
{
  "mcpServers": {
    "ghidra": {
      "command": "python",
      "args": ["C:/path/to/ghidra-mcp/bridge_mcp_ghidra.py"]
    }
  }
}
```

Canonical HTTP entry shape when the bridge is run as streamable HTTP:

```json
{
  "mcpServers": {
    "ghidra-mcp-http": {
      "url": "http://127.0.0.1:8081/mcp"
    }
  }
}
```

### Frontend-Specific MCP Config Targets

Use the frontend's official UI/config path when possible. Treat the paths below as discovery candidates, not guaranteed truth.

| Frontend | Preferred detection/config flow | Notes |
|---|---|---|
| Cursor | Check workspace `.cursor/mcp.json`, then global Cursor MCP config. Use Cursor Settings > Tools & MCP / MCP UI when available. | JSON config with `mcpServers` is common. Preserve existing servers. |
| Antigravity | Use the editor Agent panel or Settings/Customizations flow to open "Manage MCP Servers" / "View raw config" / MCP config. | Do not guess a path if the UI exposes the raw config. Shared Gemini/Antigravity configs may exist; verify before editing. |
| Codex CLI | Check the Codex config TOML, commonly `~/.codex/config.toml`, and use the schema documented by the installed Codex version. | Codex MCP configuration is TOML-based; do not write JSON into it. |
| OpenCode | Check `opencode.json` / `opencode.jsonc` in the repo and user config locations documented by the installed OpenCode version. | OpenCode uses JSON/JSONC config. Verify the current `mcp`/tools schema before editing. |
| Claude Desktop | Use Settings > Developer > Edit Config when available; common file is `claude_desktop_config.json` under the platform app-data folder. | Restart Claude Desktop after editing. |
| Warp | Prefer Warp Settings > Agents > MCP servers or the documented file-based/Oz config path. | Warp supports several MCP connection styles; do not force stdio if Warp expects another transport. |
| VS Code-compatible clients | Check workspace `.vscode/mcp.json` or the extension's MCP settings. | Preserve extension-specific schema. |
| Other | Ask the user for the frontend name or config file path, then apply the generic backup-and-merge rule. | Do not invent config paths. |

### MCP Verification Packet

After configuration, record:

```text
Frontend:
Config path:
Backup path:
Transport: stdio / streamable-http / SSE / other
Ghidra install path:
XEXLoaderWV installed: yes/no/version/path
GhidraMCP repo/release:
Bridge command:
Ghidra health check result:
Frontend tool-list result:
Target XEX open in Ghidra:
Known limitations:
```

If the frontend can list tools, verify that Ghidra tools appear before asking it to analyze the XEX. If the frontend cannot list tools, run a harmless GhidraMCP command such as version/status before any write operation.

### GhidraMCP Safety Rules

- Keep GhidraMCP bound to loopback for normal local development.
- Do not expose the GhidraMCP server to a LAN or the internet unless authentication and file-root restrictions are explicitly configured.
- Prefer read-only evidence gathering before renaming, typing, patching, or running scripts.
- Enable arbitrary script execution only when the task requires it and the user understands the risk.
- If using write operations, document the evidence and the exact rename/type/edit performed.

### Setup Failure Triage

Common setup failures:

```text
Ghidra not found on PATH but installed elsewhere
Wrong Ghidra version for extension ZIP
XEXLoaderWV installed but Ghidra not restarted
XEX imported as raw PPC instead of XEX loader
GhidraMCP extension installed but plugin not enabled
GhidraMCP server not started inside Ghidra
Bridge running but plugin health check failing
Frontend config edited in global path while using workspace-only frontend
JSONC file rewritten as strict JSON and comments lost
TOML client accidentally given JSON MCP config
Relative bridge path fails because frontend working directory differs
Multiple `ghidra` entries competing
HTTP transport configured while bridge is running stdio only
Firewall/security tool blocks local HTTP port
```

Patch the setup layer that failed. Do not reinterpret bad Ghidra/MCP setup as a game-analysis failure.
