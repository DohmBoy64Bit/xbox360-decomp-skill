## Frontend Rule / Agent Instruction File Setup

Use this section when setting up a project for Cursor, Antigravity, Codex, OpenCode, Claude Code/Desktop, Warp, VS Code-compatible agents, or another AI frontend. The goal is to keep future agents constrained by the same project rules without duplicating this whole skill.

Create or update the active frontend's rule/instruction file after the selected track, asset boundary, toolchain, and verified commands are known. Keep this file short and project-specific.

Discovery candidates:

| Frontend / agent | Rule or instruction file target | Notes |
|---|---|---|
| Generic / unknown | Root `AGENTS.md` | Safe default for repository-wide instructions. |
| Cursor | Root `AGENTS.md`, `.cursor/rules/*.mdc`, and `.cursor/skills/xbox360-decomp/SKILL.md` | Do not replace existing Cursor rules; keep `AGENTS.md` short and project-specific; use this skill for the long workflow. Stuck-loop rules in `.cursor/rules/` (e.g. cross-recomp before retry) override generic advice when both apply. |
| Antigravity / Gemini-style agents | Nearest applicable `AGENTS.md` and/or `.gemini/AGENTS.md` when the project uses it | Follow the frontend's active instruction discovery rules instead of inventing a path. |
| Codex | Root `AGENTS.md`; Codex config is for tool/MCP settings unless project docs say otherwise | Keep persistent repo behavior in `AGENTS.md`, not only in `~/.codex/config.toml`. |
| OpenCode | Root `AGENTS.md` plus `opencode.json` / `opencode.jsonc` only for tool/config wiring | Do not put long project policy only in tool config. |
| Claude Code / Claude Desktop | `CLAUDE.md` when the project uses Claude instructions; MCP config remains separate | Keep Claude-specific instructions consistent with `AGENTS.md`. |
| Warp | Workspace/project agent instructions if supported; otherwise root `AGENTS.md` | Verify Warp's current rule/config mechanism before editing. |
| VS Code-compatible agents | Existing workspace agent-instruction file, if present, plus root `AGENTS.md` | Preserve extension-specific schema and avoid conflicting copies. |

Minimum content for the rule/instruction file:

```text
Project track: ReXGlue / XenonRecomp-XenonAnalyse / matching-full decomp / hybrid.
Legal boundary: do not commit or request retail game binaries, decrypted assets, keys, SDK files, or proprietary data.
Evidence rule: do not invent commands, config fields, hook APIs, generated paths, function boundaries, or instruction semantics.
Research order: local project/tool source and generated output first, then logs/GhidraMCP/user artifacts, then official docs, then permitted public references, then labeled inference.
Ghidra rule: use XEXLoaderWV-backed import when available; use GhidraMCP first for functions, xrefs, disassembly, vtables, switch tables, and crash mapping.
Generated-code rule: do not patch generated output first; prefer selected-tool config, hooks, runtime glue, or documented source replacements.
Documentation rule: keep address, asset, regression, toolchain, compiler-matching, and Ghidra/MCP setup docs current.
Build/run rule: list only verified local commands and paths.
```

Do not copy the full skill into frontend rule files. The rule file should be a compact project-local contract that points agents toward the verified docs, ledgers, and setup files.
