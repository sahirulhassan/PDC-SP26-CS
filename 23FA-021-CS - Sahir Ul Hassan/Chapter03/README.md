# Chapter 03 - Process-Based Parallelism and Inter-Process Communication

## What I learned overall

- Multiprocessing gives true parallel execution with separate memory spaces, so data sharing must be explicit.
- Process lifecycle control (`start`, `join`, `terminate`, `daemon`) changes both correctness and program shutdown behavior.
- Process coordination primitives like `Barrier` and `Lock` are just as important as in threading.
- IPC tools (`Queue`, `Pipe`) are the practical way to move data safely between processes.

## Concepts from each file

### `myFunc.py`
This file defines a reusable worker function:

- Prints which process invoked the function.
- Loops from `0` to `i-1` to show per-process output.

My takeaway: keeping worker logic in a dedicated function/module makes process experiments cleaner and reusable.

### `spawning_processes.py`
This is the basic process creation example:

- Creates a `multiprocessing.Process` for each value in a loop.
- Passes arguments to the target function with `args=(i,)`.
- Uses `start()` and `join()` for each process.

My observation: because each process is joined inside the loop, this behaves mostly sequentially; it clearly shows how `join()` placement affects parallelism.

### `spawning_processes_namespace.py`
This repeats process spawning but imports the target from another file:

- Uses `from myFunc import myFunc`.
- Demonstrates modular organization across files.

Conclusion: separating worker code from launcher code helps maintainability, especially as multiprocessing projects grow.

### `naming_processes.py`
This demonstrates process naming and lifecycle tracing:

- One process is created with a custom name.
- Another process uses the default generated name.
- Both print start/exit messages and are joined.

Takeaway: named processes are much easier to debug when many workers are running.

### `run_background_processes.py`
This introduces daemon vs non-daemon process behavior:

- Creates `background_process` with `daemon = True`.
- Creates `NO_background_process` with `daemon = False`.
- Starts both without explicit joins.
- The main program exits immediately after starting, which causes the daemon process to be killed, while the non-daemon process continues until completion.

My learning: daemon processes are background helpers that may stop when the main program exits, so they are not suitable for critical work that must always complete.

### `run_background_processes_no_daemons.py`
This variation runs both processes as non-daemon:

- Sets both `.daemon` flags to `False`.
- Uses the same worker function and start pattern as the daemon example.

Takeaway: comparing this file with the previous one helped me understand how daemon mode directly changes shutdown semantics.

### `process_in_subclass.py`
This file shows subclassing `multiprocessing.Process`:

- Defines `MyProcess` and overrides `run()`.
- Instantiates and starts multiple custom process objects.

Conclusion: subclassing is useful when process behavior/state should be encapsulated in an object-oriented design.

### `process_pool.py`
This demonstrates high-level parallel mapping with a process pool:

- Creates `Pool(processes=4)`.
- Uses `pool.map(function_square, inputs)` to parallelize work.
- Closes and joins the pool after completion.
- Note: the `close()` method prevents new tasks from being submitted, and `join()` waits for all worker processes to 
  finish. You cannot use `join()` without first calling `close()`, as it will raise a `ValueError` because the pool is still accepting tasks.
- better to use a context manager like this:
``` python
with multiprocessing.Pool(processes=4) as pool:
    pool_outputs = pool.map(function_square, inputs)
```

My takeaway: `Pool.map` is a concise and scalable pattern for applying one function over many inputs.

### `killing_processes.py`
This file demonstrates forceful process termination:

- Starts a long-running worker.
- Calls `terminate()` before normal completion.
- Shows process state with `is_alive()` and prints `exitcode`.
- After calling `terminate()`, the process is still alive because the operation is asynchronous, but it will be killed soon after.
- Once we wait by `join()` the `isAlive()` method returns false.
- The `foo()` fn never prints anything as it never gets scheduled, adding a delay can help.

Learning point: `terminate()` is useful for control/recovery, but it can interrupt work abruptly, so graceful shutdown is usually safer when possible.

### `processes_barrier.py`
This demonstrates process synchronization with a barrier:

- Two processes wait on `Barrier(2)` before proceeding.
- A `Lock` serializes prints so output stays readable.
- Two additional processes run without barrier for comparison.

Takeaway: barriers are great for phase alignment across processes, while locks keep shared output predictable.

### `communicating_with_queue.py`
This is a producer-consumer model using `multiprocessing.Queue`:

- Producer process generates random items and pushes them with `put()`.
- Consumer process pops with `get()`.
- Queue provides safe cross-process communication.

My observation: this demonstrates IPC clearly, and also highlights that consumer stop conditions need careful design to avoid early exit or waiting issues.

### `communicating_with_pipe.py`
This demonstrates `multiprocessing.Pipe` for direct process-to-process communication:

- First process sends values `0..9` into a pipe.
- Second process reads those values, squares them, and sends to a second pipe.
- Main process receives and prints transformed results until EOF.
- Closing the sending end of the pipe is crucial to signal the receiver that no more data is coming (`EOFError`).

My conclusion: `Pipe` is a lightweight IPC option for point-to-point communication, while `Queue` is often easier for multi-producer/multi-consumer patterns.

## Personal chapter conclusions

- I now understand that multiprocessing is not just "threading with another API"; memory isolation changes everything.
- Process management decisions (`join`, daemon mode, termination) directly affect reliability and output behavior.
- IPC tools are central in process-based designs because normal Python objects are not automatically shared.
- Synchronization patterns (barrier, lock) still matter in multiprocessing when phases and shared resources must be coordinated.

