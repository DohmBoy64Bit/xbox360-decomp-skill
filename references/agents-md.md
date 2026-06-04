## Project Documentation / AGENTS.md

Create or update a project-local `AGENTS.md` once the project has repeatable commands, known constraints, or verified conventions.

Document only verified facts:

- Project purpose and lawful-use boundary.
- Selected track: ReXGlue recomp, XenonRecomp/XenonAnalyse recomp, matching/full decomp, or hybrid.
- ReXGlue, XenonRecomp, XenonAnalyse, XenosRecomp, compiler, and related tool versions/commits as applicable.
- XEX path, hash, image base, and revision assumptions.
- Extracted game-data root and asset-boundary policy.
- Generated vs handwritten folders.
- Build, generation, analysis, run, and verification commands.
- Tool versions or pinned commits.
- Ghidra/XEXLoaderWV/GhidraMCP setup requirements, including Ghidra version, XEXLoaderWV extension path/version, GhidraMCP extension path/version, bridge command, MCP client config location, frontend name, and transport.
- Frontend rule/instruction file path, such as `AGENTS.md`, `.cursor/rules/*.mdc`, `.cursor/skills/xbox360-decomp/SKILL.md`, `.gemini/AGENTS.md`, `CLAUDE.md`, or another verified project-specific instruction file.
- Address mapping conventions for guest virtual addresses, XEX image offsets, generated code, guest memory, runtime assets, and host pointers.
- Hook, runtime, renderer, audio, input, filesystem, save, and crash-analysis conventions.
- Known failure triage and next expected failure points.

Do not invent documentation. If evidence is missing, write a clear TODO naming the exact artifact needed.

Suggested minimal `AGENTS.md` shape:

```md
# Agent Instructions
