# Module 06: Object-Oriented Programming (OOP)

Object-Oriented Programming (OOP) is a paradigm centered around objects that bundle state (attributes) and behavior (methods). In this module, you will learn how to design robust, modular, and maintainable systems using classes, inheritance, encapsulation, polymorphism, abstract base classes, dunder methods, and dataclasses.

---

## 📖 Table of Contents
1. [Core Principles of OOP](#1-core-principles-of-oop)
2. [Classes, Instances & The `__init__` Constructor](#2-classes-instances--the-__init__-constructor)
3. [Instance Attributes vs Class Attributes](#3-instance-attributes-vs-class-attributes)
4. [Method Varieties: Instance, `@classmethod`, and `@staticmethod`](#4-method-varieties-instance-classmethod-and-staticmethod)
5. [Encapsulation & Access Modifiers](#5-encapsulation--access-modifiers)
6. [Properties & Managed Attributes (`@property`)](#6-properties--managed-attributes-property)
7. [Inheritance & The `super()` Function](#7-inheritance--the-super-function)
8. [Multiple Inheritance & Method Resolution Order (MRO)](#8-multiple-inheritance--method-resolution-order-mro)
9. [Polymorphism & Duck Typing](#9-polymorphism--duck-typing)
10. [Abstract Base Classes (ABCs)](#10-abstract-base-classes-abcs)
11. [Dunder (Magic) Methods In Depth](#11-dunder-magic-methods-in-depth)
12. [Modern Python Dataclasses (`@dataclass`)](#12-modern-python-dataclasses-dataclass)
13. [Practice Questions](#13-practice-questions)
14. [Hands-On Module Activity](#14-hands-on-module-activity)

---

## 1. Core Principles of OOP

1. **Encapsulation**: Bundling data with the methods that operate on it, restricting direct access to internal details.
2. **Abstraction**: Exposing clean interfaces while hiding complex internal implementations.
3. **Inheritance**: Creating specialized child classes that inherit and extend the attributes and behaviors of parent classes.
4. **Polymorphism**: Treating different objects through a common interface without needing to know their specific concrete types.

---

## 2. Classes, Instances & The `__init__` Constructor

A class is a blueprint; an instance is a concrete object created from that blueprint.

```python
class Server:
    def __init__(self, hostname: str, ip: str):
        # Instance attributes
        self.hostname = hostname
        self.ip = ip
        self.is_running = False

    def start(self):
        self.is_running = True
        print(f"Server {self.hostname} ({self.ip}) is now ONLINE.")

# Creating instances
srv1 = Server("web-prod-01", "192.168.1.10")
srv1.start()
```

---

## 3. Instance Attributes vs Class Attributes

- **Instance Attributes**: Unique to each instance, stored in `self.__dict__`.
- **Class Attributes**: Shared across all instances of the class.

```python
class DatabaseConnection:
    # Class attribute (shared)
    connection_pool_limit = 10

    def __init__(self, database_name: str):
        # Instance attribute (unique)
        self.database_name = database_name

db1 = DatabaseConnection("users_db")
db2 = DatabaseConnection("orders_db")

print(db1.connection_pool_limit) # 10
DatabaseConnection.connection_pool_limit = 25
print(db2.connection_pool_limit) # 25
```

---

## 4. Method Varieties: Instance, `@classmethod`, and `@staticmethod`

```python
class DateUtil:
    calendar_type = "Gregorian"

    def __init__(self, year: int, month: int, day: int):
        self.year = year
        self.month = month
        self.day = day

    # 1. Instance method: Operates on an instance ('self')
    def format_iso(self) -> str:
        return f"{self.year:04d}-{self.month:02d}-{self.day:02d}"

    # 2. Class method: Operates on the class ('cls'), often used as alternative constructor
    @classmethod
    def from_string(cls, date_str: str):
        year, month, day = map(int, date_str.split("-"))
        return cls(year, month, day)

    # 3. Static method: Independent utility function, bound to the class namespace
    @staticmethod
    def is_leap_year(year: int) -> bool:
        return year % 4 == 0 and (year % 100 != 0 or year % 400 == 0)

# Calling alternative constructor
d = DateUtil.from_string("2026-10-09")
print(d.format_iso()) # "2026-10-09"
print(DateUtil.is_leap_year(2024)) # True
```

---

## 5. Encapsulation & Access Modifiers

Python relies on naming conventions to signal visibility:
- **Public**: `attribute` (open for public access).
- **Protected**: `_attribute` (intended for internal use by the class and its subclasses).
- **Private**: `__attribute` (triggers name mangling: transformed to `_ClassName__attribute` to prevent accidental subclass override).

```python
class BankVault:
    def __init__(self, balance: float):
        self._audit_code = "SEC-001"  # Protected
        self.__balance = balance       # Private

v = BankVault(5000)
# print(v.__balance) # Raises AttributeError!
print(v._BankVault__balance) # 5000 (accessible via mangled name, but discouraged)
```

---

## 6. Properties & Managed Attributes (`@property`)

The `@property` decorator allows getters, setters, and deleters with natural attribute syntax while enforcing validation:

```python
class TemperatureSensor:
    def __init__(self, celsius: float):
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        """Getter for celsius"""
        return self._celsius

    @celsius.setter
    def celsius(self, value: float):
        """Setter with validation"""
        if value < -273.15:
            raise ValueError("Temperature below absolute zero is impossible!")
        self._celsius = value

    @property
    def fahrenheit(self) -> float:
        """Read-only computed property"""
        return (self._celsius * 9 / 5) + 32

sensor = TemperatureSensor(25.0)
print(sensor.fahrenheit) # 77.0
sensor.celsius = 30.0    # Calls setter transparently
```

---

## 7. Inheritance & The `super()` Function

Subclasses inherit attributes and methods from a parent class, and can override or extend them using `super()`:

```python
class Employee:
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary

    def get_annual_compensation(self) -> float:
        return self.salary

class Manager(Employee):
    def __init__(self, name: str, salary: float, bonus: float):
        super().__init__(name, salary)
        self.bonus = bonus

    # Method overriding
    def get_annual_compensation(self) -> float:
        return super().get_annual_compensation() + self.bonus

mgr = Manager("Alice", 120_000, 25_000)
print(f"{mgr.name} Total: ${mgr.get_annual_compensation():,.2f}")
```

---

## 8. Multiple Inheritance & Method Resolution Order (MRO)

Python supports multiple inheritance using the **C3 Linearization Algorithm** to resolve method lookups without ambiguity:

```python
class LoggerMixin:
    def log(self, message: str):
        print(f"[{self.__class__.__name__} LOG]: {message}")

class SerializableMixin:
    def serialize(self):
        return str(self.__dict__)

class APIEndpoint(LoggerMixin, SerializableMixin):
    def __init__(self, route: str):
        self.route = route

ep = APIEndpoint("/api/v1/users")
ep.log("Route registered.")
print(APIEndpoint.mro()) # [APIEndpoint, LoggerMixin, SerializableMixin, object]
```

---

## 9. Polymorphism & Duck Typing

*"If it walks like a duck and quacks like a duck, it's a duck."*
In Python, functions depend on the methods an object implements, not its inheritance tree.

```python
class PDFExporter:
    def export(self, data):
        return f"Rendering PDF with {len(data)} records."

class JSONExporter:
    def export(self, data):
        return f"Rendering JSON with {len(data)} records."

def generate_report(exporter, data):
    # Polymorphic call: exporter just needs an export() method
    print(exporter.export(data))

generate_report(PDFExporter(), [1, 2, 3])
generate_report(JSONExporter(), [1, 2, 3])
```

---

## 10. Abstract Base Classes (ABCs)

ABCs define interfaces that subclasses **must** implement:

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        """Process transaction"""
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        """Refund transaction"""
        pass

class StripeGateway(PaymentGateway):
    def process_payment(self, amount: float) -> bool:
        print(f"Charging ${amount} via Stripe API...")
        return True

    def refund(self, transaction_id: str) -> bool:
        print(f"Refunding {transaction_id} via Stripe...")
        return True

# gateway = PaymentGateway() # TypeError: Can't instantiate abstract class
```

---

## 11. Dunder (Magic) Methods In Depth

Dunder methods (`__method__`) allow custom classes to hook into Python's native operators and protocols.

```python
class Vector2D:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    # Informal string representation for users
    def __str__(self) -> str:
        return f"Vector({self.x}, {self.y})"

    # Formal representation for debugging / REPL
    def __repr__(self) -> str:
        return f"Vector2D(x={self.x}, y={self.y})"

    # Addition operator: v1 + v2
    def __add__(self, other: "Vector2D") -> "Vector2D":
        return Vector2D(self.x + other.x, self.y + other.y)

    # Equality operator: v1 == v2
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Vector2D):
            return False
        return self.x == other.x and self.y == other.y

    # Magnitude / absolute value: abs(v)
    def __abs__(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

v1 = Vector2D(3, 4)
v2 = Vector2D(1, 2)
print(v1 + v2)     # Vector(4.0, 6.0)
print(abs(v1))     # 5.0
```

---

## 12. Modern Python Dataclasses (`@dataclass`)

Introduced in Python 3.7, `@dataclass` generates boilerplate methods (`__init__`, `__repr__`, `__eq__`) automatically:

```python
from dataclasses import dataclass, field
from typing import List

@dataclass(order=True)
class Customer:
    id: int
    name: str = field(compare=False)
    loyalty_points: int = 0
    tags: List[str] = field(default_factory=list, compare=False)

c1 = Customer(1, "Aman", loyalty_points=150)
c2 = Customer(2, "Bob", loyalty_points=50)

print(c1) # Customer(id=1, name='Aman', loyalty_points=150, tags=[])
print(c1 > c2) # Compares by 'id'
```

---

## 13. Practice Questions

### Conceptual Questions

**Q1: What is the difference between `__str__` and `__repr__`?**
> **Answer:** `__str__` provides a human-readable string representation for display to end users (called by `str()` and `print()`). `__repr__` provides an unambiguous, detailed representation primarily for developers and debugging (ideally code that could recreate the object).

**Q2: What is the difference between `@classmethod` and `@staticmethod`?**
> **Answer:** `@classmethod` receives the class object (`cls`) as its first argument and can inspect or modify class state. `@staticmethod` receives neither `self` nor `cls`; it behaves like an ordinary function namespaced inside the class.

**Q3: How does Python's name mangling work for private attributes?**
> **Answer:** Any attribute prefixed with two leading underscores (e.g., `__secret`) is rewritten by the compiler to `_ClassName__secret` to prevent name clashes in subclasses.

**Q4: What is the role of `super()` in multiple inheritance?**
> **Answer:** `super()` follows the Method Resolution Order (MRO) computed by C3 linearization, ensuring every base class in the hierarchy is called once and in the correct order.

**Q5: What makes a class an Abstract Base Class?**
> **Answer:** Inheriting from `abc.ABC` and having at least one method decorated with `@abc.abstractmethod`. Such classes cannot be directly instantiated.

---

### Coding Challenges

#### Challenge 1: Custom Stack with Dunder Methods
Implement a `Stack` class supporting `push()`, `pop()`, `len(stack)`, `item in stack`, and bracket indexing `stack[0]`.

```python
# Solution 1:
class Stack:
    def __init__(self):
        self._items = []

    def push(self, val):
        self._items.append(val)

    def pop(self):
        if not self._items:
            raise IndexError("pop from empty stack")
        return self._items.pop()

    def __len__(self):
        return len(self._items)

    def __contains__(self, item):
        return item in self._items

    def __getitem__(self, index):
        return self._items[index]
```

#### Challenge 2: Immutable Coordinates Dataclass
Create a frozen 3D Coordinate dataclass that calculates Euclidean distance to another point.

```python
# Solution 2:
from dataclasses import dataclass
import math

@dataclass(frozen=True)
class Point3D:
    x: float
    y: float
    z: float

    def distance_to(self, other: "Point3D") -> float:
        return math.sqrt(
            (self.x - other.x)**2 + (self.y - other.y)**2 + (self.z - other.z)**2
        )
```

#### Challenge 3: Property with Type Validation
Create a `User` class with an `email` property that validates the presence of an `@` symbol and a dot `.` in the domain.

```python
# Solution 3:
class User:
    def __init__(self, email: str):
        self.email = email

    @property
    def email(self) -> str:
        return self._email

    @email.setter
    def email(self, val: str):
        if "@" not in val or "." not in val.split("@")[-1]:
            raise ValueError(f"Invalid email address: {val}")
        self._email = val
```

#### Challenge 4: Callable Function Counter
Implement a class `CallCounter` using `__call__` that wraps any function and tracks how many times it was called.

```python
# Solution 4:
class CallCounter:
    def __init__(self, func):
        self.func = func
        self.call_count = 0

    def __call__(self, *args, **kwargs):
        self.call_count += 1
        return self.func(*args, **kwargs)

@CallCounter
def greet(name):
    return f"Hello, {name}"

greet("Alice")
greet("Bob")
print(greet.call_count) # 2
```

#### Challenge 5: Singleton Pattern via `__new__`
Implement the Singleton pattern in Python ensuring only one instance ever exists.

```python
# Solution 5:
class DatabaseSingleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = DatabaseSingleton()
s2 = DatabaseSingleton()
print(s1 is s2) # True
```

---

## 14. Hands-On Module Activity

### 🎯 Project: Object-Oriented Digital Banking System

#### Objectives
Design an enterprise-style banking framework demonstrating:
1. Abstract base class `Account` with common behaviors (`deposit`, `withdraw`, transaction history).
2. `SavingsAccount` with interest rate and minimum balance restrictions.
3. `CheckingAccount` with overdraft limit and transaction fees.
4. Immutable `Transaction` dataclass.
5. `Bank` controller class managing customers and processing inter-account transfers.

#### Implementation
Create a file named `banking_system.py`:

```python
# banking_system.py
"""
Object-Oriented Digital Banking System
Module 06 Hands-On Project
"""
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime
from typing import List, Dict

@dataclass(frozen=True)
class Transaction:
    timestamp: str
    action: str
    amount: float
    balance_after: float

class Account(ABC):
    def __init__(self, account_number: str, owner: str, initial_deposit: float = 0.0):
        self.account_number = account_number
        self.owner = owner
        self._balance = initial_deposit
        self._history: List[Transaction] = []
        if initial_deposit > 0:
            self._record_txn("Initial Deposit", initial_deposit)

    @property
    def balance(self) -> float:
        return self._balance

    def _record_txn(self, action: str, amount: float):
        record = Transaction(
            timestamp=datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            action=action,
            amount=amount,
            balance_after=self._balance
        )
        self._history.append(record)

    def deposit(self, amount: float):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive.")
        self._balance += amount
        self._record_txn("Deposit", amount)
        print(f"✅ Deposited ${amount:.2f} to {self.account_number}. Balance: ${self._balance:.2f}")

    @abstractmethod
    def withdraw(self, amount: float):
        """Must be implemented by subclasses"""
        pass

    def statement(self):
        print(f"\n--- Statement for {self.owner} ({self.account_number}) ---")
        for txn in self._history:
            print(f"[{txn.timestamp}] {txn.action:<18}: ${txn.amount:>8.2f} | Balance: ${txn.balance_after:>8.2f}")

class SavingsAccount(Account):
    def __init__(self, account_number: str, owner: str, initial_deposit: float = 0.0, interest_rate: float = 0.04):
        super().__init__(account_number, owner, initial_deposit)
        self.interest_rate = interest_rate
        self.min_balance = 100.0

    def withdraw(self, amount: float):
        if amount <= 0:
            raise ValueError("Withdrawal must be positive.")
        if (self._balance - amount) < self.min_balance:
            raise ValueError(f"Withdrawal denied. Minimum balance of ${self.min_balance:.2f} required.")
        self._balance -= amount
        self._record_txn("Withdrawal", amount)
        print(f"✅ Withdrew ${amount:.2f}. Balance: ${self._balance:.2f}")

    def apply_interest(self):
        earned = self._balance * self.interest_rate
        self._balance += earned
        self._record_txn(f"Interest ({self.interest_rate*100:.1f}%)", earned)
        print(f"💰 Applied interest: +${earned:.2f}. Balance: ${self._balance:.2f}")

class CheckingAccount(Account):
    def __init__(self, account_number: str, owner: str, initial_deposit: float = 0.0, overdraft_limit: float = 500.0):
        super().__init__(account_number, owner, initial_deposit)
        self.overdraft_limit = overdraft_limit

    def withdraw(self, amount: float):
        if amount <= 0:
            raise ValueError("Withdrawal must be positive.")
        if (self._balance - amount) < -self.overdraft_limit:
            raise ValueError(f"Exceeds overdraft limit (${self.overdraft_limit:.2f}).")
        self._balance -= amount
        self._record_txn("Withdrawal", amount)
        print(f"✅ Withdrew ${amount:.2f}. Balance: ${self._balance:.2f}")

class Bank:
    def __init__(self, name: str):
        self.name = name
        self.accounts: Dict[str, Account] = {}

    def register_account(self, account: Account):
        self.accounts[account.account_number] = account
        print(f"🏦 Registered account {account.account_number} for {account.owner}")

    def transfer(self, from_acc: str, to_acc: str, amount: float):
        if from_acc not in self.accounts or to_acc not in self.accounts:
            raise KeyError("One or both account numbers not found.")
        src = self.accounts[from_acc]
        dest = self.accounts[to_acc]
        src.withdraw(amount)
        dest.deposit(amount)
        print(f"🔁 Transferred ${amount:.2f} from {from_acc} to {to_acc} successfully.")

def main():
    bank = Bank("Apex National Bank")

    sa = SavingsAccount("SA-1001", "Alice Johnson", initial_deposit=1000.0)
    ca = CheckingAccount("CA-2002", "Bob Smith", initial_deposit=200.0)

    bank.register_account(sa)
    bank.register_account(ca)

    print("\n--- Performing Transactions ---")
    sa.deposit(500)
    sa.apply_interest()

    ca.withdraw(400) # Overdraft allowed

    bank.transfer("SA-1001", "CA-2002", 300)

    sa.statement()
    ca.statement()

if __name__ == "__main__":
    main()
```

Run and test your program:
```bash
python banking_system.py
```

---
**Next Step:** Proceed to [**Module 07: Advanced Python & Metaprogramming**](../07-advanced-python/README.md)!
