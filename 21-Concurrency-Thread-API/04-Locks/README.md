# 04 — Locks

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 21 — Concurrency: Thread API

## Overview

Locks (mutexes) provide **mutual exclusion** — ensuring only one thread executes a critical section at a time.

## Core Routines

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

- If the lock is **free**: the calling thread acquires it and proceeds.
- If the lock is **held**: the calling thread **blocks** until the holder releases it.

## Initialization

### Static (preferred for global locks)
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

### Dynamic (for runtime/heap-allocated locks)
```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);  // initialize
// ... use lock ...
pthread_mutex_destroy(&lock);     // cleanup when done
```

## Additional Routines

| Function | Description |
|----------|-------------|
| `pthread_mutex_trylock()` | Try to acquire lock; returns immediately with error if held |
| `pthread_mutex_timedlock()` | Try to acquire with a timeout |

## Example: Protecting a Critical Section

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

void *increment(void *arg) {
    pthread_mutex_lock(&lock);
    counter++;           // critical section
    pthread_mutex_unlock(&lock);
    return NULL;
}
```

## Always Check Return Codes

```c
int rc = pthread_mutex_lock(&lock);
if (rc != 0) {
    fprintf(stderr, "Lock failed: %s\n", strerror(rc));
    exit(1);
}
```

Lock calls can fail silently. Always check the return value.

## Key Points

- Always initialize locks before use.
- Always unlock after the critical section to avoid **deadlock**.
- Never call blocking operations while holding a lock (can lead to starvation).
- Use `trylock` when you need non-blocking lock attempts.

---
*Source: Educative.io — OS: Virtualization, Concurrency & Persistence — Ch.21*
