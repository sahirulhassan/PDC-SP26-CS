# Chapter 02 - Threading and Synchronization Primitives

## What I learned overall

- Creating threads is easy; coordinating them safely is the hard part.
- Different synchronization primitives solve different coordination problems.
- Thread lifecycle (`start`, `run`, `join`) and naming make debugging much easier.
- Some examples reveal common pitfalls (deadlocks, starvation, infinite waits), which were very useful learning points.

## Concepts from each file

### `Thread_definition.py`
This file shows the basic thread creation pattern:

- Define worker function.
- Create `threading.Thread(target=..., args=...)` objects.
- Start and join threads.

My observation: because each thread is joined right after starting inside the loop, execution becomes effectively sequential. That helped me understand how `join()` placement affects concurrency.

### `Thread_determine.py`
This demonstrates thread naming and lifecycle logs:

- Three thread functions with start/exiting messages.
- Explicit names (`function_A`, `function_B`, `function_C`).
- Simulated work using `sleep(2)`.

Takeaway: naming threads is very practical for tracing execution order and debugging.

### `Thread_name_and_processes.py`
This extends thread subclassing basics:

- Custom class derived from `Thread`.
- Thread names attached as object state.
- `run()` overridden for custom work.

My conclusion: subclassing is useful when worker logic carries state and needs a clean object-oriented structure.

### `MyThreadClass.py`
This is a larger thread-subclass example:

- Creates many thread objects with random durations.
- Demonstrates concurrent starts and final joins.
- Reports total execution time.

I learned that with no lock, outputs interleave naturally, and total time tends to be bounded by the slowest thread rather than sum of all durations.

### `MyThreadClass_lock.py`
This introduces mutual exclusion using `threading.Lock`:

- Global lock is acquired at thread start and released at end.
- Entire critical section includes `sleep(duration)`.

My understanding: this guarantees one-thread-at-a-time behavior, but it also serializes almost all work, which reduces parallel benefit.

### `MyThreadClass_lock_2.py`
This is a refinement of lock scope:

- Lock is acquired only around the critical print section.
- Lock is released before `sleep(duration)`.

This taught me a key design rule: keep critical sections as small as possible. It protects shared output while preserving more concurrency.

### `Rlock.py`
This explains reentrant locking (`threading.RLock`):

- `Box` methods call each other while acquiring the same lock.
- `RLock` allows the same thread to lock multiple times safely.
- Separate adder/remover threads update shared state.

Takeaway: `RLock` is useful for nested synchronized calls where a normal `Lock` could deadlock.

### `Semaphore.py`
This demonstrates signaling availability with `threading.Semaphore`:

- Consumer waits on `acquire()`.
- Producer generates item and calls `release()`.
- Initial semaphore count is `0`, so consumer blocks until produced.

My conclusion: semaphores are great for controlling access/counting permits, especially in producer-consumer style flows.

### `Event.py`
This introduces event-based signaling:

- Producer appends items and toggles `event.set()` / `event.clear()`.
- Consumer waits with `event.wait()` before popping items.

Learning point: events are simple for "something happened" notifications. But I also noticed the consumer loop is infinite, while producer runs only 5 times, so program completion needs extra shutdown logic in real applications.

### `Condition.py`
This shows condition variables for coordinated waiting:

- Shared list acts as bounded buffer.
- Producer waits when buffer reaches size 10.
- Consumer waits when buffer is empty.
- `condition.wait()` and `condition.notify()` handle handoff.

My takeaway: `Condition` combines a lock + wait/notify protocol and is very useful when waiting depends on a shared-state predicate.

### `Barrier.py`
This demonstrates phase synchronization:

- Multiple runner threads sleep random times.
- Each thread waits at a common barrier.
- All proceed only when everyone arrives.

I understood barriers as a rendezvous point: useful when threads must align before moving to the next phase.

### `Threading_with_queue.py`
This uses `queue.Queue` for thread-safe producer-consumer exchange:

- Producer pushes items with `put()`.
- Consumers pull with `get()` and acknowledge with `task_done()`.
- Queue handles internal synchronization automatically.

My conclusion: when possible, I should prefer `Queue` over managing shared lists with manual locks because it is safer and cleaner.