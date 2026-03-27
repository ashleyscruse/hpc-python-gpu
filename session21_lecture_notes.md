# Session 21: GPU Computing — Lecture Notes

## Thursday, March 26, 2026 | 1:00 PM | 75 minutes

---

## Before Class

- Have these open:
  - Browser tab: [tap.tacc.utexas.edu](https://tap.tacc.utexas.edu) (logged in)
  - Browser tab: [morehouse-supercomputing.github.io/hpc-python-gpu](https://morehouse-supercomputing.github.io/hpc-python-gpu/) (Method 2 section)
  - Browser tab: [github.com/morehouse-supercomputing/hpc-python-gpu](https://github.com/morehouse-supercomputing/hpc-python-gpu)
- PyTorch is pre-installed on Vista with CUDA support — no module loading or pip install needed.

---

## 0:00–0:05 | Recap (5 min)

"Tuesday you did it the hard way: SSH, idev, modules, tunnel. You saw the CPU doing 37 GFLOPS on matrix multiplication, and the 20,000x20,000 multiply took over 7 minutes."

"Today: the easy way to launch Jupyter, and the GPU. Let's see if we can do better than 7 minutes."

---

## 0:05–0:15 | TAP Setup — Everyone Together (10 min)

Screen share the TACC Analysis Portal (TAP).

"Go to tap.tacc.utexas.edu. Log in with your TACC credentials."

Walk through the form. Wait for students at each field:

| Field | Value |
|-------|-------|
| System | Vista |
| Application | Jupyter notebook |
| Project | TRA25001 |
| Queue | gh |
| Nodes | 1 |
| Tasks | 1 |
| Job Name | hpc-python-lab |
| Time Limit | 02:00:00 |
| Reservation | Morehouse |

**TACC Reservation Details:**

| Field | Value |
|-------|-------|
| Reservation Name | Morehouse |
| Partition | gh |
| Account | TRA25001 |
| Date | March 26, 2026 |
| Start Time | 11:45 AM CT |
| End Time | 1:45 PM CT |
| Duration | 2 hours |
| Nodes | 20 |
| Cores | 1,440 |
| State | INACTIVE (activates at start time) |

**Reserved Nodes:** `c608-121, c610-121, c610-122, c610-131, c613-072, c620-072, c620-081, c636-122, c636-131, c637-091, c637-092, c637-131, c637-132, c639-111, c639-112, c639-121, c640-151, c640-152, c641-001, c641-002`

"Click Submit. Now we wait for it to go from Pending to Running. When it says Running, click Connect."

"That's it. No SSH tunnel. No second terminal. No copying tokens. Just click."

Wait for everyone to have Jupyter open. Help anyone stuck.

"If you already have the repo from Tuesday, open a terminal in Jupyter and run:"
```bash
cd $WORK/hpc-python-gpu && git pull
```

"If you're starting fresh:"
```bash
cd $WORK
git clone https://github.com/morehouse-supercomputing/hpc-python-gpu.git
```

"Navigate to notebooks/ and open gpu_computing.ipynb."

Pause. Make sure everyone has it open.

---

## 0:15–0:20 | Part 0: Prove You Have a GPU (~5 min)

Run the first cell. Same system info as Tuesday plus the GPU.

"Same 72 cores, same 223 GB RAM. But now look at the GPU line: NVIDIA GH200 120GB with 102 GB of GPU memory. And we're running CUDA 12.6."

"That GPU has its own processor — thousands of small cores — and its own memory, completely separate from the CPU."

Draw on board or say:
- "CPU: 72 cores. Think of 72 really smart people who can each solve complex problems."
- "GPU: thousands of cores. Think of thousands of people who can each do simple math. Individually they're slower, but together they're overwhelming."

"When your problem is doing the same thing to millions of numbers, the GPU wins."

---

## 0:20–0:25 | Part 1: Hello GPU with PyTorch (~5 min)

Run the PyTorch device info cell.

"PyTorch is the most widely used framework for AI and machine learning. It lets you run the same math on either CPU or GPU — you just tell it where to put the data using `device='cuda'`."

"This is the same tool that researchers use to train large language models, computer vision systems, and scientific simulations. Learning PyTorch here means you're learning the real tool, not a toy."

Run the tensor creation cell.

"Notice the devices: `cpu` vs `cuda:0`. The CPU tensor lives in RAM. The GPU tensor lives in GPU memory. They look the same when you print them, but they're in completely different places."

Run the math cell.

"Same code, same results. The difference is invisible to you but matters a lot to the hardware."

---

## 0:25–0:45 | Part 2: CPU vs GPU Matrix Multiply (~20 min)

Run the warm-up cell.

"The very first GPU operation is always slow because it's loading CUDA libraries. We get that out of the way first."

Explain the benchmark functions:

"Two things to notice in the GPU function:"
1. "`torch.randn(n, n, device='cuda')` creates the data directly on the GPU. We don't create it on the CPU and transfer it."
2. "`torch.cuda.synchronize()` — the GPU works asynchronously. It starts work and immediately gives control back to Python. We have to explicitly wait for it to finish before stopping our timer. Otherwise we'd be measuring how fast Python can *ask* the GPU to do work, not how fast the GPU actually *does* it."

Run the comparison cell. **This is the big moment.**

Actual output from the run:
```
    Size   CPU (sec)    GPU (sec)    Speedup   GPU TFLOPS
----------------------------------------------------------
     500       0.0119       0.0389       0.3x        0.01
    1000       0.0213       0.0002      98.3x        9.22
    2000       0.1649       0.0011     145.7x       14.14
    5000       2.5255       0.0058     436.6x       43.22
   10000      20.0552       0.0418     480.0x       47.87
```

Walk through the key numbers:
- "500x500: the GPU is actually *slower* here. 0.3x means the CPU won. The problem is too small for the GPU to even bother with. All that overhead of launching GPU work isn't worth it for a tiny matrix."
- "1000x1000: now the GPU is 98x faster. Just doubling the size flipped everything."
- "10,000x10,000: the CPU took 20 seconds. The GPU did it in 0.04 seconds. That's 480 times faster."
- "Look at the TFLOPS column. TFLOPS means TRILLION operations per second. Tuesday the CPU was doing 37 GFLOPS, that's 0.037 TFLOPS. The GPU hit nearly 48 TFLOPS. That's over 1,000 times more math per second."

Run the plot cell.

"Left chart: at small sizes they're both tiny bars. At 10,000 the CPU bar is enormous and the GPU bar is barely visible. That's the whole point."

"Right chart: the speedup grows with size. The GPU gets relatively better as the problem gets bigger. Same lesson as Tuesday: big problems use the hardware better."

Give them 2 minutes for YOUR TURN.

Expected answers:
1. Around 1000x1000. At 500x500 the GPU was actually slower (0.3x). At 1000x1000 it jumps to 98x faster.
2. Small problems don't give the GPU's thousands of cores enough work. The overhead of launching the GPU computation dominates. With large problems, the massive parallelism pays off.

---

## 0:45–0:50 | Part 3: Data Transfer Cost (~5 min)

Run the transfer benchmark.

"This shows how fast data moves from CPU to GPU memory. We use `.to('cuda')` to send data to the GPU and `.cpu()` to bring it back."

"On the GH200 it's very fast because the CPU and GPU are connected by NVLink-C2C — a direct high-speed link."

"On a regular workstation with a GPU plugged into a PCIe slot, this transfer would be 5-10x slower. The GH200 is special because the CPU and GPU are on the same chip package."

"The takeaway: moving data costs time. If you only do a tiny computation on the GPU, the transfer can take longer than the math. That's why GPUs shine on big problems."

Give them 1 minute for YOUR TURN.

Expected answers:
1. Yes, actually. The transfer benchmark showed 100 MB takes about 0.0004 seconds (226 GB/s bandwidth on the GH200). So the transfer is faster than the computation. But on a regular PCIe GPU, the transfer would be much slower, and it might not be worth it.
2. NVLink-C2C has much higher bandwidth than PCIe (we saw 200+ GB/s vs typical 12-25 GB/s for PCIe). This means the "break-even" point where GPU becomes worth it comes sooner.

---

## 0:50–0:55 | Part 4: Image Processing (~5 min)

Run the image processing cell.

"64 million pixels. The CPU processes them one region at a time. The GPU processes millions of regions simultaneously."

"The notebook uses `torch.nn.functional.conv2d`, the same convolution operation used inside neural networks for image recognition. When you hear about AI models 'seeing' images, this is the fundamental operation under the hood."

Actual output: CPU blur took 0.433 seconds, GPU blur took 0.152 seconds, about a 2.9x speedup.

"The speedup here is smaller than matrix multiply because the image processing is more memory-bound than compute-bound. But we're still doing the same operation on 64 million pixels in a fraction of a second."

Run the visualization cell.

"Left: random noise. Right: the same image after a blur filter. Every pixel was averaged with its 15x15 neighbors. The GPU did this for all 64 million pixels essentially at once."

---

## 0:55–1:05 | Part 5: The Big Finish (~10 min)

Run the 20,000x20,000 GPU multiply.

"Remember Tuesday? This took over 7 minutes — 430 seconds. 16 trillion operations. Let's see what the GPU does."

Actual output from the run:
```
Matrix size: 20,000 x 20,000
Operations:  16.0 trillion

GPU time:       0.3 seconds
GPU throughput: 49.9 TFLOPS

Tuesday's CPU time: ~430 seconds
Today's GPU time:   0.3 seconds
Speedup:            ~1341x
```

"The problem that took 7 minutes on the CPU took a third of a second on the GPU. 1,341 times faster. Same math. Same node. Different hardware."

"And look at the throughput: 49.9 TFLOPS. That's nearly 50 trillion operations per second. Tuesday the CPU did 0.037 TFLOPS. The GPU is doing over 1,000 times more math per second."

Review the summary table. Key points:
- "Not everything belongs on a GPU. Reading a file, parsing text, complex if/else logic — that's CPU work."
- "The GPU is for parallel math on big data. If your loop does the same thing to every element, that's a GPU problem."
- "In the real world: AI training is GPU. Physics simulations are GPU. Image processing is GPU. Data cleaning is CPU."

---

## 1:05–1:15 | Exit Ticket + Wrap Up (10 min)

"Scroll to the exit ticket. Answer the 4 questions and submit on Canvas."

Expected answers:
1. NVIDIA GH200 120GB, 102 GB of GPU memory.
2. About 480x faster (20 seconds on CPU vs 0.04 seconds on GPU).
3. Use `device='cuda'` when creating a tensor (e.g., `torch.tensor([1, 2, 3], device='cuda')`) or `.to('cuda')` to move existing data to the GPU. Use `.cpu()` to move data back to the CPU.
4. Small datasets, complex branching logic, string processing, file I/O, or any task that can't be parallelized.

**Closing:**

"In two sessions you've seen the three levels of HPC computing: single core, multicore, and GPU. You've run Python on a supercomputer, you've seen what happens when problems get too big for a laptop, and you've seen how a GPU can turn a 7-minute problem into a few-second problem."

"And you did it using PyTorch — the same framework that powers ChatGPT, image generators, and scientific AI. This isn't a classroom exercise; this is the real tool."

"Save your notebook. When you're done, just close the browser tab. TAP will clean up the job when the time runs out."

---

## If Things Go Wrong

**GPU not detected (`torch.cuda.is_available()` returns False):** Make sure you're on a `gh` queue node. If you're on a CPU-only node, PyTorch won't find a GPU. Resubmit through TAP with Queue set to `gh`.

**Out of GPU memory:** Run `torch.cuda.empty_cache()` to release unused GPU memory. Or restart the kernel. You can also `del` large tensors you no longer need.

**TAP job stuck in Pending:** The queue may be full. Wait a few minutes. If it doesn't start, try reducing the time limit to `01:00:00`.

**Student didn't git pull:** They'll have the old notebook. Have them open a terminal in Jupyter and run:
```bash
cd $WORK/hpc-python-gpu && git pull
```
Then refresh the file browser.

**Plots don't show:** Run `%matplotlib inline` in a cell before the plot.

**Slow GPU times (no speedup):** The kernel might not be using the GPU. Check `torch.cuda.is_available()` — should return `True`. If `False`, the CUDA driver isn't loaded. Restart the kernel.
