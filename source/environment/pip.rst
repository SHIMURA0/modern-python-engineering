Working with the Python Package Manager — pip
=============================================

Overview
--------

**pip** is Python’s built‑in **package installer** and dependency management tool.
It allows you to install, upgrade, and remove third‑party packages from the
**Python Package Index** (PyPI) or any custom repository.

It is one of the core foundations of the modern Python ecosystem.  
Combined with a virtual environment (``venv``), pip lets you maintain clean,
version‑isolated dependencies for every project.

.. note::

   Since Python 3.4, ``pip`` is included automatically with the standard
   installation of Python. You usually don’t need to install it separately.

---

Checking pip Installation
-------------------------

Use the following command to confirm that ``pip`` is available:

.. code-block:: bash

   python -m pip --version

Example output:

::

   pip 24.2 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)

If ``pip`` cannot be found or reports an error:

.. code-block:: bash

   python -m ensurepip --upgrade

.. tip::
   Always invoke pip via ``python -m pip`` instead of calling ``pip`` directly.  
   This ensures that the correct interpreter and environment are used.

---

Installing and Uninstalling Packages
------------------------------------

### Installing Packages

Install a package from PyPI:

.. code-block:: bash

   pip install package_name

Example:

.. code-block:: bash

   pip install requests

Test installation interactively:

>>> import requests
>>> requests.__version__  # doctest: +ELLIPSIS
'2.32.3'

.. note::
   By default pip installs into the currently active environment.
   If you’re inside a virtual environment, it installs there;
   otherwise it installs globally (which often requires admin privileges).

### Uninstalling Packages

.. code-block:: bash

   pip uninstall requests

This will prompt for confirmation and remove the package files.

---

Listing and Inspecting Installed Packages
-----------------------------------------

List all installed packages:

.. code-block:: bash

   pip list

Show information for one specific package:

.. code-block:: bash

   pip show requests

Example output:

::

   Name: requests
   Version: 2.32.3
   Summary: Python HTTP for Humans.
   Home-page: https://requests.readthedocs.io
   Location: /Users/alex/project/.venv/lib/python3.13/site-packages
   Requires: charset-normalizer, urllib3, certifi, idna

.. hint::

   Use ``pip list --outdated`` to find packages with newer versions available.

---

Upgrading and Version Control
-----------------------------

### Upgrade a Package

.. code-block:: bash

   pip install --upgrade package_name

Example:

.. code-block:: bash

   pip install --upgrade numpy

### Install a Specific Version

.. code-block:: bash

   pip install "Django==5.0.4"

### Require Minimum Version

.. code-block:: bash

   pip install "requests>=2.31"

.. important::
   Always pin and record versions for production deployments to ensure
   deterministic, reproducible builds across environments.

---

Freezing and Exporting Dependencies
-----------------------------------

Capturing your current environment into a requirements file is a best‑practice
for sharing or redeploying your setup.

.. code-block:: bash

   pip freeze > requirements.txt

Later, recreate the same environment:

.. code-block:: bash

   pip install -r requirements.txt

.. seealso::
   For complex dependency resolution and lockfiles, consider tools like
   :doc:`poetry <poetry>` or :doc:`hatch <hatch>`.

---

Configuring Mirrors (Index URLs)
--------------------------------

In regions with slow PyPI access, you can use alternative mirrors.

### Temporary Mirror

.. code-block:: bash

   pip install numpy -i https://pypi.tuna.tsinghua.edu.cn/simple

### Persistent Mirror

Create or edit:

* **macOS / Linux** → ``~/.config/pip/pip.conf``  
* **Windows** → ``%APPDATA%\pip\pip.ini``

Content:

::

   [global]
   index-url = https://pypi.tuna.tsinghua.edu.cn/simple

.. tip::
   You can verify the current configuration with:
   ``pip config list``

---

Upgrading pip Itself
--------------------

Keep pip up to date to avoid dependency resolver issues:

.. code-block:: bash

   python -m pip install --upgrade pip

---

Common Issues and Fixes
-----------------------

### 1. SSL Errors or Connection Timeouts

.. warning::

   These are often caused by firewalls, proxies, or CA‑certificate issues.  
   Try switching to a trusted mirror such as Tsinghua or USTC, or use:

   .. code-block:: bash

      pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org requests

### 2. “Module Not Found” After Installation

You likely installed using a different interpreter or environment.

.. tip::
   Run ``which python`` (Unix/macOS) or ``where python`` (Windows)
   to check your current interpreter path.
   It should match your project’s virtual environment.

   When in doubt, always install with:

   .. code-block:: bash

      python -m pip install <package>

### 3. Permission Denied Errors

.. error::
   ``PermissionError: [Errno 13] Permission denied``  
   indicates you’re trying to install packages globally without admin rights.

   **Fixes:**
   * Activate a virtual environment and retry.  
   * or install for the current user only:  

     .. code-block:: bash

        pip install --user package_name

---

Best Practices
--------------

1. **Use virtual environments.**  
   Avoid polluting your global Python with project‑specific dependencies.

2. **Track dependencies explicitly.**  
   Keep `requirements.txt` under version control.

3. **Prefer deterministic builds.**  
   Use pinned versions (`package==x.y.z`) for deployment.

4. **Upgrade dependencies carefully.**  
   Always test after running ``pip install --upgrade``.

5. **Combine pip with modern tools.**  
   - ``pip-tools`` for dependency locking  
   - ``hatch`` / ``poetry`` for full project management

.. important::
   Pip remains the backbone of the Python ecosystem even when you
   adopt modern package managers — under the hood, almost all of
   them still rely on pip’s installation engine.

---

Doctest Example — Checking Installation Path
--------------------------------------------

>>> import sys
>>> sys.prefix  # doctest: +ELLIPSIS
'/Users/alex/project/.venv'
>>> import requests
>>> requests.get("https://httpbin.org/get").status_code
200

This doctest confirms that packages are installed in your active environment
and working as expected.

---

Summary
-------

``pip`` is the universal package installation utility that powers nearly all
Python dependency management workflows.  
Whether you use it directly or through higher‑level tools,
understanding pip’s fundamentals ensures you can debug, troubleshoot,
and manage dependencies confidently.

For larger, more automated dependency handling, explore tools like
:doc:`poetry <poetry>`, :doc:`hatch <hatch>`,
and :doc:`uv <uv>`. They all build upon the foundation
that ``pip`` provides.