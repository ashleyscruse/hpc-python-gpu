---
layout: default
title: GPU Computing Lab Setup
---

# GPU Computing Lab Setup

This page covers everything you need to run the GPU computing notebook on TACC's Vista system.

---

## Required Modules

Open a terminal in JupyterLab (File > New > Terminal) and load these modules:

```bash
module load cuda
module load gcc/13.2.0
module load python3
```

These give you access to NVIDIA's CUDA toolkit, the GCC compiler, and Python 3 with pip.

---

## Required Python Packages

After loading the modules, install CuPy (NumPy for the GPU):

```bash
pip install --user cupy-cuda12x
```

You only need to do this once. The package persists across sessions.

The notebook also uses these packages, which are already available on Vista:

| Package | Purpose |
|---------|---------|
| `numpy` | CPU array operations |
| `matplotlib` | Plotting |
| `scipy` | CPU image processing (for comparison) |
| `cupy` | GPU array operations (installed above) |
| `cupyx.scipy` | GPU image processing |

---

## Verify Your Setup

After loading modules and installing CuPy, restart your Jupyter kernel, then run this in a notebook cell:

```python
import numpy as np
import cupy as cp
import matplotlib
from scipy.ndimage import uniform_filter

print("All imports successful!")

# Check GPU
gpu_name = !nvidia-smi --query-gpu=name --format=csv,noheader
print(f"GPU: {gpu_name[0]}")

device = cp.cuda.Device(0)
mem_free, mem_total = device.mem_info
print(f"GPU memory: {mem_total / 1e9:.0f} GB total, {mem_free / 1e9:.0f} GB free")
```

You should see:
- All imports successful
- GPU: NVIDIA GH200 120GB
- ~98 GB of GPU memory

If you don't see a GPU, make sure you launched your job on the `gh` queue.

---

## Get the Notebook

Open a terminal in JupyterLab:

**First time:**
```bash
cd $WORK
git clone https://github.com/ashleyscruse/hpc-python-gpu.git
```

**Already have the repo:**
```bash
cd $WORK/hpc-python-gpu && git pull
```

Then navigate to `notebooks/` and open `gpu_computing.ipynb`.

---

## Troubleshooting

**`ModuleNotFoundError: No module named 'cupy'`**
- Make sure you ran `module load cuda` and `module load python3` before installing.
- Run `pip install --user cupy-cuda12x` again.
- Restart the Jupyter kernel.

**`No CUDA device found` or GPU memory shows 0**
- You're on a CPU-only node. Resubmit your job through TAP with Queue set to `gh`.

**`OutOfMemoryError` on the GPU**
- Run this in a cell to free GPU memory:
```python
import cupy as cp
cp._default_memory_pool.free_all_blocks()
```
- Or restart the kernel.

**Plots don't display**
- Run `%matplotlib inline` in a cell before any plot cells.

**`module: command not found`**
- You're running the command in a notebook cell instead of a terminal. Open a terminal (File > New > Terminal) and run it there.

**Still stuck?** Email me.
