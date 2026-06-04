# 02 — Thread Creation

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 21 — Concurrency: Thread API

## Overview

To write any multi-threaded program, you first need to create threads. In POSIX, this is done using `pthread_create()`.

## `pthread_create()` Signature

```c
#include <pthread.h>

int pthread_create(
    pthread_t       *thread,
    const pthread_attr_t *attr,
    void *(*start_routine)(void *),
    void            *arg
);
```

## Parameter Breakdown

| Parameter | Type | Description |
|-----------|------|-------------|
| `thread` | `pthread_t *` | Pointer to store thread ID; used later to join |
| `attr` | `pthread_attr_t *` | Thread attributes (stack size, priority). Pass `NULL` for defaults |
| `start_routine` | `void *(*)(void *)` | Function pointer the new thread will execute |
| `arg` | `void *` | Argument passed to `start_routine`. Cast to/from `void *` |

## Example

```c
#include <pthread.h>
#include <stdio.h>

void *my_func(void *arg) {
    int val = *(int *)arg;
    printf("Thread value: %d\n", val);
    return NULL;
}

int main() {
    pthread_t t;
    int x = 42;
    pthread_create(&t, NULL, my_func, &x);
    pthread_join(t, NULL);
    return 0;
}
```

## Key Points

- The new thread executes `start_routine` immediately after creation.
- Thread execution order is decided by the OS scheduler — not guaranteed.
- Pack complex arguments into a `struct` and pass a pointer to it.
- Pass pointers to **heap-allocated** data to avoid dangling references.

---
*Source: Educative.io — OS: Virtualization, Concurrency & Persistence — Ch.21*
