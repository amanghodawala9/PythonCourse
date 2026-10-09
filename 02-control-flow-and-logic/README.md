# Module 02: Control Flow & Logic

Welcome to Module 02! Programs become truly useful when they make decisions and repeat tasks. In this module, you will master conditional logic, loop mechanics, modern structural pattern matching (`match-case`), and loop optimization patterns.

---

## 📖 Table of Contents
1. [Boolean Expressions & Comparison Operators](#1-boolean-expressions--comparison-operators)
2. [Logical Operators & Short-Circuiting](#2-logical-operators--short-circuiting)
3. [Truthiness and Falsiness](#3-truthiness-and-falsiness)
4. [Identity (`is`) vs Equality (`==`)](#4-identity-is-vs-equality-)
5. [Membership Operators (`in`, `not in`)](#5-membership-operators-in-not-in)
6. [Conditional Branching (`if`, `elif`, `else`)](#6-conditional-branching-if-elif-else)
7. [The Conditional Expression (Ternary Operator)](#7-the-conditional-expression-ternary-operator)
8. [Structural Pattern Matching (`match-case`)](#8-structural-pattern-matching-match-case)
9. [`while` Loops & Loop Sentinel Conditions](#9-while-loops--loop-sentinel-conditions)
10. [`for` Loops & The `range()` Function](#10-for-loops--the-range-function)
11. [Loop Control: `break`, `continue`, `pass`](#11-loop-control-break-continue-pass)
12. [The Loop `else` Clause](#12-the-loop-else-clause)
13. [Iteration Utilities: `enumerate()` & `zip()`](#13-iteration-utilities-enumerate--zip)
14. [Practice Questions](#14-practice-questions)
15. [Hands-On Module Activity](#15-hands-on-module-activity)

---

## 1. Boolean Expressions & Comparison Operators

Comparison operators evaluate expressions down to a boolean (`True` or `False`):

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `==` | Equal to | `5 == 5` | `True` |
| `!=` | Not equal to | `5 != 4` | `True` |
| `>` | Greater than | `10 > 3` | `True` |
| `<` | Less than | `7 < 2` | `False` |
| `>=` | Greater than or equal to | `10 >= 10` | `True` |
| `<=` | Less than or equal to | `9 <= 8` | `False` |

### Chained Comparisons
Python supports chained comparisons in a mathematically natural syntax:
```python
age = 22
if 18 <= age < 65:
    print("Working-age adult")
```

---

## 2. Logical Operators & Short-Circuiting

Logical operators combine boolean evaluations:
- `and`: True only if **both** operands are True.
- `or`: True if **at least one** operand is True.
- `not`: Inverts the boolean value.

### Short-Circuit Evaluation
Python stops evaluating as soon as the outcome is determined:
- In `A and B`, if `A` is `False`, `B` is never evaluated.
- In `A or B`, if `A` is `True`, `B` is never evaluated.

```python
def check_server():
    print("Pinging server...")
    return True

# check_server() will NOT run because the left side is False
result = False and check_server() 
```

---

## 3. Truthiness and Falsiness

Every object in Python has an inherent boolean value.

### Falsy Values:
- Constants: `None`, `False`
- Numeric zeros: `0`, `0.0`, `0j`
- Empty sequences and collections: `""`, `[]`, `()`, `{}`, `set()`

### Truthy Values:
- Everything else (non-zero numbers, non-empty strings, lists with elements).

```python
username = input("Enter username: ").strip()

# Idiomatic check for non-empty string:
if username:
    print(f"Welcome, {username}!")
else:
    print("Username cannot be blank.")
```

---

## 4. Identity (`is`) vs Equality (`==`)

This is one of the most common pitfalls in Python:
- `==` checks for **value equality** (Do they hold the same contents?).
- `is` checks for **object identity** (Do they refer to the exact same memory address?).

```python
list_a = [1, 2, 3]
list_b = [1, 2, 3]

print(list_a == list_b)  # True (same values)
print(list_a is list_b)  # False (different objects in memory)
print(id(list_a) == id(list_b)) # False

# Always use 'is' when checking for None:
val = None
if val is None:
    print("Value is explicitly None")
```

---

## 5. Membership Operators (`in`, `not in`)

Check whether a value is present inside a sequence or container:
```python
allowed_roles = ["admin", "editor", "moderator"]
role = "guest"

if role not in allowed_roles:
    print("Access Denied!")
```

---

## 6. Conditional Branching (`if`, `elif`, `else`)

```python
score = 84

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Your Grade: {grade}")
```

---

## 7. The Conditional Expression (Ternary Operator)

Syntax: `value_if_true if condition else value_if_false`

```python
status = "Adult" if age >= 18 else "Minor"

# Useful for inline defaults
port = user_port if user_port else 8080
```

---

## 8. Structural Pattern Matching (`match-case`)

Introduced in Python 3.10, `match-case` is far more powerful than traditional `switch` statements, supporting destructuring and guard conditions.

```python
command = input("Enter command: ").split()

match command:
    case ["quit"] | ["exit"]:
        print("Exiting application...")
    case ["load", filename]:
        print(f"Loading file from: {filename}")
    case ["move", ("up" | "down" | "left" | "right") as direction, steps] if steps.isdigit():
        print(f"Moving {direction} by {steps} steps")
    case _:
        print("Unknown or invalid command syntax.")
```

---

## 9. `while` Loops & Loop Sentinel Conditions

`while` loops execute as long as the condition remains `True`.

```python
countdown = 5
while countdown > 0:
    print(f"T-minus {countdown}...")
    countdown -= 1
print("Liftoff! 🚀")
```

### Sentinel-controlled Loop
```python
total = 0
while True:
    entry = input("Enter a number to add (or 'done' to finish): ")
    if entry.lower() == "done":
        break
    total += float(entry)

print(f"Final Total: {total}")
```

---

## 10. `for` Loops & The `range()` Function

`for` loops in Python are "for-each" iterators over iterable sequences.

### Using `range(start, stop[, step])`
```python
# 0 to 4
for i in range(5):
    print(i, end=" ") # 0 1 2 3 4
print()

# 2 to 10 step by 2
for i in range(2, 11, 2):
    print(i, end=" ") # 2 4 6 8 10
print()

# Countdown from 5 down to 1
for i in range(5, 0, -1):
    print(i, end=" ") # 5 4 3 2 1
print()
```

---

## 11. Loop Control: `break`, `continue`, `pass`

- `break`: Terminates the loop immediately.
- `continue`: Skips the remainder of the current iteration and jumps to the next iteration.
- `pass`: A placeholder statement that performs no action.

```python
for num in range(1, 10):
    if num % 2 == 0:
        continue  # Skip even numbers
    if num == 7:
        print("Found lucky 7! Stopping.")
        break
    print(f"Processing odd: {num}")
```

---

## 12. The Loop `else` Clause

In Python, loops can have an `else` clause. It executes **only if the loop completes without encountering a `break`**.

```python
target = 13
primes = [2, 3, 5, 7, 11, 13, 17]

for p in primes:
    if p == target:
        print(f"Found {target} in list!")
        break
else:
    print(f"{target} was not in the primes list.")
```

---

## 13. Iteration Utilities: `enumerate()` & `zip()`

### `enumerate(iterable, start=0)`
Generates index-value pairs without manual counter variables:
```python
fruits = ["Apple", "Mango", "Blueberry"]

for index, fruit in enumerate(fruits, start=1):
    print(f"{index}. {fruit}")
```

### `zip(*iterables)`
Iterates through multiple sequences simultaneously:
```python
names = ["Alice", "Bob", "Charlie"]
scores = [92, 88, 95]

for name, score in zip(names, scores):
    print(f"{name}: {score}/100")
```

---

## 14. Practice Questions

### Conceptual Questions

**Q1: What will `print([] or "Python" and 0)` print?**
> **Answer:** `0`.
> Step 1: Precedence: `and` has higher precedence than `or`.
> Step 2: `"Python" and 0` evaluates to `0` (short-circuit logic: left is truthy, returns right).
> Step 3: `[] or 0` evaluates to `0` (left is falsy, returns right).

**Q2: When does a loop's `else` block NOT execute?**
> **Answer:** It does NOT execute if the loop is terminated by a `break` statement or an unhandled exception.

**Q3: Why shouldn't you check `if x == None`?**
> **Answer:** Idiomatic Python requires `if x is None`. `==` calls the `__eq__` method which could be overridden by custom objects to return custom booleans, while `is` performs an identity comparison against the single `NoneType` singleton.

**Q4: What is the difference between `pass` and `continue`?**
> **Answer:** `pass` does nothing and lets execution proceed to the next statement in the same block. `continue` immediately jumps to the next iteration of the enclosing loop.

**Q5: What is pattern destructuring in `match-case`?**
> **Answer:** It extracts elements from compound data types (tuples, lists, dicts, objects) directly into variable names matching the pattern schema.

---

### Coding Challenges

#### Challenge 1: FizzBuzz Classic
Print numbers 1 to 50:
- If divisible by 3: "Fizz"
- If divisible by 5: "Buzz"
- If divisible by both: "FizzBuzz"
- Otherwise, the number itself.

```python
# Solution 1:
for i in range(1, 51):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

#### Challenge 2: Prime Number Checker
Write a function or script that checks whether an input integer $N > 1$ is prime using a `for...else` loop.

```python
# Solution 2:
num = int(input("Enter integer: "))

if num > 1:
    for factor in range(2, int(num**0.5) + 1):
        if num % factor == 0:
            print(f"{num} is composite ({factor} x {num // factor}).")
            break
    else:
        print(f"{num} is a prime number!")
else:
    print(f"{num} is not prime.")
```

#### Challenge 3: Password Strength Evaluator
Evaluate a password:
- Length >= 8
- Has at least one uppercase letter
- Has at least one digit
- Has at least one special character (`!@#$%^&*`)

```python
# Solution 3:
pwd = input("Enter password: ")
has_upper = any(c.isupper() for c in pwd)
has_digit = any(c.isdigit() for c in pwd)
has_special = any(c in "!@#$%^&*" for c in pwd)
is_long = len(pwd) >= 8

if is_long and has_upper and has_digit and has_special:
    print("Strong Password! ✅")
else:
    print("Weak Password. Deficiencies:")
    if not is_long: print("- Must be at least 8 characters")
    if not has_upper: print("- Needs an uppercase letter")
    if not has_digit: print("- Needs a number")
    if not has_special: print("- Needs a special character (!@#$%^&*)")
```

#### Challenge 4: Matrix Diagonal Printer
Given a square list of numbers, print the elements on the primary and secondary diagonals.

```python
# Solution 4:
matrix = [
    [10, 20, 30],
    [40, 50, 60],
    [70, 80, 90]
]
n = len(matrix)

print("Primary diagonal:", [matrix[i][i] for i in range(n)])
print("Secondary diagonal:", [matrix[i][n - 1 - i] for i in range(n)])
```

#### Challenge 5: Pattern Generator
Print an ASCII pyramid of height $N$:
```
    *
   ***
  *****
 *******
```

```python
# Solution 5:
height = 5
for row in range(height):
    spaces = " " * (height - row - 1)
    stars = "*" * (2 * row + 1)
    print(spaces + stars)
```

---

## 15. Hands-On Module Activity

### 🎯 Project: Mystery Number Quest (CLI Engine)

#### Objectives
Develop an interactive CLI game engine featuring:
1. Difficulty selection (Easy: 1-50 [10 tries], Medium: 1-100 [7 tries], Hard: 1-500 [5 tries]).
2. Random target number generation (`random.randint`).
3. Feedback loop: "Too High", "Too Low", or "Congratulations".
4. History tracker of all previous guesses in the session.
5. Score multiplier based on remaining guesses and chosen difficulty.
6. Option to replay without restarting the program.

#### Implementation
Create a file named `number_quest.py`:

```python
# number_quest.py
"""
Mystery Number Quest
Module 02 Hands-On Project
"""
import random

DIFFICULTY_SETTINGS = {
    "1": {"name": "Easy", "max_val": 50, "max_attempts": 10, "score_mult": 1.0},
    "2": {"name": "Medium", "max_val": 100, "max_attempts": 7, "score_mult": 2.5},
    "3": {"name": "Hard", "max_val": 500, "max_attempts": 5, "score_mult": 5.0},
}

def play_game():
    print("\n" + "=" * 50)
    print("         🎯 MYSTERY NUMBER QUEST 🎯         ")
    print("=" * 50)
    print("Choose Difficulty:")
    print("  1. Easy (1 to 50, 10 Attempts)")
    print("  2. Medium (1 to 100, 7 Attempts)")
    print("  3. Hard (1 to 500, 5 Attempts)")

    choice = input("Select (1/2/3): ").strip()
    if choice not in DIFFICULTY_SETTINGS:
        print("Invalid choice, defaulting to Medium.")
        choice = "2"

    config = DIFFICULTY_SETTINGS[choice]
    secret = random.randint(1, config["max_val"])
    max_tries = config["max_attempts"]
    history = []

    print(f"\nGame started on {config['name']} mode!")
    print(f"Guess the number between 1 and {config['max_val']}.\n")

    for attempt in range(1, max_tries + 1):
        remaining = max_tries - attempt + 1
        guess_raw = input(f"[Attempt {attempt}/{max_tries} | {remaining} left] Enter guess: ").strip()

        if not guess_raw.isdigit():
            print("❌ Invalid input! Please enter an integer.")
            continue

        guess = int(guess_raw)
        history.append(guess)

        if guess == secret:
            score = int((remaining / max_tries) * 1000 * config["score_mult"])
            print("\n🎉 BINGO! YOU FOUND THE SECRET NUMBER!")
            print(f"Target: {secret}")
            print(f"Total Attempts Used: {attempt}")
            print(f"Guess History: {history}")
            print(f"🏆 Final Score: {score} points!\n")
            break
        elif guess < secret:
            print(f"🔼 Too low! The secret number is HIGHER than {guess}.")
        else:
            print(f"🔽 Too high! The secret number is LOWER than {guess}.")
    else:
        print("\n💀 GAME OVER! You ran out of attempts.")
        print(f"The secret number was: {secret}")
        print(f"Your guesses: {history}\n")

def main():
    while True:
        play_game()
        replay = input("Play again? (y/n): ").strip().lower()
        if replay != "y":
            print("\nThanks for playing Mystery Number Quest! Goodbye 👋")
            break

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python number_quest.py
```

---
**Next Step:** Proceed to [**Module 03: Data Structures & Collections**](../03-data-structures/README.md) to explore Lists, Tuples, Dictionaries, and Sets!
