# xbox360-decomp (Cursor Agent Skill)

Agent skill for **Xbox 360 / XBLA reverse engineering and static recompilation** — ReXGlue (Track A), XenonRecomp / XenonAnalyse (Track B), matching / full decompilation (Track C), and [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) pipeline (Track D).

This skill is a **methodology and reference layer** for the AI agent. It does **not** include game binaries, ReXGlue SDK, XenonRecomp, 360tools, or Ghidra.

---

## What you get

| Included in the skill | Not included (you provide separately) |
|----------------------|----------------------------------------|
| `SKILL.md` — track selection, evidence rules, stuck loop | Clones of [rexglue-sdk](https://github.com/rexglue/rexglue-sdk), [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated); XenonRecomp only for Track B |
| `references/` — Ghidra MCP, tracks, debug triage, XBLA/STFS | `default.xex`, STFS/LIVE packages, extracted assets (you must own the game) |
| `evals/evals.json` (development only) | Ghidra, CMake, Clang/MSVC, Python per your track |

---

## Requirements

- **Cursor** (or compatible agent) with skills support
- **Lawful game files** — do not commit or redistribute retail XEX/assets
- Tooling depends on track: ReXGlue SDK, [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated), Ghidra 12.x + XEXLoaderWV + GhidraMCP; XenonRecomp for Track B only

See `references/dev-environment.md` and per-track files under `references/`.

---

## Installation

### Option 1: Skill folder

Copy or symlink this directory:

| Scope | Path |
|-------|------|
| Personal (Cursor) | `~/.cursor/skills/xbox360-decomp/` |
| Agents (Codex-style) | `~/.agents/skills/xbox360-decomp/` |
| Project (repo-only) | `.cursor/skills/xbox360-decomp/` |

The folder must contain `SKILL.md` at its root.

### Option 2: Download `xbox360-decomp.skill` (recommended)

**[GitHub Releases](https://github.com/DohmBoy64Bit/xbox360-decomp-skill/releases)** ship a pre-built **`xbox360-decomp.skill`** file.

| What it is | What it is for |
|------------|----------------|
| ZIP archive (`.skill` extension) | One-file install in Cursor |
| `SKILL.md` + `references/` | Agent playbook for 360 RE / recomp |
| Does **not** include upstream toolkits | Clone ReXGlue SDK and 360toolsUpdated separately |

**Steps:**

1. Open [Releases](https://github.com/DohmBoy64Bit/xbox360-decomp-skill/releases) and download **`xbox360-decomp.skill`** from the latest tag.
2. Install via Cursor’s skill import UI, or unpack into `~/.cursor/skills/xbox360-decomp/`.
3. Clone the toolchains your track needs (see **Related skills** below).

`evals/` is omitted from the `.skill` package (development-only).

### Option 3: Build `.skill` yourself

From [skill-creator](https://github.com/anthropics/skills) (or your local copy):

```bash
python -m scripts.package_skill path/to/xbox360-decomp path/to/output
```

Produces `xbox360-decomp.skill`. The packager skips `evals/` by default.

---

## How to use in Cursor

1. Open a workspace with your 360 port or analysis tree (and tool clones on disk).
2. Start an **Agent** chat.
3. Attach the **xbox360-decomp** skill or ask a matching task, for example:
   - *“Guest 0x82701234 unregistered VA — same crash twice after nulling a hook.”*
   - *“Set up Ghidra MCP for default.xex, image base 0x82000000.”*
   - *“Guardian Heroes LIVE package — 360toolsUpdated script order before rexglue codegen.”*

The agent should verify commands and APIs from **your local SDK/tool trees**, not invent flags or hook macros.

### Track overview

| Track | Stack |
|-------|--------|
| **A** | ReXGlue codegen from XEX |
| **B** | XenonRecomp / XenonAnalyse + project runtime |
| **C** | Ghidra / matching / decomp.me → handwritten C++ |
| **D** | 360toolsUpdated extract → `rexglue init` / `codegen` → optional `templates/advanced/` + SDK patches |

Track D (current) is ReXGlue-native — no XenonRecomp step. Legacy sp00nznet/360tools XenonRecomp path is documented in `references/track-360tools.md`.

---

## Skill layout

```
xbox360-decomp/
├── README.md
├── SKILL.md
├── evals/evals.json
└── references/
    ├── track-rexglue.md
    ├── track-xenon.md
    ├── track-full-decomp.md
    ├── track-360tools.md
    ├── ghidra-mcp.md
    ├── stuck-cross-recomp.md
    ├── debug-triage.md
    └── …
```

---

## Related skills

| Skill | When |
|-------|------|
| **[360tools-skill](https://github.com/DohmBoy64Bit/360tools-skill)** | Extraction + ReXGlue quickstart (may lag 360toolsUpdated — verify repo) |
| **[xboxrecomp-skill](https://github.com/DohmBoy64Bit/xboxrecomp-skill)** | Original Xbox `default.xbe` (x86), not Xenon |
| **windows-game-matching-decomp** | Win32 / Unity / Unreal PE matching without 360 context |

---

## Skill vs `.skill` vs toolkits

| Artifact | Purpose |
|----------|---------|
| **This Git repo** | Source for `SKILL.md`, references, README |
| **Release `xbox360-decomp.skill`** | Pre-built ZIP for Cursor install |
| **Upstream toolkits** | ReXGlue SDK, 360toolsUpdated — always separate |

---

## Track D quick pipeline (360toolsUpdated)

From [DohmBoy64Bit/360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) — **ReXGlue-native** (no `extract_pe`, no XenonRecomp):

```bash
# 360toolsUpdated repo root
pip install -r requirements.txt
python tools/extract_stfs.py path/to/LIVE_PACKAGE extracted/

git clone --recursive https://github.com/rexglue/rexglue-sdk.git tools/rexglue-sdk
# optional: apply patches/0001-0005 per patches/rexglue_patches_audit.md

rexglue init --app_name mygame --app_root my_project/
cp extracted/default.xex my_project/assets/
rexglue codegen mygame_config.toml

cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

Optional: `xex_info.py` / `parse_xex_imports.py` before codegen; copy `templates/advanced/` after init; VdSwap QPC fix in `docs/speed-fix.md`. Legacy XenonRecomp path (`extract_pe`, `post_codegen`) is **sp00nznet/360tools only** — see `references/track-360tools.md`.

---

## Development / evals

Built with **skill-creator**. Test prompts: `evals/evals.json`. Benchmark runs live in `xbox360-decomp-workspace/iteration-N/` (not shipped in `.skill`).

| Iteration | Skill version | Result |
|-----------|---------------|--------|
| 1 | v1.2.0 | 100% pass both configs; Track D eval still used legacy XenonRecomp order |
| 2 | v1.3.0 | 100% pass both configs; **Track D eval fixed** — `extract_stfs` → `rexglue codegen`; stuck eval cites SDK `file:line` |

Review HTML: `xbox360-decomp-workspace/iteration-2/review.html` (compare vs iteration-1).

To repackage after edits:

```powershell
robocopy xbox360-decomp _pkg\xbox360-decomp /E /XD .git dist
python -m scripts.package_skill _pkg\xbox360-decomp release
```

---

## Changelog

### v1.3.1
- README: Track D quick pipeline, eval iteration summary, repackage notes

### v1.3.0
- **Track D rewrite** for [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) ReXGlue-native workflow
- Removed XenonRecomp/`extract_pe`/`post_codegen` from current Track D; added SDK patches `0001`–`0005`, `templates/advanced/`
- Iteration-2 evals validate new pipeline

### v1.2.0
- Initial public release; optimized trigger description

---

## License and legal

- Skill text: follow your project policy; upstream toolkits have their own licenses.
- You must **own** any game you extract or recompile. No piracy, keys, or DRM-bypass instructions.

---

## Links

- **Releases (`.skill` download):** https://github.com/DohmBoy64Bit/xbox360-decomp-skill/releases  
- ReXGlue SDK: https://github.com/rexglue/rexglue-sdk  
- 360toolsUpdated: https://github.com/DohmBoy64Bit/360toolsUpdated  
- XenonRecomp (Track B): https://github.com/hedge-dev/XenonRecomp  
- Legacy 360tools: https://github.com/sp00nznet/360tools  
- Xbox Dev Wiki: https://xboxdevwiki.net/  
