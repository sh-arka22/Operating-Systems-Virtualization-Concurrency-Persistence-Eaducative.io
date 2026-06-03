# 📘 Operating Systems: Three Easy Pieces
### Virtualization · Concurrency · Persistence

> **Course:** [Educative.io — Operating Systems: Virtualization, Concurrency & Persistence](https://www.educative.io/courses/operating-systems-virtualization-concurrency-persistence)
> **Authors:** Remzi H. Arpaci-Dusseau & Andrea C. Arpaci-Dusseau (OSTEP)

---

## 📚 Table of Contents

### Part 1 — Virtualization
| # | Chapter | Topics |
|---|---|---|
| 1 | Introduction | OS overview, design goals, history |
| 2 | Processes | Abstraction, process API, states, data structures |
| 3 | Process API | fork(), wait(), exec() |
| 4 | Direct Execution | Limited direct execution, restricted ops, switching |
| 5 | CPU Scheduling | FIFO, SJF, STCF, Round Robin, I/O |
| 6 | Multi-Level Feedback | MLFQ rules, priority boost, accounting |
| 7 | Lottery Scheduling | Tickets, CFS, weighting |
| 8 | Multi-CPU Scheduling | Cache affinity, single/multi-queue, Linux schedulers |
| 9 | Address Space | Early systems, multiprogramming, goals |
| 10 | Memory API | malloc(), free(), common errors |
| 11 | Address Translation | Dynamic relocation, hardware support |
| 12 | Segmentation | Base/bounds, stack, sharing |
| 13 | Free Space Management | Low-level mechanisms, strategies |
| 14 | Paging | Page tables, where stored, TLB |
| 15 | Translation Lookaside Buffers | TLB algorithm, context switches |
| 16 | Advanced Page Tables | Multi-level, inverted, hybrid |
| 17 | Swapping: Mechanisms | Swap space, page fault, control flow |
| 18 | Swapping: Policies | LRU, FIFO, optimal, clock algorithm |
| 19 | Complete VM Systems | VAX/VMS, Linux VM |

### Part 2 — Concurrency
| # | Chapter | Topics |
|---|---|---|
| [20](./20-Concurrency-Concurrency-and-Threads/) | **Concurrency and Threads** | Threads, TCB, context switch, race conditions |
| 21 | Thread API | pthread create, join, locks, condition variables |
| 22 | Locks | Spin locks, test-and-set, compare-and-swap |
| 23 | Locked Data Structures | Concurrent counters, lists, queues, hash tables |
| 24 | Condition Variables | Producer/consumer, bounded buffer |
| 25 | Semaphores | Binary semaphores, ordering, reader-writer locks |
| 26 | Concurrency Bugs | Deadlock, atomicity violation, order violation |
| 27 | Event-based Concurrency | select(), poll(), async I/O |

### Part 3 — Persistence
| # | Chapter | Topics |
|---|---|---|
| 28 | I/O Devices | I/O bus, DMA, device drivers |
| 29 | Hard Disk Drives | Geometry, scheduling, RAID |
| 30 | Files and Directories | File system interface, links |
| 31 | File System Implementation | Inodes, bitmaps, superblock |
| 32 | Fast File System | FFS, locality, large files |
| 33 | FSCK and Journaling | Crash consistency, journaling |
| 34 | Log-structured File Systems | LFS, segments, cleaning |
| 35 | Flash-based SSDs | NAND flash, FTL, wear leveling |
| 36 | Data Integrity and Protection | Checksums, redundancy |

---

## 🗂️ Folder Structure

```
📁 Operating-Systems-Virtualization-Concurrency-Persistence-Eaducative.io/
│
├── 📄 README.md                          ← You are here (Book Index)
│
├── 📁 20-Concurrency-Concurrency-and-Threads/
│   ├── 📁 01-Introduction-to-Concurrency-and-Threads/
│   │   └── 📄 README.md                 ← Visual Mermaid Notes
│   ├── 📁 02-Why-Use-Threads/
│   ├── 📁 03-An-Example-Thread-Creation/
│   ├── 📁 04-Why-It-Gets-Worse-Shared-Data/
│   ├── 📁 05-The-Heart-Of-The-Problem-Uncontrolled-Scheduling/
│   ├── 📁 06-The-Wish-For-Atomicity/
│   └── 📁 07-One-More-Problem-Waiting-For-Another/
│
└── ... (more chapters to be added)
```

---

## 🚀 How to Use These Notes

- Each **chapter folder** maps to a section in the Educative.io course
- Each **lesson subfolder** contains a `README.md` with:
  - Full **Mermaid diagrams** (mindmaps, flowcharts, sequence diagrams, state machines)
  - **Comparison tables**
  - **Cheat sheets**
  - **Key concept cards**

---

> *Visual notes for maximum retention — every concept as a diagram.*
