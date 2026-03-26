---
layout: default
title: GPU Computing Lab Setup
---

# GPU Computing Lab Setup

This page covers everything you need to run the GPU computing notebook on TACC's Vista system.

All of the steps below happen inside JupyterLab after you've connected through [TAP](tap-setup).

---

## Step 1: Open a Terminal

In JupyterLab, go to **File > New > Terminal**. All of the following commands run here.

---

## Step 2: Load Modules

Load these modules:

```bash
module load cuda
module load gcc/13.2.0
module load python3
```

These give you access to NVIDIA's CUDA toolkit, the GCC compiler, and Python 3 with pip.

---

## Step 3: Install CuPy

With the modules loaded, install CuPy (NumPy for the GPU):

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

## Step 4: Get the Notebook

Still in the terminal:

**First time:**
```bash
cd $WORK
git clone https://github.com/ashleyscruse/hpc-python-gpu.git
```

**Already have the repo:**
```bash
cd $WORK/hpc-python-gpu && git pull
```

Navigate to `notebooks/` in the file browser and open `gpu_computing.ipynb`.

---

## Step 5: Verify Your Setup

Run the first two cells in the notebook. You should see:
- All imports successful
- GPU: NVIDIA GH200 120GB
- ~98 GB of GPU memory

If you don't see a GPU, make sure you launched your job on the `gh` queue.

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
