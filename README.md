# xbox360-decomp (Cursor Agent Skill)

Agent skill for **Xbox 360 / XBLA reverse engineering and static recompilation** — ReXGlue (Track A), XenonRecomp / XenonAnalyse (Track B), matching / full decompilation (Track C), and [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) pipeline (Track D).

This skill is a **methodology and reference layer** for the AI agent. It does **not** include game binaries, ReXGlue SDK, XenonRecomp, 360tools, or Ghidra.

Modeled after [ps2-recomp-Agent-SKILL](https://github.com/hkmodd/ps2-recomp-Agent-SKILL): lean `SKILL.md` hub, numbered `resources/`, and **persistent project memory**. This README is the **operator playbook** — how you, the human driver, should use the skill to get reliable agent behavior.

---

## How to treat the agent

**Do not treat it as a chatbot. Treat it as a junior reverse engineer working in your port repo.**

1. **It should be evidence-driven.** The agent must verify hook APIs and config fields from your local ReXGlue SDK (`file:line`), raw PPC/Ghidra, and build logs — not invent macros or guest semantics.
2. **It has persistent memory.** The agent creates or maintains **`XBOX360_PROJECT_STATE.md`** in your **port project root** (from `scripts/project-state-template.md`). That file is its external hippocampus: title, track, XEX hash, image base, active blocker, verified build/run commands, crash table. Pause Monday, open a new chat Thursday, point at the state file — the agent resumes from recorded facts instead of guessing.
3. **It has circuit breakers.** Same guest crash twice → `resources/12-stuck-cross-recomp.md` **before** another patch. No blind `git apply` of all five SDK patches. No overwriting `.cursor/mcp.json` without backup. If the agent loops without new evidence, tell it to stop and follow the stuck protocol.

---

## What you get

| Included in the skill | Not included (you provide separately) |
|----------------------|----------------------------------------|
| `SKILL.md` — PS2-style hub (decision router, boot sequence, prohibitions) | Clones of [rexglue-sdk](https://github.com/rexglue/rexglue-sdk), [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated); XenonRecomp only for Track B |
| `resources/` — numbered deep refs (tracks, Ghidra, triage, guardrails) | `default.xex`, STFS/LIVE packages, extracted assets (you must own the game) |
| `scripts/project-state-template.md` → `XBOX360_PROJECT_STATE.md` | Persistent session memory in your port repo |
| `evals/evals.json` (development only) | Ghidra, CMake, Clang/MSVC, Python per your track |

---

## Prerequisites (your PC setup)

For the agent to work reliably, have these ready **before** the first session:

1. **Cursor** (or compatible agent) with **xbox360-decomp** skill installed — see [Installation](#installation) below.
2. **Lawful game files** — you must own the title. Do not commit or redistribute retail `default.xex`, STFS/LIVE packages, or extracted assets.
3. **Visual Studio** (Desktop development with C++) + **CMake** + **Ninja** — typical for ReXGlue/Xenon native builds. Use a **x64 Developer PowerShell** or Developer Command Prompt when MSVC is required (`resources/01-dev-environment.md`).
4. **Python 3.8+** — for [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) (`pip install -r requirements.txt`) on Track D / XBLA extract.
5. **ReXGlue SDK** — clone/build per your port (`thirdparty/rexglue-sdk` or `tools/rexglue-sdk`). Optional patches `0001`–`0005` only per symptom (`resources/25-rexglue-sdk-patches.md`).
6. **Ghidra 12.x** + **XEXLoaderWV** + **GhidraMCP** — for static analysis and MCP-driven disasm/decompile:
   - Open `default.xex` in Ghidra with XEXLoaderWV; confirm image base (often `0x82000000` — **verify**, do not guess).
   - Start GhidraMCP in CodeBrowser; merge (do not blind-overwrite) `.cursor/mcp.json` per `resources/07-ghidra-mcp.md`.
7. **Track-specific clones** — [360toolsUpdated](https://github.com/DohmBoy64Bit/360toolsUpdated) (Track D); [XenonRecomp](https://github.com/hedge-dev/XenonRecomp) (Track B only).

Full toolchain detail: `resources/01-dev-environment.md` and `resources/db-xbox360-index.md`.

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
| `SKILL.md` + `resources/` + `scripts/` | Agent playbook for 360 RE / recomp |
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

## Start a session

Open your **port workspace** in Cursor (the repo with `assets/`, `config/`, `generated/`, or an empty folder you are bootstrapping). Attach or enable the **xbox360-decomp** skill, then paste one of the blocks below.

### Universal starter prompt

Copy-paste this entire block to activate the skill. Works for a new project, a half-done port, or resuming after context loss.

```text
Read the xbox360-decomp skill SKILL.md and execute §2 BOOT SEQUENCE.
Also read these boot references before changing anything:
  resources/11-operational-phases.md
  resources/22-decisional-brain.md
Do NOT proceed until you have read SKILL.md §2 and those two files.

Then execute this startup sequence:

1. INSPECT my workspace. Look for:
   - XBOX360_PROJECT_STATE.md (persistent memory from a previous session)
   - assets/default.xex or extracted/default.xex (or other XEX path)
   - config/*_config.toml, manifest, switch tables
   - build/, CMakeCache.txt, generated/ (tool output — do not patch first)
   - thirdparty/rexglue-sdk or tools/rexglue-sdk (ReXGlue SDK)
   - 360toolsUpdated clone (Track D)
   - docs/ ledgers (toolchain.md, regression_log.md, xbla_package_ledger.md)
   ⚠️ DANGER: Do NOT bulk-read or list every file under generated/.
   Use targeted grep/read on specific guest PCs or filenames the crash log names.

   NOTE: Tool clones and game packages may live OUTSIDE this workspace.
   If you cannot find default.xex, ReXGlue SDK, or the LIVE package path, ASK me.

2. PERSISTENT MEMORY:
   - If XBOX360_PROJECT_STATE.md exists → read it fully and resume.
   - If missing → create it from scripts/project-state-template.md in the skill.

3. If a build directory exists, READ CMakeCache.txt and report:
   - Generator (Ninja? Visual Studio?)
   - Compiler (clang-cl? MSVC?)
   - Build type (Release? Debug?)
   Then rate the config (Optimal / Acceptable / Critical) and suggest
   improvements if needed. Do NOT change CMake or apply SDK patches without my OK.

4. PHASE 0 GATE: If XEX SHA-256 or image base is unknown and we need guest
   addresses, complete Phase 0 in 11-operational-phases.md before rexglue
   codegen or runtime patches.

5. ASK me for anything you still do not know. Typical questions:
   - What game / title ID? (XBLA LIVE package path if applicable)
   - Which track? (A ReXGlue / B XenonRecomp / C matching / D 360toolsUpdated)
   - Where is game_data_root for --game_data_root at runtime?
   - Current blocker? (guest PC, log line, unregistered VA, TDR, etc.)

6. REPORT: What you found, current phase, track, and the next concrete step.
   Then wait for my go-ahead before extract, codegen, or patches.
```

> **Why this works:** Read first → detect second → ask third → act last. The agent cannot skip the skill boot sequence, cannot assume image base `0x82000000`, and cannot jump to `rexglue codegen` on an empty folder without Phase 0.

### Quick resume (new chat, same project)

You already ran the universal starter once; context is full or you opened a new chat:

```text
Read the xbox360-decomp skill SKILL.md §2 BOOT SEQUENCE.
Read XBOX360_PROJECT_STATE.md in this workspace to recover our progress.
Re-read resources/12-stuck-cross-recomp.md if the same crash happened twice.
Resume from the active blocker and verified commands in the state file.
Report the next step and wait for my OK.
```

### Fresh game (you know paths and track)

Skip Q&A when you can fill in the blanks:

```text
Read the xbox360-decomp skill SKILL.md §2 and resources/11-operational-phases.md.
I'm porting [GAME NAME] ([XBLA title ID if known], e.g. 58410B2B).
Track: [A / B / C / D]
Package or XEX: [ABSOLUTE PATH to LIVE/.con or default.xex]
Port project root: [ABSOLUTE PATH]
game_data_root: [ABSOLUTE PATH to extracted assets folder]
ReXGlue SDK: [ABSOLUTE PATH to rexglue-sdk checkout]
(If any path is "same as workspace", say so.)

Create or update XBOX360_PROJECT_STATE.md from scripts/project-state-template.md.
Start at Phase 0 — record SHA-256 and image base (xex_info.py), detect build
config, report before extract/codegen/patches.
```

### Example task prompts (after boot)

Once booted, you can ask targeted questions:

- *“Guest 0x82701234 unregistered VA — same crash twice after nulling a hook.”*
- *“Set up Ghidra MCP for default.xex — verify image base, don't overwrite mcp.json.”*
- *“Guardian Heroes LIVE package — 360toolsUpdated order before rexglue codegen.”*
- *“Should I git apply all five ReXGlue SDK patches?”* (expect: symptom-driven, not blind apply-all)

The agent should verify commands and APIs from **your local SDK/tool trees**, not invent flags or hook macros.

---

## How to collaborate (human-in-the-loop)

- **Monitor `XBOX360_PROJECT_STATE.md`** — keep it open in split view. The agent should update phase, blocker, crash table, and learned patterns after verified changes. If it hallucinated a path or SDK API, **edit the markdown directly**; the agent reads your correction on the next refresh.
- **Keep Ghidra ready for MCP** — you open `default.xex`, run auto-analysis, start GhidraMCP; the **agent** drives disasm/xrefs via MCP. You do not need to manually export everything unless MCP is down.
- **Own the lawful assets** — provide package/XEX paths when asked; the skill will not source pirated binaries.
- **Context degradation** — long chats → agent asks dumb questions or forgets track D vs legacy XenonRecomp. **Stop.** New chat + **Quick resume** prompt above.
- **Let builds finish** — when the agent runs `cmake --build` or `rexglue codegen`, let it read the full log. Interrupting mid-build loses the evidence packet it needs for the next fix.

---

## Troubleshooting

| Problem | Tell the agent |
|---------|----------------|
| Jumps straight to codegen on an empty folder | *“Follow SKILL.md §2 boot — create XBOX360_PROJECT_STATE.md and Phase 0 first.”* |
| Nulls the same hook / same patch twice | *“Stuck loop — read resources/12-stuck-cross-recomp.md before another patch.”* |
| Invents `REX_HOOK_*` or config fields | *“Quote ReXGlue SDK file:line from my thirdparty/rexglue-sdk checkout.”* |
| Recommends `extract_pe` + XenonRecomp on 360toolsUpdated | *“Track D is ReXGlue-native — read resources/06-track-360tools.md current path.”* |
| Overwrites `.cursor/mcp.json` | *“Backup and merge per resources/07-ghidra-mcp.md.”* |
| Forgets title / track / blocker | *“Context refresh — read XBOX360_PROJECT_STATE.md.”* |
| Applies all five SDK patches blindly | *“Read resources/25-rexglue-sdk-patches.md — symptom + git apply --check only.”* |

---

### Track overview

| Track | Stack |
|-------|--------|
| **A** | ReXGlue codegen from XEX |
| **B** | XenonRecomp / XenonAnalyse + project runtime |
| **C** | Ghidra / matching / decomp.me → handwritten C++ |
| **D** | 360toolsUpdated extract → `rexglue init` / `codegen` → optional `templates/advanced/` + SDK patches |

Track D (current) is ReXGlue-native — no XenonRecomp step. Legacy sp00nznet/360tools XenonRecomp path is in `resources/06-track-360tools.md` (§ Legacy).

---

## Skill layout (v1.4+ — PS2-style)

Modeled after [ps2-recomp-Agent-SKILL](https://github.com/hkmodd/ps2-recomp-Agent-SKILL): lean hub + numbered `resources/`.

```
xbox360-decomp/
├── README.md
├── SKILL.md                 # §1 router, §2 boot, §3–§10 constraints
├── scripts/
│   └── project-state-template.md   # → XBOX360_PROJECT_STATE.md in port repo
├── evals/evals.json         # dev only (not in .skill package)
└── resources/
    ├── 01-dev-environment.md
    ├── 02-xbla-stfs.md
    ├── 03-track-rexglue.md … 06-track-360tools.md
    ├── 07-ghidra-mcp.md … 09-original-game-evidence.md
    ├── 10-agent-guardrails.md
    ├── 11-operational-phases.md
    ├── 12-stuck-cross-recomp.md … 24-ledgers-confidence.md
    ├── 22-decisional-brain.md
    └── db-xbox360-index.md   # master router
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
| **This Git repo** | Source for `SKILL.md`, `resources/`, README |
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

Optional: `xex_info.py` / `parse_xex_imports.py` before codegen; copy `templates/advanced/` after init; VdSwap QPC fix in `docs/speed-fix.md`. Legacy XenonRecomp path (`extract_pe`, `post_codegen`) is **sp00nznet/360tools only** — see `resources/06-track-360tools.md`.

---

## Development / evals

Built with **skill-creator**. Test prompts: `evals/evals.json`. Benchmark runs live in `xbox360-decomp-workspace/iteration-N/` (not shipped in `.skill`).

| Iteration | Skill version | Result |
|-----------|---------------|--------|
| 1 | v1.2.0 | 100% pass both configs; Track D eval still used legacy XenonRecomp order |
| 2 | v1.3.0 | 100% pass both configs; **Track D eval fixed** — `extract_stfs` → `rexglue codegen`; stuck eval cites SDK `file:line` |
| 3 | v1.4.1 | 6 evals; with_skill **100%** vs without_skill **50%** (+50pp delta) |
| 4 | v1.4.2 | 7 evals inline (+ optional SDK patches) |
| 5 | v1.4.2 | **7 evals live subagents (Auto)** — with_skill **100%** vs without_skill **82%** (+18pp) |
| 6 | v1.4.3 | **7 evals, isolated baselines** — with_skill **100%** vs without_skill **75%** (+25pp); session-boot without_skill **0/3** |

Review HTML: `xbox360-decomp-workspace/iteration-6/review.html` (compare vs iteration-5).

To repackage after edits:

```powershell
robocopy xbox360-decomp _pkg\xbox360-decomp /E /XD .git dist
python -m scripts.package_skill _pkg\xbox360-decomp release
```

---

## Changelog

### v1.4.4
- README: PS2-style operator playbook — persistent memory (`XBOX360_PROJECT_STATE.md`), universal/quick-resume/fresh-game starter prompts, collaboration + troubleshooting tables

### v1.4.3
- Eval iteration-6: isolated without_skill baselines (must not read skill dir); **100% / 75%** with_skill vs without_skill
- Document **0005** cumulative-rollup caveat in `resources/25-rexglue-sdk-patches.md` (`git apply --check` often fails on clean SDK)

### v1.4.2
- Bundled optional ReXGlue SDK patches: `patches/0001`–`0005` + `rexglue_patches_audit.md`
- New `resources/25-rexglue-sdk-patches.md` — source-of-truth gate; apply per symptom/title, not all five blindly
- Eval iteration-4 adds `optional-sdk-patches` eval

### v1.4.1
- Parity pass: restore v1.3.1 hub content into `resources/` where PS2-style files were too thin (`11`, `22`, `23`, `24`, `10` §0) without bloating `SKILL.md`

### v1.4.0
- **PS2-style refactor** ([hkmodd/ps2-recomp-Agent-SKILL](https://github.com/hkmodd/ps2-recomp-Agent-SKILL)): lean `SKILL.md` hub with decision router, boot sequence, prohibitions, build gate, state protocol
- `references/` → numbered `resources/` (01–24 + `db-xbox360-index.md`); feature parity preserved
- New: `10-agent-guardrails.md`, `11-operational-phases.md`, `22-decisional-brain.md`, `23-xenon-execution-discipline.md`, `24-ledgers-confidence.md`
- New: `scripts/project-state-template.md` for `XBOX360_PROJECT_STATE.md`

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
