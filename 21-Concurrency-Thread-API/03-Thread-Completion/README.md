# 03 — Thread Completion

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Chapter:** 21 — Concurrency: Thread API

## Overview

After creating a thread, you often need to wait for it to finish. POSIX provides `pthread_join()` for this.

## `pthread_join()` Signature

```c
int pthread_join(pthread_t thread, void **value_ptr);
```

| Parameter | Description |
|-----------|-------------|
| `thread` | The thread ID to wait for |
| `value_ptr` | Captures the return value. Pass `NULL` if not needed |

## How It Works

1. The calling thread **blocks** until the target thread finishes.
2. The return value of `start_routine` is placed at `*value_ptr`.
3. Join each thread only once — re-joining is undefined behavior.

## Example

```c
void *compute(void *arg) {
    int *result = malloc(sizeof(int));
    *result = 42;
    return (void *)result;  // heap-allocated: safe
}

int main() {
    pthread_t t;
    int *retval;
    pthread_create(&t, NULL, compute, NULL);
    pthread_join(t, (void **)&retval);
    printf("Result: %d\n", *retval);
    free(retval);
}
```

## Critical Pitfall: Never Return Stack Pointers

```c
// WRONG!
void *bad_func(void *arg) {
    int result = 42;
    return (void *)&result;  // DANGLING POINTER - stack destroyed on exit!
}
```

When a thread exits, its **stack is destroyed**. Always use **heap-allocated memory** (`malloc`) for values returned from threads.

## Key Points

- `pthread_join()` is the thread equivalent of `waitpid()` for processes.
- **Detached threads** (`pthread_detach()`) run independently and cannot be joined.
- A thread that is neither joined nor detached **leaks resources**.

---
*Source: Educative.io — OS: Virtualization, Concurrency & Persistence — Ch.21*
