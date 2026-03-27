---
layout: default
title: GPU Computing Lab Setup
---

# GPU Computing Lab Setup

This page covers everything you need to run the GPU computing notebook on TACC's Vista system.

All of the steps below happen inside JupyterLab after you've connected through [TAP](tap-setup).

---

## Step 1: Get the Notebook

Once JupyterLab opens, go to **File > New > Terminal** and run:

**Already have the repo from Tuesday:**
```bash
cd $WORK/hpc-python-gpu && git pull
```

**First time:**
```bash
cd $WORK
git clone https://github.com/morehouse-supercomputing/hpc-python-gpu.git
```

Then close the terminal. In the file browser on the left, navigate to `notebooks/` and open `gpu_computing.ipynb`.

---

## Step 2: Verify Your Setup

Run the first two cells in the notebook. You should see:
- All imports successful
- GPU: NVIDIA GH200 120GB
- ~98 GB of GPU memory

If you don't see a GPU, make sure you launched your job on the `gh` queue.

The notebook uses these packages, all of which are already available on Vista:

| Package | Purpose |
|---------|---------|
| `torch` | GPU tensor operations and CUDA support |
| `numpy` | CPU array operations |
| `matplotlib` | Plotting |
| `scipy` | CPU image processing (for comparison) |

No modules or pip installs are needed. PyTorch bundles its own CUDA libraries.

---

## Troubleshooting

**`ModuleNotFoundError: No module named 'torch'`**
- This shouldn't happen on Vista. Restart the Jupyter kernel and try again.
- If it persists, open a terminal and run `python3 -c "import torch; print(torch.__version__)"` to confirm PyTorch is available.

**`No CUDA device found` or `torch.cuda.is_available()` returns `False`**
- You're on a CPU-only node. Resubmit your job through TAP with Queue set to `gh`.

**`OutOfMemoryError` on the GPU**
- Run this in a cell to free GPU memory:
```python
import torch
torch.cuda.empty_cache()
```
- Or restart the kernel.

**Plots don't display**
- Run `%matplotlib inline` in a cell before any plot cells.

**Still stuck?** Email me.
