# Module 09: Tooling, Testing & Packaging

Professional Python software development requires more than writing algorithms. It demands automated testing, code quality verification, dependency management, and reproducible packaging. In this module, you will learn modern Python tooling including **`pyproject.toml`**, **`ruff`**, **`mypy`**, **`pytest`**, mocking, and distribution standards.

---

## 📖 Table of Contents
1. [Modern Project Structure & `pyproject.toml`](#1-modern-project-structure--pyprojecttoml)
2. [Environment Management: `venv` & Next-Gen Tools (`uv`)](#2-environment-management-venv--next-gen-tools-uv)
3. [Static Analysis & Linting: `ruff` and `mypy`](#3-static-analysis--linting-ruff-and-mypy)
4. [Testing Frameworks: `unittest` vs `pytest`](#4-testing-frameworks-unittest-vs-pytest)
5. [Deep Dive into `pytest`](#5-deep-dive-into-pytest)
6. [Fixtures & Dependency Injection](#6-fixtures--dependency-injection)
7. [Parametrization & Table-Driven Tests](#7-parametrization--table-driven-tests)
8. [Mocking & Test Isolation (`unittest.mock`)](#8-mocking--test-isolation-unittestmock)
9. [Packaging and Distribution](#9-packaging-and-distribution)
10. [Practice Questions](#10-practice-questions)
11. [Hands-On Module Activity](#11-hands-on-module-activity)

---

## 1. Modern Project Structure & `pyproject.toml`

Modern Python projects adhere to the **`src` layout** and use `pyproject.toml` (PEP 518/621) as the single source of truth for dependencies, metadata, and tool configuration.

### Recommended Layout
```
my-package/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── core.py
│       └── utils.py
└── tests/
    ├── __init__.py
    ├── test_core.py
    └── test_utils.py
```

### Sample `pyproject.toml`
```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-package"
version = "0.1.0"
description = "A production-grade Python package"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.31.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-cov>=4.0.0",
    "ruff>=0.3.0",
    "mypy>=1.9.0",
]

[tool.ruff]
line-length = 88
target-version = "py310"

[tool.pytest.ini_options]
testpaths = ["tests"]
```

---

## 2. Environment Management: `venv` & Next-Gen Tools (`uv`)

### Standard `venv`
```bash
python -m venv .venv

# Activate:
# Windows PowerShell:
.\.venv\Scripts\Activate.ps1
# Linux/macOS:
source .venv/bin/activate
```

### Modern Lightning-Fast Tooling: `uv`
Astral's `uv` is written in Rust and is 10-100x faster than traditional pip:
```bash
# Create venv and install dependencies in milliseconds
uv venv
uv pip install -e ".[dev]"
```

---

## 3. Static Analysis & Linting: `ruff` and `mypy`

### Ruff: Ultra-Fast Linter and Formatter
`ruff` replaces Flake8, Black, isort, and Bandit with sub-millisecond Rust speed.

```bash
# Check for lint errors
ruff check .

# Automatically fix lint issues
ruff check --fix .

# Format code (replaces Black)
ruff format .
```

### Mypy: Static Type Checking
Mypy verifies that your type annotations are mathematically consistent without running the code:
```bash
mypy src/
```

---

## 4. Testing Frameworks: `unittest` vs `pytest`

Python includes `unittest` in the standard library (inspired by JUnit), but the industry standard is **`pytest`** due to its concise syntax, plain `assert` statements, and rich plugin ecosystem.

```python
# unittest approach (Verbose)
import unittest

class TestMath(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(2 + 2, 4)

# pytest approach (Clean & Pythonic)
def test_addition():
    assert 2 + 2 == 4
```

---

## 5. Deep Dive into `pytest`

### Running Tests
```bash
pytest
pytest -v               # Verbose mode
pytest -k "test_user"   # Run tests matching expression
pytest --cov=src        # Run test coverage
```

### Testing Expected Exceptions
```python
import pytest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def test_divide_zero_raises_error():
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)
```

---

## 6. Fixtures & Dependency Injection

Fixtures provide clean setup and teardown logic for test dependencies:

```python
import pytest

@pytest.fixture
def sample_user_db():
    # Setup
    db = {"1": "Aman", "2": "Alice"}
    yield db
    # Teardown
    db.clear()

def test_user_exists(sample_user_db):
    assert "1" in sample_user_db
    assert sample_user_db["1"] == "Aman"
```

---

## 7. Parametrization & Table-Driven Tests

Instead of writing repetitive test cases, use `@pytest.mark.parametrize`:

```python
import pytest

def is_palindrome(text: str) -> bool:
    cleaned = "".join(c.lower() for c in text if c.isalnum())
    return cleaned == cleaned[::-1]

@pytest.mark.parametrize("input_str, expected", [
    ("racecar", True),
    ("Radar", True),
    ("A man, a plan, a canal: Panama", True),
    ("python", False),
    ("", True),
])
def test_is_palindrome(input_str, expected):
    assert is_palindrome(input_str) == expected
```

---

## 8. Mocking & Test Isolation (`unittest.mock`)

Mocking isolates the unit under test from external dependencies like network requests or databases.

```python
from unittest.mock import patch, MagicMock
import requests

def get_ip_info():
    response = requests.get("https://api.ipify.org?format=json")
    return response.json()["ip"]

@patch("requests.get")
def test_get_ip_info(mock_get):
    # Setup mock response
    mock_response = MagicMock()
    mock_response.json.return_value = {"ip": "123.45.67.89"}
    mock_get.return_value = mock_response

    ip = get_ip_info()
    assert ip == "123.45.67.89"
    mock_get.assert_called_once_with("https://api.ipify.org?format=json")
```

---

## 9. Packaging and Distribution

To distribute your library as a reusable wheel package:

```bash
# 1. Install build tool
pip install build twine

# 2. Compile wheel and source distribution
python -m build

# Output generated in dist/:
# dist/my_package-0.1.0-py3-none-any.whl
# dist/my_package-0.1.0.tar.gz

# 3. Publish to PyPI / TestPyPI
twine upload --repository testpypi dist/*
```

---

## 10. Practice Questions

### Conceptual Questions

**Q1: What is the main purpose of `pyproject.toml` in modern Python?**
> **Answer:** It acts as the unified standard specification (PEP 518/621) for defining project metadata, build dependencies, tools configuration (like Ruff, pytest, coverage, black), and runtime dependencies in a single standardized configuration file.

**Q2: How does `pytest` know where to find tests?**
> **Answer:** By following naming conventions: files matching `test_*.py` or `*_test.py`, test classes named `Test*` (without `__init__`), and test functions or methods prefixed with `test_*`.

**Q3: What does the `yield` statement do inside a `@pytest.fixture`?**
> **Answer:** Code before `yield` is executed during test setup. The yielded value is injected into the test function. Code after `yield` is guaranteed to execute during teardown when the test finishes.

**Q4: What is the role of `unittest.mock.MagicMock`?**
> **Answer:** It creates a dummy mock object capable of simulating any method, attribute, or magic method call (like `__len__`, `__iter__`, `__enter__`) without requiring the actual underlying dependency.

**Q5: What is the difference between a source distribution (`.tar.gz`) and a wheel (`.whl`)?**
> **Answer:** A wheel is a pre-built binary package format that does not require compilation upon installation, making `pip install` much faster. A source distribution contains raw source files and requires a build step.

---

### Coding Challenges

#### Challenge 1: Custom Pytest Marker
Demonstrate how to register and apply a custom marker `@pytest.mark.slow` in pytest.

```python
# In conftest.py or test file:
import pytest

# pyproject.toml needs: markers = ["slow: marks tests as slow"]
@pytest.mark.slow
def test_heavy_computation():
    assert sum(range(1000)) > 0
```

#### Challenge 2: Test Temporary Directory Fixture
Write a test using pytest's built-in `tmp_path` fixture to verify creating and reading a file.

```python
# Solution 2:
def test_temp_file_write(tmp_path):
    test_file = tmp_path / "greeting.txt"
    test_file.write_text("Hello Pytest!", encoding="utf-8")
    assert test_file.read_text(encoding="utf-8") == "Hello Pytest!"
```

#### Challenge 3: Mocking Environment Variables with `monkeypatch`
Use pytest's `monkeypatch` fixture to simulate setting an API key.

```python
# Solution 3:
import os

def get_api_key():
    return os.environ.get("SECRET_API_KEY", "default")

def test_custom_api_key(monkeypatch):
    monkeypatch.setenv("SECRET_API_KEY", "secure_token_123")
    assert get_api_key() == "secure_token_123"
```

#### Challenge 4: Parametrized Phone Number Sanitizer Test
Write tests verifying that phone numbers with dashes, spaces, and brackets normalize to 10 digits.

```python
# Solution 4:
import pytest

def sanitize_phone(raw: str) -> str:
    cleaned = "".join(c for c in raw if c.isdigit())
    if len(cleaned) != 10:
        raise ValueError("Invalid phone number length")
    return cleaned

@pytest.mark.parametrize("raw, expected", [
    ("(555) 123-4567", "5551234567"),
    ("555-123-4567", "5551234567"),
    ("555 123 4567", "5551234567"),
])
def test_sanitize_phone(raw, expected):
    assert sanitize_phone(raw) == expected
```

#### Challenge 5: Testing Custom Exception Attributes
Verify that a custom exception carries expected attributes when caught.

```python
# Solution 5:
import pytest

class ValidationError(Exception):
    def __init__(self, field: str, code: int):
        self.field = field
        self.code = code
        super().__init__(f"Error in {field}")

def test_validation_error_attributes():
    with pytest.raises(ValidationError) as exc_info:
        raise ValidationError("email", 400)
    assert exc_info.value.field == "email"
    assert exc_info.value.code == 400
```

---

## 11. Hands-On Module Activity

### 🎯 Project: Test-Driven Development (TDD) Data Sanitizer Library

#### Objectives
Practice Test-Driven Development (TDD) to build a robust input validation & sanitization utility:
1. Write the test suite first (`test_sanitizer.py`).
2. Write unit tests for:
   - Email format validator.
   - Slugifier (URL friendly strings).
   - Credit card masked display (`**** **** **** 1234`).
   - Strong password validator.
3. Write the implementation in `sanitizer.py` and run tests until all pass 100%.

#### Implementation

#### Step 1: Implementation Module (`sanitizer.py`)
```python
# sanitizer.py
"""
Data Sanitizer & Validation Toolkit
Module 09 Hands-On Project
"""
import re

def validate_email(email: str) -> bool:
    """Validates basic email structure."""
    if not isinstance(email, str):
        return False
    pattern = r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$"
    return bool(re.match(pattern, email.strip()))

def slugify(text: str) -> str:
    """Converts a string into a clean, URL-safe slug."""
    if not text:
        return ""
    # Lowercase & strip
    text = text.lower().strip()
    # Remove characters other than letters, numbers, and spaces
    text = re.sub(r"[^\w\s-]", "", text)
    # Replace spaces and multiple dashes with a single dash
    return re.sub(r"[\s_-]+", "-", text).strip("-")

def mask_credit_card(card_number: str) -> str:
    """Masks credit card numbers showing only the final 4 digits."""
    digits = "".join(c for c in card_number if c.isdigit())
    if len(digits) < 13 or len(digits) > 19:
        raise ValueError("Invalid credit card length.")
    return f"{'*' * (len(digits) - 4)}{digits[-4:]}"

def validate_password_strength(password: str) -> bool:
    """Enforces min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 symbol."""
    if len(password) < 8:
        return False
    if not re.search(r"[A-Z]", password):
        return False
    if not re.search(r"[a-z]", password):
        return False
    if not re.search(r"[0-9]", password):
        return False
    if not re.search(r"[!@#$%^&*(),.?\":{}|<>]", password):
        return False
    return True
```

#### Step 2: Pytest Suite (`test_sanitizer.py`)
```python
# test_sanitizer.py
import pytest
from sanitizer import validate_email, slugify, mask_credit_card, validate_password_strength

class TestSanitizer:
    @pytest.mark.parametrize("email, expected", [
        ("user@example.com", True),
        ("alice.dev+newsletter@domain.co.uk", True),
        ("plainaddress", False),
        ("@missingusername.com", False),
        ("missingdot@domain", False),
        (None, False),
    ])
    def test_email_validation(self, email, expected):
        assert validate_email(email) == expected

    @pytest.mark.parametrize("text, expected", [
        ("Python 3.12: What's New?", "python-312-whats-new"),
        ("  Hello World!  ", "hello-world"),
        ("Spaces   and---Dashes", "spaces-and-dashes"),
        ("", ""),
    ])
    def test_slugify(self, text, expected):
        assert slugify(text) == expected

    def test_mask_credit_card_valid(self):
        result = mask_credit_card("1234-5678-9012-3456")
        assert result == "************3456"

    def test_mask_credit_card_invalid_raises_error(self):
        with pytest.raises(ValueError, match="Invalid credit card length"):
            mask_credit_card("123")

    @pytest.mark.parametrize("password, is_valid", [
        ("StrongP@ss1", True),
        ("short1!", False),       # Too short
        ("nouppercase1!", False),  # Missing uppercase
        ("NOLOWERCASE1!", False),  # Missing lowercase
        ("NoDigitsHere!", False),  # Missing digits
        ("NoSymbols123", False),   # Missing symbol
    ])
    def test_password_strength(self, password, is_valid):
        assert validate_password_strength(password) == is_valid

if __name__ == "__main__":
    pytest.main(["-v", __file__])
```

Run the tests directly:
```bash
python -m pytest test_sanitizer.py -v
```

---
**Next Step:** Proceed to [**Module 10: Real-World Capstone Projects**](../10-capstone-projects/README.md)!
