# Module 03: Data Structures & Collections

Data structures are the backbone of any software system. In this module, you will master Python's core collection types: **Lists**, **Tuples**, **Dictionaries**, and **Sets**. You will also learn about time complexities, memory models, and shallow versus deep copies.

---

## 📖 Table of Contents
1. [Collection Types Overview](#1-collection-types-overview)
2. [Lists: Dynamic Arrays in Python](#2-lists-dynamic-arrays-in-python)
3. [List Methods & Operations](#3-list-methods--operations)
4. [Tuples: Immutable Sequences](#4-tuples-immutable-sequences)
5. [Dictionaries: Key-Value Hash Maps](#5-dictionaries-key-value-hash-maps)
6. [Dictionary Methods & Idioms](#6-dictionary-methods--idioms)
7. [Sets & Frozensets: Unique Mathematical Collections](#7-sets--frozensets-unique-mathematical-collections)
8. [Time Complexity Reference (Big-O)](#8-time-complexity-reference-big-o)
9. [Shallow Copy vs Deep Copy](#9-shallow-copy-vs-deep-copy)
10. [Nested Data Structures](#10-nested-data-structures)
11. [Practice Questions](#11-practice-questions)
12. [Hands-On Module Activity](#12-hands-on-module-activity)

---

## 1. Collection Types Overview

| Structure | Syntax | Ordered | Mutable | Duplicates | Underlying Implementation |
|-----------|--------|---------|---------|------------|---------------------------|
| **List** | `[1, 2, 3]` | Yes | Yes | Yes | Dynamic Array of Pointers |
| **Tuple** | `(1, 2, 3)` | Yes | No | Yes | Fixed-size Array |
| **Dictionary** | `{"a": 1}` | Yes (3.7+) | Yes (Keys unique) | Keys: No, Values: Yes | Hash Table |
| **Set** | `{1, 2, 3}` | No | Yes | No | Hash Table (Keys only) |

---

## 2. Lists: Dynamic Arrays in Python

A Python list is a contiguous dynamic array of references to objects.

### Creation and Indexing
```python
fruits = ["apple", "banana", "cherry", "date"]

# Positive and Negative Indexing
print(fruits[0])   # 'apple'
print(fruits[-1])  # 'date'

# Slicing: [start:stop:step]
print(fruits[1:3])    # ['banana', 'cherry']
print(fruits[::-1])   # ['date', 'cherry', 'banana', 'apple']
```

### Mutability
Lists can be modified without altering their memory identity:
```python
numbers = [10, 20, 30]
numbers[1] = 99
print(numbers) # [10, 99, 30]
```

---

## 3. List Methods & Operations

```python
tech = ["Python", "Rust"]

# Adding elements
tech.append("Go")               # Add to end: ['Python', 'Rust', 'Go']
tech.insert(1, "TypeScript")    # Insert at index: ['Python', 'TypeScript', 'Rust', 'Go']
tech.extend(["C++", "Kotlin"])  # Extend by iterable

# Removing elements
tech.remove("TypeScript")       # Removes first matching value
popped_item = tech.pop()        # Removes & returns last item ('Kotlin')
popped_first = tech.pop(0)      # Removes & returns item at index 0

# Searching and Counting
idx = tech.index("Rust")        # Index of first match
cnt = tech.count("Rust")        # Number of occurrences

# Sorting and Reversing (in-place)
nums = [42, 11, 88, 3]
nums.sort()                     # In-place ascending: [3, 11, 42, 88]
nums.sort(reverse=True)         # In-place descending: [88, 42, 11, 3]

# Non-destructive sorting:
sorted_nums = sorted(nums)      # Returns a new list, original unmodified
```

---

## 4. Tuples: Immutable Sequences

Tuples cannot be modified after creation. They are faster, consume less memory than lists, and are hashable (can be used as dictionary keys if all their contents are hashable).

### Packing and Unpacking
```python
# Tuple packing
coordinates = (37.7749, -122.4194)

# Unpacking
lat, lng = coordinates

# Extended unpacking with the asterisk (*) operator
record = ("Alice", 28, "Software Engineer", "San Francisco", "USA")
name, age, *details = record
print(name)     # "Alice"
print(age)      # 28
print(details)  # ['Software Engineer', 'San Francisco', 'USA']
```

---

## 5. Dictionaries: Key-Value Hash Maps

Dictionaries store mappings of unique **keys** to **values**. Keys must be **hashable** (immutable objects like strings, numbers, or tuples of immutables).

```python
user = {
    "id": 1001,
    "username": "amang",
    "is_active": True,
    "roles": ["developer", "admin"]
}

# Accessing
print(user["username"])  # "amang" (Raises KeyError if key not found)
print(user.get("email", "Not Provided")) # Safe access with fallback
```

---

## 6. Dictionary Methods & Idioms

```python
product = {"name": "Mechanical Keyboard", "price": 120, "stock": 15}

# Key, Value, and Item Iteration
for key in product.keys():
    print(key)

for value in product.values():
    print(value)

for key, value in product.items():
    print(f"{key}: {value}")

# Modifying and Updating
product["warranty_years"] = 2
product.update({"price": 110, "discount": 10})

# setdefault: Sets default if key absent, returns current/new value
views = product.setdefault("views", 0)

# Deleting
deleted_stock = product.pop("stock", 0)
```

---

## 7. Sets & Frozensets: Unique Mathematical Collections

A `set` is an unordered collection of unique, hashable items.

### Creating Sets
```python
unique_ids = {101, 102, 103, 101} # Duplicates automatically dropped: {101, 102, 103}
empty_set = set() # Note: {} creates an empty dict, not a set!
```

### Mathematical Set Operations
```python
dev_team = {"Python", "JavaScript", "Docker"}
ops_team = {"Docker", "Kubernetes", "Terraform", "Python"}

# Union (all skills):
print(dev_team | ops_team) 
# {'Python', 'JavaScript', 'Docker', 'Kubernetes', 'Terraform'}

# Intersection (shared skills):
print(dev_team & ops_team)
# {'Python', 'Docker'}

# Difference (skills only in dev_team):
print(dev_team - ops_team)
# {'JavaScript'}

# Symmetric Difference (skills in either, but not both):
print(dev_team ^ ops_team)
# {'JavaScript', 'Kubernetes', 'Terraform'}
```

### `frozenset`
An immutable set that can itself be used as a set element or dictionary key:
```python
immutable_set = frozenset([1, 2, 3])
```

---

## 8. Time Complexity Reference (Big-O)

| Operation | List | Dict | Set |
|-----------|------|------|-----|
| Index / Access by Key | $O(1)$ | $O(1)$ | N/A |
| Append / Insert at End | $O(1)$ amortized | $O(1)$ | $O(1)$ |
| Insert / Delete at Beginning | $O(N)$ (requires shifting) | N/A | N/A |
| Search / Membership (`x in s`) | $O(N)$ | $O(1)$ | $O(1)$ |
| Delete Item | $O(N)$ | $O(1)$ | $O(1)$ |

> **Performance Tip:** If you frequently check `if item in collection:`, convert your `list` to a `set` to achieve $O(1)$ lookup time!

---

## 9. Shallow Copy vs Deep Copy

```python
import copy

original = [1, [2, 3], 4]

# Assignment creates a new pointer to the exact same list
ref = original

# Shallow copy: Copies outer list, but nested objects remain shared
shallow = original.copy() # or list(original) or copy.copy(original)

# Deep copy: Recursively clones all nested objects
deep = copy.deepcopy(original)

# Demonstration:
original[1][0] = 999

print(shallow[1][0]) # 999 (affected by outer change!)
print(deep[1][0])    # 2   (isolated and unaffected!)
```

---

## 10. Nested Data Structures

In real-world applications, data is rarely flat. Working with nested trees of dictionaries and lists is essential:

```python
company = {
    "Engineering": [
        {"name": "Devin", "skills": ["Python", "C++"], "years_exp": 5},
        {"name": "Sara", "skills": ["Go", "Kubernetes"], "years_exp": 3}
    ],
    "Design": [
        {"name": "Liam", "skills": ["Figma", "CSS"], "years_exp": 4}
    ]
}

# Accessing Sara's first skill:
print(company["Engineering"][1]["skills"][0]) # 'Go'
```

---

## 11. Practice Questions

### Conceptual Questions

**Q1: Why can't a Python `list` be used as a dictionary key or set element?**
> **Answer:** Dictionary keys and set elements must be hashable, meaning their hash value must never change during their lifetime. Because lists are mutable, their contents can change, which would corrupt the internal hash bucket indexing.

**Q2: What is the average time complexity of `item in my_list` vs `item in my_set`?**
> **Answer:** `item in my_list` is $O(N)$ because it must iterate sequentially across all elements. `item in my_set` is $O(1)$ on average because it hashes the item to look up the bucket directly.

**Q3: How do you create an empty set in Python?**
> **Answer:** Using `set()`. Writing `{}` creates an empty dictionary.

**Q4: Explain the difference between `list.sort()` and `sorted(list)`.**
> **Answer:** `list.sort()` sorts the list in-place and returns `None`. `sorted(list)` creates and returns a brand-new sorted list while keeping the original unchanged.

**Q5: What happens when two different keys generate the same hash in a Python dictionary?**
> **Answer:** This is a hash collision. CPython resolves hash collisions using open addressing with quadratic probing (pseudo-random perturbation) to locate the next free slot.

---

### Coding Challenges

#### Challenge 1: Deduplicate while Preserving Order
Remove duplicate values from a list while maintaining the original insertion order (without using standard `set()` which loses order).

```python
# Solution 1:
raw = [4, 5, 2, 4, 1, 5, 3, 2]
# Using dict.fromkeys (preserves order in Python 3.7+)
deduped = list(dict.fromkeys(raw))
print(deduped) # [4, 5, 2, 1, 3]
```

#### Challenge 2: Word Frequency Counter
Count the frequency of each word in a paragraph (case-insensitive, punctuation stripped).

```python
# Solution 2:
text = "Python is great and Python is versatile! Great code leads to great software."
clean_text = "".join(c.lower() if c.isalnum() or c.isspace() else "" for c in text)
words = clean_text.split()

counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

print(counts)
```

#### Challenge 3: Invert a Dictionary
Given a dictionary mapping student names to grades, invert it to map each grade to a list of students who achieved it.

```python
# Solution 3:
grades = {"Alice": "A", "Bob": "B", "Charlie": "A", "David": "C", "Eva": "B"}
inverted = {}

for student, grade in grades.items():
    inverted.setdefault(grade, []).append(student)

print(inverted)
# {'A': ['Alice', 'Charlie'], 'B': ['Bob', 'Eva'], 'C': ['David']}
```

#### Challenge 4: Common Elements Across Multiple Lists
Find elements that appear in all three lists using set operations.

```python
# Solution 4:
list1 = [1, 2, 3, 4, 5]
list2 = [3, 4, 5, 6, 7]
list3 = [0, 3, 5, 8, 9]

common = list(set(list1) & set(list2) & set(list3))
print("Common elements:", common) # [3, 5]
```

#### Challenge 5: Flatten a Nested 2D List
Flatten `[[1, 2], [3, 4, 5], [6]]` into `[1, 2, 3, 4, 5, 6]`.

```python
# Solution 5:
matrix = [[1, 2], [3, 4, 5], [6]]
flattened = [item for sublist in matrix for item in sublist]
print(flattened) # [1, 2, 3, 4, 5, 6]
```

---

## 12. Hands-On Module Activity

### 🎯 Project: Student Gradebook & Statistical Analyzer

#### Objectives
Build a CLI-driven Gradebook system that:
1. Stores student records in a nested data structure (`dict` of student IDs mapping to student info including names, and subject scores).
2. Supports adding new students and recording subject marks.
3. Computes:
   - Each student's total, average, and letter grade (A, B, C, D, F).
   - Class-wide statistics (mean score, highest scorer, lowest scorer).
   - Subject toppers.
4. Formats and prints an attractive ASCII leaderboard table.

#### Implementation
Create a file named `gradebook.py`:

```python
# gradebook.py
"""
Student Gradebook & Statistical Analyzer
Module 03 Hands-On Project
"""

def calculate_letter_grade(score: float) -> str:
    if score >= 90: return "A"
    if score >= 80: return "B"
    if score >= 70: return "C"
    if score >= 60: return "D"
    return "F"

def display_report(gradebook: dict):
    if not gradebook:
        print("\nGradebook is currently empty.")
        return

    print("\n" + "=" * 70)
    print(f"{'STUDENT PERFORMANCE REPORT':^70}")
    print("=" * 70)
    print(f"{'ID':<8} {'Name':<20} {'Subjects':<20} {'Average':<10} {'Grade':<8}")
    print("-" * 70)

    class_averages = []

    for sid, data in gradebook.items():
        name = data["name"]
        scores = data["scores"]
        avg = sum(scores.values()) / len(scores) if scores else 0.0
        class_averages.append((name, avg))
        grade = calculate_letter_grade(avg)
        sub_count = f"{len(scores)} subjects"

        print(f"{sid:<8} {name:<20} {sub_count:<20} {avg:<10.2f} {grade:<8}")

    print("-" * 70)
    
    # Statistical Overview
    if class_averages:
        all_avgs = [a for _, a in class_averages]
        class_mean = sum(all_avgs) / len(all_avgs)
        top_student, top_score = max(class_averages, key=lambda x: x[1])
        low_student, low_score = min(class_averages, key=lambda x: x[1])

        print(f"📊 Class Overall Average: {class_mean:.2f}")
        print(f"🏆 Top Performer:         {top_student} ({top_score:.2f})")
        print(f"⚠️ Needs Improvement:     {low_student} ({low_score:.2f})")
    print("=" * 70)

def main():
    gradebook = {
        "S101": {"name": "Alice Johnson", "scores": {"Math": 95, "Physics": 88, "CS": 92}},
        "S102": {"name": "Bob Smith", "scores": {"Math": 72, "Physics": 68, "CS": 81}},
        "S103": {"name": "Clara Oswald", "scores": {"Math": 88, "Physics": 94, "CS": 96}},
        "S104": {"name": "Dan Evans", "scores": {"Math": 55, "Physics": 62, "CS": 58}},
    }

    while True:
        print("\n--- GRADEBOOK MENU ---")
        print("1. View Performance Report")
        print("2. Add New Student")
        print("3. Add/Update Subject Score")
        print("4. Exit")
        choice = input("Enter choice (1-4): ").strip()

        if choice == "1":
            display_report(gradebook)
        elif choice == "2":
            sid = input("Enter Student ID (e.g., S105): ").strip().upper()
            if sid in gradebook:
                print("❌ Student ID already exists!")
                continue
            name = input("Enter Student Name: ").strip()
            gradebook[sid] = {"name": name, "scores": {}}
            print(f"✅ Added {name} successfully.")
        elif choice == "3":
            sid = input("Enter Student ID: ").strip().upper()
            if sid not in gradebook:
                print("❌ Student ID not found!")
                continue
            subject = input("Enter Subject Name: ").strip().title()
            try:
                score = float(input(f"Enter score for {subject} (0-100): "))
                if not (0 <= score <= 100):
                    print("❌ Score must be between 0 and 100.")
                    continue
                gradebook[sid]["scores"][subject] = score
                print(f"✅ Recorded {score} in {subject} for {gradebook[sid]['name']}.")
            except ValueError:
                print("❌ Invalid numeric score!")
        elif choice == "4":
            print("Exiting Gradebook System. Goodbye!")
            break
        else:
            print("Invalid choice, please try again.")

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python gradebook.py
```

---
**Next Step:** Proceed to [**Module 04: Functions & Scope**](../04-functions-and-scope/README.md) to master reusable modular code!
