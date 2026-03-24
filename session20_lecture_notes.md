# Session 20: Python on HPC — Lecture Notes
## Tuesday, March 25, 2026 | 1:00 PM | 75 minutes

---

## Before Class
- Have these open:
  - Terminal (SSH'd into Vista — don't launch idev yet, do it live)
  - Browser tab: [ashleyscruse.github.io/jupyter-on-tacc](https://ashleyscruse.github.io/jupyter-on-tacc/)
  - Browser tab: [github.com/ashleyscruse/hpc-python-gpu](https://github.com/ashleyscruse/hpc-python-gpu)

---

## 0:00–0:05 | Context (5 min)

"So far in this course we've been submitting jobs with Slurm scripts and waiting for results. Today we're going to do something different — we're going to open a Jupyter notebook directly on a supercomputer and run Python interactively, just like you would on your laptop. Except this machine has way more power than your laptop."

Ask: "How many CPU cores does your laptop have?" (Most will say 4-8)
Ask: "How much RAM?" (Most will say 8-16 GB)

"The node we're about to get on has over 100 GB of RAM and 72+ cores. Let's see if that matters."

---

## 0:05–0:10 | Why $WORK? (5 min)

Before we SSH in, quick question.

"TACC gives you three places to store files. Can anyone name them?"

Show the table (write on board or share screen):

| Location | Size | Backed Up? | Purged? |
|----------|------|-----------|---------|
| $HOME | ~10 GB | Yes | No |
| $WORK | ~1 TB | No | No |
| $SCRATCH | Unlimited | No | Yes (10 days) |

"We're going to clone our repo into $WORK. Why not $HOME?" (Answer: 10 GB is too small)
"Why not $SCRATCH?" (Answer: files get deleted after 10 days)

---

## 0:10–0:35 | Live Setup — Everyone Together (25 min)

Screen share your terminal. Walk through every step. Wait for students at each step.

### Step 1: SSH in

```bash
ssh your_username@vista.tacc.utexas.edu
```

"Enter your password — nothing shows on screen, that's normal. Then your MFA token."

Wait for everyone to get in. Help anyone who can't log in.

### Step 2: Clone the repo

```bash
cd $WORK
git clone https://github.com/ashleyscruse/hpc-python-gpu.git
```

"This puts the notebooks on TACC. You only do this once."

### Step 3: Request a compute node

```bash
idev -N 1 -n 1 -t 01:00:00 -A TRA25001
```

When the reservation prompt appears:
"You should see ML-TUTORIAL-TUESDAY as option 1. Type 1 and press Enter."

Wait for everyone to get a node. This is where students will get stuck — give it a few minutes.

"Your prompt changed. You're now on a compute node, not the login node. This is a real piece of the supercomputer dedicated to you."

### Step 4: Load modules and start Jupyter

```bash
module load gcc/13.2.0
module load python3
jupyter notebook --ip=0.0.0.0 --no-browser
```

"Copy the URL with the token. And look at your prompt — write down your node name (the part like c642-081)."

### Step 5: SSH tunnel

"Open a SECOND terminal on your laptop. Keep the first one running."

```bash
ssh -N -L 8888:NODE_NAME:8888 your_username@vista.tacc.utexas.edu
```

"Replace NODE_NAME with your actual node. Enter password and MFA again. It will look like nothing happened — that's correct."

### Step 6: Open in browser

"Go to localhost:8888 in your browser. Paste the token if it asks."

"Navigate to hpc-python-gpu/notebooks/ and open session20_jupyter_on_hpc.ipynb."

Pause here. Make sure everyone has the notebook open. This is the milestone.

"If you're stuck, raise your hand. If your neighbor is stuck, help them."

---

## 0:35–0:55 | Run the Notebook (20 min)

"Now hit play on each cell. I'll walk you through what's happening."

### Part 0: Prove you're on HPC (~2 min)
Run the cell. Read out the hostname and core count.
"That's not your laptop. That's a supercomputer."

### Part 1: Matrix Multiplication (~8 min)
Run the small matrix cell first.
"100x100 — instant. Your laptop can do this."

Run the scaling cell.
"Watch the time column as the matrix gets bigger. At 10,000x10,000, we're doing 2 trillion operations."

Run the plot cell.
"The GFLOPS chart shows the machine gets MORE efficient with bigger problems. Small problems don't give the hardware enough to chew on."

Give them 2 minutes for the YOUR TURN questions.

### Part 2: Memory Matters (~5 min)
Run the 20,000x20,000 cell.
"That's 9.6 GB just for three matrices. Your laptop with 8 GB of RAM would choke. This node doesn't even notice."

Give them 1 minute for the YOUR TURN questions.

### Part 3: Multiprocessing (~5 min)
Run the Monte Carlo cell.
"1 core vs all cores. Look at the speedup."

"Why isn't it perfect linear speedup? There's overhead — splitting the work, collecting results. That's a real tradeoff in parallel computing."

Give them 1 minute for the YOUR TURN question.

---

## 0:55–1:05 | Exit Ticket (10 min)

"Scroll to the bottom. Answer the 4 questions. Submit on Canvas before you leave."

1. What is the hostname of your compute node?
2. How many CPU cores and how much RAM?
3. How long did the 10,000x10,000 matrix multiply take?
4. In one sentence: why would a researcher use HPC instead of their laptop?

---

## 1:05–1:15 | Wrap Up (10 min)

"Today you did it the hard way — SSH, idev, modules, tunnel. You understand what's happening under the hood."

"Thursday, I'll show you the easy way — a web portal where you click a button and get the same thing. Plus we'll use the GPU."

"Before you go:"
- Save your notebook (Ctrl+S)
- Close the browser tab
- In your first terminal: Ctrl+C to stop Jupyter, then `exit` to leave the node
- Close the tunnel terminal

"The guide page has everything we did today if you need to do it again: ashleyscruse.github.io/jupyter-on-tacc"

---

## If Things Go Wrong

**Student can't SSH in:** Check username, MFA. Have them try on their phone hotspot if campus wifi blocks port 22.

**idev takes forever:** The reservation should make this instant. If not, have them try `idev -p gh-dev -N 1 -n 1 -t 01:00:00 -A TRA25001` and pick option 3.

**Tunnel doesn't work:** Most common mistake is typing the node name wrong. Have them check their first terminal for the actual node.

**Jupyter won't open in browser:** Make sure they're going to `localhost:8888`, not the TACC URL. Make sure the tunnel terminal is still running.

**Student falls behind:** Pair them with a neighbor who got it working. Don't let anyone sit stuck for more than 2 minutes.