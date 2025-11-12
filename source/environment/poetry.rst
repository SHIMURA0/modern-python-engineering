Managing Dependencies and Packaging with Poetry
===============================================

Overview
--------

**Poetry** is a modern Python dependency and project management tool that
simplifies how you declare, install, and build Python packages.
It replaces the traditional combination of ``pip``, ``venv``, and
manual configuration files with a unified workflow based on the
``pyproject.toml`` standard (PEP 518 / PEP 621).

In short:

* Handles **dependency resolution** automatically
* Manages **virtual environments** for each project
* Builds and publishes **distributable packages**
* Uses one central configuration: ``pyproject.toml``

Poetry has become a popular choice for developers who want a predictable,
reproducible, and elegant package management workflow.

---

Installation
------------

Poetry provides an official installation script independent from system pip:

.. code-block:: bash

   curl -sSL https://install.python-poetry.org | python3 -

After installation, ensure it’s on your path:

.. code-block:: bash

   poetry --version

Output example:

::

   Poetry (version 1.8.3)

You can update Poetry at any time with:

.. code-block:: bash

   poetry self update

If you prefer environmental isolation, you can also install Poetry with
``pipx`` (recommended for developers managing multiple environments):

.. code-block:: bash

   pipx install poetry

---

Creating a New Project
----------------------

Poetry can initialize projects automatically.

.. code-block:: bash

   poetry new myproject
   cd myproject

The generated structure:

::

   myproject/
   ├── myproject/
   │   └── __init__.py
   ├── tests/
   │   └── __init__.py
   ├── pyproject.toml
   └── README.rst

The ``pyproject.toml`` file contains all necessary project metadata
(name, version, description, dependencies, etc.).

To convert an existing project:

.. code-block:: bash

   cd existing_project
   poetry init

This launches an interactive process to add dependencies and create the
``pyproject.toml`` file.

---

Installing and Managing Dependencies
------------------------------------

Poetry handles dependencies declaratively:

.. code-block:: bash

   # Add runtime dependency
   poetry add requests

   # Add development dependency
   poetry add --group dev pytest black

   # Remove a dependency
   poetry remove requests

To install all dependencies from the project configuration:

.. code-block:: bash

   poetry install

This will automatically create or reuse a dedicated virtual environment
for the project (stored in ``~/.cache/pypoetry/virtualenvs`` by default).

---

Virtual Environment Management
------------------------------

Poetry integrates virtual environment creation and activation:

.. code-block:: bash

   # Show which virtual environment Poetry manages
   poetry env info

   # Activate it in a new shell
   poetry shell

   # Run a command within the Poetry environment
   poetry run python main.py

To configure Poetry to create virtual environments **inside your project folder**
(rather than globally):

.. code-block:: bash

   poetry config virtualenvs.in-project true

   # New environments will be created as:
   #  myproject/.venv/
---

Dependency Resolution and Locking
---------------------------------

Poetry uses a deterministic solver to install and lock exact dependency versions.
Dependencies are recorded in two files:

* ``pyproject.toml`` – human‑editable manifest of requirements
* ``poetry.lock`` – automatically generated snapshot of exact versions

To refresh dependency resolution:

.. code-block:: bash

   poetry lock        # re‑resolve and update lock file
   poetry update      # update all dependencies to newest compatible versions

Reproducible installs are guaranteed by:

.. code-block:: bash

   poetry install --sync

which ensures the environment exactly matches the lock file.

---

Handling Python Versions
------------------------

Poetry can restrict compatible Python versions for your project:

In ``pyproject.toml``:

.. code-block:: toml

   [tool.poetry.dependencies]
   python = "^3.12"
   requests = "^2.31"

When installing, Poetry will enforce that the interpreter matches.

---

Using Scripts and Tasks
-----------------------

You can define executable scripts directly in ``pyproject.toml`` so they
can be triggered via ``poetry run``.

.. code-block:: toml

   [tool.poetry.scripts]
   start = "myproject.app:main"

Run it:

.. code-block:: bash

   poetry run start

This is useful for packaging entry points or lightweight CLI utilities.

---

Building and Publishing Packages
--------------------------------

To build distributable artifacts (wheel and source archive):

.. code-block:: bash

   poetry build

Output example:

::

   dist/
   ├── myproject-1.0.0.tar.gz
   └── myproject-1.0.0-py3-none-any.whl

To publish directly to `PyPI`:

.. code-block:: bash

   poetry publish --build

For private repositories, specify the source via configuration:

.. code-block:: bash

   poetry config repositories.private https://your.repo/simple/
   poetry publish -r private

---

Project Configuration (pyproject.toml)
--------------------------------------

A typical Poetry-managed ``pyproject.toml``:

.. code-block:: toml

   [tool.poetry]
   name = "myproject"
   version = "0.1.0"
   description = "A simple Poetry example"
   authors = ["Alice <alice@example.com>"]

   [tool.poetry.dependencies]
   python = "^3.13"
   requests = "^2.32"

   [tool.poetry.group.dev.dependencies]
   pytest = "^8.3"

   [build-system]
   requires = ["poetry-core"]
   build-backend = "poetry.core.masonry.api"

---

Advanced Features
-----------------

### Exporting dependencies for pip compatibility

If you need to use Poetry‑managed dependencies in an environment without Poetry:

.. code-block:: bash

   poetry export -f requirements.txt --output requirements.txt --without-hashes

Then install with pip as usual:

.. code-block:: bash

   pip install -r requirements.txt

---

### Managing multiple dependency groups

Divide dependencies by groups to isolate optional components or test requirements.

.. code-block:: bash

   poetry add --group test pytest coverage
   poetry install --with test

Groups can also be combined:

.. code-block:: bash

   poetry install --with dev,test

---

### Interacting with the system environment

To create or activate an environment with an alternative Python version:

.. code-block:: bash

   poetry env use /usr/local/bin/python3.13

---

Best Practices
--------------

1. **Track both `pyproject.toml` and `poetry.lock` in version control.**
   Ensures reproducible installs for all developers and CI systems.

2. **Store virtual environments inside each project (`.venv/`).**
   Keeps environments isolated and portable.

3. **Use dependency groups (dev/test/docs).**
   Simplifies selective installation.

4. **Publish only from clean, built artifacts (`poetry build`).**
   Avoid publishing from editable source trees.

5. **Combine with CI tools.**
   Example GitHub Action snippet:

   .. code-block:: yaml

      - uses: actions/setup-python@v4
      - uses: abatilo/actions-poetry@v3
      - run: poetry install --no-root
      - run: poetry run pytest

---

Troubleshooting
---------------

* **“Package not found”** → check repository sources with ``poetry source list``.
* **“Incompatible Python version”** → ensure the active interpreter matches ``python = "^3.x"``.
* **Resolving takes long** → use ``poetry lock --no-update`` or consider ``uv`` integration for faster solving.
* **Permission errors** → always install packages *inside* Poetry environments, not globally.

---

Integration with Modern Tools
-----------------------------

Poetry can cooperate with new‑generation tools for even better performance:

* **uv** – as a fast installer/resolver backend (`uv sync` reads `pyproject.toml`)
* **hatch** – for lightweight build customization (uses `poetry-core` backend)
* **rye** – can delegate dependency handling to Poetry for packaging standard compliance
* **pixi** – manages external system dependencies while Poetry manages pure Python ones

This modular approach allows Poetry to remain part of a modern,
composable toolchain in professional workflows.

---

Conclusion
----------

``Poetry`` simplifies dependency management, environment isolation, and packaging
into a single consistent workflow built on modern Python standards.
By maintaining ``pyproject.toml`` and ``poetry.lock``, you gain
reproducible environments, automated version resolution, and seamless publishing.

Its predictable behavior, clear configuration, and integration with tools like
``uv`` and ``hatch`` make Poetry a foundation of modern Python engineering practice.