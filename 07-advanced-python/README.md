# Module 07: Advanced Python & Metaprogramming

Welcome to advanced Python engineering. In this module, you will step beyond day-to-day scripting into the inner mechanics of the Python runtime: the iterator protocol, generators, custom decorators, context managers, the descriptor protocol, metaclasses, and high-performance standard library utilities.

---

## 📖 Table of Contents
1. [The Iterator Protocol (`__iter__` & `__next__`)](#1-the-iterator-protocol-__iter__--__next__)
2. [Generators & Pipeline Stream Processing](#2-generators--pipeline-stream-processing)
3. [Decorators in Depth](#3-decorators-in-depth)
4. [Context Managers & `contextlib`](#4-context-managers--contextlib)
5. [Power Collections (`collections` module)](#5-power-collections-collections-module)
6. [Functional Superpowers: `itertools` & `functools`](#6-functional-superpowers-itertools--functools)
7. [The Descriptor Protocol](#7-the-descriptor-protocol)
8. [Metaclasses & Class Construction](#8-metaclasses--class-construction)
9. [Practice Questions](#9-practice-questions)
10. [Hands-On Module Activity](#10-hands-on-module-activity)

---

## 1. The Iterator Protocol (`__iter__` & `__next__`)

An **iterable** is any object that can return an iterator (implements `__iter__`).
An **iterator** is an object representing a stream of data that implements `__next__` and returns `self` from `__iter__`.

```python
class CountDown:
    def __init__(self, start: int):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self.current <= 0:
            raise StopIteration
        val = self.current
        self.current -= 1
        return val

for num in CountDown(3):
    print(num) # 3, 2, 1
```

---

## 2. Generators & Pipeline Stream Processing

Generators allow writing iterators using simple function syntax with the `yield` keyword. Execution freezes at `yield` and resumes on the next request.

```python
def infinite_fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Data streaming pipeline
def read_log_lines(filepath):
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()

def filter_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line

# Chained generator pipeline processes gigabytes of logs in O(1) memory!
```

### Delegating with `yield from`
`yield from` delegates iteration directly to a sub-generator:
```python
def chain_generators(gen_a, gen_b):
    yield from gen_a
    yield from gen_b
```

---

## 3. Decorators in Depth

A decorator is a callable that takes a function (or class) as an argument and returns a replacement callable.

### Preserving Metadata with `@functools.wraps`
Without `@wraps`, the decorated function loses its `__name__` and `__doc__`.

```python
import functools
import time

def timing_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        duration = time.perf_counter() - start
        print(f"⏱️ [{func.__name__}] executed in {duration:.6f}s")
        return result
    return wrapper

@timing_decorator
def compute_heavy_task():
    """Performs CPU computations."""
    return sum(i * i for i in range(1_000_000))

compute_heavy_task()
print(compute_heavy_task.__name__) # 'compute_heavy_task' (preserved!)
```

### Decorators with Arguments (Decorator Factory)
To pass arguments to a decorator, wrap it in an outer factory function:

```python
def repeat(num_times: int):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(num_times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(num_times=3)
def ping():
    print("Ping!")

ping() # Prints "Ping!" 3 times
```

---

## 4. Context Managers & `contextlib`

Context managers manage resource lifecycles (`setup` and `teardown`).

### Class-based Context Manager
```python
class DatabaseSession:
    def __enter__(self):
        print("🔌 Opening database connection...")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            print(f"⚠️ Transaction rolled back due to: {exc_val}")
        else:
            print("💾 Transaction committed successfully.")
        print("🔌 Connection closed.")
        return False # Propagate exception if any

with DatabaseSession():
    print("Executing SQL queries...")
```

### Generator-based Context Manager with `@contextmanager`
```python
from contextlib import contextmanager
import time

@contextmanager
def execution_timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"[{label}] Elapsed: {time.perf_counter() - start:.4f}s")

with execution_timer("File Processing"):
    time.sleep(0.1)
```

---

## 5. Power Collections (`collections` module)

```python
from collections import defaultdict, Counter, deque, namedtuple

# 1. defaultdict: Automatically initializes missing keys
graph = defaultdict(list)
graph["A"].append("B") # No KeyError!

# 2. Counter: Frequency multiset
inventory = Counter(["apple", "banana", "apple", "apple"])
print(inventory.most_common(1)) # [('apple', 3)]

# 3. deque: Double-ended queue with O(1) appends and pops on BOTH ends
queue = deque(maxlen=3)
queue.extend([1, 2, 3])
queue.append(4) # Evicts 1 from left: deque([2, 3, 4], maxlen=3)

# 4. namedtuple: Lightweight object with named fields
Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
print(p.x, p.y)
```

---

## 6. Functional Superpowers: `itertools` & `functools`

```python
import itertools
import functools

# itertools.chain: Flatten sequence of iterables
combined = list(itertools.chain([1, 2], [3, 4], [5])) # [1, 2, 3, 4, 5]

# itertools.groupby: Group consecutive keys
data = [("fruit", "apple"), ("fruit", "banana"), ("veg", "carrot")]
for k, group in itertools.groupby(data, key=lambda x: x[0]):
    print(k, list(group))

# functools.lru_cache: Built-in memoization
@functools.lru_cache(maxsize=128)
def expensive_calculation(n: int) -> int:
    time.sleep(0.01)
    return n * 42

# functools.partial: Freeze partial arguments
base_two = functools.partial(int, base=2)
print(base_two("1010")) # 10
```

---

## 7. The Descriptor Protocol

Descriptors power Python's internal attribute access (`@property`, `@classmethod`, `__slots__`). A descriptor defines `__get__`, `__set__`, or `__delete__`.

```python
class NonNegative:
    """Descriptor that enforces positive numeric values."""
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name, 0)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"{self.name} cannot be negative! Got: {value}")
        instance.__dict__[self.name] = value

class Product:
    price = NonNegative()
    stock = NonNegative()

    def __init__(self, price: float, stock: int):
        self.price = price
        self.stock = stock

item = Product(19.99, 10)
# item.price = -5 # Raises ValueError!
```

---

## 8. Metaclasses & Class Construction

A metaclass is a "class that creates classes". In Python, `type` is the default metaclass.

```python
class EnforceUppercaseMethods(type):
    """Metaclass that ensures all custom methods are uppercase."""
    def __new__(mcs, name, bases, attrs):
        for attr_name, attr_val in attrs.items():
            if callable(attr_val) and not attr_name.startswith("__"):
                if not attr_name.isupper():
                    raise TypeError(f"Method '{attr_name}' must be UPPERCASE in class '{name}'")
        return super().__new__(mcs, name, bases, attrs)

class StrictService(metaclass=EnforceUppercaseMethods):
    def EXECUTE(self):
        print("Executing...")

    # def run(self): # Would raise TypeError at import/definition time!
```

---

## 9. Practice Questions

### Conceptual Questions

**Q1: What is the difference between an iterable and an iterator?**
> **Answer:** An iterable implements `__iter__()` which returns an iterator. An iterator implements `__next__()` (returning values until `StopIteration`) as well as `__iter__()` (returning `self`).

**Q2: Why must `@functools.wraps(func)` be used inside custom decorators?**
> **Answer:** It copies the original function's attributes (like `__name__`, `__doc__`, and `__annotations__`) to the inner wrapper function, preventing introspective tools, debuggers, and doc generators from seeing the wrapper's generic name.

**Q3: How do non-data descriptors differ from data descriptors?**
> **Answer:** A data descriptor implements `__set__` (or `__delete__`). A non-data descriptor implements only `__get__` (e.g. standard methods). Data descriptors take precedence over instance dictionary (`instance.__dict__`) lookups, while non-data descriptors do not.

**Q4: What is the advantage of `collections.deque` over a standard `list` for FIFO queue operations?**
> **Answer:** `deque.popleft()` and `deque.appendleft()` are $O(1)$ operations. In contrast, `list.pop(0)` and `list.insert(0, val)` are $O(N)$ operations because all subsequent elements must be shifted in memory.

**Q5: When does a metaclass's `__new__` method execute?**
> **Answer:** It executes at class definition time (when Python finishes parsing the class block), BEFORE any instances of that class are created.

---

### Coding Challenges

#### Challenge 1: Custom Rate-Limiter Decorator
Create a decorator `@rate_limit(max_calls=3, period_sec=1.0)` that raises `RuntimeError` if invoked more than `max_calls` within the specified time window.

```python
# Solution 1:
import functools
import time

def rate_limit(max_calls: int, period_sec: float):
    def decorator(func):
        timestamps = []
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            # Remove timestamps outside window
            nonlocal timestamps
            timestamps = [t for t in timestamps if now - t < period_sec]
            if len(timestamps) >= max_calls:
                raise RuntimeError("Rate limit exceeded! Please slow down.")
            timestamps.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

#### Challenge 2: Generator Window Slicer
Implement `sliding_window(iterable, n)` yielding overlapping tuples of size $n$.
Example: `[1, 2, 3, 4]`, $n=3 \rightarrow `(1, 2, 3), (2, 3, 4)`$.

```python
# Solution 2:
from collections import deque

def sliding_window(iterable, n: int):
    it = iter(iterable)
    window = deque(itertools.islice(it, n), maxlen=n)
    if len(window) == n:
        yield tuple(window)
    for x in it:
        window.append(x)
        yield tuple(window)
```

#### Challenge 3: In-Memory File Buffer Context Manager
Implement a context manager that redirects standard output (`sys.stdout`) to an in-memory string stream and restores it upon exit.

```python
# Solution 3:
import io
import sys
from contextlib import contextmanager

@contextmanager
def capture_stdout():
    buffer = io.StringIO()
    old_stdout = sys.stdout
    sys.stdout = buffer
    try:
        yield buffer
    finally:
        sys.stdout = old_stdout
```

#### Challenge 4: Type-Enforcing Descriptor
Create a descriptor `Typed(expected_type)` that ensures assigned values match `expected_type`.

```python
# Solution 4:
class Typed:
    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None: return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"{self.name} must be of type {self.expected_type.__name__}")
        instance.__dict__[self.name] = value
```

#### Challenge 5: Auto-Registration Metaclass
Build a metaclass that registers every subclass into a global dictionary registry automatically.

```python
# Solution 5:
PLUGIN_REGISTRY = {}

class PluginMeta(type):
    def __new__(mcs, name, bases, attrs):
        cls = super().__new__(mcs, name, bases, attrs)
        if bases:  # Exclude base plugin class itself
            PLUGIN_REGISTRY[name] = cls
        return cls

class PluginBase(metaclass=PluginMeta):
    pass

class AudioPlugin(PluginBase): pass
class VideoPlugin(PluginBase): pass

print(PLUGIN_REGISTRY) # {'AudioPlugin': <class ...>, 'VideoPlugin': <class ...>}
```

---

## 10. Hands-On Module Activity

### 🎯 Project: Production-Grade API Middleware & Caching Suite

#### Objectives
Build an enterprise API toolset combining advanced Python features:
1. `@memoize(ttl_seconds)`: Caching decorator that expires cached results after a Time-To-Live.
2. `@audit_log(logger_file)`: Decorator logging execution arguments, timestamps, and execution durations to disk.
3. `@throttle(max_per_second)`: Sliding-window rate limiter preventing excessive request floods.
4. Integrate all 3 decorators onto simulated API handlers.

#### Implementation
Create a file named `middleware_suite.py`:

```python
# middleware_suite.py
"""
Production-Grade API Middleware & Caching Suite
Module 07 Hands-On Project
"""
import functools
import time
from collections import deque
from datetime import datetime

def memoize(ttl_seconds: float = 60.0):
    """Caches function results with an expiration Time-To-Live."""
    def decorator(func):
        cache = {} # key -> (result, timestamp)

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, frozenset(kwargs.items()))
            now = time.time()

            if key in cache:
                val, timestamp = cache[key]
                if now - timestamp < ttl_seconds:
                    print(f"⚡ [CACHE HIT] {func.__name__}{args}")
                    return val

            print(f"🔄 [CACHE MISS] Computing {func.__name__}{args}")
            result = func(*args, **kwargs)
            cache[key] = (result, now)
            return result

        wrapper.clear_cache = lambda: cache.clear()
        return wrapper
    return decorator

def throttle(calls_per_second: int):
    """Enforces an execution frequency cap."""
    def decorator(func):
        history = deque()

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            while history and now - history[0] > 1.0:
                history.popleft()

            if len(history) >= calls_per_second:
                raise RuntimeError(f"🚨 429 Too Many Requests: Rate limit of {calls_per_second}/sec reached.")

            history.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator

def audit_log():
    """Logs runtime execution metrics."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            iso_timestamp = datetime.now().isoformat()
            try:
                res = func(*args, **kwargs)
                duration = time.perf_counter() - start
                print(f"📝 [AUDIT {iso_timestamp}] {func.__name__} SUCCESS ({duration*1000:.2f}ms)")
                return res
            except Exception as err:
                duration = time.perf_counter() - start
                print(f"❌ [AUDIT {iso_timestamp}] {func.__name__} FAILED: {err} ({duration*1000:.2f}ms)")
                raise
        return wrapper
    return decorator

# --- Simulated API Endpoints ---

@throttle(calls_per_second=3)
@memoize(ttl_seconds=2.0)
@audit_log()
def get_user_profile(user_id: int):
    time.sleep(0.1) # Simulate database query latency
    return {"user_id": user_id, "username": f"developer_{user_id}", "status": "active"}

def main():
    print("=" * 65)
    print("      PRODUCTION API MIDDLEWARE & CACHING DEMO      ")
    print("=" * 65)

    print("\n1. Testing Cache Hit / Miss:")
    print("Call 1:", get_user_profile(101))
    print("Call 2 (Immediate):", get_user_profile(101)) # Should Hit Cache

    print("\n2. Waiting 2.1s for cache to expire...")
    time.sleep(2.1)
    print("Call 3 (Expired):", get_user_profile(101)) # Should Miss Cache

    print("\n3. Testing Rate Limiting (Throttle limit = 3 per sec):")
    try:
        for i in range(5):
            print(f"Request {i+1} result:", get_user_profile(200 + i))
    except RuntimeError as e:
        print(e)

    print("\nDemo completed.")

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python middleware_suite.py
```

---
**Next Step:** Proceed to [**Module 08: Concurrency & Asynchronous Programming**](../08-concurrency-and-async/README.md)!
