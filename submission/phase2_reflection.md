# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->249.936<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->0.983<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->459398.5<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12667.9<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4932<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2709.644 ms
2 threads: 2190.477 ms
4 threads: 2345.048 ms
8 threads: 2736.945 ms 
The fastest count is achieved with 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive version wastes time on Python interpreter overhead: each loop iteration involves type checks and the creation of objects. The vectorized version delegates matrix calculations to a BLAS library compiled in C. On the machine, this utilizes the processor’s AVX/SIMD instructions, allowing the same arithmetic operation to be performed on multiple floating-point numbers in a single clock cycle.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The curve flattens out and then worsens after 2 threads (with the minimum reached at 2190.477 ms for 2 threads). The main reason is hardware-related: the free Colab environment provides only 2 vCPUs. Running 4 or 8 threads forces the processor to perform context switching and increases the synchronization overhead between threads, which degrades performance.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
A transformer training step is dominated by dense matrix multiplications, a computation that is perfectly parallelizable. The GPU, with its thousands of CUDA cores, excels at this massively parallel task. A workload that would not benefit from this acceleration would be an iterative and strictly sequential algorithm, such as Dijkstra’s algorithm (shortest path), where the extraction of the nearest node blocks the update of its neighbors and prevents efficient concurrent execution.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism ceases to be useful when the cost of coordination exceeds the computational gain, or when the hardware is saturated. Vectorization eliminated the software overhead (reducing the time from 249.936 ms to 0.983 ms), but the thread sweep demonstrated the hardware limit: adding threads beyond Colab’s 2 physical vCPUs increased the execution time (from 2,190.477 ms to 2,736.945 ms) due to context switching. Finally, although the GPU significantly outperforms the CPU (12,667.9 ms vs. 459,398.5 ms), it will itself be limited by memory bandwidth (data transfers between the host and the device) if the batches are too small or if it is waiting for data.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would vary the batch size from 1 to 256. If, for very small batches, the CPU were to become faster than the GPU again, that would prove that the overhead associated with launching CUDA kernels and data transfers completely negates the benefits of massive parallelism.
<!--/answer-->
