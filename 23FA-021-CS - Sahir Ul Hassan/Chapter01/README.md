# Chapter 01 - Python Basics to Parallel Execution

## What I learned overall

- Python basics are not separate from concurrency; they are the foundation for writing correct threaded/process code.
- Measuring time (`time.time()`) is essential before making claims about "faster" code.
- Threading and multiprocessing have different tradeoffs, especially when sharing data.
- Small mistakes in thread/process setup can completely change behavior, so careful API usage matters.

## Concepts from each file

### `flow.py`
This file helped me revise core control flow:

- `if/elif/else` for branching decisions.
- `for` loop to iterate over a list and accumulate values.
- `while` loop for repeated work with an explicit counter.

My conclusion: even in advanced topics like concurrency, most logic still comes down to these constructs.

### `dir.py`
This repeats and reinforces two ideas:

- Classifying a number as positive/zero/negative using conditionals.
- Summing a list using iteration.

I see this as deliberate practice. Repetition made me faster at reading and predicting code behavior.

### `lists.py`
This file introduces Python collections and mutability:

- Lists can hold mixed types and can be updated by index.
- Dictionaries map keys to values and support key updates.
- Tuples are fixed-size ordered collections.
- Functions are first-class objects (`myfunc = len`) and can be called through variables.

My takeaway: choosing the right container directly affects code clarity and thread-safety later.

### `classes.py`
This file taught me object-oriented basics and class-vs-instance behavior:

- Defining classes with `__init__` and methods.
- Difference between class variables (`common`) and instance attributes.
- Overriding behavior in subclasses (`AnotherClass(Myclass)`).
- Dynamically adding attributes to an object (`instance.test = 10`).

Important realization: class attributes are shared defaults, while instance attributes are per-object. That distinction is very relevant for shared state in concurrent programs.

### `file.py` and `test.txt`
This is a basic file I/O cycle:

- Open file in write mode (`'w'`) and write lines.
- Reopen in read mode and load full content.
- Explicitly close files.

`test.txt` is the result artifact of these operations.

My conclusion: file handling is straightforward, but in real concurrent programs, reading/writing the same file from multiple workers would require synchronization.

### `do_something.py`
This is a reusable worker function:

- Generates `count` random numbers.
- Appends them to a provided list.

It is used by serial, thread, and process benchmark files. I understood this as a shared workload so execution models can be compared fairly.

### `serial_test.py`
This is the baseline timing experiment:

- Runs `do_something()` repeatedly in a normal loop (`n_exec = 10`).
- Measures total runtime.

My conclusion: always keep a serial baseline; otherwise parallel timing numbers are hard to interpret.

### `multithreading_test.py`
This introduces thread-based parallel structure:

- Creates multiple `threading.Thread` jobs.
- Starts and joins each thread.
- Reports elapsed time.

What I noticed while learning: the `Thread` constructor is currently used as `target=do_something(size, out_list)`, which calls the function immediately instead of passing it as a callable target. The common pattern is:

```python
threading.Thread(target=do_something, args=(size, out_list))
```

This was a good lesson that API details can silently break intended concurrency.

### `multiprocessing_test.py`
This demonstrates process-based parallelism:

- Creates multiple `multiprocessing.Process` workers.
- Starts and joins all processes.
- Uses the `if __name__ == "__main__":` guard (especially important on Windows).

My understanding: processes can bypass Python's GIL for CPU-heavy tasks, but each process has its own memory space, so shared normal lists do not automatically combine results.

### `thread_and_processes.py`
This file combines multiple ideas in one script:

- A serial section exists (currently commented).
- A threading section and a multiprocessing section run the same workload.
- Prints timing comparisons.

Again, I observed the same threading target pattern issue (`target=do_something(size, out_list)`), which helps explain why careful thread construction matters.

## Personal chapter conclusions

- I now see concurrency as a correctness problem first, then a speed problem.
- Serial, threading, and multiprocessing must be compared with equal workloads and correct setup.
- Shared data behaves very differently between threads and processes.
- Clean basics (loops, data structures, classes) make advanced code much easier to reason about.

