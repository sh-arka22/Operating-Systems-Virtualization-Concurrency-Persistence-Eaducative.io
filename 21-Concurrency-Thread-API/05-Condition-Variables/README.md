# 05 — Condition Variables

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 21 — Concurrency: Thread API

## Overview

Condition variables allow threads to **signal each other** — one thread waits for a condition to become true, another thread signals when it changes. They must always be used with a mutex lock.

## Core Routines

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

| Routine | Behaviour |
|---------|----------|
| `pthread_cond_wait` | Puts the calling thread to **sleep**; atomically releases the mutex while sleeping and re-acquires it before returning |
| `pthread_cond_signal` | Wakes **one** sleeping thread waiting on the condition |
| `pthread_cond_broadcast` | Wakes **all** sleeping threads waiting on the condition |

## Initialization

```c
// Static
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// Dynamic
pthread_cond_t cond;
pthread_cond_init(&cond, NULL);
// ... use ...
pthread_cond_destroy(&cond);
```

## Correct Usage Pattern

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER;
int ready = 0;

// --- WAITER THREAD ---
void *waiter(void *arg) {
    pthread_mutex_lock(&lock);
    while (ready == 0)               // use WHILE, not if
        pthread_cond_wait(&cond, &lock);
    // condition is now true
    pthread_mutex_unlock(&lock);
    return NULL;
}

// --- SIGNALER THREAD ---
void *signaler(void *arg) {
    pthread_mutex_lock(&lock);
    ready = 1;                       // change state
    pthread_cond_signal(&cond);      // wake waiter
    pthread_mutex_unlock(&lock);
    return NULL;
}
```

## Why `while` and NOT `if`?

Always use a **`while` loop** to re-check the condition after waking up:

- Some pthread implementations cause **spurious wakeups** — a thread wakes up even though the condition hasn't changed.
- Treat the wakeup as a **hint** — always re-verify the condition.

```c
// WRONG - susceptible to spurious wakeups
if (ready == 0)
    pthread_cond_wait(&cond, &lock);

// CORRECT
while (ready == 0)
    pthread_cond_wait(&cond, &lock);
```

## Why Not a Simple Flag/Spin?

```c
// BAD - busy-wait/spin flag
while (ready == 0);
    ; // wastes CPU, error-prone
```

Research shows roughly **half of ad hoc synchronization** attempts using simple flags contain bugs. Always use condition variables.

## Key Points

- The lock **must be held** when calling both `wait` and `signal`.
- `pthread_cond_wait` atomically releases the lock — no race between check and sleep.
- `signal` wakes one waiter; `broadcast` wakes all waiters.
- Always use a `while` loop, never `if`, when checking the condition.

---
*Source: Educative.io — OS: Virtualization, Concurrency & Persistence — Ch.21*
