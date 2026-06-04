## Asset and Legal Boundaries

Keep proprietary game data out of source control unless the user explicitly approves a lawful private repository policy.

Recommended `.gitignore` entries:

```gitignore
# Build output
build/
out/
CMakeFiles/
CMakeCache.txt
cmake-build-*/

# Local environment
.venv/
workspace_env/
__pycache__/
*.pyc

# Proprietary Xbox/Xbox 360 binaries and extracted assets
assets/default.xex
assets/pe_image.bin
assets/game_files/
game/
extracted/
Content/
content/
*.live
*.stfs
*.con
*.pirs
*.xex
*.iso
*.dvd
*.xbe
*.wad
*.bin
*.xcp
*.xzp
*.pak
*.bik
*.wmv

# Local logs and scratch output
*.log
*.tmp
```

Keep these under version control when useful:

```text
config/
source/
scripts/
docs/
CMakeLists.txt
AGENTS.md
README.md
small generated metadata, if policy allows it
```

Keep these separate from committed source:

```text
default.xex
ISO/DVD dumps
decrypted assets
proprietary archives
keys
SDK headers/libraries
large generated build output unless the project explicitly tracks it
```
