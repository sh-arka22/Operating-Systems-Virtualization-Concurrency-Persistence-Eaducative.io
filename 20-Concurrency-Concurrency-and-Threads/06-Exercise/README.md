# 06 — Exercise: Concurrency and Threads (x86.py Simulator)

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 20 — Concurrency: Concurrency and Threads
> **Topic:** x86 Thread Simulator — Race Conditions, Registers, Shared Memory

---

## Overview

This exercise uses the `x86.py` simulator to explore how different thread interleavings either cause or avoid race conditions. The simulator runs assembly programs with varying numbers of threads and interrupt intervals to observe concurrency effects.

**Programs used:**
- `loop.s` — simple loop counting down a register
- `looping-race-nolock.s` — shared memory updates without locks
- `wait-for-me.s` — thread synchronization via shared memory

**Key flags:**
| Flag | Description |
|------|-------------|
| `-p` | Program file |
| `-t` | Number of threads |
| `-i` | Interrupt interval |
| `-R` | Trace a specific register |
| `-M` | Trace a specific memory address |
| `-a` | Initialize register values |
| `-r` | Use random interrupt intervals |
| `-s` | Random seed |
| `-c` | Check answers (show actual values after each instruction) |

---

## Questions & Answers

### Q1 — `loop.s` with a single thread

**Command:** `./x86.py -p loop.s -t 1 -i 100 -R dx`

This specifies a single thread, an interrupt every 100 instructions, and traces register `%dx`. What will `%dx` be during the run? Use `-c` to check.

**Answer:**
`loop.s` decrements `%dx` each iteration until it reaches 0. With a single thread and interrupt interval of 100, there is no interleaving. `%dx` starts at 0 (default), counts from 0 downward — but since the program initializes `%dx` from its default (0), the loop doesn't execute iterations (already at 0). `%dx` remains **0** throughout. Use `-c` to verify the values after each instruction.

---

### Q2 — `loop.s` with two threads, `dx=3` each

**Command:** `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx`

This specifies two threads, each with `%dx` initialized to 3. What values will `%dx` see? Does the presence of multiple threads affect calculations? Is there a race in this code?

**Answer:**
Each thread has its **own private register set** (registers are not shared between threads). Thread 0 counts `%dx` from 3 down to 0, and Thread 1 independently does the same. The outputs are interleaved in the trace but each thread's `%dx` is isolated. **There is no race condition** — registers are per-thread, only shared memory can cause races. The presence of multiple threads does not affect each thread's own `%dx` calculation.

---

### Q3 — `loop.s` with random interrupt intervals

**Command:** `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`

This makes the interrupt interval small/random. Use different seeds (`-s`) to see different interleavings. Does the interrupt frequency change anything?

**Answer:**
Since `%dx` is a private per-thread register, **different interrupt frequencies do not change the correctness** of each thread's computation. Each thread still counts its own `%dx` from 3 to 0 correctly regardless of when interrupts occur. The interleaving order changes (different seeds produce different interleavings), but the final result for each thread is always correct — **no race condition exists** on registers.

---

### Q4 — `looping-race-nolock.s` with one thread

**Command:** `./x86.py -p looping-race-nolock.s -t 1 -M 2000`

This program accesses a shared variable at memory address 2000 (`value`). Run with a single thread. What is `value` throughout the run? Use `-c` to check.

**Answer:**
With a single thread, there is no interleaving and no race. The program loads `mem[2000]`, increments it, and stores it back repeatedly. With one thread, `value` at address 2000 **increments correctly** by 1 each iteration. The final value equals the number of loop iterations (default is 1 iteration, so final `value = 1`). No race condition is possible with a single thread.

---

### Q5 — `looping-race-nolock.s` with two threads, `bx=3`

**Command:** `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`

Each thread loops 3 times (controlled by `bx`). Why does each thread loop three times? What is the final value of `value`?

**Answer:**
`bx` controls the number of iterations per thread — the loop decrements `bx` and continues while `bx > 0`, so `bx=3` means 3 iterations. With 2 threads each looping 3 times, you'd expect `value = 6` if no race occurs. However, because **no lock protects the critical section** (`load → add → store`), a race condition may occur depending on the interrupt timing. The final value of `value` **may be less than 6** (e.g., 4 or 5) if threads interleave inside the critical section causing lost updates.

---

### Q6 — Random interrupt intervals and the critical section

**Command:** `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`

Try different seeds (`-s 1`, `-s 2`, etc.). Can you tell by looking at the thread interleaving what the final value of `value` will be? Does the timing of the interrupt matter? Where can it safely occur? Where not? Where is the critical section exactly?

**Answer:**
- **Critical section:** The three instructions `load mem[2000]` → `add $1, %ax` → `store %ax, mem[2000]`. All three must execute atomically to avoid a race.
- **Safe interrupt locations:** Anywhere **outside** the critical section (before the load or after the store). If the interrupt occurs between these 3 instructions, the update can be lost.
- **Unsafe interrupt locations:** Between `load` and `store` — if Thread 1 loads the old value, gets interrupted, Thread 2 increments and stores, then Thread 1 stores its stale incremented value, the second update is **lost**.
- The timing of the interrupt **does** matter — it determines whether a race condition occurs.

---

### Q7 — Fixed interrupt intervals

**Command:** `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`

What will the final value of `value` be? What about `-i 2`, `-i 3`, etc.? For which interrupt intervals does the program give the correct answer?

**Answer:**
- **`-i 1`:** Interrupt after every instruction — threads interleave maximally. The critical section (3 instructions) will almost certainly be interrupted mid-way → **race condition, value likely = 1** instead of 2.
- **`-i 2`:** Still interrupts inside the critical section (after 2 of the 3 instructions) → **race condition possible**.
- **`-i 3`:** The critical section is exactly 3 instructions. If the interrupt fires at a multiple of 3 that aligns correctly, it may execute the full critical section atomically → **may give correct answer (value = 2)** depending on alignment.
- **`-i 4` or larger:** More likely to complete the critical section before an interrupt → **correct answer more often**.
- Intervals that are **multiples of 3 or larger** tend to produce correct results; intervals of 1 or 2 tend to cause races.

---

### Q8 — More loops and correct interrupt intervals

**Command:** `./x86.py -p looping-race-nolock.s -a bx=100 -M 2000 ...`

Run for more loops (e.g., set `-a bx=100`). What interrupt intervals (`-i`) lead to a correct outcome? Which are surprising?

**Answer:**
- With more iterations, **only interrupt intervals that never land inside the 3-instruction critical section** guarantee correctness.
- **Multiples of 3** (e.g., `-i 3`, `-i 6`, `-i 9`) that start from before the critical section are safe because the scheduler always switches between complete critical section executions.
- **Surprising:** `-i 3` does NOT always work — it depends on the alignment of the interrupt with respect to where in the code the thread currently is. If the thread starts mid-critical-section, an interval of 3 can still cause a race.
- The only truly safe approach is **proper locking** (`pthread_mutex_lock`), not relying on interrupt timing.

---

### Q9 — `wait-for-me.s`: Thread 0 signals, Thread 1 waits

**Command:** `./x86.py -p wait-for-me.s -a ax=1,ax=0 -R ax -M 2000`

This sets `%ax` to 1 for thread 0, and 0 for thread 1, and watches `%ax` and memory location 2000. How should the code behave? How is the value at location 2000 being used by the threads? What will its final value be?

**Answer:**
- **Thread 0** (`ax=1`): Acts as the **signaler** — it sets `mem[2000] = 1` to signal it is done.
- **Thread 1** (`ax=0`): Acts as the **waiter** — it spin-loops checking `mem[2000]` until it sees the value 1.
- `mem[2000]` is used as a **flag/condition variable** for synchronization between threads.
- **Final value of `mem[2000]` = 1** (set by Thread 0 to signal completion).
- Thread 1 loops spinning (wastefully consuming CPU) until Thread 0 sets the flag, then Thread 1 proceeds and exits.

---

### Q10 — `wait-for-me.s`: Switch inputs, observe efficiency

**Command:** `./x86.py -p wait-for-me.s -a ax=0,ax=1 -R ax -M 2000`

How do the threads behave now? What is thread 0 doing? How would changing the interrupt interval (e.g., `-i 1000`, or random intervals) change the trace outcome? Is the program efficiently using the CPU?

**Answer:**
- Now **Thread 0** (`ax=0`): Acts as the **waiter** — it spin-loops on `mem[2000]`.
- **Thread 1** (`ax=1`): Acts as the **signaler** — it sets `mem[2000] = 1`.
- **Thread 0** spins in a busy-wait loop, **wasting CPU cycles** until Thread 1 gets scheduled and sets the flag.
- With `-i 1000`, Thread 0 may spin for a very long time before Thread 1 gets a turn → **very inefficient**, many wasted CPU cycles shown in the trace.
- With random/small intervals, Thread 1 gets scheduled sooner → less spinning.
- **The program is NOT efficiently using the CPU** — spin-waiting (busy-waiting) wastes processor time. The correct solution is to use `pthread_cond_wait()` / `pthread_cond_signal()` which puts the waiting thread to sleep instead of spinning.

---

## Key Takeaways

| Concept | Insight |
|---|---|
| **Registers are per-thread** | `%dx`, `%ax` etc. are private — no race on registers |
| **Shared memory causes races** | `mem[2000]` is shared — unprotected increments cause lost updates |
| **Critical section** | `load → add → store` (3 instructions) must be atomic |
| **Safe interrupt intervals** | Intervals that never land inside the critical section avoid races |
| **Spin-waiting** | Correct but CPU-wasteful — replace with `pthread_cond_wait` |
| **Locks are the real fix** | Relying on interrupt timing is fragile; always use proper synchronization |

---

*Source: Educative.io — Operating Systems: Virtualization, Concurrency & Persistence — Chapter 20 Exercise*
