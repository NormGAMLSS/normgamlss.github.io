# Installation Guide

This guide walks you through setting up the NormGAMLSS package and its dependencies.

## Prerequisites

- **Python**: 3.10 (the only version currently tested and supported)
- **R**: Version 4.1.2 or higher
- **Operating System**: Windows, macOS, or Linux

---

## Installation Method 1: Recommended — Conda Environment

This is the only fully supported installation path. It ensures R, Python, and rpy2 are all compatible and sourced from conda-forge, which prevents the most common class of crashes (version conflicts between system R and Python-bound rpy2).

### Step 1: Download the Package

```bash
git clone https://github.com/NormGAMLSS/NormGAMLSS
cd NormGAMLSS
```

### Step 2: Create the Conda Environment

```bash
conda env create -f environment.yml
conda activate gamlss
```

This installs:
- Python 3.10
- rpy2 3.5.11 from conda-forge (pinned to last compatible version — see note below)
- R from conda-forge, ensuring compatibility with rpy2
- All required Python packages (numpy, pandas, scipy, matplotlib, etc.)

> **Why rpy2 3.5.11?** From rpy2 3.6.x onward, the `pandas2ri.activate()` API was removed and raises an exception rather than a deprecation warning. The package has been updated to not use that API, but 3.5.11 remains the pinned version for stability until full 3.6.x compatibility is confirmed and tested.

### Step 3: Install the Package

```bash
python -m pip install -e . --no-deps
```

The `--no-deps` flag is important — it prevents pip from overwriting the dependency versions that conda already resolved.

### Step 4: Install the R GAMLSS Package

```bash
R -q -e "install.packages('gamlss', repos='https://cloud.r-project.org')"
```

Optional additional R packages:

```bash
R -q -e "install.packages(c('gamlss.dist', 'gamlss.add', 'gamlss.mx'), repos='https://cloud.r-project.org')"
```

### Step 5: (Optional) Jupyter Support

```bash
python -m ipykernel install --user --name gamlss --display-name "Python (gamlss)"
jupyter notebook
```

### Step 6: Verify Installation

```bash
python -c "import normgamlss; print(normgamlss.__file__)"
python -c "import rpy2.robjects as ro; print(ro.r('R.version.string'))"
python -c "from normgamlss import Gamlss; print('✅ Gamlss import OK')"
```

Expected output:

```
/path/to/normgamlss/__init__.py
R version 4.x.x (...)
✅ Gamlss import OK
```

---

## Installation Method 2: Existing Environment (Advanced Users Only)

> ⚠️ **Read this section carefully before proceeding.**
>
> This method is intended for users who already have a working Python environment with a compatible rpy2 version installed. It is **not a general-purpose alternative** to Method 1.
>
> **Tested configuration:** Python 3.10, rpy2 3.5.11, R 4.1.2+. Other combinations are untested and may not work.
>
> Running `pip install -e .` in a shared environment can silently overwrite existing package versions and break other projects. If you are working in an environment used by multiple projects, use Method 1 instead and keep them isolated.
>
> If you encounter any issues with this method, switch to **Method 1 — it is the only fully supported path.**

### Step 1: Download the Package

```bash
git clone https://github.com/NormGAMLSS/NormGAMLSS
cd NormGAMLSS
```

### Step 2: Activate Your Environment

```bash
# Conda environment
conda activate your_env

# Or virtual environment (macOS/Linux)
source your_venv/bin/activate

# Or virtual environment (Windows)
your_venv\Scripts\activate
```

### Step 3: Verify rpy2 Version

Before installing, confirm your rpy2 version is compatible:

```bash
python -c "import rpy2; print(rpy2.__version__)"
```

You should see `3.5.x`. If you have 3.6.x or later, the package may not work correctly — use Method 1 instead.

### Step 4: Install the Package

```bash
pip install -e . --no-deps
```

Using `--no-deps` is strongly recommended to avoid pip silently changing dependency versions in your environment.

### Step 5: Install R Packages

In R or RStudio:

```r
install.packages("gamlss")
install.packages("gamlss.dist")
install.packages("gamlss.add")

# Optional
install.packages(c("gamlss.mx", "gamlss.tr", "gamlss.cens"))

# Verify
library(gamlss)
```

### Step 6: Verify Installation

Create and run `test_installation.py`:

```python
import numpy as np
import pandas as pd
import rpy2.robjects as robjects
from rpy2.robjects.packages import importr

print("R version:", robjects.r('R.version.string')[0])

try:
    gamlss = importr('gamlss')
    print("✅ GAMLSS R package loaded successfully")
except Exception as e:
    print("❌ Error loading GAMLSS:", e)

try:
    from normgamlss import Gamlss
    print("✅ normgamlss package imported successfully")
except Exception as e:
    print("❌ Error importing normgamlss:", e)

result = robjects.r('1 + 1')[0]
print(f"✅ Basic R computation works: 1 + 1 = {result}")
print("🎉 Installation successful!")
```

```bash
python test_installation.py
```

### Step 7: Restart Python Kernel

If using Jupyter notebooks, restart the kernel after installation.

---

## Which Method Should I Use?

| Situation | Recommended Method |
|---|---|
| Fresh installation | ✅ Method 1 (Conda) |
| Existing environment with rpy2 3.5.x confirmed working | Method 2 (with caution) |
| rpy2 3.6 or newer (check with `python -c "import rpy2; print(rpy2.__version__)"`) | ✅ Method 1 (Conda) |
| Jupyter notebooks | ✅ Method 1 (Conda) |
| Shared/multi-project environment | ✅ Method 1 (Conda) |
| Experiencing crashes or segfaults | ✅ Method 1 (Conda) |
| Server installation (e.g. denBi) | ✅ Method 1 (Conda) |

---

## Package Structure

```
NormGAMLSS/
├── normgamlss/
│   ├── __init__.py            # Package initialization
│   ├── gamlss_main.py         # Core GAMLSS class
│   ├── utils.py               # Utility functions
│   ├── model_utils.py         # Model comparison tools
│   ├── plot_style.py          # Shared figure style and plot helpers
│   └── site_transfer_utils.py # Multi-site transfer functions
├── gamlss_python/             # Old package name (alias of normgamlss, shows a warning)
├── docs/
├── environment.yml
└── pyproject.toml
```

---

## Troubleshooting

### `DeprecationWarning: activate and deactivate are deprecated`

This error means you have rpy2 3.6.x or later installed. From that version, `pandas2ri.activate()` raises an exception instead of just printing a warning.

**Solution:** Pin rpy2 to 3.5.11.

```bash
# In your conda environment:
conda install conda-forge::rpy2=3.5.11

# Or with pip (not recommended — use conda for rpy2):
pip install rpy2==3.5.11
```

If using Method 1 with the provided `environment.yml`, this is already handled for you.

### `NotImplementedError: Conversion 'rpy2py' not defined for objects of type 'numpy.ndarray'`

This occurs in `plot_diagnostics()` when rpy2 has already converted R objects to numpy arrays before the manual conversion call is reached. This bug has been fixed in the current version. If you see it:

1. Make sure you are on the latest version of the package (`git pull`)
2. Restart your Python kernel

### rpy2 installation fails

**Best solution in all cases: use Method 1 (conda).** Conda installs a pre-built, compatible rpy2 binary alongside R — no compilation needed.

If you must install via pip:

```bash
# Windows — may need Visual Studio build tools first
pip install --upgrade setuptools wheel
pip install rpy2==3.5.11

# macOS — may need Xcode CLI tools first
xcode-select --install
pip install rpy2==3.5.11

# Linux
sudo apt-get install r-base-dev python3-dev
pip install rpy2==3.5.11
```

### R packages not found

```python
import os
os.environ["R_LIBS_USER"] = "/path/to/R/library"
```

Or set it before launching Python:

```bash
export R_LIBS_USER=/path/to/R/library
```

### Graphics/plotting problems

```bash
# Ubuntu/Debian
sudo apt-get install libcairo2-dev libxt-dev

# macOS
brew install cairo
```

### Crashes or segmentation faults (especially in Jupyter)

This almost always means system R and conda Python are being mixed. Use Method 1, which installs R from conda-forge into the same environment as Python.

### Permission errors

```bash
pip install --user <package_name>
```

---

## Environment Variables

If needed:

```bash
# Linux/macOS (.bashrc or .zshrc)
export R_HOME=/usr/local/lib/R
export R_LIBS_USER=/path/to/R/library

# Windows
set R_HOME=C:\Program Files\R\R-4.x.x
set R_LIBS_USER=C:\path\to\R\library
```

---

## Verification Checklist

- [ ] Python 3.10 environment activated
- [ ] rpy2 3.5.11 installed
- [ ] R 4.1.2+ installed and accessible
- [ ] GAMLSS R package loads without errors
- [ ] `normgamlss` imports successfully
- [ ] Test script runs to completion

---

## Next Steps

1. 📖 [Quick Start Guide](quick_start.md) — a 5-minute end-to-end example
2. 📊 [Data Setup and Preprocessing](data_and_setup_theory.md)
3. 🔧 [Troubleshooting](troubleshooting.md) if you hit issues