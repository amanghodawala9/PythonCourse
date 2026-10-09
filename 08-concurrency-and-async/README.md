# Module 08: Concurrency & Asynchronous Programming

Modern applications must handle high-throughput network requests, parallel calculations, and responsive I/O. In this module, you will master Python concurrency paradigms: **Multithreading**, **Multiprocessing**, and **Asyncio**, understand the **Global Interpreter Lock (GIL)**, and learn when to choose each tool.

---

## 📖 Table of Contents
1. [Concurrency vs Parallelism vs Asynchrony](#1-concurrency-vs-parallelism-vs-asynchrony)
2. [The Python Global Interpreter Lock (GIL)](#2-the-python-global-interpreter-lock-gil)
3. [Multithreading & Thread Pools (`threading`, `ThreadPoolExecutor`)](#3-multithreading--thread-pools-threading-threadpoolexecutor)
4. [Multiprocessing for CPU-Bound Tasks (`multiprocessing`, `ProcessPoolExecutor`)](#4-multiprocessing-for-cpu-bound-tasks-multiprocessing-processpoolexecutor)
5. [Asyncio Fundamentals: Event Loop, Coroutines & Tasks](#5-asyncio-fundamentals-event-loop-coroutines--tasks)
6. [Concurrent Async Execution: `asyncio.gather` vs `asyncio.TaskGroup`](#6-concurrent-async-execution-asynciogather-vs-asynciotaskgroup)
7. [Async Synchronization: Semaphores & Queues](#7-async-synchronization-semaphores--queues)
8. [Decision Framework: Which Concurrency Model to Use?](#8-decision-framework-which-concurrency-model-to-use)
9. [Practice Questions](#9-practice-questions)
10. [Hands-On Module Activity](#10-hands-on-module-activity)

---

## 1. Concurrency vs Parallelism vs Asynchrony

- **Concurrency**: Managing multiple computations at the same time (interleaved execution; e.g. juggling multiple balls with two hands).
- **Parallelism**: Executing multiple computations simultaneously on separate CPU cores (e.g. two people juggling side-by-side).
- **Asynchrony**: Cooperative, non-blocking single-threaded multitasking where an operation releases control while waiting for an external event (e.g. placing an order at a cafe and picking it up when your buzzer rings).

| Workload Type | Bottleneck | Ideal Solution |
|---------------|------------|----------------|
| **I/O-Bound** (Network, disk, DB queries) | Waiting on external devices | `asyncio` or `threading` |
| **CPU-Bound** (Encryption, ML, video processing) | CPU compute cycles | `multiprocessing` |

---

## 2. The Python Global Interpreter Lock (GIL)

In CPython, the **GIL** is a mutual exclusion lock that prevents multiple native threads from executing Python bytecode simultaneously.

### Why does the GIL exist?
CPython uses reference counting for garbage collection. Without the GIL, concurrent threads modifying reference counts would cause race conditions, memory corruption, and leaks.

### Implications:
- Multithreading in standard Python does **not** speed up CPU-bound code across multiple cores.
- Multithreading **is effective** for I/O-bound code because Python releases the GIL during operating system I/O calls (e.g., waiting for network packets or disk reads).
- *Note:* Python 3.13 introduces experimental free-threaded builds (`--disable-gil`).

---

## 3. Multithreading & Thread Pools (`threading`, `ThreadPoolExecutor`)

The `concurrent.futures` module provides a high-level API for thread pools:

```python
from concurrent.futures import ThreadPoolExecutor
import time

def download_asset(asset_id: int) -> str:
    print(f"Downloading asset {asset_id}...")
    time.sleep(1.0) # Simulating network latency
    return f"Asset {asset_id} downloaded"

# Manage a pool of worker threads
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(download_asset, range(1, 6)))

print("All downloads finished:", results)
```

### Thread Safety & Race Conditions
When multiple threads write to shared state, you must protect access using a `threading.Lock`:
```python
import threading

balance = 100
balance_lock = threading.Lock()

def safe_deposit(amount):
    global balance
    with balance_lock:
        balance += amount
```

---

## 4. Multiprocessing for CPU-Bound Tasks (`multiprocessing`, `ProcessPoolExecutor`)

Multiprocessing sidesteps the GIL completely by spawning separate OS processes, each with its own independent Python interpreter and memory space.

```python
from concurrent.futures import ProcessPoolExecutor
import math

def compute_heavy_factorials(n: int) -> int:
    return sum(math.factorial(i % 500) for i in range(n))

if __name__ == "__main__":
    workloads = [20000, 25000, 30000, 35000]
    with ProcessPoolExecutor() as executor:
        results = list(executor.map(compute_heavy_factorials, workloads))
    print("Computed parallel results:", len(results))
```

> ⚠️ **Important on Windows:** Always wrap multiprocessing execution code inside `if __name__ == "__main__":` to prevent infinite child process spawning.

---

## 5. Asyncio Fundamentals: Event Loop, Coroutines & Tasks

Asyncio is Python's built-in framework for cooperative multitasking using `async`/`await`.

### Key Concepts:
- **Coroutine**: A function defined with `async def` that can pause execution using `await`.
- **Event Loop**: The scheduler running in a single thread that manages and dispatches coroutines when events complete.
- **Task**: An event-loop managed wrapper around a coroutine scheduling it for immediate background execution.

```python
import asyncio

async def fetch_weather_report(city: str):
    print(f"Fetching weather for {city}...")
    await asyncio.sleep(1.5) # Non-blocking sleep!
    return f"{city}: 22°C, Clear Sky"

async def main():
    report = await fetch_weather_report("San Francisco")
    print(report)

asyncio.run(main())
```

---

## 6. Concurrent Async Execution: `asyncio.gather` vs `asyncio.TaskGroup`

### Modern `asyncio.TaskGroup` (Python 3.11+)
TaskGroup enforces structured concurrency: if any child task fails, all other sibling tasks in the group are cancelled cleanly.

```python
import asyncio

async def fetch_url(url: str, delay: float):
    await asyncio.sleep(delay)
    return f"Response from {url}"

async def main():
    # Structured concurrency with TaskGroup
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_url("https://api.github.com", 0.5))
        task2 = tg.create_task(fetch_url("https://httpbin.org", 0.3))

    # After exiting block, all tasks are guaranteed complete
    print(task1.result())
    print(task2.result())

asyncio.run(main())
```

### Legacy `asyncio.gather(*coros)`
Runs multiple coroutines concurrently and returns all results as a list:
```python
results = await asyncio.gather(
    fetch_url("url1", 0.2),
    fetch_url("url2", 0.4)
)
```

---

## 7. Async Synchronization: Semaphores & Queues

### `asyncio.Semaphore`
Throttles concurrent access to resources (e.g. preventing hitting an API with 1,000 simultaneous connections):

```python
semaphore = asyncio.Semaphore(5) # Allow at most 5 concurrent requests

async def throttled_fetch(url: str):
    async with semaphore:
        await asyncio.sleep(0.5)
        return f"Fetched {url}"
```

### `asyncio.Queue`
Producer-consumer messaging pipeline:
```python
queue = asyncio.Queue(maxsize=10)

async def producer():
    for item in range(5):
        await queue.put(item)

async def consumer():
    while True:
        item = await queue.get()
        print("Consumed:", item)
        queue.task_done()
```

---

## 8. Decision Framework: Which Concurrency Model to Use?

```
Is the workload CPU-Bound or I/O-Bound?
  ├── CPU-Bound (Heavy calculations, data transforms, rendering)
  │     └── Use MULTIPROCESSING (ProcessPoolExecutor)
  └── I/O-Bound (Web requests, DB queries, file streams)
        ├── Do third-party libraries support async/await?
        │     ├── YES ──> Use ASYNCIO (Highest scalability, low overhead)
        │     └── NO  ──> Use MULTITHREADING (ThreadPoolExecutor)
```

---

## 9. Practice Questions

### Conceptual Questions

**Q1: Why doesn't Python multithreading utilize multiple CPU cores for CPU-heavy tasks?**
> **Answer:** Because of the Global Interpreter Lock (GIL). The GIL enforces that only one thread can execute CPython bytecode at any given moment, limiting CPU-bound multithreaded programs to single-core throughput.

**Q2: What is the fundamental difference between `time.sleep()` and `asyncio.sleep()`?**
> **Answer:** `time.sleep()` blocks the entire operating system thread synchronously, preventing any other code from running on that thread. `asyncio.sleep()` yields execution back to the event loop, allowing other concurrent coroutines to run during the delay.

**Q3: What happens if an unhandled exception occurs inside one task within an `asyncio.TaskGroup`?**
> **Answer:** The `TaskGroup` catches the exception, cancels all other pending sibling tasks scheduled within the group, and raises an `ExceptionGroup` containing the failures.

**Q4: When should you prefer `multiprocessing` over `asyncio`?**
> **Answer:** Whenever tasks are CPU-bound (e.g., cryptographic hashing, image processing, matrix multiplication) rather than waiting on external network/disk I/O.

**Q5: What is a race condition and how is it prevented in multithreaded applications?**
> **Answer:** A race condition occurs when multiple threads concurrently read and write shared data without synchronization, causing unpredictable outcomes depending on thread scheduling. It is prevented using synchronization primitives like `threading.Lock` or `threading.RLock`.

---

### Coding Challenges

#### Challenge 1: Async Countdown Timer
Create an asynchronous timer that prints countdown updates concurrently for 3 separate timers (e.g. 3s, 5s, 2s).

```python
# Solution 1:
import asyncio

async def timer(name: str, seconds: int):
    for sec in range(seconds, 0, -1):
        print(f"[{name}] {sec} seconds remaining...")
        await asyncio.sleep(1)
    print(f"[{name}] ⏰ Timer Finished!")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(timer("Tea", 2))
        tg.create_task(timer("Pasta", 4))

asyncio.run(main())
```

#### Challenge 2: ThreadPool Image Processing Simulator
Simulate processing 10 images concurrently using a 4-thread `ThreadPoolExecutor`.

```python
# Solution 2:
from concurrent.futures import ThreadPoolExecutor
import time

def process_image(img_id: int):
    time.sleep(0.2)
    return f"Image_{img_id}.png compressed"

with ThreadPoolExecutor(max_workers=4) as pool:
    results = list(pool.map(process_image, range(1, 11)))
print("Finished:", len(results), "images.")
```

#### Challenge 3: Async Batch Rate-Limiter with Semaphore
Fetch 10 mock URLs allowing at most 3 requests concurrently.

```python
# Solution 3:
import asyncio

sem = asyncio.Semaphore(3)

async def limited_fetch(url_id: int):
    async with sem:
        print(f"Starting fetch {url_id}")
        await asyncio.sleep(0.5)
        print(f"Finished fetch {url_id}")
        return url_id

async def main():
    tasks = [limited_fetch(i) for i in range(10)]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

#### Challenge 4: Parallel Prime Range Checker with Multiprocessing
Distribute the calculation of prime count in ranges across multiple CPU processes.

```python
# Solution 4:
from concurrent.futures import ProcessPoolExecutor

def is_prime(n):
    if n < 2: return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0: return False
    return True

def count_primes(start, end):
    return sum(1 for n in range(start, end) if is_prime(n))

if __name__ == "__main__":
    ranges = [(1, 50000), (50000, 100000), (100000, 150000)]
    with ProcessPoolExecutor() as executor:
        counts = list(executor.map(lambda r: count_primes(*r), ranges))
    print("Total primes found:", sum(counts))
```

#### Challenge 5: Async Retry with Exponential Backoff
Create a coroutine `retry_async(coro_func, max_attempts=3)` that retries failed async calls with doubling backoff delays.

```python
# Solution 5:
import asyncio

async def retry_async(coro_func, max_attempts=3, base_delay=0.2):
    for attempt in range(1, max_attempts + 1):
        try:
            return await coro_func()
        except Exception as e:
            if attempt == max_attempts:
                raise
            delay = base_delay * (2 ** (attempt - 1))
            print(f"Attempt {attempt} failed: {e}. Retrying in {delay:.2f}s...")
            await asyncio.sleep(delay)
```

---

## 10. Hands-On Module Activity

### 🎯 Project: Concurrent Asynchronous Service Health Monitor

#### Objectives
Build an async system monitoring simulated distributed microservices:
1. Define a list of endpoints with simulated latencies and failure rates.
2. Monitor all services concurrently using `asyncio.TaskGroup`.
3. Enforce maximum connection concurrency via `asyncio.Semaphore`.
4. Implement automatic retries with timeouts using `asyncio.wait_for`.
5. Display live status logs and print an aggregated summary matrix upon completion.

#### Implementation
Create a file named `health_monitor.py`:

```python
# health_monitor.py
"""
Concurrent Asynchronous Service Health Monitor
Module 08 Hands-On Project
"""
import asyncio
import random
import time
from dataclasses import dataclass
from typing import Optional

@dataclass
class ServiceResult:
    endpoint: str
    status: str
    latency_ms: float
    error: Optional[str] = None

# Mock Services
SERVICES = [
    {"name": "Auth Service", "url": "https://auth.internal/health", "flaky": False},
    {"name": "Payment Gateway", "url": "https://pay.internal/health", "flaky": True},
    {"name": "Order Engine", "url": "https://order.internal/health", "flaky": False},
    {"name": "Notification API", "url": "https://notify.internal/health", "flaky": True},
    {"name": "Inventory DB", "url": "https://inv.internal/health", "flaky": False},
    {"name": "Analytics Service", "url": "https://analytics.internal/health", "flaky": True},
]

SEMAPHORE = asyncio.Semaphore(3) # Max 3 concurrent network requests

async def mock_http_ping(service: dict) -> ServiceResult:
    """Simulates an asynchronous HTTP health ping."""
    async with SEMAPHORE:
        start = time.perf_counter()
        
        # Simulated network latency between 100ms and 600ms
        delay = random.uniform(0.1, 0.6)
        await asyncio.sleep(delay)

        elapsed_ms = (time.perf_counter() - start) * 1000

        # Simulate intermittent network drop
        if service["flaky"] and random.random() < 0.35:
            raise ConnectionError(f"503 Service Unavailable / Connection Refused")

        return ServiceResult(
            endpoint=service["name"],
            status="HEALTHY 🟢",
            latency_ms=round(elapsed_ms, 2)
        )

async def check_service_with_retry(service: dict, max_retries: int = 2) -> ServiceResult:
    for attempt in range(1, max_retries + 1):
        try:
            # 500ms timeout per request
            result = await asyncio.wait_for(mock_http_ping(service), timeout=0.55)
            return result
        except (asyncio.TimeoutError, ConnectionError) as err:
            if attempt == max_retries:
                return ServiceResult(
                    endpoint=service["name"],
                    status="UNHEALTHY 🔴",
                    latency_ms=0.0,
                    error=str(err) or "Timeout"
                )
            await asyncio.sleep(0.1 * attempt) # Backoff before retry

async def main():
    print("=" * 65)
    print("      CONCURRENT ASYNC HEALTH MONITOR INITIALIZED      ")
    print("=" * 65)
    print(f"Pinging {len(SERVICES)} microservices (Max Concurrency: 3)...\n")

    start_all = time.perf_counter()
    results = []

    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(check_service_with_retry(s)) for s in SERVICES]

    results = [t.result() for t in tasks]
    total_time = time.perf_counter() - start_all

    # Summary Report
    print("-" * 65)
    print(f"{'Service Endpoint':<24} {'Status':<15} {'Latency':<12} {'Details'}")
    print("-" * 65)
    for r in results:
        details = r.error if r.error else f"{r.latency_ms} ms"
        print(f"{r.endpoint:<24} {r.status:<15} {details}")

    print("=" * 65)
    healthy_count = sum(1 for r in results if "HEALTHY" in r.status)
    print(f"Summary: {healthy_count}/{len(SERVICES)} Services Operational.")
    print(f"Total Sweep Duration: {total_time:.2f} seconds (Non-blocking Concurrent).")
    print("=" * 65)

if __name__ == "__main__":
    asyncio.run(main())
```

Run and test your program:
```bash
python health_monitor.py
```

---
**Next Step:** Proceed to [**Module 09: Tooling, Testing & Packaging**](../09-tooling-testing-packaging/README.md)!
