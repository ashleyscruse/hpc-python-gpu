# Session 20: Python on HPC — Lecture Notes

## Tuesday, March 25, 2026 | 1:00 PM | 75 minutes

---

## Before Class

- Have these open:
  - Terminal (SSH'd into Vista — don't launch idev yet, do it live)
  - Browser tab: [ashleyscruse.github.io/jupyter-on-tacc](https://ashleyscruse.github.io/jupyter-on-tacc/)
  - Browser tab: [github.com/morehouse-supercomputing/hpc-python-gpu](https://github.com/morehouse-supercomputing/hpc-python-gpu)

---

## 0:00–0:05 | Context (5 min)

"So far in this course we've been submitting jobs with Slurm scripts and waiting for results. Today we're going to do something different — we're going to open a Jupyter notebook directly on a supercomputer and run Python interactively, just like you would on your laptop. Except this machine has way more power than your laptop."

Ask: "How many CPU cores does your laptop have?" (Most will say 4-8)
Ask: "How much RAM?" (Most will say 8-16 GB)

"The node we're about to get on has 72 cores, 223 GB of RAM, and an NVIDIA GH200 GPU with 98 GB of GPU memory. Let's see if that matters."

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
git clone https://github.com/morehouse-supercomputing/hpc-python-gpu.git
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

"Navigate to hpc-python-gpu/notebooks/ and open jupyter_on_hpc.ipynb."

Pause here. Make sure everyone has the notebook open. This is the milestone.

"If you're stuck, raise your hand. If your neighbor is stuck, help them."

---

## 0:35–1:00 | Run the Notebook (25 min)

"Now hit play on each cell. I'll walk you through what's happening."

### Part 0: Prove you're on HPC (~3 min)

Run the cell. Talk through what they see.

Expected output:
```
Hostname:  c642-021.vista.tacc.utexas.edu
CPU cores: 72
Platform:  aarch64
RAM:       223 GB
GPU:       NVIDIA GH200 120GB, 97871 MiB
```

"That's not your laptop. That's a supercomputer. 72 cores — your laptop has maybe 4 or 8. 223 GB of RAM — your laptop has 8 or 16. And there's a GPU with almost 100 GB of its own memory."

"Notice the platform says `aarch64` — that's ARM. Same architecture as your phone. This is not an Intel chip. Vista uses NVIDIA Grace CPUs which are ARM-based."

### Part 1: Matrix Multiplication (~10 min)

**Before the code, teach GFLOPS:**

"Before we run anything, I want to introduce a term: GFLOPS. G-F-L-O-P-S."

"A floating point operation is one math operation on a decimal number — an add, a multiply, a divide. Every time the computer does 2.5 + 3.7, that's one floating point operation."

"FLOPS means Floating Point Operations Per Second — how many of those the computer can do every second."

"GFLOPS is Giga-FLOPS. Giga means billion. So 37 GFLOPS means the machine is doing 37 billion math operations every single second."

"That's how we measure how fast a computer does math. Your laptop might do 10-50 GFLOPS. A high-end GPU can do thousands. We're going to measure how many GFLOPS this node can do."

**Why matrix multiplication?**

"Before we run this, you might be wondering — why matrix multiplication? Why not something else?"

"Matrix multiplication is THE core operation in modern computing. It's not just a math exercise:"
- "Every layer of a neural network is a matrix multiply. When ChatGPT generates a word, it's doing thousands of matrix multiplies. Training an AI model means doing millions of them."
- "Physics simulations — climate models, fluid dynamics, structural engineering — they all solve systems of equations, which is matrix math."
- "Computer graphics — every frame of a video game or Pixar movie transforms 3D objects using matrix multiplication."
- "Data science — linear regression, recommendation systems, dimensionality reduction — all matrix math under the hood."

"So when we benchmark a supercomputer, we use matrix multiplication because it's the single most important operation in scientific computing and AI. If a machine is fast at this, it's fast at the things researchers actually need."

**Explain the code (cell 5, the small multiply):**

"Let's read this code line by line."
- `np.random.randn(n, n)` — creates an n-by-n matrix filled with random numbers. `np` is NumPy, the standard library for math in Python.
- `A @ B` — this is matrix multiplication. The `@` operator multiplies two matrices together.
- `time.time()` — records the current time. We call it before and after to measure how long the multiply takes.
- `2 * n**3` — that's how many arithmetic operations a matrix multiply does. For a 100x100 matrix, that's 2 million operations.

Run the small cell.

Expected output:
```
100x100 matrix multiply: 0.000545 seconds
Operations: 2,000,000
Your laptop handles this fine.
```

"Half a millisecond. 2 million operations. No sweat."

**Now run the scaling cell (cell 6):**

"Same code, but we go from 100 all the way up to 10,000. Watch what happens to the time."

Expected output:
```
    Size   Time (sec)         Operations     GFLOPS
----------------------------------------------------
     100       0.0001          2,000,000       27.2
     500       0.0071        250,000,000       35.0
    1000       0.0559      2,000,000,000       35.8
    2000       0.4395     16,000,000,000       36.4
    5000       6.7202    250,000,000,000       37.2
   10000      53.5020  2,000,000,000,000       37.4
```

Walk through the key numbers:
- "100x100: instant. 500x500: still fast. 1000x1000: half a second."
- "5000x5000: nearly 7 seconds. Now we're doing 250 billion operations."
- "10,000x10,000: almost a minute. 2 TRILLION operations. That's 2,000,000,000,000."
- "Look at the GFLOPS column. That's billions of operations per second. It stays around 35-37. The machine is doing the same amount of work per second — the problem just gets bigger."

**Run the plot cell.** Two charts appear. Walk through both:

**Left chart (Time vs Matrix Size):**
"This is the time it takes as we make the matrix bigger. Notice it's not a straight line — it curves up fast. That's because matrix multiply scales with the cube. Going from 5,000 to 10,000 doubles the size but takes 8x longer. Small matrices are basically instant. At 10,000 it takes almost a minute."

**Right chart (Compute Throughput / GFLOPS):**
"This is how hard the machine is working — billions of math operations per second. GFLOPS stands for Giga Floating Point Operations Per Second. It's how we measure how fast a computer does math."

"At 100x100 it's only doing about 23 GFLOPS because the problem is too small — most of the time is spent on overhead, not actual math. By 1000x1000 it hits about 35 GFLOPS and stays flat. That means the machine is maxed out. Bigger problems don't make it faster — they just give it more to do."

"The takeaway: small problems waste the hardware. You need big problems to keep a supercomputer busy. That's why HPC matters — your problems need to be big enough to justify the machine."

Give them 2 minutes for the YOUR TURN questions.

Expected answers:
1. Around 2000x2000 or 5000x5000 (depends on exact timing)
2. Small matrices have overhead (memory allocation, function calls) that dominates. Larger matrices let the CPU spend more time on actual math vs overhead.

### Part 2: Memory Matters (~5 min)

**Before running, explain the math:**

"A 20,000x20,000 matrix has 400 million numbers. Each number is 8 bytes (64-bit float). That's 3.2 GB for ONE matrix. We need three — A, B, and the result C. That's 9.6 GB."

"How much RAM does your laptop have? 8 GB? 16 GB? At 9.6 GB just for the matrices, plus your operating system and other apps, your laptop would be swapping to disk or crashing."

Run the cell. **This takes about 7 minutes.** Tell students: "This is going to take a while. 16 trillion operations. Let's talk while it runs."

While waiting, discussion points:
- "Who here has had their laptop freeze because they opened too many Chrome tabs? That's a memory problem. Same idea here — just bigger."
- "This node has 223 GB of RAM. 9.6 GB is nothing to it. We could do 40,000x40,000 and still have room."
- "This is one of the main reasons researchers use HPC — the problem literally doesn't fit on their laptop."

Expected output:
```
Matrix size: 20,000 x 20,000
Memory per matrix: 3.2 GB
Total memory needed (A + B + C): 9.6 GB
This node has 223 GB of RAM. No problem.
Your laptop probably has 8-16 GB. This would be tight.

Running...
Time: 430.4 seconds
Operations: 16.0 trillion
Throughput: 37 GFLOPS
```

"430 seconds — over 7 minutes. 16 trillion operations. And the throughput is the same 37 GFLOPS. The hardware is working just as hard, the problem is just 8x bigger than before."

Give them 1 minute for YOUR TURN questions.

Expected answers:
1. 9.6 GB total. A laptop with 8 GB couldn't do it at all. A laptop with 16 GB could technically fit it but would be very slow with the OS and other apps competing for memory.
2. The system either throws a MemoryError (Python crashes) or the OS starts swapping to disk (the hard drive pretends to be RAM), which makes everything extremely slow.

### Part 3: Multiprocessing (~5 min)

**Explain the code:**

"This is a Monte Carlo simulation. We throw 100 million random darts at a square and count how many land inside a circle. The ratio tells us pi."

- `np.random.default_rng()` — creates a random number generator
- `x**2 + y**2 <= 1.0` — checks if the point is inside a unit circle
- `Pool(n_cores)` — creates a pool of worker processes, one per core
- `pool.map(function, list)` — splits the list across all workers and runs the function on each chunk in parallel

"First we run it on 1 core. Then we split the 100 million samples across all 72 cores — each core handles about 1.4 million."

Run the cell.

Expected output:
```
Monte Carlo pi estimation: 100,000,000 samples
Comparing 1 core vs 72 cores

1 core:  pi=3.141812  time=1.14s
72 cores: pi=3.141570  time=0.12s

Speedup: 9.2x
Actual pi: 3.141593
```

"Look at that. 1 core took 1.14 seconds. 72 cores took 0.12 seconds. That's a 9.2x speedup."

"But wait — we have 72 cores. Why isn't it 72x faster? Because there's overhead. Splitting the work into 72 pieces, sending each piece to a core, collecting the results, combining them — that all takes time. For a problem that only takes 1 second on 1 core, the overhead is a big chunk of the total."

"This is a real tradeoff in parallel computing. There's a concept called Amdahl's Law that says: the speedup you get from adding more cores is limited by the parts of your code that can't be parallelized. The splitting and collecting is serial — only one core can do it."

"If we had a problem that took 10 minutes on 1 core instead of 1 second, the overhead would be a tiny fraction and we'd get much closer to 72x. Bigger problems parallelize better — same lesson as the matrix multiply."

"Also notice — both estimates are close to the real pi (3.141593). More samples = more accuracy. We just ran 100 million random experiments in a tenth of a second."

Give them 1 minute for the YOUR TURN question.

Expected answer:
- 9.2x speedup. It's not 72x because the problem is small (only 1 second on 1 core), so the overhead of creating 72 processes, sending data to each, and collecting results is a significant fraction of the total time. With a bigger problem (minutes instead of seconds), the speedup would be much closer to 72x.

---

## 1:00–1:10 | Exit Ticket (10 min)

"Scroll to the bottom. Answer the 4 questions. Submit on Canvas before you leave."

1. What is the hostname of your compute node?
2. How many CPU cores and how much RAM?
3. How long did the 10,000x10,000 matrix multiply take?
4. In one sentence: why would a researcher use HPC instead of their laptop?

**Expected answers (for grading):**
1. Something like `c642-021.vista.tacc.utexas.edu` — any Vista hostname is correct
2. 72 CPU cores, 223 GB RAM (accept approximate numbers)
3. Around 53 seconds (will vary slightly between nodes)
4. Any answer that captures one of: problems too big for a laptop (memory), problems that take too long on a laptop (compute), or ability to use many cores / GPU. Examples: "HPC lets you work on problems that are too large or too slow for a personal computer." "A researcher would use HPC when their data doesn't fit in their laptop's memory or when they need more cores to finish in a reasonable time."

---

## 1:10–1:15 | Wrap Up (5 min)

"Today you did it the hard way — SSH, idev, modules, tunnel. You understand what's happening under the hood."

"Thursday, I'll show you the easy way — a web portal where you click a button and get the same thing. Plus we'll use the GPU — that GH200 we saw in Part 0."

"Before you go:"
- Save your notebook (Ctrl+S)
- Close the browser tab
- In your first terminal: Ctrl+C to stop Jupyter, then `exit` to leave the node
- Close the tunnel terminal

"The guide page has everything we did today if you need to do it again: ashleyscruse.github.io/jupyter-on-tacc"

---

## If Things Go Wrong

**Student can't SSH in:** Check username, MFA. Have them try on their phone hotspot if campus wifi blocks port 22.

**idev / reservation fails:** Have them try `idev -p gh-dev -N 1 -n 1 -t 01:00:00 -A TRA25001` and pick option 3 (no reservation).

**Tunnel doesn't work:** Most common mistake is typing the node name wrong. Have them check their first terminal for the actual node.

**"Name or service not known":** They typed NODE_NAME literally instead of their actual node. Show them where to find it in their prompt.

**Jupyter won't open in browser:** Make sure they're going to `localhost:8888`, not the TACC URL. Make sure the tunnel terminal is still running.

**Plot doesn't show:** Have them run `%matplotlib inline` in a cell before the plot cell.

**20k matrix takes too long for class:** Skip it. Tell them the numbers and move on. The important part is 10k.

**Student falls behind:** Pair them with a neighbor who got it working. Don't let anyone sit stuck for more than 2 minutes.