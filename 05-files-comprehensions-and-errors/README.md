# Module 05: Files, Comprehensions & Error Handling

Real-world applications must interact with persistent storage (files, databases, network streams) and handle unpredictable runtime anomalies safely. In this module, you will master comprehensions, generator expressions, robust exception handling, and persistent data formats (CSV, JSON, pathlib).

---

## 📖 Table of Contents
1. [List, Dictionary, and Set Comprehensions](#1-list-dictionary-and-set-comprehensions)
2. [Generator Expressions vs List Comprehensions](#2-generator-expressions-vs-list-comprehensions)
3. [Exception Handling: `try`, `except`, `else`, `finally`](#3-exception-handling-try-except-else-finally)
4. [Exception Hierarchy & Exception Chaining](#4-exception-hierarchy--exception-chaining)
5. [Custom Exception Classes](#5-custom-exception-classes)
6. [Working with Plain Files & The `with` Statement](#6-working-with-plain-files--the-with-statement)
7. [Structured Files: CSV (`csv` module)](#7-structured-files-csv-csv-module)
8. [Structured Files: JSON (`json` module)](#8-structured-files-json-json-module)
9. [Modern Filesystem Operations: `pathlib.Path`](#9-modern-filesystem-operations-pathlibpath)
10. [Practice Questions](#10-practice-questions)
11. [Hands-On Module Activity](#11-hands-on-module-activity)

---

## 1. List, Dictionary, and Set Comprehensions

Comprehensions provide a concise, declarative syntax for transforming iterables.

### List Comprehensions
Syntax: `[expression for item in iterable if condition]`
```python
# Square only even numbers
evens = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]
```

### Dictionary Comprehensions
Syntax: `{key_expr: value_expr for item in iterable if condition}`
```python
names = ["Alice", "Bob", "Charlie"]
name_lengths = {name: len(name) for name in names}
# {'Alice': 5, 'Bob': 3, 'Charlie': 7}
```

### Set Comprehensions
Syntax: `{expression for item in iterable if condition}`
```python
words = ["hello", "world", "hello", "python"]
unique_lengths = {len(w) for w in words}
# {5, 6}
```

---

## 2. Generator Expressions vs List Comprehensions

A generator expression uses parentheses `()` instead of square brackets `[]`. It yields values lazily on-demand rather than allocating the entire list in memory at once.

```python
import sys

# List comprehension: All 1,000,000 items allocated immediately in RAM
list_comp = [x for x in range(1_000_000)]
print(f"List Size: {sys.getsizeof(list_comp)} bytes") # ~8.4 MB

# Generator expression: Only holds the current state generator object
gen_exp = (x for x in range(1_000_000))
print(f"Generator Size: {sys.getsizeof(gen_exp)} bytes") # ~104 bytes!
```

---

## 3. Exception Handling: `try`, `except`, `else`, `finally`

Never let an unexpected runtime crash crash your application.

```python
def read_config_value(filename: str, key: str):
    try:
        file = open(filename, "r")
        # Do some processing
        data = file.read()
    except FileNotFoundError as err:
        print(f"Error: Could not locate file {filename}: {err}")
    except PermissionError:
        print("Error: Insufficient read permissions.")
    else:
        # Executes ONLY if NO exception was raised in the try block
        print("File read successfully!")
        return data
    finally:
        # ALWAYS executes (cleanup actions, closing connections)
        print("Closing open handles if needed.")
```

---

## 4. Exception Hierarchy & Exception Chaining

All exceptions in Python inherit from `BaseException`. You should always catch `Exception` or its subclasses, never bare `except:`.

```
BaseException
 ├── KeyboardInterrupt
 ├── SystemExit
 └── Exception
      ├── ArithmeticError (ZeroDivisionError, OverflowError)
      ├── LookupError (IndexError, KeyError)
      ├── OSError (FileNotFoundError, PermissionError)
      ├── TypeError
      └── ValueError
```

### Exception Chaining (`raise ... from ...`)
Preserve the root cause while raising an application-specific error:
```python
try:
    port = int("invalid_port")
except ValueError as original_error:
    raise RuntimeError("Failed to initialize server configuration") from original_error
```

---

## 5. Custom Exception Classes

Define domain-specific exceptions by subclassing `Exception`:

```python
class InsufficientFundsError(Exception):
    """Raised when an account balance is inadequate for a withdrawal."""
    def __init__(self, current_balance: float, requested_amount: float):
        self.current_balance = current_balance
        self.requested_amount = requested_amount
        super().__init__(
            f"Withdrawal of ${requested_amount:.2f} failed. Available: ${current_balance:.2f}"
        )

def withdraw(balance: float, amount: float):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount
```

---

## 6. Working with Plain Files & The `with` Statement

Always manage file handles using the `with` context manager. It guarantees the file stream is closed, even if an exception occurs.

```python
# Writing to a text file
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("Line 1: Foundations\n")
    f.writelines(["Line 2: Data Structures\n", "Line 3: OOP\n"])

# Reading lines safely
with open("notes.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
```

---

## 7. Structured Files: CSV (`csv` module)

### Reading and Writing with `DictWriter` and `DictReader`
```python
import csv

inventory = [
    {"sku": "A101", "name": "USB-C Cable", "price": 9.99},
    {"sku": "B202", "name": "Mechanical Keyboard", "price": 89.99},
]

# Writing CSV with headers
with open("products.csv", "w", newline="", encoding="utf-8") as csvfile:
    fields = ["sku", "name", "price"]
    writer = csv.DictWriter(csvfile, fieldnames=fields)
    writer.writeheader()
    writer.writerows(inventory)

# Reading CSV
with open("products.csv", "r", encoding="utf-8") as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        print(f"Product: {row['name']} -> ${float(row['price']):.2f}")
```

---

## 8. Structured Files: JSON (`json` module)

JSON is the standard format for web APIs and configuration.

| Python Type | JSON Equivalent |
|-------------|-----------------|
| `dict` | Object `{}` |
| `list`, `tuple` | Array `[]` |
| `str` | String `""` |
| `int`, `float` | Number |
| `True` / `False` | `true` / `false` |
| `None` | `null` |

```python
import json

payload = {
    "server": "us-east-1",
    "ports": [80, 443, 8080],
    "ssl_enabled": True
}

# Serialize to JSON file
with open("config.json", "w", encoding="utf-8") as f:
    json.dump(payload, f, indent=4)

# Deserialize from JSON file
with open("config.json", "r", encoding="utf-8") as f:
    loaded_config = json.load(f)
    print(loaded_config["ports"])
```

---

## 9. Modern Filesystem Operations: `pathlib.Path`

Python 3's `pathlib` provides an object-oriented approach to paths across Windows, macOS, and Linux:

```python
from pathlib import Path

# Constructing paths with the '/' operator
project_root = Path.cwd()
data_dir = project_root / "data" / "exports"

# Create directories if they do not exist
data_dir.mkdir(parents=True, exist_ok=True)

target_file = data_dir / "summary.txt"
target_file.write_text("Export finished successfully.", encoding="utf-8")

if target_file.exists():
    print("File size:", target_file.stat().st_size, "bytes")
    print("Content:", target_file.read_text(encoding="utf-8"))
```

---

## 10. Practice Questions

### Conceptual Questions

**Q1: What is the benefit of using `with open(...)` instead of `f = open(...); f.close()`?**
> **Answer:** The `with` statement utilizes Python's context manager protocol (`__enter__` and `__exit__`). It guarantees that `close()` is invoked automatically, even if an unhandled exception or early return occurs inside the block.

**Q2: What is the fundamental difference between `[x for x in seq]` and `(x for x in seq)`?**
> **Answer:** The list comprehension evaluates all elements immediately and stores the entire list in memory ($O(N)$ space). The generator expression creates a generator object that evaluates items lazily on demand ($O(1)$ auxiliary space).

**Q3: When does the `else` block in a `try...except...else...finally` statement execute?**
> **Answer:** It executes only when the `try` block finishes without raising any exceptions.

**Q4: What is the difference between `json.dump()` and `json.dumps()`?**
> **Answer:** `json.dump()` serializes a Python object directly to an open file-like stream. `json.dumps()` serializes a Python object into a Python `str` in memory.

**Q5: Why is `except Exception:` preferred over bare `except:`?**
> **Answer:** A bare `except:` catches `BaseException`, including `KeyboardInterrupt` (`Ctrl+C`) and `SystemExit`, preventing the user or OS from terminating the program normally.

---

### Coding Challenges

#### Challenge 1: Matrix Transposition with Comprehension
Given a 2D matrix, compute its transpose using a single nested list comprehension.

```python
# Solution 1:
matrix = [
    [1, 2, 3],
    [4, 5, 6]
]
transposed = [[row[col] for row in matrix] for col in range(len(matrix[0]))]
print(transposed) # [[1, 4], [2, 5], [3, 6]]
```

#### Challenge 2: Word Length Dictionary Filtering
Given a list of words, construct a dictionary containing words as keys and their lengths as values, keeping only words with length > 4.

```python
# Solution 2:
words = ["python", "go", "typescript", "c", "rust", "javascript"]
long_words = {w: len(w) for w in words if len(w) > 4}
print(long_words) # {'python': 6, 'typescript': 10, 'javascript': 10}
```

#### Challenge 3: Safe JSON Reader with Fallback
Write a function `load_json_safe(filepath, default_data)` that reads a JSON file, returning `default_data` if the file doesn't exist or has malformed JSON.

```python
# Solution 3:
import json
from pathlib import Path

def load_json_safe(filepath: str, default_data: dict) -> dict:
    path = Path(filepath)
    if not path.exists():
        return default_data
    try:
        with open(path, "r", encoding="utf-8") as f:
            return json.load(f)
    except (json.JSONDecodeError, OSError):
        return default_data
```

#### Challenge 4: File Line Counter & Character Tally
Count total lines, words, and characters of a text file in a memory-efficient manner.

```python
# Solution 4:
def file_stats(filename: str):
    lines = words = chars = 0
    with open(filename, "r", encoding="utf-8") as f:
        for line in f:
            lines += 1
            words += len(line.split())
            chars += len(line)
    return {"lines": lines, "words": words, "chars": chars}
```

#### Challenge 5: Custom Input Validator with Retry
Ask the user for an integer between 1 and 100, continuing to prompt until valid input is given, handling non-numeric entries gracefully.

```python
# Solution 5:
def get_bounded_integer(prompt: str, min_val: int = 1, max_val: int = 100) -> int:
    while True:
        try:
            val = int(input(prompt))
            if not (min_val <= val <= max_val):
                raise ValueError(f"Value must be between {min_val} and {max_val}.")
            return val
        except ValueError as err:
            print(f"Invalid input: {err}. Please try again.")
```

---

## 11. Hands-On Module Activity

### 🎯 Project: Persistent Personal Finance & Expense Tracker

#### Objectives
Build a production-grade personal finance CLI application that:
1. Loads existing transactions from a local JSON file on startup (or initializes a clean ledger if none exists).
2. Allows users to record income and expenses (Amount, Category, Description, Date).
3. Provides category-wise expense breakdown, total savings, and average spending metrics.
4. Exports transactions to CSV format.
5. Employs comprehensive error handling for corrupted files and invalid user entries.

#### Implementation
Create a file named `expense_tracker.py`:

```python
# expense_tracker.py
"""
Persistent Personal Finance & Expense Tracker
Module 05 Hands-On Project
"""
import json
import csv
from datetime import datetime
from pathlib import Path
from typing import List, Dict, Any

DATA_FILE = Path("ledger.json")
CSV_EXPORT_FILE = Path("ledger_export.csv")

def load_ledger() -> List[Dict[str, Any]]:
    """Loads transactions from disk safely."""
    if not DATA_FILE.exists():
        return []
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except (json.JSONDecodeError, OSError) as e:
        print(f"⚠️ Warning: Could not read {DATA_FILE} ({e}). Starting with empty ledger.")
        return []

def save_ledger(transactions: List[Dict[str, Any]]) -> None:
    """Persists transactions to disk."""
    try:
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump(transactions, f, indent=2)
    except OSError as e:
        print(f"❌ Failed to save data: {e}")

def add_transaction(transactions: List[Dict[str, Any]]):
    print("\n--- Add Transaction ---")
    kind = input("Type (1: Expense, 2: Income): ").strip()
    txn_type = "Expense" if kind == "1" else "Income" if kind == "2" else None

    if not txn_type:
        print("❌ Invalid transaction type.")
        return

    try:
        amount = float(input("Amount ($): "))
        if amount <= 0:
            print("❌ Amount must be positive.")
            return
    except ValueError:
        print("❌ Invalid numeric amount.")
        return

    category = input("Category (e.g., Food, Rent, Salary, Tech): ").strip().title() or "Miscellaneous"
    description = input("Short Description: ").strip() or "N/A"
    date_str = datetime.now().strftime("%Y-%m-%d %H:%M")

    record = {
        "id": len(transactions) + 1,
        "date": date_str,
        "type": txn_type,
        "category": category,
        "amount": amount,
        "description": description
    }

    transactions.append(record)
    save_ledger(transactions)
    print(f"✅ Recorded {txn_type} of ${amount:.2f} under '{category}'")

def view_summary(transactions: List[Dict[str, Any]]):
    if not transactions:
        print("\nNo transactions recorded yet.")
        return

    total_income = sum(t["amount"] for t in transactions if t["type"] == "Income")
    total_expense = sum(t["amount"] for t in transactions if t["type"] == "Expense")
    net_savings = total_income - total_expense

    # Category breakdown for expenses using dict comprehension
    expense_categories = {}
    for t in transactions:
        if t["type"] == "Expense":
            cat = t["category"]
            expense_categories[cat] = expense_categories.get(cat, 0.0) + t["amount"]

    print("\n" + "=" * 55)
    print(f"{'FINANCIAL SUMMARY REPORT':^55}")
    print("=" * 55)
    print(f"Total Income:       ${total_income:>12.2f}")
    print(f"Total Expenses:     ${total_expense:>12.2f}")
    print(f"Net Balance:        ${net_savings:>12.2f}")
    print("-" * 55)
    print("Expenses by Category:")
    for cat, amt in sorted(expense_categories.items(), key=lambda x: x[1], reverse=True):
        pct = (amt / total_expense * 100) if total_expense > 0 else 0
        print(f"  • {cat:<18}: ${amt:>8.2f} ({pct:5.1f}%)")
    print("=" * 55)

def export_to_csv(transactions: List[Dict[str, Any]]):
    if not transactions:
        print("\nNo records to export.")
        return
    try:
        with open(CSV_EXPORT_FILE, "w", newline="", encoding="utf-8") as f:
            fields = ["id", "date", "type", "category", "amount", "description"]
            writer = csv.DictWriter(f, fieldnames=fields)
            writer.writeheader()
            writer.writerows(transactions)
        print(f"✅ Successfully exported {len(transactions)} records to {CSV_EXPORT_FILE.resolve()}")
    except OSError as e:
        print(f"❌ Error during CSV export: {e}")

def main():
    transactions = load_ledger()
    print("💰 Personal Expense & Finance Tracker Initialized.")

    while True:
        print("\nMENU:")
        print("1. Add Transaction")
        print("2. View Financial Summary")
        print("3. Export Records to CSV")
        print("4. Exit")
        choice = input("Select an option (1-4): ").strip()

        if choice == "1":
            add_transaction(transactions)
        elif choice == "2":
            view_summary(transactions)
        elif choice == "3":
            export_to_csv(transactions)
        elif choice == "4":
            print("Goodbye!")
            break
        else:
            print("Invalid option. Please enter 1, 2, 3, or 4.")

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python expense_tracker.py
```

---
**Next Step:** Proceed to [**Module 06: Object-Oriented Programming (OOP)**](../06-object-oriented-programming/README.md)!
