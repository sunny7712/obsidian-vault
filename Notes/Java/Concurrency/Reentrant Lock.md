`ReentrantLock` in Java, found within the `java.util.concurrent.locks` package, is an explicit, reentrant mutual exclusion lock. It provides more flexible control over thread synchronization compared to the implicit `synchronized` keyword.

Key characteristics of ReentrantLock:

- **Reentrancy:** 
    A thread that already holds a `ReentrantLock` can acquire it again without blocking itself. This is crucial for avoiding self-deadlocks in methods that might call other synchronized methods within the same object. The lock maintains a "hold count" for the owning thread, incrementing it with each re-acquisition and decrementing it with each release. The lock is only fully released when the hold count reaches zero.
- **Explicit Locking:** 
    Unlike `synchronized` blocks, where locking and unlocking are handled implicitly, `ReentrantLock` requires explicit calls to `lock()` to acquire the lock and `unlock()` to release it. This grants finer-grained control over when and how locks are obtained and released.
- **Flexibility and Features:**
    - **Fairness Policy:** `ReentrantLock` can be constructed with a fairness policy (e.g., `new ReentrantLock(true)`). A fair lock grants access to the longest-waiting thread, preventing potential thread starvation.
    - `tryLock()`: This method allows a thread to attempt to acquire the lock without blocking. It can also be used with a timeout (`tryLock(long timeout, TimeUnit unit)`) to limit the waiting time.
    - `lockInterruptibly()`: This method allows a thread waiting for the lock to be interrupted, which is not possible with `synchronized`.
    - **Multiple Condition Variables:** `ReentrantLock` supports the creation of multiple `Condition` objects (using `newCondition()`), enabling more sophisticated thread communication and signaling compared to the single wait-set of `synchronized`.
    

Example Usage:

Java

```java
import java.util.concurrent.locks.ReentrantLock;

public class SharedResource {
    private final ReentrantLock lock = new ReentrantLock();
    private int counter = 0;

    public void incrementCounter() {
        lock.lock(); // Acquire the lock
        try {
            counter++;
            System.out.println(Thread.currentThread().getName() + ": Counter = " + counter);
        } finally {
            lock.unlock(); // Release the lock in a finally block to ensure it's always released
        }
    }

    public int getCounter() {
        lock.lock();
        try {
            return counter;
        } finally {
            lock.unlock();
        }
    }
}

```

## Key differences: `synchronized` vs `ReentrantLock`

Both `synchronized` (intrinsic monitor) and `java.util.concurrent.locks.ReentrantLock` are reentrant in Java — they allow a thread to re-acquire the same lock. Differences lie in features and control:

**1. Acquisition / release style**
- `synchronized`: implicit. JVM acquires on entry and releases automatically on block/method exit (even on exception).
- `ReentrantLock`: explicit. You call `lock()` and **must** call `unlock()` (usually in a `finally` block).

**2. Interruptibility**
- `synchronized`: acquisition cannot be interrupted.
- `ReentrantLock`: supports `lockInterruptibly()` — thread can be interrupted while waiting for the lock.

**3. Try / timed attempts**
- `synchronized`: no `tryLock` equivalent.
- `ReentrantLock`: provides `tryLock()` and `tryLock(long timeout, TimeUnit)` so you can avoid blocking indefinitely.

**4. Fairness**

- `synchronized`: fairness semantics are JVM-dependent and generally not selectable.
- `ReentrantLock`: can be created fair (`new ReentrantLock(true)`) so longest-waiting thread gets the lock next (at cost of throughput).

**5. Condition variables**
- `synchronized`: one wait-set per object, use `wait()`/`notify()`/`notifyAll()`.
- `ReentrantLock`: supports one or more `Condition` objects (`lock.newCondition()`), allowing multiple independent wait-sets and finer signaling (`await()`/`signal()`/`signalAll()`).

**6. Diagnostics & introspection**
- `synchronized`: limited introspection (you can use `Thread.holdsLock(obj)`).
- `ReentrantLock`: richer API (`isLocked()`, `isHeldByCurrentThread()`, `getHoldCount()`, `hasQueuedThreads()`, etc.).

**7. Performance & JVM behavior**
- Historically `ReentrantLock` could outperform `synchronized` under some contention patterns; modern JVMs have optimized `synchronized` a lot (biased locking, lock coarsening, etc.), so differences are smaller. (Use microbenchmarks for your specific workload.)

**8. Exception-safety**
- `synchronized`: safer by default (release happens automatically).
- `ReentrantLock`: you must always `unlock()` in a `finally` block to avoid leaving the lock held.

# When to use which
- Use `synchronized` when your locking needs are simple (method/block-level locking) and you want correctness with minimal boilerplate.
- Use `ReentrantLock` when you need:
    - `tryLock()` or timed waits,
    - `lockInterruptibly()` to respond to interrupts,
    - multiple `Condition`s for more complex wait/notify patterns,
    - fairness guarantees, or
    - diagnostic/monitoring methods.