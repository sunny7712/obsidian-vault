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