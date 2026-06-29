# Module 26 — Python Glossary (Tutorial Reference)

> **Source:** Based on [Python Tutorial Glossary](https://docs.python.org/3/glossary.html) (Python 3.14.6).  
> This glossary captures the essential terms from the Python tutorial, organized for quick reference. Each entry maps back to where the term first appears in the tutorial.

---

## Table of Contents

- [A — C](#a--c)
- [D — F](#d--f)
- [G — I](#g--i)
- [L — N](#l--n)
- [P — S](#p--s)
- [T — Z](#t--z)

---

## A — C

### abstract base class (ABC)

Abstract base classes complement duck-typing by providing a way to define interfaces when other techniques like `hasattr()` would be clumsy or **wrong** (for example with magic methods). ABCs introduce virtual subclasses, which are classes that don't inherit from a class but are still recognized by `isinstance()` and `issubclass()`. See the `abc` module.

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `interface` / `abstract class`

### argument

A value passed to a function (or method) when calling the function. There are two kinds:

| Type | Description | Example |
|------|-------------|---------|
| **Positional** | Passed by position | `complex(3, 5)` |
| **Keyword** | Passed as `name=value` | `complex(real=3, imag=5)` |

> **Tutorial link:** Chapter 4 — More Control Flow Tools

### attribute

A value associated with an object which is usually referenced by name using dotted expressions. For example, `o.a` references the attribute `a` of object `o`.

> **Tutorial link:** Chapter 9 — Classes

### binary file

A file object able to read and write bytes-like objects. Examples include files opened in binary mode (`'rb'`, `'wb'`), `sys.stdin.buffer`, and instances of `io.BytesIO`.

> **Tutorial link:** Chapter 7 — Input and Output

### bytecode

Python source code is compiled into bytecode, the internal representation of a Python program in the CPython interpreter. The bytecode is also cached in `.pyc` files for faster subsequent execution.

> **Tutorial link:** Chapter 1 — Whetting Your Appetite  
> **Note:** Not to be confused with machine code — Python runs on a virtual machine.

### callable

An object that can be called, possibly with arguments, using the syntax `callable(arg1, arg2, ...)`. Functions, methods, and classes implementing `__call__()` are callables.

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** `(arg: T) => R` (function type)

### class

A template for creating user-defined objects. Class definitions normally contain method definitions which operate on instances of the class.

```python
class User:
    def __init__(self, name: str):
        self.name = name

    def greet(self) -> str:
        return f"Hello, {self.name}!"
```

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `class`

### closure variable

A free variable referenced from a nested scope that is defined in an outer scope rather than being resolved at runtime from globals or built-in namespaces.

```python
def outer():
    x = 0
    def inner():
        nonlocal x
        x += 1
        return x
    return inner
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools

### complex number

An extension of real numbers expressed as a sum of a real part and an imaginary part (e.g., `3+1j`). Python has built-in support; use `cmath` for advanced operations.

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python

### concurrency

The ability of a computer program to perform multiple tasks at the same time. Python provides:

| Library | Purpose |
|---------|---------|
| `asyncio` | Asynchronous tasks and coroutines |
| `threading` | OS threads |
| `multiprocessing` | OS processes |

> **Tutorial link:** Chapter 12 — Async/Await Patterns

### context manager

An object which implements the context management protocol (`__enter__()` and `__exit__()`) and controls the environment seen in a `with` statement.

```python
with open('file.txt', 'r') as f:
    data = f.read()
# File is automatically closed here
```

> **Tutorial link:** Chapter 8 — Errors and Exceptions

### coroutine

A more generalized form of subroutine that can be entered, exited, and resumed at many different points. Implemented with `async def`.

> **Tutorial link:** Chapter 12 — Async/Await Patterns  
> **TS equivalent:** `async function` / `Promise`

### CPython

The canonical implementation of the Python programming language, as distributed on python.org. Distinguished from other implementations like Jython or IronPython.

> **Tutorial link:** Throughout

---

## D — F

### decorator

A function returning another function, usually applied using the `@wrapper` syntax. Common examples include `classmethod()` and `staticmethod()`.

```python
@staticmethod
def helper():
    pass

# Equivalent to:
def helper():
    pass
helper = staticmethod(helper)
```

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `@decorator` decorator (very similar!)

### descriptor

Any object which defines the methods `__get__()`, `__set__()`, or `__delete__()`. Descriptors are the basis for properties, class methods, static methods, and reference to super classes.

> **Tutorial link:** Chapter 9 — Classes  
> **Deep dive:** Module 03 — OOP Deep Dive

### dictionary

An associative array (hash map) where arbitrary keys are mapped to values. Keys must have `__hash__()` and `__eq__()` methods.

```python
mapping = {"a": 1, "b": 2}
```

> **Tutorial link:** Chapter 5 — Data Structures  
> **TS equivalent:** `{[K]: V}` / `Record<K, V>`

### dictionary comprehension

A compact way to process elements and return a dictionary:

```python
squares = {n: n ** 2 for n in range(10)}
# {0: 0, 1: 1, 2: 4, ..., 9: 81}
```

> **Tutorial link:** Chapter 5 — Data Structures  
> **TS equivalent:** `Object.fromEntries()` + `map()`

### docstring

A string literal which appears as the first expression in a class, function, or module. It is put into the `__doc__` attribute for introspection.

```python
def greet(name: str) -> str:
    """Return a greeting message for the given name."""
    return f"Hello, {name}!"

print(greet.__doc__)  # Return a greeting message...
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** JSDoc comments (`/** ... */`)

### duck-typing

A programming style that does not look at an object's type to determine if it has the right interface. "If it looks like a duck and quacks like a duck, it must be a duck." Emphasizes interfaces rather than specific types.

> **Tutorial link:** Throughout  
> **TS equivalent:** Structural typing (interfaces)

### dunder

Informal shorthand for "double underscore". For example, `__init__` is pronounced "dunder init". Used when talking about special methods.

> **Tutorial link:** Chapter 9 — Classes

### EAFP

**E**asier to ask for **F**orgiveness than **P**ermission. A common Python coding style that assumes valid keys/attributes exist and catches exceptions if the assumption is false. Characterized by many `try`/`except` statements.

> Contrasts with TypeScript's more LBYL (Look Before You Leap) approach of null checks.

```python
# EAFP style
try:
    value = d['key']
except KeyError:
    value = default
```

### expression

A piece of syntax which can be evaluated to some value. Accumulation of literals, names, attribute access, operators, or function calls that return a value. Unlike statements, expressions always produce a value.

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `expression` (same concept)

### f-string

String literals prefixed with `f` or `F` ("formatted string literals"). Allows embedded expressions:

```python
name = "World"
message = f"Hello, {name}!"  # Hello, World!
```

> **Tutorial link:** Chapter 7 — Input and Output  
> **TS equivalent:** Template literals (`` ` ``) — identical concept!

### file object

An object exposing a file-oriented API (`read()`, `write()`) to an underlying resource. Also called "file-like objects" or "streams". Three categories: raw binary, buffered binary, and text files.

> **Tutorial link:** Chapter 7 — Input and Output  
> **TS/Node.js equivalent:** `fs.FileHandle` / `BufferedStream`

### floor division

Mathematical division that rounds down to the nearest integer using the `//` operator.

```python
11 // 4   # 2 (not 2.75)
(-11) // 4  # -3 (rounds downward)
```

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `Math.floor(a / b)`

### free threading

A threading model where multiple threads can run Python bytecode simultaneously within the same interpreter. Introduced in Python 3.13 via `--disable-gil` build configuration (PEP 703). Contrasts with GIL.

> **Tutorial link:** Chapter 12 — Async/Await Patterns

### free variable

A variable used in a namespace which is not a local variable in that namespace. See "closure variable" for a specific kind of free variable.

> **Tutorial link:** Chapter 4 — More Control Flow Tools

### function

A series of statements that returns some value to a caller. Can accept zero or more arguments.

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** `function` / arrow function `=>`

### garbage collection

The process of freeing memory when it is not used anymore. Python performs GC via reference counting and a cyclic garbage collector that detects and breaks reference cycles.

> **Tutorial link:** Chapter 9 — Classes  
> **VS TypeScript:** V8 uses mark-and-sweep; CPython uses reference counting + cycle detector

---

## G — I

### generator

A function or iterator that produces values lazily using `yield`. Can be resumed from where it left off.

```python
def count_up_to(n):
    i = 0
    while i < n:
        yield i
        i += 1

for num in count_up_to(5):
    print(num)  # 0, 1, 2, 3, 4
```

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `Generator` type / iterator with `next()`

### generator expression

An expression that returns an iterator, looking like a list comprehension but with parentheses:

```python
total = sum(i * i for i in range(10))  # 285
```

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `Array.from()` + spread or custom generator function

### generic type

A type that can be parameterized, typically a container class such as `list` or `dict`. Used for type hints.

```python
from typing import List, Dict

names: List[str] = ["a", "b"]
mapping: Dict[str, int] = {"x": 1}
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** Generics `<T>` — nearly identical syntax!

### GIL (Global Interpreter Lock)

The mechanism used by CPython to assure that only one thread executes Python bytecode at a time. Simplifies the implementation but limits true parallelism in multi-threaded code. Can be disabled in Python 3.13+ with `--disable-gil`.

> **Tutorial link:** Chapter 5 — Concurrency & Parallelism  
> **TS equivalent:** N/A (Node.js is single-threaded by design)

### hashable

An object that has a hash value which never changes during its lifetime (needs `__hash__()`) and can be compared to other objects (`__eq__()`). Hashable objects can be dictionary keys or set members.

> **Tutorial link:** Chapter 5 — Data Structures  
> **TS equivalent:** Any type can be a keyof record, but Python requires hashability

### immutable

An object with a fixed value. Numbers, strings, and tuples are immutable. Cannot be altered after creation; a new object must be created for different values.

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `readonly` / `const` primitives

### interpreter

Python is an interpreted language. Source files can be run directly without explicitly creating an executable. This means shorter development/debug cycles than compiled languages, though programs generally run more slowly.

> **Tutorial link:** Chapter 1 — Whetting Your Appetite

### interactive mode

Python has an interactive interpreter (`>>>` prompt) for testing ideas or inspecting modules on the fly. Launch with `python` and no arguments.

> **Tutorial link:** Chapter 2 — Using the Python Interpreter  
> **TS equivalent:** `node -e` / REPL in Node.js

### iterable

An object capable of returning its members one at a time. Examples: `list`, `str`, `tuple`, `dict`, file objects, and any class with `__iter__()` method.

```python
for item in [1, 2, 3]:
    print(item)
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** `Iterable<T>` / any object with `[Symbol.iterator]`

### iterator

An object representing a stream of data. Repeated calls to `__next__()` return successive items. Raises `StopIteration` when exhausted. Must also implement `__iter__()` that returns itself.

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `Iterator<T>` / objects with `[Symbol.iterator]`

---

## L — N

### lambda

An anonymous inline function consisting of a single expression, evaluated when called.

```python
add = lambda x, y: x + y
result = add(3, 5)  # 8
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** Arrow functions `const add = (x, y) => x + y;` — identical!

### list

A built-in Python sequence. Despite the name, it's more like an array in other languages than a linked list. Element access is O(1).

```python
fruits = ["apple", "banana", "cherry"]
fruits.append("date")
```

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `Array<T>` / `[T]`

### list comprehension

A compact way to process elements and return a list:

```python
squares = [n ** 2 for n in range(10)]       # [0, 1, 4, ..., 81]
evens    = [n for n in range(20) if n % 2 == 0]
```

> **Tutorial link:** Chapter 5 — Data Structures  
> **TS equivalent:** `array.map()` + `array.filter()`

### lock

A synchronization primitive that allows only one thread at a time to access a shared resource. Python's `threading` module provides `Lock` and `RLock`.

> **Tutorial link:** Chapter 12 — Async/Await Patterns  
> **TS equivalent:** N/A (JavaScript has no native locks)

### module

An object that serves as an organizational unit of Python code. Modules have a namespace containing arbitrary Python objects. Loaded via the import system.

```python
import math
result = math.sqrt(16)  # 4.0
```

> **Tutorial link:** Chapter 6 — Modules  
> **TS equivalent:** `module` / ES module `import`

### mutable

An object with state that is allowed to change during the program's execution. Lists, dictionaries, and sets are mutable.

> **VS immutable:** Immutable objects (numbers, strings, tuples) cannot be altered after creation.

> **Tutorial link:** Chapter 5 — Data Structures  
> **TS equivalent:** `let` (mutable) vs `const` on primitives / `readonly` types

### nested scope

The ability to refer to a variable in an enclosing definition. A function defined inside another function can refer to variables in the outer function.

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor  # 'factor' from outer scope
    return multiply

double = make_multiplier(2)
print(double(5))  # 10
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** Closures — same concept!

### object

Any data with state (attributes or value) and defined behavior (methods). Also the ultimate base class of any new-style class. "Everything is an object" in Python.

> **Tutorial link:** Chapter 1 — Whetting Your Appetite  
> **TS equivalent:** `object` / `any`

### parameter

A named entity in a function definition that specifies an argument the function can accept. Five kinds:

| Kind | Syntax | Example |
|------|--------|---------|
| Positional-or-Keyword | default | `def f(a, b=1)` |
| Positional-only | after `/` | `def f(a, /, b)` |
| Keyword-only | after `*` | `def f(*, a)` |
| Var-positional | `*args` | `def f(*args)` |
| Var-keyword | `**kwargs` | `def f(**kwargs)` |

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** Function parameters (fewer kinds)

### package

A Python module which can contain submodules or recursively, subpackages. Technically, a module with a `__path__` attribute.

> **Tutorial link:** Chapter 6 — Modules  
> **TS equivalent:** npm package / ES module directory

### class variable

A variable defined in a class and intended to be modified only at class level (not in an instance).

```python
class Dog:
    species = "Canis familiaris"  # class variable
    
    def __init__(self, name):
        self.name = name  # instance variable
```

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** `static` property

---

## P — S

### PEP

Python Enhancement Proposal. A design document providing information to the Python community or describing a new feature. Key PEPs for this course:

| PEP | Title | Relevance |
|-----|-------|-----------|
| **484** | Type Hints | `typing` module |
| **526** | Annotated Assignment | Variable annotations |
| **492** | Coroutines | `async def` / `await` |
| **498** | f-strings | Formatted string literals |
| **343** | Context Managers | `with` statement |
| **585** | Type Hinting Generics | Built-in generic types |
| **703** | Free Threading | PEP 703 — disable GIL |

> **Tutorial link:** Throughout  
> **Reference:** https://peps.python.org/

### pickle module

Standard library module for serializing and deserializing Python object structures. "Pickling" converts a Python object hierarchy into a byte stream; "unpickling" converts it back.

> **Tutorial link:** Chapter 11 — Brief tour of the standard library

### Py3k (Python 3000)

Nickname for the Python 3.x release line, coined long ago when the release of version 3 was something distant in the future.

> **Tutorial link:** Throughout

### Pythonic

An idea or piece of code which closely follows the most common idioms of the Python language, rather than implementing code using concepts common to other languages.

```python
# Pythonic ✅
for item in food:
    print(item)

# Not Pythonic ❌
for i in range(len(food)):
    print(food[i])
```

> **Tutorial link:** Throughout  
> **TS equivalent:** "idiomatic TypeScript"

### qualified name

A dotted name showing the "path" from a module's global scope to a class, function, or method. Accessible via `__qualname__`.

```python
class C:
    class D:
        def meth(self):
            pass

print(C.__qualname__)      # 'C'
print(C.D.__qualname__)    # 'C.D'
print(C.D.meth.__qualname__)  # 'C.D.meth'
```

> **Tutorial link:** Chapter 9 — Classes

### REPL (Read-Eval-Print Loop)

Another name for the interactive interpreter shell. Type statements at `>>>` and see results immediately.

> **Tutorial link:** Chapter 2 — Using the Python Interpreter  
> **TS equivalent:** Node.js REPL or `node -e`

### slice

An object of type `slice`, used to describe a portion of a sequence, created by slicing syntax (`[start:stop:step]`).

```python
seq = [0, 1, 2, 3, 4, 5]
seq[1:4]     # [1, 2, 3]
seq[::2]     # [0, 2, 4] — every other element
seq[::-1]    # [5, 4, 3, 2, 1, 0] — reverse
```

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `Array.slice()` / `String.slice()`

### special method

A method called implicitly by Python to execute an operation on a type. Also known as "dunder methods" (double-underscore). Examples:

| Method | Triggered By |
|--------|-------------|
| `__init__` | Object creation |
| `__str__` | `str(obj)` / `print()` |
| `__repr__` | `repr(obj)` |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[key]` |
| `__call__` | `obj()` |
| `__enter__` / `__exit__` | `with` statement |

> **Tutorial link:** Chapter 9 — Classes  
> **TS equivalent:** Symbol-based operators (`Symbol.iterator`, etc.) or decorators

### statement

A part of a suite (a "block" of code). Either an expression or one of several constructs with a keyword like `if`, `while`, or `for`. Unlike expressions, statements do not produce a value.

> **Tutorial link:** Throughout  
> **TS equivalent:** `statement` — same concept

### standard library (stdlib)

The collection of packages, modules, and extension modules distributed as part of the official Python interpreter. Documentation at [docs.python.org/3/library/](https://docs.python.org/3/library/).

```python
import sys       # System-specific parameters
import os         # Operating system interface
import json       # JSON encoding/decoding
import math       # Mathematical functions
```

> **Tutorial link:** Chapter 10-11 — Brief tour of the standard library  
> **TS equivalent:** Node.js core modules / built-in APIs

---

## T — Z

### triple-quoted string

A string bounded by three quotation marks (`'''` or `"""`). Allows unescaped single/double quotes and multi-line content. Essential for docstrings.

```python
docstring = """
This is a multi-line
docstring with 'quotes' inside.
"""
```

> **Tutorial link:** Chapter 7 — Input and Output  
> **TS equivalent:** Template literals (backticks) with newlines

### type

The type of a Python object determines what kind of object it is. Every object has a type accessible via `__class__` attribute or `type(obj)`.

```python
type(42)         # <class 'int'>
type("hello")    # <class 'str'>
type([1, 2, 3])  # <class 'list'>
```

> **Tutorial link:** Chapter 3 — An Informal Introduction to Python  
> **TS equivalent:** `typeof` / TypeScript's type system (compile-time)

### type hint

An annotation specifying the expected type for a variable, attribute, or function parameter. Optional and not enforced at runtime; useful for static type checkers (mypy) and IDEs.

```python
def greet(name: str, times: int = 1) -> list[str]:
    return [f"Hello, {name}!" for _ in range(times)]
```

> **Tutorial link:** Throughout  
> **TS equivalent:** TypeScript's type annotations — nearly identical!

### virtual environment (venv)

A cooperatively isolated runtime environment that allows installing Python packages without interfering with other applications on the same system. Similar to `node_modules` + npm in the JavaScript ecosystem.

```bash
python -m venv .venv
source .venv/bin/activate   # Unix
.venv\Scripts\activate     # Windows
pip install requests
```

> **Tutorial link:** Chapter 12 — Virtual Environments and Packages  
> **TS equivalent:** `node_modules` / `npm install`

### walrus operator

The assignment expression operator `:=`, so-called because it looks like a walrus. Allows assigning values within expressions.

```python
if (n := len(data)) > 10:
    print(f"List is too long ({n} elements, expected <= 10)")
```

> **Tutorial link:** Chapter 4 — More Control Flow Tools  
> **TS equivalent:** No direct equivalent; use separate declaration + assignment

### Zen of Python

Listing of Python design principles and philosophies. Discovered by typing `import this` at the interactive prompt. Famous lines:

```
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Readability counts.
```

> **Tutorial link:** Chapter 1 — Whetting Your Appetite

---

## Quick Reference: TS → Python Term Mapping

| TypeScript / JS | Python | Notes |
|-----------------|--------|-------|
| Interface | Protocol / ABC | Structural vs nominal typing |
| Type alias | `type` / `TypeAlias` | `type X = Y` (3.12+) |
| Union type | `X \| Y` | Pipe syntax (3.10+) |
| Optional (`T\|null`) | `X \| None` | Same concept |
| Generic `<T>` | `TypeVar("T")` + `Generic[T]` | Nearly identical |
| Enum | `enum.Enum` | More variants in Python |
| Array | `list[T]` / `tuple[T, ...]` | Homogeneous vs heterogeneous |
| Record/Map | `dict[K, V]` | Same hash map concept |
| Class decorator | `@dataclass`, etc. | Very similar syntax |
| Closures | Nested functions + `nonlocal` | Same lexical scoping |
| Arrow function | `lambda` | Single expression only |
| Module | `import module` | ES modules → Python modules |
| node_modules | venv / pip | Virtual environment isolation |
| Promise | `async def` / `await` | Coroutines vs Promises |
| `typeof` | `type(obj)` | Runtime type inspection |
| `instanceof` | `isinstance(obj, Class)` | Type checking at runtime |
| Template literal | f-string (`f"..."`) | Nearly identical features |

---

> **Source:** Official [Python 3.14 Glossary](https://docs.python.org/3/glossary.html) — adapted with TypeScript equivalents for this course's audience.
