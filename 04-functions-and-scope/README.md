# Module 04: Functions, Scope & Functional Concepts

Functions allow you to write clean, reusable, and maintainable software. In this module, you will master parameter passing, arbitrary arguments, the LEGB scope hierarchy, first-class functions, lambdas, functional primitives, closures, and modern type hints.

---

## 📖 Table of Contents
1. [Function Anatomy & The `return` Statement](#1-function-anatomy--the-return-statement)
2. [Arguments & Default Parameters (The Mutable Trap)](#2-arguments--default-parameters-the-mutable-trap)
3. [`*args` and `**kwargs`](#3-args-and-kwargs)
4. [Positional-Only (`/`) & Keyword-Only (`*`) Parameters](#4-positional-only--and-keyword-only--parameters)
5. [The LEGB Scope Hierarchy](#5-the-legb-scope-hierarchy)
6. [`global` and `nonlocal` Keywords](#6-global-and-nonlocal-keywords)
7. [First-Class Functions & Closures](#7-first-class-functions--closures)
8. [Anonymous Functions (`lambda`)](#8-anonymous-functions-lambda)
9. [Functional Primitives: `map`, `filter`, `reduce`](#9-functional-primitives-map-filter-reduce)
10. [Modern Python Type Annotations](#10-modern-python-type-annotations)
11. [Recursion Fundamentals](#11-recursion-fundamentals)
12. [Practice Questions](#12-practice-questions)
13. [Hands-On Module Activity](#13-hands-on-module-activity)

---

## 1. Function Anatomy & The `return` Statement

A function is declared using the `def` keyword:

```python
def calculate_rectangle_area(width: float, height: float) -> float:
    """Compute and return the area of a rectangle."""
    return width * height

# Calling the function
area = calculate_rectangle_area(5.5, 4.0)
print(f"Area: {area}")
```

> **Note:** A function without an explicit `return` statement returns `None` by default.

---

## 2. Arguments & Default Parameters (The Mutable Trap)

Parameters can have default values. However, default parameter values are evaluated **once at function definition time**, not every time the function is called.

### ⚠️ The Infamous Mutable Default Argument Trap
```python
# ANTI-PATTERN: DO NOT DO THIS!
def add_item_bad(item, item_list=[]):
    item_list.append(item)
    return item_list

print(add_item_bad("apple"))  # ['apple']
print(add_item_bad("banana")) # ['apple', 'banana'] - Oops! Shared state!
```

### ✅ The Idiomatic Python Solution
Use `None` as the sentinel default value:
```python
def add_item_clean(item, item_list=None):
    if item_list is None:
        item_list = []
    item_list.append(item)
    return item_list

print(add_item_clean("apple"))  # ['apple']
print(add_item_clean("banana")) # ['banana'] - Clean!
```

---

## 3. `*args` and `**kwargs`

- `*args` packs variable positional arguments into a `tuple`.
- `**kwargs` packs variable keyword arguments into a `dict`.

```python
def build_profile(user_id, *roles, **attributes):
    print(f"User ID: {user_id}")
    print(f"Roles Tuple: {roles}")
    print(f"Attributes Dict: {attributes}")

build_profile(
    42,
    "admin", "developer", "tester",
    email="dev@example.com",
    department="Cloud Infrastructure"
)
```

### Unpacking Arguments
You can also unpack collections when calling functions:
```python
params = [10, 20]
kwargs = {"sep": " -- ", "end": "\nDone\n"}
print(*params, **kwargs) # 10 -- 20\nDone
```

---

## 4. Positional-Only (`/`) & Keyword-Only (`*`) Parameters

Python allows fine-grained control over how callers invoke your functions:

```python
def configure_network(ip_address, /, retries=3, *, timeout=30):
    """
    - ip_address: Positional-only (left of '/')
    - retries: Can be positional OR keyword
    - timeout: Keyword-only (right of '*')
    """
    return f"Connecting to {ip_address} with {retries} retries, timeout={timeout}s"

# Valid calls:
configure_network("192.168.1.1", 5, timeout=10)
configure_network("192.168.1.1", retries=5, timeout=10)

# Invalid calls:
# configure_network(ip_address="192.168.1.1") # TypeError: positional-only
# configure_network("192.168.1.1", 5, 10)     # TypeError: timeout is keyword-only
```

---

## 5. The LEGB Scope Hierarchy

When resolving a variable name, Python searches four scopes sequentially:

1. **L**ocal: Inside the current function block.
2. **E**nclosing: In enclosing/outer functions (closures).
3. **G**lobal: At the top level of the module file.
4. **B**uilt-in: Python's pre-loaded namespace (`len`, `range`, `ValueError`).

```
+------------------------------------------+
| Built-in (e.g. print, len, int)          |
|  +-------------------------------------+ |
|  | Global (Module level)               | |
|  |  +--------------------------------+ | |
|  |  | Enclosing (Outer function)     | | |
|  |  |  +---------------------------+ | | |
|  |  |  | Local (Inner function)    | | | |
|  |  |  +---------------------------+ | | |
|  |  +--------------------------------+ | |
|  +-------------------------------------+ |
+------------------------------------------+
```

---

## 6. `global` and `nonlocal` Keywords

- `global`: Instructs Python to bind assignments to the module-level variable.
- `nonlocal`: Instructs Python to bind assignments to the nearest enclosing (outer) function variable.

```python
counter = 0

def increment_global():
    global counter
    counter += 1

def outer():
    value = 10
    def inner():
        nonlocal value
        value += 5
    inner()
    return value

print(outer()) # 15
```

---

## 7. First-Class Functions & Closures

In Python, functions are first-class citizens: they can be assigned to variables, passed as arguments, and returned from other functions.

### Closures
A closure occurs when an inner function remembers and has access to variables from its lexical enclosing scope, even after the outer function has finished executing.

```python
def make_multiplier(factor: int):
    # 'factor' is captured in the closure
    def multiply(number: int) -> int:
        return number * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10)) # 20
print(triple(10)) # 30
```

---

## 8. Anonymous Functions (`lambda`)

A `lambda` is an inline, one-line function expression without a name:
`lambda param1, param2: expression`

```python
add = lambda a, b: a + b
print(add(3, 7)) # 10

# Commonly used for custom sorting keys:
pairs = [(1, "one"), (3, "three"), (2, "two")]
pairs.sort(key=lambda item: item[1]) # Sort alphabetically by word
print(pairs)
```

---

## 9. Functional Primitives: `map`, `filter`, `reduce`

### `map(func, iterable)`
Applies a function to all elements:
```python
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers)) # [1, 4, 9, 16, 25]
```

### `filter(func, iterable)`
Keeps elements where the function returns `True`:
```python
evens = list(filter(lambda x: x % 2 == 0, numbers)) # [2, 4]
```

### `functools.reduce(func, iterable)`
Repeatedly applies a binary function to reduce an iterable to a single value:
```python
from functools import reduce
product = reduce(lambda acc, x: acc * x, numbers, 1) # 1 * 2 * 3 * 4 * 5 = 120
```

---

## 10. Modern Python Type Annotations

Type annotations improve maintainability and enable static analysis with tools like `mypy`.

```python
from typing import Callable, Optional

def fetch_user_data(
    user_id: int,
    transform: Optional[Callable[[dict], dict]] = None
) -> dict | None: # Python 3.10+ union operator
    if user_id <= 0:
        return None
    raw = {"id": user_id, "name": "Aman"}
    return transform(raw) if transform else raw
```

---

## 11. Recursion Fundamentals

A recursive function calls itself to solve smaller subproblems until hitting a base case.

```python
def factorial(n: int) -> int:
    # 1. Base case
    if n <= 1:
        return 1
    # 2. Recursive step
    return n * factorial(n - 1)

print(factorial(5)) # 120
```

---

## 12. Practice Questions

### Conceptual Questions

**Q1: Why is defining `def fn(data=[])` problematic?**
> **Answer:** The empty list `[]` is created once when the function definition is executed. Every invocation that relies on the default parameter references the exact same list instance in memory, causing unexpected side effects across calls.

**Q2: What is the purpose of the `nonlocal` keyword?**
> **Answer:** It declares that a variable refers to an identifier in the nearest enclosing function scope rather than creating a new local variable in the inner function.

**Q3: Explain the difference between `*args` and `**kwargs`.**
> **Answer:** `*args` gathers non-keyword positional arguments into a `tuple`. `**kwargs` gathers named keyword arguments into a `dict`.

**Q4: How does Python determine when to stop evaluating in a recursive function?**
> **Answer:** By checking the base case condition. Without a terminating base case, Python raises a `RecursionError: maximum recursion depth exceeded`.

**Q5: What is a closure and what conditions must be met to form one?**
> **Answer:** A closure is an inner function that retains access to variables from its enclosing scope. Requirements: 1) A nested function; 2) The nested function references a variable in the enclosing function; 3) The enclosing function returns the nested function.

---

### Coding Challenges

#### Challenge 1: Memoized Fibonacci
Implement Fibonacci with memoization using a dictionary closure.

```python
# Solution 1:
def make_fibonacci():
    cache = {0: 0, 1: 1}
    def fib(n: int) -> int:
        if n not in cache:
            cache[n] = fib(n - 1) + fib(n - 2)
        return cache[n]
    return fib

fib = make_fibonacci()
print([fib(i) for i in range(10)]) # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

#### Challenge 2: Custom Pipeline Combiner
Create a function `pipeline(*funcs)` that chains multiple unary functions into one composite function.

```python
# Solution 2:
def pipeline(*funcs):
    def composed(data):
        for fn in funcs:
            data = fn(data)
        return data
    return composed

clean = pipeline(
    lambda s: s.strip(),
    lambda s: s.lower(),
    lambda s: s.replace(" ", "_")
)
print(clean("   Hello World Python   ")) # "hello_world_python"
```

#### Challenge 3: Deep Flatten Recursively
Flatten arbitrarily nested lists: `[1, [2, [3, 4], 5], 6]` -> `[1, 2, 3, 4, 5, 6]`.

```python
# Solution 3:
def deep_flatten(nested):
    result = []
    for item in nested:
        if isinstance(item, list):
            result.extend(deep_flatten(item))
        else:
            result.append(item)
    return result

print(deep_flatten([1, [2, [3, 4], 5], 6]))
```

#### Challenge 4: Safe Division with Keyword-Only Default
Create a function `safe_divide(a, b, /, *, fallback=0.0)` that prevents division by zero without raising an exception.

```python
# Solution 4:
def safe_divide(a: float, b: float, /, *, fallback: float = 0.0) -> float:
    return fallback if b == 0 else a / b

print(safe_divide(10, 2))              # 5.0
print(safe_divide(10, 0, fallback=-1)) # -1
```

#### Challenge 5: Top-N Words by Frequency using Functional Tools
Given a list of words, find the top `N` most frequent words using `filter`, `sorted`, and lambdas.

```python
# Solution 5:
words = ["apple", "banana", "apple", "cherry", "banana", "apple", "date"]
counts = {w: words.count(w) for w in set(words)}
top_2 = sorted(counts.items(), key=lambda pair: pair[1], reverse=True)[:2]
print(top_2) # [('apple', 3), ('banana', 2)]
```

---

## 13. Hands-On Module Activity

### 🎯 Project: Modular Text Analytics & Transformation Engine

#### Objectives
Build a functional pipeline engine that analyzes and transforms text documents:
1. Provide higher-order functions to register custom transformations.
2. Extract text metrics (word count, character count, sentence count, average word length, vocabulary richness).
3. Compute crude sentiment heuristic (+1 for positive words, -1 for negative words).
4. Apply multiple formatting transformers sequentially (slugify, title-case, censor profanity).

#### Implementation
Create a file named `text_engine.py`:

```python
# text_engine.py
"""
Modular Text Analytics & Transformation Engine
Module 04 Hands-On Project
"""
from typing import Callable, Dict, Any, List

# Standard Sentiment Lexicon
POSITIVE_WORDS = {"great", "good", "excellent", "awesome", "fantastic", "clean", "pythonic", "fast", "love"}
NEGATIVE_WORDS = {"bad", "terrible", "slow", "buggy", "horrible", "ugly", "error", "fail", "hate"}

def analyze_text(text: str) -> Dict[str, Any]:
    """Analyzes text structure and statistical characteristics."""
    words = [w.strip(".,!?;:\"'()").lower() for w in text.split() if w.strip()]
    sentences = [s for s in text.replace("?", ".").replace("!", ".").split(".") if s.strip()]

    word_count = len(words)
    unique_words = len(set(words))
    lexical_richness = (unique_words / word_count) if word_count > 0 else 0.0
    avg_word_len = sum(len(w) for w in words) / word_count if word_count > 0 else 0.0

    pos_score = sum(1 for w in words if w in POSITIVE_WORDS)
    neg_score = sum(1 for w in words if w in NEGATIVE_WORDS)
    net_sentiment = pos_score - neg_score

    sentiment_label = "Neutral"
    if net_sentiment > 0:
        sentiment_label = "Positive"
    elif net_sentiment < 0:
        sentiment_label = "Negative"

    return {
        "word_count": word_count,
        "char_count": len(text),
        "sentence_count": len(sentences),
        "average_word_length": round(avg_word_len, 2),
        "lexical_richness": round(lexical_richness, 3),
        "sentiment": sentiment_label,
        "sentiment_score": net_sentiment,
    }

def create_pipeline(*transforms: Callable[[str], str]) -> Callable[[str], str]:
    """Composes several string transformation functions into a single pipeline."""
    def run_pipeline(text: str) -> str:
        current = text
        for transform in transforms:
            current = transform(current)
        return current
    return run_pipeline

# Transformation Utilities
def strip_whitespace(s: str) -> str:
    return " ".join(s.split())

def censor_banned_words(banned: List[str]) -> Callable[[str], str]:
    def transformer(text: str) -> str:
        result = text
        for b in banned:
            result = result.replace(b, "*" * len(b))
        return result
    return transformer

def to_url_slug(text: str) -> str:
    cleaned = "".join(c.lower() if c.isalnum() or c.isspace() else "" for c in text)
    return "-".join(cleaned.split())

def main():
    sample_text = (
        "  Python is a fantastic and clean language! However, buggy code "
        "and slow algorithms can cause terrible headaches for developers.  "
    )

    print("=" * 60)
    print("        TEXT ANALYTICS & TRANSFORMATION ENGINE        ")
    print("=" * 60)
    print("Original Text:\n", sample_text)
    print("-" * 60)

    # 1. Run Analytics
    metrics = analyze_text(sample_text)
    print("📊 Analysis Metrics:")
    for metric, val in metrics.items():
        print(f"  • {metric.replace('_', ' ').title():<22}: {val}")

    # 2. Build and Apply Transformation Pipelines
    clean_and_censor = create_pipeline(
        strip_whitespace,
        censor_banned_words(["buggy", "terrible"])
    )

    censored_output = clean_and_censor(sample_text)
    print("-" * 60)
    print("🛡️ Cleaned & Censored Output:\n", censored_output)

    slug_output = to_url_slug(sample_text)
    print("-" * 60)
    print("🔗 URL Slug Generation:\n", slug_output)
    print("=" * 60)

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python text_engine.py
```

---
**Next Step:** Proceed to [**Module 05: Files, Comprehensions & Error Handling**](../05-files-comprehensions-and-errors/README.md)!
