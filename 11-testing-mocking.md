# Module 11 — Testing & Mocking (pytest, unittest, Fixtures, Coverage, Property-Based Testing)

## Table of Contents

- [1. Why Test in Python? (vs TypeScript/jest)](#1-why-test-in-python-vs-typescriptjest)
- [2. pytest Deep Dive — Every Feature Exhaustive Reference](#2-pytest-deep-dive--every-feature-exhaustive-reference)
  - [2.1 Test Discovery & File Conventions](#21-test-discovery--file-conventions)
  - [2.2 Assertions — Python's Built-in assert vs TypeScript expect()](#22-assertions--pythons-built-in-assert-vs-typescript-expect)
  - [2.3 Fixtures — Dependency Injection for Tests](#23-fixtures--dependency-injection-for-tests)
  - [2.4 Parametrized Tests — @pytest.mark.parametrize](#24-parametrized-tests--pytestmarkparametrize)
  - [2.5 Markers & Skips — xfail, skip, custom markers](#25-markers--skips--xfail-skip-custom-markers)
  - [2.6 Hooks — pytest_configure, pytest_runtest_*, Custom Behavior](#26-hooks--pytestconfigure-pytestruntest--custom-behavior)
  - [2.7 Plugin Architecture & Popular Plugins](#27-plugin-architecture--popular-plugins)
- [3. unittest — The Built-in Alternative (Complete Reference)](#3-unittest--the-built-in-alternative-complete-reference)
- [4. Mocking Exhaustive Reference (unittest.mock)](#4-mocking-exhaustive-reference-unittestmock)
  - [4.1 patch() at Every Level](#41-patch-at-every-level)
  - [4.2 MagicMock, AsyncMock, PropertyMock](#42-magicmock-asyncmock-propertymock)
  - [4.3 side_effect Patterns](#43-side_effect-patterns)
  - [4.4 assert_called_with / call chains](#44-assert_called_with--call-chains)
  - [4.5 spec / spec_set](#45-spec--spec_set)
  - [4.6 Mock Context Managers & Chaining](#46-mock-context-managers--chaining)
  - [4.7 Complete TypeScript jest.mock() → Python unittest.mock() Mapping Table](#47-complete-typescript-jestmock--python-unittestmock-mapping-table)
- [5. Coverage Patterns & Configuration](#5-coverage-patterns--configuration)
- [6. Test Organization Strategies](#6-test-organization-strategies)
  - [6.1 conftest.py Hierarchy (Root → Module → Function Level)](#61-conftestpy-hierarchy-root--module--function-level)
  - [6.2 Fixtures as Classes](#62-fixtures-as-classes)
  - [6.3 Test Directory Structure Patterns](#63-test-directory-structure-patterns)
- [7. Property-Based Testing with hypothesis](#7-property-based-testing-with-hypothesis)
- [8. Performance Comparison: pytest vs unittest vs nose2](#8-performance-comparison-pytest-vs-unittest-vs-nose2)
- [9. Quizzes (25+ Questions with Answers)](#9-quizzes-25-questions-with-answers)
- [10. Exercises (20+ Problems with Solutions)](#10-exercises-20-problems-with-solutions)

---

## 1. Why Test in Python? (vs TypeScript/jest)

### TypeScript Testing Ecosystem vs Python Testing Ecosystem

| Aspect | TypeScript/jest | Python/pytest |
|--------|----------------|--------------|
| **Test runner** | jest (bundled with most TS projects) | pytest (third-party but universally adopted — like jest in the TS world!) |
| **Assertion library** | `expect(actual).toBe(expected)` built into jest | `assert actual == expected` — Python's built-in `assert` statement! No external library needed. |
| **Mocking** | `jest.fn()`, `jest.mock()` built into framework | `unittest.mock.patch()` — separate import but more powerful (mock anything) |
| **Async testing** | `async/await` or `.resolves`/`.rejects` matchers | `pytest-asyncio` plugin for `async def` tests |
| **CLI commands** | `npx jest` | `pytest tests/` |
| **Coverage tooling** | `--coverage` built into jest | `pytest-cov` plugin (separate package) |
| **Snapshot testing** | Built into jest (`toMatchSnapshot()`) | ` syrupy`, `dirty-equals`, or `instruments` as plugins |
| **Parallel testing** | jest's `--maxWorkers` | `pytest-xdist` plugin (`-n auto`) |
| **Test isolation** | Each `describe()` block is isolated | Each test function is isolated by default (no shared mutable state) |
| **Type checking tests** | Full type inference in test code | `mypy` for type-checking test files separately |
| **Fuzzy matching** | `.toContain()`, `.toMatch()` | `dirty-equals.IgnoreExtraKeys`, `==` with fuzzy packages |

### Mermaid: TypeScript vs Python Testing Architecture

```mermaid
flowchart TD
    subgraph TS_TESTING["TypeScript Testing Stack"]
        direction TB
        T1["npm install --save-dev jest @types/jest"] --> T2["jest.config.js / package.json config"]
        T2 --> T3["expect().toBe() / .toEqual() / .rejects.toThrow()"]
        T3 --> T4["jest.fn() / jest.mock() / jest.spyOn()"]
        T4 --> T5["npx jest --coverage"]
        
        style T1 fill:#3178c6,color:white
        style T2 fill:#3178c6,color:white
        style T3 fill:#3178c6,color:white
        style T4 fill:#3178c6,color:white
        style T5 fill:#3178c6,color:white
    end

    subgraph PY_TESTING["Python Testing Stack"]
        direction TB
        P1["pip install pytest pytest-cov pytest-asyncio pytest-xdist hypothesis"] --> P2["[tool.pytest] in pyproject.toml"]
        P2 --> P3["assert actual == expected (built-in!)"]
        P3 --> P4["from unittest.mock import patch, MagicMock"]
        P4 --> P5["pytest --cov=src -n auto"]
        
        style P1 fill:#3069bd,color:white
        style P2 fill:#3069bd,color:white
        style P3 fill:#3069bd,color:white
        style P4 fill:#3069bd,color:white
        style P5 fill:#3069bd,color:white
    end

    T1 -.->|"Bundled monolith\nOne package does everything"| P1
    T3 -.->|"Chained matchers: expect(x).toBe(y)"| P3
    T4 -.->|"jest.fn() / jest.mock()"| P4
    T5 -.->|"Built-in coverage flag"| P5
```

### Key Notes: Testing Philosophy

1. **pytest is the universal standard** for Python testing — like jest in TypeScript. In TypeScript you get jest bundled by default; in Python, install pytest via `pip install pytest`.

2. **Python's `assert` is built-in** — unlike TypeScript where you need an assertion library (jest.expect or chai), Python has `assert` as a language feature!

3. **unittest.mock.patch() is more powerful than jest.mock()** — it manipulates the import system directly, so you can mock anything: functions, classes, modules, attributes, even built-ins like `open()`.

4. **Python testing requires more packages for feature parity** with jest (pytest-cov, pytest-asyncio, pytest-xdist, hypothesis) but each is focused and composable.

---

## 2. pytest Deep Dive — Every Feature Exhaustive Reference

### 2.1 Test Discovery & File Conventions

pytest auto-discovers tests using these rules:

```python
# === File naming (all of these are discovered automatically) ===
test_math.py           # test_*.py pattern
math_test.py           # *_test.py pattern
tests/test_math.py     # in a 'tests/' directory
src/tests/test_api.py  # anywhere on sys.path

# === Function/method naming ===
def test_add():                    # Discovered because it starts with 'test_'
    assert add(1, 2) == 3

class TestMath:                    # Class starting with 'Test' is discovered
    def test_subtract(self):       # Method starting with 'test_' is discovered
        assert subtract(5, 3) == 2
    
    async def test_async_fetch(self):
        pytest.fail("Needs pytest-asyncio plugin")

# === Files/directories skipped by default ===
tests/__pycache__/               # Skipped (Python cache)
tests/.pytest_cache/             # Skipped (pytest's own cache)
tests/conftest.py                # NOT a test file, but loaded for fixtures/hooks
```

#### TypeScript jest File Conventions vs pytest

```typescript
// === TypeScript/jest: File conventions ===
// Files matching these patterns are discovered automatically:
// - **/__tests__/**/*.ts          (Jest default)
// - **/*.test.ts                  (common convention)
// - **/*.spec.ts                  (Bazel/TSLint convention)

// jest.config.js
module.exports = {
  testMatch: ['**/__tests__/**/*.(ts|tsx)', '**/*.test.ts'],  // Glob pattern
  testPathIgnorePatterns: ['/node_modules/'],                  // Skip node_modules
  roots: ['<rootDir>/src'],                                  // Search in src/
};

// Usage: npx jest                    // Run all tests
// Usage: npx jest math.test.ts       // Run specific file
// Usage: npx jest -t "adds numbers"  // Run tests matching name
```

```python
# === Python/pytest: File conventions ===
# Files matching these patterns are discovered automatically:
import pytest

# pyproject.toml configuration (equivalent to jest.config.js)
# [tool.pytest.ini_options]
# testpaths = ["tests"]                    # Search in tests/ directory
# python_files = ["test_*.py", "*_test.py"]  # File naming patterns
# python_classes = ["Test*"]               # Class naming pattern
# python_functions = ["test_*"]            # Function naming pattern

# Run all tests:     pytest
# Run specific file: pytest tests/test_math.py
# Match by name:     pytest -k "add"           (matches test names containing 'add')
# Match by marker:   pytest -m "slow"          (runs only @pytest.mark.slow tests)
# Fail fast on first: pytest --maxfail=1       (stop after N failures)
# Verbosity:         pytest -v or pytest -vv   (verbose/very verbose output)
```

#### Mermaid: pytest Test Discovery Flow

```mermaid
flowchart TD
    A["pytest invoked\n$ pytest"] --> B["Scan for config files\npyproject.toml / pytest.ini / setup.cfg"]
    B --> C["Load testpaths from config\ndefaults to current directory"]
    C --> D["Find all .py files matching\npython_files patterns (test_*.py, *_test.py)"]
    D --> E["For each file:\n1. Import the module\n2. Find classes starting with 'Test'\n3. Find functions/methods starting with 'test_'"]
    E --> F["Build test collection\nEach test = test function + fixtures needed"]
    F --> G{"Any errors during import?\nImportError, SyntaxError, etc."}
    G -->|Yes| H["Report collection error\ntest is NOT discovered"]
    G -->|No| I["Execute tests in order\nFixture setup → test body → fixture teardown"]
    I --> J["Print results\npassed / failed / skipped / xfailed"]

    style A fill:#3069bd,color:white
    style B fill:#3069bd,color:white
    style D fill:#3069bd,color:white
    style F fill:#3069bd,color:white
    style I fill:#3069bd,color:white
    style J fill:#3069bd,color:white
```

### 2.2 Assertions — Python's Built-in assert vs TypeScript expect()

| Assertion | Python `assert` | TypeScript/jest expect() |
|-----------|----------------|-------------------------|
| Equal values | `assert a == b` | `expect(a).toBe(b)` |
| Equal (deep) | `assert a == b` (same — Python == is deep) | `expect(a).toEqual(b)` |
| Is None | `assert a is None` | `expect(a).toBeNull()` or `expect(a).toBeFalsy()` |
| Is not None | `assert a is not None` | `expect(a).toBeDefined()` |
| Is truthy | `assert bool(a)` or `assert a` | `expect(a).toBeTruthy()` |
| Is falsy | `assert not a` | `expect(a).toBeFalsy()` |
| Contains in list/dict | `"key" in my_dict` / `"val" in lst` | `expect(my_dict).toHaveProperty("key")` |
| List contains | `assert "x" in lst` | `expect(lst).toContain("x")` |
| Raises exception | `with pytest.raises(ValueError, match="msg"): func()` | `await expect(func()).rejects.toThrow("msg")` |
| String match regex | `import re; assert re.search(r"\d+", text)` | `expect(text).toMatch(/\d+/)` |
| Approximate float | `assert abs(a - b) < 1e-6` or `assert pytest.approx(a, rel=1e-6) == b` | `expect(a).toBeCloseTo(b, delta)` |
| Instance check | `assert isinstance(obj, MyClass)` | `expect(obj).toBeInstanceOf(MyClass)` |
| Attribute exists | `assert hasattr(obj, "attr")` | `expect(obj).toHaveProperty("attr")` |
| Custom error message | `assert a == b, f"Expected {b}, got {a}"` | `expect(a).toBe(b)` (pytest auto-shows diff!) |

#### Comprehensive Assertion Examples

```python
# === Python: Every assertion pattern with examples ===
import pytest
import re

def test_basic_assertions():
    # Equality
    assert 1 + 1 == 2
    assert {"a": 1} == {"a": 1}           # Deep equality (same as Object.is for primitives)
    
    # None checks (CRITICAL — never use == None, always use is None)
    result = None
    assert result is None                  # Like expect(result).toBeNull()
    assert result is not None              # Like expect(result).toBeDefined()
    
    # Truthy/falsy
    assert bool([1, 2, 3])                 # List with items is truthy
    assert not []                          # Empty list is falsy
    
    # Membership
    assert "key" in {"key": "value"}       # Dictionary key check
    assert "x" in ["a", "b", "x"]          # List membership
    assert "hello" in "hello world"          # Substring check
    
    # String operations
    assert "hello".startswith("he")
    assert "hello".endswith("ld")
    assert "hello".islower()
    assert "HELLO".isupper()
    
    # Numeric comparisons
    assert 5 > 3
    assert 10 >= 10
    assert -1 < 0
    assert abs(3.14159 - 3.1416) < 1e-4   # Float comparison
    
    # Instance checks
    class MyClass: pass
    obj = MyClass()
    assert isinstance(obj, MyClass)
    
    # Attribute checks
    assert hasattr(obj, "__class__")

def test_assertion_errors_with_custom_messages():
    """Python assertions show DIFFS automatically in pytest output!"""
    actual = {"name": "Alice", "age": 30}
    expected = {"name": "Bob", "age": 30}
    
    # pytest shows a beautiful colored diff:
    # E       Left:  {'name': 'Alice', 'age': 30}
    # E       Right: {'name': 'Bob', 'age': 30}
    assert actual == expected, f"Expected {expected}, got {actual}"

# === TypeScript equivalent for every assertion above ===
/*
// TypeScript/jest equivalents:
expect(1 + 1).toBe(2);
expect({a: 1}).toEqual({a: 1});           // Deep equality
expect(result).toBeNull();                  // null check
expect(result).toBeDefined();               // not null/undefined
expect([1, 2, 3]).toBeTruthy();
expect([]).toBeFalsy();
expect(obj).toHaveProperty("key");
expect(arr).toContain("x");
expect(str).toContain("hello");
expect(str).toMatch(/he/);
expect(value).toBeCloseTo(3.1416, 0.001);
expect(obj).toBeInstanceOf(MyClass);
expect(() => func()).toThrow("error message");

// Jest also has:
expect(value).not.toBe(...);                // Negated assertions
expect.arrayContaining([...]);               // Partial array match
expect.objectContaining({...});              // Partial object match
*/
```

#### pytest.raises vs TypeScript expect().rejects

```python
# === Python: Exception assertions with pytest.raises ===
import pytest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def test_exception_assertions():
    # Simple exception check
    with pytest.raises(ValueError):
        divide(1, 0)
    
    # With match pattern (regex!)
    with pytest.raises(ValueError, match="Cannot divide"):
        divide(1, 0)
    
    # With additional attributes check
    err = pytest.raises(TypeError, lambda: int("not-a-number"))
    assert "not-a-number" in str(err.value)
    
    # Async exception (with pytest-asyncio)
    # async def test_async_exception():
    #     with pytest.raises(ConnectionError):
    #         await some_async_function()

# === TypeScript equivalent ===
/*
// TypeScript/jest:
await expect(divide(1, 0)).rejects.toThrow(ValueError);
await expect(divide(1, 0)).rejects.toThrow("Cannot divide by zero");

// Jest also supports regex matchers:
await expect(divide(1, 0)).rejects.toThrow(/Cannot divide/);
*/
```

### 2.3 Fixtures — Dependency Injection for Tests

#### pytest Fixture Complete Lifecycle & Features

```python
# === COMPLETE FIXTURE REFERENCE ===
import pytest
import os

# --- Basic fixture (simplest form) ---
@pytest.fixture
def sample_user():
    """Returns a basic user dict."""
    return {"name": "Alice", "age": 30, "email": "alice@example.com"}

def test_basic_fixture(sample_user):
    assert sample_user["name"] == "Alice"


# --- Fixture with yield (setup/teardown pattern) ---
@pytest.fixture
def temp_config():
    """Fixture that sets up and tears down a config file."""
    config_path = "/tmp/test_config.json"
    # SETUP: Create the resource
    with open(config_path, "w") as f:
        f.write('{"debug": true}')
    
    yield config_path                          # Give the value to the test
    
    # TEARDOWN: Clean up after the test
    os.remove(config_path)


# --- Fixture that depends on another fixture ---
@pytest.fixture
def admin_user(sample_user):
    """Derives from sample_user, adding admin role."""
    user = sample_user.copy()                  # Like {...sampleUser} in JS spread
    user["role"] = "admin"
    return user


def test_admin(admin_user):
    assert admin_user["role"] == "admin"       # Fixture chain works automatically!


# --- Autouse fixture (runs before every test automatically) ---
@pytest.fixture(autouse=True)
def cleanup_env():
    """Like Jest's beforeEach() that runs for EVERY test in this scope."""
    original_env = os.environ.copy()           # Save current env vars
    yield
    os.environ.clear()
    os.environ.update(original_env)            # Restore after each test


# --- Fixture with scope parameter (lifecycle control) ---
@pytest.fixture(scope="session")        # Created once for entire test run (slowest setup)
def slow_database():
    """Expensive to set up — shared across ALL tests in the session!"""
    db = create_expensive_db()             # Setup only once per pytest invocation
    yield db
    db.shutdown()

@pytest.fixture(scope="module")         # Created once per module (file)
def module_config():
    """Shared within a single test file."""
    yield load_config()

@pytest.fixture(scope="class")          # Python 3.12+ only! Created once per class
def class_database():
    """Shared within a test class."""
    db = create_db()
    yield db

@pytest.fixture(scope="function")       # Default — created for each test function
def per_test_data():
    """Fresh data for every test (default scope)."""
    yield generate_random_data()


# --- Parametrized fixture ---
@pytest.fixture(params=["chrome", "firefox", "safari"])
def browser(request):
    """Runs the test 3 times — once per browser type."""
    return create_browser(request.param)   # request.param = "chrome" / "firefox" / "safari"


# --- Fixture that returns a factory function ---
@pytest.fixture
def user_factory():
    """Fixture that returns a function for creating users on demand."""
    def _create(name, age=18):
        return {"name": name, "age": age}
    return _create

def test_with_factory(user_factory):
    u1 = user_factory("Alice")   # Create multiple instances in one test
    u2 = user_factory("Bob")
    assert len([u1, u2]) == 2
```

#### Mermaid: pytest Fixture Lifecycle

```mermaid
flowchart TD
    subgraph COLLECT["Test Collection Phase"]
        C1["pytest discovers test functions\nthat request fixtures as parameters"] --> C2["Resolves fixture dependency graph\nfixture_a → fixture_b (if a depends on b)"]
        C2 --> C3{"What is the scope?"}
        C3 -->|session| C4["Create once for entire run"]
        C3 -->|module| C5["Create once per test file"]
        C3 -->|class| C6["Create once per test class"]
        C3 -->|function| C7["Create fresh for each test (DEFAULT)"]
    end

    subgraph SETUP["Setup Phase (before each test)"]
        S1["Run fixture code UP TO the yield statement\nAllocate resources: DB connections, files, mocks..."] --> S2["Yield the resource value\nto the test function as a parameter"]
    end

    subgraph TEST["Test Execution Phase"]
        T1["Test runs with all fixtures available\nas function parameters (dependency injection!)"] --> T2{"All assertions pass?"}
    end

    subgraph TEARDOWN["Teardown Phase (after each test)"]
        T3["Continue fixture code AFTER the yield statement\nRelease resources: close files, shut down services..."] --> T4["Fixture is garbage collected"]
    end

    C7 -.-> S1
    S2 -.-> T1
    T2 -->|Yes| T3
    T2 -->|No| T3
    style C1 fill:#3069bd,color:white
    style S1 fill:#e8a53b,color:black
    style S2 fill:#e8a53b,color:black
    style T1 fill:#4caf50,color:white
    style T3 fill:#f44336,color:white
```

#### Complete Fixture Patterns Comparison with TypeScript

```python
# === Pattern 1: Simple value fixture (like Jest's module-level variable) ===
@pytest.fixture
def counter():
    return {"value": 0}

def test_counter(counter):
    counter["value"] += 1
    assert counter["value"] == 1


# === Pattern 2: Setup/teardown with yield (like Jest's beforeEach/afterEach) ===
@pytest.fixture
def temp_dir(tmp_path):
    """Fixture that creates a temporary directory."""
    d = tmp_path / "test_subdir"           # tmp_path is a built-in pytest fixture!
    d.mkdir()
    return d

def test_in_temp_dir(temp_dir):
    file = temp_dir / "test.txt"
    file.write_text("hello")
    assert file.read_text() == "hello"


# === Pattern 3: Session-scoped shared resource (like Jest's once()) ===
@pytest.fixture(scope="session")
def database_pool():
    """Expensive connection pool — created ONCE for entire test suite."""
    pool = create_connection_pool(size=10)
    yield pool
    pool.close()

class TestDatabase:
    def test_query_1(self, database_pool):      # Both use SAME pool!
        results = database_pool.query("SELECT 1")
        assert results == [(1,)]

    def test_query_2(self, database_pool):      # Same pool, no re-setup!
        results = database_pool.query("SELECT 2")
        assert results == [(2,)]


# === Pattern 4: Fixture with request object (dynamic behavior) ===
@pytest.fixture
def dynamic_config(request):
    """Fixture that reads pytest configuration options at runtime."""
    config_value = request.config.getoption("--my-option")
    return load_config(config_value)

# Usage: pytest --my-option=production


# === Pattern 5: Generator fixture (complex setup/teardown) ===
@pytest.fixture
def web_server():
    """Start a test server, yield it, then stop it."""
    server = start_test_server(port=8765)
    base_url = f"http://localhost:{8765}"
    
    # Any exceptions in the test will propagate through the yield:
    try:
        yield base_url         # Give the URL to the test
    finally:
        server.stop()          # Always clean up


# === Pattern 6: Fixtures that return mocks ===
@pytest.fixture
def mock_http_client():
    """Fixture that returns a pre-configured mock."""
    from unittest.mock import MagicMock, AsyncMock
    
    client = MagicMock()
    client.get = AsyncMock(return_value={"status": "ok"})
    client.post = AsyncMock(side_effect=ValueError("POST not allowed"))
    return client
```

### 2.4 Parametrized Tests — @pytest.mark.parametrize

#### Complete Parametrization Reference

```python
import pytest

# --- Basic parametrize (single parameter) ---
@pytest.mark.parametrize("input_val", [1, 2, 3, -1, 0])
def test_positive(input_val):
    assert input_val > -2    # Runs 5 separate test cases


# --- Multiple parameters (tuple unpacking) ---
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (-1, -2, -3),
    (0, 0, 0),
    (100, 200, 300),
    (999999, 1, 1000000),
])
def test_add(a, b, expected):
    assert add(a, b) == expected


# --- Parameter names and ids for readable output ---
@pytest.mark.parametrize(
    "input_val,expected",
    [
        (2, 4),     # id="small_numbers"
        (10, 100),  # id="medium_numbers"
        (-3, 9),    # id="negative_number"
    ],
    ids=["small_numbers", "medium_numbers", "negative_number"]
)
def test_square(input_val, expected):
    assert square(input_val) == expected

# Test names in output: test_square[small_numbers], test_square[medium_numbers]...


# --- Parametrize with fixtures (fixture as one of the params) ---
@pytest.fixture(params=["alice", "bob", "charlie"])
def username(request):
    return request.param

@pytest.mark.parametrize("role", ["admin", "user", "guest"])
def test_user_roles(username, role):
    """Runs 3 × 3 = 9 test cases (all combinations of username × role)."""
    user = create_user(username, role)
    assert user["name"] == username
    assert user["role"] == role


# --- Class-level parametrize ===
class TestCalculator:
    @pytest.mark.parametrize("a,b,op,expected", [
        (1, 2, "add", 3),
        (5, 3, "sub", 2),
        (4, 3, "mul", 12),
        (10, 2, "div", 5.0),
    ])
    def test_calculate(self, a, b, op, expected):
        assert calculate(a, b, op) == expected


# --- Parametrize with fixture objects (not just primitive values) ---
@pytest.fixture(params=[
    {"engine": "sqlite"},
    {"engine": "postgresql"},
    {"engine": "mysql"},
])
def db_config(request):
    return request.param

def test_db_connection(db_config):
    conn = connect(db_config["engine"])
    assert conn.is_open


# --- Nested parametrize (cross-product of two sets) ---
@pytest.mark.parametrize("method", ["GET", "POST", "PUT", "DELETE"])
@pytest.mark.parametrize("status_code", [200, 201, 400, 404, 500])
def test_http_response(method, status_code):
    """Runs 4 × 5 = 20 test cases."""
    response = make_request(method)
    assert response.status == status_code


# --- Using pytest.lazy_fixture for lazy evaluation (pytest-lazy-fixture plugin) ===
# Without plugin, use a fixture that returns a factory:
@pytest.fixture
def user_builder():
    def build(name="Alice", age=30):
        return {"name": name, "age": age}
    return build
```

#### TypeScript jest.each() → Python @pytest.mark.parametrize Mapping

| Feature | TypeScript/jest | Python/pytest |
|---------|----------------|--------------|
| Single param | `it.each([1,2,3])("test %d", n => ...)` | `@pytest.mark.parametrize("n", [1,2,3])` |
| Multiple params | `it.each([[1,2,3],[-1,-2,-3]])("a+b=%d", (a,b,e) => ...)` | `@pytest.mark.parametrize("a,b,expected", [(1,2,3),(-1,-2,-3)])` |
| Custom test names | `.each(testCases)("name %s", data => ...)` with custom template | `ids=["small", "large"]` parameter in decorator |
| Class-level params | `describe.each([...])("class", () => { it(...) })` | `@pytest.mark.parametrize` on class methods |
| Cross-product params | Two `.each()` calls or manual combinations | Two `@pytest.mark.parametrize` decorators stacked |

### 2.5 Markers & Skips — xfail, skip, custom markers

#### Every pytest Marker Reference

```python
import pytest
import sys

# --- Built-in markers ---

# Skip unconditionally
@pytest.mark.skip(reason="Not implemented yet")
def test_not_yet():
    pass

# Skip conditionally (like Jest's .skipIf)
@pytest.mark.skipif(sys.platform == "win32", reason="Linux-only feature")
def test_linux_only():
    assert True

# Expected to fail (xfail) — counts as XFAIL not FAIL in results
@pytest.mark.xfail(reason="Known bug: issue #123")
def test_known_bug():
    assert False  # Shows as xfailed, NOT failed!

# xfail with strict mode (if test passes, it becomes XPASS and ERRORs!)
@pytest.mark.xfail(strict=True, reason="Must remain broken until fixed")
def test_must_fail():
    pass

# Skip on specific Python version
@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_match_statement():
    match "hello":
        case "hello": pass

# --- Custom markers ---
@pytest.mark.slow                         # Run with: pytest -m slow
def test_slow_operation():
    import time; time.sleep(10)

@pytest.mark.integration                  # Run with: pytest -m integration
def test_db_integration():
    pass

@pytest.mark.api                          # Run with: pytest -m "api and not slow"
def test_api_endpoint():
    pass

@pytest.mark.unit                         # Run specific subsets
def test_unit_logic():
    pass

# --- Combining markers (AND / OR logic) ---
@pytest.mark.slow
@pytest.mark.integration
def test_slow_integration():                # Marked with BOTH 'slow' AND 'integration'
    pass

# CLI usage:
# pytest -m "slow and not integration"   → slow non-integration tests
# pytest -m "api or unit"                 → api OR unit tests
# pytest -m "not slow"                    → all except slow


# --- Registering custom markers (prevents warnings!) ---
# In pyproject.toml:
# [tool.pytest.ini_options]
# markers = [
#     "slow: marks tests as slow (deselect with '-m \"not slow\"')",
#     "integration: marks tests as integration tests",
#     "api: marks API-related tests",
# ]


# --- Using markers in hooks for custom behavior ---
def pytest_collection_modifyitems(session, config, items):
    """Hook: runs after test collection, before execution."""
    for item in items:
        if "slow" in item.keywords:
            item.add_marker(pytest.mark.slow)  # Programmatically add markers


# --- parametrize marker (special case) ---
@pytest.mark.parametrize(
    "input,output",
    [([1,2,3], 6), ([], 0), ([5, -1], 4)],
    ids=["normal", "empty", "with_negative"]
)
def test_sum(input, output):
    assert sum(input) == output
```

#### Mermaid: Marker Selection Flow

```mermaid
flowchart TD
    A["pytest CLI flags\n-m 'slow' OR -m 'api and not slow' OR --strict-markers"] --> B{"Marker expression parsed?"}
    B -->|Yes| C["Collect ALL tests in the project"]
    C --> D["Filter: keep tests whose markers match expression?"]
    D -->|Yes| E["Test is INCLUDED in run"]
    D -->|No| F["Test is EXCLUDED (not even shown)"]
    E --> G{"Any @pytest.mark.skip if condition?"}
    G -->|True| H["Marked as SKIPPED ⊘"]
    G -->|False| I{"Any @pytest.mark.xfail?"}
    I -->|True| J["Expected to fail — XFAIL if fails, XPASS error if passes"]
    I -->|False| K["Run normally — PASS or FAIL ✓/✗"]

    style A fill:#3069bd,color:white
    style E fill:#4caf50,color:white
    style H fill:#ff9800,color:black
    style K fill:#e0e0e0,color:black
```

### 2.6 Hooks — pytest_configure, pytest_runtest_*, Custom Behavior

#### All Major pytest Hooks Reference

```python
# conftest.py — Hooks are defined here and automatically picked up by pytest!

# --- Hook: Configuration phase (before test collection) ---
def pytest_configure(config):
    """Register custom markers and set up configuration."""
    config.addinivalue_line(
        "markers", "slow: mark test as slow (takes >1s)"
    )
    config.addinivalue_line(
        "markers", "integration: requires external services"
    )

# --- Hook: Test collection (after collecting, before running) ---
def pytest_collection_modifyitems(session, config, items):
    """Modify the collected test items."""
    # Add skip markers dynamically based on conditions
    for item in items:
        if "integration" in item.name:
            item.add_marker(pytest.mark.integration)

# --- Hook: Setup/teardown around each test phase ---
def pytest_runtest_setup(item):
    """Runs BEFORE each test (between 'collection' and 'call' phases)."""
    print(f"\n>>> Setting up: {item.name}")

def pytest_runtest_teardown(item, nextitem):
    """Runs AFTER each test (between 'call' and 'logreport' phases)."""
    print(f">>> Tearing down: {item.name}")

# --- Hook: Each test phase boundaries ---
def pytest_runtest_logstart(nodeid, location):
    """Called at the start of a test phase (setup/call/teardown)."""
    pass

def pytest_runtest_logreport(report):
    """Called after each test phase with the result."""
    if report.failed:
        print(f"FAILED: {report.nodeid} — {report.longrepr}")

# --- Hook: When a test fails ---
def pytest_exception_interact(node, exc_info):
    """Intercept test exceptions (like Jest's test hooks)."""
    tb = exc_info.get_traceback()
    print(f"Test {node.name} raised {exc_info.type.__name__}: {exc_info.value}")

# --- Hook: Session lifecycle ---
def pytest_sessionstart(session):
    """Called at the very beginning of the test session."""
    print("Test session starting!")

def pytest_sessionfinish(session, exitstatus):
    """Called at the very end of the test session."""
    print(f"Test session finished with status: {exitstatus}")

# --- Hook: When generating test IDs ---
def pytest_generate_tests(metafunc):
    """Dynamic fixture generation (like parametrize but programmatic)."""
    if "browser" in metafunc.fixturenames:
        metafunc.parametrize("browser", ["chrome", "firefox", "safari"])

# --- Hook: Command-line options ---
def pytest_addoption(parser):
    """Add custom CLI options."""
    parser.addoption(
        "--env",
        default="dev",
        help="Target environment: dev, staging, prod"
    )

# Usage: conftest.py reads it via request.config.getoption("--env")
```

#### Hook Execution Order

```mermaid
sequenceDiagram
    participant P as pytest CLI
    participant H as Hooks (conftest.py)
    participant T as Test Functions
    
    P->>H: pytest_sessionstart()
    P->>H: pytest_configure()
    P->>H: pytest_collection() → items collected
    P->>H: pytest_collection_modifyitems()
    
    loop For each test file
        H->>T: setup fixture chain
        H->>H: pytest_runtest_setup()
        H->>T: run test body
        H->>H: pytest_runtest_logstart(call)
        T-->>H: PASS / FAIL / SKIP
        H->>H: pytest_runtest_logreport()
        H->>H: pytest_runtest_teardown()
        H->>H: teardown fixture chain
    end
    
    P->>H: pytest_sessionfinish()
```

### 2.7 Plugin Architecture & Popular Plugins

#### Essential pytest Plugins Reference

| Plugin | pip Install | Purpose | TypeScript Equivalent | Key API |
|--------|-----------|---------|---------------------|---------|
| **pytest-asyncio** | `pip install pytest-asyncio` | Run `async def` tests with `@pytest.mark.asyncio` | jest's native async support | `@pytest.mark.asyncio`, `async def test_*()` |
| **pytest-cov** | `pip install pytest-cov` | Code coverage reports | `jest --coverage` | `pytest --cov=src --cov-report=html` |
| **pytest-xdist** | `pip install pytest-xdist` | Parallel test execution across CPUs | `jest --maxWorkers=N` | `pytest -n auto` |
| **pytest-benchmark** | `pip install pytest-benchmark` | Performance benchmarking | Benchmark.js / vitest/bench | `@pytest.mark.benchmark` |
| **hypothesis** | `pip install hypothesis` | Property-based testing | fast-check (npm) | `@given`, `@reify`, `@composite` |
| **pytest-mock** | `pip install pytest-mock` | Cleaner mock integration (`mocker.patch()`) | jest's built-in mocking | `mocker.patch()`, `mocker.MagicMock` |
| **pytest-lazy-fixture** | `pip install pytest-lazy-fixture` | Lazy fixture evaluation (avoid setup overhead) | No direct equivalent | `@lazy_fixture` |
| **pytest-html** | `pip install pytest-html` | HTML test reports | jest's `--json` + custom viewer | `pytest --html=report.html` |
| **pytest-instafail** | `pip install pytest-instafail` | Show failures immediately during run | No direct equivalent | `pytest --instafail` |
| **pytest-sugar** | `pip install pytest-sugar` | Beautiful progress bar output | tapir / custom formatters | `pytest --sugar` |
| **pytest-dotenv** | `pip install pytest-dotenv` | Load .env files for test config | dotenv packages | `.env` auto-loading in tests |
| **pytest-subtests** | `pip install pytest-subtests` | Sub-tests within a single test (like jest.each inside it) | No direct equivalent | `with subtest(...) as sub:` |

#### pytest-asyncio Complete Reference

```python
import pytest
import asyncio

# Configure in pyproject.toml:
# [tool.pytest.ini_options]
# asyncio_mode = "auto"  # All async tests auto-detected (no decorator needed!)
# OR
# asyncio_mode = "strict"  # Requires @pytest.mark.asyncio on every async test


# --- Mode 1: Explicit marker ---
@pytest.mark.asyncio
async def test_async_function():
    result = await some_async_function()
    assert result == "expected"


# --- Mode 2: Async fixture ---
@pytest.fixture
async def db_connection():
    conn = await create_db_connection()
    yield conn
    await conn.close()

@pytest.mark.asyncio
async def test_with_async_fixture(db_connection):
    rows = await db_connection.query("SELECT 1")


# --- Mode 3: Auto mode (set asyncio_mode = "auto" in config) ---
# All async def test_*() are automatically marked as async!

# --- Async context fixture ---
@pytest.fixture
async def temp_file(tmp_path):
    f = tmp_path / "temp.txt"
    await f.write_text("hello")
    yield f
    await f.unlink()

# --- Event loop scope (shared across tests in a session/module) ---
@pytest.fixture(scope="module")
def event_loop():
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()
```

#### pytest-cov Complete Reference

```python
# === CLI Usage ===
# Basic coverage:
pytest --cov=my_package                    # Terminal output
pytest --cov=my_package --cov-report=html  # HTML report in htmlcov/
pytest --cov=my_package --cov-report=xml   # XML for CI integration (Coveralls, Codecov)
pytest --cov=my_package --cov-branch       # Branch coverage
pytest --cov=my_package --cov-fail-under=80  # Fail if coverage < 80%

# In pyproject.toml:
# [tool.pytest.ini_options]
# addopts = "--cov=src --cov-report=term-missing --cov-fail-under=90"
```

#### pytest-xdist Complete Reference

```bash
# Run tests in parallel using all CPU cores:
pytest -n auto                  # Use all available CPUs
pytest -n 4                     # Use exactly 4 workers
pytest -n logical               # Use logical CPUs (hyperthreading-aware)
pytest --dist=load              # Load-balance tasks across workers
pytest --dist=worksteal         # Steal work from busy workers

# Per-worker output:
pytest -n auto --dist=loadgroup  # Keep test classes together per worker
```

#### pytest-benchmark Complete Reference

```python
import pytest

@pytest.mark.benchmark(min_time=0.1, max_time=1.0)
def test_sort_performance(benchmark):
    """benchmark() is the fixture provided by pytest-benchmark."""
    data = list(range(1000))
    result = benchmark(sorted, data)  # Runs multiple iterations, reports stats
    assert len(result) == 1000

# Output:
# ---------- benchmarks --------------
# test_sort_performance       50.2ms ± 1.2ms  (mean ± std. dev., 10 runs)
```

---

## 3. unittest — The Built-in Alternative (Complete Reference)

### When to Use unittest vs pytest

| Criterion | unittest (built-in) | pytest (recommended) |
|-----------|-------------------|---------------------|
| **No pip install** | ✅ Standard library | ❌ Requires `pip install pytest` |
| **Readability** | `self.assertEqual(a, b)` | `assert a == b` (cleaner!) |
| **Fixtures/Setup** | `setUp()`, `setUpClass()` | `@pytest.fixture` (more flexible!) |
| **Parametrization** | Manual loops | `@pytest.mark.parametrize` (declarative!) |
| **Async support** | `IsolatedAsyncioTestCase` (3.8+) | `pytest-asyncio` plugin |
| **Mocking** | `unittest.mock.patch()` | Same, plus `mocker.patch()` via pytest-mock |
| **Plugins** | ❌ No plugin ecosystem | ✅ Huge ecosystem |
| **Test discovery** | Manual class structure | Auto-discovers `test_*.py` files |

### Complete unittest.TestCase API Reference

```python
import unittest
import sys

# === Every TestCase method available ===
class TestMathComplete(unittest.TestCase):
    """Full unittest test class — every method explained."""
    
    # --- Class-level hooks (run ONCE before/after all tests in class) ---
    @classmethod
    def setUpClass(cls):
        """Like Jest's beforeAll() — runs once for entire class."""
        cls.expensive_resource = create_expensive_resource()
        print(f"setUpClass: {cls.__name__}")  # Access via cls
    
    @classmethod
    def tearDownClass(cls):
        """Like Jest's afterAll() — runs once after all tests."""
        cls.expensive_resource.cleanup()
    
    # --- Per-test hooks (run before/after EACH test method) ---
    def setUp(self):
        """Like Jest's beforeEach() — runs before every test method."""
        self.shared_value = 0
        print(f"setUp: {self._testMethodName}")  # Access via self
    
    def tearDown(self):
        """Like Jest's afterEach() — runs after every test method."""
        del self.shared_value  # Clean up per-test resources
    
    # === Every assertion type in unittest.TestCase ===
    
    # Equality assertions
    def test_assertEqual(self):
        self.assertEqual(1 + 1, 2)           # Like expect(2).toBe(2)
        self.assertNotEqual(1 + 1, 3)         # Like expect(2).not.toBe(3)
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=7)  # Float comparison
    
    # Identity assertions
    def test_isNone(self):
        self.assertIsNone(None)               # Like expect(x).toBeNull()
        self.assertIsNotNone("hello")          # Like expect(x).toBeDefined()
        self.assertIs(True, True)             # Like expect(x).toBe(true) (identity)
        self.assertIsNot("a", "b")            # like expect(a).not.toBe(b)
    
    # Boolean assertions
    def test_truthy(self):
        self.assertTrue(bool([1, 2, 3]))      # Like expect(x).toBeTruthy()
        self.assertFalse([])                   # Like expect(x).toBeFalsy()
    
    # Container assertions
    def test_container(self):
        self.assertIn("x", ["a", "b", "x"])  # Like expect(arr).toContain(x)
        self.assertNotIn("z", ["a", "b", "x"])
        self.assertIn("key", {"key": "val"})
        self.assertIsInstance([], list)       # Like expect(x).toBeInstanceOf(Array)
        self.assertNotIsInstance(5, str)
    
    # Exception assertions
    def test_exception(self):
        with self.assertRaises(ValueError):
            raise ValueError("error")
        
        with self.assertRaisesRegex(TypeError, "expected type"):
            raise TypeError("expected type")
    
    # Context manager assertions (Python 3.1+)
    def test_warns(self):
        import warnings
        with self.assertWarns(DeprecationWarning):
            warnings.warn("deprecated", DeprecationWarning)
        
        with self.assertNoWarnings():
            pass
    
    # Skip/xfail in unittest (different from pytest!)
    @unittest.skip("Not implemented")
    def test_skipped(self):
        pass
    
    @unittest.skipIf(sys.platform == "win32", "Linux only")
    def test_conditional_skip(self):
        pass
    
    @unittest.expectedFailure
    def test_expected_failure(self):
        self.assertTrue(False)  # Will show as XFAIL not FAIL


# === Async unittest (Python 3.8+) ===
class TestAsyncUnittest(unittest.IsolatedAsyncioTestCase):
    """unittest's built-in async test case — no pytest-asyncio needed!"""
    
    async def test_async_assertion(self):
        result = await some_async_function()
        self.assertEqual(result, "expected")
    
    async def test_async_exception(self):
        with self.assertRaises(ConnectionError):
            await failing_async_function()

# === Running tests: ===
if __name__ == "__main__":
    unittest.main()
    # Or: python -m unittest discover -s tests
```

### Mermaid: Test Organization in Python vs TypeScript

```mermaid
flowchart TD
    subgraph TS_PROJECT["TypeScript Project Structure"]
        direction TB
        T1["src/\n├── math.ts\n└── api.ts"] --> T2["__tests__/\n├── math.test.ts\n└── api.test.ts"]
        T3["jest.config.js / jest in package.json\n→ testMatch, roots, coverage thresholds"]
        style T1 fill:#3178c6,color:white
        style T2 fill:#3178c6,color:white
        style T3 fill:#3178c6,color:white
    end

    subgraph PY_PROJECT["Python Project Structure"]
        direction TB
        P1["src/\n├── my_package/\n│   ├── __init__.py\n│   ├── math.py\n│   └── api.py"] --> P2["tests/\n├── conftest.py  ← shared fixtures & hooks!\n├── test_math.py\n└── test_api.py"]
        P4["[tool.pytest]\nin pyproject.toml\n→ addopts, testpaths, markers"]
        style P1 fill:#3069bd,color:white
        style P2 fill:#3069bd,color:white
        style P4 fill:#3069bd,color:white
    end

    T3 -.->|"jest.config.js configures everything"| P4
```

### Key Notes: pytest vs unittest

1. **pytest is the dominant framework** for Python testing — like jest in TypeScript. More flexible, readable, and powerful than unittest.
2. **unittest is built into the standard library** — no pip install needed, but less readable (`self.assertEqual` vs `assert`).
3. **pytest fixtures use explicit dependency injection** (parameters) rather than implicit hooks (`setUp()`).
4. **Use pytest for all new projects**; use unittest only when third-party packages are unavailable.

---

## 4. Mocking Exhaustive Reference (unittest.mock)

### 4.1 patch() at Every Level

#### Complete @patch Reference — Where to Patch, How, and Why

```python
# === CRITICAL RULE: Patch where the name is LOOKED UP, not where it's DEFINED! ===
# If file `service.py` does: from database import connect
# → You must patch "service.connect" (where it's imported), NOT "database.connect"!

from unittest.mock import patch, MagicMock, AsyncMock, call, ANY
import asyncio

# --- Pattern 1: Patch as decorator (most common) ---
@patch("my_module.database.connect")
def test_db_connect(mock_connect):
    mock_connect.return_value = {"status": "connected"}
    result = service.get_connection()
    assert result == {"status": "connected"}
    mock_connect.assert_called_once()


# --- Pattern 2: Patch as context manager ---
def test_with_context_manager():
    with patch("my_module.requests.get") as mock_get:
        mock_get.return_value.status_code = 200
        mock_get.return_value.json.return_value = {"name": "Alice"}
        
        result = service.fetch_user(1)
        assert result["name"] == "Alice"


# --- Pattern 3: Patch object attributes (not just functions) ---
@patch.object(service.UserService, "validate", return_value=True)
def test_validation(mock_validate):
    user = service.UserService()
    user.create({"name": "Alice"})
    mock_validate.assert_called_once_with({"name": "Alice"})


# --- Pattern 4: Patch class instantiation (new instances) ---
@patch("my_module.http.Client")
def test_client_instantiation(mock_client_class):
    mock_instance = MagicMock()
    mock_instance.get.return_value.json.return_value = {"data": "ok"}
    mock_client_class.return_value = mock_instance  # Client() → mock_instance
    
    result = service.make_request()
    assert result["data"] == "ok"


# --- Pattern 5: Patch built-in functions (like open()) ---
@patch("builtins.open", new_callable=MagicMock)
def test_file_operations(mock_open):
    mock_file = MagicMock()
    mock_file.__enter__.return_value.read.return_value = '{"key": "value"}'
    mock_open.return_value.__enter__.return_value = mock_file
    
    result = load_json("data.json")
    assert result["key"] == "value"


# --- Pattern 6: Multiple patches (stacked decorators — order is REVERSE!) ---
@patch("my_module.db.save")
@patch("my_module.api.fetch_user")      # This runs FIRST (innermost)
def test_full_flow(mock_fetch, mock_save):   # Parameters are in REVERSE order!
    mock_fetch.return_value = {"name": "Alice"}
    
    result = service.create_user(1)
    
    assert mock_fetch.called
    mock_save.assert_called_once_with(result)


# --- Pattern 7: Patch with return_value chains ---
@patch("my_module.api.Client")
def test_chained_calls(mock_client):
    mock_resp = MagicMock()
    mock_resp.json.return_value = {"users": [{"name": "Alice"}]}
    mock_resp.status_code = 200
    
    mock_instance = MagicMock()
    mock_instance.get.return_value = mock_resp
    mock_client.return_value = mock_instance
    
    result = service.get_users()
    assert len(result["users"]) == 1


# --- Pattern 8: Patch with side_effect for dynamic behavior ---
@patch("my_module.api.get_user")
def test_dynamic_response(mock_get):
    def side_effect(user_id):
        if user_id == 1:
            return {"name": "Alice"}
        elif user_id == 2:
            raise ValueError("Not found")
        else:
            return None
    
    mock_get.side_effect = side_effect


# --- Pattern 9: AsyncMock for async functions (Python 3.8+) ---
@patch("my_module.api.fetch_users", new_callable=AsyncMock)
async def test_async_mock(mock_fetch):
    mock_fetch.return_value = [{"name": "Alice"}]
    result = await service.get_users()
    assert result == [{"name": "Alice"}]


# --- Pattern 10: New patch with lambda/inline return_value ===
with patch("my_module.config", new={"DEBUG": True, "ENV": "test"}) as mock_config:
    assert mock_config.DEBUG is True
```

#### Mermaid: Mocking Chain — How patch() Works Internally

```mermaid
flowchart TD
    A["@patch('module.func') decorator applied"] --> B["pytest discovers test function\ntest_func(mock_name)"]
    B --> C["@patch intercepts the call to service.func"]
    C --> D["Replaces service.func with a MagicMock instance"]
    D --> E["Test runs — calls service.func() which returns mock's return_value"]
    E --> F["After test: @patch restores original function\nto module.func"]
    F --> G["Assert methods available:\n- assert_called_once_with(args)\n- assert_any_call(args)\n- call_count\n- mock_calls (list of all calls)"]

    style A fill:#3069bd,color:white
    style C fill:#ff9800,color:black
    style D fill:#4caf50,color:white
    style G fill:#e0e0e0,color:black
```

### 4.2 MagicMock, AsyncMock, PropertyMock

#### Every Mock Type Reference

| Mock Type | Purpose | When to Use |
|-----------|---------|-------------|
| `MagicMock()` | Full-featured mock with auto-created attributes | General mocking of functions/methods |
| `AsyncMock()` | Async version of MagicMock (returns awaitable) | Mocking `async def` functions |
| `PropertyMock()` | Mock Python properties (`@property`) | Mocking `@property` decorated methods |
| `Mock()` | Minimal mock (no auto-created sub-mocks) | Performance-critical tests |
| `NonCallableMagicMock()` | Mock objects that aren't called as functions | Mocking classes/data objects |

```python
from unittest.mock import MagicMock, AsyncMock, PropertyMock, Mock

# === MagicMock: The Swiss Army Knife ===
mock = MagicMock()

# Auto-creates any attribute access (chainable!)
mock.database.connect.return_value = "connected"  # mock.database → auto-created MagicMock!
assert mock.database.connect() == "connected"     # Returns the configured value

# Configure different return values for sequential calls
mock.get_users.side_effect = [
    [{"id": 1}],       # First call
    [{"id": 2}],       # Second call
    [],                 # Third call (empty list)
]
assert mock.get_users() == [{"id": 1}]
assert mock.get_users() == [{"id": 2}]
assert mock.get_users() == []


# === AsyncMock: For async functions ===
async def test_async_mock():
    mock_fetch = AsyncMock(return_value={"data": "ok"})
    result = await mock_fetch("https://api.example.com")
    assert result == {"data": "ok"}

    # AsyncMock also has assert_called_once, etc.
    mock_fetch.assert_awaited_once_with("https://api.example.com")


# === PropertyMock: Mock @property decorators ===
class Database:
    @property
    def connection_string(self):
        return "postgresql://localhost/db"

db = Database()

# Patch the property
with patch.object(Database, "connection_string", new_callable=PropertyMock) as mock_conn:
    mock_conn.return_value = "postgresql://test:test@localhost/testdb"
    assert db.connection_string == "postgresql://test:test@localhost/testdb"


# === Mock vs MagicMock difference ===
minimal_mock = Mock()      # No auto-created sub-mocks — AttributeError on any attribute access
full_mock = MagicMock()    # Auto-creates attributes, returns new MagicMock for each access

# Use Mock when you want strict behavior (catches typos in test code)
# Use MagicMock when you need chainable calls like `mock.db.users.get()`
```

### 4.3 side_effect Patterns

#### Every side_effect Pattern with Examples

| Pattern | Code | Effect |
|---------|------|--------|
| Static value | `side_effect = ValueError("error")` | Always raise this exception |
| Iterable | `side_effect = [1, 2, 3]` | Return successive values until exhausted, then raise StopIteration |
| Callable | `side_effect = lambda x: x * 2` | Call the function with args each time |
| Exception list | `side_effect = [ValueError(), TypeError()]` | Raise exceptions in order per call |
| Reset | `side_effect = None` | Clear side_effect (returns return_value or MagicMock) |

```python
from unittest.mock import patch, MagicMock

# --- Pattern 1: side_effect with exception ---
@patch("my_module.api.fetch")
def test_side_effect_exception(mock_fetch):
    mock_fetch.side_effect = ConnectionError("Connection refused")
    
    with pytest.raises(ConnectionError, match="Connection refused"):
        service.fetch_data()


# --- Pattern 2: side_effect with successive values ---
@patch("my_module.service.next_id")
def test_side_effect_values(mock_next_id):
    mock_next_id.side_effect = [100, 101, 102]  # Each call returns the next value
    
    assert service.next_id() == 100
    assert service.next_id() == 101
    assert service.next_id() == 102

    # After exhausted: raises StopIteration!


# --- Pattern 3: side_effect with callable (dynamic behavior) ---
@patch("my_module.service.calculate_price")
def test_side_effect_callable(mock_calc):
    def calc(item):
        prices = {"A": 10, "B": 20, "C": 30}
        return prices.get(item["name"], 0)
    
    mock_calc.side_effect = calc
    
    assert service.calculate_price({"name": "A"}) == 10
    assert service.calculate_price({"name": "B"}) == 20


# --- Pattern 4: side_effect with list of exceptions (simulate progressive failures) ---
@patch("my_module.api.retry_request")
def test_side_effect_exception_list(mock_retry):
    mock_retry.side_effect = [
        TimeoutError(),      # First call → timeout
        TimeoutError(),      # Second call → timeout  
        {"data": "success"}, # Third call → success!
    ]
    
    with pytest.raises(TimeoutError):
        service.retry_request()  # Raises TimeoutError
    
    with pytest.raises(TimeoutError):
        service.retry_request()  # Raises TimeoutError again
    
    result = service.retry_request()  # Finally returns {"data": "success"}
    assert result["data"] == "success"


# --- Pattern 5: side_effect that checks arguments then returns ---
@patch("my_module.api.get_user")
def test_side_effect_with_validation(mock_get):
    def get_user(user_id):
        if user_id < 0:
            raise ValueError("Invalid ID")
        return {"id": user_id, "name": f"User {user_id}"}
    
    mock_get.side_effect = get_user
    
    with pytest.raises(ValueError):
        service.get_user(-1)
    
    assert service.get_user(5)["name"] == "User 5"
```

### 4.4 assert_called_with / call chains

#### Every Assertion Method on Mock Objects

| Mock Assertion | Description | TypeScript Equivalent |
|---------------|-------------|---------------------|
| `mock.assert_called()` | Called at least once | `expect(mock).toHaveBeenCalled()` |
| `mock.assert_called_once()` | Called exactly once | `expect(mock).toHaveBeenCalledTimes(1)` |
| `mock.assert_called_with(*args, **kwargs)` | Last call had these args | `expect(mock).toHaveBeenCalledWith(args)` |
| `mock.assert_called_once_with(*args, **kwargs)` | Called once with these args | `expect(mock).toHaveBeenCalledTimes(1) AND toHaveBeenCalledWith` |
| `mock.assert_any_call(*args, **kwargs)` | Any call (not just last) had these args | Check mock.mock.calls for any matching |
| `mock.assert_not_called()` | Never called | `expect(mock).not.toHaveBeenCalled()` |
| `mock.call_args` | Last call's args/kwargs as tuple | `mock.mock.calls[mock.mock.calls.length - 1]` |
| `mock.call_count` | Number of calls | `mock.mock.calls.length` |
| `mock.call_args_list` | List of all calls as Call objects | `[mock.mock.calls[0], mock.mock.calls[1], ...]` |
| `mock.method_calls` | Calls to sub-mock methods | N/A (Jest doesn't track sub-calls) |

```python
from unittest.mock import patch, call

@patch("my_module.service.fetch")
def test_mock_assertions(mock_fetch):
    # Call the function under test multiple times
    service.fetch(1)
    service.fetch(2)
    service.fetch(3)
    
    # Every assertion method:
    mock_fetch.assert_called()                   # Called at least once ✓
    assert mock_fetch.call_count == 3            # Called exactly 3 times
    
    # Last call assertions:
    mock_fetch.assert_called_with(3)             # LAST call was with (3)
    
    # Any call assertion:
    mock_fetch.assert_any_call(1)                # SOME call had args (1)
    mock_fetch.assert_any_call(2)                # SOME call had args (2)
    
    # All calls tracked:
    expected_calls = [call(1), call(2), call(3)]
    assert mock_fetch.call_args_list == expected_calls
    
    # Never called assertion:
    mock_fetch.reset_mock()                      # Reset all tracking
    assert not mock_fetch.called                 # Now not called


# --- Comparing with exact call sequences (order matters!) ===
@patch("my_module.api.get")
def test_call_order(mock_get):
    service.get_user(1)
    service.get_post(2)
    
    # Check the EXACT sequence of calls:
    mock_get.assert_has_calls([
        call("/api/users/1"),
        call("/api/posts/2"),
    ], any_order=False)  # If any_order=True, order doesn't matter


# --- Asserting called with ANY matching kwargs (partial match) ===
@patch("my_module.api.post")
def test_partial_kwargs(mock_post):
    service.create_user(name="Alice", email="a@b.com", age=30)
    
    # Assert that name="Alice" was in the call (ignore other kwargs!)
    from unittest.mock import ANY
    mock_post.assert_called_with(
        "/api/users",
        json={"name": "Alice", "age": ANY, "email": ANY}  # ANY matches any value
    )


# --- AsyncMock assertions ===
@patch("my_module.api.fetch", new_callable=AsyncMock)
async def test_async_mock_assertions(mock_fetch):
    await service.fetch_user(1)
    
    mock_fetch.assert_awaited()                    # Awaits happened
    mock_fetch.assert_awaited_once()               # Awaited exactly once
    mock_fetch.assert_awaited_with(1)              # Last await had args (1,)
```

### 4.5 spec / spec_set

#### Using spec to Catch Typos in Test Code

```python
from unittest.mock import MagicMock, patch

# --- spec: Enforce real object's interface on mock ===
class RealUser:
    def get_name(self): return "Alice"
    def get_age(self): return 30

# With spec: only allow attributes that exist on the real class!
mock = MagicMock(spec=RealUser)
mock.get_name()     # ✅ OK — exists on RealUser
mock.get_age()      # ✅ OK
mock.get_email()    # ❌ AttributeError! Not on RealUser — catches typos!


# --- spec_set: Like spec but raises on ANY attribute access (even write!) ===
mock_strict = MagicMock(spec_set=RealUser)
mock_strict.get_name()  # ✅
mock_strict.fake_attr = "x"  # ❌ AttributeError! Can't even set fake attributes!

# Usage in @patch:
@patch("my_module.UserService", spec=True)
def test_with_spec(mock_service):
    instance = mock_service()
    instance.get_name()     # Only allowed methods that exist on real UserService
```

### 4.6 Mock Context Managers & Chaining

#### Mocking `with` Statements and Chainable APIs

```python
from unittest.mock import patch, MagicMock

# --- Mocking a context manager (async with / with) ===
class FileHandler:
    async def __aenter__(self):
        return self
    async def __aexit__(self, *args):
        pass
    def read(self):
        return "data"

@patch("my_module.FileHandler", new_callable=AsyncMock)
async def test_context_manager(mock_fh_class):
    mock_instance = AsyncMock()
    mock_instance.read.return_value = "mocked data"
    mock_fh_class.return_value.__aenter__ = AsyncMock(return_value=mock_instance)
    
    async with FileHandler() as handler:
        data = await handler.read()
        assert data == "mocked data"


# --- Mocking a chainable API (fluent interface) ===
@patch("my_module.http.Client")
def test_chainable_api(mock_client_class):
    mock_resp = MagicMock()
    mock_resp.json.return_value = {"users": [{"name": "Alice"}]}
    
    mock_instance = MagicMock()
    mock_instance.get.return_value = mock_resp
    
    mock_client_class.return_value = mock_instance
    
    # The real code does: client.get("/users").json()
    result = service.get_users()  # Inside: client.get("/users").json()
    assert result == {"users": [{"name": "Alice"}]}


# --- Mocking with statement (non-async context manager) ===
@patch("my_module.database.connect")
def test_sync_context_manager(mock_connect):
    mock_conn = MagicMock()
    mock_conn.__enter__ = MagicMock(return_value=mock_conn)  # self on __enter__
    mock_conn.__exit__ = MagicMock(return_value=False)
    mock_connect.return_value = mock_conn
    
    with service.connect_db() as conn:
        conn.query("SELECT 1")
    
    mock_connect.assert_called_once()
```

### 4.7 Complete TypeScript jest.mock() → Python unittest.mock() Mapping Table

| Feature | TypeScript/jest | Python/pytest + unittest.mock | Notes |
|---------|----------------|------------------------------|-------|
| **Create mock function** | `const fn = jest.fn()` | `from unittest.mock import MagicMock; fn = MagicMock()` | Same concept |
| **Set return value** | `fn.mockReturnValue(42)` | `fn.return_value = 42` | Simple attribute assignment! |
| **Set async return value** | `fn.mockResolvedValue(data)` | `fn = AsyncMock(return_value=data)` | Use AsyncMock for async functions |
| **Set error to throw** | `fn.mockImplementation(() => { throw new Error("err") })` | `fn.side_effect = ValueError("err")` | side_effect with exception raises it |
| **Mock module** | `jest.mock("./module")` | `@patch("module.func")` decorator or context manager | Python patches at import location |
| **Verify called once** | `expect(fn).toHaveBeenCalledTimes(1)` | `fn.assert_called_once()` | Method call vs chained matcher |
| **Verify called with args** | `expect(fn).toHaveBeenCalledWith(x, y)` | `fn.assert_called_with(x, y)` | Same concept |
| **Verify any call** | Check `fn.mock.calls` array | `fn.assert_any_call(x, y)` | Direct method available in Python! |
| **Spy on existing function** | `jest.spyOn(obj, "method").mockReturnValue(42)` | `with patch.object(obj, "method", return_value=42): ...` | Patch object's method in context |
| **Restore all mocks** | `jest.restoreAllMocks()` | Mocks auto-restored by @patch decorator/context manager! | Python's @patch cleans up automatically |
| **Mock constructor** | `const C = jest.fn().mockReturnValue({})` | `@patch("module.Class")` → `mock.return_value = instance` | Same pattern, different syntax |
| **Mock property** | `Object.defineProperty(obj, "prop", { get: () => 42 })` | `@patch.object(Class, "prop", new_callable=PropertyMock)` | PropertyMock for @property decorators |
| **Clear mock state** | `fn.mockClear()` | `fn.reset_mock()` | Same concept |
| **Get call arguments** | `fn.mock.calls[0][0]` (first arg of first call) | `fn.call_args_list[0][0]` or `fn.call_args[0][0]` | Index into call list |
| **Mock with side effect function** | `fn.mockImplementation((x) => x * 2)` | `fn.side_effect = lambda x: x * 2` | Lambda works as side_effect! |

---

## 5. Coverage Patterns & Configuration

### Complete pytest-cov Reference

```bash
# === Installation ===
pip install pytest-cov

# === Basic Usage ===
pytest --cov=my_package                    # Terminal output
pytest --cov=my_package --cov-report=html  # HTML report → htmlcov/index.html
pytest --cov=my_package --cov-report=xml   # XML → coverage.xml (for Coveralls/Codecov)
pytest --cov=my_package --cov-report=json  # JSON → coverage.json (for CI/CD)
pytest --cov=my_package --cov-branch       # Branch-level coverage
pytest --cov=my_package --cov-fail-under=80  # FAIL if coverage < 80%

# === Targeted Coverage ===
pytest --cov=my_package --cov=tests        # Coverage on multiple targets
pytest --cov=my_package --cov-ignore-errors  # Don't fail on syntax errors in code
pytest --cov=my_package --cov-skip-covered   # Skip already-covered lines
```

#### pyproject.toml Coverage Configuration

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = [
    "--cov=src",
    "--cov-report=term-missing",     # Show missing lines in terminal
    "--cov-report=html:htmlcov",      # Generate HTML report
    "--cov-report=xml:coverage.xml",  # XML for CI integration
    "--cov-branch",                    # Branch coverage
    "--cov-fail-under=90",            # Fail if below 90%
]

[tool.coverage.run]
source = ["src"]                      # What to measure
omit = [
    "*/tests/*",                       # Exclude tests from coverage
    "*/__init__.py",                   # Exclude init files
    "*/migrations/*",                  # Exclude migrations
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",                # Lines marked with this won't be required
    "def __repr__",                    # Don't require __repr__ coverage
    "if TYPE_CHECKING:",               # Type-only imports
    "@abstractmethod",                  # Abstract methods
]

[tool.coverage.html]
directory = "htmlcov"                 # HTML report output directory
```

#### Mermaid: Coverage Report Pipeline

```mermaid
flowchart TD
    A["pytest --cov=src runs all tests"] --> B["Coverage.py instruments executed bytecode\ntracks which lines/functions/branches are executed"]
    B --> C{"Source files analyzed?"}
    C -->|Yes| D["Calculate coverage percentages\nLine %\nBranch %\nFunction %"]
    C -->|"No (missing from source)"| E["Mark as NOT measured (gap in report)"]
    D --> F["Generate reports:\n- Terminal: term-missing\n- HTML: htmlcov/\n- XML: coverage.xml\n- JSON: coverage.json"]
    E --> F
    F --> G{"--cov-fail-under threshold met?"}
    G -->|Yes| H["Tests PASS with coverage data"]
    G -->|No| I["Tests FAIL with coverage gap details"]

    style A fill:#3069bd,color:white
    style D fill:#4caf50,color:white
    style F fill:#ff9800,color:black
    style H fill:#4caf50,color:white
    style I fill:#f44336,color:white
```

---

## 6. Test Organization Strategies

### 6.1 conftest.py Hierarchy (Root → Module → Function Level)

#### Mermaid: conftest.py Hierarchy

```mermaid
flowchart TD
    subgraph PROJECT["Project Root"]
        direction TB
        P1["project/\n├── conftest.py  ← Session-scoped fixtures\n   available to ALL tests in the project!"] --> P2["pytest_configure runs here for entire project"]
    end

    subgraph APP_LEVEL["app/ directory"]
        A1["app/conftest.py  ← Module-scoped fixtures\n   available to ALL tests under app/"] --> A2["Module-level hooks for app/ tests"]
    end

    subgraph TESTS_LEVEL["tests/ directory"]
        T1["tests/conftest.py  ← Test-specific fixtures\n   only visible to tests in tests/"] --> T2["Test-level hooks for test discovery"]
    end

    P2 -.->|"inherited by| A2
    A2 -.->|"inherited by| T2
    
    style P1 fill:#3069bd,color:white
    style A1 fill:#e8a53b,color:black
    style T1 fill:#4caf50,color:white
```

#### Complete conftest.py Patterns at Every Level

```python
# ==========================================
# project_root/conftest.py  (project-wide)
# ==========================================
# These fixtures/hooks are available to EVERY test in the entire project.

import pytest

@pytest.fixture(scope="session")
def global_config():
    """Created once for the entire test session — shared across all files."""
    return {"database_url": "postgresql://test:test@localhost/testdb"}

def pytest_configure(config):
    """Register markers and custom options at project level."""
    config.addinivalue_line("markers", "slow: slow tests (>1s)")
    config.addinivalue_line("markers", "integration: requires external services")


# ==========================================
# project_root/tests/conftest.py  (test suite-wide)
# ==========================================

import pytest
from unittest.mock import MagicMock, AsyncMock

@pytest.fixture
def api_client():
    """Shared HTTP client for all API tests."""
    from httpx import AsyncClient
    return AsyncClient(base_url="http://localhost:8000")

@pytest.fixture(autouse=True)
async def clear_database(db_connection):
    """Automatically clear DB before every test in this directory."""
    await db_connection.execute("DELETE FROM *")
    yield


# ==========================================
# project_root/tests/api/conftest.py  (API tests only)
# ==========================================

@pytest.fixture
def auth_token():
    """Generate auth token for API tests."""
    return create_test_token(role="admin")

@pytest.fixture
def authenticated_client(auth_token, api_client):
    """Client with authentication headers automatically set."""
    api_client.headers["Authorization"] = f"Bearer {auth_token}"
    return api_client


# ==========================================
# project_root/tests/integration/conftest.py  (integration tests only)
# ==========================================

@pytest.fixture(scope="module")
def live_database():
    """Expensive DB setup — shared across ALL integration tests in this module."""
    db = start_real_postgres_container()
    yield db
    db.shutdown()
```

### 6.2 Fixtures as Classes

#### When to Use Class-Based Fixtures

```python
# === Pattern: Fixtures as a class (organized, reusable test helpers) ===
import pytest
from dataclasses import dataclass
from unittest.mock import MagicMock

@dataclass
class UserFixtures:
    """Class of related fixtures — use each fixture individually by name."""
    
    @pytest.fixture
    def user_admin(self):
        return {"id": 1, "name": "Admin", "role": "admin"}
    
    @pytest.fixture
    def user_normal(self):
        return {"id": 2, "name": "Normal", "role": "user"}
    
    @pytest.fixture
    def user_guest(self):
        return {"id": 3, "name": "Guest", "role": "guest"}

# In conftest.py (project-wide):
from .fixtures import UserFixtures
user_fixtures = UserFixtures()

# Now in test files:
def test_admin_can_delete(user_admin):
    assert user_admin["role"] == "admin"

def test_guest_cannot_delete(user_guest):
    assert user_guest["role"] == "guest"


# === Pattern: Fixtures as a class with setup/teardown ===
class DatabaseFixtures:
    @pytest.fixture(scope="module")
    def db(self):
        conn = create_test_db()
        yield conn
        conn.close()
    
    @pytest.fixture
    def seed_users(self, db):
        """Seeds the database with test users."""
        db.execute("INSERT INTO users VALUES (1, 'Alice')")
        yield
    
    @pytest.fixture
    def seed_posts(self, db):
        db.execute("INSERT INTO posts VALUES (1, 'Hello')")
        yield

# In conftest.py:
from .fixtures import DatabaseFixtures
db_fixtures = DatabaseFixtures()
```

### 6.3 Test Directory Structure Patterns

#### Recommended Project Layouts

```
# === Layout 1: Flat (small projects) ===
project/
├── src/
│   ├── __init__.py
│   ├── math.py
│   └── api.py
├── tests/
│   ├── conftest.py          # Shared fixtures for all tests
│   ├── test_math.py         # Tests for math.py
│   └── test_api.py          # Tests for api.py

# === Layout 2: Mirrored (medium projects — recommended) ===
project/
├── src/
│   ├── __init__.py
│   ├── auth/
│   │   ├── __init__.py
│   │   ├── login.py
│   │   └── register.py
│   └── api/
│       ├── __init__.py
│       └── endpoints.py
├── tests/
│   ├── conftest.py          # Project-wide fixtures
│   ├── auth/
│   │   ├── conftest.py      # Auth-specific fixtures
│   │   └── test_login.py    # Tests mirror src/auth/login.py
│   └── api/
│       ├── conftest.py      # API-specific fixtures
│       └── test_endpoints.py

# === Layout 3: Functional separation (large projects) ===
project/
├── src/
├── tests/
│   ├── unit/                # Fast, isolated unit tests
│   │   ├── conftest.py      # Unit-level mocks & fixtures
│   │   └── test_math.py
│   ├── integration/          # Tests requiring real services
│   │   ├── conftest.py      # Live DB servers, API endpoints
│   │   └── test_api_flow.py
│   ├── e2e/                  # End-to-end browser tests
│   │   └── test_login_flow.py
│   └── fixtures/             # Reusable fixture classes
│       └── user_fixtures.py
```

---

## 7. Property-Based Testing with hypothesis

### What is Property-Based Testing?

Property-based testing generates random inputs and checks that certain properties (invariants) hold for ALL inputs — rather than checking specific input/output pairs like traditional testing. In TypeScript, use `fast-check` (npm). In Python, use `hypothesis`.

#### TypeScript fast-check → Python hypothesis Mapping

| Concept | TypeScript (fast-check) | Python (hypothesis) |
|---------|----------------------|-------------------|
| Property test | `fc.property(fc.integer(), fc.string(), ...)` | `@given(...)` decorator |
| Generate integers | `fc.integer()` | `st.integers()` |
| Generate strings | `fc.string()` | `st.text()` or `st.characters()` |
| Generate booleans | `fc.boolean()` | `st.booleans()` |
| Generate lists | `fc.array(fc.integer())` | `st.lists(st.integers())` |
| Generate dicts | `fc.record({key: fc.string(), value: fc.integer()})` | `st.dictionaries(st.text(), st.integers())` |
| Shrinking (find minimal failing input) | Automatic in fast-check | Automatic in hypothesis! |
| Example-based testing | `check(...).withExamples(...)` | `.example(x, y, ...)` on strategy |

#### Complete hypothesis Reference

```python
from hypothesis import given, settings, assume, strategies as st, Phase
from hypothesis.strategies import just, one_of, builds
import pytest


# --- Basic @given: Property-based test with auto-generated inputs ---
@given(st.integers(), st.integers())
def test_addition_is_commutative(a, b):
    """Property: a + b == b + a for ALL integers (generated automatically!)."""
    assert add(a, b) == add(b, a)


# --- @given with multiple strategies ===
@given(
    st.lists(st.integers(), min_size=1),   # Non-empty list of integers
    st.integers(min_value=0)                # Non-negative index
)
def test_index_access(lst, idx):
    """Property: indexing a non-empty list at a valid index never crashes."""
    assume(idx < len(lst))  # Skip test if condition doesn't hold (don't count as failure!)
    _ = lst[idx]            # Should not raise IndexError


# --- Common Strategies Reference ===

# Primitive strategies
st.integers(min_value=-1000, max_value=1000)     # Integers in range
st.floats(min_value=0.0, max_value=100.0, allow_nan=False)  # Floats without NaN
st.booleans()                                      # True/False
st.text(min_size=1, max_size=100)                 # Strings
st.binary(min_size=0, max_size=1024)              # Bytes

# Collection strategies
st.lists(st.integers(), min_size=0, max_size=10)  # Lists of integers
st.sets(st.text())                                 # Sets of strings
st.dictionaries(                                   # Dicts: str → int
    keys=st.text(min_size=1), 
    values=st.integers(),
    min_size=1, max_size=5
)
st.tuples(st.integers(), st.text())               # Fixed tuples

# Special strategies
st.just(42)                                        # Always 42 (like a constant generator)
st.one_of(st.integers(), st.floats())              # Choose from multiple strategies
st.none()                                          # Always None
st.sampled_from(["apple", "banana", "cherry"])    # Sample from list (random choice)

# Build custom objects
@dataclass
class User:
    name: str
    age: int

st.builds(User, name=st.text(), age=st.integers(0, 150))


# --- Settings: Control test behavior ===
@given(st.integers())
@settings(
    max_examples=500,              # Run 500 examples (default: 100)
    verbosity=Verbosity.verbose,   # Show every failing example
    timeout=60,                    # Timeout after 60 seconds
    phases=[Phase.generate],       # Only generate (don't shrink or replay)
    deadline=500                   # Fail if single example takes >500ms
)
def test_with_settings(x):
    assert x * 0 == 0  # Always true


# --- assume(): Skip examples that don't meet condition ===
@given(st.integers(), st.integers())
def test_division_property(a, b):
    assume(b != 0)              # Skip when b is 0 (don't count as test case)
    result = divide(a, b)
    assert result * b == a


# --- .example(): Provide specific examples that are ALWAYS tested ===
@given(st.integers())
@settings(max_examples=100)
@example(0)                    # Always test with 0 first
@example(1)
@example(-1)
@example(2**63 - 1)           # Edge case: max int64
def test_edge_cases(x):
    assert is_valid_number(x)


# --- @composite: Create custom strategies from other strategies ===
from hypothesis.strategies import composite

@composite
def user_strategy(draw):
    """Custom strategy that generates valid User objects."""
    name = draw(st.text(min_size=1, max_size=50))
    age = draw(st.integers(0, 150))
    email = draw(st.email())     # Hypothesis has built-in email generator!
    return {"name": name, "age": age, "email": email}

@given(user_strategy())
def test_user_validity(user):
    assert len(user["name"]) > 0
    assert 0 <= user["age"] <= 150
    assert "@" in user["email"]


# --- Finding the minimal failing input (shrinking) ===
@given(st.integers(min_value=0, max_value=10**6))
def test_square_root(x):
    result = square_root_approximation(x)
    # hypothesis automatically SHRINKS the failing input to find the minimum!
    assert abs(result * result - x) < 1e-6


# --- Using hypothesis with pytest.mark.parametrize ===
@pytest.mark.parametrize("input_val", [1, 2, 3])
@given(st.integers())
def test_combined(input_val, x):
    """Combines explicit params (pytest) with auto-generated inputs (hypothesis)."""
    assert process(input_val, x) is not None
```

#### Mermaid: Property-Based Testing Flow

```mermaid
flowchart TD
    A["@given(st.integers(), st.text())\nDefines the input strategies"] --> B["hypothesis generates 100 random test cases\n(e.g., (42, 'hello'), (-7, ''), (999999, 'abc')]"]
    B --> C["For each generated case:\n1. Run the test function with these inputs\n2. Check all assertions"]
    C --> D{"Any assertion fails?"}
    D -->|"No (all 100 pass)"| E["✓ Test PASSES — property holds for random samples!"]
    D -->|"Yes (case 47 fails: (-5, 'bad'))"| F["hypothesis SHRINKS the failing input\nfinds minimal reproducer: ((-5,) → -5)"]
    F --> G["Report:\nFalsifying example: test_func(-5)\nSmallest failing case found!"]

    style A fill:#3069bd,color:white
    style B fill:#e8a53b,color:black
    style E fill:#4caf50,color:white
    style G fill:#f44336,color:white
```

---

## 8. Performance Comparison: pytest vs unittest vs nose2

| Framework | Install | Speed (10k tests) | Features | Plugin Ecosystem | Type Support | Recommendation |
|-----------|---------|-------------------|----------|-----------------|-------------|---------------|
| **pytest** | `pip install pytest` | ~5s | ✅⭐⭐⭐ Best | ✅ Largest | ✅ pytest-mypy-plugins | **Default choice** |
| **unittest** | Built-in | ~8s | ⭐ Basic only | ❌ Minimal | ✅ typing support | Legacy / no-install needed |
| **nose2** | `pip install nose2` | ~7s | ⭐⭐ Medium | ⭐ Some | ⭐ Partial | Deprecated (not recommended) |

> **Verdict**: Use pytest for all new projects. It's the fastest-developing, most-feature-rich option with the largest community.

---

## 9. Quizzes (25+ Questions with Answers)

### Multiple Choice & Short Answer

1. **Q: What command runs all pytest tests?**
   A: `pytest` (or `python -m pytest`)

2. **Q: How does Python's `assert` differ from TypeScript's `expect().toBe()`?**
   A: Python's `assert` is a built-in language statement — no chaining, no library needed. Type `assert x == y`. TypeScript's `expect(x).toBe(y)` is a chained matcher API requiring the jest (or chai) library.

3. **Q: What does `@pytest.fixture(scope="session")` do?**
   A: The fixture is created once and shared across the entire test session, not recreated per test.

4. **Q: Where do you patch when using `@patch("module.func")` — at the definition site or import site?**
   A: At the **import site** (where the name is looked up). If `service.py` does `from database import connect`, patch `"service.connect"`, not `"database.connect"`.

5. **Q: What is the difference between `MagicMock` and `Mock`?**
   A: `MagicMock` auto-creates sub-mocks for any attribute access (chainable). `Mock` raises `AttributeError` on unknown attributes — use it for strict tests that catch typos.

6. **Q: How do you run pytest with parallel workers?**
   A: `pip install pytest-xdist` then `pytest -n auto`

7. **Q: What does `@pytest.mark.xfail` do vs `@pytest.mark.skip`?**
   A: `xfail` marks a test as expected to fail (shows as XFAIL, not FAIL). `skip` skips the test entirely (shows as SKIP, doesn't run it).

8. **Q: How do you mock an async function in pytest?**
   A: Use `AsyncMock`: `mock_func = AsyncMock(return_value=data)` or `@patch("path.func", new_callable=AsyncMock)`.

9. **Q: What is the difference between `assert_called_with` and `assert_any_call`?**
   A: `assert_called_with(x)` checks the LAST call had args `x`. `assert_any_call(x)` checks that ANY call (anywhere in history) had args `x`.

10. **Q: How do you configure pytest to require 90% coverage and fail below it?**
    A: In pyproject.toml: `--cov=src --cov-fail-under=90` in `[tool.pytest.ini_options].addopts`

11. **Q: What does `pytest.mark.parametrize("a,b", [(1,2),(3,4)])` do?**
    A: Runs the test twice — once with `a=1, b=2` and once with `a=3, b=4`. Each is a separate test case.

12. **Q: What is the purpose of `conftest.py` at different directory levels?**
    A: Fixtures/hooks defined in a conftest.py are scoped to tests in that directory and subdirectories. Root conftest = project-wide, module-level conftest = directory-specific.

13. **Q: What does `side_effect = [ValueError(), TypeError()]` do on a mock?**
    A: First call raises ValueError, second call raises TypeError, then subsequent calls return MagicMock (empty behavior).

14. **Q: How do you use `spec_set` on a MagicMock?**
    A: `MagicMock(spec_set=RealClass)` — allows no attribute access or setting that doesn't exist on RealClass. Catches typos in test code at runtime.

15. **Q: What is property-based testing and which library implements it in Python?**
    A: Property-based testing generates random inputs and checks that properties (invariants) hold for all of them. Library: `hypothesis` (`pip install hypothesis`). TypeScript equivalent: `fast-check`.

16. **Q: How do you run only tests marked as @pytest.mark.slow?**
    A: `pytest -m slow`

17. **Q: What's the default scope of a pytest fixture?**
    A: `"function"` — created fresh for each test function.

18. **Q: How does `pytest.raises` differ from Jest's `expect().rejects.toThrow()`?**
    A: Same concept — both catch exceptions raised during a block. Python uses `with pytest.raises(ValueError): func()`. TypeScript uses `await expect(func()).rejects.toThrow()`.

19. **Q: What is `pytest_generate_tests` hook used for?**
    A: Dynamic fixture generation at collection time — programmatic parametrize logic.

20. **Q: How do you mock a class constructor (i.e., patch what `ClassName()` returns)?**
    A: `@patch("module.ClassName")` — the decorator's arg gets the MagicMock for the class. Set `mock.return_value = fake_instance`.

21. **Q: What happens if you use `python -O` to run your tests?**
    A: Python strips all `assert` statements! Assertions won't be evaluated. Always test without the `-O` flag, or use `if not condition: raise AssertionError(...)` for critical checks.

22. **Q: How do you mock a property (`@property`) in Python?**
    A: Use `PropertyMock`: `@patch.object(Class, "prop", new_callable=PropertyMock)`.

23. **Q: What is the difference between `return_value` and `side_effect` on a mock?**
    A: `return_value` always returns the same static value. `side_effect` can be a callable (invoked each time), an iterable (successive values), or an exception (raised).

24. **Q: How do you generate coverage for both source and tests in pytest?**
    A: `pytest --cov=src --cov=tests --cov-report=term-missing`

25. **Q: When would you use `unittest` over `pytest`?**
    A: Only when you cannot install third-party packages (unittest is built into Python's standard library). For all other cases, pytest is recommended for its flexibility and feature set.

26. **Q: What does `@given(st.integers(), st.text())` do in hypothesis?**
    A: Generates random pairs of (integer, string) inputs across 100 examples by default, testing the function against diverse inputs to find edge cases that manual tests might miss.

27. **Q: How do you mock a context manager for `with db.session() as session:`?**
    A: `@patch.object(db.Session, "__enter__", return_value=mock_session)` and `@patch.object(db.Session, "__exit__", return_value=False)`. Or simpler: `@patch("module.db.Session")` and configure the mock's `__enter__`/`__exit__` attributes.

---

## 10. Exercises (20+ Problems with Solutions)

### Exercise 1: Write pytest Tests for a Math Module
<details><summary>📝 Problem — Click to expand</summary>

```python
# math_utils.py — Code under test
def add(a, b): return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b
def divide(a, b):
    if b == 0: raise ValueError("Division by zero")
    return a / b
```

**Tasks:**
1. Write `test_math_utils.py` with basic tests for each function
2. Add parametrized tests covering edge cases (zero, negative, large numbers)
3. Test the division-by-zero exception
4. Run with coverage: `pytest --cov=math_utils`</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# test_math_utils.py
import pytest
from math_utils import add, subtract, multiply, divide

class TestAdd:
    def test_basic(self):
        assert add(1, 2) == 3
    
    @pytest.mark.parametrize("a,b,expected", [
        (0, 0, 0),
        (-1, -1, -2),
        (100, 200, 300),
        (999999, 1, 1000000),
    ])
    def test_various_inputs(self, a, b, expected):
        assert add(a, b) == expected

class TestDivide:
    def test_normal(self):
        assert divide(10, 2) == 5.0
    
    def test_zero_division(self):
        with pytest.raises(ValueError, match="Division by zero"):
            divide(1, 0)
```
</details>

---

### Exercise 2: Fixtures for Database Tests
<details><summary>📝 Problem — Click to expand</summary>

Write a fixture that creates an in-memory SQLite database with a users table, seeds it with test data, and yields the connection. Use it across multiple tests.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# conftest.py
import pytest
import sqlite3

@pytest.fixture
def db():
    """Create in-memory SQLite database with seed data."""
    conn = sqlite3.connect(":memory:")
    conn.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)")
    conn.execute("INSERT INTO users VALUES (1, 'Alice', 'alice@example.com')")
    conn.execute("INSERT INTO users VALUES (2, 'Bob', 'bob@example.com')")
    conn.commit()
    yield conn
    conn.close()

# test_db.py
def test_get_all_users(db):
    rows = db.execute("SELECT * FROM users").fetchall()
    assert len(rows) == 2

def test_get_user_name(db):
    row = db.execute("SELECT name FROM users WHERE id = 1").fetchone()
    assert row[0] == "Alice"
```
</details>

---

### Exercise 3: Mock External API Calls
<details><summary>📝 Problem — Click to expand</summary>

Write a function that calls an external REST API and test it by patching the HTTP client.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# user_service.py
import requests

def get_user(user_id):
    response = requests.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()

# test_user_service.py
from unittest.mock import patch, MagicMock
from user_service import get_user

@patch("user_service.requests.get")
def test_get_user(mock_get):
    mock_response = MagicMock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 1, "name": "Alice"}
    mock_get.return_value = mock_response
    
    result = get_user(1)
    
    assert result["name"] == "Alice"
    mock_get.assert_called_once_with("https://api.example.com/users/1")
```
</details>

---

### Exercise 4: Parametrized Tests with Multiple Parameters
<details><summary>📝 Problem — Click to expand</summary>

Write tests for a URL validation function that accepts multiple input/output pairs.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# test_url_validation.py
import pytest
from url_validator import is_valid_url

@pytest.mark.parametrize("url,expected", [
    ("https://example.com", True),
    ("http://test.org/path", True),
    ("not-a-url", False),
    ("ftp://files.example.com", False),  # Only http/https allowed
    ("https://sub.domain.co.uk/page?query=1", True),
    ("", False),
])
def test_is_valid_url(url, expected):
    assert is_valid_url(url) == expected
```
</details>

---

### Exercise 5: Conftest Hierarchy — Shared Fixtures Across Directories
<details><summary>📝 Problem — Click to expand</summary>

Create a conftest.py hierarchy where API tests share an `auth_token` fixture and database tests share a `db_connection` fixture.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# tests/conftest.py (project-wide)
import pytest

@pytest.fixture(scope="session")
def config():
    return {"database_url": "sqlite:///:memory:", "api_base": "http://localhost:8000"}

# tests/api/conftest.py (API-specific)
@pytest.fixture
def auth_token():
    return "test-jwt-token-12345"

@pytest.fixture
def authenticated_api_client(auth_token, config):
    headers = {"Authorization": f"Bearer {auth_token}"}
    return {"base_url": config["api_base"], "headers": headers}

# tests/api/test_endpoints.py
def test_get_profile(authenticated_api_client):
    client = authenticated_api_client
    assert "Authorization" in client["headers"]
```
</details>

---

### Exercise 6: Mock with side_effect for Dynamic Responses
<details><summary>📝 Problem — Click to expand</summary>

Write a test that verifies a retry mechanism works by making a mock return different values on successive calls.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# service.py
import requests

def fetch_with_retry(url, max_retries=3):
    for i in range(max_retries):
        try:
            response = requests.get(url)
            response.raise_for_status()
            return response.json()
        except requests.RequestException:
            if i == max_retries - 1:
                raise
    return None

# test_service.py
from unittest.mock import patch
from service import fetch_with_retry

@patch("service.requests.get")
def test_fetch_with_retry(mock_get):
    # First two calls fail, third succeeds
    mock_get.side_effect = [
        requests.RequestException(),   # 1st attempt: error
        requests.RequestException(),   # 2nd attempt: error
        MagicMock(status_code=200, json=lambda: {"status": "ok"}),  # 3rd: success!
    ]
    
    result = fetch_with_retry("https://api.example.com")
    
    assert result == {"status": "ok"}
    assert mock_get.call_count == 3  # Called exactly 3 times
```
</details>

---

### Exercise 7: Async Test with pytest-asyncio
<details><summary>📝 Problem — Click to expand</summary>

Write an async test that fetches data from a mock endpoint using pytest-asyncio.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# test_async.py
import pytest
import asyncio
import aiohttp
from unittest.mock import AsyncMock, patch

@patch("aiohttp.ClientSession.get", new_callable=AsyncMock)
@pytest.mark.asyncio
async def test_fetch_user(mock_get):
    mock_resp = AsyncMock()
    mock_resp.json.return_value = {"name": "Alice"}
    mock_resp.status = 200
    mock_get.return_value = mock_resp
    
    async with aiohttp.ClientSession() as session:
        result = await session.get("/api/users/1")
        data = await result.json()
    
    assert data["name"] == "Alice"
    mock_get.assert_awaited_once_with("/api/users/1")
```
</details>

---

### Exercise 8: Property-Based Testing with hypothesis
<details><summary>📝 Problem — Click to expand</summary>

Write a property-based test that verifies sorting is commutative and idempotent for any list of integers.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# test_sorting.py
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_is_idempotent(lst):
    """Sorting a sorted list produces the same result."""
    first = sorted(lst)
    second = sorted(first)
    assert first == second

@given(st.lists(st.integers(), min_size=1))
def test_sorted_contains_same_elements(lst):
    """Sorted list contains exactly the same elements as original."""
    s = sorted(lst)
    assert set(s) == set(lst)
    assert len(s) == len(lst)
```
</details>

---

### Exercise 9: Mock with spec to Catch Typos
<details><summary>📝 Problem — Click to expand</summary>

Write a test using `spec` that catches a typo in the mock attribute name.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# user.py
class User:
    def get_name(self): return "Alice"
    def get_age(self): return 30

# test_user.py
from unittest.mock import MagicMock, patch

@patch("user.User")
def test_typed_mock(mock_user_class):
    instance = mock_user_class(spec=User)
    
    # These work:
    instance.get_name()   # ✅ OK — exists on User
    instance.get_age()    # ✅ OK
    
    # This raises AttributeError — typo detected!
    with pytest.raises(AttributeError):
        instance.get_emal()   # ❌ Typo: "emal" vs "email"
```
</details>

---

### Exercise 10: Coverage Report Configuration
<details><summary>📝 Problem — Click to expand</summary>

Configure pyproject.toml for pytest with coverage that generates HTML, XML, and terminal reports.</details>

<details><summary>✅ Solution — Click to expand</summary>

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = [
    "--cov=src",
    "--cov-report=term-missing",
    "--cov-report=html:htmlcov",
    "--cov-report=xml:coverage.xml",
    "--cov-branch",
    "--cov-fail-under=90",
]

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "@abstractmethod",
]
```
</details>

---

### Exercise 11: Custom pytest Hook — Log Test Duration
<details><summary>📝 Problem — Click to expand</summary>

Write a conftest.py hook that logs how long each test takes.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# conftest.py
import pytest
import time

def pytest_runtest_setup(item):
    item._start_time = time.time()

def pytest_runtest_logreport(report):
    if report.when == "call" and hasattr(report, "duration"):
        duration = time.time() - getattr(report.node, '_start_time', 0)
        status = "PASS" if report.passed else ("SKIP" if report.skipped else "FAIL")
        print(f"\n[{status}] {report.nodeid} — {duration:.3f}s")
```
</details>

---

### Exercise 12: Mock with Chained Calls (Fluent Interface)
<details><summary>📝 Problem — Click to expand</summary>

Mock a chained API call like `client.users.filter(active=True).get()`.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
from unittest.mock import patch, MagicMock

@patch("app.service.Client")
def test_chained_api(mock_client_class):
    # Configure the chain: users → filter → get
    mock_users = MagicMock()
    mock_filter = MagicMock()
    
    mock_instance = MagicMock()
    mock_instance.users = mock_users       # client.users → mock_users
    mock_users.filter.return_value = mock_filter  # .filter(...) → mock_filter
    mock_filter.get.return_value = [{"id": 1}]  # .get() → [result]
    mock_client_class.return_value = mock_instance
    
    result = app.service.Client().users.filter(active=True).get()
    
    assert result == [{"id": 1}]
    mock_users.filter.assert_called_once_with(active=True)
    mock_filter.get.assert_called_once()
```
</details>

---

### Exercise 13: Session-Scoped Fixture for Expensive Setup
<details><summary>📝 Problem — Click to expand</summary>

Create a session-scoped fixture that starts a test server and yields its base URL.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# conftest.py
import pytest

@pytest.fixture(scope="session")
def test_server():
    """Start server once for entire test session."""
    server = start_test_server(port=9876)
    base_url = f"http://localhost:9876"
    yield base_url
    server.shutdown()  # Runs only at end of ALL tests

# tests/test_api.py
def test_users_endpoint(test_server):
    response = requests.get(f"{test_server}/users")
    assert response.status_code == 200

def test_posts_endpoint(test_server):
    response = requests.get(f"{test_server}/posts")
    assert response.status_code == 200
    # Both tests share the SAME server instance — no re-setup!
```
</details>

---

### Exercise 14: Class-Based Fixtures for E2E Tests
<details><summary>📝 Problem — Click to expand</summary>

Create a class of E2E test fixtures that manage user accounts.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# tests/fixtures/e2e_users.py
import pytest

class E2EUserFixtures:
    @pytest.fixture
    def admin_user(self):
        return self._create_user("admin@test.com", role="admin")
    
    @pytest.fixture
    def normal_user(self):
        return self._create_user("user@test.com", role="user")
    
    @pytest.fixture
    def deleted_user(self):
        user = self._create_user("deleted@test.com", role="user")
        self._delete_user(user["id"])
        yield user
    
    def _create_user(self, email, role):
        return {"email": email, "role": role}  # Simplified
    
    def _delete_user(self, user_id):
        pass  # In reality: API call

# In conftest.py:
from tests.fixtures.e2e_users import E2EUserFixtures
e2e_users = E2EUserFixtures()

# In test files:
def test_admin_can_delete_user(admin_user):
    assert admin_user["role"] == "admin"

def test_deleted_user_cannot_login(deleted_user):
    assert deleted_user["email"] == "deleted@test.com"
```
</details>

---

### Exercise 15: Mock with AsyncMock for Event Loop Test
<details><summary>📝 Problem — Click to expand</summary>

Test an async function that awaits multiple async dependencies.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# app.py
async def process_users():
    users = await fetch_users()     # async
    posts = await fetch_posts()     # async
    return {"users": users, "posts": posts}

# test_app.py
from unittest.mock import AsyncMock, patch
import asyncio

@patch("app.fetch_users", new_callable=AsyncMock)
@patch("app.fetch_posts", new_callable=AsyncMock)
@pytest.mark.asyncio
async def test_process_users(mock_posts, mock_users):
    mock_users.return_value = [{"id": 1}]
    mock_posts.return_value = [{"id": "post-1"}]
    
    result = await process_users()
    
    assert result == {"users": [{"id": 1}], "posts": [{"id": "post-1"}]}
    mock_users.assert_awaited_once()
    mock_posts.assert_awaited_once()
```
</details>

---

### Exercise 16: Multiple Marker Combinations
<details><summary>📝 Problem — Click to expand</summary>

Create a test suite with tests tagged with different markers and verify marker selection works correctly.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# conftest.py (register the markers)
def pytest_configure(config):
    config.addinivalue_line("markers", "slow: slow tests")
    config.addinivalue_line("markers", "db: requires database")
    config.addinivalue_line("markers": "unit: fast unit tests")

# test_suite.py
import pytest

@pytest.mark.unit
def test_add_simple():
    assert add(1, 2) == 3

@pytest.mark.slow
@pytest.mark.db
def test_db_migration():
    run_migration()

@pytest.mark.unit
@pytest.mark.slow
def test_complex_algorithm():
    complex_compute()

# CLI commands:
# pytest -m "unit and not slow"     → only fast unit tests
# pytest -m "db or slow"            → db OR slow tests
# pytest -m "not slow"              → all except slow
# pytest --strict-markers           → fail if any unregistered marker used
```
</details>

---

### Exercise 17: Mock with call_args_list Verification
<details><summary>📝 Problem — Click to expand</summary>

Verify that a function makes calls in the exact right order.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# service.py
def deploy_pipeline():
    validate_config()
    build_artifact()
    push_to_registry()
    deploy_to_prod()

# test_service.py
from unittest.mock import patch, call

@patch("service.validate_config")
@patch("service.build_artifact")
@patch("service.push_to_registry")
@patch("service.deploy_to_prod")
def test_deploy_order(mock_deploy, mock_push, mock_build, mock_validate):
    service.deploy_pipeline()
    
    # Verify exact call order:
    expected_calls = [
        call(),           # validate_config()
        call(),           # build_artifact()
        call(),           # push_to_registry()
        call(),           # deploy_to_prod()
    ]
    assert mock_validate.call_args_list == expected_calls[:1]
    assert mock_build.call_args_list == expected_calls[1:2]
    assert mock_push.call_args_list == expected_calls[2:3]
    assert mock_deploy.call_args_list == expected_calls[3:]
```
</details>

---

### Exercise 18: hypothesis Strategy for Custom Objects
<details><summary>📝 Problem — Click to expand</summary>

Generate valid email addresses and verify an email validation function works with all of them.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
from hypothesis import given, strategies as st

@given(st.emails())  # Hypothesis generates VALID emails!
def test_email_validator_accepts_valid_emails(email):
    assert is_valid_email(email) is True

@given(st.text(min_size=1))
def test_email_validator_rejects_invalid(tldless):
    assume("@" not in tldless)  # Skip if it accidentally contains @
    assert is_valid_email(tldless) is False

@given(st.emails(), st.integers(0, 1000))
@example("test@example.com")   # Always include this specific example
@settings(max_examples=200)
def test_email_length(email, n):
    extended = f"{email}.{'x' * n}"
    if len(extended) < 320:  # Email max length is 320 chars
        assert len(extended) >= len(email)
```
</details>

---

### Exercise 19: Complete Mock of Class with Multiple Methods
<details><summary>📝 Problem — Click to expand</summary>

Mock a complete class with multiple async methods, return values, and side effects.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# payment.py
class PaymentProcessor:
    async def validate(self, amount):
        if amount <= 0: raise ValueError("Invalid amount")
        return True
    
    async def charge(self, user_id, amount):
        # ... real logic ...
        pass
    
    async def refund(self, transaction_id):
        # ... real logic ...
        pass

# test_payment.py
from unittest.mock import AsyncMock, patch
import pytest

class TestPaymentProcessor:
    @pytest.fixture
    def mock_processor(self):
        processor = AsyncMock(spec=PaymentProcessor)
        processor.validate = AsyncMock(side_effect=lambda a: True if a > 0 else ValueError("Invalid"))
        processor.charge = AsyncMock(return_value={"tx_id": "txn-123"})
        processor.refund = AsyncMock(return_value={"status": "refunded"})
        return processor

    @pytest.mark.asyncio
    async def test_valid_charge(self, mock_processor):
        mock_processor.validate.return_value = True
        result = await mock_processor.charge("user-1", 100)
        assert result["tx_id"] == "txn-123"
        mock_processor.validate.assert_called_with(100)

    @pytest.mark.asyncio
    async def test_invalid_amount(self, mock_processor):
        mock_processor.validate.side_effect = ValueError("Invalid amount")
        with pytest.raises(ValueError):
            await mock_processor.charge("user-1", -50)
```
</details>

---

### Exercise 20: Test Directory Structure for a Large Project
<details><summary>📝 Problem — Click to expand</summary>

Design a test directory structure for a project with auth, payments, and API layers. Include conftest.py at each level.</details>

<details><summary>✅ Solution — Click to expand</summary>

```
project/
├── src/
│   ├── auth/
│   ├── payments/
│   └── api/
├── tests/
│   ├── conftest.py                    # Session-scoped: config, event_loop
│   │
│   ├── unit/                          # Fast, no external deps
│   │   ├── conftest.py                # Unit-level mocks
│   │   ├── test_auth_utils.py
│   │   └── test_payment_calcs.py
│   │
│   ├── integration/                   # Real DB/API
│   │   ├── conftest.py                # Live containers, fixtures
│   │   ├── test_auth_flow.py
│   │   └── test_payment_flow.py
│   │
│   ├── api/                           # API endpoint tests
│   │   ├── conftest.py                # Auth tokens, client setup
│   │   └── test_endpoints.py
│   │
│   └── e2e/                           # Browser-level
│       ├── conftest.py                # Browser driver
│       └── test_full_checkout.py
```

Each `conftest.py` provides fixtures scoped to its directory:

```python
# tests/conftest.py (session-scoped, project-wide)
import pytest
@pytest.fixture(scope="session")
def config(): return {"db": "sqlite:///:memory:"}

# tests/unit/conftest.py
import pytest
from unittest.mock import MagicMock
@pytest.fixture
def mock_logger(): return MagicMock()

# tests/integration/conftest.py
import pytest
@pytest.fixture(scope="module")
def postgres_container():
    container = start_postgres()
    yield container
    container.stop()
```
</details>

---

### Exercise 21: Use @patch.object to Mock Instance Methods
<details><summary>📝 Problem — Click to expand</summary>

Mock a method on an existing instance (not the class).</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
from unittest.mock import patch, MagicMock

class Service:
    def __init__(self):
        self.helper = Helper()
    
    def process(self):
        return self.helper.transform("input")

class Helper:
    def transform(self, data):
        return f"transformed:{data}"

# Test with patch.object on the INSTANCE
def test_process_with_mock():
    service = Service()
    
    with patch.object(service.helper, "transform", return_value="mocked_output") as mock:
        result = service.process()
        
        assert result == "mocked_output"
        mock.assert_called_once_with("input")
```
</details>

---

### Exercise 22: Test with Capfd / Capsys (Capture stdout/stderr)
<details><summary>📝 Problem — Click to expand</summary>

Test that a function prints the correct output to stdout.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# utils.py
def greet(name):
    print(f"Hello, {name}!")
    return f"Greeting sent to {name}"

# test_utils.py
def test_greet_output(capsys):
    """capsys fixture captures stdout/stderr."""
    result = greet("Alice")
    
    captured = capsys.readouterr()
    assert captured.out == "Hello, Alice!\n"
    assert result == "Greeting sent to Alice"

def test_greet_stderr(capsys):
    import sys
    
    def error_fn():
        print("Error occurred", file=sys.stderr)
    
    error_fn()
    captured = capsys.readouterr()
    assert captured.err == "Error occurred\n"
```
</details>

---

### Exercise 23: Monkeypatch Environment Variables in Tests
<details><summary>📝 Problem — Click to expand</summary>

Test code that reads `os.environ["API_KEY"]` without permanently modifying the environment.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# app.py
import os

def get_api_key():
    return os.environ.get("API_KEY", "default-key")

# test_app.py
def test_get_api_key(monkeypatch):
    """monkeypatch fixture safely modifies attributes."""
    monkeypatch.setenv("API_KEY", "test-key-123")
    
    assert get_api_key() == "test-key-123"
    
    # After the test, API_KEY is automatically restored!

def test_missing_api_key(monkeypatch):
    """Test behavior when env var is absent."""
    monkeypatch.delenv("API_KEY", raising=False)
    
    assert get_api_key() == "default-key"
```
</details>

---

### Exercise 24: Write a Custom pytest Plugin
<details><summary>📝 Problem — Click to expand</summary>

Create a plugin that adds a `--red-green` flag: green output for passing tests, red for failing.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# pytest_redgreen.py (custom plugin)
def pytest_addoption(parser):
    parser.addoption(
        "--red-green",
        action="store_true",
        default=False,
        help="Color output: green for pass, red for fail"
    )

def pytest_configure(config):
    if config.getoption("--red-green"):
        config.pluginmanager.register(GreenRedReporter())

class GreenRedReporter:
    def pytest_terminal_summary(self, terminalreporter):
        for key in sorted(terminalreporter.stats):
            stat = terminalreporter.stats[key]
            color = "green" if key == "passed" else "red" if key == "failed" else None
            if color:
                msg = f"{key}: {len(stat)}"
                # Apply ANSI color codes for terminal output
```
</details>

---

### Exercise 25: Async Context Manager Test with Mocking
<details><summary>📝 Problem — Click to expand</summary>

Write a test for an async context manager that opens/closes a database connection.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# db_manager.py
from contextlib import asynccontextmanager

@asynccontextmanager
async def db_connection(url):
    conn = await create_connection(url)  # async open
    try:
        yield conn
    finally:
        await conn.close()  # guaranteed cleanup

# test_db_manager.py
from unittest.mock import AsyncMock, patch
import pytest

@pytest.mark.asyncio
async def test_db_connection_closes_on_success():
    mock_conn = AsyncMock()
    
    with patch("db_manager.create_connection", new_callable=AsyncMock) as mock_create:
        mock_create.return_value = mock_conn
        
        async with db_connection("postgresql://test") as conn:
            assert conn is mock_conn
        
        # Connection was closed after the 'with' block
        mock_conn.close.assert_awaited_once()

@pytest.mark.asyncio
async def test_db_connection_closes_on_exception():
    mock_conn = AsyncMock()
    
    with patch("db_manager.create_connection", new_callable=AsyncMock) as mock_create:
        mock_create.return_value = mock_conn
        
        with pytest.raises(ValueError):
            async with db_connection("postgresql://test") as conn:
                raise ValueError("Test error!")
        
        # Even on exception, connection was closed!
        mock_conn.close.assert_awaited_once()
```
</details>

---

### Exercise 26: Combine pytest Subtests with Regular Tests
<details><summary>📝 Problem — Click to expand</summary>

Use pytest-subtests to run multiple sub-checks within a single test function.</details>

<details><summary>✅ Solution — Click to expand</summary>

```python
# Requires: pip install pytest-subtests
def test_multiple_user_validations(subtests):
    users = [
        {"name": "Alice", "age": 30, "valid": True},
        {"name": "", "age": -1, "valid": False},   # Invalid name and age
        {"name": "Bob", "age": 200, "valid": False}, # Age too high
    ]
    
    for user in users:
        with subtests.test(name=user["name"], age=user["age"]):
            errors = validate_user(user)
            if user["valid"]:
                assert len(errors) == 0
            else:
                assert len(errors) > 0
```
</details>

---

## Quick Reference Card

### pytest Command Cheat Sheet

| Goal | Command |
|------|---------|
| Run all tests | `pytest` |
| Run specific file | `pytest tests/test_math.py` |
| Match test name pattern | `pytest -k "add"` |
| Marked tests only | `pytest -m slow` |
| Stop on first failure | `pytest --maxfail=1` |
| With coverage | `pytest --cov=src` |
| Parallel execution | `pytest -n auto` |
| Verbose output | `pytest -v` or `-vv` |
| HTML report | `pytest --html=report.html` |
| Only failed tests (rerun) | `pytest --lf` |
| Run last failed first | `pytest --ff` |

### Mock Method Cheat Sheet

| Goal | Code |
|------|------|
| Create mock | `m = MagicMock()` |
| Set return value | `m.return_value = 42` |
| Set side effect (callable) | `m.side_effect = lambda x: x*2` |
| Set side effect (exception) | `m.side_effect = ValueError("err")` |
| Assert called once | `m.assert_called_once()` |
| Assert called with args | `m.assert_called_with(1, 2)` |
| Assert any call | `m.assert_any_call(3)` |
| Assert not called | `m.assert_not_called()` |
| Get call count | `m.call_count` |
| Get all calls | `m.call_args_list` |
| Reset mock | `m.reset_mock()` |
| Async mock | `m = AsyncMock(return_value=data)` |
| Spec enforcement | `MagicMock(spec=RealClass)` |

### Fixture Scope Cheat Sheet

| Scope | Created | Ideal For |
|-------|---------|-----------|
| `function` (default) | Before each test | Per-test data, mocks |
| `class` | Once per test class | Database connections per class |
| `module` | Once per file | Expensive setup, config objects |
| `session` | Once for entire run | Shared server, DB pools |

---

> **Next:** [Module 12 — Async/Await Patterns (asyncio, coroutines, event loop)](./12-async-await-patterns.md)
