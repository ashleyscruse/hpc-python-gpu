---
layout: default
title: HPC Python Lab Guide
---

# HPC Python Lab: Setup Guide

Two ways to run Jupyter on TACC's Vista system. Tuesday we do it from the command line. Thursday we use the web portal.

---

# Method 1: Command Line (Tuesday)

This is the "under the hood" way. You SSH into Vista, request a compute node, and launch Jupyter yourself.

## Step 1: SSH into Vista

Open a terminal on your laptop and run:

```bash
ssh your_username@vista.tacc.utexas.edu
```

Enter your TACC password and MFA token when prompted.

> Nothing shows when you type your password. That's normal.

**Windows users:** Use PowerShell or [PuTTY](https://www.putty.org/).

---

## Step 2: Request a Compute Node

You're on the login node. **Don't run code here.** Request a dedicated compute node with `idev`:

```bash
idev -N 1 -n 1 -t 01:00:00 -A TRA25001
```

| Flag | Meaning |
|------|---------|
| `-N 1` | 1 node |
| `-n 1` | 1 task |
| `-t 01:00:00` | 1 hour time limit |
| `-A TRA25001` | Our allocation |

### Reservation Prompt

After running the command, you'll see a list of reservations. It looks something like this:

```
 => Select RESERVATION,  options are:
     No  Res_Name            Account  Partition
      1  ML-TUTORIAL-TUESDAY TRA25001 gh

      2  EXIT  from idev
      3  CONT. without using reservation
 => Please type a NUMBER or hit return (default=3):
```

**Type `1`** to use the class reservation. This puts you on the `gh` queue (Grace-Hopper GPU nodes) and gets you a node quickly.

> If you don't see a reservation, type `3` to continue without one. It will use the default queue.

Wait for it to start. You'll see your prompt change to something like `c642-081[gh]$`. That's your compute node.

---

## Step 3: Load Modules and Start Jupyter

```bash
module load gcc/13.2.0
module load python3
jupyter notebook --ip=0.0.0.0 --no-browser
```

Jupyter will print something like:

```
http://127.0.0.1:8888/tree?token=abc123def456...
```

**Copy the full URL with the token.** You'll need it.

Also note the **node name** from your prompt (e.g., `c642-022`).

---

## Step 4: Open a Second Terminal and Create an SSH Tunnel

**Keep the first terminal running.** Open a **new terminal window** on your laptop and run:

```bash
ssh -N -L 8888:NODE_NAME:8888 your_username@vista.tacc.utexas.edu
```

Replace `NODE_NAME` with your actual node (e.g., `c642-022`).

This command won't show any output. That's normal. It's forwarding the connection.

---

## Step 5: Open Jupyter in Your Browser

Paste the URL from Step 3 into your browser:

```
http://localhost:8888/tree?token=abc123def456...
```

You should see JupyterLab. You're now running Python on a Vista compute node.

---

## Step 6: Clone the Repo and Open the Notebook

In JupyterLab, open a terminal (File > New > Terminal):

```bash
cd $WORK
git clone https://github.com/ashleyscruse/hpc-python-gpu.git
cd hpc-python-gpu
```

Then navigate to `notebooks/` in the file browser and open today's notebook.

> Already cloned? Just `cd $WORK/hpc-python-gpu && git pull`

---

## When You're Done

1. Save your notebook (Ctrl+S)
2. Close the browser tab
3. In your first terminal, press Ctrl+C to stop Jupyter
4. Type `exit` to leave the compute node
5. Close the SSH tunnel terminal

---

# Method 2: TACC Analysis Portal (Thursday)

This is the easy way. Everything happens in your browser.

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
| **Reservation** | Leave blank |
| **VNC Resolution** | Leave blank |

Click **Submit**.

---

## Step 3: Connect

Your job appears under **Current Jobs**. Wait for it to switch from Pending to **Running**, then click **Connect**.

JupyterLab opens in your browser. Done.

---

## Step 4: Clone and Open

Same as Tuesday -- open a terminal in JupyterLab:

```bash
cd $WORK
git clone https://github.com/ashleyscruse/hpc-python-gpu.git
```

Open the notebook from the file browser.

---

# Important Notes

- **Save often.** If your job hits the time limit, Jupyter shuts down.
- **Your files persist.** Anything in `$WORK` stays between sessions.
- **Don't run heavy code on the login node.** Use idev or TAP.
- **1-2 hour limit** on dev queues. Resubmit if you need more.

---

# Troubleshooting

**"I can't log in via SSH"**
- Make sure MFA is set up. Use the Okta app, not SMS.

**"idev is taking forever"**
- The queue might be full. Try a shorter time limit: `-t 00:30:00`

**"The SSH tunnel isn't working"**
- Double check the node name. It changes every time you run idev.
- Make sure you're using the right port (8888).
- Make sure the first terminal is still running Jupyter.

**"I can't find my files in Jupyter"**
- You're probably in `$HOME`. Run `cd $WORK` in the terminal.

**"Module not found"**
- Run `!module load python3` in a notebook cell, or `module load python3` in the terminal.

**Still stuck?** Message me.

---

# Quick Reference

| Task | Command |
|------|---------|
| SSH into Vista | `ssh user@vista.tacc.utexas.edu` |
| Request compute node | `idev -N 1 -n 1 -t 01:00:00 -A TRA25001` (pick reservation if offered) |
| Start Jupyter | `jupyter notebook --ip=0.0.0.0 --no-browser` |
| SSH tunnel (new terminal) | `ssh -N -L 8888:NODE:8888 user@vista.tacc.utexas.edu` |
| Launch via TAP | [tap.tacc.utexas.edu](https://tap.tacc.utexas.edu) |
| Clone repo | `cd $WORK && git clone https://github.com/ashleyscruse/hpc-python-gpu.git` |
| Check your node | `hostname` |