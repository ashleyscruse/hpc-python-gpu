---
layout: default
title: TAP Setup Guide
---

# TACC Analysis Portal (TAP): Setup Guide

This guide walks you through launching Jupyter on TACC's Vista system using the web portal. No SSH, no tunnels, just your browser.

---

## Step 1: Go to TAP

Navigate to [tap.tacc.utexas.edu](https://tap.tacc.utexas.edu) and log in with your TACC username, password, and MFA token.

---

## Step 2: Submit a Jupyter Job

Fill in the form on the left side:

| Field | What to Enter |
|-------|--------------|
| **System** | `Vista` |
| **Application** | `Jupyter notebook` |
| **Project** | `TRA25001` |
| **Queue** | `gh` |
| **Nodes** | `1` |
| **Tasks** | `1` |
| **Job Name** | `hpc-python-lab` |
| **Time Limit** | `02:00:00` |
| **Reservation** | `Morehouse` |
| **VNC Resolution** | Leave blank |

Click **Submit**.

---

## Step 3: Connect

Your job appears under **Current Jobs**. Wait for it to switch from Pending to **Running**, then click **Connect**.

JupyterLab opens in your browser. That's it. No SSH tunnel. No second terminal. No copying tokens. Just click.

---

## Step 4: Get the Code

Open a terminal in JupyterLab (File > New > Terminal).

**If you already have the repo from Tuesday:**

```bash
cd $WORK/hpc-python-gpu && git pull
```

**If you're starting fresh:**

```bash
cd $WORK
git clone https://github.com/morehouse-supercomputing/hpc-python-gpu.git
```

Navigate to `notebooks/` in the file browser and open `gpu_computing.ipynb`.

---

## When You're Done

1. Save your notebook (Ctrl+S)
2. Close the browser tab
3. TAP will clean up the job when the time limit runs out

---

## Reservation Details

Our class has a reserved block of nodes on Vista today:

| Field | Value |
|-------|-------|
| Reservation Name | Morehouse |
| Partition | gh |
| Account | TRA25001 |
| Start Time | 11:45 AM CT |
| End Time | 1:45 PM CT |
| Duration | 2 hours |
| Nodes | 20 |
| Cores | 1,440 |

This means your job should start immediately instead of waiting in the queue.

---

## Troubleshooting

**"I can't log in"**
- Make sure MFA is set up. Use the Okta app, not SMS.

**Job stuck in Pending**
- The queue may be full. Wait a few minutes. If it doesn't start, try reducing the time limit to `01:00:00`.

**"Module not found" when running the notebook**
- Open a terminal in JupyterLab and run:
```bash
module load cuda
module load python3
pip install --user cupy-cuda12x
```
Then restart the kernel.

**"I can't find my files in Jupyter"**
- You're probably in `$HOME`. Run `cd $WORK` in the terminal.

**Plots don't show**
- Run `%matplotlib inline` in a cell before the plot cell.

**Still stuck?** Email me.
