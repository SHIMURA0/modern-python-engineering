Managing Data Science Environments with Conda and Mamba
=======================================================

Overview
--------

**Conda** and **Mamba** are cross-language package and environment managers
commonly used in data science, machine learning, and scientific computing.

They manage not only Python packages but also **system-level libraries** (e.g., ``numpy``’s C dependencies,
``pytorch``, ``ffmpeg``, or ``gcc``), making them indispensable when lightweight tools such as ``pip`` cannot resolve complex binaries.

- **Conda** – the original package manager from Anaconda, Inc.
- **Mamba** – a re‑implementation in **C++**, designed to drastically improve Conda’s performance and parallelism.

You can think of **Mamba** as **Conda done right**: same commands, same ecosystem, up to **50× faster**.

---

Installation
------------

**Option 1 — Install Miniconda (Lightweight)**

.. code-block:: bash

   # Linux / macOS
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   bash Miniconda3-latest-Linux-x86_64.sh
   # Follow prompts, restart your shell

**Option 2 — Install Mambaforge (Recommended)**

Mambaforge is a community lightweight distribution that ships with ``mamba`` as default:

.. code-block:: bash

   # macOS / Linux example
   curl -LO https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh
   bash Mambaforge-$(uname)-$(uname -m).sh

   # Windows (PowerShell)
   Invoke-WebRequest -Uri https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Windows-x86_64.exe -OutFile Mambaforge.exe
   Start-Process .\Mambaforge.exe -Wait

Confirm installation:

.. code-block:: bash

   conda --version
   mamba --version

Both executables are 100% compatible.

---

Creating and Activating Environments
------------------------------------

Create a new isolated environment with Conda or Mamba:

.. code-block:: bash

   conda create -n myenv python=3.12
   # or equivalently
   mamba create -n myenv python=3.12

Activate it:

.. code-block:: bash

   conda activate myenv
   # deactivate
   conda deactivate

List all environments:

.. code-block:: bash

   conda env list

Remove an environment:

.. code-block:: bash

   conda remove -n myenv --all

---

Installing and Managing Packages
--------------------------------

Use ``install`` to add packages from repositories such as ``conda-forge`` or the default Anaconda repo.

.. code-block:: bash

   mamba install numpy pandas matplotlib

By default, Conda will ask for confirmation; Mamba performs dependency resolution **in parallel**
and usually completes installation much faster.

For example:

| Operation | Conda | Mamba |
|------------|--------|--------|
| Install 10 scientific packages | ~60 s | **~6 s** |
| Create new env w/ dependencies | ~1 min | **~7 s** |

To install from a specific channel:

.. code-block:: bash

   mamba install pytorch torchvision torchaudio -c pytorch -c nvidia

or add the **conda-forge** channel for community packages:

.. code-block:: bash

   conda config --add channels conda-forge
   conda config --set channel_priority strict

---

Environment Files (YAML)
------------------------

Conda environments are easy to export or reproduce.

Export the current environment:

.. code-block:: bash

   conda env export --no-builds > environment.yml

Recreate the same environment:

.. code-block:: bash

   conda env create -f environment.yml

Update an existing one:

.. code-block:: bash

   conda env update -f environment.yml --prune

This YAML workflow is a best practice for data‑analysis teams, enabling full reproducibility.

---

Dependency Resolution
---------------------

One of the biggest improvements of ``mamba`` is its **parallel dependency resolver**.
Under the hood, ``mamba`` shares Conda’s environment database format,
so environments created by Conda and Mamba can coexist and interoperate.

If you ever face slow solving with Conda:

.. code-block:: bash

   mamba repoquery whoneeds numpy
   mamba repoquery depends matplotlib

These commands let you quickly inspect dependency trees and conflicts.

---

Mixing Conda and pip
--------------------

Sometimes you may need packages that are only available on PyPI.
You can use pip *inside* a Conda environment safely:

.. code-block:: bash

   conda create -n webapp python=3.12 pip
   conda activate webapp
   pip install fastapi uvicorn

To record both conda and pip dependencies when exporting:

.. code-block:: bash

   conda env export > environment.yml

The file will contain a section similar to:

.. code-block:: yaml

   - pip:
     - fastapi==0.115.0
     - uvicorn[standard]==0.31.0

---

Managing Multiple Channels
--------------------------

Conda repositories are organized into *channels*.
You can prioritize faster, open‑source channels like ``conda-forge``:

.. code-block:: bash

   conda config --show channels
   # Add conda-forge as highest priority
   conda config --prepend channels conda-forge
   conda config --set channel_priority strict

Your default configuration file is stored at ``~/.condarc``.

---

Performant and Reproducible Workflows
-------------------------------------

**Typical workflow using Mamba:**

.. code-block:: bash

   # 1. Create new environment
   mamba create -n analysis python=3.12 jupyterlab pandas seaborn

   # 2. Activate
   conda activate analysis

   # 3. Run interactive session
   jupyter lab

   # 4. Export environment definition
   conda env export --from-history > environment.yml

   # 5. Reproduce on another machine
   mamba env create -f environment.yml

This approach is standard in data-science and machine-learning pipelines.

---

Integration with Jupyter and IDEs
---------------------------------

Register a conda environment with Jupyter:

.. code-block:: bash

   conda install ipykernel
   python -m ipykernel install --user --name analysis --display-name "Python (analysis)"

Then in VS Code or JupyterLab, select *Python (analysis)* as the interpreter.

PyCharm and other IDEs detect conda environments automatically.

---

Using Mamba in CI/CD
--------------------

Because of its speed and deterministic resolver,
**Mamba** is excellent for continuous integration (CI) builds.

Example GitHub Actions workflow:

.. code-block:: yaml

   name: Conda CI
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Set up Mambaforge
           uses: conda-incubator/setup-miniconda@v3
           with:
             activate-environment: test
             miniforge-variant: Mambaforge
             python-version: 3.12
         - name: Install dependencies
           run: mamba install --yes pytest pandas
         - name: Run tests
           run: pytest -v

---

Comparison: Conda vs Mamba vs pip
---------------------------------

| Feature | Conda | Mamba | pip / uv |
|----------|-------|--------|----------|
| Language | Python | C++ (libsolv) | Python / Rust |
| Environment isolation | Yes | Yes | venv‑based |
| Speed | Moderate | **Very fast** | Fast (uv), slower pip |
| Non‑Python deps | ✅ | ✅ | ❌ |
| Binary caching | ✅ | ✅ | Limited |
| Supported config | YAML | YAML | requirements / lock |
| Best for | Data science, ML, research | Same (faster) | App development |

---

Best Practices
--------------

1. **Prefer Mamba over Conda for heavy workloads.**
   Mamba resolves dependencies in parallel using libsolv.

2. **Use `conda-forge` as the primary channel.**
   Provides more up-to-date and open binaries than the default Anaconda repo.

3. **Export environments to YAML.**
   Track your entire computational stack (`python`, `numpy`, `gcc`, etc.) for reproducibility.

4. **Combine with pip only when necessary.**
   Keep pure Conda environments where possible to ensure consistent binary dependencies.

5. **Use Mambaforge or Miniforge distributions.**
   They are open‑source, minimal, and channel‑neutral.

6. **Keep environment size under control.**
   Use per‑project environments instead of one large global base environment.

---

Troubleshooting
----------------

* **Slow solving:** try running with Mamba (`mamba install ...`).
* **Conflicting packages:** use ``mamba repoquery depends`` to inspect dependencies.
* **Cache issues:** clear package cache with ``conda clean -a`` or ``mamba clean --all``.
* **Path problems:** ensure environment binaries are activated (the `conda activate myenv` step).

---

Conclusion
----------

``Conda`` and ``Mamba`` remain the standard for complex, binary-heavy projects —
especially in data science and research — where managing native dependencies is essential.

While lighter tools like ``hatch`` and ``uv`` shine for pure‑Python ecosystems,
``mamba`` offers unparalleled **speed**, **stability**, and **cross‑language interoperability**.

For best results:
- use **Mambaforge** for everyday development,
- and integrate ``mamba env create`` or ``conda env export`` into your CI/CD pipelines.

This combination gives you repeatable, high‑performance, production-ready environments
for modern scientific and engineering workflows.