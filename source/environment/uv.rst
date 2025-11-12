High‑Performance Dependency Management with uv
===============================================

Overview
--------

**uv** is a blazing‑fast Python package installer, dependency resolver, and environment
manager written in **Rust**, developed by *Astral* (the creators of Ruff, Rye, and Pixi).

It is designed as a drop‑in replacement for ``pip`` and ``poetry`` installation steps,
providing **massive performance improvements** and **deterministic reproducibility**.

With ``uv``, you can:

* Resolve and install dependencies up to **10–100× faster** than pip
* Work directly with a ``pyproject.toml`` or ``requirements.txt``
* Manage virtual environments automatically
* Generate and sync deterministic lock files
* Integrate easily with other Python tools (`hatch`, `poetry`, `rye`, etc.)

``uv`` focuses purely on **speed, robustness, and interoperability**, making it ideal for
both development and CI/CD pipelines.

---

Installation
------------

Install ``uv`` globally with a simple script (no Python dependency required):

.. code-block:: bash

   curl -LsSf https://astral.sh/uv/install.sh | sh

or with ``pipx``:

.. code-block:: bash

   pipx install uv

Verify installation:

.. code-block:: bash

   uv version


**Upgrade at any time:**

.. code-block:: bash

   uv self upgrade

---

Getting Started
---------------

You can use ``uv`` in place of ``pip``:

.. code-block:: bash

   uv pip install requests

To initialize a new project:

.. code-block:: bash

   uv init myproject
   cd myproject

This creates:

::
   myproject/
   ├── pyproject.toml
   ├── uv.lock
   └── .venv/

The ``uv.lock`` file ensures consistent, reproducible installs.

---

Working with Dependencies
-------------------------

### Install from pyproject.toml

If your project uses ``pyproject.toml``:

.. code-block:: bash

   uv add fastapi "uvicorn[standard]"

This updates both your manifest and lock file automatically.

To install everything defined in your project:

.. code-block:: bash

   uv sync

That command:

1. Reads dependencies from ``pyproject.toml``
2. Resolves and installs them
3. Creates a virtual environment (if not already present)
4. Reuses the cached wheels for future speed

### Install from requirements.txt

``uv`` can also read traditional files:

.. code-block:: bash

   uv pip install -r requirements.txt

---

Creating and Managing Environments
----------------------------------

``uv`` automatically manages isolated virtual environments.
You can view or activate them easily:

.. code-block:: bash

   uv venv create    # create a new environment
   uv venv list      # list existing environments
   uv venv activate  # activate the default environment

By default, the environment is stored in ``.venv`` for your project.

You can configure this with:

.. code-block:: bash

   uv config set venv.in-project true

---

Lock Files
----------

The heart of ``uv`` is its **deterministic lockfile** (``uv.lock``), which ensures the same resolution
for all developers and CI systems.

Generate or update the lock file:

.. code-block:: bash

   uv lock

Reinstall exactly from the lock file (without resolving):

.. code-block:: bash

   uv sync --locked

Compare or inspect lock content:

.. code-block:: bash

   uv lock diff
   uv lock list

---

Unifying with Other Tools
--------------------------

``uv`` is designed to **coexist** with tools like:
- **hatch** (for builds, versioning)
- **poetry** (for project metadata)
- **rye** (as the higher‑level runtime manager)
- **pixi** (for cross‑language environments)

Common integration examples:

### With Hatch

.. code-block:: bash

   # Use uv for fast dependency resolution
   uv sync
   # Then build with hatch
   hatch build

### With Poetry

.. code-block:: bash

   # Install dependencies using uv for speed
   uv sync --from pyproject.toml
   # Lock normally with poetry if publishing
   poetry build

---

Performance Comparison
----------------------

| Operation | pip | uv |
|------------|------|------|
| Install 100 packages (cold cache) | ~90 s | **~4 s** |
| Install with cache | ~35 s | **~1 s** |
| Resolve complex tree | Slow back‑tracking | **Linear‑time resolver** |
| Package caching | Basic wheel cache | **Multi‑platform binary cache** |
| Lock file | No native lock | **`uv.lock` deterministic** |

More benchmarks are available on `astral.sh/uv` and GitHub.

---

Advanced Usage
--------------

### Global vs Local Installs

You can install packages to specific scopes:

.. code-block:: bash

   uv tool install black        # global tool like pipx
   uv pip install requests      # local project install

### Using a Specific Python Interpreter

.. code-block:: bash

   uv python install 3.13
   uv python pin 3.13
   uv sync

This ensures that all collaborators use the same Python runtime.

### Check for Security Updates

.. code-block:: bash

   uv inspect requests

   # with security flag
   uv check --security

---

Integrating with CI/CD
----------------------

Example: **GitHub Actions**

.. code-block:: yaml

   name: CI
   on: [push, pull_request]
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Setup uv
           run: curl -LsSf https://astral.sh/uv/install.sh | sh
         - name: Install dependencies
           run: uv sync --locked
         - name: Run tests
           run: uv run pytest

**Tip**: Since ``uv`` downloads prebuilt wheels in parallel, CI builds
are typically 10‑30× faster and require no pip cache configuration.

---

Configuration
-------------

List or change settings using:

.. code-block:: bash

   uv config list
   uv config get venv.in-project
   uv config set cache-dir ~/.cache/uv

You can also define project‑specific configurations inside ``pyproject.toml``:

.. code-block:: toml

   [tool.uv]
   python = "3.12"
   cache-dir = ".uv-cache"

---

Best Practices
--------------

1. **Commit both `pyproject.toml` and `uv.lock`**
   This guarantees reproducibility across developers and CI.

2. **Use `uv sync --locked` in CI/CD**
   Avoid re‑resolving dependencies during automated builds.

3. **Integrate with higher‑level tools**
   Let ``hatch`` or ``poetry`` handle metadata and building, while ``uv`` manages fast installation.

4. **Prefer local virtual environments (`.venv/`)**
   Keeps setups portable and avoids polluting the global Python environment.

5. **Clean cache periodically**
   Run ``uv cache clean`` to remove stale wheels, saving disk space.

---

Comparison with Other Tools
---------------------------

| Tool | Language | Focus | Locking | Speed | Typical Use |
|------|-----------|--------|----------|--------|--------------|
| pip | Python | Installing single deps | No | Slow | Legacy installs |
| poetry | Python | Full project lifecycle | Yes (poetry.lock) | Moderate | Apps and libs |
| hatch | Python | Build, version, environments | Yes (hatch.lock) | Fast | Libraries, CI |
| uv | Rust | Install & resolve dependencies | Yes (uv.lock) | **Very fast** | Any environment |
| pixi | Rust | Multi‑language env mgmt | Partial | Fast | Data / AI |
| rye | Rust | Project orchestration | Yes | Fast | Unified workflow |

---

Conclusion
----------

``uv`` represents the next generation of Python dependency management:
**fast, deterministic, cross‑platform, and tool‑agnostic**.

It plays well with existing projects that use poetry, hatch, or rye, offering a
drop‑in way to accelerate installs and improve reproducibility.

If you value *performance, simplicity, and reliability*,
then making ``uv`` part of your modern Python toolchain is a clear best practice.