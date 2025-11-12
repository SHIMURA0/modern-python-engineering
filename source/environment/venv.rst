Working with Python Virtual Environments
========================================

Overview
--------

A **virtual environment** (commonly known as *venv*) is an isolated workspace that allows you
to manage Python projects with their own dependencies, packages, and interpreter versions,
without interfering with the global Python installation.

It helps ensure **reproducibility**, **dependency isolation**, and **project-specific configuration** —
which are critical for modern Python development, especially in engineering, scientific, or web projects.

.. note::

   A virtual environment *does not contain its own Python standard library source* —
   it links to the base installation, but isolates all installed packages (``site-packages``).
   This keeps environments lightweight.

---

Creating a Virtual Environment
------------------------------

You can create a virtual environment using Python’s built-in module ``venv`` (available since Python 3.3):

.. code-block:: bash

   # 1. Create a new project directory and enter it
   mkdir my_project
   cd my_project

   # 2. Create a virtual environment inside the project root
   python -m venv .venv

   # 3. Activate the virtual environment
   # On macOS / Linux:
   source .venv/bin/activate

   # On Windows (PowerShell):
   .venv\Scripts\Activate.ps1

   # 4. When finished, deactivate the environment
   deactivate

When activated, your shell prompt usually changes to indicate the active environment, e.g.::

   (.venv) user@macbook:~/project$

.. tip::

   If the activation step fails on **Windows PowerShell**,
   run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` once
   to allow activation scripts to run.

---

Quick Check
-----------

Inside an activated environment, you can test isolation using a *doctest block*:

>>> import sys, site
>>> print(sys.prefix)          # venv directory, not system Python
... # e.g. /Users/name/project/.venv
>>> print(site.getsitepackages())
... # ['/Users/name/project/.venv/lib/python3.13/site-packages']

Your `sys.prefix` path confirms which Python environment you’re currently using.

---

Virtual Environment Folder Structure
------------------------------------

A typical ``.venv`` directory contains:

::

   .venv/
   ├── bin/ or Scripts/        # Executables (Python, pip, etc.)
   ├── include/                # C headers for compiling extensions
   ├── lib/                    # site-packages (installed dependencies)
   └── pyvenv.cfg              # environment configuration file

The virtual environment’s interpreter lives inside this directory, so it’s completely independent
from the system Python.

.. note::

   You can delete or recreate your ``.venv`` folder at any time —
   no essential project data is stored inside it.
   Only installed package binaries will be removed and can always be re‑installed.

---

Detailed Structure Example
--------------------------

On macOS or Linux, ``venv`` typically looks like this:

::

   .venv/
   ├── bin/                   # executables and activation scripts
   ├── include/
   ├── lib/
   │   └── python3.13/
   │       └── site-packages/
   └── pyvenv.cfg

On Windows:

::

   .venv/
   ├── Scripts\
   ├── Include\
   ├── Lib\
   │   └── site‑packages\
   └── pyvenv.cfg

---

Example: Inspecting a Real `.venv`
----------------------------------

.. parsed-literal::

   $ ls -1 .venv/bin
   activate
   python
   pip
   pip3
   python3.13
   deactivate

These executables belong uniquely to the environment.
When you type ``python`` or ``pip`` inside, they point to this folder.

>>> import sys, os
>>> os.path.samefile(sys.executable, ".venv/bin/python")
True

---

Understanding ``pyvenv.cfg``
----------------------------

The ``pyvenv.cfg`` file defines essential metadata about the environment:

::

   home = /Library/Frameworks/Python.framework/Versions/3.13
   include-system-site-packages = false
   version = 3.13.5
   prompt = .venv

* **home** – path to the base Python installation.
* **include-system-site-packages** – whether it inherits global site packages.
* **version** – interpreter version.
* **prompt** – the environment name displayed in the terminal prompt.

.. note::

   If you ever manually modify ``prompt`` here,
   the displayed name when activating the venv will also change.

---

Why Some Folders May Be Missing
-------------------------------

It’s normal if some subfolders don’t appear immediately:

* ``include/`` appears only when compiling native extensions.
* ``lib/`` shows up after you install your first dependency.
* IDEs may hide or auto‑ignore ``.venv/`` for bookkeeping.

.. tip::

   To avoid committing environments to git,
   always include this line in `.gitignore`:
