# 01 — Introduction to Thread API

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 21 — Concurrency: Thread API

## Overview

This lesson introduces the POSIX Thread (pthread) API — the standard interface for creating and managing threads in C programs on Unix/Linux systems. It serves as a high-level reference before deeper dives into synchronization mechanisms.

## Key Crux

**CRUX: HOW TO CREATE AND CONTROL THREADS**
> What interfaces should the OS present for thread creation and control? How should these interfaces be designed to enable ease of use as well as utility?

## What is the Thread API?

- The **POSIX thread library** (`pthreads`) provides the standard set of functions for:
  - Creating threads
  - Waiting for thread completion
  - Mutual exclusion (locks)
  - Signaling between threads (condition variables)
- Include `<pthread.h>` in your C program to use it.
- Each concept (locks, condition variables) is introduced here briefly and explored in depth in subsequent chapters.

## Why Learn the Thread API?

| Reason | Explanation |
|--------|-------------|
| Concurrency | Run multiple tasks in parallel within a single process |
| Shared Memory | Threads share the same address space — easy data sharing |
| Responsiveness | One thread can block on I/O while others continue running |
| Standard | POSIX threads are portable across Linux, macOS, Unix systems |

---
*Source: Educative.io — OS: Virtualization, Concurrency & Persistence — Ch.21*
