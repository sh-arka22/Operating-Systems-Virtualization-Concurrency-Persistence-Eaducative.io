# Operating Systems: Virtualization, Concurrency & Persistence

> 📚 Notes from [Educative.io](https://www.educative.io/courses/operating-systems-virtualization-concurrency-persistence) | Chapter 20: Concurrency and Threads

---

## 🧵 Introduction to Concurrency and Threads

### 📌 What is a Thread?

A **thread** is a new abstraction for a single running process — like a separate process, except threads **share the same address space** and can access the same data.

---

## 🗺️ Big Picture: OS Abstractions

```mermaid
mindmap
  root((OS Abstractions))
    Virtual CPU
      Processes
        Single point of execution
        Own address space
        PCB stores state
    Virtual Memory
      Address Space
        Code segment
        Heap segment
        Stack segment
    Concurrency
      Threads
        Multiple points of execution
        Shared address space
        TCB stores state
        Own stack per thread
```

---

## ⚡ Single-Threaded vs Multi-Threaded

```mermaid
flowchart LR
    subgraph ST["🔵 Single-Threaded Process"]
        direction TB
        PC1(["PC → single\ninstruction stream"])
        PC1 --> I1[Instruction 1]
        I1 --> I2[Instruction 2]
        I2 --> I3[Instruction 3]
    end

    subgraph MT["🟣 Multi-Threaded Process"]
        direction TB
        PC_A(["PC₁ → Thread 1"]) --> A1[Instruction A]
        PC_B(["PC₂ → Thread 2"]) --> B1[Instruction X]
        PC_C(["PC₃ → Thread 3"]) --> C1[Instruction M]
        note1["All share the SAME\naddress space!"]
    end

    ST -- "add threads" --> MT
```

---

## 🧠 Thread State: What Each Thread Owns

```mermaid
flowchart TD
    T(["🧵 THREAD"])
    T --> PRIV["🔒 PRIVATE per-thread"]
    T --> SHARED["🌐 SHARED across all threads"]

    PRIV --> PC["Program Counter\n(tracks current instruction)"]
    PRIV --> REG["Register Set\n(EAX, EBX, ESP, EBP...)"]
    PRIV --> STK["Stack\n(local vars, params,\nreturn values)"]

    SHARED --> CODE["Code Segment\n(program instructions)"]
    SHARED --> HEAP["Heap Segment\n(malloc'd memory)"]
    SHARED --> GLOB["Global Variables"]

    style PRIV fill:#4c1d95,color:#f3f4f6
    style SHARED fill:#14532d,color:#f3f4f6
    style PC fill:#1e1b4b,color:#c7d2fe
    style REG fill:#1e1b4b,color:#c7d2fe
    style STK fill:#1e1b4b,color:#c7d2fe
    style CODE fill:#052e16,color:#bbf7d0
    style HEAP fill:#052e16,color:#bbf7d0
    style GLOB fill:#052e16,color:#bbf7d0
```

---

## ⚖️ Thread vs Process: Key Differences

| Property | Process | Thread |
|---|---|---|
| **Address Space** | Own (isolated) | **Shared** with other threads |
| **Page Table** | Own (switched on context switch) | **Same** — not switched! |
| **Control Block** | PCB (Process Control Block) | **TCB** (Thread Control Block) |
| **Stack** | Single stack | **One stack per thread** |
| **Context Switch Cost** | Heavyweight (TLB flush required) | **Lightweight** (registers only) |
| **Data Access** | Isolated | **Shared** → risk of race conditions |

---

## 🔄 Context Switch: Process vs Thread

```mermaid
sequenceDiagram
    participant T1 as Thread 1 (T1)
    participant OS as 🖥️ OS Scheduler
    participant T2 as Thread 2 (T2)

    Note over T1,T2: THREAD Context Switch (Lightweight ✅)
    T1->>OS: Save T1 registers → TCB₁
    Note over OS: ✅ NO page table switch!<br/>✅ NO TLB flush!<br/>Address space stays the same.
    OS->>T2: Restore T2 registers ← TCB₂
    T2->>T2: Continues execution
```

```mermaid
sequenceDiagram
    participant P1 as Process A
    participant OS as 🖥️ OS Scheduler
    participant P2 as Process B

    Note over P1,P2: PROCESS Context Switch (Heavyweight ❌)
    P1->>OS: Save P1 registers → PCB₁
    OS->>OS: Switch Page Table
    OS->>OS: Flush TLB (expensive!)
    OS->>P2: Restore P2 registers ← PCB₂
    P2->>P2: Continues execution
```

---

## 📦 Thread Control Block (TCB) Structure

```mermaid
flowchart LR
    subgraph Process["🏠 Process"]
        PCB_BOX["PCB\n(Process Control Block)"]
        subgraph Threads["Threads within Process"]
            TCB1["TCB₁\n─────────\nThread ID: 1\nPC: 0x4A2F\nSP: 0xFF10\nRegisters: [...]\nState: RUNNING"]
            TCB2["TCB₂\n─────────\nThread ID: 2\nPC: 0x8C11\nSP: 0xEE40\nRegisters: [...]\nState: READY"]
            TCB3["TCB₃\n─────────\nThread ID: 3\nPC: 0x1D55\nSP: 0xDD20\nRegisters: [...]\nState: BLOCKED"]
        end
    end

    style TCB1 fill:#1e3a5f,color:#bfdbfe
    style TCB2 fill:#14532d,color:#bbf7d0
    style TCB3 fill:#3b0000,color:#fecaca
```

---

## 🏠 Address Space Layout: Single vs Multi-Threaded

```mermaid
flowchart TB
    subgraph ST["Single-Threaded Process"]
        direction TB
        st_code["📜 Code (0KB)"]
        st_heap["🟢 Heap (grows ↓)"]
        st_free["⬜ Free Space"]
        st_stack["🟣 Stack (grows ↑)\n(16KB)"]
        st_code --> st_heap --> st_free --> st_stack
    end

    subgraph MT["Multi-Threaded Process (2 threads)"]
        direction TB
        mt_code["📜 Code (0KB)"]
        mt_heap["🟢 Heap (grows ↓)"]
        mt_free["⬜ Free Space"]
        mt_s2["🟣 Stack (Thread 2)"]
        mt_free2["⬜ Free Space"]
        mt_s1["🟣 Stack (Thread 1)\n(16KB)"]
        mt_code --> mt_heap --> mt_free --> mt_s2 --> mt_free2 --> mt_s1
    end

    ST -- "add Thread 2" --> MT
```

> ⚠️ Each thread gets its **own stack** = thread-local storage. Code + Heap are **SHARED**.

---

## 🔁 Thread Lifecycle (State Machine)

```mermaid
stateDiagram-v2
    [*] --> NEW : Thread created
    NEW --> READY : Initialized
    READY --> RUNNING : OS dispatches (time slice)
    RUNNING --> READY : Preempted / time slice ends
    RUNNING --> BLOCKED : Waiting for I/O or event
    BLOCKED --> READY : I/O complete / event occurs
    RUNNING --> [*] : Thread exits / completes

    note right of RUNNING
        Thread has the CPU.
        PC and registers active.
    end note

    note right of BLOCKED
        Waiting — not using CPU.
        State saved in TCB.
    end note
```

---

## ⚠️ The Concurrency Problem: Race Conditions

```mermaid
sequenceDiagram
    participant T1 as 🔵 Thread 1
    participant MEM as 📦 Shared counter
    participant T2 as 🔴 Thread 2

    Note over MEM: counter = 50

    T1->>MEM: READ counter (gets 50)
    T2->>MEM: READ counter (gets 50) ← scheduled before T1 writes!
    T1->>T1: increment → 51
    T2->>T2: increment → 51
    T1->>MEM: WRITE counter = 51
    T2->>MEM: WRITE counter = 51 ❌ (should be 52!)

    Note over MEM: ❌ LOST UPDATE!<br/>counter = 51 (expected 52)
    Note over T1,T2: This is a RACE CONDITION<br/>caused by uncontrolled scheduling
```

---

## 🛡️ Solutions to Concurrency Problems (Preview)

```mermaid
mindmap
  root((Concurrency
Solutions))
    Locks and Mutexes
      Only one thread in critical section
      pthread_mutex_lock
      pthread_mutex_unlock
    Semaphores
      Counting mechanism
      Binary semaphore = lock
      sem_wait and sem_post
    Condition Variables
      Thread waits for a condition
      pthread_cond_wait
      pthread_cond_signal
    Atomic Operations
      Hardware-level guarantee
      Test-and-Set
      Compare-and-Swap
      Fetch-and-Add
```

---

## 📋 Complete Concept Map

```mermaid
flowchart TD
    OS["🖥️ Operating System"] --> VIRT["Virtualization"]
    OS --> CONC["Concurrency"]
    OS --> PERS["Persistence"]

    VIRT --> VCPU["Virtual CPU\n→ Processes"]
    VIRT --> VMEM["Virtual Memory\n→ Address Spaces"]

    CONC --> THR["Threads"]
    THR --> SHR["Share Address Space"]
    THR --> OWN["Own: PC + Registers + Stack"]
    THR --> TCB["Stored in TCB"]
    THR --> CTX["Context Switch\n(lightweight)"]

    SHR --> RACE["⚠️ Race Conditions"]
    RACE --> LOCK["🔒 Locks"]
    RACE --> SEM["🚦 Semaphores"]
    RACE --> COND["📢 Condition Variables"]
    RACE --> ATOM["⚛️ Atomic Ops"]

    style OS fill:#1e293b,color:#f1f5f9
    style CONC fill:#4c1d95,color:#f3f4f6
    style THR fill:#1e1b4b,color:#c7d2fe
    style RACE fill:#7f1d1d,color:#fecaca
    style LOCK fill:#14532d,color:#bbf7d0
    style SEM fill:#14532d,color:#bbf7d0
    style COND fill:#14532d,color:#bbf7d0
    style ATOM fill:#14532d,color:#bbf7d0
```

---

## 📝 Quick-Reference Cheat Sheet

| Term | Definition |
|---|---|
| **Thread** | Unit of execution within a process; shares address space |
| **Multi-threaded** | Program with multiple PCs (points of execution) |
| **TCB** | Thread Control Block — stores thread's saved register state |
| **PCB** | Process Control Block — stores process state |
| **Context Switch** | Save current thread state → restore next thread state |
| **Thread-local** | Data private to one thread (its own stack) |
| **Race Condition** | Bug from unsynchronized access to shared data |
| **Critical Section** | Code region accessing shared/mutable data |
| **Atomicity** | "All or nothing" — operation completes without interruption |
| **Shared Address Space** | Threads see the same code, heap, and globals |

---

## 🔗 Source

- **Course:** [Operating Systems: Virtualization, Concurrency & Persistence — Educative.io](https://www.educative.io/courses/operating-systems-virtualization-concurrency-persistence)
- **Chapter:** 20 · Concurrency and Threads
- **Lesson:** Introduction to Concurrency and Threads

---

> *Notes compiled for visual learning — every concept represented as a Mermaid diagram for maximum retention.*
