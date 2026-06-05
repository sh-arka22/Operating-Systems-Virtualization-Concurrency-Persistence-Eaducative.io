# Chapter 21 — Concurrency: Thread API
## Key Questions & Answers

> **Course:** Operating Systems: Virtualization, Concurrency & Persistence
> **Source:** Educative.io — Chapter 21

---

### Q1. What is the POSIX Thread API and why do we use it?

**A:** The POSIX Thread (pthread) API is the standard interface for creating and managing threads in C on Unix/Linux systems. We use it to:
- Run tasks in **parallel** within a single process
- Allow one thread to **block on I/O** while others continue
- **Share memory** easily between threads (same address space)
- Write **portable** concurrent programs across Linux, macOS, Unix

Include `<pthread.h>` and compile with `-pthread`.

---

### Q2. What is the signature of `pthread_create()` and what does each parameter mean?

**A:**
```c
int pthread_create(
    pthread_t       *thread,           // stores the new thread's ID
    const pthread_attr_t *attr,        // attributes (NULL = defaults)
    void *(*start_routine)(void *),    // function the thread runs
    void            *arg               // argument passed to the function
);
```
| Parameter | Meaning |
|---|---|
| `thread` | Pointer to `pthread_t`; used later with `pthread_join` |
| `attr` | Thread attributes (stack size, priority). Pass `NULL` for defaults |
| `start_routine` | Function pointer the new thread executes |
| `arg` | Single `void *` argument passed to `start_routine` |

---

### Q3. Is thread execution order guaranteed after `pthread_create()`?

**A:** **No.** The OS scheduler decides when each thread runs. After calling `pthread_create()`, the new thread may run immediately, or the calling thread may continue first. The order is **non-deterministic** and can change every run. Never rely on a specific execution order without synchronization.

---

### Q4. What is `pthread_join()` and why is it needed?

**A:**
```c
int pthread_join(pthread_t thread, void **value_ptr);
```
`pthread_join()` **blocks** the calling thread until the specified thread finishes. It is needed to:
1. Wait for a thread to complete before using its results
2. Retrieve the return value from a thread's function
3. Free the resources associated with the thread (prevents resource leaks)

Without joining, the main program may exit before worker threads finish.

---

### Q5. What is the critical pitfall when returning values from threads?

**A:** **Never return a pointer to a stack-allocated (local) variable.**

```c
// WRONG - undefined behaviour!
void *bad(void *arg) {
    int result = 42;
    return (void *)&result;  // stack destroyed when thread exits!
}

// CORRECT - heap-allocated, survives thread exit
void *good(void *arg) {
    int *result = malloc(sizeof(int));
    *result = 42;
    return (void *)result;  // caller must free()
}
```

When a thread exits, its **stack is destroyed**. A pointer to a local variable becomes a **dangling pointer** — accessing it is undefined behaviour.

---

### Q6. What are the two routines for mutual exclusion (locks) in pthreads?

**A:**
```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
- **`lock`**: If the mutex is free, the thread acquires it and continues. If held by another thread, the caller **blocks** until it is released.
- **`unlock`**: Releases the mutex, allowing a waiting thread to proceed.

Always pair every `lock` with an `unlock`.

---

### Q7. How do you initialize a mutex (lock)? What are the two methods?

**A:**

**Static (for global/static variables — preferred):**
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

**Dynamic (for heap-allocated locks or local variables):**
```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);
// ... use lock ...
pthread_mutex_destroy(&lock);  // cleanup!
```

Always initialize before use. An uninitialized mutex causes **undefined behaviour**.

---

### Q8. Why must you always check return codes from pthread functions?

**A:** pthread functions can **fail silently** — they return a non-zero error code on failure but do not crash the program or print an error. If you ignore the return code, your program may proceed with a broken lock or uninitialized resource, leading to subtle, hard-to-debug race conditions.

```c
// Always do this:
int rc = pthread_mutex_lock(&lock);
if (rc != 0) {
    fprintf(stderr, "Error: %s\n", strerror(rc));
    exit(1);
}
```

---

### Q9. What is a condition variable and when do you use it?

**A:** A **condition variable** allows one thread to wait (sleep) until another thread signals that a condition has changed. Use it when:
- Thread A must wait for Thread B to finish some work before proceeding
- You want to avoid wasteful busy-waiting (spin loops)

Core routines:
```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

`pthread_cond_wait` puts the thread to sleep AND **atomically releases the lock** while sleeping. It re-acquires the lock before returning.

---

### Q10. Show the correct pattern for using condition variables.

**A:**
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER;
int ready = 0;

// WAITER
void *waiter(void *arg) {
    pthread_mutex_lock(&lock);
    while (ready == 0)               // WHILE not IF
        pthread_cond_wait(&cond, &lock);
    // safe to proceed
    pthread_mutex_unlock(&lock);
    return NULL;
}

// SIGNALER
void *signaler(void *arg) {
    pthread_mutex_lock(&lock);
    ready = 1;                       // update shared state
    pthread_cond_signal(&cond);      // wake one waiter
    pthread_mutex_unlock(&lock);
    return NULL;
}
```

**Key rules:**
1. Always hold the lock when calling `wait` or `signal`
2. Always use `while` (not `if`) to check the condition — guards against **spurious wakeups**
3. Always update shared state before signalling

---

### Q11. Why use `while` instead of `if` when waiting on a condition variable?

**A:** Because of **spurious wakeups** — some pthread implementations can wake a thread even when the condition hasn't actually changed. Using `if` means the thread proceeds without re-checking, potentially acting on a false assumption. Using `while` re-checks the condition after every wakeup:

```c
// WRONG - vulnerable to spurious wakeups
if (ready == 0)
    pthread_cond_wait(&cond, &lock);

// CORRECT - safe
while (ready == 0)
    pthread_cond_wait(&cond, &lock);
```

---

### Q12. Why should you NOT use a simple flag/spin-wait instead of condition variables?

**A:** Using a spin-wait like `while (ready == 0);` is bad because:
1. **Wastes CPU** — the waiting thread burns 100% of a CPU core doing nothing useful
2. **Bug-prone** — research shows roughly **half of ad hoc spin-based synchronization** attempts contain bugs
3. **No atomicity** — there is a race between checking the flag and acting on it without a lock

Condition variables solve all three problems: the waiting thread **sleeps** (zero CPU), the check-and-sleep is **atomic** with the lock, and the pattern is well-defined.

---

### Q13. How do you compile a C program that uses pthreads?

**A:**
```bash
gcc -o program program.c -Wall -pthread
```

| Part | Purpose |
|---|---|
| `-pthread` | Links pthreads library AND sets required compile-time macros |
| `-Wall` | Enable all warnings |

Use `-pthread`, **not** `-lpthread`. The `-lpthread` flag only links the library but misses important macro definitions needed for thread-safe code.

---

### Q14. What is `pthread_mutex_trylock()` and when is it useful?

**A:**
```c
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```
Attempts to acquire the lock but **returns immediately** with an error code (`EBUSY`) if the lock is already held — it does NOT block.

Useful when:
- You want to try acquiring a lock but fall back to doing something else if unavailable
- Implementing lock-ordering or deadlock avoidance strategies
- Building non-blocking data structures

---

### Q15. What are the Thread API guidelines to follow for safe concurrent programming?

**A:** From the OSTEP Thread API Guidelines (ASIDE):

| Guideline | Why |
|---|---|
| **Keep it simple** | Complex thread interactions are the main source of bugs |
| **Initialize everything** | Always initialize locks and condition variables before use |
| **Check return codes** | Pthread calls fail silently — always verify |
| **Be careful with arguments** | Never pass stack pointers between threads |
| **Each thread has its own stack** | Share data via heap or global memory only |
| **Use condition variables** | Never use simple flags/spin-waits for signaling |
| **Use documentation** | Run `man pthread_create` etc. for full details |

---

## Quick Reference: All Key pthread Functions

| Function | Purpose |
|---|---|
| `pthread_create()` | Create a new thread |
| `pthread_join()` | Wait for a thread to finish |
| `pthread_detach()` | Mark thread as detached (auto-cleanup, no join needed) |
| `pthread_mutex_lock()` | Acquire a mutex (block if held) |
| `pthread_mutex_unlock()` | Release a mutex |
| `pthread_mutex_trylock()` | Try to acquire mutex (non-blocking) |
| `pthread_mutex_init()` | Initialize a mutex dynamically |
| `pthread_mutex_destroy()` | Destroy a mutex |
| `pthread_cond_wait()` | Sleep and release lock atomically |
| `pthread_cond_signal()` | Wake one waiting thread |
| `pthread_cond_broadcast()` | Wake all waiting threads |
| `pthread_cond_init()` | Initialize a condition variable dynamically |
| `pthread_cond_destroy()` | Destroy a condition variable |

---

*Source: Educative.io — Operating Systems: Virtualization, Concurrency & Persistence — Chapter 21*
