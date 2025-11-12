Building and Managing Python Projects with Hatch
================================================

Overview
--------

**Hatch** is a modern, powerful Python project management tool that unifies
project creation, package building, version management, and environment handling
— all while remaining **lightweight**, **extensible**, and **standards‑compliant**.

Unlike Poetry, which focuses heavily on dependency resolution,
Hatch emphasizes **speed**, **flexibility**, and **plugin‑based extensibility**.
It fully supports the modern ``pyproject.toml`` standard (PEP 517/518/621).

Hatch can:

* Create new projects quickly from templates
* Manage multiple isolated environments (for dev, test, docs, etc.)
* Build wheels and source distributions
* Automatically bump, sync, or derive versions
* Integrate deeply with modern CI pipelines

---

Installation
------------

Install Hatch globally using ``pipx`` or ``pip``:

.. code-block:: bash

   pipx install hatch
   # or
   pip install --user hatch

Confirm installation:

.. code-block:: bash

   hatch --version

---

Creating a New Project
----------------------

To scaffold a new project with a default structure:

.. code-block:: bash

   hatch new mypackage
   cd mypackage

The generated structure:

::

   mypackage/
   ├── README.md
   ├── pyproject.toml
   ├── src/
   │   └── mypackage/
   │       └── __init__.py
   └── tests/
       └── test_mypackage.py

All configuration lives in ``pyproject.toml``, compliant with PEP 621:

.. code-block:: toml

   [project]
   name = "mypackage"
   version = "0.1.0"
   authors = [{name = "Alice", email = "alice@example.com"}]
   description = "A modern example project using Hatch"
   readme = "README.md"
   requires-python = ">=3.9"
   dependencies = ["requests >=2.31"]

---

Managing Environments
---------------------

Hatch provides built‑in **environment management**, similar to ``tox`` or ``virtualenv``,
but declared directly in ``pyproject.toml``.

Example configuration:

.. code-block:: toml

   [tool.hatch.envs.default]
   dependencies = [
       "pytest",
       "black",
       "ruff",
   ]

   [tool.hatch.envs.docs]
   dependencies = [
       "sphinx",
       "furo",
   ]

Use commands:

.. code-block:: bash

   hatch env show          # list available environments
   hatch run pytest        # run pytest within default env
   hatch run docs:sphinx-build -b html docs build/html

You can also activate a shell inside an environment:

.. code-block:: bash

   hatch shell

Each environment is stored locally (in ``.hatch/`` by default),
ensuring reproducible and isolated execution contexts.

---

Dependency Management
---------------------

To add or update dependencies, edit ``pyproject.toml`` directly
or use Hatch’s CLI shortcuts:

.. code-block:: bash

   hatch dep add fastapi
   hatch dep add --dev pytest-cov

To freeze (lock) the current resolved dependencies:

.. code-block:: bash

   hatch dep lock

This creates ``hatch.lock`` for deterministic builds.

---

Version Management
------------------

Hatch can automatically bump and sync package versions across files.

Increase version numbers automatically:

.. code-block:: bash

   hatch version patch   # increments 0.1.0 → 0.1.1
   hatch version minor   # increments 0.1.1 → 0.2.0
   hatch version major   # increments 0.2.0 → 1.0.0

Read the version in your project configuration:

.. code-block:: bash

   hatch version

You can also derive the version dynamically (for example from Git tags):

.. code-block:: toml

   [tool.hatch.version]
   source = "vcs"  # use git tag (e.g., v1.2.0) as version source

---

Building and Packaging
----------------------

Hatch uses its own modern, PEP‑517‑compatible build backend (``hatchling``),
which is optimized for speed and minimal dependencies.

To build your package:

.. code-block:: bash

   hatch build

Generated artifacts:

::

   dist/
   ├── mypackage-0.1.0.tar.gz
   └── mypackage-0.1.0-py3-none-any.whl

To verify files included in the package:

.. code-block:: bash

   hatch build --list

Publish directly to PyPI (or any custom repository):

.. code-block:: bash

   hatch publish
   # custom repository
   hatch publish --repo testpypi

---

Testing and Automation
----------------------

Hatch encourages a “one command per environment” discipline.

Run tests:

.. code-block:: bash

   hatch run test:pytest

If you define scripts in the environment configuration:

.. code-block:: toml

   [tool.hatch.envs.test]
   dependencies = ["pytest", "coverage[toml]"]
   scripts.test = "pytest -v"

You can then run simply:

.. code-block:: bash

   hatch run test

---

Linting and Formatting Workflows
--------------------------------

A robust Hatch configuration for development may look like this:

.. code-block:: toml

   [tool.hatch.envs.dev]
   dependencies = ["ruff", "black", "mypy"]

   [tool.hatch.envs.dev.scripts]
   lint = "ruff src tests"
   format = "black src tests"
   typecheck = "mypy src"

Now you can run:

.. code-block:: bash

   hatch run dev:lint
   hatch run dev:format
   hatch run dev:typecheck

This lightweight structure replaces heavy tools like ``tox`` or large Makefiles.

---

Using Hatch with CI/CD
----------------------

Integrate Hatch easily in continuous integration pipelines.

**Example: GitHub Actions**

.. code-block:: yaml

   name: CI
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-python@v5
           with:
             python-version: "3.12"
         - run: pip install hatch
         - run: hatch run test

**Deterministic builds** in CI:

.. code-block:: bash

   hatch dep sync --frozen

ensures the environment exactly matches ``hatch.lock``.

---

Extending Hatch with Plugins
----------------------------

Hatch supports a plugin system, allowing you to extend functionality without
modifying the core. Popular community plugins provide additional environment types,
dynamic metadata sources, or specialized builders.

List available plugins:

.. code-block:: bash

   hatch plugin show

Install a plugin:

.. code-block:: bash

   pip install hatch-fancy-pypi-readme

Configure it in ``pyproject.toml`` as usual for any hatch extension.

---

Best Practices
--------------

1. **Single `pyproject.toml` configuration**
   Use ``pyproject.toml`` as the central manifest for all metadata, dependencies, and environments.

2. **Use isolated environments per task**
   Define distinct envs (``dev``, ``test``, ``docs``) to avoid dependency interference.

3. **Lock dependencies for CI/CD**
   Commit your ``hatch.lock`` for reproducible builds.

4. **Keep builds clean**
   Use ``hatch build --clean`` before packaging to remove old artifacts.

5. **Combine Hatch with uv or pixi when needed**
   Hatch can focus on building and versioning,
   while ``uv`` (fast installer) or ``pixi`` (system‑dependency manager) handle environment creation.

---

Differences Compared to Poetry
------------------------------

| Feature | Poetry | Hatch |
|----------|---------|--------|
| Primary focus | Dependency resolution + packaging | Build & environment orchestration |
| Speed | Slower (Python resolver) | Very fast (lightweight backend) |
| Build backend | `poetry-core` | `hatchling` |
| Plugin system | Basic hooks | Extensive plugin interface |
| Virtual environment control | Automatic | Highly configurable |
| Lock file format | `poetry.lock` | `hatch.lock` |
| Ideal for | Application projects | Libraries & automation workflows |

---

Conclusion
----------

Hatch provides a **unified, composable, and high‑performance** approach to Python project management.
It integrates well with modern workflows, supports PEP standards, and keeps configuration simple while remaining powerful.

For developers aiming for advanced, reproducible engineering practices,
**Hatch** represents a top‑tier industry choice — complementing tools like ``uv``, ``pixi``, and ``rye``.
