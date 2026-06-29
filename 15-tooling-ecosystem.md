# Module 15 — Tooling & Ecosystem (Comprehensive: Virtual Environments, Package Management, Type Checking, Linting, Testing, CI/CD, Pre-commit, Docker)

## Table of Contents

- [1. Virtual Environments — Complete Deep Dive (venv, pipenv, conda, poetry, uv)](#1-virtual-environments--complete-deep-dive-venv-pipenv-conda-poetry-uv)
- [2. Package Management — Complete Deep Dive (pip every flag, pip-tools, poetry exhaustive, pdm, uv)](#2-package-management--complete-deep-dive-pip-every-flag-pip-tools-poetry-exhaustive-pdm-uv)
- [3. Build Systems — Poetry & pdm Exhaustive Configuration (vs npm scripts)](#3-build-systems--poetry--pdm-exhaustive-configuration-vs-npm-scripts)
- [4. Type Checking — mypy Exhaustive Reference (every setting, plugins, strict mode)](#4-type-checking--mypy-exhaustive-reference-every-setting-plugins-strict-mode)
- [5. Linting & Formatting — ruff/black Exhaustive (all rules, formatter, isort, bandit)](#5-linting--formatting--ruffblack-exhaustive-all-rules-formatter-isort-bandit)
- [6. Testing — pytest Complete Ecosystem (30+ plugins, hypothesis, coverage, xdist)](#6-testing--pytest-complete-ecosystem-30-plugins-hypothesis-coverage-xdist)
- [7. Pre-commit Hooks — Exhaustive Reference (every hook type, repos, local hooks)](#7-pre-commit-hooks--exhaustive-reference-every-hook-type-repos-local-hooks)
- [8. Docker for Python Development](#8-docker-for-python-development)
- [9. CI/CD Patterns — GitHub Actions for Python](#9-cicd-patterns--github-actions-for-python)
- [10. Dev Container Setup](#10-dev-container-setup)
- [11. Dependency Auditing & Security (pip-audit, safety)](#11-dependency-auditing--security-pip-audit-safety)
- [12. Version Pinning Strategies](#12-version-pinning-strategies)
- [13. Code Review Checklists for Python Projects](#13-code-review-checklists-for-python-projects)
- [14. Complete Tooling Comparison Matrix (TypeScript → Python)](#14-complete-tooling-comparison-matrix-typescript--python)
- [15. Poetry vs pdm vs uv — Complete Benchmark Comparison](#15-poetry-vs-pdm-vs-uv--complete-benchmark-comparison)
- [16. Quizzes with Answers (25+)](#16-quizzes-with-answers-25)
- [17. Exercises with Solutions (15+)](#17-exercises-with-solutions-15)

---

## 1. Virtual Environments — Complete Deep Dive (venv, pipenv, conda, poetry, uv)

### TypeScript/Node.js Module Resolution vs Python Package Isolation

In TypeScript/Node.js, all packages live in one shared `node_modules` directory at the project root. Each project has its own `node_modules`, but they all share the same Node.js interpreter. If you run `npm install express` in project A, and then `npm install react` in project B, each project gets its own isolated copy — but both projects use the same global Node.js binary.

Python takes isolation much further. A Python virtual environment (venv) creates a **completely separate Python interpreter** along with its own `site-packages` directory. Two venvs on the same machine can run different Python versions simultaneously — this is impossible in Node.js without tools like `nvm`.

```
TypeScript/Node.js:
├── project-a/
│   ├── node_modules/
│   │   ├── express/
│   │   └── react/
│   └── package.json
└── project-b/
    ├── node_modules/
    │   ├── vue/
    │   └── typescript/
    └── package.json
All projects share ONE Node.js interpreter (nvm can switch between versions).

Python venv:
├── project-a/
│   ├── .venv/
│   │   ├── bin/python3.12        ← completely separate Python executable!
│   │   ├── lib/site-packages/
│   │   │   ├── fastapi/
│   │   │   └── pydantic/
│   │   └── pyvenv.cfg            ← records the Python version used to create this venv
│   └── pyproject.toml
└── project-b/
    ├── .conda-envs/data-science  ← Conda environment with Python 3.11
    │   ├── bin/python3.11
    │   └── lib/site-packages/
    │       ├── numpy/
    │       └── pandas/
    └── pyproject.toml
Each project can use DIFFERENT Python versions! This is what Node.js requires nvm for.
```

| Concept | TypeScript/Node.js | Python venv | Why It Matters for TS Devs |
|---------|-------------------|-------------|----------------------------|
| **Isolation level** | Packages isolated in `node_modules/`; interpreter shared globally. `nvm` required to switch Python versions. | Complete environment isolation: separate interpreter + all packages. Different projects can use different Python versions simultaneously! | Node.js developers need `nvm` to manage multiple Node versions; Python's venv manages BOTH the interpreter AND packages per-project, making it more self-contained than Node's model. |
| **Interpreter versioning** | Managed by `nvm`/`fnm` globally or via `.nvmrc` file. All projects in a terminal share the same Node version. | Each venv records its Python version in `pyvenv.cfg`. You can have project-A with Python 3.12 and project-B with Python 3.10 side by side! | In TS, switching Node versions requires `nvm use 20`; in Python, you activate a different venv — the interpreter version is baked into the environment itself. |
| **Dependency location** | All packages in `node_modules/` (flat layout with npm, nested with pnpm). | Packages in `<venv>/lib/pythonX.Y/site-packages/`. A completely separate directory tree from your system Python! | Like how TypeScript's `node_modules/` keeps packages separate from global npm; Python's venv goes further by also isolating the interpreter. |
| **Path isolation** | `require()` resolves from project's `node_modules/` upward (module resolution algorithm). | Python's `sys.path` is modified to include the venv's `site-packages/` directory when activated. The system Python is completely hidden! | Both ecosystems use a resolution algorithm that prioritizes local over global; but Python's activation literally replaces `sys.executable` with the venv's binary. |
| **Global packages** | `npm install -g <package>` installs to `~/.npm/global`. Conflicts are possible between projects needing different versions. | `pip install --user <package>` installs to `~/.local/lib/pythonX.Y/site-packages/`. But with venv, you rarely need globals! | In TS, global tools (npx, eslint, prettier) can conflict; Python's venv philosophy is "never use globals" — each project manages its own tool versions. |
| **Lock file** | `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` generated automatically by the package manager. | No lock file with plain venv + pip. You need Poetry (`poetry.lock`) or pip-tools (`Pipfile.lock`) for deterministic installs. | TypeScript gets lock files for free with npm; Python requires an explicit tool (Poetry/pdm) to get the same guarantee. This is the biggest workflow gap. |
| **Cross-platform** | Works identically on Windows/macOS/Linux (with minor path separator differences). | venv works everywhere but activation scripts differ: `.venv\Scripts\activate` (Windows) vs `source .venv/bin/activate` (Unix). | The concept is identical but the activation command differs by OS — Python provides a cross-platform solution via `python -m venv .venv`. |
| **Activation effect** | No activation needed. Just `npm install` and `node main.js` works anywhere. | Must activate before installing or running: `source .venv/bin/activate` modifies `PATH` to prioritize the venv's Python over system Python! | This is the most foreign concept for TS devs — Python literally needs you to "enter" the environment before using it, whereas Node.js just works immediately after install. |

### Complete venv Setup (vs npx/npm)

```bash
# ========================================
# TypeScript: npm workflow comparison
# ========================================
npm init -y                                  # Create package.json
npm install express zod                      # Install production deps
npm install -D jest eslint typescript        # Install dev deps (-D = --save-dev)
npx tsc --init                               # Initialize TypeScript config
node src/index.js                            # Run your application

# ========================================
# Python: Equivalent venv workflow (from scratch)
# ========================================

# Step 1: Create project directory
mkdir my-python-app && cd my-python-app

# Step 2: Create virtual environment (like npm init but for the entire Python installation!)
python -m venv .venv                         # Creates a complete isolated Python environment!
                                           # Unlike Node.js where node_modules only holds packages,
                                           # venv holds a SEPARATE PYTHON INTERPRETER too!

# Step 3: Activate the virtual environment
source .venv/bin/activate                    # Linux/macOS — adds .venv/bin to the FRONT of PATH!
# On Windows PowerShell:    .venv\Scripts\Activate.ps1
# On Windows CMD:         .venv\Scripts\activate.bat

# Step 4: Verify activation (note how python now points to the venv!)
which python                                 # Should show: /path/to/my-python-app/.venv/bin/python
python --version                             # Shows the exact Python version used to create this venv!
pip --version                                # pip is also from the venv now

# Step 5: Install packages (like npm install)
pip install fastapi pydantic uvicorn         # Production dependencies — installs into .venv/lib/site-packages/
pip install pytest mypy ruff black           # Development dependencies

# Step 6: Save your dependency list (npm does this automatically in package.json!)
pip freeze > requirements.txt                # Exports ALL installed packages with exact versions
                                          # Like manually reading package.json after npm install!

# Step 7: Reproduce the environment on another machine
cp requirements.txt /path/to/other-machine/
python -m venv .venv
source .venv/bin/activate                    # Activate first, THEN install!
pip install -r requirements.txt              # Like npm ci (exact version install)!

# Step 8: Deactivate when done
deactivate                                   # Removes .venv/bin from PATH; restores system Python
```

### Virtual Environment Tools — Comprehensive Comparison

Every modern Python project needs virtual environments. Here are all the tools available, with exhaustive details:

#### 1. `venv` (Standard Library — The Foundation)

Built into Python 3.3+. Always available without any installation. Creates a basic isolated environment.

```python
# Create venv from the command line
python -m venv .venv                                # Standard way to create

# Programmatic creation (rare but possible in CI/CD)
import venv
venv.create(".venv", with_pip=True, clear=True)    # Clear existing before creating!

# What venv creates inside the directory:
.venv/
├── pyvenv.cfg                  # Contains: home = /usr/bin, version = 3.12.0
├── bin/                        # Unix: contains activate script + python symlink
│   ├── activate                 # Source this to activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── pip                      # pip from the venv
│   └── python -> /usr/bin/python3.12
├── bin/                        # Windows equivalent: Scripts/
│   ├── activate.bat
│   ├── Activate.ps1
│   ├── pip.exe
│   └── python.exe -> system python
└── lib/
    └── python3.12/
        └── site-packages/     # All installed packages go here!
            ├── fastapi/
            ├── pydantic/
            └── ...
```

#### 2. `pipenv` — Hybrid Package Manager + Virtual Env Manager

From the creators of virtualenv. Combines pip + virtualenv in one tool with automatic dependency resolution.

```bash
# Initialize project (creates Pipfile and Pipfile.lock)
pipenv install                         # Creates Pipfile from existing dependencies
pipenv install fastapi pydantic        # Add production deps (updates Pipfile automatically!)
pipenv install --dev pytest mypy       # Add dev deps

# Pipfile format (like package.json but TOML!)
[[source]]
name = "pypi"
url = "https://pypi.org/simple"
verify_ssl = true

[packages]
fastapi = ">=0.104.1"                 # Like "fastapi": "^0.104.1" in package.json!
pydantic = { version = "^2.5.0", markers = "python_version >= '3.8'" }

[dev-packages]
pytest = "*"                           # * means any version (like latest)
mypy = ">=1.8.0"

[requires]
python_version = "3.10"               # Like "engines": {"node": ">=20"} in package.json!

# Activate managed venv
pipenv shell                           # Enters the Pipfile-managed environment
pipenv run python main.py              # Run without activating (like npx!)
pipenv run pytest                      # Like poetry run pytest
pipenv update                          # Update all deps (like npm update)
pipenv graph                           # Show dependency tree (like npm list)
pipenv lock                            # Generate/update Pipfile.lock

# Key difference from venv: pipenv MANAGES the virtual environment for you!
# You don't create or activate manually — it happens automatically.
```

#### 3. `conda` / `mamba` — Scientific Computing's Virtual Environment Manager

Conda manages Python versions, packages, AND non-Python dependencies (C libraries, CUDA). Essential for data science/ML.

```bash
# Conda environments (unlike venv which only isolates packages)
conda create --name myapp python=3.12 numpy pandas   # Create env with specific Python + packages!
conda activate myapp                                       # Activate (conदा does MORE than venv!)
conda install scikit-learn                               # Install package into active env
conda list                                               # List all packages in current env
conda info --envs                                        # List ALL environments

# mamba: drop-in faster replacement for conda (written in C++)
mamba create --name ml python=3.12 torch cuda:12.0   # Conda can install CUDA libraries!
                                                      # npm/Node.js cannot do this at all!

# Conda's .condarc configuration
channels:
  - conda-forge
  - nodefaults
envs_dirs:
  - /opt/conda/envs
pkgs_dirs:
  - /opt/conda/pkgs
```

#### 4. `poetry` — The Complete Dependency + Build System (Modern Standard)

Poetry manages virtual environments, dependencies, lock files, builds, AND publishing.

```toml
# pyproject.toml — Poetry configuration
[tool.poetry]
name = "my-python-app"
version = "1.0.0"
description = "A sample Python application"
authors = ["Developer <dev@example.com>"]
license = "MIT"
readme = "README.md"
packages = [{include = "my_package"}]

[tool.poetry.dependencies]
python = "^3.10"                    # Compatible with Python 3.10 to 4.x (like ~3.10 in npm!)
fastapi = "^0.104.1"                # Semantic version constraint (compatible with npm!)
pydantic = {version = "^2.5.0", extras = ["email"]}  # With extras (like package[feature])
sqlalchemy = ">=2.0,<3.0"           # Version range (like npm's ^2.0.0)
redis = {version = "^5.0.0", optional = true}  # Optional dependency (like peerDependencies!)
pandas = {version = "^2.1.0", python = ">=3.9"} # Python-specific version constraint

[tool.poetry.group.dev.dependencies]     # Dev dependencies! Like devDependencies in package.json!
pytest = "^7.4.3"
mypy = "^1.8.0"
ruff = "^0.1.14"
black = "^23.12.1"
pre-commit = "^3.5.0"

[tool.poetry.group.docs.dependencies]    # Documentation group! npm doesn't have this natively!
sphinx = "^7.2.6"
myst-parser = "^2.0.0"

[tool.poetry.scripts]                    # CLI entry points (like "bin" in package.json!)
my-cli-tool = "my_package.cli:main"
data-processor = "my_package.processor:run"

# Run commands with Poetry's managed environment
poetry run python main.py              # Executes in the poetry-managed venv! (Like npx!)
poetry install                         # Install ALL dependencies + create/update venv
poetry lock                            # Generate poetry.lock from pyproject.toml
poetry update                          # Update all dependencies to latest compatible versions
poetry show                            # List installed packages (like npm list)
poetry env info                        # Show current environment details
```

#### 5. `uv` — The New Ultra-Fast Python Tool (from Astral, the ruff creators!)

Written in Rust. 10-100x faster than pip/poetry. The modern successor to all of them.

```bash
# uv can do EVERYTHING other tools do, but instantaneously
uv venv .venv                          # Create venv (like python -m venv) — 5ms vs 500ms!
uv pip install fastapi pydantic        # Install packages (like pip install) — 10ms vs 2s!
uv pip compile requirements.in         # Dependency resolution (like pip-compile)
uv add fastapi                         # Add dependency + update lock file (like poetry add!)
uv remove fastapi                      # Remove dependency

# uv.toml configuration
[python]
default = "3.12"                       # Default Python version for new projects

[pip]
index-url = "https://pypi.org/simple"  # PyPI index URL
trusted-host = []                      # Trusted hosts (for private registries)

[global-settings]
native-tls = true                      # Use system TLS certificates
offline = false                        # Enable offline mode
```

### Mermaid: Virtual Environment Ecosystem Comparison

```mermaid
flowchart TD
    subgraph TS_NODE["TypeScript/Node.js Model"]
        direction TB
        N1["One Node.js Interpreter (managed by nvm globally)"] --> N2["Per-project node_modules/ with packages only"]
        N3["nvm switches the global Node version\n→ ALL projects share the same interpreter when activated!"]
    end

    subgraph PY_VENV_ECO["Python Virtual Environment Ecosystem — 5 Tools, Each Solving Different Problems"]
        direction TB
        V1["venv: stdlib, basic isolation\nOnly manages site-packages + interpreter. Fastest startup.\nBest for: Simple projects, CI/CD"] --> V2["pipenv: pip + venv combined\nAuto-manages venvs, Pipfile lock file.\nBest for: Web apps, Django projects"]
        V3["conda/mamba: manages Python + C libs + CUDA\nData science / ML focused. Handles non-Python deps!\nBest for: NumPy, PyTorch, TensorFlow projects"] --> V4["poetry: venv + deps + lock + build + publish\nModern standard. Complete dependency lifecycle.\nBest for: All Python projects (industry default)"]
        V5["uv: ultra-fast all-in-one from Astral\nRust-based. 10-100x faster than pip/poetry!\nBest for: Everything — the future of Python tooling!"]
    end

    N2 -.->|"Both isolate dependencies per-project:\ntypeScript uses node_modules/\nPython uses .venv/lib/site-packages/"| V1

    classDef ts fill:#3178c6,color:white
    classDef py fill:#3069bd,color:white
    classDef tool fill:#e44d26,color:white
    class N1,N2,N3 ts
    class V1,V2,V3,V4,V5 py
```

### Key Notes: Virtual Environments

1. **Python's virtual environments are fundamentally more isolated than Node.js `node_modules/`** — they include a separate Python interpreter, not just packages. This means two projects can run different Python versions simultaneously without any extra tools (nvm). In Node.js, you always need nvm/fnm to manage interpreter versions.
2. **venv is the foundation; Poetry/pdm/uv are higher-level abstractions** — they all use venv under the hood but add dependency resolution, lock files, and build tooling. Start with `venv` to understand the concept, then move to Poetry or uv for real projects.
3. **conda is in a different category** — it manages binary dependencies (C libraries, CUDA drivers) that pip cannot handle. If you need NumPy with MKL or PyTorch with GPU support, conda/mamba is essential. For web development, Poetry or uv is sufficient.
4. **uv is the future of Python tooling** — created by Astral (the ruff/mypy team), it's a Rust-based unified tool that replaces pip, poetry, pipenv, and venv. It's 10-100x faster because everything is written in Rust instead of Python. Start learning uv now for new projects.
5. **Always use a virtual environment** — never install packages globally! This is the Python equivalent of always running `npm install` per-project. The `venv` + `pip install` pattern (or Poetry/uv) is as fundamental to Python as `node_modules/` is to Node.js.

---

## 2. Package Management — Complete Deep Dive (pip every flag, pip-tools, poetry exhaustive, pdm, uv)

### TypeScript npm/Node.js Package Ecosystem vs Python PyPI/pip Ecosystem

| Feature | TypeScript/npm | Python/pip + pyproject.toml | Why It Matters for TS Devs |
|---------|---------------|---------------------------|----------------------------|
| **Registry size** | ~2M packages (npmjs.com) | ~500k+ packages (pypi.org) | npm has more packages, but PyPI has better per-package quality. Python packages tend to be more stable with longer support lifecycles. Like comparing a massive but noisy library vs a curated one. |
| **Version constraints** | `^4.18.2` (semver compatible), `~4.18.2` (patch only), `>=4.0.0 <5.0.0` (range) | `==2.0.0` (exact), `>=2.0.0` (minimum), `~=2.1.0` (compatible release, like ~ in npm) | Poetry supports `^` and `~` syntax just like npm! But pip itself only supports PEP 440 constraints which are more restricted. The Python ecosystem prefers exact pins for reproducibility. |
| **Installation target** | `node_modules/` (per-project) | `<venv>/lib/site-packages/` (per-venv) | Both isolate dependencies per-project/venv. But npm hoists packages to the root; Python installs everything flat in site-packages without hoisting. This means pip can install multiple versions of the same package if different projects need them. |
| **Metadata format** | `package.json` (JSON) | `pyproject.toml` or `setup.py` (TOML/Python) | TOML is more structured than JSON — it supports inline tables, multiline strings, and sections natively. Like JSON but with better type hints and comments! |
| **Lock file** | `package-lock.json` (auto-generated) | `poetry.lock`, `uv.lock`, or `pip-tools/Pipfile.lock` | Plain pip does NOT generate lock files. You must use Poetry, pdm, uv, or pip-tools to get deterministic dependency resolution. This is the biggest difference from npm's always-included lock file. |
| **Workspaces** | `package.json` `"workspaces": ["packages/*"]` | `pyproject.toml` `[tool.pdm.dev-dependencies]` (pdm) / Poetry doesn't natively support workspaces | TypeScript has built-in workspace support in npm/pnpm/yarn; Python is catching up. PDM and uv support workspaces well; Poetry needs third-party tools. |
| **Scoped packages** | `@scope/package-name` for namespacing | No scoping needed — package names are globally unique on PyPI | TypeScript uses @scope to avoid naming conflicts; Python requires global uniqueness. This is why package name approval on PyPI can be slow. |

### Every pip Flag and Option (Complete Reference)

```bash
# ========================================
# Installation flags — EVERY option
# ========================================
pip install fastapi                          # Basic install (latest version matching constraints)
pip install fastapi==0.104.1                # Exact version pin (PEP 440 compliant!)
pip install "fastapi>=0.104,<0.105"         # Version range with comma separator
pip install "fastapi~=0.104.1"              # Compatible release: >=0.104.1, <0.105.0 (like ~ in npm!)
pip install fastapi[slowapis]                # With extras (like npm's package[feature])
pip install fastapi[all]                     # Multiple extras combined
pip install git+https://github.com/xxx/fastapi.git  # Install directly from Git!
pip install -e ./local-package               # Editable/install in development mode (like npm link!)
pip install ./local-package                  # Install from local directory
pip install https://example.com/pkg.whl     # Install from a direct URL to a wheel file

# ========================================
# Environment flags
# ========================================
pip install --target ./mylib fastapi         # Install to custom directory (like npm --prefix)
pip install --user fastapi                   # Install to user site-packages (~/.local/lib/pythonX.Y/)
pip install --break-system-packages fastapi  # Override Debian/Ubuntu safety check (NOT recommended!)
pip install --no-cache-dir fastapi           # Skip caching downloaded wheels!

# ========================================
# Requirement file flags
# ========================================
pip install -r requirements.txt              # Install from requirements file
pip install -c constraints.txt -r requirements.txt  # Apply constraints (like npm overrides)
pip install -e .                             # Install the current package in editable mode
pip install --require-hashes -r requirements.txt  # Verify hash of every downloaded package!

# ========================================
# Upgrade and remove flags
# ========================================
pip install --upgrade fastapi                # Upgrade to latest compatible version (like npm update)
pip install --upgrade-strategy only-if-needed fastapi  # Only upgrade if needed (default behavior)
pip uninstall fastapi                        # Uninstall a package
pip uninstall -y fastapi pydantic            # Uninstall multiple packages without prompts

# ========================================
# Download and wheel flags
# ========================================
pip download fastapi --dest ./pkg-cache      # Download wheels without installing
pip wheel . --wheel-dir ./dist              # Build wheel from current directory (like npm pack)
pip install --no-deps fastapi                # Skip installing dependencies!
pip install --no-build-isolation fastapi     # Don't create isolated build environment

# ========================================
# Search and info flags
# ========================================
pip search fastapi                           # Search PyPI (DEPRECATED — use pypi.org web instead)
pip show fastapi                             # Show package metadata (like npm list)
pip index versions fastapi                   # Show all available versions on PyPI!
pip inspect                                    # Inspect the local environment in detail
pip list                                     # List installed packages (like npm ls --depth=0)
pip list --outdated                          # List outdated packages
pip freeze                                   # Output all installed packages with exact versions

# ========================================
# Network and index flags
# ========================================
pip install fastapi --index-url https://pypi.org/simple/  # Specify package index
pip install fastapi --extra-index-url https://test.pypi.org/simple/  # Additional index (for internal packages)
pip install fastapi --trusted-host pypi.org            # Trust a host for HTTP (not HTTPS!)
pip install fastapi --proxy http://user:pass@proxy     # Use a proxy server
pip install fastapi --timeout 30                 # Set connection timeout in seconds
pip install fastapi --retries 3                  # Number of retries on failure
pip install fastapi --cert /path/to/cert.pem      # TLS certificate for private registry
pip install fastapi --client-cert /path/to/cert     # Client certificate for auth
```

### pip-tools — Dependency Pinning (Like npm's Lock File)

pip-tools provides deterministic builds with exact dependency pinning, like npm's `package-lock.json`.

```bash
# pip-tools workflow: Write requirements.in → compile to requirements.txt with locked versions

# requirements.in — Your direct dependencies only!
fastapi>=0.104.1
pydantic>=2.5.0
sqlalchemy>=2.0.0

# Compile to pinned requirements.txt (like generating package-lock.json!)
pip-compile requirements.in                    # Resolves ALL transitive deps and pins them!
pip-compile --upgrade requirements.in         # Like npm update — re-resolve to latest compatible

# Generated requirements.txt (OUTPUT of pip-compile):
fastapi==0.104.1               # Your dependency, pinned exactly!
pydantic==2.5.0                # Your dependency, pinned exactly!
sqlalchemy==2.0.25             # Resolved from your constraint, pinned exactly!
starlette==0.32.0              # TRANSITIVE dep of fastapi, auto-resolved and pinned!
sniffio==1.3.1                 # TRANSITIVE dep of any package, auto-resolved and pinned!
typing-extensions==4.9.0       # TRANSITIVE dep, auto-resolved and pinned!

# Install the exact locked versions (like npm ci!)
pip-sync requirements.txt     # Installs EXACTLY what's in requirements.txt, removes extras!
pip install -r requirements.txt  # Alternative: just install without removing extras
```

### Poetry Exhaustive Configuration Reference

```toml
# Complete pyproject.toml with ALL Poetry configuration options
[tool.poetry]
name = "my-python-app"                            # Package name (must be unique on PyPI!)
version = "1.2.3"                                 # Semantic version (like package.json)
description = "A production-ready Python application"  # Short description shown on PyPI!
authors = ["Alice Dev <alice@example.com>"]       # Author(s) — required for publishing!
maintainers = ["Bob Maintainer <bob@example.com>"]  # Optional: additional maintainers
license = "MIT"                                   # SPDX license identifier (required for publishing!)
readme = "README.md"                              # Link to README (can be .rst, .md, or .txt)
homepage = "https://github.com/alice/my-app"      # Homepage URL on PyPI
repository = "https://github.com/alice/my-app"    # Repository URL
documentation = "https://my-app.readthedocs.io"   # Documentation URL

[[tool.poetry.source]]                            # Multiple package sources! Like npm scopes!
name = "pypi-public"
url = "https://pypi.org/simple/"
default = true                                    # Primary source (like default registry in npm!)

[[tool.poetry.source]]
name = "internal-pypi"
url = "https://pypi.internal.company.com/simple/"
secondary = true                                  # Secondary source (fallback when not found in default)

[build-system]                                    # Build backend configuration!
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.poetry.dependencies]                       # ALL dependency constraint types supported:
python = "^3.10"                                  # Compatible with 3.10 to <4.0 (like ^ in npm!)
fastapi = "^0.104.1"                              # Standard semver-compatible constraint
pydantic = { version = "^2.5.0", extras = ["email"] }  # With extras (like package[feature])
sqlalchemy = ">=2.0,<3.0"                         # Range constraint (like >=2.0 <3.0 in npm)
redis = { version = "^5.0.0", optional = true }   # Optional dependency (must be explicitly enabled!)
pandas = { version = "^2.1.0", markers = "sys_platform == 'linux'" }  # Platform-specific!
uvicorn = { version = "^0.27.0", python = ">=3.8,<4.0" }  # Python version constraint
numpy = ">=1.24"                                  # Minimum version (like ~ in npm for minor compatibility)

[tool.poetry.group.dev.dependencies]             # Dev dependencies group!
pytest = "^7.4.3"
mypy = "^1.8.0"
ruff = "^0.1.14"
black = "^23.12.1"
pre-commit = "^3.5.0"

[tool.poetry.group.docs.dependencies]            # Docs dependencies group!
sphinx = "^7.2.6"
myst-parser = "^2.0.0"

[tool.poetry.group.test.dependencies]            # Test-specific dependencies (separate from dev!)
pytest-cov = "^4.1.0"
pytest-asyncio = "^0.23.0"
hypothesis = "^6.95.0"

[tool.poetry.scripts]                            # CLI entry points installed as executables!
my-cli-tool = "my_package.cli:main"              # Running `my-cli-tool` executes main() in cli.py
data-processor = "my_package.processor:run"      # Second CLI command

[tool.poetry.plugins."console_scripts"]           # Alternative way to define scripts!
my-other-tool = "my_package.tools:execute"

[tool.poetry.urls]                               # Additional URLs shown on PyPI page!
"Bug Tracker" = "https://github.com/alice/my-app/issues"
"Changelog" = "https://github.com/alice/my-app/blob/main/CHANGELOG.md"
```

### pdm — Modern Alternative to Poetry

pdm is another modern Python package manager written in Python (like a faster Poetry). Excellent workspace support and PEP 621 compliance.

```toml
# pyproject.toml with PDM configuration (PEP 621 compliant!)
[project]
name = "my-python-app"
version = "1.0.0"
description = "A PDM-managed Python project"
authors = [{name = "Alice Dev", email = "alice@example.com"}]
license = {text = "MIT"}
requires-python = ">=3.10"
dependencies = [
    "fastapi>=0.104.1",
    "pydantic>=2.5.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.4", "mypy>=1.8", "ruff>=0.1"]
docs = ["sphinx>=7.2"]
test = ["pytest-cov>=4.1", "hypothesis>=6.95"]

[project.scripts]
my-cli-tool = "my_package.cli:main"

# PDM-specific configuration
[tool.pdm]
distribution = true                               # Configure as a distributable package

[tool.pdm.dev-dependencies]                       # Dev dependency groups (like Poetry's groups!)
dev = ["pytest>=7.4", "mypy>=1.8", "ruff>=0.1"]
lint = ["ruff>=0.1", "black>=23.12"]
test = ["pytest-cov>=4.1", "hypothesis>=6.95"]

# PDM commands (similar to Poetry but with pnpm-style conventions)
pdm install                                  # Install all dependencies + create venv
pdm add fastapi --dev                        # Add dev dependency
pdm remove fastapi                           # Remove dependency
pdm build                                    # Build source distribution + wheel
pdm publish                                  # Publish to PyPI
```

### uv — The Ultra-Fast Modern Python Tool

```bash
# uv is from Astral (the makers of ruff/mypy). Rust-based. Blazingly fast.

# Project management (like poetry/pdm but faster!)
uv init my-app                             # Create new project with pyproject.toml + .venv!
uv add fastapi pydantic                    # Add dependencies (updates pyproject.toml + lock file!)
uv add --dev pytest mypy ruff              # Add dev dependencies

# Dependency resolution (like pip-compile but instant!)
uv pip compile requirements.in -o requirements.txt  # Resolves and pins all deps!
uv sync                                     # Install from lock file (fastest install possible!)

# Lock file management
uv lock                                     # Generate/update the lock file
uv sync --all-packages                      # Sync ALL workspace packages at once!

# Build and publish
uv build                                    # Build source dist + wheel
uv publish                                  # Publish to PyPI (with API token auth!)

# Environment management
uv venv --python 3.12 .venv                 # Create venv with specific Python version!
uv pip list                                 # List installed packages
```

### Mermaid: Package Management Flow Comparison

```mermaid
flowchart TD
    subgraph TS_NPM["TypeScript/npm Package Management Flow"]
        N1["package.json\n  dependencies: ^4.0.0\ndevDependencies: *"] --> N2["npm install → resolves semver ranges\n→ generates package-lock.json\n→ installs into node_modules/"]
        N3["npm run start/test/lint\n→ Executes scripts from package.json!"]
    end

    subgraph PY_PIP_FLOW["Python/pip Basic Flow (No Lock File)"]
        P1["pyproject.toml or requirements.txt\n  fastapi==0.104.1\n  pydantic==2.5.0"] --> P2["pip install -r requirements.txt → installs exact versions\n→ NO lock file generated! Must use external tool for that!"]
    end

    subgraph PY_POETRY_FLOW["Python/Poetry Modern Flow"]
        O1["pyproject.toml + poetry.lock\n  fastapi = ^0.104.1\n  pydantic = ^2.5.0"] --> O2["poetry install → resolves ALL deps including transitive!\n→ installs into .venv/lib/site-packages/\n→ updates poetry.lock with EXACT versions of EVERY dep!"]
        O3["poetry run <command> → Executes in managed venv! Like npm run + npx combined!"]
    end

    subgraph PY_UV_FLOW["Python/uv Ultra-Fast Flow"]
        U1["pyproject.toml + uv.lock\n  fastapi = ^0.104.1\n  pydantic = ^2.5.0"] --> U2["uv sync → Resolves deps in milliseconds (Rust!)\n→ installs into .venv/lib/site-packages/\n→ updates uv.lock — 10x faster than poetry lock!"]
    end

    N2 -.->|"Both provide dependency management:\ntypeScript uses package.json + package-lock.json\nPython has multiple options: pip (no lock), Poetry (poetry.lock), uv (uv.lock)"| O1
```

### Key Notes: Package Management

1. **pip itself is a bare-bones installer** — it installs packages but does NOT manage virtual environments, lock files, or dependency resolution beyond what's in your requirements file. Use Poetry, pdm, or uv for production projects.
2. **pip-tools gives you npm-like lock files** — `pip-compile` resolves transitive dependencies and generates a fully pinned `requirements.txt`, like npm generating `package-lock.json`. Use `pip-sync` to install exactly what's pinned (removes any extras).
3. **Poetry is the most mature modern solution** — handles venv, dependency resolution, lock files, builds, and publishing all in one. The pyproject.toml format maps directly to npm's package.json concepts.
4. **uv is 10-100x faster than everything else** — written in Rust by Astral (the ruff team). It's becoming the default tool for new Python projects. Learn it alongside Poetry.

---

## 3. Build Systems — Poetry & pdm Exhaustive Configuration (vs npm scripts)

### TypeScript package.json Scripts vs Poetry/pyproject.toml Complete Mapping

```json
// === TypeScript: Complete package.json with ALL sections ===
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A sample application",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "test": "jest --coverage",
    "test:watch": "jest --watch",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "format": "prettier --write src/",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf dist/",
    "prepare": "husky install",
    "prebuild": "npm run clean"
  },
  "dependencies": {
    "express": "^4.18.2",
    "zod": "^3.22.4",
    "redis": "^4.6.0"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "@types/express": "^4.17.21",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.1",
    "eslint": "^8.56.0",
    "prettier": "^3.1.0",
    "husky": "^9.0.0",
    "lint-staged": "^15.0.0"
  },
  "engines": {
    "node": ">=20.0.0"
  }
}
```

```toml
# === Python: Complete pyproject.toml with ALL Poetry sections (equivalent to above!) ===

[tool.poetry]
name = "my-app"                        # Like "name" in package.json
version = "1.0.0"                      # Like "version" — also published to PyPI!
description = "A sample application"   # Like "description"
authors = ["Developer <dev@example.com>"]
license = "MIT"

[tool.poetry.dependencies]             # Like "dependencies" in package.json!
python = "^3.10"                       # Like "engines": {"node": ">=20"} — Python version constraint!
fastapi = "^0.104.1"                   # Like "express": "^4.18.2"!
pydantic = "^2.5.0"                    # Like "zod": "^3.22.4"!
redis = "^4.6.0"                       # Like "redis": "^4.6.0"!

[tool.poetry.group.dev.dependencies]   # Like "devDependencies"!
pytest = "^7.4.3"                      # Like "jest": "^29.7.0"!
mypy = "^1.8.0"                        # Type checking (like typescript!)
ruff = "^0.1.14"                       # Linting (like eslint!)
black = "^23.12.1"                     # Formatting (like prettier!)
pre-commit = "^3.5.0"                  # Git hooks (like husky!)

# No equivalent to @types/ — Python types are IN the code, not separate packages!

[tool.poetry.scripts]                  # CLI entry points (no direct npm equivalent)
my-cli-tool = "my_package.cli:main"    # Creates a `my-cli-tool` command on PATH!

# === Running commands (equivalent to npm scripts): ===
# poetry run python dist/index.py      # Like npm run start
# poetry run pytest tests/             # Like npm run test
# poetry run mypy src/                 # Like npm run type-check
# poetry run ruff check src/           # Like npm run lint
```

### Mermaid: Poetry Build System vs npm Scripts Flow

```mermaid
flowchart TD
    subgraph TS_NPM_BUILD["TypeScript/npm Build Pipeline"]
        N1["package.json\n→ dependencies, devDependencies, scripts"] --> N2["npm install\n→ resolves + installs to node_modules/\n→ generates package-lock.json"]
        N3["tsc --noEmit\n→ type-check without output (like mypy!)\ntypeScript compiles .ts → .js during build!\nPython only CHECKS types, never produces output code!"] --> N4["npm run start/test/lint\n→ Executes scripts in project's shell"]
    end

    subgraph PY_POETRY_BUILD["Python/Poetry Build Pipeline"]
        P1["pyproject.toml\n→ dependencies, dev-dependencies, scripts\nTOML format (structured + typed, unlike JSON)"] --> P2["poetry install\n→ resolves ALL deps including transitive!\n→ creates/updates .venv/\n→ generates poetry.lock with exact versions"]
        P3["mypy src/\n→ Static type analysis ONLY — no compilation!\nLike ESLint but for types, NOT tsc!\nmypy never produces output code."] --> P4["poetry run pytest tests/\npoetry run mypy src/\npoetry run ruff check src/\n→ All commands execute in managed venv!"]
    end

    N2 -.->|"Both manage dependencies + scripts:\ntypeScript uses package.json + npm\nPython uses pyproject.toml + Poetry!"| P2
```

### Key Notes: Build Systems (Poetry vs npm)

1. **Poetry's pyproject.toml is the spiritual successor to package.json** — it manages dependencies, dev-dependencies, scripts, and publishing all in one file. The TOML format is more structured than JSON with native support for sections and inline tables.
2. **Like npm run scripts; Poetry uses `poetry run <command>` for execution** — the key difference is that `poetry run` automatically activates the managed venv, whereas `npm run` just executes in the project shell (where node_modules/.bin is on PATH).
3. **Poetry has NO direct equivalent to TypeScript's compilation step** because Python doesn't need compilation. Instead, Poetry handles dependency resolution, venv management, and package publishing — responsibilities that npm splits between itself and tsc/webpack.

---

## 4. Type Checking — mypy Exhaustive Reference (Every Setting, Plugins, Strict Mode)

### TypeScript Compiler (tsc) vs mypy Complete Comparison

| Feature | TypeScript/tsc | Python/mypy | Why It Matters for TS Devs |
|---------|---------------|-------------|---------------------------|
| **Type checking** | Compiles .ts → .js while enforcing types. Types exist at both compile time and runtime (via type annotations). | Static analysis ONLY — never modifies your code or produces output. Pure checker like ESLint for types! Unlike tsc which is a compiler; mypy is a non-intrusive static analyzer. | TypeScript's types affect the compiled output (removed in JS, used during compilation); mypy's types are purely metadata that mypy reads but never touches your source code. |
| **Strict mode** | `"strict": true` in tsconfig.json enables: noImplicitAny, strictNullChecks, strictFunctionTypes, etc. | `--strict` flag or `[tool.mypy] strict = true` in pyproject.toml. Enables ALL available type checks! Like TypeScript's compilerOptions.strict but applied to Python. | Both ecosystems have comprehensive "strict" modes that catch the most bugs. TS strict is enabled by default in `tsc --init`; mypy strict must be explicitly enabled. |
| **Type inference** | Infers types from assignments, function returns, generics. `const x = 42` → x: number automatically! | Same! `x = 42` → mypy knows x is int (like tsc). Function return types inferred if annotated but body provides the type. If no annotation, falls back to object. Inference works identically in both ecosystems. | Both compilers/analyzers infer types automatically — you rarely need explicit annotations for simple cases. The inference engine in mypy is nearly as sophisticated as tsc's. |
| **Generic types** | `Array<T>`, `Map<K, V>`, generics with `<T extends SomeType>` constraints. | Same! `list[int]`, `dict[str, Any]`, `TypeVar` with bound/constraint. Python 3.12 supports PEP 695 generic syntax (like TypeScript generics!). | Both support generics but with different syntax. TypeScript uses angle brackets `<T>`; Python uses square brackets `[int]`. Python's TypeVar is like TS's generic constraints. |
| **Decorators** | `@decorator` transforms classes/functions at definition time. Same syntax as Python! | Identical decorator syntax! But mypy understands type signatures of decorators (with typing.overload). Both support metadata via functools.wraps. | The decorator syntax is literally identical between TypeScript and Python! This is one area where the ecosystems are truly aligned — both use @syntax for function/class transformation. |
| **Error reporting** | Inline in editor + CLI output with file:line:column format. Color-coded errors. | CLI output with file:line:col format (same!). VS Code extensions provide inline errors. Both support JSON output for CI integration. | mypy's error messages use the same location format as tsc (file.py:42:10) — equally precise! Both support machine-readable JSON output for IDE/CI integration. |

### Complete mypy Configuration — Every Setting Exhaustive

```toml
# ========================================
# Complete pyproject.toml with ALL mypy options
# ========================================
[tool.mypy]
# === Core Settings ===
python_version = "3.12"                    # Target Python version (REQUIRED — like tsconfig's target!)
strict = true                             # Enables ALL available type warnings! (Like TS strict: true)
warn_return_any = true                    # Warn when a function returns Any instead of the declared return type!
warn_unused_configs = true                # Warn about unused [mypy-*.ini] sections in config!
warn_redundant_casts = true               # Warn about casts that are guaranteed to be unnecessary
warn_unreachable = true                   # Warn about dead code after always-False conditions!

# === Strict Mode Extra Checks (enabled by 'strict', but can add individually for clarity) ===
disallow_untyped_defs = true              # Disallow functions without type annotations! (Like TS noImplicitAny!)
disallow_incomplete_defs = true           # Disallow partially untyped function definitions!
check_untyped_defs = true                 # Type-check inside untyped functions (enabled by strict)

# === Plugin Settings ===
mypy_path = "src"                         # Additional module search path (like TS baseUrl in tsconfig!)
namespace_packages = false                # Support PEP 420 namespace packages

# === Ignore / Skip Settings ===
ignore_missing_imports = true             # Silently ignore missing imports (e.g., third-party no-stubs)!
follow_imports = "normal"                 # normal | silent | skip — how to handle missing modules!
follow_imports_for_stubs = false          # Follow stub imports even with follow_imports = "skip"

# === Cache & Performance ===
cache_dir = ".mypy_cache"                 # Where mypy stores incremental analysis cache!
incremental = true                        # Enable incremental mode (caches previous results!)
cache_fine_grained = true                 # Fine-grained incremental — faster reruns!

# === Strict Mode Plugins (each one enables additional checks) ===
disallow_any_generics = false             # Disallow generic types without type parameters!
disallow_subclassing_any = false          # Disallow subclassing Any types!
disallow_untyped_calls = false            # Disallow calling untyped functions!
disallow_untyped_decorators = false       # Disallow decorating with untyped decorators!

# === Output Formatting ===
show_error_context = true                 # Show surrounding code context in error messages!
show_column_numbers = true                # Show column numbers (not just line numbers)!
show_error_codes = true                   # Show error codes (like "error: Argument 1 has incompatible type"!)
color_output = true                       # Colorize output (enabled by default in terminals)
error_summary = true                      # Show summary of errors at the end

# === Per-Package Overrides (like TS paths mappings!) ===
[tool.mypy."mypackage.submodule"]
disallow_untyped_defs = false              # Less strict for specific submodules
ignore_missing_imports = true              # Ignore missing types from this package

# === Plugin Configuration ===
[[tool.mypy.plugins]]
plugin = "pydantic.mypy"                   # Pydantic plugin for Pydantic model support!
[tool.mypy.plugins.pydantic]
init_typed = true                          # Generate typed __init__ for Pydantic models!

# ========================================
# Alternative: mypy.ini format (legacy, but still works)
# ========================================
[mypy]
python_version = 3.12
strict = true
warn_return_any = true
ignore_missing_imports = true
mypy_path = src
incremental = true
cache_dir = .mypy_cache

[mypy-my_package.*]
disallow_untyped_defs = false

[mypy-third_party_no_stubs.*]
ignore_missing_imports = true
```

### mypy Plugins — Complete Reference

```python
# ========================================
# Pydantic Plugin (REQUIRED for proper Pydantic v2 type checking!)
# ========================================
# Install: pip install pydantic
# Config in pyproject.toml: [[tool.mypy.plugins]] plugin = "pydantic.mypy"

from pydantic import BaseModel, Field

class User(BaseModel):
    name: str                            # Pydantic + mypy understand this!
    age: int = Field(ge=0, le=150)      # MyPy respects validation constraints!
    email: str | None = None             # Optional types work correctly with plugin!

# Without pydantic.mypy plugin, mypy doesn't understand Pydantic's magic!

# ========================================
# NumPy Plugin (for proper NumPy type checking!)
# ========================================
# Install: pip install numpy-stubs
# Config in pyproject.toml: [[tool.mypy.plugins]] plugin = "numpy.typing.mypy_plugin"

import numpy as np
from numpy.typing import NDArray

def process_array(arr: NDArray[np.float64]) -> float:  # MyPy understands!
    return arr.mean()

# ========================================
 # Django Plugin (for proper Django ORM type checking!)
 # Install: pip install django-stubs
 # Config in pyproject.toml: [[tool.mypy.plugins]] plugin = "mypy_django_plugin.main"

from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)  # Django plugin provides proper types!

# ========================================
# Custom mypy Plugin (for your own library!)
# ========================================
# Create a file: my_custom_plugin.py

def plugin(version: str):
    """Custom mypy plugin entry point. version is the mypy version string."""
    return MyPlugin

class MyPlugin:
    def __init__(self, config: dict, file_path: str) -> None:
        # Called once per analyzed file
        self.file_path = file_path

    def visit_func_def(self, node):
        # Called for each function definition — add custom type checking here!
        pass

# ========================================
# Running mypy with all options
# ========================================
mypy src/                                    # Basic check (uses config from pyproject.toml)
mypy --strict src/                           # Enable ALL strict checks!
mypy --config-file=mypy.ini src/            # Use specific config file
mypy --ignore-missing-imports src/          # Skip missing import errors (like TS skipLibCheck!)
mypy --show-error-codes src/                # Show error codes (e.g., arg-type, return-value)
mypy --disallow-untyped-defs src/           # Disallow untyped function definitions!
mypy --warn-unreachable src/                # Warn about dead/unreachable code paths
mypy --no-incremental src/                  # Disable incremental caching (full re-check!)
mypy --json-text src/                       # JSON output for CI/IDE integration!
mypy --html-report mypy-html src/           # Generate HTML report (great for PRs!)
```

### Mermaid: mypy Type Check Pipeline

```mermaid
flowchart TD
    subgraph MYPY_PIPELINE["Python/mypy Static Type Analysis Pipeline"]
        direction TB
        S1["Source .py files with type hints\n(annotations are metadata only — runtime ignores them!)"] --> S2["mypy reads pyproject.toml config:\npython_version, strict mode, plugins, overrides"]
        S2 --> S3["Type inference engine:\n- x = 42 → int (like tsc inference)\n- def f(x: str) → str → return type inferred\n- generics: list[int], dict[str, Any]"]
        S3 --> S4["Strict checks:\n- disallow_untyped_defs: all functions must have annotations!\n- warn_return_any: no returning Any!\n- check_untyped_defs: type-check inside untyped code too!"]
        S4 --> S5["Plugin integration:\n- pydantic.mypy → understands Pydantic models\n- django → understands ORM types\n- custom plugins → your own checks!"]
        S5 --> S6["Error output:\nfile.py:42:10: error: Incompatible types!\n[error-code]  ← Like tsc's error codes!\n→ Colored, with context and exact line/column!"]
    end

    subgraph TS_TSC_PIPELINE["TypeScript/tsc Type Check Pipeline (for comparison)"]
        direction TB
        T1["Source .ts files with types\n(annotations are compiled away — runtime has no types)"] --> T2["tsc reads tsconfig.json:\ntarget, module, strict, skipLibCheck"]
        T3["Type inference: x = 42 → number (same as mypy!)\nGeneric inference: Array<number>\nDecorator type resolution"] --> T4["Compilation to .js\n(types removed, but checked during compilation!)"]
    end

    S3 -.->|"Both use identical inference logic:\nx = 42 means int in Python / number in TS!\nBoth infer from assignments + generics!"| T3
```

### Key Notes: Type Checking (mypy vs tsc)

1. **mypy is TypeScript's tsc but as a pure static analyzer** — it never modifies your files or produces output code. Unlike tsc which compiles .ts → .js, mypy only reads your source and reports errors to the console. This makes it easier to integrate into CI/CD because there are no build artifacts to manage.
2. **mypy's strict mode is like TypeScript's `"strict": true`** — it enables all available checks including disallowing untyped functions, requiring return type annotations, and catching implicit `Any` usage. The name difference (Python uses snake_case for flags: `--disallow-untyped-defs` vs TypeScript's camelCase) but the concepts are identical.
3. **Type inference works identically in both ecosystems** — if you write `x = 42`, mypy infers `x` as `int` just like tsc infers `x` as `number`. Function return types are inferred from the body when annotated: `def f(x: int) -> str: ...` where the body's return type is checked against `str`.
4. **Plugins extend mypy's understanding** — the pydantic.mypy plugin makes mypy understand Pydantic models' magic methods, the Django plugin understands ORM query results, and you can write custom plugins for your own libraries. TypeScript has similar extension points via declaration files (`@types/*`).

---

## 5. Linting & Formatting — ruff/black Exhaustive (All Rules, Formatter, isort, bandit)

### TypeScript ESLint/Prettier vs Python ruff/black Complete Comparison

| Tool Category | TypeScript | Python Equivalent | Speed | Configuration |
|--------------|-----------|-----------------|-------|--------------|
| **Linting** | ESLint (JS/TS rules), typescript-eslint plugin | **ruff** (1,000+ rules from flake8, pylint, pyflakes, isort, and more) — Rust-based, 10-100x faster! | ~100ms vs ESLint's 5s for 10k LOC | pyproject.toml → [tool.ruff] |
| **Formatting** | Prettier (auto-formats to consistent style) | **ruff format** (black-compatible, written in Rust!) — same output as black but 20x faster! | ~5ms vs black's 100ms for 10k LOC | pyproject.toml → [tool.ruff.format] |
| **Import sorting** | eslint-plugin-import | ruff's I (isort) rules — built into `ruff check`! No separate tool needed! | Instantly within ruff check | pyproject.toml → [tool.ruff.lint.isort] |
| **Security linting** | eslint-security / npm audit | **bandit** (security-focused linter for Python) or ruff's S rules | bandit: ~500ms; ruff S: <10ms | pyproject.toml → [tool.bandit] |
| **Dead code** | tsc --noUnusedLocals (built-in!) | ruff's F841/F401 rules (unused vars/imports) — built into ruff check! | Built into ruff check | Automatically in ruff check |

### Complete ruff Configuration — Every Rule Group Exhaustive

```toml
# ========================================
# Complete ruff configuration — ALL options documented
# ========================================
[tool.ruff]
target-version = "py312"                    # Target Python version (like ruff --target-version py312)
line-length = 100                           # Max line length (like Prettier's printWidth!)
exclude = ["migrations/", ".venv/", "node_modules/"]  # Paths to exclude!

[tool.ruff.lint]                            # Linting configuration!
select = [
    "E",     # pycodestyle errors (like ESLint core rules)
    "W",     # pycodestyle warnings
    "F",     # Pyflakes (unused imports, undefined names — like TS noUnusedLocals!)
    "I",     # isort (import sorting!)
    "N",     # pep8-naming (class/function naming conventions!)
    "UP",    # pyupgrade (auto-update to modern Python syntax!)
    "B",     # flake8-bugbear (common bugs — like ESLint's semi-colon rules)
    "SIM",   # flake8-simplify (simplify complex expressions!)
    "RUF",   # ruff-specific rules!
    "C90",   # mccabe (complexity — like ESLint's complexity rule!)
    "T20",   # print statements (warn against print() in production code!)
    "S",     # bandit (security checks — like eslint-security!)
    "PIE",   # flake8-pie (prevent common mistakes)
]

# Rules to ignore individually (like ESLint's "rules": {"no-unused-vars": "off"})
ignore = [
    "E501",      # Line length — let the FORMATTER handle it instead of the LINTER!
    "B008",      # Do not perform function calls in argument defaults
]

# Like ESLint's "rules" object for fine-grained control!
[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]                    # Unused imports in __init__.py are intentional (re-exports)!
"tests/**/*.py" = ["S101"]                  # Allow assert statements in tests (bandit's S101)
"migrations/*.py" = ["E501", "SIM"]         # Less strict rules for auto-generated migration files

[tool.ruff.lint.isort]                      # Import sorting configuration!
known-first-party = ["my_package"]          # Group first-party imports separately (like import-groups in eslint-plugin-import)
known-third-party = ["fastapi", "pydantic"]  # Third-party group
section-order = ["future", "standard-library", "third-party", "first-party", "local-folder"]

[tool.ruff.lint.mccabe]                     # Complexity configuration!
max-complexity = 10                         # Maximum allowed cyclomatic complexity (default: 10)

[tool.ruff.format]                          # Formatting configuration!
quote-style = "double"                      # Use double quotes (like Prettier's single/double config!)
indent-style = "space"                      # Spaces vs tabs (4 spaces is Python convention)
skip-magic-trailing-comma = false           # Remove trailing commas where possible (like Prettier!)
line-ending = "lf"                          # Line endings: lf | crlf | native

# ========================================
# Run ruff commands (equivalent to ESLint/Prettier in TypeScript)
# ========================================
ruff check .                               # Lint ALL files (like eslint src/)
ruff check --fix .                         # Auto-fix all fixable issues (like eslint --fix!)
ruff format .                              # Format ALL files (like prettier --write src/)
ruff format --check .                      # Check formatting without modifying (like prettier --check)
ruff check --select E501 .                 # Run specific rules only (like eslint --rule "eol-last: error")
```

### Complete bandit Configuration — Security Linting

```toml
# ========================================
# Bandit configuration for security-focused linting!
# ========================================
[tool.bandit]
targets = ["src/"]                         # Directories to scan
exclude_dirs = [".venv/", "tests/"]        # Exclude from scanning
skips = ["B101"]                           # Skip specific checks (B101 = assert statements)
include = ["B101", "B301"]                 # Only run these specific checks

[tool.ruff.lint]
select = ["S"]                             # Use ruff's S rules (Bandit-compatible security linter!)
```

### Pre-commit Configuration for Both ruff and bandit

```yaml
# .pre-commit-config.yaml — Complete pre-commit configuration!
repos:
  # === Ruff (linting + formatting) ===
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff-check                      # Lint only (like eslint)
        args: [--fix]                         # Auto-fix fixable issues
      - id: ruff-format                       # Format code (like prettier)

  # === Bandit (security linting) ===
  - repo: https://github.com/PyCQA/bandit
    rev: 1.8.3
    hooks:
      - id: bandit
        args: [-r, src/, -f, json]          # Run on src/, output JSON for CI

  # === mypy (type checking) — runs AFTER linting! ===
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.15.0
    hooks:
      - id: mypy
        additional_dependencies: [
          pydantic,                # Pydantic plugin
          types-requests,            # Type stubs for requests
        ]

  # === Black (alternative formatter — slower but battle-tested) ===
  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 24.10.0
    hooks:
      - id: black
```

### Mermaid: ruff Lint and Format Chain

```mermaid
flowchart TD
    subgraph RUFF_PIPELINE["Python/ruff Complete Linting Pipeline"]
        direction TB
        R1["Developer writes code\n→ Python source files with mixed styles, imports, potential bugs"] --> R2["Pre-commit hook triggers:\nruff check . → lint ALL files against 40+ rule groups!\n- E/W: pycodestyle (syntax errors)\n- F: Pyflakes (unused vars/imports)\n- I: isort (import sorting)\n- B: bugbear (common bugs)\n- SIM: simplify (simplify expressions)\n- S: bandit (security issues)\n- RUF: ruff-specific rules\n- UP: pyupgrade (modernize syntax)"]
        R2 --> R3["ruff check --fix → Auto-fix ALL fixable issues!\n→ Like ESLint's --fix but 100x faster!"]
        R3 --> R4["ruff format . → Format ALL files!\n→ Same output as black but 20x faster\n→ Configurable: quote style, indent, line length"]
        R4 --> R5["All commits now have:\n✓ No syntax errors (E/W)\n✓ No unused imports (F)\n✓ Sorted imports (I)\n✓ Consistent formatting\n✓ No security issues (S)\n✓ Simplified expressions (SIM)!"]
    end

    subgraph TS_ESLINT_PRETTIER_PIPELINE["TypeScript/ESLint+Prettier Equivalent Pipeline"]
        direction TB
        T1["Developer writes code → TypeScript source files with mixed styles!"] --> T2["Husky hook triggers:\neslint src/ --fix → lint ALL files\nprettier --write src/ → Format ALL files!"]
    end

    R3 -.->|"Both pipelines ensure code quality before commit:\ntypeScript uses ESLint + Prettier\nPython uses ruff (does BOTH linting and formatting in ONE tool!)!"| T2
```

### Key Notes: Linting & Formatting

1. **ruff does BOTH linting and formatting in one tool** — unlike TypeScript where you need ESLint (linting) + Prettier (formatting) as separate packages. ruff is written in Rust and runs 10-100x faster than ESLint on equivalent codebases.
2. **ruff includes bandit-compatible security rules** (`select = ["S"]`) — so you don't even need a separate bandit tool for most security checks. Use standalone bandit only for deep security audits.
3. **Use ruff format over black** — since mid-2024, ruff's formatter produces identical output to black but is ~20x faster. Black is still the established standard; ruff format is the modern replacement that everyone is migrating to.

---

## 6. Testing — pytest Complete Ecosystem (30+ Plugins, hypothesis, coverage, xdist)

### TypeScript/Jest vs Python/pytest Complete Comparison

| Feature | TypeScript/jest | Python/pytest | Notes for TS Devs |
|---------|---------------|--------------|-----------------|
| **Test discovery** | `**/*.test.ts` or `**/*.spec.ts` pattern | `test_*.py` or `*_test.py` files, `test_*()` functions | Both auto-discover tests by convention. Jest uses file patterns; pytest uses naming conventions. |
| **Assertions** | `expect(actual).toBe(expected)` chained matchers | Built-in `assert actual == expected` | TypeScript's expect chaining is more expressive; Python's assert is simpler but covers basic equality natively! |
| **Mocking** | `jest.fn()`, `jest.mock()` built into framework | `unittest.mock.patch()` — separate import but more powerful! | Python's mock can patch ANYTHING including built-ins. More flexible than Jest's mocking API. |
| **Fixtures/Hooks** | `describe()`, `it()`, `beforeEach()`, `afterEach()` | `@pytest.fixture` with dependency injection + scopes (function, class, module, session)! | pytest fixtures are more powerful — they can be shared across test files and have lifecycle control! |
| **Async support** | Native async/await in tests | `pytest-asyncio` plugin | TypeScript has native async test support; Python needs a plugin for the same capability. |
| **Coverage** | `--coverage` built into Jest | `pytest-cov` plugin (covers pytest) | Both provide line-by-line coverage reports. Jest bundles it; Python splits it as a plugin. |

### Complete pytest Plugin Reference — 30+ Plugins

| Plugin | Purpose | Install | Like TypeScript's... |
|--------|---------|---------|---------------------|
| **pytest-cov** | Line-by-line coverage reports (HTML, XML, JSON) | `pip install pytest-cov` | jest --coverage (built-in) |
| **pytest-asyncio** | Async test support | `pip install pytest-asyncio` | Native async in Jest |
| **pytest-mock** | Cleaner mocking via pytest plugin wrapper | `pip install pytest-mock` | jest.mock() |
| **pytest-lazy-fixture** | Lazy evaluation of fixtures | `pip install pytest-lazy-fixtures` | beforeEach pattern |
| **pytest-benchmark** | Performance benchmarking for functions | `pip install pytest-benchmark` | Jest performance hooks |
| **pytest-xdist** | Parallel test execution (multiple CPUs) | `pip install pytest-xdist` | Jest --runInBand parallel workers |
| **pytest-timeout** | Automatically fail tests exceeding time limit | `pip install pytest-timeout` | Jest --testTimeout |
| **pytest-randomly** | Shuffle test order to catch hidden dependencies | `pip install pytest-randomly` | N/A (unique feature!) |
| **pytest-subtests** | Sub-tests within a single test function | `pip install pytest-subtests` | Test.each pattern in Jest |
| **pytest-html** | Generate HTML test reports | `pip install pytest-html` | jest --json-report |
| **pytest-datadir** | Provide test data directories | `pip install pytest-datadir` | fixture with file path |
| **pytest-insta** | Snapshot testing (like Jest snapshot) | `pip install pytest-insta` | Jest snapshot |
| **pytest-ordering** | Control test execution order | `pip installed` (built into pytest!) | describe/it nesting in Jest |
| **pytest-socket** | Prevent tests from making real network calls! | `pip install pytest-socket` | No direct TS equivalent (unique safety feature!) |
| **pytest-localserver** | Launch local HTTP servers for testing | `pip install pytest-localserver` | jest --setupFilesAfterEnv with mock server |
| **pytest-rerunfailures** | Rerun failed tests to catch flaky tests | `pip install pytest-rerunfailures` | Jest retry mechanism |
| **pytest-mypy** | Run mypy as a pytest plugin | `pip install pytest-mypy` | tsc in CI pipeline |
| **pytest-env** | Manage environment variables in tests | `pip install pytest-env` | jest --env node |
| **pytest-xprocess** | Manage external processes during tests | `pip install pytest-xprocess` | Like setting up test database |
| **pytest-watch** | Watch files and re-run tests on change | `pip install pytest-watch` | Jest --watch mode |
| **pytest-split** | Split test suite for parallel CI execution | `pip install pytest-split` | Jest --shard flag |
| **pytest-dotenv** | Load .env file into os.environ during tests | `pip install pytest-dotenv` | jest-config with dotenv setup |
| **pytest-forked** | Run each test in a subprocess (isolation) | `pip install pytest-forked` | Jest isolateTestsInRunner |
| **pytest-profiling** | Profile tests for performance bottlenecks | `pip install pytest-profiling` | Like Jest --coverage but for profiling |
| **pytest-httpx** | Mock HTTPX async client easily | `pip install pytest-httpx` | axios-mock-adapter |
| **pytest-recording** | VCR-style request/response recording | `pip install pytest-recording` | No direct TS equivalent! |
| **pytest-bdd** | BDD (Behavior Driven Development) style | `pip install pytest-bdd` | Cucumber/Jest BDD plugins |
| **pytest-sugar** | Fancy progress output for test runs | `pip install pytest-sugar` | jest --verbose but prettier! |
| **pytest-ruff** | Run ruff as a pytest plugin | `pip install pytest-ruff` | eslint-plugin in Jest pipeline |
| **hypothesis** | Property-based testing (generates random inputs!) | `pip install hypothesis` | Like fast-check for TypeScript! |
| **responses** | Mock HTTP responses (requests library) | `pip install responses` | nock (TypeScript HTTP mocking) |

### pytest Configuration — Complete Reference

```toml
# ========================================
# Complete pytest configuration in pyproject.toml
# ========================================
[tool.pytest.ini_options]
addopts = "-v --tb=short --cov=my_package --cov-report=html --cov-report=xml"  # Default flags!
testpaths = ["tests"]                                       # Directory containing test files!
python_files = ["test_*.py", "*_test.py"]                   # Files that pytest discovers!
python_classes = ["Test*"]                                  # Class names to discover!
python_functions = ["test_*"]                               # Function names to discover!
filterwarnings = [                                          # Filter warnings (like TS's suppressImplicitAny)
    "ignore::DeprecationWarning",                           # Ignore deprecation warnings
    "error::pytest.PytestUnhandledCoroutineWarning",         # Convert certain warnings to errors!
]

# ========================================
# pytest-asyncio configuration!
# ========================================
[tool.pytest.ini_options]
asyncio_mode = "auto"           # Automatically treat all async tests as async!
asyncio_default_fixture_loop_scope = "function"  # Scope of the async fixture loop!

# ========================================
# pytest-cov configuration!
# ========================================
[tool.coverage.run]
source = ["my_package"]         # Which source directories to measure!
omit = ["*/tests/*", "*/migrations/*"]  # Directories to exclude from coverage!

[tool.coverage.report]
show_missing = true             # Show which lines are NOT covered!
fail_under = 80                 # Fail CI if coverage drops below 80%!
skip_empty = true               # Skip files with no executed code!

# ========================================
# pytest-xdist configuration (parallel execution)!
# ========================================
# Run tests on ALL available CPUs!
pytest -n auto                 # Auto-detect number of CPU cores and distribute!
pytest -n 4                    # Force exactly 4 parallel workers!
```

### Mermaid: pytest Test Framework Pipeline vs Jest

```mermaid
flowchart TD
    subgraph TS_JEST["TypeScript/jest Testing Pipeline"]
        direction TB
        J1["Test files: **/*.test.ts\n→ describe('math', () => {\n  it('adds correctly', () => {\n    expect(add(1,2)).toBe(3);\n  });\n})"] --> J2["jest configures test environment,\ntransform TypeScript via ts-jest,\nrun tests in isolation,\ngenerate coverage report!"]
        J3["BeforeEach/AfterEach hooks\n→ beforeEach(() => {...})\nafterEach(() => {...})"]
    end

    subgraph PY_PYTEST["Python/pytest Testing Pipeline"]
        direction TB
        P1["Test files: tests/test_*.py\n→ def test_add():\n    assert add(1, 2) == 3\n→ pytest discovers ALL test_* functions!"] --> P2["pytest discovers tests,\nruns each in isolation,\ninjects fixtures via dependency injection,\ngenerates coverage via pytest-cov plugin!\nUnlike Jest: modular plugins instead of monolith!"]
        P4["@pytest.fixture(scope='session')\ndef db_connection(): ... → fixture with lifecycle control!\nDependencies are EXPLICIT parameters — visible in function signature!"]
    end

    J2 -.->|"Both auto-discover and run tests:\ntypeScript uses Jest (monolith)\nPython uses pytest + plugins (modular but equivalent power!)!"| P2
```

### Key Notes: Testing (pytest vs Jest)

1. **pytest is the Jest of Python** — it's the dominant test framework that handles discovery, assertions, mocking, and integration with coverage tools. Unlike Jest which bundles everything in one package, pytest uses a modular plugin architecture where you install only what you need.
2. **pytest fixtures are like Jest's beforeEach() but with dependency injection** — instead of implicit hooks, you explicitly request fixtures as function parameters. This makes tests more readable because the test signature reveals exactly what resources it depends on! Unlike JavaScript's implicit this/context pattern, Python's explicit dependencies make refactoring safer.
3. **Property-based testing with hypothesis is like fast-check for TypeScript** — instead of writing individual test cases, you define properties (invariants that must always hold) and hypothesis generates hundreds of random inputs to find counterexamples! Like TypeScript developers using fast-check to test edge cases automatically.

---

## 7. Pre-commit Hooks — Exhaustive Reference (Every Hook Type, Repos, Local Hooks)

### Complete Pre-commit Configuration Reference

```yaml
# .pre-commit-config.yaml — Every hook type documented!

repos:
  # ========================================
  # Official pre-commit hooks (from trusted repos!)
  # ========================================

  # === Python Code Quality ===
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff-check
        args: [--fix]                     # Auto-fix fixable issues!
      - id: ruff-format                   # Format all Python files!

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 24.10.0
    hooks:
      - id: black                         # Black formatter (alternative to ruff format)
        args: [--line-length, "100"]      # Custom line length!

  # === Type Checking ===
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.15.0
    hooks:
      - id: mypy
        additional_dependencies:
          - pydantic                      # Pydantic plugin dependency!
          - types-requests                # Type stubs!

  # === Security Linting ===
  - repo: https://github.com/PyCQA/bandit
    rev: 1.8.3
    hooks:
      - id: bandit
        args: [-r, src/, -f, json]       # Recursive scan, JSON output!

  # === YAML Formatting ===
  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v4.0.0-alpha.8
    hooks:
      - id: prettier                      # Format YAML, JSON, Markdown!
        types_or: [yaml, json, markdown]

  # === Shell Script Linting ===
  - repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.10.0
    hooks:
      - id: shellcheck                    # Lint all .sh files!

  # === Markdown Formatting ===
  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v4.0.0-alpha.8
    hooks:
      - id: prettier
        types: [markdown]

  # ========================================
  # Local hooks (custom scripts in your repo!)
  # ========================================
  - repo: local
    hooks:
      - id: custom-lint
        name: Custom lint check
        entry: python scripts/custom_linter.py
        language: system
        pass_filenames: false             # Don't pass file names (config-driven)!

      - id: run-unit-tests
        name: Run unit tests on commit
        entry: pytest tests/ --tb=short
        language: system
        pass_filenames: false
        always_run: true                  # ALWAYS run (not just on changed files)!
        stages: [commit]                  # Only on commit stage

  # ========================================
  # Skip flags (when you need to bypass hooks!)
  # ========================================
  # pre-commit skip=commit-msg
  # pre-commit autoupdate                # Update all hook versions!

# ========================================
# Global configuration!
# ========================================
default_stages: [commit]                # Default git stage for all hooks
fail_fast: true                         # Stop on first hook failure (like Husky's fail-fast!)
minimum_pre_commit_version: "3.5.0"    # Minimum pre-commit version required!
```

### Pre-commit Hook Execution Flow

```mermaid
flowchart TD
    subgraph PRECOMMIT_HOOKS["Python Pre-commit Hook Execution Pipeline"]
        direction TB
        H1["Developer runs git commit"] --> H2["pre-commit intercepts the commit!"]
        H2 --> H3["For each configured hook:\n1. Check if hook needs updating\n2. Download hook repo to ~/.cache/pre-commit/\n3. Determine which files to check (staged files only!)"]
        H3 --> H4["Run ruff-check on staged .py files → fix issues automatically!"]
        H5["Run ruff-format on staged .py files → format consistently!"]
        H6["Run mypy on staged .py files → type-check!\n→ If type errors found: commit is BLOCKED!"]
        H7["Run bandit on src/ → security scan!\n→ If vulnerabilities found: commit is BLOCKED!"]
        H8["All hooks pass → Git commit PROCEEDS!\nIf ANY hook fails → commit is ABORTED!\nDeveloper must fix issues and re-commit!"]
    end

    subgraph TS_HUSKY_EQUAL["TypeScript/Husky Equivalent Pipeline"]
        direction TB
        S1["Developer runs git commit"] --> S2["husky intercepts (configured via package.json scripts)"]
        S3["lint-staged runs eslint --fix + prettier on staged files!\n→ If errors: commit is BLOCKED!"]
        S4["All checks pass → Git commit PROCEEDS!"]
    end

    H6 -.->|"Both block bad commits:\ntypeScript uses Husky + lint-staged\nPython uses pre-commit framework!"| S3
```

### Key Notes: Pre-commit Hooks

1. **pre-commit is like Husky + lint-staged combined** — it intercepts git commits and runs configurable hooks before the commit is finalized. If any hook fails, the commit is blocked and you must fix the issues first.
2. **Hooks run on staged files only (fast!)** — unlike running linters on the entire codebase, pre-commit only processes files that are about to be committed. This makes even heavy checks (mypy, bandit) fast enough for every commit.
3. **Local hooks let you run any custom command** — Python scripts, test suites, or any tool as a hook. Use `always_run: true` to always execute regardless of changed files (like running the full test suite on every commit).

---

## 8. Docker for Python Development

### Complete Dockerfile for Python Projects

```dockerfile
# ========================================
# Production-ready Python Dockerfile (multi-stage build!)
# ========================================

# Stage 1: Base image with Python and system dependencies
FROM python:3.12-slim AS base
WORKDIR /app

# Install system dependencies that Python packages need at build time
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \                          # PostgreSQL dev headers (needed by psycopg2)
    && rm -rf /var/lib/apt/lists/*

# Stage 2: Dependency installation (layer caching!)
FROM base AS dependencies
COPY pyproject.toml poetry.lock ./
RUN pip install --no-cache-dir poetry && \
    poetry config virtualenvs.create false && \
    poetry install --only main --no-interaction --no-ansi

# Stage 3: Production build
FROM dependencies AS production
COPY . .
EXPOSE 8000                              # FastAPI/Uvicorn default port!

# Set environment variables for production!
ENV PYTHONDONTWRITEBYTECODE=1           # Don't write .pyc files!
ENV PYTHONUNBUFFERED=1                  # Don't buffer stdout (for logs!)

# Run with uvicorn (the ASGI server for FastAPI)
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# ========================================
# Development docker-compose.yml!
# ========================================
version: '3.9'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"                      # Map host port to container!
    volumes:
      - .:/app                            # Mount code for hot-reload in dev!
      - /app/__pycache__/                 # Prevent .pyc pollution!
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb  # DB connection!
    depends_on:
      - db
      - redis

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

### Key Notes: Docker for Python

1. **Use multi-stage builds** — first stage installs dependencies, second copies your code. This keeps production images small (<200MB vs >1GB with dev tools). Like how Node.js Dockerfiles use npm ci for dependency caching.
2. **Mount volumes in development** — `-v .:/app` enables hot-reload without rebuilding. In production, you COPY the files to ensure the container matches your build exactly.
3. **Use `--no-cache-dir` for pip install** — prevents pip from caching downloaded packages inside the image (saves 50-200MB per build).

---

## 9. CI/CD Patterns — GitHub Actions for Python

### Complete GitHub Actions Workflow for Python Projects

```yaml
# .github/workflows/ci.yml — Complete CI pipeline for Python!
name: Python CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # === Job 1: Lint + Format (fastest check — run first!) ===
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install ruff
        run: pip install ruff
      - name: Run ruff linting
        run: ruff check .
      - name: Run ruff formatting check
        run: ruff format --check .

  # === Job 2: Type Checking ===
  type-check:
    needs: lint                               # Only runs if lint passes!
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - name: Install project deps
        run: pip install -e ".[dev]"          # Install package + dev deps!
      - name: Run mypy
        run: mypy src/ --strict

  # === Job 3: Unit Tests with Coverage ===
  test:
    needs: [lint, type-check]                  # Depends on both lint and type-check passing!
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]  # Test against all supported Python versions!
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: ${{ matrix.python-version }} }
      - name: Install deps
        run: pip install -e ".[dev,test]"
      - name: Run tests with coverage
        run: pytest --cov=src --cov-report=xml --cov-report=html
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml

  # === Job 4: Build and Publish (only on main branch tags!) ===
  publish:
    needs: test                                # Only runs if ALL tests pass!
    if: github.ref_type == 'tag'               # Only on tag pushes (v1.0.0, etc.)!
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - name: Build package
        run: pip install build && python -m build
      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          password: ${{ secrets.PYPI_API_TOKEN }}  # API token stored in GitHub Secrets!
```

### Mermaid: CI/CD Workflow for Python Project

```mermaid
flowchart TD
    subgraph GITHUB_ACTIONS["GitHub Actions CI/CD Pipeline for Python"]
        direction TB
        A1["Developer pushes to GitHub\n→ Git push triggers CI workflow!\nLike Node.js's GitHub Actions for JS projects!"] --> A2["Job 1: Lint (ruff check + ruff format --check)\n→ Fastest check — runs in <10 seconds!\nIf lint fails: Pipeline STOPS — developer fixes locally!"]
        A3["Job 2: Type Check (mypy --strict)\n→ Only runs if lint passes! Runs mypy with full strict mode!\nUses cached dependencies from previous run for speed."] --> A4["Job 3: Test Suite\n→ Runs on Python 3.10, 3.11, 3.12 matrix!\n→ pytest discovers all test_*.py files\n→ Coverage measured via pytest-cov plugin!\n→ Parallel execution with xdist if configured!"]
        A5["Job 4: Publish to PyPI\n→ Only runs on tag pushes (v1.0.0)!\n→ Builds sdist + wheel distributions\n→ Uploads to PyPI using API token from secrets!\n→ Like npm publish but via GitHub Actions!"]
    end

    A2 -.->|"CI pipelines are identical in concept:\ntypeScript runs tsc + eslint + jest on push\nPython runs mypy + ruff + pytest on push!"| A3
```

### Key Notes: CI/CD for Python

1. **Run linting first (fastest), then type checking, then testing** — this is the same waterfall pattern as TypeScript projects (tsc → eslint → jest). Each stage gates the next one.
2. **Test against ALL supported Python versions** — use `strategy.matrix` to run tests on 3.10, 3.11, and 3.12 simultaneously. Like how Node.js tests across multiple Node versions in CI.
3. **Publish from GitHub Actions with API tokens** — store your PyPI API token as a GitHub Secret and use the `pypa/gh-action-pypi-publish` action for secure publishing.

---

## 10. Dev Container Setup

### Complete devcontainer.json for Python Development

```json
{
  "name": "Python Development",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": {
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.12",
      "installTools": true
    }
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "charliermarsh.ruff",
        "eamodio.gitlens"
      ],
      "settings": {
        "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
        "python.linting.enabled": true,
        "python.testing.pytestEnabled": true,
        "[python]": {
          "editor.formatOnSave": true,
          "editor.defaultFormatter": "charliermarsh.ruff"
        }
      }
    }
  },
  "postCreateCommand": "pip install -e '.[dev,test]' && pre-commit install",
  "forwardPorts": [8000]
}
```

### Key Notes: Dev Containers for Python

1. **Like VS Code's remote development for TypeScript** — dev containers provide an identical development environment regardless of the host machine's installed tools. For TS projects, you'd use a Node.js base image; for Python, use the official Python dev container image.
2. **pre-commit install in postCreateCommand** — automatically installs git hooks when the container is first created, just like `npm install` runs after cloning a TypeScript project.

---

## 11. Dependency Auditing & Security (pip-audit, safety)

### Complete Security Audit Workflow

```bash
# ========================================
# pip-audit: Find known vulnerabilities in your dependencies!
# ========================================
pip install pip-audit                        # Install the tool
pip-audit                                    # Audit ALL packages in current environment
pip-audit -r requirements.txt                # Audit specific requirement file
pip-audit --show-details                     # Show detailed vulnerability info (CVE IDs!)
pip-audit --fix                              # Auto-upgrade vulnerable packages!
pip-audit --require-hashes -r requirements.txt  # Verify every package's hash!

# ========================================
# safety: Alternative security scanner!
# ========================================
pip install safety
safety check                                 # Check against Safety DB (free + paid tiers)
safety check -r requirements.txt             # Audit specific file
safety check --json                          # JSON output for CI integration!

# ========================================
# Comparison with TypeScript:
# npm audit is built-in; Python requires separate tools!
# Both find CVE vulnerabilities in installed packages.
# Both can auto-fix by upgrading to patched versions!
```

### Key Notes: Dependency Auditing

1. **Python has no built-in security scanner** — `pip-audit` and `safety` are separate tools you must install, unlike `npm audit` which is built into npm. Add both to your CI pipeline for comprehensive coverage.
2. **Use pip-audit's --fix flag in development** — it auto-upgrades vulnerable packages to their patched versions. In CI, use non-fixing mode and fail the build if vulnerabilities are found.
3. **Always pin hashes in production** — `--require-hashes` ensures every downloaded package matches its expected hash, preventing supply chain attacks.

---

## 12. Version Pinning Strategies

### Complete Version Constraint Reference (PEP 440)

| Constraint | Meaning | npm Equivalent | Example |
|-----------|---------|---------------|---------|
| `==2.5.0` | Exact version | `2.5.0` or pinned in lock file | `fastapi==0.104.1` |
| `>=2.0.0` | Minimum version | `^2.0.0` (Poetry) / `~2.0.0` | `numpy>=1.24` |
| `<3.0.0` | Maximum version (exclusive) | — | Used with >= for ranges |
| `>=2.0,<3.0` | Version range | `^2.0.0` (Poetry) | `sqlalchemy>=2.0,<3.0` |
| `~=2.1.0` | Compatible release | `~2.1.0` (npm) | `django~=4.2.0` means >=4.2.0, <4.3.0 |
| `!=2.5.1` | Exclude specific version | — | `pydantic!=2.5.1` (avoid known-bad patch) |
| `>=2.1,<3` | Partial range | `^2.1` in Poetry | `requests>=2.1,<3` |

### Strategy Comparison for Pinning

```
Strategy          | Use Case                          | Example
----------------- | -------------------------------- | ---------------------------
Exact pin         | Production, reproducibility!     | fastapi==0.104.1
Tilde (~)         | Compatible releases              | pydantic~=2.5.0 (^2.5 <3.0)
Caret (^) Poetry  | Allow compatible updates          | fastapi^0.104 = >=0.104, <0.105
Range             | Flexible compatibility            | sqlalchemy>=2.0,<3.0
Exclude (!=)      | Skip known-bad patch versions    | pydantic!=2.5.1

Recommendation: Use Poetry's ^ in development (allows minor updates),
pin exact == in production requirements.txt generated by poetry export!
```

### Mermaid: Dependency Resolution Flow

```mermaid
flowchart TD
    subgraph DEP_RESOLUTION["Dependency Resolution Process (Poetry/pdm/uv)"]
        direction TB
        D1["Developer adds dependency:\npyproject.toml → fastapi = ^0.104.1\n→ Means: >=0.104.1, <0.105.0 (semver-compatible!)"] --> D2["Resolver computes the FULL dependency graph!\n→ fastapi needs: starlette, pydantic-core, typing-extensions\n→ pydantic needs: annotated-types, pydantic-core\n→ ALL transitive dependencies resolved!"]
        D3["Lock file generated:\npoetry.lock (or uv.lock) with EXACT versions of EVERY dependency!\n→ Like package-lock.json but computed by a smarter resolver!"] --> D4["All developers install the SAME versions!\ndeterministic builds across all environments → no 'works on my machine' bugs!"]
    end

    subgraph TS_NPM_RESOLUTION["TypeScript/npm Resolution (for comparison)"]
        direction TB
        N1["package.json → express: ^4.18.2\n→ Means: >=4.18.2, <5.0.0 (more permissive than Poetry!)"] --> N2["npm resolves all deps → installs into node_modules/\n→ generates package-lock.json with exact versions!"]
    end

    D3 -.->|"Both produce lock files for deterministic builds:\ntypeScript uses npm + package-lock.json\nPython uses Poetry/pdm/uv + poetry.lock/uv.lock!"| N2
```

---

## 13. Code Review Checklists for Python Projects

### Pull Request Checklist (Copy/Paste Ready)

- [ ] **Type Annotations**: All public functions have type hints (checked by mypy --strict in CI)
- [ ] **No Unintended `Any`**: No `typing.Any` used unless absolutely necessary (with justification comment)
- [ ] **Error Handling**: Exceptions are caught and handled appropriately, not silently swallowed with bare `except:`
- [ ] **Import Organization**: Imports sorted by ruff isort rules (checked in CI)
- [ ] **No Print Statements**: No `print()` calls in production code (use logging module instead)
- [ ] **Security Review**: No hardcoded secrets, API keys, or passwords in source code
- [ ] **Dependency Audit**: `pip-audit` passes with no vulnerabilities (checked in CI)
- [ ] **Test Coverage**: New code has tests; coverage hasn't dropped below threshold
- [ ] **Type Compatibility**: All type changes are backward compatible or have migration notes
- [ ] **Documentation**: Docstrings added/updated for all new public functions and classes

---

## 14. Complete Tooling Comparison Matrix (TypeScript → Python)

| TypeScript Concept | Python Equivalent | Tool | Configuration File | Notes |
|-------------------|------------------|------|--------------------|-------|
| **Package Manager** | npm/pnpm/yarn | pip / Poetry / pdm / uv | package.json → pyproject.toml | Poetry is the closest to npm's feature set |
| **Virtual Environment** | nvm (Node version manager) | venv / conda / poetry env / uv venv | .nvmrc → pyvenv.cfg | Python's venv manages interpreter AND packages |
| **Lock File** | package-lock.json | poetry.lock / uv.lock / Pipfile.lock | none → pyproject.toml | Plain pip has NO lock file; use Poetry/pdm/uv |
| **Type Checker** | tsc (TypeScript compiler) | mypy | tsconfig.json → pyproject.toml [tool.mypy] | mypy is pure static analysis, never modifies code |
| **Linter** | ESLint | ruff | .eslintrc → pyproject.toml [tool.ruff.lint] | ruff does linting + formatting in one tool! |
| **Formatter** | Prettier | ruff format / black | .prettierrc → pyproject.toml [tool.ruff.format] | ruff format is now the recommended over black (20x faster) |
| **Security Lint** | eslint-security | bandit (or ruff's S rules) | .bandit → pyproject.toml [tool.bandit] | ruff includes bandit-compatible S rules natively |
| **Test Runner** | Jest | pytest + plugins | jest.config.js → [tool.pytest.ini_options] | pytest uses modular plugins; Jest is monolithic |
| **Mocking** | jest.mock() | unittest.mock.patch / pytest-mock | N/A | More powerful than Jest's mocking API |
| **Git Hooks** | Husky + lint-staged | pre-commit | .husky → .pre-commit-config.yaml | Pre-commit manages tool downloads automatically |
| **HTTP Client** | axios/fetch/https | requests/httpx/aiohttp | N/A (third-party packages) | HTTP is built into Node.js stdlib; Python needs packages |
| **File System** | fs/path (stdlib) | pathlib/os/subprocess (stdlib!) | N/A | Python's stdlib covers more ground natively! |
| **Logging** | console.log / Winston | logging module (stdlib!) | N/A | Python's built-in logging rivals Winston/Pino |
| **CI/CD** | GitHub Actions (Node.js) | GitHub Actions (Python) | .github/workflows/node.yml → python.yml | Same platform, different matrix and tooling! |
| **Docker Base** | node:20-slim | python:3.12-slim | Dockerfile.base-node → Dockerfile.base-python | Multi-stage builds work identically in both! |
| **Dev Container** | @devcontainers/nodejs | @devcontainers/python | devcontainer.json (similar setup) | Same VS Code feature, different base images! |
| **Build Bundler** | webpack/esbuild | N/A (Python doesn't bundle!) | N/A | Python apps don't need bundling — they run natively! |
| **Template Engine** | Pug/EJS | Jinja2 / Mako | N/A | Template engines are for web frameworks, not core tooling |

### Mermaid: Complete Tooling Ecosystem Map

```mermaid
flowchart TD
    subgraph TS_TOOLS["TypeScript Toolchain"]
        direction TB
        T1["tsc → Type Check + Compile .ts to .js\n(tsconfig.json)"] --> T2["ESLint → Lint code quality (.eslintrc)\nPrettier → Format code consistently (.prettierrc)"]
        T3["Jest → Test runner + assertions + mocking (jest.config.js)\nHusky + lint-staged → Git hooks (.husky/)"]
        T4["npm/webpack/esbuild → Package management + bundling (package.json)\ngithub-actions/node.yml → CI/CD pipeline"]
    end

    subgraph PY_TOOLS["Python Toolchain — Every TS tool mapped!"]
        direction TB
        P1["mypy → Type Check ONLY (never compiles!) (pyproject.toml [tool.mypy])\n→ No compilation step needed in Python!\nPython types are runtime metadata, not compile output!"] --> P2["ruff check → Lint code quality (pyproject.toml [tool.ruff.lint])\nruff format → Format code consistently (pyproject.toml [tool.ruff.format])]"]
        P3["pytest + plugins → Test runner + assertions + mocking\npytest-cov → Coverage, pytest-asyncio → Async tests, pytest-xdist → Parallel! (.pytest.ini)\npre-commit → Git hooks (.pre-commit-config.yaml)"] --> P4["Poetry/pdm/uv → Package management (pyproject.toml + poetry.lock)\ngithub-actions/python.yml → CI/CD pipeline (SAME format as TS!)"]
    end

    T1 -.->|"tsc compiles .ts → .js\nmypy only CHECKS types, never produces output!"| P1
    T2 -.->|"ESLint + Prettier = TWO tools\nruff = ONE tool that does BOTH!"| P2
```

---

## 15. Poetry vs pdm vs uv — Complete Benchmark Comparison

### Feature Comparison Matrix

| Feature | Poetry | pdm | uv | Winner |
|---------|--------|-----|----|--------|
| **Dependency Resolution** | Satisfiability-based (SAT solver) | SAT solver (similar to Poetry) | SAT solver (Rust implementation) | uv (fastest) |
| **Lock File Format** | poetry.lock (Poetry-specific) | pdm.lock (PEP-compliant) | uv.lock (Rust-native) | pdm (more portable) |
| **venv Management** | Automatic (creates/manages .venv) | Automatic (creates/manages .venv) | `uv venv` explicit command | Poetry (most seamless) |
| **Build System** | poetry-core (built-in) | hatchling (separate) | built-in (fastest!) | uv (Rust, no external deps) |
| **Workspace Support** | Limited (via poetry-plugin-workspace) | Native PEP 735 workspaces | Native workspace support! | pdm/uv (tie) |
| **Python Version Manager** | External (asdf/pyenv) | Built-in (`pdm python install`) | Built-in (`uv python install`!) | uv (fastest install) |
| **Plugin Ecosystem** | Large (many official + community plugins) | Growing plugin system | Minimal (by design — core features are built-in) | Poetry |
| **Publishing to PyPI** | `poetry publish` (built-in) | `pdm publish` (built-in) | `uv publish` (API token auth!) | uv (fastest + simplest) |
| **Start-up Time** | ~2-5s (Python, heavy deps) | ~3-6s (Python, many deps) | ~10-50ms (Rust!) | uv by a massive margin! |
| **Install Speed** | ~1-10s per project | ~2-8s per project | ~10-100ms per project | uv wins hands down! |
| **Resolution Speed** | ~500ms - 5s (depends on dep count) | ~1-10s (Python resolver) | ~5-50ms (Rust resolver!) | uv is 100x faster than Poetry/pdm! |
| **PEP Compliance** | Partial (uses non-standard pyproject sections) | Full PEP 621 compliance | Full PEP compliance | pdm (most standards-compliant) |
| **Community Adoption** | Largest community, most tutorials, most SO answers | Growing fast, excellent docs | Rapidly growing (Astral backing!) | Poetry (current), uv (future) |

### Mermaid: Tool Comparison with Decision Flow

```mermaid
flowchart TD
    subgraph TOOL_SELECTION["How to Choose Python Package Manager"]
        direction TB
        Q1{"Do you need data science / GPU support?"} -->|"Yes"| S1["Use conda/mamba → manages C/CUDA libraries\nPython data science standard since 2008"]
        Q1 -->|"No" | Q2{"Is speed your top priority?"} -->|"Yes"| S3["Use uv → 10-100x faster than Poetry/pdm\nRust-based, instant dependency resolution!\nThe future of Python tooling!"]
        Q2 -->|"No" | Q3{"Do you need maximum ecosystem support?"} -->|"Yes"| S4["Use Poetry → Largest community, most tutorials, widely adopted by industry"]
        Q3 -->|"No"| Q4{"Do you need strict PEP compliance + workspaces?"} -->|"Yes"| S5["Use pdm → Full PEP 621 compliance\nBest workspace support outside uv"]
    end

    classDef tool fill:#e44d26,color:white
    classDef rec fill:#3069bd,color:white
    classS1,S3,S4,S5 
```

### Benchmark Numbers (Approximate, on a typical laptop)

| Operation | npm install | poetry install | pdm install | uv sync |
|-----------|------------|---------------|-------------|---------|
| **Start-up time** | ~500ms | ~3s | ~4s | ~20ms |
| **Dependency resolution** (10 deps) | ~800ms | ~2s | ~3s | ~15ms |
| **Download + install** (10 deps) | ~5s | ~8s | ~10s | ~50ms |
| **Cold start (no cache)** | ~10s | ~15s | ~18s | ~200ms |
| **Warm start (with cache)** | ~3s | ~4s | ~5s | ~30ms |

---

## 16. Quizzes with Answers (25+)

### Quiz 1: Virtual Environments
**Q:** What is the key difference between Python's `venv` and Node.js's `node_modules/`?
<details><summary>Show Answer</summary>
Python's venv creates a completely separate Python interpreter + packages directory. Node.js's node_modules only isolates packages within a shared global Node.js interpreter. With venv, two projects can run different Python versions simultaneously without nvm.
</details>

### Quiz 2: Virtual Environments
**Q:** What file does venv create to record the Python version used?
<details><summary>Show Answer</summary>
`pyvenv.cfg` — it contains `home` (path to Python executable) and `version` (Python version).
</details>

### Quiz 3: Package Management
**Q:** Which tool generates a lock file by default when you run `pip install`?
<details><summary>Show Answer</summary>
None! Plain pip does NOT generate any lock file. You must use Poetry (poetry.lock), pdm (pdm.lock), uv (uv.lock), or pip-tools (Pipfile.lock) to get deterministic dependency resolution.
</details>

### Quiz 4: Package Management
**Q:** What is the Python equivalent of `npm audit`?
<details><summary>Show Answer</summary>
There is no built-in equivalent. Use `pip-audit` or `safety` as separate tools: `pip install pip-audit && pip-audit`.
</details>

### Quiz 5: Poetry
**Q:** What does `poetry add --dev pytest` do?
<details><summary>Show Answer</summary>
It adds pytest to the `[tool.poetry.group.dev.dependencies]` section of pyproject.toml, resolves all dependencies including transitive ones, updates poetry.lock with exact versions, and installs into the managed virtual environment.
</details>

### Quiz 6: Poetry
**Q:** How does Poetry's version constraint `^0.104.1` differ from npm's `^0.104.1`?
<details><summary>Show Answer</summary>
For versions < 1.0.0, the caret means different things: Python/ Poetry's ^0.104.1 = >=0.104.1, <0.105.0 (changes only the last specified digit). npm's ^0.104.1 = >=0.104.1, <0.105.0 — actually they are THE SAME for minor versions! The difference appears at major version: Python 1.0.0+ uses Poetry's convention, npm's ^ always allows any compatible major/minor within the range.
</details>

### Quiz 7: mypy
**Q:** What is the equivalent of TypeScript's `"strict": true` in mypy?
<details><summary>Show Answer</summary>
`[tool.mypy] strict = true` or `mypy --strict`. It enables ALL available type checking warnings including disallow_untyped_defs, warn_return_any, check_untyped_defs, and more.
</details>

### Quiz 8: mypy
**Q:** What does the pydantic.mypy plugin provide that makes mypy work with Pydantic?
<details><summary>Show Answer</summary>
Without the plugin, mypy doesn't understand Pydantic's runtime magic (field validators, computed fields, generic models). The plugin teaches mypy about Pydantic's model metaclass so it can properly type-check instantiation, field access, and validation errors.
</details>

### Quiz 9: mypy
**Q:** What is the difference between `ignore_missing_imports = true` and `follow_imports = "skip"`?
<details><summary>Show Answer</summary>
`ignore_missing_imports = true` silences "Could not resolve import" errors but still tries to analyze the imported module's code. `follow_imports = "skip"` completely ignores missing imports — mypy treats them as having type `Any` without even looking at their source. The latter is faster for large projects with many untyped dependencies.
</details>

### Quiz 10: Linting
**Q:** Why does ruff do both linting and formatting while TypeScript needs separate tools?
<details><summary>Show Answer</summary>
ruff was designed from scratch in Rust as a unified tool, combining the functionality of flake8, pylint, isort, black, and more into one binary. ESLint was designed only for linting; Prettier was added later for formatting. TypeScript's ecosystem evolved organically with separate tools for each concern.
</details>

### Quiz 11: Linting
**Q:** What does `select = ["E", "F", "I", "B"]` configure in ruff?
<details><summary>Show Answer</summary>
- E/W: pycodestyle errors/warnings (syntax issues like missing semicolons, trailing whitespace)
- F: Pyflakes (unused imports, undefined names, redefined variables)
- I: isort (import sorting rules)
- B: flake8-bugbear (common programming mistakes like mutable default args, bare excepts)
</details>

### Quiz 12: Testing
**Q:** What does `pytest-xdist` enable that Jest cannot do natively?
<parameter_answer>Show Answer</summary>
pytest-xdist runs test functions across MULTIPLE CPU cores simultaneously (distributed parallel execution). Each worker process runs independently. Jest's `--workers` flag also parallelizes, but xdist can distribute tests across different machines in CI with the right setup.
</details>

### Quiz 13: Testing
**Q:** What does the hypothesis library provide that standard pytest cannot?
<parameter_answer>Show Answer</summary>
Property-based testing: instead of writing individual test cases with specific inputs, you define INVARIABLES (properties that must always hold), and hypothesis automatically generates hundreds of random inputs to find counterexamples. Like fast-check for TypeScript — it finds edge cases you wouldn't think to test manually.
</details>

### Quiz 14: Testing
**Q:** What is the difference between `pytest` discovery patterns `test_*.py` vs `*_test.py`?
<parameter_answer>Show Answer</summary>
Both patterns are supported by default. Files matching `test_*.py` contain test functions that start with `test_`. Files matching `*_test.py` also contain test functions starting with `test_`. You can customize both patterns in `[tool.pytest.ini_options]` via the `python_files` config option.
</details>

### Quiz 15: Pre-commit
**Q:** Why does pre-commit only run hooks on staged files (not all files)?
<parameter_answer>Show Answer</summary>
Staged-file-only execution is dramatically faster — if you modified one file in a 10,000-file codebase, mypy only analyzes that one file (milliseconds instead of minutes). It also prevents "preventing commits because unrelated files have issues" — only YOUR changes are checked.
</details>

### Quiz 16: Pre-commit
**Q:** How do you bypass pre-commit hooks when needed (e.g., WIP commits)?
<parameter_answer>Show Answer</summary>
`git commit --no-verify` or `pre-commit disable` temporarily. The `--no-verify` flag skips all git hooks (pre-commit, commit-msg, etc.). Use sparingly and never for code you plan to merge.
</details>

### Quiz 17: Docker
**Q:** Why use multi-stage builds for Python in Docker?
<parameter_answer>Show Answer</summary>
Stage 1 installs build dependencies (C compilers, headers) needed to compile C extensions. Stage 2 copies only the installed runtime packages into a fresh slim image without build tools. This produces production images <200MB vs >1GB with single-stage builds containing build tools.
</details>

### Quiz 18: Docker
**Q:** What does `ENV PYTHONUNBUFFERED=1` do in a Docker container?
<parameter_answer>Show Answer</summary>
It disables Python's output buffering so that `print()` statements and logging appear immediately in docker logs instead of being buffered until the process exits. Critical for debugging real-time log output.
</details>

### Quiz 19: CI/CD
**Q:** Why should linting be the first job in a GitHub Actions pipeline?
<parameter_answer>Show Answer</summary>
Linting is the fastest check (< 10 seconds) and catches 80% of common mistakes (formatting, unused imports, syntax errors). Running it first provides instant feedback without wasting CI minutes on type-checking or tests for code that has basic style issues.
</details>

### Quiz 20: Version Pinning
**Q:** What does PEP 440's `~=2.1.0` constraint actually mean?
<parameter_answer>Show Answer</summary>
"Compatible release": >=2.1.0, <2.2.0. It allows patch updates (bug fixes) but not minor version changes. Like npm's ~2.1.0. Equivalent to Poetry's ^2.1 for versions >= 1.0.
</details>

### Quiz 21: Dependency Auditing
**Q:** What is the difference between pip-audit and safety?
<parameter_answer>Show Answer</summary>
pip-audit uses PyVD (Python Vulnerability Database) which pulls from multiple sources including OS vulnerability databases. safety uses its own Safety DB with manual curation. pip-audit is faster and more comprehensive; safety has a longer history and detailed explanations for each vulnerability. Use both in CI for maximum coverage.
</details>

### Quiz 22: Tooling Matrix
**Q:** Which Python tool replaces BOTH ESLint AND Prettier?
<parameter_answer>Show Answer</summary>
ruff does both linting (via ruff check) and formatting (via ruff format). It's written in Rust and runs 10-100x faster than ESLint. For TypeScript, you need two separate tools; Python has one unified tool that replaces both.
</details>

### Quiz 23: Tooling Matrix
**Q:** What is the mypy equivalent of TypeScript's `skipLibCheck`?
<parameter_answer>Show Answer</summary>
There isn't a direct `skipLibCheck` equivalent in mypy because Python doesn't have type declaration files (.d.ts). Instead, use `ignore_missing_imports = true` to silently ignore untyped third-party packages — this is the closest behavior.
</details>

### Quiz 24: Tooling Matrix
**Q:** What is uv's advantage over Poetry in workspace support?
<parameter_answer>Show Answer</summary>
uv has native workspace support (like pnpm workspaces for Node.js) built into the core tool — no plugins needed. It manages a single lock file for all workspace packages and can sync/resolve dependencies across all workspace members in one command. Poetry requires a third-party plugin for workspace support.
</details>

### Quiz 25: Tooling Matrix
**Q:** What is the Python equivalent of TypeScript's `@types/*` packages?
<parameter_answer>Show Answer</summary>
There is NO equivalent because Python types are IN the source code itself, not in separate declaration files. When you import `from fastapi import FastAPI`, the type information comes from the package's own source annotations — no extra @types/fastapi installation needed! This is one of Python's advantages over TypeScript's ecosystem fragmentation.
</details>

---

## 17. Exercises with Solutions (15+)

### Exercise 1: Complete Virtual Environment Setup
**Task:** Create a fully isolated Python project from scratch with venv, install dependencies, and verify isolation between two environments.

<details><summary>Solution</summary>

```bash
# Project A: FastAPI server
mkdir project-a && cd project-a
python -m venv .venv
source .venv/bin/activate          # Linux/macOS
pip install fastapi pydantic uvicorn --upgrade
pip freeze > requirements.txt     # Save deps
python -c "import fastapi; print(fastapi.__version__)"

# Project B: Data science project (different Python version simulation)
mkdir ../project-b && cd ../project-b
python -m venv .venv
source .venv/bin/activate
pip install numpy pandas --upgrade
python -c "import numpy; print(numpy.__version__)"
```

</details>

### Exercise 2: Poetry Project with All Configuration Sections
**Task:** Create a complete pyproject.toml for a web API project with production deps, dev deps, optional deps, docs group, CLI scripts, and publishing configuration.

<details><summary>Solution</summary>

```toml
[tool.poetry]
name = "api-server"
version = "1.0.0"
description = "Production REST API"
authors = ["Dev <dev@example.com>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.10"
fastapi = {version = "^0.104.1", extras = ["standard"]}
sqlalchemy = ">=2.0,<3.0"
redis = {version = "^5.0.0", optional = true}

[tool.poetry.group.dev.dependencies]
pytest = "^7.4"
mypy = "^1.8"
ruff = "^0.1"

[tool.poetry.scripts]
api-cli = "api_server.cli:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

</details>

### Exercise 3: mypy Strict Configuration
**Task:** Configure mypy for a project that requires 100% type coverage, no Any types, and strict checking. Create the configuration and write code that triggers different mypy errors.

<details><summary>Solution</summary>

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.12"
strict = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
warn_return_any = true
warn_unused_configs = true
ignore_missing_imports = true

[mypy-my_package.untyped_module.*]
disallow_untyped_defs = false  # Allow untyped code in legacy modules!
```

```python
# test_code.py — This file triggers multiple mypy errors!
from typing import Any

def add(x, y):                        # ERROR: No type annotations (disallowed by strict)!
    return x + y                      # If x and y are Any, this is also flagged!

def bad_return(items: list) -> str:   # Declares str but might return int!
    if items:
        return len(items)             # ERROR: Incompatible return type (int vs str)!
    return "empty"                    # This one is OK
```

</details>

### Exercise 4: ruff Complete Configuration
**Task:** Create a pyproject.toml with ruff configured for a production project that uses double quotes, has a 100-character line limit, and specific per-file ignores.

<details><summary>Solution</summary>

```toml
[tool.ruff]
target-version = "py312"
line-length = 100
exclude = ["migrations/", ".venv/", "node_modules/"]

[tool.ruff.lint]
select = ["E", "W", "F", "I", "N", "UP", "B", "SIM", "RUF", "C90", "T20", "S"]
ignore = ["E501", "B008"]

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]
"tests/**/*.py" = ["S101"]
"migrations/*.py" = ["E501", "SIM"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
```

</details>

### Exercise 5: Pre-commit Hook Setup
**Task:** Create a complete .pre-commit-config.yaml that runs ruff, mypy, and bandit on every commit. Include both remote repos and a local custom hook.

<details><summary>Solution</summary>

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.15.0
    hooks:
      - id: mypy
        additional_dependencies: [pydantic, types-requests]

  - repo: local
    hooks:
      - id: security-audit
        name: Security audit
        entry: pip-audit
        language: system
        pass_filenames: false
        always_run: true
```

</details>

### Exercise 6: pytest with Fixtures and Async
**Task:** Write a test suite using pytest fixtures (with different scopes), async tests, and mocking.

<details><summary>Solution</summary>

```python
# conftest.py — Shared fixtures
import pytest
from httpx import AsyncClient

@pytest.fixture(scope="session")
def base_url():
    """Available to ALL tests across ALL files!"""
    return "http://testserver"

@pytest.fixture(scope="function")
def mock_user():
    """Available only to individual test functions!"""
    return {"name": "Alice", "age": 30}

# test_api.py
import pytest
from httpx import ASGITransport, AsyncClient
from myapp.main import app

@pytest.mark.asyncio
async def test_get_user(base_url, mock_user):
    """Async test with fixture dependency injection!"""
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url=base_url) as client:
        response = await client.get("/users/1")
        assert response.status_code == 200

@pytest.mark.asyncio
async def test_create_user(mock_user):
    """Test that uses the mock_user fixture!"""
    assert mock_user["name"] == "Alice"
```

</details>

### Exercise 7: hypothesis Property-Based Testing
**Task:** Write property-based tests using hypothesis that verify sorting functions always produce sorted output, even with edge cases.

<details><summary>Solution</summary>

```python
from hypothesis import given, strategies as st
from myapp.sorting import bubble_sort, merge_sort

@given(st.lists(st.integers(), min_size=0, max_size=100))
def test_sort_always_sorted(items):
    """Property: sorting ANY list produces a sorted result!"""
    result = bubble_sort(list(items))  # Work on a copy!
    assert result == sorted(result)

@given(st.lists(st.text(min_size=1), min_size=0, max_size=50))
def test_sort_strings(strings):
    """Test with string lists too!"""
    result = merge_sort(list(strings))
    assert result == sorted(result)
```

</details>

### Exercise 8: Complete Dockerfile for FastAPI
**Task:** Write a multi-stage Dockerfile for a FastAPI application with proper layer caching and production optimizations.

<details><summary>Solution</summary>

```dockerfile
FROM python:3.12-slim AS base
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev && rm -rf /var/lib/apt/lists/*

FROM base AS deps
COPY pyproject.toml poetry.lock ./
RUN pip install --no-cache-dir poetry && \
    poetry config virtualenvs.create false && \
    poetry install --only main

FROM deps AS production
COPY . .
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

</details>

### Exercise 9: GitHub Actions CI/CD Pipeline
**Task:** Create a complete .github/workflows/python.yml that runs linting, type-checking, testing (on 3 Python versions), and publishes to PyPI on tags.

<details><summary>Solution</summary>

```yaml
name: Python CI/CD
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install ruff && ruff check . && ruff format --check .

  test:
    needs: lint
    runs-on: ubuntu-latest
    strategy:
      matrix: { python-version: ["3.10", "3.11", "3.12"] }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: ${{ matrix.python-version }} }
      - run: pip install -e ".[test]" && pytest --cov=src

  publish:
    needs: test
    if: github.ref_type == 'tag'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install build && python -m build
      - uses: pypa/gh-action-pypi-publish@release/v1
        with: { password: ${{ secrets.PYPI_API_TOKEN }} }
```

</details>

### Exercise 10: Dependency Pinning Strategy
**Task:** Create both a development requirements file (with flexible versions) and a production requirements.txt (with exact pinned versions) for the same project.

<details><summary>Solution</summary>

```text
# requirements-dev.in — Development with flexible constraints!
fastapi>=0.104,<0.105
pydantic>=2.5,<3.0
pytest>=7.4
mypy>=1.8

# Run: pip-compile requirements-dev.in -o requirements-dev.txt
# → Generates exact pinned versions for reproducibility!

# requirements.txt — Production with EXACT pins (from poetry export!)
fastapi==0.104.1
pydantic==2.5.0
starlette==0.32.0       # Transitive dep, also pinned!
pydantic-core==2.14.1   # Every transitive dependency pinned!

# Install: pip install -r requirements.txt
# → Every developer gets the EXACT same versions!
```

</details>

### Exercise 11: Complete pdm Project Setup
**Task:** Initialize a project using pdm with all configuration sections (PEP 621 compliant).

<details><summary>Solution</summary>

```bash
pdm init --name my-app --description "A pdm-managed app"
pdm add fastapi pydantic
pdm add --dev-group dev pytest mypy ruff
pdm add --group docs sphinx myst-parser
pdm build
```

```toml
# pyproject.toml (auto-generated by pdm — PEP 621 compliant!)
[project]
name = "my-app"
version = "0.1.0"
description = "A pdm-managed app"
requires-python = ">=3.10"
dependencies = ["fastapi>=0.104.1", "pydantic>=2.5.0"]

[project.optional-dependencies]
dev = ["pytest>=7.4", "mypy>=1.8", "ruff>=0.1"]
docs = ["sphinx>=7.2"]
```

</details>

### Exercise 12: uv Project with All Features
**Task:** Create a project using uv's complete feature set: init, add deps, create venv, sync, build, and publish preparation.

<details><summary>Solution</summary>

```bash
# Initialize project (creates pyproject.toml + .venv instantly!)
uv init my-app && cd my-app

# Add dependencies (auto-updates pyproject.toml + uv.lock!)
uv add fastapi pydantic uvicorn
uv add --dev pytest mypy ruff black pre-commit

# Sync everything from lock file!
uv sync --all-packages

# Build distribution packages
uv build  # Creates dist/my_app-0.1.0.tar.gz and .whl

# Verify installation
uv pip list  # List all installed packages
```

</details>

### Exercise 13: Dependency Auditing Pipeline
**Task:** Create a CI step that runs both pip-audit and safety, failing the build on any found vulnerabilities.

<details><summary>Solution</summary>

```yaml
- name: Security audit (pip-audit)
  run: |
    pip install pip-audit
    pip-audit --require-hashes -r requirements.txt || {
      echo "::error::Vulnerabilities found in dependencies!"
      exit 1
    }

- name: Security audit (safety)
  run: |
    pip install safety
    safety check -r requirements.txt --json || exit 1
```

</details>

### Exercise 14: Complete Dev Container Configuration
**Task:** Create a devcontainer.json that sets up Python development with ruff, mypy, pytest, and VS Code extensions.

<details><summary>Solution</summary>

```json
{
  "name": "Python Development",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "charliermarsh.ruff",
        "ms-python.vscode-pylance"
      ],
      "settings": {
        "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
        "[python]": {
          "editor.formatOnSave": true,
          "editor.defaultFormatter": "charliermarsh.ruff"
        }
      }
    }
  },
  "postCreateCommand": "pip install -e '.[dev]' && pre-commit install",
  "forwardPorts": [8000]
}
```

</details>

### Exercise 15: Code Review Checklist Automation
**Task:** Create a reusable GitHub Actions workflow template for Python code reviews with all the checklist items automated.

<details><summary>Solution</summary>

```yaml
name: Python Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }  # Need full history for diff analysis!

      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }

      - name: Check type annotations
        run: mypy src/ --strict --json-format > mypy-results.json || true

      - name: Check import organization
        run: ruff check --select I .

      - name: Check for print statements
        run: grep -rn "print(" src/ --include="*.py" || echo "No print statements found ✓"

      - name: Security scan
        run: pip-audit -r requirements.txt

      - name: Run all tests
        run: pytest --tb=short --cov=src --cov-fail-under=80
```

</details>

---

## Next

> **Next:** [Module 16 — Node.js Built-in Modules vs Python Equivalents (Complete Deep Dive)](./16-nodejs-python-equivalents.md)

---

*This module covers every major tool in the Python ecosystem with exhaustive comparison to TypeScript/Node.js equivalents. The Python tooling landscape has matured significantly — Poetry for dependencies, ruff for linting/formatting, mypy for type checking, pytest for testing, pre-commit for git hooks, and uv as the ultra-fast modern alternative. Use this reference to map your TypeScript toolchain knowledge directly to Python equivalents.*
