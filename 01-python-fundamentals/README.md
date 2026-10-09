# Module 01: Python Fundamentals & Environment Setup

Welcome to Module 01! In this module, you will learn how Python works under the hood, how to set up your programming environment, write your first scripts, and master core fundamentals like variables, data types, arithmetic operations, user input/output, and modern string formatting.

---

## 📖 Table of Contents
1. [What is Python & How It Works](#1-what-is-python--how-it-works)
2. [Setting Up Your Environment](#2-setting-up-your-environment)
3. [Your First Python Program & The REPL](#3-your-first-python-program--the-repl)
4. [Python Syntax, Indentation & Comments](#4-python-syntax-indentation--comments)
5. [Variables & Dynamic Typing](#5-variables--dynamic-typing)
6. [Core Data Types](#6-core-data-types)
7. [Arithmetic Operators & Numeric Operations](#7-arithmetic-operators--numeric-operations)
8. [Type Conversion & Inspection](#8-type-conversion--inspection)
9. [User Input and Output](#9-user-input-and-output)
10. [String Operations & Modern Formatting (f-strings)](#10-string-operations--modern-formatting-f-strings)
11. [PEP 8 Style Essentials](#11-pep-8-style-essentials)
12. [Practice Questions](#12-practice-questions)
13. [Hands-On Module Activity](#13-hands-on-module-activity)

---

## 1. What is Python & How It Works

Python is a high-level, interpreted, dynamically typed programming language created by **Guido van Rossum** in 1991. It emphasizes code readability, developer productivity, and expressive syntax.

### How Python Executes Code
When you run a Python script (e.g. `python script.py`):
1. **Source Code (`.py`)**: You write human-readable code.
2. **Bytecode Compilation (`.pyc`)**: CPython (the default implementation) compiles the source code into low-level, platform-independent bytecode instructions.
3. **Python Virtual Machine (PVM)**: The PVM runs an execution loop that translates bytecode instructions into CPU machine code.

```
+---------------+      +-------------------+      +-------------------------+
| Source (.py)  | ---> | Bytecode (.pyc)   | ---> | Python Virtual Machine  | ---> CPU Execution
+---------------+      +-------------------+      | (PVM Interpreter)       |
                                                  +-------------------------+
```

### The Zen of Python
Python's core design philosophy can be explored directly in the terminal:
```python
import this
```
Key principles include:
- *Beautiful is better than ugly.*
- *Explicit is better than implicit.*
- *Simple is better than complex.*
- *Readability counts.*

---

## 2. Setting Up Your Environment

### Verifying Python Installation
Check your Python and pip versions:
```bash
python --version
pip --version
```
> **Note:** On macOS/Linux systems where Python 2 used to exist, you may need to type `python3` and `pip3`.

### Recommended VS Code Setup
1. Install **Visual Studio Code**.
2. Install the **Python** extension by Microsoft (`ms-python.python`).
3. Set your Python interpreter via `Ctrl+Shift+P` (or `Cmd+Shift+P`) > `Python: Select Interpreter`.

---

## 3. Your First Python Program & The REPL

### The Interactive Shell (REPL)
Type `python` into your terminal to enter the Read-Eval-Print Loop:
```python
>>> 2 + 2
4
>>> name = "Developer"
>>> f"Hello, {name}!"
'Hello, Developer!'
>>> exit()
```

### Running a Script
Create a file named `main.py`:
```python
# main.py
print("Hello, Python World!")
```
Run it in your terminal:
```bash
python main.py
```

---

## 4. Python Syntax, Indentation & Comments

Unlike languages like C++, Java, or JavaScript that use curly braces `{}` to define blocks of code, Python uses **indentation** (standard is 4 spaces).

### Indentation Rules
```python
total_score = 95

if total_score >= 90:
    print("Grade: A")
    print("Excellent work!")  # Both statements belong to the if block
print("Evaluation complete.") # Outside the if block
```

### Comments
```python
# Single-line comment: explain the "why", not the "what"

"""
Multi-line docstring:
Often used at the top of files, modules,
classes, and functions.
"""
```

---

## 5. Variables & Dynamic Typing

In Python, variables are **labels (references)** pointing to objects in memory. You do not declare variable types explicitly.

```python
x = 10          # x references an integer object
print(type(x))  # <class 'int'>

x = "Hello"     # x now references a string object (Dynamic Typing)
print(type(x))  # <class 'str'>
```

### Variable Naming Rules & Conventions (PEP 8)
- Must start with a letter or an underscore `_`.
- Cannot start with a digit.
- Can contain letters, numbers, and underscores (`user_age`, `max_value`).
- Case-sensitive: `age`, `Age`, and `AGE` are 3 distinct variables.
- Use `snake_case` for variables and functions.
- Constants are written in `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRIES = 5`).

---

## 6. Core Data Types

Python has several built-in primitive types:

| Type | Name | Example | Mutability |
|------|------|---------|------------|
| `int` | Integer | `42`, `-7`, `1_000_000` | Immutable |
| `float` | Floating-point | `3.14159`, `-0.001`, `1e-4` | Immutable |
| `complex` | Complex numbers | `2 + 3j` | Immutable |
| `bool` | Boolean | `True`, `False` | Immutable |
| `str` | String | `"Python"`, `'Coding'` | Immutable |
| `NoneType`| Absence of value | `None` | Immutable |

```python
age = 25              # int
pi = 3.14159          # float
is_active = True      # bool
greeting = "Welcome"  # str
optional_data = None  # NoneType
```

---

## 7. Arithmetic Operators & Numeric Operations

| Operator | Description | Example | Result |
|----------|-------------|---------|--------|
| `+` | Addition | `10 + 3` | `13` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `10 * 3` | `30` |
| `/` | True Division (always returns float) | `10 / 2` | `5.0` |
| `//` | Floor Division (discards remainder) | `10 // 3` | `3` |
| `%` | Modulo (remainder) | `10 % 3` | `1` |
| `**` | Exponentiation (power) | `2 ** 4` | `16` |

### Operator Precedence (PEMDAS)
Parentheses `()` > Exponents `**` > Multiplication/Division/Modulo `*`, `/`, `//`, `%` > Addition/Subtraction `+`, `-`.

```python
result = (2 + 3) * 4 ** 2 / 2
# 5 * 16 / 2 = 80 / 2 = 40.0
```

---

## 8. Type Conversion & Inspection

### Checking Types
```python
value = 42
print(type(value))                # <class 'int'>
print(isinstance(value, int))     # True
print(isinstance(value, (int, float))) # True (check multiple types)
```

### Explicit Type Casting
```python
# String to Integer & Float
num_str = "120"
num_int = int(num_str)       # 120
price_float = float("19.99") # 19.99

# Number to String
text = str(450)              # "450"

# Boolean conversion (Truthiness)
print(bool(0))        # False
print(bool(1))        # True
print(bool(""))       # False (empty string)
print(bool("text"))   # True
print(bool(None))     # False
```

---

## 9. User Input and Output

### The `print()` Function
`print(*objects, sep=' ', end='\n', file=None, flush=False)`

```python
print("apple", "banana", "cherry", sep=", ") # "apple, banana, cherry"
print("Loading...", end="")                  # Doesn't print a newline
print("Done!")                               # Prints on the same line
```

### The `input()` Function
> ⚠️ **Important:** `input()` **always** returns data as a `str`. If you need a number, convert it explicitly!

```python
user_name = input("Enter your name: ")
user_age_raw = input("Enter your age: ")
user_age = int(user_age_raw)

print(f"Next year, {user_name} will be {user_age + 1} years old.")
```

---

## 10. String Operations & Modern Formatting (f-strings)

Strings are ordered, immutable sequences of characters.

### Indexing & Slicing
```python
language = "Python3"

# Indexing
print(language[0])    # 'P' (First character)
print(language[-1])   # '3' (Last character)

# Slicing: [start:stop:step]
print(language[0:4])  # 'Pyth' (stop index is exclusive)
print(language[:2])   # 'Py'
print(language[2:])   # 'thon3'
print(language[::-1]) # '3nohtyP' (Reversed string)
```

### Formatted String Literals (f-strings - Python 3.6+)
f-strings provide the most readable, expressive way to format strings:

```python
product = "Laptop"
price = 1299.4567
discount = 0.15

# Basic interpolation
print(f"Product: {product}, Final Price: ${price:.2f}")

# Arithmetic inside expressions
print(f"Price after discount: ${price * (1 - discount):.2f}")

# Alignment and padding
item = "RAM"
qty = 8
print(f"{item:<10} | {qty:>5}") # 'RAM        |     8'

# Debugging with '=' syntax (Python 3.8+)
x = 25
print(f"{x=}") # 'x=25'
```

---

## 11. PEP 8 Style Essentials

PEP 8 is Python's official style guide. Writing idiomatic Python ("Pythonic" code) is vital:
- **Indentation**: 4 spaces per indentation level (never tabs).
- **Line Length**: Limit all lines to a maximum of 79 to 88 characters.
- **Naming Conventions**:
  - `snake_case` for variables, attributes, and functions.
  - `PascalCase` for classes.
  - `SCREAMING_SNAKE_CASE` for constants.
- **Spacing**: Use a single space around binary operators (`x = y + 1`, not `x=y+1`).

---

## 12. Practice Questions

### Conceptual Questions

**Q1: What is the difference between `/` and `//` in Python?**
> **Answer:** `/` is true division which always returns a floating-point number (e.g. `10 / 2` is `5.0`). `//` is floor division which rounds down the result to the nearest mathematical integer (e.g. `10 // 3` is `3`, and `-10 // 3` is `-4`).

**Q2: What happens when you execute `x = (5)` vs `y = (5,)`?**
> **Answer:** `x = (5)` evaluates parentheses as grouping; `x` is an `int` with value `5`. `y = (5,)` has a trailing comma, which creates a single-element `tuple`.

**Q3: Why are strings called "immutable" in Python?**
> **Answer:** Once a string object is allocated in memory, its characters cannot be modified in-place. Operations that seem to modify a string (like `.replace()`, `.upper()`, or slicing) return a new string object.

**Q4: What will `bool("False")` evaluate to and why?**
> **Answer:** `True`. Any non-empty string in Python is truthy, regardless of the text it contains.

**Q5: How does Python manage memory for variables?**
> **Answer:** Python variables are pointers/references to objects in the heap. CPython uses reference counting alongside a cyclic garbage collector to automatically free memory when an object's reference count reaches zero.

---

### Coding Challenges

#### Challenge 1: Temperature Converter
Write a program that takes a temperature in Celsius from the user and outputs both Fahrenheit and Kelvin equivalents formatted to 2 decimal places.
- Formulas: $F = (C \times 9/5) + 32$, $K = C + 273.15$

```python
# Solution 1:
celsius = float(input("Enter temperature in Celsius: "))
fahrenheit = (celsius * 9 / 5) + 32
kelvin = celsius + 273.15

print(f"{celsius:.2f}°C = {fahrenheit:.2f}°F = {kelvin:.2f}K")
```

#### Challenge 2: Digits Sum
Given a 3-digit integer, extract each digit mathematically using `%` and `//` without converting it to a string, and compute the sum of digits.

```python
# Solution 2:
num = 742  # Example number
units = num % 10
tens = (num // 10) % 10
hundreds = num // 100
total = units + tens + hundreds

print(f"Sum of digits in {num}: {hundreds} + {tens} + {units} = {total}")
```

#### Challenge 3: Seconds Breakdown
Write a script that takes total seconds (e.g., `86400 + 3661 = 90061`) and formats it into `Days : Hours : Minutes : Seconds`.

```python
# Solution 3:
total_sec = int(input("Enter seconds: "))

days = total_sec // 86400
remainder = total_sec % 86400

hours = remainder // 3600
remainder = remainder % 3600

minutes = remainder // 60
seconds = remainder % 60

print(f"{days}d {hours}h {minutes}m {seconds}s")
```

#### Challenge 4: Simple Palindrome Checker (Basic Slicing)
Prompt the user for a word, clean any leading/trailing spaces, convert to lowercase, and check if it reads the same forward and backward using string slicing.

```python
# Solution 4:
word = input("Enter a word: ").strip().lower()
is_palindrome = word == word[::-1]
print(f"Is '{word}' a palindrome? {is_palindrome}")
```

#### Challenge 5: Receipt Line Generator
Format a store receipt line showing Item name (left-aligned 20 chars), Quantity (centered 6 chars), and Total Price (right-aligned 10 chars with currency).

```python
# Solution 5:
item = "Wireless Mouse"
qty = 2
unit_price = 24.99
total = qty * unit_price

print(f"{'Item':<20} {'Qty':^6} {'Total':>10}")
print("-" * 38)
print(f"{item:<20} {qty:^6} {f'${total:.2f}':>10}")
```

---

## 13. Hands-On Module Activity

### 🎯 Project: Multi-Currency Bill & Tip Splitter

#### Objectives
Build an interactive terminal application that:
1. Greets the customer with a formatted header.
2. Prompts for:
   - Bill total amount (float).
   - Tip percentage (e.g. 15, 18, 20).
   - Number of people splitting the bill (int).
   - Currency code (`USD`, `EUR`, `GBP`, `INR`).
3. Computes:
   - Tip amount.
   - Grand total.
   - Each person's share.
   - Conversion to an alternate currency using fixed conversion rates.
4. Outputs an ASCII-bordered summary receipt with aligned currency symbols and precision formatting.

#### Implementation
Create a file named `bill_splitter.py`:

```python
# bill_splitter.py
"""
Multi-Currency Bill & Tip Splitter
Module 01 Hands-On Project
"""

EXCHANGE_RATES_FROM_USD = {
    "USD": 1.00,
    "EUR": 0.92,
    "GBP": 0.79,
    "INR": 83.50,
}

CURRENCY_SYMBOLS = {
    "USD": "$",
    "EUR": "€",
    "GBP": "£",
    "INR": "₹",
}

print("=" * 45)
print("     WELCOME TO THE SMART BILL SPLITTER     ")
print("=" * 45)

# 1. Inputs
raw_bill = float(input("Enter bill amount (in USD): $"))
tip_pct = float(input("Enter tip percentage (e.g., 10, 15, 20): "))
people = int(input("How many people are splitting the bill? "))
target_currency = input("Convert to currency (USD, EUR, GBP, INR): ").strip().upper()

# 2. Computations
tip_amount = raw_bill * (tip_pct / 100.0)
grand_total_usd = raw_bill + tip_amount
per_person_usd = grand_total_usd / people

rate = EXCHANGE_RATES_FROM_USD.get(target_currency, 1.00)
symbol = CURRENCY_SYMBOLS.get(target_currency, "$")
grand_total_target = grand_total_usd * rate
per_person_target = per_person_usd * rate

# 3. Formatted Receipt
print("\n" + "#" * 45)
print(f"{'BILL BREAKDOWN RECEIPT':^45}")
print("#" * 45)
print(f" Subtotal (USD):               ${raw_bill:>10.2f}")
print(f" Tip Rate:                     {f'{tip_pct:.1f}%':>10}")
print(f" Tip Amount (USD):             ${tip_amount:>10.2f}")
print("-" * 45)
print(f" Total (USD):                  ${grand_total_usd:>10.2f}")
print(f" Party Count:                  {people:>10}")
print(f" Per Person (USD):             ${per_person_usd:>10.2f}")
print("-" * 45)
print(f" Target Currency:              {target_currency:>10}")
print(f" Total ({target_currency}):               {f'{symbol}{grand_total_target:.2f}':>10}")
print(f" Per Person ({target_currency}):          {f'{symbol}{per_person_target:.2f}':>10}")
print("#" * 45)
print(f"{'Thank you for dining with us!':^45}")
print("#" * 45)
```

Run and test your program:
```bash
python bill_splitter.py
```

---
**Next Step:** Proceed to [**Module 02: Control Flow & Logic**](../02-control-flow-and-logic/README.md) to master decision making, loops, and pattern matching!
