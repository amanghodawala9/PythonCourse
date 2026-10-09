# Module 10: Real-World Capstone Projects

Congratulations on reaching the final module of the **Python Mastery** curriculum! In this module, you will synthesize everything you have mastered—from core syntax and OOP to concurrency, testing, and architecture—by building three industry-grade capstone projects.

---

## 📖 Table of Contents
1. [Architecture & Engineering Mindset](#1-architecture--engineering-mindset)
2. [Capstone 1: TaskMaster CLI (SQLite & Repository Pattern)](#2-capstone-1-taskmaster-cli-sqlite--repository-pattern)
3. [Capstone 2: FastFlow Async ETL Pipeline](#3-capstone-2-fastflow-async-etl-pipeline)
4. [Capstone 3: MicroPy Web Framework (From Scratch)](#4-capstone-3-micropy-web-framework-from-scratch)
5. [Architecture & Design Review Questions](#5-architecture--design-review-questions)
6. [Final Capstone Challenge](#6-final-capstone-challenge)
7. [The Python Mastery Graduation Roadmap](#7-the-python-mastery-graduation-roadmap)

---

## 1. Architecture & Engineering Mindset

Writing production-grade Python involves:
- **Separation of Concerns (SoC)**: Keeping business logic, data access, and presentation layers decoupled.
- **Fail Fast, Fail Loudly**: Validating data early and avoiding silent failures.
- **Defensive Programming**: Using type hints, context managers, and custom exceptions.
- **Testability**: Structuring code with dependency injection so units can be verified in isolation.

---

## 2. Capstone 1: TaskMaster CLI (SQLite & Repository Pattern)

A robust command-line task and project manager backed by SQLite.

### Architectural Overview
- **Domain Model**: `Task` dataclass.
- **Repository Layer**: `TaskRepository` interface wrapping `sqlite3`.
- **Service Layer**: Business rules (validation, deadlines, statuses).
- **CLI Interface**: Interactive user menu and formatting.

### Complete Implementation
Create a file named `task_master.py`:

```python
# task_master.py
"""
TaskMaster CLI: SQLite-backed Task Management
Module 10 Capstone 1
"""
import sqlite3
from dataclasses import dataclass
from datetime import datetime
from typing import List, Optional

@dataclass
class Task:
    id: Optional[int]
    title: str
    category: str
    priority: str # LOW, MEDIUM, HIGH
    is_completed: bool = False
    created_at: str = ""

class TaskRepository:
    def __init__(self, db_path: str = "tasks.db"):
        self.db_path = db_path
        self._init_db()

    def _get_connection(self):
        return sqlite3.connect(self.db_path)

    def _init_db(self):
        with self._get_connection() as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS tasks (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT NOT NULL,
                    category TEXT NOT NULL,
                    priority TEXT NOT NULL,
                    is_completed BOOLEAN NOT NULL CHECK (is_completed IN (0, 1)),
                    created_at TEXT NOT NULL
                )
            """)

    def add(self, task: Task) -> int:
        now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with self._get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(
                "INSERT INTO tasks (title, category, priority, is_completed, created_at) VALUES (?, ?, ?, ?, ?)",
                (task.title, task.category, task.priority, int(task.is_completed), now)
            )
            return cursor.lastrowid

    def list_all(self, completed: Optional[bool] = None) -> List[Task]:
        query = "SELECT id, title, category, priority, is_completed, created_at FROM tasks"
        params = []
        if completed is not None:
            query += " WHERE is_completed = ?"
            params.append(int(completed))
        query += " ORDER BY id DESC"

        with self._get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            rows = cursor.fetchall()

        return [
            Task(id=r[0], title=r[1], category=r[2], priority=r[3], is_completed=bool(r[4]), created_at=r[5])
            for r in rows
        ]

    def complete(self, task_id: int) -> bool:
        with self._get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute("UPDATE tasks SET is_completed = 1 WHERE id = ?", (task_id,))
            return cursor.rowcount > 0

    def delete(self, task_id: int) -> bool:
        with self._get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
            return cursor.rowcount > 0

def render_table(tasks: List[Task]):
    if not tasks:
        print("\n📭 No tasks found.")
        return
    print("\n" + "=" * 75)
    print(f"{'ID':<5} {'Status':<10} {'Priority':<10} {'Category':<15} {'Title'}")
    print("-" * 75)
    for t in tasks:
        status = "✅ Done" if t.is_completed else "⏳ Pending"
        print(f"{t.id:<5} {status:<10} {t.priority:<10} {t.category:<15} {t.title}")
    print("=" * 75)

def main():
    repo = TaskRepository()
    print("📋 TaskMaster CLI Initialized.")

    while True:
        print("\nCOMMANDS: [1] List [2] Add [3] Complete [4] Delete [5] Exit")
        cmd = input("Select command: ").strip()

        if cmd == "1":
            filter_choice = input("Filter: (a)ll, (p)ending, (c)ompleted? [default: a]: ").strip().lower()
            comp_filter = True if filter_choice == "c" else False if filter_choice == "p" else None
            render_table(repo.list_all(comp_filter))
        elif cmd == "2":
            title = input("Task title: ").strip()
            cat = input("Category (Work, Personal, Study): ").strip().title() or "General"
            priority = input("Priority (LOW, MEDIUM, HIGH): ").strip().upper()
            if priority not in {"LOW", "MEDIUM", "HIGH"}:
                priority = "MEDIUM"
            new_id = repo.add(Task(None, title, cat, priority))
            print(f"✨ Task #{new_id} added successfully.")
        elif cmd == "3":
            try:
                tid = int(input("Task ID to complete: "))
                if repo.complete(tid):
                    print(f"🎉 Task #{tid} marked as complete!")
                else:
                    print(f"❌ Task #{tid} not found.")
            except ValueError:
                print("Invalid ID.")
        elif cmd == "4":
            try:
                tid = int(input("Task ID to delete: "))
                if repo.delete(tid):
                    print(f"🗑️ Task #{tid} deleted.")
                else:
                    print(f"❌ Task #{tid} not found.")
            except ValueError:
                print("Invalid ID.")
        elif cmd == "5":
            print("Goodbye!")
            break

if __name__ == "__main__":
    main()
```

---

## 3. Capstone 2: FastFlow Async ETL Pipeline

An event-driven, asynchronous ETL (Extract-Transform-Load) streaming pipeline processing incoming JSON telemetry records using `asyncio.Queue` and structured concurrency.

### Complete Implementation
Create a file named `async_pipeline.py`:

```python
# async_pipeline.py
"""
FastFlow Async ETL Pipeline
Module 10 Capstone 2
"""
import asyncio
import json
import random
import time
from dataclasses import dataclass
from typing import Dict, Any

@dataclass
class TelemetryRecord:
    device_id: str
    temperature: float
    humidity: float
    timestamp: float

async def extractor(queue: asyncio.Queue, total_records: int):
    """Producer: Ingests raw telemetry events."""
    print("📡 [Extractor] Ingestion stream started...")
    for i in range(1, total_records + 1):
        await asyncio.sleep(random.uniform(0.05, 0.15))
        payload = {
            "device_id": f"SENSOR-{(i % 3) + 1:02d}",
            "temp": round(random.uniform(18.0, 36.0), 2),
            "humidity": round(random.uniform(30.0, 80.0), 2),
            "ts": time.time()
        }
        await queue.put(payload)
        print(f"📥 [Extractor] Ingested record #{i} from {payload['device_id']}")

    # Signal completion
    await queue.put(None)

async def transformer(in_queue: asyncio.Queue, out_queue: asyncio.Queue):
    """Transformer: Validates and enriches records."""
    print("⚙️ [Transformer] Processing engine active...")
    while True:
        raw = await in_queue.get()
        if raw is None:
            await out_queue.put(None)
            in_queue.task_done()
            break

        # Transformation logic
        is_alert = raw["temp"] > 32.0
        record = {
            "device": raw["device_id"],
            "temp_c": raw["temp"],
            "temp_f": round((raw["temp"] * 9/5) + 32, 2),
            "humidity_pct": raw["humidity"],
            "status": "ALERT_HIGH_TEMP" if is_alert else "NORMAL"
        }
        await out_queue.put(record)
        in_queue.task_done()

async def loader(out_queue: asyncio.Queue, storage: list):
    """Consumer: Writes transformed records to persistent store."""
    print("💾 [Loader] Storage sink active...")
    while True:
        record = await out_queue.get()
        if record is None:
            out_queue.task_done()
            break
        storage.append(record)
        status_flag = "🔥" if record["status"] == "ALERT_HIGH_TEMP" else "✅"
        print(f"  {status_flag} [Loader] Saved: {record['device']} | {record['temp_c']}°C ({record['temp_f']}°F) -> {record['status']}")
        out_queue.task_done()

async def main():
    raw_queue = asyncio.Queue(maxsize=10)
    transformed_queue = asyncio.Queue(maxsize=10)
    persisted_records = []

    start = time.perf_counter()

    # Structured concurrency
    async with asyncio.TaskGroup() as tg:
        tg.create_task(extractor(raw_queue, total_records=10))
        tg.create_task(transformer(raw_queue, transformed_queue))
        tg.create_task(loader(transformed_queue, persisted_records))

    elapsed = time.perf_counter() - start
    print("\n" + "=" * 60)
    print("            ETL PIPELINE EXECUTION SUMMARY            ")
    print("=" * 60)
    print(f"Total Records Streamed & Processed: {len(persisted_records)}")
    print(f"Total Execution Duration:           {elapsed:.2f} seconds")
    alerts = sum(1 for r in persisted_records if r["status"] == "ALERT_HIGH_TEMP")
    print(f"High Temperature Alerts Triggered:  {alerts}")
    print("=" * 60)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 4. Capstone 3: MicroPy Web Framework (From Scratch)

Learn how frameworks like Flask and FastAPI function under the hood by implementing a lightweight WSGI HTTP server and routing engine using Python's standard library.

### Architectural Overview
- **Request / Response**: Abstractions over WSGI environment dict and status tuples.
- **Router**: Regular-expression based URL dispatcher mapping endpoints to handler callables.
- **Middleware Pipeline**: Layered interception for request timing and headers.

### Complete Implementation
Create a file named `micropy_framework.py`:

```python
# micropy_framework.py
"""
MicroPy Web Framework
Module 10 Capstone 3
"""
import re
import json
from wsgiref.simple_server import make_server
from typing import Callable, Dict, List, Tuple

class Request:
    def __init__(self, environ: Dict):
        self.method = environ.get("REQUEST_METHOD", "GET")
        self.path = environ.get("PATH_INFO", "/")
        self.query_string = environ.get("QUERY_STRING", "")

class Response:
    def __init__(self, body: str = "", status: str = "200 OK", content_type: str = "text/html"):
        self.status = status
        self.headers = [("Content-Type", content_type)]
        self.body = body.encode("utf-8") if isinstance(body, str) else body

    @classmethod
    def json(cls, data: dict, status: str = "200 OK"):
        body = json.dumps(data)
        return cls(body=body, status=status, content_type="application/json")

class MicroPyApp:
    def __init__(self):
        self.routes: List[Tuple[re.Pattern, str, Callable]] = []
        self.middlewares: List[Callable] = []

    def route(self, path_pattern: str, methods: List[str] = None):
        if methods is None:
            methods = ["GET"]
        regex = re.compile(f"^{path_pattern}$")

        def decorator(handler: Callable):
            for method in methods:
                self.routes.append((regex, method.upper(), handler))
            return handler
        return decorator

    def use(self, middleware: Callable):
        self.middlewares.append(middleware)

    def __call__(self, environ: Dict, start_response: Callable):
        req = Request(environ)
        res = None

        # Route matching
        for pattern, method, handler in self.routes:
            match = pattern.match(req.path)
            if match and req.method == method:
                kwargs = match.groupdict()
                res = handler(req, **kwargs)
                break

        if res is None:
            res = Response.json({"error": "Route Not Found", "path": req.path}, status="404 Not Found")

        # Middleware processing
        for mw in self.middlewares:
            res = mw(req, res)

        start_response(res.status, res.headers)
        return [res.body]

# --- Demo Application Setup ---
app = MicroPyApp()

# Middleware: Add server header
def custom_headers_middleware(req: Request, res: Response) -> Response:
    res.headers.append(("X-Powered-By", "MicroPy-Framework/1.0"))
    return res

app.use(custom_headers_middleware)

# Routes
@app.route("/", methods=["GET"])
def home(req: Request) -> Response:
    return Response("<h1>Welcome to MicroPy Framework! 🚀</h1>")

@app.route("/api/hello/(?P<name>[a-zA-Z]+)", methods=["GET"])
def greet(req: Request, name: str) -> Response:
    return Response.json({"message": f"Hello, {name.capitalize()}!", "method": req.method})

@app.route("/api/stats", methods=["GET"])
def stats(req: Request) -> Response:
    return Response.json({"uptime": "99.99%", "active_routes": len(app.routes)})

def main():
    port = 8080
    print(f"🚀 MicroPy Server running at http://localhost:{port}/")
    print("Try opening:")
    print(f"  • http://localhost:{port}/")
    print(f"  • http://localhost:{port}/api/hello/developer")
    print(f"  • http://localhost:{port}/api/stats")
    
    # Run WSGI server
    server = make_server("127.0.0.1", port, app)
    # Server can be started using server.serve_forever()

if __name__ == "__main__":
    main()
```

---

## 5. Architecture & Design Review Questions

**Q1: In TaskMaster, why use the Repository Pattern rather than executing raw SQL queries directly in user-facing CLI loops?**
> **Answer:** The Repository pattern creates an abstraction layer between business logic and database persistence. This allows switching the database backend (e.g. SQLite to PostgreSQL or an in-memory mock during unit tests) without modifying user interaction code.

**Q2: In the FastFlow Async ETL pipeline, how does `asyncio.Queue` enable backpressure control?**
> **Answer:** Setting `maxsize=10` on the queues forces upstream producers/extractors to pause when queues become full (`await queue.put()` blocks) until slower downstream consumers have finished processing, preventing memory exhaustion.

**Q3: In the MicroPy framework, what allows `app` to be passed directly to `make_server`?**
> **Answer:** The `__call__` magic method turns the `MicroPyApp` instance into a callable object conforming to the WSGI specification: `callable(environ, start_response)`.

---

## 6. Final Capstone Challenge

Choose one of the three capstone architectures and extend it with the following production features:
1. **Extend TaskMaster**: Add full-text search across titles and categories, task due dates with overdue warnings, and CSV export.
2. **Extend FastFlow Pipeline**: Add persistent writeout to disk, dead-letter queues for corrupted records, and real-time anomaly alerts.
3. **Extend MicroPy Framework**: Add query parameter parsing (`?page=2&limit=10`), JSON request body parsing for `POST` requests, and basic token authentication middleware.

---

## 7. The Python Mastery Graduation Roadmap

Congratulations! You have covered:
- ✅ Python Syntax, Data Types & Formatting
- ✅ Conditionals, Loops, Pattern Matching
- ✅ Lists, Tuples, Dictionaries, Sets & Complexity
- ✅ Scopes, Closures, Functional Tools & Typing
- ✅ Files, Formats (CSV/JSON), pathlib, Exceptions
- ✅ Object-Oriented Design, ABCs, Dunders, Dataclasses
- ✅ Iterators, Generators, Decorators, Context Managers, Metaclasses
- ✅ Multithreading, Multiprocessing, Asyncio Concurrency
- ✅ Modern Tooling, `pyproject.toml`, Pytest, TDD
- ✅ Complete End-to-End Architectural Systems

### Where to Go Next:
1. **Web APIs & Services**: Explore **FastAPI**, **Django**, and **SQLAlchemy**.
2. **Data Engineering & Science**: Master **Pandas**, **Polars**, **NumPy**, and **DuckDB**.
3. **Cloud & DevOps**: Build containerized microservices with **Docker**, deploy on Kubernetes, and write CI/CD pipelines with GitHub Actions.
4. **Open Source Contribution**: Pick an open-source Python package on GitHub, review open issues, and submit your first Pull Request!
