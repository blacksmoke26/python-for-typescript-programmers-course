# Module 16 — Node.js Built-in Modules vs Python Equivalents (Comprehensive: Every Core Module, Every Popular Package, Complete Migration Guide)

## Table of Contents

- [1. Node.js Core Modules Exhaustive Reference (Every Module Mapped to Python)](#1-nodejs-core-modules-exhaustive-reference-every-module-mapped-to-python)
- [2. File System Operations: fs vs pathlib/os/aiofiles (Complete Deep Dive)](#2-file-system-operations-fs-vs-pathlibosaiofiles-complete-deep-dive)
- [3. HTTP/Networking: http/https vs requests/httpx/aiohttp (Complete Deep Dive)](#3-httpnetworking-httphttps-vs-requestsaiohttpaiohttp-complete-deep-dive)
- [4. Process & OS: process vs os/sys/subprocess/platform (Complete Deep Dive)](#4-process--os-process-vs-ossys-subprocessplatform-complete-deep-dive)
- [5. Streams: Node.js Stream API vs Python Iterators/Generators/asyncio Streams (Complete Deep Dive)](#5-streams-nodejs-stream-api-vs-python-iteratorsgeneratorsasyncio-streams-complete-deep-dive)
- [6. Events & Pub/Sub: EventEmitter → asyncio.Queue/Blinker/asyncio.create_task (Complete Deep Dive)](#6-events--pubsub-eventemitter-asyncioqueueblinkerasynciocreate_task-complete-deep-dive)
- [7. Buffer & Binary Data: Buffer → bytes/bytearray/mmap/array (Complete Deep Dive)](#7-buffer--binary-data-buffer-bytesbytearraymmmaparray-complete-deep-dive)
- [8. Child Process: child_process → subprocess Complete Reference (Every Method Mapped)](#8-child-process-child_process-subprocess-complete-reference-every-method-mapped)
- [9. DNS Module Complete Reference (Node.js dns → Python dnspython/asyncio.getaddrinfo)](#9-dns-module-complete-reference-nodejs-dns--python-dnspythonasynciogetaaddrinfo)
- [10. dgram (UDP) Socket Programming: Node.js dgram → Python socket (Complete Deep Dive)](#10-dgram-udp-socket-programming-nodejs-dgram--python-socket-complete-deep-dive)
- [11. URL Parsing Exhaustive: Node.js URL/URLSearchParams → urllib.parse/yarl (Complete Deep Dive)](#11-url-parsing-exhaustive-nodejs-urlurlsearchparams-urllibparseyar-complete-deep-dive)
- [12. querystring Module → URLSearchParams Mapping (Complete Reference)](#12-querystring-module--urlsearchparams-mapping-complete-reference)
- [13. zlib Compression/Decompression: Node.js zlib → Python zlib/bzip2/gzip/zstandard (Complete Deep Dive)](#13-zlib-compressiondecompression-nodejs-zlib--python-zlibbzip2gzipzstandard-complete-deep-dive)
- [14. assert Module → pytest Comparison (Exhaustive)](#14-assert-module--pytest-comparison-exhaustive)
- [15. diagnostics_channel Module → Python Logging Comparison](#15-diagnosticschannel-module--python-logging-comparison)
- [16. timers Module: setTimeout/setInterval → asyncio.sleep/time.sleep Complete Reference](#16-timers-module-settimeoutsetinterval-asynciosleeptime-sleep-complete-reference)
- [17. util Module: promisify/inspect/deepEqual → Python functools/inspect (Complete Deep Dive)](#17-util-module-promisifyinspectdeepequal-python-functoolsinspect-deep-dive)
- [18. module Resolution: Node.js require → Python import Complete Reference](#18-module-resolution-nodejs-require--python-import-complete-reference)
- [19. constants Module → os/constants Comparison](#19-constants-module--osconstants-comparison)
- [20. Popular npm Packages → Python Libraries (Every Package Mapped — 50+ Pairs)](#20-popular-npm-packages--python-libraries-every-package-mapped-50-pairs)
- [21. Migration Patterns: Express → FastAPI Complete Guide](#21-migration-patterns-express--fastapi-complete-guide)
- [22. lodash Utilities → Python functools/operator/itertools Mapping](#22-lodash-utilities-python-functoolsoperatoritertools-mapping)
- [23. Zod Validation → Pydantic v2 Mapping (Complete Deep Dive)](#23-zod-validation--pydantic-v2-mapping-complete-deep-dive)
- [24. Package Ecosystems: npm Registry → PyPI Complete Comparison](#24-package-ecosystems-npm-registry--pypi-complete-comparison)
- [25. Performance: Node.js Event Loop vs asyncio Event Loop (Benchmarks + Deep Analysis)](#25-performance-nodejs-event-loop-vs-asyncio-event-loop-benchmarks--deep-analysis)
- [26. Complete Migration Cheatsheet](#26-complete-migration-cheatsheet)
- [27. Quizzes with Answers (25+)](#27-quizzes-with-answers-25)
- [28. Exercises with Solutions (15+)](#28-exercises-with-solutions-15)

---

## 1. Node.js Core Modules Exhaustive Reference (Every Module Mapped to Python)

### TypeScript/Node.js Built-in Modules vs Python's Standard Library — Every Single Module Mapped

| Node.js Module | Purpose | Python Equivalent | Import Statement | Key Differences for TS Devs |
|---------------|---------|-----------------|------------------|----------------------------|
| **`path`** | Path manipulation (join, resolve, extname, dirname, basename, parse, format) | **`pathlib.Path`** — object-oriented with `/` operator + properties like `.name`, `.suffix`, `.stem`, `.parent`, `.parts` | `from pathlib import Path` | Python's pathlib is SUPERIOR — no function calls needed. `Path("a") / "b" / "c"` instead of `path.join("a", "b", "c")`. Access extension via `.suffix` not `extname()` |
| **`fs`** | File system (read, write, stat, readdir, mkdir, rm, watch, createReadStream, createWriteStream) | **`pathlib.Path`** + **`open()`** + **`os`** + **`aiofiles`** for async | `from pathlib import Path; import os; import aiofiles` | Python's `Path.read_text()` is ONE call vs Node.js's TWO calls (fs.readFile + toString). No callback hell in Python sync code! |
| **`http`** | HTTP server and client (low-level) | **No equivalent in stdlib!** Use **`requests`** (sync), **`httpx`** or **`aiohttp`** (async) | `import requests` / `import httpx` | Biggest gap: Node.js has HTTP built-in; Python needs a third-party package. Like how TS needs axios for clean HTTP APIs! |
| **`https`** | HTTPS server and client (low-level) | Same as `http`: **`requests`**, **`httpx`**, **`aiohttp`** | Same imports | No separate https module needed — the HTTP libraries handle TLS automatically |
| **`url`** / **`URL`** class | URL parsing, construction, and manipulation | **`urllib.parse`** (stdlib) or **`yarl.URL`** (third-party, superior API!) | `from urllib.parse import urlparse, urljoin, urlencode, parse_qs` | Python's stdlib is verbose: `urlparse(url).scheme` vs Node.js's `new URL(url).protocol`. Use `yarl` for a cleaner object-oriented API! |
| **`crypto`** | Hashing (SHA, MD5), encryption (AES, RSA), random data, PBKDF2 | **`hashlib`**, **`secrets`**, **`cryptography`** (third-party) | `import hashlib; import secrets; from cryptography.fernet import Fernet` | Python has built-in SHA-256 via `hashlib.sha256(data).hexdigest()`. For advanced crypto, use the `cryptography` package (like @types/crypto in TS) |
| **`os`** | OS-level operations (environ, path separators, file stats, permissions, temp dirs) | **`os`**, **`sys`**, **`platform`**, **`tempfile`** | `import os; import sys; import platform` | Node.js's process.env → Python's `os.environ`. os.getcwd() vs process.cwd(). Very similar conceptually! |
| **`process`** | Process info (PID, version, memory usage, exit codes), signal handling | **`os.getpid()`**, **`sys.version_info`**, **`sys.exit()`**, **`signal.signal()`** | `import os; import sys; import signal` | process.memoryUsage() → `psutil.Process().memory_info()` (third-party!). Both provide similar process introspection! |
| **`events`** | Event emitter / pub-sub pattern (`on`, `emit`, `once`, `removeListener`) | No direct equivalent! Use **`asyncio.Queue`**, **`blinker`** (third-party), or custom observer pattern | `import asyncio; from blinker import signal` | This is a gap — no built-in EventEmitter in Python. Use asyncio's event loop for async pub-sub, or blinker for sync pub-sub! |
| **`stream`** | Streaming data (Readable, Writable, Duplex, Transform streams) | **Iterators**, **generators (`yield`)**, **file objects**, **`io.StringIO`/`io.BytesIO`** | No import needed — generators are built-in! | Python's `for line in file:` IS streaming. No need for separate Stream module — it's built into the language via iterators! |
| **`util`** | Utility functions (promisify, inspect, deepEqual, inherits) | **`functools`**, **`inspect`**, **`operator`** | `import functools; import inspect` | Node.js promisify → functools.partial or direct async. util.inspect → pprint.pprint(). No deepEqual needed — Python's == does structural comparison for dicts! |
| **`buffer`** | Binary data handling (Buffer class, slicing, concatenation) | **`bytes`**, **`bytearray`**, **`struct`**, **`array.array`**, **`mmap`** | No import needed for bytes/bytearray; `import struct; import mmap` | Python has native `bytes` and `bytearray` types — no separate Buffer class needed! More integrated with the language. |
| **`child_process`** | Spawn child processes (exec, execFile, spawn, spawnSync, fork) | **`subprocess.run()`**, **`subprocess.Popen()`**, **`subprocess.call()`**, **`subprocess.check_output()`** | `import subprocess` | Nearly identical API! subprocess.run([...], capture_output=True, text=True) is like child_process.exec(). Both have sync and async variants! |
| **`timers`** / **`setTimeout`** | Delayed execution (setTimeout, setInterval, clearTimeout, clearInterval) | **`asyncio.sleep()`**, **`time.sleep()`**, **`threading.Timer()`** | `import asyncio; import time; import threading` | Python has both blocking (time.sleep) and non-blocking (await asyncio.sleep) timers. Like Node.js having setTimeout but with explicit sync/async choice! |
| **`console`** | Console logging (log, warn, error, info, debug, table) | **`print()`**, **`logging`** module | Built-in for print; `import logging` for structured logging | Python's logging module is FAR more robust than console.log — supports levels, formatters, handlers, file output. Like having Winston built into the stdlib! |
| **`assert`** | Assertion testing (strict/non-strict modes) | **Built-in `assert` statement**, **`unittest`**, **pytest.raises** | Built-in | Python's assert is a language keyword (like TypeScript's ==). For production assertions, use pytest.raises() for exception checking! |
| **`constants`** / **`os.constants`** | OS-specific constants (SIGINT, SIGTERM, errno values) | **`signal.SIGINT`**, **`errno`**, **`os.EX_OK`** | `import signal; import errno; import os` | Python has these spread across different modules. Node.js has them in a single constants module! |
| **`diagnostics_channel`** | Runtime diagnostics and observability (publish/subscribe for debugging) | **`logging`**, **`logging.handlers`**, **`structlog`** | `import logging; import logging.handlers` | No direct equivalent. Node.js's diagnostics_channel is for deep runtime instrumentation. Python's logging module covers most use cases! |
| **`module`** | Module resolution and loading (require, exports, module) | **`import` statement**, **`importlib`**, **`sys.modules`** | `import importlib; import sys` | Node.js uses require()/exports pattern. Python uses import statements. Both use a module cache (node_modules vs sys.modules)! |
| **`net`** | Network sockets (TCP client/server) | **`socket`** (stdlib), **`asyncore`/`aiohttp`** for async | `import socket; import asyncio` | Very similar API! socket.create_connection() is like net.connect(). Node.js wraps it in a higher-level Stream API; Python exposes raw sockets directly! |
| **`dns`** | DNS resolution (lookup, resolve4, resolve6, reverse) | **`socket.getaddrinfo()`**, **`dnspython`** (third-party) | `import socket; from dns import resolver` | Node.js has built-in DNS; Python requires dnspython for advanced queries. Basic lookups work via socket.getaddrinfo(). |
| **`dgram`** | UDP datagram sockets | **`socket` with `AF_PACKET`/`SOCK_DGRAM`** | `import socket` | Both use raw sockets but Node.js wraps them in a Stream-like API; Python exposes the socket directly! |
| **`querystring`** / **`URLSearchParams`** | URL query string parsing and encoding | **`urllib.parse.urlencode()`**, **`urllib.parse.parse_qs()`** | `from urllib.parse import urlencode, parse_qs` | Both handle query string manipulation. Python's stdlib is more verbose; use yarl or httpx.URL for a cleaner API! |
| **`zlib`** / **`pako`** | Compression/decompression (gzip, deflate, zlib) | **`zlib`**, **`gzip`**, **`bz2`**, **`lzma`**, **`zstandard`** | `import zlib; import gzip; import bz2` | Python has ALL compression formats built-in! No separate pako package needed. zstandard for modern compression (like lz4 in Node.js)! |
| **`readline`** | Interactive command-line input | **`input()`**, **`readline`** module, **`prompt_toolkit`** | `import readline; from prompt_toolkit import prompt` | Similar concept! Python's input() is like readline's basic usage. For advanced CLI (auto-complete, history), use prompt_toolkit! |
| **`tty`** / **`termios`** | Terminal control (raw mode, terminal settings) | **`termios`**, **`tty`**, **`curses`** | `import termios; import tty; import curses` | Python has built-in termios for terminal control. Like Node.js's tty module but with more direct access to the terminal! |
| **`readline`** / **`repl`** | Interactive JavaScript REPL | **Python REPL** (built-in!), **`bpython`**, **`ipython`** | Built-in | Python's REPL is actually MORE powerful than Node.js's by default — supports tab completion, syntax highlighting! ipython adds notebook support! |
| **`worker_threads`** | Multi-threading / worker pools | **`concurrent.futures.ThreadPoolExecutor`**, **`multiprocessing.Process`**, **`asyncio.to_thread()`** | `from concurrent.futures import ThreadPoolExecutor; import multiprocessing` | Node.js uses true parallel workers; Python's GIL limits threading for CPU-bound work. Use multiprocessing for true parallelism! |
| **`cluster`** | Process clustering (load balancing across multiple processes) | **`multiprocessing`**, **`gunicorn`** (web server level!) | `import multiprocessing` / `pip install gunicorn` | Node.js cluster module creates worker processes that share a port. Python uses gunicorn/uwsgi for the same thing at the web server level! |
| **`v8`** / **`vm2`** | V8 engine introspection / sandboxed code execution | **`ast.literal_eval()`**, **`exec()`/`eval()`** (with restrictions), **` RestrictedPython`** | `import ast; from restrictedpython import compile_restricted` | Node.js uses V8 APIs for introspection. Python has ast module for code analysis and RestrictedPython for sandboxing! |

### Mermaid: Complete Node.js → Python Module Mapping Architecture

```mermaid
flowchart TD
    subgraph NODE_CORE["Node.js Core Modules (built into Node.js!)"]
        N1["fs — file system read/write/watch/streams"] --> N2["path — path manipulation functions"]
        N3["http / https — HTTP server/client"]
        N4["url — URL parsing class"]
        N5["crypto — SHA/AES/random"]
        N6["os / process — OS & process info"]
        N7["events — EventEmitter pub/sub"]
        N8["stream — Readable/Writable/Duplex/Transform"]
        N9["child_process — spawn/exec/fork"]
        N10["timers — setTimeout/setInterval"]
        N11["console — logging methods"]
        N12["dns / dgram — DNS resolution + UDP"]
        N13["zlib — compression/decompression"]
    end

    subgraph PYTHON_STD["Python Standard Library Equivalents"]
        P1["pathlib.Path / os — file ops + paths (builtin!)\npathlib is SUPERIOR: Path(a) / b vs path.join(a,b)!"]
        P2["requests/httpx/aiohttp — HTTP client\n→ NOT built in! Must pip install! Biggest gap from Node.js!"]
        P3["urllib.parse — URL parsing (stdlib but verbose)\nyarl for modern API"]
        P4["hashlib / secrets / cryptography\n→ ALL built into stdlib unlike Node.js!\nPython covers crypto more comprehensively in stdlib!"] --> P5["os / sys / platform\n→ Similar to Node's os/process but split across modules"]
        P6["asyncio.Queue / blinker\n→ No built-in EventEmitter! Use async patterns instead."]
        P7["yield generators / file objects\n→ Streaming IS the language! No separate Stream module needed!"] --> P8["subprocess.run() / spawn()\n→ Nearly identical to child_process in Node.js!"]
        P9["asyncio.sleep() / time.sleep()\n→ Both sync and async timer options available"]
        P10["logging module\n→ FAR more powerful than console.log!\nSupports levels, formatters, handlers, file output — like Winston built-in!"] --> P11["dnspython + socket\n→ DNS queries need third-party package; UDP uses raw sockets"]
        P12["zlib / gzip / bz2 / lzma / zstandard\n→ All compression formats built into stdlib! No external packages needed for basic compression!"]
    end

    N1 -.->|"Both provide file access:\ntypeScript uses fs module with callbacks/promises\nPython uses pathlib.Path for sync + aiofiles for async!"| P2
    N6 -.->|"Both provide process info:\ntypeScript uses process.env/pid/version\nPython uses os.environ/os.getpid()/sys.version!"| P5
```

### Key Notes: Core Module Mapping

1. **Node.js has MORE built-in modules than Python's standard library** — particularly networking (http, https) and event handling (events). Python requires third-party packages for HTTP (requests/httpx), but Python covers areas Node.js lacks natively: hashlib (crypto), secrets (secure randomness), uuid, csv, json, decimal.
2. **Python's pathlib.Path is objectively superior to Node.js's path module** — object-oriented API with `/` operator for joining paths and properties like `.name`, `.suffix`, `.stem` instead of separate functions like `path.join()`, `path.extname()`. Like how TypeScript uses template literals; Python's Path `/` operator is even cleaner!
3. **Python has built-in logging (logging module) that rivals Winston/Pino** — unlike TypeScript which needs external packages for structured logging. Python's logging module supports level-based output, formatters, handlers, and file/console output automatically via just `import logging`. Like having Winston built into Python's stdlib!

---

## 2. File System Operations: fs vs pathlib/os/aiofiles (Complete Deep Dive)

### Node.js fs API vs Python pathlib — Side-by-Side Code (Every Operation Mapped)

```typescript
// ========================================
// TypeScript/Node.js: Complete fs + path operations
// ========================================
import * as fs from "fs";
import path from "path";
import { readFile, writeFile, readdir, stat, mkdir, rm, watch } from "fs/promises";

// === Read a JSON file (TWO calls in Node.js: readFile + JSON.parse!) ===
const configPath = path.join(process.cwd(), "config", "settings.json");
const rawContent = await fs.promises.readFile(configPath, "utf-8");  // Call 1: read bytes
const config = JSON.parse(rawContent);                                // Call 2: parse JSON manually

// === Write a JSON file (TWO calls: JSON.stringify + writeFile) ===
await fs.promises.writeFile(
    path.join(process.cwd(), "output", "result.json"),
    JSON.stringify(config, null, 2),                                 // stringify call!
    { encoding: "utf-8" }
);

// === Read a text file with error handling ===
try {
    const content = await fs.promises.readFile("data.txt", "utf-8");
} catch (err) {
    if ((err as NodeJS.ErrnoException).code === "ENOENT") {
        console.error("File not found!");
    }
}

// === List directory contents with full stat for each file ===
const files = await fs.promises.readdir("src", { withFileTypes: true });
for (const dirent of files) {
    const fullPath = path.join("src", dirent.name);                // Re-join needed!
    const statResult = await fs.promises.stat(fullPath);           // Second call for stats!
    console.log(
        `${dirent.name}: ${statResult.size} bytes, ` +
        `${dirent.isDirectory() ? "dir" : "file"} (modified: ${new Date(statResult.mtimeMs)})`
    );
}

// === Create nested directories (recursive) ===
await fs.promises.mkdir(path.join("output", "nested", "deep"), { recursive: true });

// === Delete directory recursively ===
await fs.promises.rm("temp/", { recursive: true, force: true });

// === Check if file exists ===
const exists = await fs.promises.access(filePath).then(() => true).catch(() => false);

// === Watch for file changes ===
const watcher = fs.watch(path.join(process.cwd(), "src"), (eventType, filename) => {
    console.log(`File changed: ${filename} (${eventType})`);
});
watcher.on("close", () => console.log("Watcher closed"));

// === Append to a file ===
await fs.promises.appendFile("logs.txt", `[${new Date()}] User logged in\n`);

// === Copy a file (requires Node 20+) ===
await fs.promises.copyFile("source.txt", "dest.txt");

// === Get current working directory ===
console.log(process.cwd());  // Built-in! No import needed!

// === Read as stream for large files ===
import { createReadStream } from "fs";
const lineCount = await new Promise((resolve) => {
    let count = 0;
    const stream = createReadStream("huge.csv", { encoding: "utf-8" });
    stream.on("data", (chunk: string) => { count += chunk.split("\n").length - 1; });
    stream.on("end", () => resolve(count));
});
```

```python
# ========================================
# Python: Complete pathlib + os operations (EVERY Node.js fs call has a Python equivalent!)
# ========================================

from pathlib import Path
import json, os, shutil, aiofiles       # Standard library for most operations!

cwd = Path.cwd()                          # Like process.cwd() — ONE call, returns Path object!

# === Read a JSON file (ONE call in Python vs TWO in Node.js!) ===
config_path = Path("config") / "settings.json"  # No path.join needed! Use / operator!
config = json.loads(config_path.read_text())   # read_text() + json.loads() = ONE expression!

# === Write a JSON file (ONE call in Python vs TWO in Node.js!) ===
(Path("output") / "result.json").write_text(  # No separate writeFile + JSON.stringify needed!
    json.dumps(config, indent=2),
    encoding="utf-8"
)

# === Read text file with error handling (try/except instead of try/catch!) ===
try:
    content = Path("data.txt").read_text(encoding="utf-8")
except FileNotFoundError:
    print("File not found!")                  # ENOENT → FileNotFoundError in Python!
except PermissionError:                        # Additional error type Node.js doesn't have!
    print("Permission denied!")

# === List directory contents (NO re-join needed — Path objects have all the info!) ===
for file_path in Path("src").iterdir():            # Returns Path objects directly (no join needed)!
    stat_info = file_path.stat()                   # Get stats on the Path object itself!
    print(f"{file_path.name}: {stat_info.st_size} bytes, "
          f"{'dir' if file_path.is_dir() else 'file'} (modified: {stat_info.st_mtime})")

# === Create nested directories (recursive — Python makes this trivial!) ===
(Path("output") / "nested" / "deep").mkdir(parents=True, exist_ok=True)  # Like mkdir -p in shell!

# === Delete directory recursively ===
shutil.rmtree("temp", ignore_errors=True)          # Like fs.rm with recursive + force!

# === Check if file exists ===
if Path("config.json").exists():                   # Simple boolean check — no try/catch needed!
    print("Config found!")

# === File existence with different checks ===
Path("file.txt").is_file()                         # Is it a regular file?
Path("dir").is_dir()                               # Is it a directory?
Path("link").is_symlink()                          # Is it a symbolic link?

# === Watch for file changes (requires watchdog package — similar to Node.js fs.watch) ===
import watchdog.events
from watchdog.observers import Observer

class ChangeHandler(watchdog.events.FileSystemEventHandler):
    def on_modified(self, event):
        if not event.is_directory:
            print(f"File modified: {event.src_path}")
    def on_created(self, event):
        print(f"File created: {event.src_path}")

observer = Observer()
observer.schedule(ChangeHandler(), "src", recursive=True)
observer.start()                                   # Runs until Ctrl+C!

# === Append to a file (Python has a built-in mode flag!) ===
with open("logs.txt", "a") as f:                  # 'a' mode = append!
    f.write(f"[{__import__('datetime').datetime.now()}] User logged in\n")

# === Copy/move directories (shutil does it all!) ===
shutil.copy2("source.txt", "dest.txt")            # Copy with metadata preservation!
shutil.move("old_dir/", "new_location/")          # Move/rename directory!

# === Get file extension, stem, parent (pathlib superpowers over Node.js path!) ===
file = Path("document.pdf")
print(file.suffix)           # ".pdf"      — Like path.extname()
print(file.stem)             # "document"   — No direct Node.js equivalent!
print(file.parent)           # Path(".")    — Like path.dirname()
print(file.name)             # "document.pdf" — Like path.basename()
parts = file.parts            # ('.', 'document.pdf') — Like splitting path manually

# === Glob pattern matching (like Node.js glob/fill but built in!) ===
for py_file in Path("src").rglob("*.py"):        # Recursive search for all .py files!
    print(py_file)

# === Read as stream for large files (Python's for loop IS the streaming!) ===
line_count = sum(1 for _ in Path("huge.csv").read_text().splitlines())  # Memory-heavy but concise!
# Better: iterate line-by-line without loading entire file into memory!
with Path("huge.csv").open() as f:                # context manager = automatic close!
    count = 0
    for line in f:                               # Streams data line-by-line! (like Node's stream!)
        count += 1
# No callback setup needed — the for loop handles streaming natively!

# === Async file operations (aiofiles package — like fs.promises!) ===
import aiofiles

async def read_config_async():
    async with aiofiles.open("config/settings.json") as f:  # Like fs.promises.readFile!
        content = await f.read()
    return json.loads(content)
```

### Mermaid: File System Operations Flow Comparison

```mermaid
flowchart TD
    subgraph NODE_FS["Node.js fs Module — Callback/Promise-based API"]
        F1["fs.promises.readFile(path, 'utf-8')\n→ Returns Promise<string>!"] --> F2["JSON.parse(content) → Manual parsing step!\nTWO separate operations needed!"]
        F3["path.join(dir1, dir2, file)\n→ Multiple function calls for path joining!"] --> F4["fs.promises.mkdir(path, {recursive: true})\n→ Separate call for directory creation!"]
    end

    subgraph PY_PATHLIB["Python pathlib.Path — Object-Oriented Unified API"]
        P1["Path('config') / 'settings.json'\n→ ONE expression creates the full path!\nNo join function calls needed! Use / operator!"] --> P2["json.loads(Path.read_text())\n→ read_text() reads + returns string in ONE call!\nOne line: config = json.loads(Path.read_text())"]
        P3["Path('output') / 'nested' / 'deep'\n.mkdir(parents=True, exist_ok=True)\n→ mkdir with parents creates all intermediate dirs!"] --> P4["for line in Path.open():\n→ Iterates line-by-line automatically!\nNo streaming setup needed — for loop handles it natively!"]
    end

    F1 -.->|"Both read files:\ntypeScript uses fs.promises.readFile + JSON.parse\nPython uses Path.read_text() + json.loads() in ONE expression!"| P2
    F3 -.->|"Both handle paths:\ntypeScript uses path.join (multiple calls)\nPython uses Path / operator (unified object!)"| P3
```

### Key Notes: File System Operations

1. **Python's pathlib is superior to Node.js's fs/path modules** — the `/` operator joins paths instead of `path.join()`, and `.name`, `.suffix`, `.stem`, `.parent` are properties, not function calls. Like how TypeScript has template literals for string building; Python has the `/` operator for path building.
2. **Python's `for line in file:` IS streaming** — unlike Node.js where you need to set up event listeners (`stream.on('data')`, `stream.on('end')`) and handle chunks manually, Python's for loop handles everything automatically. The file object is an iterator that yields one line at a time!
3. **Python has both sync (blocking) and async file I/O** — use `Path.read_text()` for synchronous (like Node.js's `fs.readFileSync`), or `aiofiles` for asynchronous (like Node.js's `fs.promises.readFile`). Python defaults to sync; you must explicitly choose async via the aiofiles package.

---

## 3. HTTP/Networking: http/https vs requests/httpx/aiohttp (Complete Deep Dive)

### Every HTTP Operation Mapped: Node.js http → Python requests/httpx/aiohttp

```typescript
// ========================================
// TypeScript/Node.js: Complete HTTP operations with built-in http module
// ========================================
import * as http from "http";
import https from "https";
import { URL } from "url";

// === GET request with the built-in http module (LOW LEVEL — 8+ lines!) ===
const apiUrl = new URL("https://api.example.com/users/42");

const getReq = https.request(apiUrl, {
    method: "GET",
    headers: {
        "Authorization": `Bearer ${process.env.API_KEY}`,
        "Accept": "application/json"
    }
}, (response) => {
    let data = "";
    response.on("data", (chunk) => { data += chunk; });    // Chunk-by-chunk handling!
    response.on("end", () => {
        const user = JSON.parse(data);                       // Parse manually!
        console.log(user.name);                              // Access parsed data!
    });
});
getReq.on("error", (err) => { console.error(err); });
getReq.end();

// === POST with request body (LOW LEVEL — 10+ lines!) ===
const postUrl = new URL("https://api.example.com/users");
const userData = JSON.stringify({ name: "Alice", email: "alice@example.com" });

const postReq = https.request(postUrl, {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Content-Length": Buffer.byteLength(userData)       // Calculate length manually!
    }
}, (response) => {
    let body = "";
    response.on("data", chunk => { body += chunk; });
    response.on("end", () => console.log(`Created user: ${body}`));
});
postReq.write(userData);              // Write body explicitly!
postReq.end();

// === PUT/DELETE requests ===
const putReq = https.request(new URL("https://api.example.com/users/42"), {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
}, (res) => { /* ... */ });
putReq.write(JSON.stringify({ name: "Alice Updated" }));
putReq.end();

// === HTTP with query parameters ===
const searchUrl = new URL("https://api.example.com/search");
searchUrl.searchParams.set("q", "python");
searchUrl.searchParams.set("page", "2");
searchUrl.searchParams.set("limit", "10");
// https.get(searchUrl, ...) → makes GET with query string!

// === HTTP with timeouts (requires manual setup!) ===
const timeoutReq = https.request(url, { timeout: 5000 }, (res) => { /* ... */ });
timeoutReq.on("timeout", () => { timeoutReq.destroy(); });
```

```python
# ========================================
# Python: Complete HTTP operations — requests (sync), httpx (async), aiohttp (async)!
# Every Node.js http operation becomes ONE line of code!
# ========================================

import requests          # pip install requests  → Sync client (like axios!)
import httpx           # pip install httpx       → Async client (like node-fetch with async!)
# import aiohttp        → Alternative async client for asyncio event loop

# === GET request (ONE call vs 8+ lines in Node.js!) ===
response = requests.get(
    "https://api.example.com/users/42",
    headers={
        "Authorization": f"Bearer {os.environ['API_KEY']}",
        "Accept": "application/json"
    },
    timeout=10              # Timeout in seconds! (Node.js requires manual setup!)
)
response.raise_for_status()           # Like Node.js: check response.statusCode == 200 manually!
user = response.json()                # Automatic JSON parsing! No JSON.parse() needed!
print(user["name"])

# === POST with JSON body (ONE call vs 10+ lines in Node.js!) ===
response = requests.post(
    "https://api.example.com/users",
    json={"name": "Alice", "email": "alice@example.com"},  # Automatically serializes to JSON + sets Content-Length!
    headers={"Authorization": f"Bearer {os.environ['API_KEY']}"}
)
print(f"Created user: {response.json()['id']}")

# === PUT/DELETE/PATCH (identical pattern!) ===
requests.put("https://api.example.com/users/42", json={"name": "Alice Updated"})  # ONE line!
requests.delete("https://api.example.com/users/42")                                   # ONE line!
requests.patch("https://api.example.com/users/42", json={"email": "new@example.com"}) # ONE line!

# === HTTP with query parameters (clean URL building!) ===
response = requests.get(
    "https://api.example.com/search",
    params={"q": "python", "page": 2, "limit": 10}   # Parameters handled automatically! No URL construction needed!
)

# === HTTP with authentication ===
response = requests.get(
    "https://api.example.com/secure",
    auth=("username", "password")       # Basic auth — built in! One line!
)
# Or: response = requests.get(url, headers={"Authorization": f"Bearer {token}"})  # Bearer auth!

# === Async GET with httpx (like Node.js's native async HTTP!) ===
async def get_user_async(user_id: int) -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:   # Async connection pool managed automatically!
        response = await client.get(f"https://api.example.com/users/{user_id}")
        response.raise_for_status()
        return response.json()

# === Async POST with httpx ===
async def create_user_async(name: str) -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:
        response = await client.post(
            "https://api.example.com/users",
            json={"name": name}                              # Like requests.json= but async!
        )
        return response.json()

# === Multipart file upload (one-liner!) ===
with open("photo.jpg", "rb") as f:
    response = requests.post(
        "https://api.example.com/upload",
        files={"file": f}                                  # File upload handled automatically!
    )

# === Session with cookie persistence (like axios interceptors!) ===
session = requests.Session()
session.headers.update({"Authorization": f"Bearer {token}"})  # Default headers for all requests!
response1 = session.get("/profile")                         # Uses same session/cookies!
response2 = session.post("/update", json={"name": "Alice"}) # Same session, same cookies!

# === Streaming response (like Node.js's response.on('data') but simpler!) ===
with requests.get("https://example.com/large-file.zip", stream=True) as response:
    with open("downloaded.zip", "wb") as f:
        for chunk in response.iter_content(chunk_size=8192):  # Like Node's data event!
            f.write(chunk)

# === HTTPX streaming (async version!) ===
async with httpx.AsyncClient() as client:
    async with client.stream("GET", "https://example.com/large-file.zip") as response:
        async for chunk in response.aiter_bytes():          # Like Node.js's stream.on('data') but async!
            await f.write(chunk)
```

### Mermaid: HTTP Request Flow Comparison (TypeScript vs Python)

```mermaid
flowchart TD
    subgraph TS_HTTP_FLOWS["TypeScript/Node.js HTTP — Low-Level http module (8-15 lines per request!)"]
        direction TB
        H1["Create URL object\nnew URL('https://...') + set search params!"] --> H2["Configure https.request options:\nmethod, headers, timeout, auth → Object config!"]
        H3["Handle response with event listeners:\nres.on('data') for chunks → res.on('end') for completion!"] --> H4["Manually JSON.parse(response_data)\n→ TWO steps: parse + access properties!\"]
    end

    subgraph PY_HTTP_FLOWS["Python HTTP — requests/httpx (ONE call does EVERYTHING!)"]
        direction TB
        S1["requests.get(url, headers={...}, params={...}, timeout=10)\n→ ALL configuration in a SINGLE function call!"] --> S2["response.raise_for_status() → Raises exception on 4xx/5xx!\nresponse.json() → Automatic JSON parsing!\nNo manual chunks, no manual parsing!"]
        S3["Async: await client.post(url, json={...})\nhttpx manages connection pooling, timeouts, retries automatically!"] --> S4["One line: response = requests.get(...)\nOne line: data = response.json()\nThat's it — done! Like having a Swiss Army knife vs individual tools!"]
    end

    H2 -.->|"Both make HTTP requests:\ntypeScript uses low-level http module with manual chunk handling\nPython uses requests (1 call) or httpx/asyncio for async!"| S1
```

### Key Notes: HTTP Networking

1. **Python's `requests` library is like TypeScript's axios** — one method call does everything: URL building, headers, authentication, timeout, JSON parsing, status checking, error handling. Unlike Node.js's built-in http module which requires 8-15 lines per request!
2. **HTTP is the biggest gap between Node.js and Python stdlib** — Node.js ships `http` and `https` as core modules; Python requires a third-party package (`pip install requests`). However, like how TypeScript developers use axios for a clean API, Python developers just `pip install requests` once and get equivalent (often better) functionality.
3. **Use `httpx` or `aiohttp` for async** — like Node.js's native async HTTP (which IS the default since everything in Node is async), Python needs explicit async packages: `httpx` (modern, clean API) or `aiohttp` (asyncio-native).

---

## 4. Process & OS: process vs os/sys/subprocess/platform (Complete Deep Dive)

### Every process/os Operation Mapped — Complete Side-by-Side Reference

```typescript
// ========================================
// TypeScript/Node.js: process module — EVERY function mapped!
// ========================================
console.log(process.cwd());                    // Current working directory
console.log(process.env.HOME);                 // Environment variable (with undefined fallback)
console.log(process.env.NODE_ENV || 'development');  // Default value with || operator

const info = process.version;                  // Node.js version string ("v20.10.0")
const pid = process.pid;                       // Process ID
const platform = process.platform;             // "linux", "darwin", "win32"
const arch = process.arch;                     // "x64", "arm64"

const mem = process.memoryUsage();             // { rss, heapTotal, heapUsed, external }
console.log(`Heap: ${(mem.heapUsed / 1024 / 1024).toFixed(2)}MB`);

console.log(process.argv);                     // Command-line arguments (array with node path + script)
console.log(process.exitCode);                 // Current exit code
process.exit(0);                               // Exit successfully
process.abort();                               // Crash and core dump

// Platform detection
const isMac = process.platform === 'darwin';
const isWindows = process.platform === 'win32';
```

```python
# ========================================
# Python: os + sys + platform — EVERY Node.js process operation mapped!
# Every single function above has an exact Python equivalent!
# ========================================

import os, sys, platform                       # Standard library imports only!

print(os.getcwd())                              # Like process.cwd()
print(os.environ.get("HOME"))                   # Like process.env.HOME (with safe .get()!)
print(os.environ.get("NODE_ENV", "development"))  # Default value with .get(default)!

python_version = platform.python_version()      # Python version string ("3.12.0")
pid = os.getpid()                               # Process ID — identical!
platform_name = platform.system()               # "Linux", "Darwin", "Windows" (like process.platform!)
architecture = platform.machine()               # "x86_64", "arm64" (like process.arch!)

# Memory usage (requires psutil third-party for detailed info!)
import psutil                                    # pip install psutil
memory = psutil.Process(os.getpid()).memory_info()  # rss, vms, etc. — like process.memoryUsage()!
print(f"RSS: {memory.rss / 1024 / 1024:.2f}MB")

command_args = sys.argv                         # Command-line arguments (like process.argv!)
# [script_path, arg1, arg2, ...] — same structure as Node.js!

exit_code = os.EX_OK                            # Exit code constant (like 0 in Node.js!)
sys.exit(0)                                     # Exit successfully — EXACTLY like process.exit(0)!

# Platform detection (same logic as TypeScript!)
is_mac = platform.system() == "Darwin"
is_windows = platform.system() == "Windows"

# Signal handling (like process.on('SIGINT', ...))
import signal

def handle_sigint(signum, frame):               # Like Node.js's process.on('SIGINT', cb)
    print("Received SIGINT — cleaning up...")
    sys.exit(0)

signal.signal(signal.SIGINT, handle_sigint)     # Register signal handler!
signal.signal(signal.SIGTERM, handle_sigint)    # Also handle SIGTERM (graceful shutdown)!
```

### Every subprocess Method Mapped — Complete Reference

```typescript
// ========================================
// TypeScript/Node.js: child_process module — every method!
// ========================================
import { exec, execFile, spawn, fork, spawnSync, execSync } from "child_process";

// === exec: Run command as shell string (async) ===
exec('ls -la && echo "Done"', (error, stdout, stderr) => {
    if (error) throw error;
    console.log(stdout);
});

// === execFile: Run binary directly with args (async, safer!) ===
execFile('ls', ['-la'], (error, stdout, stderr) => {
    console.log(stdout);
});

// === spawn: Stream-based process execution (for continuous output!) ===
const child = spawn('tail', ['-f', '/var/log/syslog']);
child.stdout.on('data', (data) => console.log(data.toString()));
child.stderr.on('data', (data) => console.error(data.toString()));
child.on('close', (code) => console.log(`Exit: ${code}`));

// === execSync: Synchronous version (blocks!) ===
const output = execSync('git log --oneline -5').toString();

// === fork: Spawn new Node.js process with IPC channel! ===
const worker = fork('./worker.js');
worker.send({ type: 'start', data: 'hello' });
worker.on('message', (msg) => console.log(msg));
```

```python
# ========================================
# Python: subprocess module — EVERY child_process method mapped!
# Each Node.js method has an exact Python equivalent!
# ========================================

import subprocess, sys

# === subprocess.run (like exec but more structured!) ===
result = subprocess.run(                    # Like child_process.exec() but returns a RESULT OBJECT!
    ["ls", "-la"],                           # Command as list of arguments (vs string in Node.js!)
    capture_output=True,                      # Capture both stdout and stderr!
    text=True,                                # Return strings (not bytes)!
    timeout=30,                               # Like setTimeout on the child process!
    check=False                               # Don't raise exception on non-zero exit code
)
print(f"stdout:\n{result.stdout}")            # Access by attribute — no callback needed!
print(f"stderr:\n{result.stderr}")
if result.returncode != 0:
    print(f"Command failed with code {result.returncode}")

# === subprocess.run synchronous (like execSync!) ===
output = subprocess.check_output(           # Like execSync + returns stdout directly!
    ["git", "log", "--oneline", "-5"],
    text=True,
    timeout=10
)

# === subprocess.Popen (like spawn — stream-based for continuous output!) ===
process = subprocess.Popen(                 # Like child_process.spawn()!
    ["tail", "-f", "/var/log/syslog"],
    stdout=subprocess.PIPE,                  # Stream output via .stdout!
    stderr=subprocess.PIPE,
    text=True
)

# Read line by line (like Node.js's data event!)
for line in process.stdout:               # Iterates over output lines — like stream.on('data')!
    print(f"> {line.strip()}")

process.wait()                            # Like child.on('close', cb)!
print(f"Exited with code {process.returncode}")

# === subprocess.run with input (like writing to stdin!) ===
result = subprocess.run(
    ["sort"],                             # sort command reads from stdin!
    input="banana\napple\ncherry",         # Pass input as string!
    text=True,
    capture_output=True
)
print(result.stdout)  # apple\nbanana\ncherry (sorted!)

# === Popen with bidirectional communication (like fork with IPC!) ===
proc = subprocess.Popen(
    ["python", "-c", "import sys; print(int(sys.stdin.read()) * 2)"],
    stdin=subprocess.PIPE,                # Write to stdin!
    stdout=subprocess.PIPE,               # Read from stdout!
    text=True
)
stdout, stderr = proc.communicate(input="21")  # Send input and read output!
print(f"Result: {stdout.strip()}")  # 42!
```

### Mermaid: Process & OS Operations Comparison

```mermaid
flowchart TD
    subgraph TS_PROCESS_OPS["Node.js process + child_process Module"]
        direction TB
        P1["process.env.KEY — Direct env var access (can be undefined!)"] --> P2["process.pid / process.platform / process.version\n→ All available on the single process object!\"]
        P3["child_process.exec('cmd', (err, out) => { console.log(out) })\n→ Callback-based execution with error/stdout/stderr parameters!"] --> P4["process.exit(0) — Exit with code. process.on('SIGINT', cb) for signals!"]
    end

    subgraph PY_OS_SYS_OPS["Python os + sys + subprocess Equivalents"]
        direction TB
        O1["os.environ.get('KEY', 'default') — Safe env var access with default fallback!\n→ No undefined possible — always returns a value (or the default)!"] --> O2["os.getpid() / platform.system() / sys.version\n→ Same information but spread across different modules.\nPython separates concerns by module, not by single object!"]
        O3["subprocess.run(['cmd'], capture_output=True, text=True)\n→ Returns a RESULT OBJECT with .stdout, .stderr, .returncode!\n→ No callbacks needed — structured return value like TypeScript's Promise result!\"] --> O4["sys.exit(0) — Exit with code. signal.signal(signal.SIGINT, cb) for signals!"]
    end

    P1 -.->|"Both access environment variables:\ntypeScript uses process.env.KEY\nPython uses os.environ.get('KEY', 'default') (safer — has default fallback!)!"| O1
    P3 -.->|"Both execute commands:\ntypeScript uses child_process.exec with callbacks\nPython uses subprocess.run with result object! More structured, no callbacks needed!"| O3
```

### Key Notes: Process & OS Operations

1. **Python's os.environ.get() is safer than Node.js's process.env.KEY** — you can provide a default value (`os.environ.get("KEY", "default")`) instead of checking for `undefined`. Like TypeScript's `process.env.KEY ?? 'default'` but built into the access method!
2. **subprocess.run returns a RESULT OBJECT (like async/await in Node.js)** — instead of callbacks with error/stdout/stderr parameters, Python returns an object you access by named attributes (`result.stdout`, `result.stderr`, `result.returncode`). Like TypeScript's `async` returning a Promise that resolves to a structured value!
3. **Python has both sync and async subprocess patterns** — `subprocess.run()` is synchronous (like Node.js's `child_process.execSync()`), while `subprocess.Popen()` with iteration or `asyncio.create_subprocess_exec()` provides streaming/async patterns.

---

## 5. Streams: Node.js Stream API vs Python Iterators/Generators/asyncio Streams (Complete Deep Dive)

### Every Streaming Pattern Mapped — Complete Side-by-Side Comparison

```typescript
// ========================================
// TypeScript/Node.js: Complete Stream API reference
// ========================================
import { createReadStream, createWriteStream } from "fs";
import { Transform } from "stream";
import { pipeline } from "stream/promises";

// === Readable stream (reading a large file!) ===
const readable = createReadStream("large-file.csv", { encoding: "utf-8", highWaterMark: 64 * 1024 });
let totalLines = 0;

readable.on("data", (chunk: string) => {           // Called for each chunk!
    const lines = chunk.split("\n");
    totalLines += lines.length - 1;                 // Count lines in this chunk!
});
readable.on("end", () => console.log(`Total: ${totalLines} lines`));
readable.on("error", (err) => console.error(err));

// === Transform stream (processing data as it flows!) ===
const transform = new Transform({
    transform(chunk, encoding, callback) {          // Called for each chunk!
        const upper = chunk.toString().toUpperCase();  // Transform logic!
        this.push(upper);                             // Push transformed data downstream!
        callback();                                   // Signal completion for this chunk!
    }
});

// === Pipe: Connect readable → transform → writable (like a data pipeline!) ===
createReadStream("input.txt")
    .pipe(transform)                                 // Chain transforms automatically!
    .pipe(createWriteStream("output.txt"));          // Write processed data!

// === pipeline: Async pipeline with error handling! ===
await pipeline(
    createReadStream("input.csv"),
    transform,
    createWriteStream("output.csv")
);  // Like Node.js's pipe() but with Promise-based error handling!

// === Writable stream (writing large output!) ===
const writable = createWriteStream("output.txt", { encoding: "utf-8" });
writable.write("Line 1\n");                          // Write chunks!
writable.end();                                      // Signal end of data!

// === Duplex stream (both readable and writable!) ===
import net from "net";
const socket = net.createConnection(3000);          // Socket is a Duplex stream!
socket.write("Hello!");                              // Write to the socket!
socket.on("data", (data) => console.log(data));    // Read from the socket!

// === Custom Transform for CSV processing! ===
class CsvTransform extends Transform {
    transform(chunk: Buffer, encoding: BufferEncoding, callback: () => void) {
        const rows = chunk.toString().split("\n")
            .filter(Boolean)
            .map(row => row.split(",").map(cell => cell.trim()));
        this.push(JSON.stringify(rows));              // Transform CSV → JSON!
        callback();
    }
}

// === Compressed stream (gzip compression!) ===
import { createGzip } from "zlib";
await pipeline(
    createReadStream("large-file.txt"),
    createGzip(),                                   // Compress on-the-fly!
    createWriteStream("large-file.txt.gz")          // Write compressed output!
);
```

```python
# ========================================
# Python: Complete Streaming reference — EVERY Node.js stream pattern mapped!
# Python uses for loops, generators, and context managers instead of event-based streams!
# ========================================

from pathlib import Path                          # Built-in streaming via file iteration!

# === Readable stream equivalent (Python's for loop IS the readable stream!) ===
total_lines = 0
with open("large-file.csv") as f:                # context manager = auto-close on completion/error!
    for line in f:                               # Iterates line-by-line automatically!
        total_lines += 1                           # No chunks, no split needed — one line at a time!
print(f"Total: {total_lines} lines")              # Done when the loop finishes!

# === Transform stream equivalent (Python's generator IS the transform!) ===
def upper_transform(text_lines):                  # Generator that transforms each input!
    for line in text_lines:                       # Like Node.js's transform callback!
        yield line.upper()                         # 'yield' = like this.push(transformed_chunk)!
                                                    # Each yield produces one output (like each chunk!)

with open("input.txt") as infile, \              # Multiple context managers (like pipe chaining!)
     open("output.txt", "w") as outfile:
    for transformed_line in upper_transform(infile):  # Apply transform to each line!
        outfile.write(transformed_line + "\n")        # Write output — like writable stream!

# === pipeline equivalent (Python's generator chain IS the pipeline!) ===
from pathlib import Path

def process_csv(data_lines):                      # Transform CSV → JSON rows!
    for line in data_lines:
        fields = line.strip().split(",")           # Split by comma (like CSV row)!
        yield {"fields": [f.strip() for f in fields]}  # Yield one transformed dict!

with open("input.csv") as infile, \
     open("output.jsonl", "w") as outfile:
    for record in process_csv(infile):             # Like pipeline's transform step!
        import json
        outfile.write(json.dumps(record) + "\n")   # Like writable stream .write()!

# === Writable stream equivalent (file writing IS the writable stream!) ===
with open("output.txt", "w") as f:                # Open file for writing (like createWriteStream)!
    f.write("Line 1\n")                            # Write data (like writable.write())!
    f.write("Line 2\n")                            # More writes (buffered automatically!)
# File is automatically closed when the 'with' block exits!

# === Compressed stream equivalent (Python's gzip module IS the compression stream!) ===
import gzip                                       # Built-in compression — like Node.js zlib!

# Read compressed file (like createReadStream + unzip pipeline!)
with gzip.open("large-file.txt.gz", "rt") as f:  # Like createReadStream + createGunzip piped together!
    for line in f:                               # Iterates decompressed lines!
        total_lines += 1

# Write compressed file (like createGzip() pipe → createWriteStream!)
with open("input.txt", "rb") as infile, \        # Read raw bytes!
     gzip.open("output.txt.gz", "wb") as outfile:  # Compress on-the-fly while writing!
    shutil.copyfileobj(infile, outfile)             # Like pipeline(read → gzip → write)!

# === Generator-based streaming for memory efficiency (like Node.js streams!) ===
def iter_large_file(filepath: str, chunk_size: int = 8192):
    """Generator that yields file contents in chunks — NEVER loads entire file into memory!"""
    with open(filepath, "rb") as f:                # Open in binary mode for raw bytes!
        while chunk := f.read(chunk_size):         # Read exactly chunk_size bytes at a time!
            yield chunk                              # Yield each chunk (like stream.on('data')!)

for byte_chunk in iter_large_file("huge-binary.bin"):  # Process each chunk as it's read!
    process_data(byte_chunk)                        # Handle the data — memory usage stays constant!

# === asyncio.StreamReader for async network streaming (like Node.js net.Socket!) ===
import asyncio                                     # Like Node.js's native async streams!

async def stream_socket():
    reader, writer = await asyncio.open_connection("api.example.com", 443, ssl=True)
    writer.write(b"GET /api HTTP/1.1\r\nHost: api.example.com\r\n\r\n")
    while True:                                    # Like socket.on('data', cb)!
        line = await reader.readline()             # Read one line (like stream readable event)!
        if not line: break                         # Like socket.on('end')!
        print(line.decode())                       # Process the received data!
    writer.close()                                 # Like socket.end()!

# === BytesIO/StringIO for in-memory streams (like Node.js Buffer streams!) ===
import io
buffer = io.BytesIO(b"hello world")              # Create an in-memory byte stream!
data = buffer.read()                             # Read all data — like Buffer.toString()!
buffer.seek(0)                                   # Reset position — like creating a new stream!

string_buffer = io.StringIO("line 1\nline 2\n")  # In-memory text stream (like Node's Stream)!
for line in string_buffer:                         # Iterate lines — just like file iteration!
    print(line.strip())
```

### Mermaid: Stream vs Iterator Comparison

```mermaid
flowchart TD
    subgraph NODE_STREAMS["Node.js Stream API — Event-based streaming"]
        direction TB
        S1["createReadStream('file.csv') → Creates a Readable stream!\nMust set up .on('data'), .on('end'), .on('error') event listeners!"] --> S2["stream.pipe(transformStream).pipe(writeStream)\n→ Connect streams with pipe() to chain transformations!\nTransform must implement _transform(chunk, encoding, callback)!"]
        S3["For large files: stream events fire as data arrives\nMemory usage stays LOW because chunks are processed individually!"]
    end

    subgraph PY_ITERATORS["Python Iterators/Generators — Language-native streaming"]
        direction TB
        I1["for line in open('file.csv'):\n→ Python's for loop IS the readable stream!\nNo .on('data'), no event listeners, no setup needed!"] --> I2["def transform(lines):\n    for line in lines:\n        yield transform(line)\n→ A generator function IS the transform stream!\nyield = push to downstream consumer!\"]
        I3["for large files: next() is called on each iteration\nMemory usage stays LOW because one item is held at a time!\nLike Node.js streams but with zero setup code!"]
    end

    S2 -.->|"Both enable streaming of large data:\ntypeScript uses createReadStream + pipe()\nPython uses for loop + yield (built into the language!)!"| I2
```

### Key Notes: Streaming

1. **Python's `for line in file:` IS the readable stream** — no event listeners, no chunk setup, no error handlers needed. The for loop automatically handles all of this internally! Like how Node.js needs a complex Stream API; Python has streaming built into the language syntax!
2. **Generators (using `yield`) ARE the transform streams** — a generator function takes input items and yields transformed output items, one at a time. Like a Node.js Transform stream but with zero boilerplate!
3. **Python's gzip module handles compression on-the-fly** — `gzip.open(file, 'rt')` decompresses as you read, just like `createReadStream + createGunzip()` piped together in Node.js!

---

## 6. Events & Pub/Sub: EventEmitter → asyncio.Queue/Blinker/asyncio.create_task (Complete Deep Dive)

### Node.js EventEmitter → Python Equivalent Patterns

```typescript
// ========================================
// TypeScript/Node.js: EventEmitter pattern
// ========================================
import { EventEmitter } from "events";

const emitter = new EventEmitter();

emitter.on("user:created", (user: { id: number; name: string }) => {
    console.log(`New user: ${user.name}`);       // Listener 1
});

emitter.on("user:created", async (user) => {     // Can be async!
    await sendWelcomeEmail(user.email);
});

emitter.emit("user:created", { id: 1, name: "Alice" });

// One-time listener
emitter.once("app:start", () => console.log("App started!"));
```

```python
# ========================================
# Python: EventEmitter equivalents (multiple approaches!)
# ========================================

# Approach 1: asyncio.Queue as pub/sub (for async code — like a message broker!)
import asyncio

async def event_bus():
    queue = asyncio.Queue()                       # Like EventEmitter with on/emit!

    async def user_created_listener():            # Like emitter.on("user:created", cb)!
        while True:
            user = await queue.get()              # Wait for next message (like on 'data')!
            print(f"New user: {user['name']}")
            queue.task_done()

    async def send_email(user):                   # Like another listener!
        while True:
            msg = await queue.get()
            if msg.get('type') == 'email':
                await send_welcome_email(msg['data']['email'])
            queue.task_done()

    # Emit events (like emitter.emit())!
    asyncio.create_task(user_created_listener())  # Start listener task!
    asyncio.create_task(send_email())             # Start email listener task!

    # Emit events!
    await queue.put({'type': 'user', 'data': {'id': 1, 'name': "Alice"}})

# Approach 2: blinker library (sync EventEmitter — closest to Node.js!)
# pip install blinker
from blinker import signal

user_created = signal("user-created")             # Create a named signal!

@user_created.connect                             # Like emitter.on('event', cb)!
def on_user_created(sender, **extra):
    user = extra.get('user')
    print(f"New user: {user['name']}")            # Listener 1!

@user_created.connect                              # Multiple listeners!
async def on_email_welcome(sender, **extra):
    user = extra.get('user')
    await send_welcome_email(user['email'])       # Async listener!

# Emit the event (like emitter.emit())!
user_created.send(None, user={"id": 1, "name": "Alice"})
```

### Key Notes: Events & Pub/Sub

1. **Python has NO built-in EventEmitter** — unlike Node.js which includes it in the stdlib. Use `asyncio.Queue` for async pub-sub (it IS a message queue that works as an event bus), or the `blinker` package for synchronous pub-sub.
2. **asyncio.Queue is Python's answer to Node.js events** — producers `put()` items onto the queue (like `emit()`), and consumers `await get()` them (like `on()` listeners). Each item is delivered to exactly one consumer, like an event listener receiving a single event.
3. **blinker provides sync EventEmitter-like behavior** — if you need synchronous pub/sub (no async), blinker's signal pattern is the closest equivalent to Node.js's EventEmitter API.

---

## 7. Buffer & Binary Data: Buffer → bytes/bytearray/mmap/array (Complete Deep Dive)

### Every Node.js Buffer Operation Mapped to Python

```typescript
// ========================================
// TypeScript/Node.js: Buffer operations
// ========================================
const buf = Buffer.from("Hello World", "utf-8");  // Create from string!
console.log(buf.toString());                        // "Hello World"
console.log(buf.length);                            // 11 (bytes, not characters!)

const sliced = buf.slice(0, 5);                    // Slice without copying (shared memory)!
const combined = Buffer.concat([buf1, buf2]);      // Concatenate buffers!

const hex = buf.toString("hex");                   // "48656c6c6f20576f726c64"
const base64 = buf.toString("base64");             // Base64 encoding!

const numBuf = Buffer.alloc(4);                    // Allocate buffer with zeros!
numBuf.writeInt32BE(42, 0);                        // Write number at offset!
const readBack = numBuf.readInt32BE(0);            // 42!
```

```python
# ========================================
# Python: Binary data — EVERY Buffer operation mapped!
# ========================================

# Create from string (like Buffer.from('string', 'utf-8'))!
data = b"Hello World"                            # bytes literal — no Buffer class needed!
text = data.decode("utf-8")                       # "Hello World" (like buf.toString())
length = len(data)                                # 11 (bytes, like buf.length!)

# Slice without copying (creates new bytes object!) — bytes in Python are immutable!
sliced = data[:5]                                 # b"Hello" (like buf.slice(0, 5))

# Concatenate bytes (like Buffer.concat!)
combined = b"Hello" + b" " + b"World"             # b"Hello World" (one expression!)

hex_string = data.hex()                            # "48656c6c6f20576f726c64"
import base64
b64 = base64.b64encode(data).decode()             # "SGVsbG8gV29ybGQ=" (like buf.toString('base64'))

# Write/read numbers at offsets (like writeInt32BE — Python's struct module!)
import struct
packed = struct.pack(">I", 42)                    # ">I" = big-endian unsigned int!
read_back, = struct.unpack(">I", packed)          # 42! (unpack returns a tuple!)

# Mutable binary data (like Buffer.alloc — use bytearray for mutability!)
mutable = bytearray(16)                            # Allocate 16 zeroed bytes!
mutable[0] = 65                                    # Set first byte to 'A'!
mutable[1:5] = b"ello"                             # Set bytes 1-4!

# Memory-mapped files (like Node.js's fs.createReadStream for huge files!)
import mmap
with open("huge-data.bin", "r+b") as f:
    mm = mmap.mmap(f.fileno(), 0)                 # Memory-map the entire file!
    chunk = mm[0:1024]                             # Access as bytes — like a Buffer!
```

### Key Notes: Buffer & Binary Data

1. **Python has native `bytes` and `bytearray` types** — no separate Buffer class needed! `b"hello"` creates an immutable bytes object (like Node.js's Buffer); `bytearray(16)` creates a mutable one. Like TypeScript having `Uint8Array` as a primitive type!
2. **Use `struct` for packing/unpacking binary data** — like Node.js's `buf.writeInt32BE()` and `buf.readInt32BE()`, Python's `struct.pack(">I", value)` packs numbers into bytes at specific offsets.

---

## 8. Child Process: child_process → subprocess Complete Reference (Every Method Mapped)

### Every Node.js child_process Method with Exact Python Equivalent

| Node.js Method | Purpose | Python Equivalent | Example |
|---------------|---------|-----------------|---------|
| `exec(cmd, cb)` | Run command as shell string (async) | `subprocess.run(["cmd", "arg"], capture_output=True, shell=True)` | Same concept but returns result object! |
| `execFile(file, args, cb)` | Run binary directly with args (safer!) | `subprocess.run([file] + args, capture_output=True)` | Identical — pass command as list of args! |
| `spawn(cmd, args)` | Stream-based execution | `subprocess.Popen([cmd, ...], stdout=PIPE)` | Popen gives you access to stdin/stdout streams! |
| `spawnSync(cmd, args)` | Synchronous stream-like execution | `subprocess.run([cmd, ...], capture_output=True)` | run() with capture_output IS the sync version! |
| `execSync(cmd)` | Synchronous command (blocks!) | `subprocess.check_output([cmd, ...])` or `run(...).stdout` | Same blocking behavior! |
| `fork(module)` | Spawn new Node.js process with IPC | `subprocess.Popen(["python", "worker.py"], stdin=PIPE, stdout=PIPE)` + message queue | Python uses multiprocessing for true fork-like behavior! |

### Mermaid: child_process → subprocess Mapping

```mermaid
flowchart TD
    subgraph NODE_CHILD["Node.js child_process — 5 Methods"]
        direction TB
        C1["exec(cmd_string, callback) — Shell string execution\n→ Returns via callback(error, stdout, stderr)!"] --> C2["execFile(binary, [args], callback)\n→ Safer: no shell injection. Binary + args separately!"]
        C3["spawn(cmd, [args]) — Stream-based!\n→ child.stdout.on('data', cb) for continuous output!\nBest for long-running processes like tail/ffmpeg!"] --> C4["fork(module.js)\n→ Spawn NEW Node.js process with IPC channel!\nLike creating a worker thread in the same machine!"]
    end

    subgraph PY_SUBPROCESS["Python subprocess — Complete Equivalents"]
        direction TB
        S1["subprocess.run(['cmd', 'arg'], shell=True, capture_output=True)\n→ Like exec() but returns result object with .stdout/.stderr/\n.returncode — no callback needed!"] --> S2["subprocess.run([binary] + args, capture_output=True)\n→ Identical to execFile: command as list of arguments!\nSafest method — no shell injection possible!"]
        S3["subprocess.Popen(['cmd'], stdout=PIPE)\n→ Like spawn: access .stdout for streaming!\nIterate over process.stdout for line-by-line output!\nLike Node.js's child.stdout.on('data') but simpler!"] --> S4["multiprocessing.Process(target=function, args=(...))\n→ True fork-like process spawning!\nProcesses share no memory — like Node.js fork with IPC!"]
    end

    C1 -.->|"Both execute shell commands:\ntypeScript uses exec(callback)\nPython uses subprocess.run(result object!) Structured return!"| S1
```

---

## 9. DNS Module Complete Reference (Node.js dns → Python dnspython/asyncio.getaddrinfo)

### Every Node.js DNS Method Mapped to Python

```typescript
// ========================================
// TypeScript/Node.js: DNS module — every method!
// ========================================
import { dns } from "node:dns";

// === Basic lookup (A record resolution) ===
dns.lookup("example.com", (err, address, family) => {
    console.log(`IPv${family}: ${address}`);  // e.g., "IPv4: 93.184.216.34"
});

// === Resolve all A records ===
dns.resolve4("example.com", (err, addresses) => {   // ["93.184.216.34"]
    console.log(addresses);
});

// === Resolve IPv6 AAAA records ===
dns.resolve6("example.com", (err, addresses) => {
    console.log(addresses);                          // e.g., ["2606:2800:220:..."]
});

// === Reverse DNS lookup (PTR record) ===
dns.reverse("93.184.216.34", (err, hostnames) => {  // ["example.com"]
    console.log(hostnames);
});

// === MX records (mail servers) ===
dns.resolveMx("gmail.com", (err, addresses) => {
    console.log(addresses);                          // [{priority: 10, exchange: "alt1.gmail-smtp-in.l.google.com"}]
});

// === TXT records (SPF, DKIM, etc.) ===
dns.resolveTxt("example.com", (err, txtRecords) => {
    console.log(txtRecords);                         // [["v=spf1 ..."]]
});

// === NS records (name servers) ===
dns.resolveNs("example.com", (err, nsRecords) => {  // ["ns1.example.com"]
    console.log(nsRecords);
});

// === ANY query (all record types!) ===
dns.resolveAny("example.com", (err, records) => {
    console.log(records);
});
```

```python
# ========================================
# Python: DNS resolution — dnspython for full coverage!
# Basic lookups via socket; advanced queries via dnspython!
# ========================================

import socket                                       # Basic DNS via stdlib!

# === Basic lookup (A record — like dns.lookup()) ===
addresses = socket.getaddrinfo("example.com", 443, type=socket.SOCK_STREAM)
for addr in addresses:
    print(f"Family: {addr[0]}, Address: {addr[4][0]}")
    # Family: <AddressFamily.AF_INET: 2>, Address: 93.184.216.34

# === Reverse DNS (like dns.reverse()) ===
hostname = socket.getfqdn("93.184.216.34")          # "example.com"
print(hostname)

# === Advanced DNS queries (requires pip install dnspython!) ===
import dns.resolver
import dns.reversename
import dns.rdataclass
import dns.rdatatype

resolver = dns.resolver.Resolver()                  # Configure custom DNS server!
resolver.nameservers = ["8.8.8.8"]                   # Use Google's DNS!

# A records (like dns.resolve4()) — with TTL and priority!
a_records = resolver.resolve("example.com", "A")
for rdata in a_records:
    print(f"A record: {rdata.address} (TTL: {a_records.rrset.ttl})")

# AAAA records (IPv6)
aaaa_records = resolver.resolve("example.com", "AAAA")
for rdata in aaaa_records:
    print(f"AAAA: {rdata}")

# MX records (mail servers) — with priority!
mx_records = resolver.resolve("gmail.com", "MX")
for rdata in mx_records:
    print(f"Priority {rdata.preference}: {rdata.exchange}")

# TXT records (SPF, DKIM, etc.)
txt_records = resolver.resolve("example.com", "TXT")
for rdata in txt_records:
    for text in rdata.strings:
        print(text.decode())                          # "v=spf1 include:..."

# NS records (name servers)
ns_records = resolver.resolve("example.com", "NS")
for rdata in ns_records:
    print(f"Name server: {rdata.target}")

# CNAME records
cname = resolver.resolve("www.example.com", "CNAME")
print(cname[0].target)                               # example.com!

# === Async DNS resolution (like dns.promises.resolve())! ===
import asyncio

async def async_lookup(hostname: str):
    """Async DNS lookup — like dns.promises.lookup()!"""
    infos = await asyncio.getaddrinfo(
        hostname, 443, type=socket.SOCK_STREAM
    )
    return [info[4][0] for info in infos]            # Return all IP addresses!

addresses = await async_lookup("example.com")         # ['93.184.216.34']
```

### Key Notes: DNS Module

1. **Basic DNS lookups work via Python's `socket.getaddrinfo()`** — this is built into the stdlib and covers A/AAAA lookups. For advanced queries (MX, TXT, NS), use the `dnspython` package (`pip install dnspython`).
2. **Python's async DNS resolution uses `asyncio.getaddrinfo()`** — like Node.js's `dns.promises.resolve4()`. This doesn't block the event loop!

---

## 10. dgram (UDP) Socket Programming: Node.js dgram → Python socket (Complete Deep Dive)

### Every Node.js UDP Operation Mapped to Python

```typescript
// ========================================
// TypeScript/Node.js: dgram (UDP) module
// ========================================
import dgram from "dgram";

// === Create a UDP socket (server side!) ===
const server = dgram.createSocket("udp4");          // IPv4 UDP socket!

server.on("message", (message, rinfo) => {         // Listen for incoming messages!
    console.log(`Received: ${message.toString()} from ${rinfo.address}:${rinfo.port}`);
    // Send response back!
    server.send(Buffer.from("Pong!"), rinfo.port, rinfo.address, (err) => {
        if (err) throw err;
    });
});

server.on("error", (err) => { console.error(err); });

server.bind(41234, "0.0.0.0");                      // Listen on port 41234!
// server.address() → { address: '0.0.0.0', family: 'IPv4', port: 41234 }

// === Create a UDP client (send to a server!) ===
const client = dgram.createSocket("udp4");          // IPv4 UDP socket!

const message = Buffer.from("Hello Server!");
client.send(message, 41234, "127.0.0.1", (err) => {
    if (err) throw err;
    client.close();                                 // Close when done!
});
```

```python
# ========================================
# Python: UDP via socket module — EVERY Node.js dgram operation mapped!
# ========================================

import socket

# === Server side (like dgram.createSocket + bind!) ===
server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  # AF_INET = IPv4, SOCK_DGRAM = UDP!
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # Like Node.js's address in use!
server.bind(("0.0.0.0", 41234))                        # Bind to port — like server.bind(41234)!

# Listen for incoming messages (like server.on('message', cb))!
while True:
    message, addr = server.recvfrom(4096)              # Receive up to 4096 bytes!
    print(f"Received: {message.decode()} from {addr}")  # Like rinfo.address:rinfo.port!

    # Send response back (like server.send(message, port, host, cb))!
    server.sendto(b"Pong!", addr)                      # Send to the same address we received from!

# === Client side (like dgram.createSocket + send!) ===
client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  # Same as server but used differently!
client.sendto(b"Hello Server!", ("127.0.0.1", 41234))     # Send to specific address:port!
response, _ = client.recvfrom(4096)                        # Receive response!
print(response.decode())                                   # "Pong!"
client.close()                                             # Like client.close() in Node.js!

# === Async UDP server (with asyncio — like Node.js's async dgram!) ===
import asyncio

async def udp_server():
    """Async UDP server — non-blocking, handles multiple clients concurrently!"""
    server = await asyncio.udp_server(("0.0.0.0", 41234))  # Async socket binding!
    while True:
        message, addr = await server.recvfrom(4096)         # Like on('message') but async!
        print(f"Received from {addr}: {message.decode()}")
        await server.sendto(b"Pong!", addr)                 # Send response (async!)

asyncio.run(udp_server())  # Run the async UDP server!
```

### Key Notes: dgram (UDP) Socket Programming

1. **Python's `socket` module with `SOCK_DGRAM` is exactly Node.js's `dgram.createSocket("udp4")`** — both create IPv4 UDP sockets. The API differs in that Python exposes the raw socket directly while Node.js wraps it in a higher-level Stream-like interface.
2. **Both ecosystems have similar UDP patterns**: server creates a socket, binds to a port, receives messages, sends responses. Client creates a socket, sends to a specific address:port pair.

---

## 11. URL Parsing Exhaustive: Node.js URL/URLSearchParams → urllib.parse/yarl (Complete Deep Dive)

### Every URL Operation Mapped — Complete Side-by-Side Reference

```typescript
// ========================================
// TypeScript/Node.js: URL class + URLSearchParams
// ========================================
import { URL, URLSearchParams } from "url";

const url = new URL("https://api.example.com:8080/users?page=2&limit=10?sort=name");

console.log(url.protocol);       // "https:"
console.log(url.hostname);       // "api.example.com"
console.log(url.port);           // "8080"
console.log(url.pathname);       // "/users"
console.log(url.search);         // "?page=2&limit=10&sort=name"
console.log(url.hash);           // "" (empty — no hash in this URL!)

// Query parameters
const params = url.searchParams;  // URLSearchParams object!
params.set("page", "3");          // Modify query param!
params.delete("limit");           // Remove a param!
params.append("tag", "python");   // Add another value for 'tag'!
console.log(params.get("page"));  // "3"
console.log(params.get("tag"));   // "python"

// Stringify modified URL
const urlString = url.toString(); // "https://api.example.com:8080/users?page=3&sort=name&tag=python"
```

```python
# ========================================
# Python: URL parsing — urllib.parse (stdlib) + yarl (modern third-party!)
# EVERY Node.js URL operation mapped!
# ========================================

from urllib.parse import urlparse, urlunparse, urlencode, parse_qs, urljoin, quote_plus
from yarl import URL                  # pip install yarl — MUCH cleaner API!

# === Parse a URL (like new URL(urlString)!) ===
parsed = urlparse("https://api.example.com:8080/users?page=2&limit=10")
print(parsed.scheme)          # "https"    — like url.protocol.split(':')[0]!
print(parsed.hostname)        # "api.example.com"  — same as Node.js!
print(parsed.port)            # 8080     — integer, like Node.js!
print(parsed.path)            # "/users"   — like url.pathname!
print(parsed.query)           # "page=2&limit=10"  — raw query string!
print(parsed.fragment)        # ""         — empty string (like url.hash!)

# === URLSearchParams equivalent with parse_qs()! ===
query_params = parse_qs(parsed.query)       # Like url.searchParams!
print(query_params["page"])            # ['2'] (list because params can repeat!)
params_dict = {k: v[0] for k, v in query_params.items()}  # {'page': '2', 'limit': '10'}

# === Modify query params and stringify (like params.set/delete!) ===
params_dict["page"] = "3"              # Set a param!
del params_dict["limit"]               # Delete a param!
params_dict["tag"] = "python"          # Add a param!
new_query = urlencode(params_dict)     # "page=3&sort=name&tag=python"!

# === Rebuild the URL (like url.toString())! ===
updated_parsed = parsed._replace(query=new_query)  # Replace query portion!
updated_url = urlunparse(updated_parsed)           # "https://api.example.com:8080/users?page=3&tag=python"

# === yarl.URL — THE MODERN PYTHON WAY (like Node.js URL class but better!) ===
url = URL("https://api.example.com:8080/users?page=2&limit=10")
print(url.scheme)                  # "https" (same as parsed.scheme above!)
print(url.host)                    # "api.example.com"
print(url.port)                    # 8080
print(url.path)                    # "/users"
print(url.query["page"])           # "2"  — Direct access like Node.js params.get()!

url = url.with_port(9000)          # Change port! (like url.port = "9000")
url = url.update_query(page="3", tag="python")   # Update multiple params at once!
print(str(url))                    # "https://api.example.com:9000/users?page=3&tag=python"
```

### Key Notes: URL Parsing

1. **Python's urllib.parse is the stdlib equivalent of Node.js's url module** — `urlparse()` is like `new URL()`, and `parse_qs()` gives you query parameters (like `url.searchParams`). However, Python's stdlib API is more verbose!
2. **Use `yarl.URL` for a cleaner API** — it's the closest thing to Node.js's URL class in terms of elegance. Direct property access (`url.host`, `url.query["key"]`) and chainable methods make URL manipulation feel natural.

---

## 12. querystring Module → URLSearchParams Mapping (Complete Reference)

### Every Node.js querystring Method with Exact Python Equivalent

| Node.js querystring | Purpose | Python Equivalent | Example |
|--------------------|---------|-----------------|---------|
| `qs.stringify(obj)` | Object → query string | `urllib.parse.urlencode(dict)` | `{"a": 1, "b": 2}` → `"a=1&b=2"` |
| `qs.parse(str)` | Query string → object | `urllib.parse.parse_qs(str)` | `"a=1&b=2"` → `{"a": ["1"], "b": ["2"]}` |
| `qs.escape(str)` | Escape special chars | `urllib.parse.quote(str)` | `/path` → `%2Fpath` |
| `qs.unescape(str)` | Unescape special chars | `urllib.parse.unquote(str)` | `%2Fpath` → `/path` |

---

## 13. zlib Compression/Decompression: Node.js zlib → Python zlib/bzip2/gzip/zstandard (Complete Deep Dive)

### Every Node.js zlib Method Mapped to Python

```typescript
// ========================================
// TypeScript/Node.js: zlib module — every method!
// ========================================
import { gzip, gunzip, deflate, inflate, createGzip, createGunzip } from "zlib";
import { promisify } from "util";

const gzipAsync = promisify(gzip);            // Promisify the callback API!
const gunzipAsync = promisify(gunzip);

// === Gzip compression (synchronous!) ===
const compressed = await gzipAsync(Buffer.from("Hello World!"));
console.log(compressed.length);                // Smaller than original!

// === Gunzip decompression (synchronous!) ===
const decompressed = await gunzipAsync(compressed);
console.log(decompressed.toString());          // "Hello World!"

// === Stream-based compression (for large data!) ===
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";

await pipeline(                                 // Like pipe() but with error handling!
    createReadStream("huge-file.txt"),
    createGzip(),                               // Compress on-the-fly!
    createWriteStream("huge-file.txt.gz")       // Write compressed output!
);
```

```python
# ========================================
# Python: Compression — EVERY Node.js zlib method mapped!
# Python has ALL formats built-in: zlib, gzip, bz2, lzma, zstandard!
# ========================================

import zlib                                    # Like Node.js's zlib module — built in!
import gzip                                    # For .gz files (like Node.js createGzip!)
import bz2                                     # Bzip2 compression!
import lzma                                    # LZMA/XZ compression!
import shutil                                  # Like pipeline() for piped operations!

# === zlib compression (like zlib.gzip — raw deflate format!) ===
original = b"Hello World!" * 100               # Large enough to see compression ratio!
compressed = zlib.compress(original)            # Compress with default level!
decompressed = zlib.decompress(compressed)      # Decompress — EXACT original!
print(f"Ratio: {len(compressed) / len(original):.2%}")  # e.g., "3.45%" — impressive!

# === gzip compression (like Node.js createGzip for .gz files!) ===
with open("input.txt", "rb") as f_in:          # Read raw bytes!
    with gzip.open("output.gz", "wb") as f_out:  # Compress to .gz automatically!
        shutil.copyfileobj(f_in, f_out)         # Like pipeline(read → gzip → write)!

# === gzip decompression (like Node.js createGunzip!) ===
with gzip.open("output.gz", "rt") as f:        # 'rt' = text mode (auto-decompress + decode!)
    content = f.read()                          # Read decompressed text directly!

# === bz2 compression (like zlib but different algorithm — great for text!) ===
compressed_bz2 = bz2.compress(original)         # Compress with bzip2!
decompressed_bz2 = bz2.decompress(compressed_bz2)  # Decompress back!

# === zstandard (zstd) compression (like Node.js's pako/zlib but faster!) ===
import zstandard as zstd                        # pip install zstandard — like npm install pako!
cctx = zstd.ZstdCompressor(level=3)             # Compression level 1-22!
compressed_zstd = cctx.compress(original)       # Compress with zstandard!
dctx = zstd.ZstdDecompressor()                  # Create decompressor!
decompressed_zstd = dctx.decompress(compressed_zstd)  # Decompress back!
```

### Key Notes: Compression/Decompression

1. **Python has ALL compression formats built-in** — `zlib`, `gzip`, `bz2`, `lzma` (xz), and `zstandard` (via third-party package). Node.js only has zlib (deflate/gzip); pako is a separate npm package for brotli/lz4.
2. **Python's gzip.open() IS the compressed file handler** — you don't need to manually pipe through createGzip/createGunzip; `gzip.open()` handles compression/decompression transparently just like opening a regular file!

---

## 14. assert Module → pytest Comparison (Exhaustive)

### Node.js assert vs Python pytest — Complete Mapping

```typescript
// ========================================
// TypeScript/Node.js: assert module
// ========================================
import assert from "assert";

// Basic assertions
assert.strictEqual(1, 1);           // Strict equality (===)
assert.deepStrictEqual({a: 1}, {a: 1});  // Deep equality
assert.throws(() => { throw new Error("fail"); }, Error);  // Assert an error is thrown!
assert.doesNotThrow(() => { return 42; });  // Assert no error!

// Custom message
assert.strictEqual(actual, expected, "Custom failure message!");
```

```python
# ========================================
# Python: assert statement + pytest — EVERY Node.js assert mapped!
# ========================================

# Basic assertion (Python's built-in assert statement!)
assert 1 == 1, "Values should be equal"              # Like assert.strictEqual(1, 1) in Node.js!
assert {"a": 1} == {"a": 1}                          # Dict comparison = deep equality!
assert [1, 2, 3] == [1, 2, 3]                        # List comparison!

# Assert exception is raised (like assert.throws())!
import pytest

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):            # Like assert.throws(() => { throw new Error() }, Error)!
        result = 1 / 0                                # This should raise ZeroDivisionError!
        assert False, "Should not reach here!"

# Assert custom exception message!
def test_invalid_age():
    with pytest.raises(ValueError, match="must be positive"):  # Check both type AND message!
        raise ValueError("Age must be positive")

# Assert no exception is raised (like assert.doesNotThrow())!
def test_no_exception():
    result = 1 + 2                                    # No error expected!
    assert result == 3                                # Standard assertion works!

# pytest's advanced assertions (built into pytest, not Node.js's basic assert!)
assert isinstance("hello", str)                       # type check!
assert callable(print)                                # function check!
assert "python" in "i love python"                    # membership check!
```

### Key Notes: assert Module

1. **Python's `assert` is a language keyword** — like TypeScript's `==` for strict equality. It stops execution with an AssertionError if the condition is False. Node.js's `assert.strictEqual()` does the same thing but as a function call!
2. **pytest.raises() IS Node.js's assert.throws()** — it catches exceptions during test execution and verifies they're of the expected type. Unlike basic Python assert which just checks conditions, pytest.raises validates that errors are raised correctly.

---

## 15. diagnostics_channel Module → Python Logging Comparison

### Node.js diagnostics_channel → Python logging module

```typescript
// ========================================
// TypeScript/Node.js: diagnostics_channel (experimental!)
// ========================================
import { channel } from "node:diagnostics_channel";

const apiChannel = channel("api.request");            // Create a diagnostic channel!
apiChannel.subscribe((message, name) => {
    console.log(`[Diagnostic] ${name}:`, message);   // Listen for published messages!
});

// Publish to the channel (like emitting an event!)
apiChannel.publish({ method: "GET", url: "/users" });
```

```python
# ========================================
# Python: logging module — production-grade observability!
# ========================================
import logging

# Create a named logger (like diagnostics_channel.createChannel())!
api_logger = logging.getLogger("api.request")        # Named logger for API events!
api_logger.setLevel(logging.INFO)                    # Only process INFO and above!

class DiagnosticFormatter(logging.Formatter):
    """Custom formatter that adds diagnostic context!"""
    def format(self, record):
        log_entry = super().format(record)
        return f"[Diagnostic] {record.name}: {log_entry}"

handler = logging.StreamHandler()                    # Output to console (like console.log!)
handler.setFormatter(DiagnosticFormatter(
    fmt="%(asctime)s [%(name)s] %(levelname)s: %(message)s"
))
api_logger.addHandler(handler)                       # Register the handler!

# Publish diagnostic messages (like channel.publish())!
api_logger.info({"method": "GET", "url": "/users"})  # Like apiChannel.publish()!
api_logger.warning({"status": 404, "duration_ms": 52})
api_logger.error({"status": 500, "error": "Internal Server Error"})

# For production-grade structured logging: use structlog (like Pino in TypeScript!)
# pip install structlog
import structlog

struct_logger = structlog.get_logger("api.request")  # Like createChannel()!
struct_logger.bind(method="GET", url="/users").info("request_started")  # Context + event!
```

### Key Notes: diagnostics_channel vs Logging

1. **Node.js's diagnostics_channel is for runtime observability** — it allows libraries to publish diagnostic events that tools can subscribe to. Python's equivalent is the `logging` module with named loggers, but for structured logging (like Pino in TypeScript), use the `structlog` package.
2. **Python has no exact equivalent to diagnostics_channel's publish/subscribe model** — the closest is `blinker` signals or custom event buses using `asyncio.Queue`.

---

## 16. timers Module: setTimeout/setInterval → asyncio.sleep/time.sleep Complete Reference

### Every Node.js Timer Operation Mapped to Python

| Node.js | Purpose | Python Equivalent | Example |
|---------|---------|-----------------|---------|
| `setTimeout(cb, ms)` | Execute once after delay (async) | `await asyncio.sleep(ms/1000)` | Run after N seconds async! |
| `setInterval(cb, ms)` | Execute repeatedly every delay | Loop with `await asyncio.sleep()` | Custom repeat loop! |
| `clearTimeout(id)` | Cancel pending timeout | Task.cancel() | Like cancelling a coroutine! |
| `setImmediate(cb)` | Execute on next event loop tick | `asyncio.create_task(cb())` + `await asyncio.sleep(0)` | Schedule for next tick! |
| `process.nextTick(cb)` | Execute after current operation, before any I/O | Just call the function directly in sync code! | Python's GIL handles this naturally! |
| `setTimeout(cb, 0)` | Defer execution to next event loop iteration | `await asyncio.sleep(0)` | Same behavior! |

---

## 17. util Module: promisify/inspect/deepEqual → Python functools/inspect (Complete Deep Dive)

### Every Node.js util Method Mapped to Python

| Node.js util | Purpose | Python Equivalent | Example |
|-------------|---------|-----------------|---------|
| `util.promisify(fn)` | Callback function → Promise function | Direct async function definition or `asyncio.wrap_future()` | Python has native async/await — no promisify needed! |
| `util.inspect(obj)` | Object → string representation for debugging | `repr(obj)` + `pprint.pprint(obj)` | Built-in repr() is like util.inspect! pprint formats it nicely! |
| `util.deepEqual(a, b)` | Deep equality check | `a == b` for dicts/lists (Python's built-in does deep comparison!) | No separate utility needed — Python's == IS deep by default! |
| `util.inherits(Child, Parent)` | Class inheritance syntax | `class Child(Parent): pass` | Python uses regular class inheritance! |
| `util.format(fmt, ...args)` | String formatting | f-strings, `.format()`, `%` operator | f-strings are like util.format but with template literal syntax! |

---

## 18. Module Resolution: Node.js require → Python import Complete Reference

### Every Node.js Module Pattern Mapped to Python

| Node.js Pattern | Purpose | Python Equivalent | Example |
|----------------|---------|-----------------|---------|
| `require('module')` | Import module (CommonJS) | `import module` | Same concept — access via `module.function()`! |
| `const { func } = require('mod')` | Destructured import | `from mod import func` | Like TypeScript's destructuring imports! |
| `const x = require('./relative/path')` | Local file import | `from . import relative_path` or `import relative_path` | Relative imports with dot notation (. and ..)! |
| `require('module').default` | Default export (CommonJS module.exports) | Direct function/class from the module | Python doesn't have default exports — everything is named! |
| `module.exports = {...}` | Export multiple values | `__all__ = ['func1', 'func2']` + individual assignments | Like TypeScript's `export { func1, func2 }` at the bottom of a file! |
| `require.cache` | Module cache (prevent reload) | `sys.modules` — identical concept! | Both caches loaded modules by name to prevent duplicate loading! |
| `require.resolve('module')` | Get absolute path to module | `import importlib; importlib.util.find_spec('module').origin` | Like TypeScript's `require.resolve()` but more complex in Python! |

### Mermaid: Node.js Module Resolution → Python Import System

```mermaid
flowchart TD
    subgraph NODE_MODULE["Node.js Module Resolution (CommonJS/ESM)"]
        direction TB
        M1["require('./file') — Resolves relative path\n→ Looks in node_modules/ first, then local files!\n→ Cache: require.cache maps module ID → module object!"] --> M2["module.exports = { func1, func2 }\n→ Export an object with properties (like export * from 'mod'!"]
        M3["import { func } from './file'\n→ ES Modules syntax (TypeScript!)\n→ Import specific named exports!"]
    end

    subgraph PYTHON_IMPORT["Python Import System"]
        direction TB
        I1["from .package import func\n→ Relative imports work in packages (. = current dir, .. = parent)!\n→ Cache: sys.modules maps module name → module object (identical to Node.js!)!"] --> I2["def func1(): pass\ndef func2(): pass\n__all__ = ['func1', 'func2']\n→ Export is automatic — everything in the module is exported!\nLike having all exports implicitly available!"]
        I3["from package import func as my_func\n→ Named imports with aliasing (like TypeScript's 'as' import!)!\nPython also supports: from package import * (but __all__ controls what gets exported!)"]
    end

    M1 -.->|"Both use module caches to prevent reload:\ntypeScript uses require.cache (object)\nPython uses sys.modules (dict) — identical mechanism! Both check cache BEFORE loading!"| I1
```

### keynotes: Module Resolution

1. **Node.js requires explicit exports (`module.exports`); Python auto-exports everything** — in TypeScript, you must explicitly list what to export; in Python, every name in a module file is available as an import unless you define `__all__`. Like how `export default` works in ES modules vs Python's implicit all-names-export.
2. **Both ecosystems use module caches** — Node.js `require.cache` and Python `sys.modules` serve the identical purpose: prevent loading the same module twice, enabling circular imports to work (partially).

---

## 19. constants Module → os/constants Comparison

| Node.js constants | Purpose | Python Equivalent |
|------------------|---------|-----------------|
| `constants.SIGINT` | SIGINT signal constant | `signal.SIGINT` |
| `constants.SIGTERM` | SIGTERM signal constant | `signal.SIGTERM` |
| `constants.errno.EEXIST` | File exists error code | `errno.EEXIST` |
| `constants.O_RDONLY` | File open flag (read-only) | `os.O_RDONLY` |

---

## 20. Popular npm Packages → Python Libraries (Every Package Mapped — 50+ Pairs)

### Complete Mapping Table: npm → Python (Production, Development, Utility)

| npm Package | Purpose | Python Equivalent Library | pip Install | Notes for TS Devs |
|-------------|---------|------------------------|------------|-----------------|
| **express** | Web framework with routing | **FastAPI** (modern, auto-docs) or **Flask** (minimal) | `pip install fastapi uvicorn` | FastAPI auto-generates OpenAPI docs from type hints — no manual swagger config needed! Like Express but with better DX! |
| **axios** | HTTP client for API calls | **requests** (sync) or **httpx** (async) | `pip install requests` | Like axios but simpler — one call does everything! No interceptor setup, no baseURL config needed! |
| **zod** | Runtime validation with TypeScript types | **Pydantic v2** (auto-validates on type hints!) | `pip install pydantic` | Pydantic's validators ARE your type hints! No separate schema definition needed — same function signature validates AND types! |
| **cors** | CORS middleware for Express | Built into FastAPI/Flask | No install needed | Add one line: `app.add_middleware(CORSMiddleware)` — like Express cors() middleware! |
| **redis** | Redis client | **redis-py** or **aioredis** | `pip install redis` | Like Node.js ioredis; separate sync (redis) and async (aioredis) packages for different I/O styles. Same Redis API in both ecosystems! |
| **dotenv** | Load .env file into process.env | **python-dotenv** or just use `os.environ.get()` | `pip install python-dotenv` | Python also has built-in os.environ — but python-dotenv auto-loads .env files at startup like Node.js dotenv does! |
| **pg/pg-promise** | PostgreSQL client | **psycopg2-binary** (sync) or **asyncpg** (async) | `pip install psycopg2-binary` | Like TypeScript's pg; Python has separate sync/async packages. psycopg2 is the most widely used PostgreSQL driver in Python! |
| **winston/pino** | Structured logging library | **logging** (stdlib) or **structlog** | Built-in or `pip install structlog` | Python's stdlib logging module rivals Winston/Pino — supports levels, formatters, handlers out of the box! Like having pino built into Python! |
| **uuid/nanoid** | Generate unique IDs | **uuid** (stdlib) | Built-in! `import uuid; print(uuid.uuid4())` | No npm package needed — Python has it built in! Like how TypeScript needs nanoid from npm but Python has uuid from stdlib! |
| **chalk** | Terminal color formatting | **rich** or **colorama** | `pip install rich` | Rich provides beautiful terminal output with colors, tables, progress bars — like chalk but WAY more powerful! |
| **commander** | CLI argument parsing | **argparse** (stdlib) or **click**/typer | Built-in or `pip install click` | argparse is like commander.js but less ergonomic. Click/Typer are more like commander with a cleaner API! |
| **glob/minimatch** | Glob pattern matching | **glob** + **pathlib.Path.glob()** (stdlib!) | Built-in! `Path("src").rglob("*.py")` | Python's pathlib has built-in glob support — no npm package needed! Like having minimatch integrated into the language! |
| **lodash/ramda** | Utility functions library | **functools**, **itertools**, **operator** (all stdlib!) + **pydash** | Built-in or `pip install pydash` | Python's stdlib has most lodash utilities built in! functools.partial, operator.itemgetter, itertools.chain — no npm package needed! |
| **mongoose** | MongoDB ODM with schemas | **Motor** (async) or **pymongo** + **pydantic-mongo** | `pip install motor` | Like TypeScript's mongoose; Motor provides async MongoDB driver. pydantic models validate documents at insertion time! |
| **eslint/prettier** | Linting + formatting | **ruff** (linting) + **ruff format** (formatting) | `pip install ruff` | One tool replaces BOTH ESLint AND Prettier! 10-100x faster than both combined! Like having a Swiss Army knife instead of individual tools! |
| **jest** | Test runner + assertions + mocking | **pytest** + **pytest-cov** (coverage) + **pytest-mock** | `pip install pytest pytest-cov` | pytest is the Jest of Python — test discovery, assertions, fixtures, coverage, all working together! Like how Jest bundles everything! |
| **typescript** | Type checking + compilation | **mypy** (type checking only) | `pip install mypy` | Like TypeScript's type checker; but mypy doesn't compile to JS — it only checks types! No output code produced. |
| **nodemon** | Auto-restart on file changes | **uvicorn --reload** (FastAPI) or **watchfiles** | Built-in or `pip install watchfiles` | Like nodemon watching for changes and restarting. uvicorn has --reload built in! FastAPI development servers auto-reload by default! |
| **webpack/esbuild/vite** | JavaScript bundlers | N/A — Python doesn't need bundling! | N/A | This is a fundamental difference: TypeScript apps are bundled; Python apps run natively without bundling! No equivalent needed! |
| **ts-node/ts-jest** | Run TS directly (no compile step) | Python runs .py files directly! No compilation ever! | Built-in! | This is the core difference: TypeScript requires a compile step; Python executes source code directly — no ts-node equivalent needed! |
| **@types/express** | TypeScript definitions for Express | NO EQUIVALENT — types are IN the source code! | N/A | Python's types come from the module itself, not separate packages! Like how Node.js has inline JSDoc; Python is more integrated. |
| **prisma/typeorm** | Database ORM/ODM | **SQLAlchemy 2.0** + **alembic** (migrations) | `pip install sqlalchemy alembic` | SQLAlchemy is like TypeScript's Prisma but more flexible! Alembic handles database migrations like Prisma migrate. Both ecosystems have mature ORM solutions! |
| **bull/bullmq** | Job queues | **celery** (async tasks) or **rq** (Redis-based jobs) | `pip install celery` | Like bull for background jobs; celery handles async task queues with Redis/RabbitMQ backend. rq is simpler, like BullMQ but for Python! |
| **socket.io** | Real-time WebSocket communication | **FastAPI/WebSocket** or **flask-socketio** | `pip install flask-socketio` | Like socket.io for real-time; Python has built-in WebSocket support in FastAPI/Flask. No separate package needed for basic WebSockets! |
| **sharp** | Image processing | **Pillow (PIL)** + **opencv-python** | `pip install Pillow opencv-python` | Sharp does image manipulation; Pillow is the Python equivalent (like @types/sharp). OpenCV adds advanced computer vision! Both packages cover similar ground! |
| **form-data** | Multipart form data builder | **requests** handles multipart automatically! | Built into requests library! | No separate package needed — just use `requests.post(url, files={'file': open('doc.pdf', 'rb')})`! Like how axios FormData works in TypeScript! |
| **uuid-apikey** | API key validation middleware | **fastapi.security.APIKeyHeader** or custom validator | Built into FastAPI! | Add one decorator: `@app.get("/protected", dependencies=[Depends(APIKeyHeader())])`. No separate npm package needed — built right into FastAPI! |
| **crypto-js** | Client-side cryptography | **cryptography** (production crypto) + **hashlib** (stdlib!) | `pip install cryptography` | Like crypto-js for encryption; Python's cryptography library handles AES/RSA/ECDSA. hash.sha256 is like crypto.createHash('sha256') in Node.js! |
| **dayjs/moment** | Date/time manipulation | **datetime.stdlib** + **pendulum** (like dayjs but more powerful!) | Built-in or `pip install pendulum` | Python's datetime module has most dayjs functionality. pendulum adds the nice API (duration, relative_time) that dayjs is known for! |
| **helmet** | Security headers middleware | **FastAPI.middleware.cors** + custom security headers middleware | No npm package — use starlette's middleware! | Like helmet.js adding security headers; add middleware to FastAPI/Flask to set X-Frame-Options, CSP, etc. headers automatically on every response! |
| **morgan** | HTTP request logging middleware | **python-multipart** or custom Starlette middleware | Built into Starlette/FastAPI! | Like morgan for access logs; add a middleware that logs method + path + status + duration for every incoming request! Like FastAPI's built-in structured logging! |
| **dotenv-flow** | Environment-specific .env files | **python-dotenv** + **os.environ.get(key, default)** | `pip install python-dotenv` | Like dotenv-flow loading .env.local in dev and .env.prod in production. Use environment variable to switch between different .env files at startup! |
| **supertest** | HTTP testing without starting server | **httpx.AsyncClient(transport=ASGITransport(app))** or **requests-testing** | `pip install httpx` | Like supertest testing Express endpoints; FastAPI's test client is built in! `TestClient(app).get("/route")` — no server needed, just direct app calls! |
| **@testing-library/react** | Component testing library | **pytest + playwright** or **pytest-bdd** for integration testing | `pip install playwright pytest-playwright` | Like React Testing Library; use Playwright for browser-level component tests. For unit testing components directly, use the same assert pattern as in regular Python tests! |
| **eslint-plugin-import** | Import organization/linting | **ruff's I rules (isort)** or **flake8-import-order** | Built into ruff! | One rule: `select = ["I"]`. Like eslint-plugin-import automatically organizing your imports alphabetically by category. Much faster in ruff! |
| **eslint-plugin-node** | Node.js-specific linting rules | **pylint** or **flake8** with Python equivalent checks | Built into ruff! | Check for things like deprecated APIs, missing error handling, etc. Like the eslint-plugin-node rules that catch Node.js-specific anti-patterns! |
| **tsup/vite-node** | TypeScript runner without full build | Python runs .py files directly! No build step ever needed! | Built-in! | This is why Python doesn't need ts-node — you just `python script.py` and it works! Like how Node.js can run JS directly but TypeScript requires tsc first! |
| **pm2/forever** | Process management / keep-alive | **gunicorn**, **systemd**, or **supervisord** | `pip install gunicorn` | PM2 keeps your app alive; gunicorn does the same for web apps. systemd/supervisord do it for any Python process. Like having a built-in process manager! |
| **cross-env** | Cross-platform environment variable setting | **python-dotenv** + conditional Python code (or Makefile/just) | `pip install python-dotenv` | cross-env sets env vars differently per OS; Python's os.environ is already cross-platform! No separate package needed! |
| **concurrently** | Run multiple commands in parallel | **make** or **just** or **parallel** shell command | Built-in! `make -j4` for 4 parallel tasks! | Like concurrently running multiple npm scripts simultaneously; use Makefile with parallel targets or just CLI tool. Same concept, different tooling! |
| **rimraf** | Delete files/directories safely | **shutil.rmtree()** (stdlib!) | Built-in! `shutil.rmtree("dir")` | Like rimraf "node_modules" — Python's shutil removes directories recursively! Both handle the edge case of locked files gracefully! |
| **tsx** | TypeScript executor | **Python executes .py directly!** No tsx equivalent needed! | N/A | The entire concept of a "TypeScript executor" is unique to compiled languages. Python doesn't need it because source code IS executable! |

---

## 21. Migration Patterns: Express → FastAPI Complete Guide

### Step-by-Step Migration: Express.js TypeScript App → FastAPI Python App

```typescript
// ========================================
// BEFORE: Express.js + TypeScript Application
// ========================================
import express from "express";
import cors from "cors";
import helmet from "helmet";
import { z } from "zod";

const app = express();
app.use(cors());
app.use(helmet());
app.use(express.json());

// === Route: GET /api/users ===
const users: Array<{ id: string; name: string; email: string }> = [];

app.get("/api/users", (req, res) => {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 20;
    const search = req.query.search as string | undefined;

    let filtered = users;
    if (search) {
        filtered = users.filter(u => u.name.toLowerCase().includes(search.toLowerCase()));
    }

    const paginated = filtered.slice((page - 1) * limit, page * limit);
    res.json({ data: paginated, total: filtered.length });
});

// === Route: POST /api/users (with Zod validation!) ===
const userSchema = z.object({
    name: z.string().min(1).max(100),
    email: z.string().email(),
    age: z.number().int().min(0).max(150)
});

app.post("/api/users", (req, res) => {
    const result = userSchema.safeParse(req.body);  // Zod validates the request body!
    if (!result.success) {
        return res.status(400).json({ errors: result.error.errors });
    }

    const newUser = { id: crypto.randomUUID(), ...result.data };
    users.push(newUser);
    res.status(201).json(newUser);
});

// === Middleware: Custom auth middleware ===
function authMiddleware(req, res, next) {
    const token = req.headers.authorization?.split(" ")[1];
    if (!token || !validateToken(token)) {
        return res.status(401).json({ error: "Unauthorized" });
    }
    next();
}

app.use("/api/admin", authMiddleware);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

```python
# ========================================
# AFTER: FastAPI + Python Application (Direct equivalent!)
# Every Express feature has a FastAPI equivalent!
# ========================================

from fastapi import FastAPI, Query, HTTPException, Depends, Header
from pydantic import BaseModel, EmailStr, field_validator  # Pydantic v2 — like Zod for Python!
from typing import Optional
import uuid

app = FastAPI()

# === Automatic CORS (like app.use(cors())!) ===
# No middleware needed for basic CORS — add it if you need cross-origin:
# from fastapi.middleware.cors import CORSMiddleware
# app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_credentials=True)

# === Route: GET /api/users (query params via function parameters!) ===
users: list[dict] = []

@app.get("/api/users")
async def get_users(
    page: int = Query(default=1),                    # Like req.query.page in Express!
    limit: int = Query(default=20),                   # With defaults — like || 1 in TypeScript!
    search: Optional[str] = Query(default=None)        # Optional query parameter (like string | undefined!)
):
    filtered = users
    if search:
        filtered = [u for u in users if search.lower() in u["name"].lower()]

    paginated = filtered[(page - 1) * limit : page * limit]
    return {"data": paginated, "total": len(filtered)}

# === Route: POST /api/users (Pydantic schema — like Zod validation!) ===
class UserCreate(BaseModel):                        # Like z.object({...}) in Zod!
    name: str                                       # Automatic string validation!
    email: EmailStr                                 # Automatic email format validation!
    age: int = field_validator("age")               # Like z.number().int().min(0).max(150)!

    @field_validator("name")
    def validate_name(cls, v):                       # Like z.string().min(1).max(100)!
        if len(v) < 1 or len(v) > 100:
            raise ValueError("Name must be 1-100 characters")
        return v

@app.post("/api/users", status_code=201)             # Like res.status(201).json() in Express!
async def create_user(user: UserCreate):              # Pydantic auto-validates the request body!
    new_user = {"id": str(uuid.uuid4()), **user.model_dump()}
    users.append(new_user)
    return new_user                                   # FastAPI auto-serializes to JSON!

# === Middleware: Auth dependency (like Express authMiddleware!) ===
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def verify_auth(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """Like Express's authMiddleware — checks the Bearer token!"""
    token = credentials.credentials
    if not validate_token(token):                      # Your validation logic here!
        raise HTTPException(status_code=401, detail="Unauthorized")
    return credentials                                # Pass user info downstream!

@app.get("/api/admin/items")                         # Like app.use('/api/admin', authMiddleware)!
async def get_admin_items(current_user = Depends(verify_auth)):  # Injects verified user!
    """Protected route — only accessible with valid Bearer token!"""
    return {"items": admin_data}

# === Automatic OpenAPI docs (like Swagger UI!) ===
# http://localhost:8000/docs → Interactive API documentation generated from type hints!
# No manual swagger configuration needed — FastAPI generates it automatically!

import uvicorn
uvicorn.run(app, host="0.0.0.0", port=8000)         # Like app.listen(PORT) in Express!
```

### Mermaid: Migration Flowchart — Express → FastAPI

```mermaid
flowchart TD
    subgraph EXPRESS_APP["Express.js + TypeScript Application"]
        direction TB
        E1["express() → Create app\napp.use(cors())\napp.use(helmet())\napp.use(express.json())"] --> E2["app.get('/api/users', (req, res) => {\n  const page = parseInt(req.query.page);\n  // Manual validation with Zod\n  const result = z.object({name: z.string()}).safeParse(req.body)\n})"]
        E3["Middleware chain:\napp.use(cors()) → app.use(helmet()) → app.use(authMW) → app.use(router)\n→ Each middleware runs in order!"] --> E4["app.listen(3000, cb)\n→ Server starts on port 3000!\nManual Swagger config for OpenAPI docs!"]
    end

    subgraph FASTAPP_APP["FastAPI + Python Application (Direct Equivalent!)"]
        direction TB
        F1["FastAPI() → Create app\nNo middleware needed for basic CORS — built-in support!"] --> F2["@app.get('/api/users')\nasync def get_users(page: int = 1):\n  # Query params are function parameters!\n  # Pydantic validates request body automatically!\n  return {'data': users, 'total': len(users)}"]
        F3["FastAPI dependencies:\nDepends(verify_auth) → Like Express middleware!\nEach dependency is a composable function injected into the route!"] --> F4["@app.get('/api/admin')\n@Depends(auth)\ndef admin_route(): ... → Protected route!\nAuto-generated Swagger docs at /docs!\nuvicorn.run(app, port=8000) → Like app.listen(3000)!"]
    end

    E2 -.->|"Express routes with Zod validation\n→ FastAPI routes with Pydantic models!\nBoth validate request bodies before route handlers run!"| F2
```

---

## 22. lodash Utilities → Python functools/operator/itertools Mapping

### Every Common Lodash Function Mapped to Python Standard Library

| lodash function | Purpose | Python Equivalent | Example |
|----------------|---------|-----------------|---------|
| `_.map(arr, fn)` | Transform array elements | List comprehension `[fn(x) for x in arr]` or `map(fn, arr)` | `list(map(str, [1,2,3]))` → `['1','2','3']` |
| `_.filter(arr, fn)` | Filter array elements | List comprehension `[x for x in arr if fn(x)]` or `filter(fn, arr)` | `[x for x in range(10) if x % 2 == 0]` → `[0,2,4,6,8]` |
| `_.reduce(arr, fn, init)` | Fold array to single value | `functools.reduce(fn, arr, init)` | `functools.reduce(lambda a,b: a+b, [1,2,3], 0)` → `6` |
| `_.forEach(arr, fn)` | Iterate without returning | `for x in arr: fn(x)` | Python for-loop with side effects! |
| `_.find(arr, fn)` | Find first matching element | `next((x for x in arr if fn(x)), None)` | `next((u for u in users if u.name == "Alice"), None)` |
| `_.groupBy(arr, fn)` | Group array by key | `collections.defaultdict(list)` | `{k: list(g) for k, g in groupby(sorted(arr, key=fn), key=fn)}` |
| `_.chunk(arr, size)` | Split array into chunks | `[arr[i:i+size] for i in range(0, len(arr), size)]` | Chunk a list into groups of 10! |
| `_.flatten(arr)` | Flatten nested arrays | `itertools.chain.from_iterable(arr)` or custom recursive flatten | Like lodash's flattenDeep but via itertools! |
| `_.uniq(arr)` | Remove duplicates | `list(dict.fromkeys(arr))` (preserves order!) or `list(set(arr))` | Both remove duplicates; dict preserves insertion order! |
| `_.union(a, b)` | Union of two arrays | `list(dict.fromkeys(a + b))` or `set(a) \| set(b)` | Union of two lists — one expression! |
| `_.intersection(a, b)` | Common elements between arrays | `list(set(a) & set(b))` | Intersection of sets! Like lodash's _.intersection() |
| `_.difference(a, b)` | Elements in a but not b | `[x for x in a if x not in b_set]` (use set for O(1) lookup!) | Like lodash's _.difference() — elements exclusive to array a! |
| `_.sortBy(arr, fn)` | Sort by key function | `sorted(arr, key=fn)` | Built-in sorted with key parameter! |
| `_.isEmpty(obj)` | Check if empty | `not obj` (for list/dict/set/str) or `len(obj) == 0` | Python's falsy check is like _.isEmpty() for collections! |
| `_.get(obj, path, default)` | Safely get nested value | `obj.get("key", {}).get("nested", default)` or `pydash.get()` | Like lodash's _.get but with dict's built-in .get() method! |
| `_.merge(a, b)` | Deep merge objects | **deepmerge** package (`pip install deepmerge`) or custom function | pydash.merge() provides the exact same functionality as lodash's _.merge! |
| `_.cloneDeep(obj)` | Deep clone an object | `copy.deepcopy(obj)` (stdlib!) | Like lodash's cloneDeep — creates a complete independent copy! |
| `_.pick(obj, keys)` | Select specific keys from object | `{k: obj[k] for k in keys if k in obj}` or `pydash.pick()` | Like _.pick() — extract specific properties from an object! |
| `_.omit(obj, keys)` | Remove specific keys | `{k: v for k, v in obj.items() if k not in omit_keys}` or `pydash.omit()` | Like _.omit() — create a new dict without specified keys! |

---

## 23. Zod Validation → Pydantic v2 Mapping (Complete Deep Dive)

### Every Zod Type Mapped to Pydantic v2

| Zod type/constraint | TypeScript Example | Pydantic v2 Equivalent | Python Code |
|--------------------|--------------------|---------------------|-------------|
| **string** | `z.string()` | `str` | `name: str` |
| **string().min(n)** | `z.string().min(3)` | `constr(min_length=n)` | `name: constr(min_length=3)` |
| **string().max(n)** | `z.string().max(100)` | `constr(max_length=n)` | `name: constr(max_length=100)` |
| **string().email()** | `z.string().email()` | `EmailStr` (from pydantic) | `email: EmailStr` |
| **number()** | `z.number()` | `float` or `int` | `age: int` |
| **number().min(n)** | `z.number().min(0)` | `field_validator` + range check | `@field_validator("age")` |
| **number().int()** | `z.number().int()` | `int` type annotation! | Like any integer field! |
| **boolean()** | `z.boolean()` | `bool` | `is_active: bool` |
| **array(T)** | `z.array(z.string())` | `list[T]` (PEP 604!) | `tags: list[str]` |
| **object({})** | `z.object({name: z.string()})` | `BaseModel` subclass | Like defining a schema class! |
| **union(A, B)** | `z.union([z.string(), z.number()])` | `str \| int` (PEP 604 union syntax!) | Same as type hints! |
| **nullable(T)** | `z.string().nullable()` | `str \| None` or `Optional[str]` | Optional types in Python! |
| **optional()** | `z.string().optional()` | `str | None = None` | Default value of None makes it optional! |
| **enum([A, B])** | `z.enum(["active", "inactive"])` | `Literal["active", "inactive"]` or `Enum` class | Literal types restrict to specific values! |
| **date()** | `z.date()` | `date` (from datetime) | `created_at: date` |
| **datetime()** | `z.datetime()` | `datetime` (from datetime) | `updated_at: datetime` |
| **uuid()** | `z.uuid4()` | `UUID` (from uuid) | `id: UUID = Field(default_factory=uuid4)` |
| **instanceof(Class)** | `z.instanceof(UserModel)` | Pydantic model validation built in! | Just use the type hint — Pydantic validates automatically! |

### Zod → Pydantic Complete Example

```typescript
// TypeScript: Zod schema for user validation!
import { z } from "zod";

const UserSchema = z.object({
    id: z.string().uuid(),                              // UUID format validation!
    name: z.string().min(1).max(100),                   // String with length constraints!
    email: z.string().email(),                           // Email format validation!
    age: z.number().int().min(0).max(150),              // Integer within range!
    role: z.enum(["admin", "user", "moderator"]),       // Enum: only these values allowed!
    tags: z.array(z.string()).optional(),               // Optional list of strings!
    createdAt: z.coerce.date(),                          // Parse string → date automatically!
    isActive: z.boolean().default(true),                // Boolean with default value!
});

// Validate and parse input (throws on validation error!)
const parsed = UserSchema.parse(inputData);
```

```python
# Python: Pydantic v2 equivalent — EXACTLY the same validation, via type hints!
from pydantic import BaseModel, EmailStr, field_validator, Field, ConfigDict
from typing import Optional
from datetime import date
from uuid import uuid4, UUID
from enum import Enum

class UserRole(str, Enum):                              # Like z.enum(["admin", "user"])!
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"

class UserCreate(BaseModel):                            # Like z.object({...}) in Zod!
    model_config = ConfigDict(populate_by_name=True)    # Pydantic v2 configuration!

    id: UUID = Field(default_factory=uuid4)             // UUID field — auto-generated!
    name: str = Field(min_length=1, max_length=100)     // String constraints (like .min/.max)!
    email: EmailStr                                     // Email validation — built-in type!
    age: int = Field(ge=0, le=150)                     // Integer with range (like .min/.max)!

    @field_validator("age")                             // Custom validators (like Zod's .refine())!
    def validate_age(cls, v):
        if v < 18:
            raise ValueError("Must be at least 18 years old")
        return v

    role: UserRole = UserRole.USER                        // Enum — only specific values allowed!
    tags: Optional[list[str]] = None                    // Optional list (like z.array().optional())!
    created_at: date = Field(default_factory=date.today) // Default value (like z.default())!
    is_active: bool = True                              // Boolean with default!

# Validate and parse input (raises ValidationError on errors!)
user = UserCreate(**input_data)                           // Like Zod's schema.parse()!
```

### Key Notes: Zod → Pydantic Mapping

1. **Pydantic v2's validators ARE your type hints** — no separate schema definition needed! `name: str` with `Field(min_length=1, max_length=100)` IS the validation AND the type in one place. Unlike TypeScript where you need both a type (`name: string`) and a Zod schema (`z.string().min(1).max(100)`)!
2. **Pydantic's Enum class is like Zod's z.enum()** — inherits from `str` so it serializes to strings (like Zod enum), but provides autocomplete and compile-time checking in your code!
3. **Pydantic v2's ConfigDict is like Zod's .catch() or .default()** — it controls validation behavior: allow extra fields, populate by name vs alias, etc. Much more flexible than Zod's one-size-fits-all approach!

---

## 24. Package Ecosystems: npm Registry → PyPI Complete Comparison

### Every Package Ecosystem Feature Mapped

| npm Feature | PyPI Equivalent | Details |
|------------|-----------------|---------|
| **Package search** | pypi.org/search | Both have web-based package discovery with download counts, ratings, and metadata! |
| **Version numbering** | SemVer (both!) | npm: `^4.18.2` means >=4.18.2 <5.0.0. PyPI/Poetry: same ^ syntax! Python convention prefers exact pins (`==`). |
| **Download packages** | `npm install pkg` → `pip install pkg` / `poetry add pkg` | Same pattern: name the package and it gets installed from the registry! |
| **Package publishing** | `npm publish` → `twine upload dist/*` or `poetry publish` | npm has a single command; Poetry also has `poetry publish` which does everything automatically! |
| **Workspace support** | `package.json` `"workspaces"` → pdm/uv native workspaces (Poetry needs plugin!) | PDM and uv have the best workspace support in Python, like pnpm's workspaces for TypeScript! |
| **Dependency overrides** | npm: `"overrides"` in package.json → Poetry/pdm: `[tool.pdm.resolution-overrides]` | Both ecosystems support overriding transitive dependencies to resolve conflicts! |
| **Private registries** | npm private orgs / Verdaccio → PyPI private indexes (e.g., Artifactory, devpi) | Both support private/internal package registries with authentication! |
| **Pre/Post hooks** | npm: preinstall/postinstall scripts → Poetry: no equivalent; use `poetry-plugin-post-install` | npm has more script hook flexibility; Python relies on post-install scripts in pyproject.toml instead! |

### Mermaid: Package Ecosystem Comparison — npm vs PyPI

```mermaid
flowchart TD
    subgraph NPM_ECO_SYSTEM["npm Package Ecosystem Features"]
        direction TB
        NM1["~2 million packages → Largest ecosystem for web/front-end development!\nReact, Vue, Angular, TypeScript tooling ALL available!"] --> NM2["package.json + package-lock.json\n→ Dependency declaration + deterministic lock file (auto-generated!)"]
        NM3["npm install/unpublish/publish/search\n→ All operations in one CLI tool! Like having a Swiss Army knife for packages!"] --> NM4["Workspaces: monorepo support built into npm/pnpm/yarn!\nOne command installs deps for ALL workspace packages simultaneously!"]
    end

    subgraph PYPI_ECO_SYSTEM["PyPI Package Ecosystem Features"]
        direction TB
        PM1["~500k+ packages → Dominates data science, ML, scientific computing!\nNumPy, pandas, scikit-learn, TensorFlow, PyTorch — ALL here! Smaller but more specialized!"] --> PM2["pyproject.toml + poetry.lock/uv.lock\n→ Dependency declaration (TOML format) + deterministic lock file (via Poetry/pdm/uv)!"]
        PM3["pip install/unpublish or poetry publish/search → Different tools for different operations!\npip handles install; twine handles publish. More fragmented tooling than npm!"] --> PM4["Workspaces: pdm and uv have native workspace support!\nPoetry needs a third-party plugin. Growing ecosystem with workspaces being added to all major tools!"]
    end

    NM2 -.->|"Both use package registries + lock files:\ntypeScript uses package.json + package-lock.json\nPython uses pyproject.toml + poetry.lock/uv.lock!"| PM2
```

### Key Notes: Package Ecosystems (npm vs PyPI)

1. **npm has more packages (~2M vs ~500k) but PyPI is more specialized** — npm dominates web/front-end development; Python dominates data science, ML, and scientific computing. Like comparing a general-purpose store vs a specialty shop with deeper expertise!
2. **Poetry/pdm/uv have caught up to npm's feature set** — dependency resolution, lock files, workspace support, and publishing all work well now. The biggest gap remains: npm generates package-lock.json automatically; Python needs explicit tooling (Poetry/pdm/uv) for the same guarantee.
3. **Python's version pinning convention is stricter** — while Poetry supports ^ and ~ syntax like npm, most Python developers pin exact versions (`==`) in production to prevent "works on my machine" bugs caused by dependency drift.

---

## 25. Performance: Node.js Event Loop vs asyncio Event Loop (Benchmarks + Deep Analysis)

### Architecture Comparison: Single-Threaded Event Loops in Both Ecosystems

```
Node.js Event Loop Architecture:
┌─────────────────────────────────────┐
│         V8 JavaScript Engine         │
│  (single-threaded, non-blocking I/O) │
│                                      │
│  ┌──────────┐ ┌──────────┐          │
│  │ libuv    │ │ Timers   │          │
│  │ Poll     │ │ Check    │          │
│  │ Close    │ │ DNS      │          │
│  └──────────┘ └──────────┘          │
│                                      │
│  All I/O is asynchronous by default! │
│  No callbacks = no blocking!         │
└─────────────────────────────────────┘

Python asyncio Event Loop Architecture:
┌─────────────────────────────────────┐
│        CPython + GIL                 │
│  (single-threaded, non-blocking I/O) │
│                                      │
│  ┌──────────┐ ┌──────────┐          │
│  │ uvloop   │ │ Selectors│          │
│  │ (fast!)  │ │ EPOLL    │          │
│  └──────────┘ └──────────┘          │
│                                      │
│  I/O must be explicitly async        │
│  (await/async keyword)!              │
│  Synchronous I/O BLOCKS the loop!    │
└─────────────────────────────────────┘
```

### Benchmark Comparison: HTTP Server Throughput

| Metric | Node.js (Express) | Python (FastAPI + uvicorn) | Notes |
|--------|------------------|---------------------------|-------|
| **Simple GET endpoint** | ~15,000 req/s | ~45,000 req/s (uvloop!) | FastAPI with uvloop is 3x faster! |
| **JSON parse + respond** | ~12,000 req/s | ~35,000 req/s | Python's built-in JSON parsing is highly optimized! |
| **Database query (PostgreSQL)** | ~3,500 req/s (node-postgres) | ~4,200 req/s (asyncpg!) | asyncpg is the fastest PostgreSQL driver in ANY language! |
| **Streaming file response** | ~8,000 req/s | ~15,000 req/s | FastAPI's streaming response + uvloop is very efficient! |
| **WebSocket connections** | ~50,000 concurrent | ~40,000 concurrent (uvicorn-websockets) | Node.js wins slightly for pure WebSocket handling! |

### Benchmark Comparison: HTTP Client Request Latency

| Operation | Node.js (axios) | Python (httpx) | Notes |
|-----------|----------------|---------------|-------|
| **Single HTTP GET** | ~2-5ms | ~3-8ms | Comparable — both highly optimized! |
| **Concurrent 100 requests** | ~50-100ms (native async!) | ~45-90ms (httpx + asyncio!) | Both handle concurrency well with native async! |
| **Connection reuse** | Built-in (http.Agent) | httpx.ConnectionPool (built in!) | Both maintain connection pools automatically! |

### Key Performance Notes: Event Loop Comparison

1. **Both Node.js and Python asyncio are single-threaded event loops** — they handle thousands of concurrent connections without threads. The difference is that Node.js's I/O is async by default; Python requires explicit `async/await` syntax.
2. **uvloop makes Python's asyncio 5-10x faster** — uvloop replaces Python's default asyncio selector with libuv (the same library Node.js uses!). With uvloop, FastAPI can match or exceed Express performance! Like how Python got a "Turbo mode" to match Node.js's performance.
3. **asyncpg is the fastest PostgreSQL driver in any language** — benchmarks consistently show asyncpg outperforming node-postgres for raw query throughput on PostgreSQL.

---

## 26. Complete Migration Cheatsheet

### Quick Reference: TypeScript/Node.js → Python One-Liners

| TypeScript/Node.js | Python Equivalent |
|--------------------|-----------------|
| `npm init -y` | `poetry new my-app` or `uv venv .venv && pip install ...` |
| `npm install pkg` | `pip install pkg` or `poetry add pkg` |
| `npm run build` | `python script.py` (no build step needed!) |
| `tsc --noEmit` | `mypy src/` (type-check without compiling) |
| `eslint . --fix` | `ruff check --fix .` |
| `prettier --write .` | `ruff format .` or `black .` |
| `npm test` | `pytest tests/` |
| `npm audit` | `pip-audit` |
| `node app.js` | `uvicorn main:app --reload` (FastAPI) or `python app.py` |
| `export PORT=3000` | `os.environ.get("PORT", "3000")` or just use env vars directly! |
| `process.cwd()` | `Path.cwd()` |
| `path.join(a, b)` | `Path(a) / b` |
| `fs.readFile(f, 'utf8')` | `Path(f).read_text(encoding='utf-8')` |
| `JSON.parse(str)` | `json.loads(str)` |
| `JSON.stringify(obj)` | `json.dumps(obj)` |
| `console.log(msg)` | `print(msg)` or `logging.info(msg)` |
| `setTimeout(cb, 1000)` | `await asyncio.sleep(1)` |
| `require('./module')` | `from . import module` |

---

## 27. Quizzes with Answers (25+)

### Quiz 1: Core Module Mapping
**Q:** Which Python module provides the closest equivalent to Node.js's `path` module?
<details><summary>Show Answer</summary>
`pathlib.Path`. Unlike Node.js's procedural `path.join()`, `path.extname()` functions, pathlib uses object-oriented access: `Path("a") / "b" / "c"` and properties like `.suffix`, `.stem`, `.parent`. It also supports `/` operator for path joining which is cleaner than multiple function calls.
</details>

### Quiz 2: HTTP Networking
**Q:** Why does Python need a third-party package for HTTP while Node.js has it built-in?
<details><summary>Show Answer</summary>
Node.js was designed from the start as a networking-first runtime, so HTTP is in its standard library. Python's standard library focuses on data processing (csv, json, decimal, etc.). For HTTP, Python developers use `requests` (sync), `httpx` (async), or `aiohttp` (asyncio). Like TypeScript developers using axios for a cleaner API than the built-in http module, Python devs just pip install requests once and get excellent functionality.
</details>

### Quiz 3: File System
**Q:** What is the Python equivalent of Node.js's `fs.promises.readFile()`?
<details><summary>Show Answer</summary>
For synchronous: `Path(file).read_text(encoding="utf-8")`. For asynchronous: use the `aiofiles` package. The synchronous version is one call vs Node.js which also needs JSON.parse() as a second step, making Python's approach more concise for JSON files specifically.
</details>

### Quiz 4: Streaming
**Q:** How do you stream a large file line-by-line in Python without loading it into memory?
<details><summary>Show Answer</summary>
`for line in open("large-file.txt"): ...` — the `for` loop on a file object IS the streaming mechanism! It reads one line at a time through an iterator, so memory usage stays constant regardless of file size. Like Node.js's `createReadStream().on('data', cb)` but with zero boilerplate!
</details>

### Quiz 5: Child Process
**Q:** What is the Python equivalent of `child_process.exec('git log -10')`?
<details><summary>Show Answer</summary>
`subprocess.run(["git", "log", "-10"], capture_output=True, text=True)` — returns a Result object with `.stdout`, `.stderr`, and `.returncode`. Unlike Node.js's callback pattern, Python's structured return value is more type-safe and easier to work with programmatically.
</details>

### Quiz 6: DNS
**Q:** What Python package provides advanced DNS queries like MX, TXT, NS records?
<details><summary>Show Answer</summary>
`dnspython` (installed via `pip install dnspython`). Basic A/AAAA lookups work via the built-in `socket.getaddrinfo()`, but for advanced record types (MX, TXT, NS, CNAME, PTR), you need the external `dnspython` package. Like TypeScript's node:dns but as a third-party dependency in Python.
</details>

### Quiz 7: UDP Sockets
**Q:** How do you create a UDP server in Python?
<details><summary>Show Answer</summary>
Use the `socket` module with `SOCK_DGRAM`: `s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)`, then `s.bind(("0.0.0.0", port))`. Unlike Node.js's `dgram.createSocket("udp4")`, Python exposes raw sockets directly — no Stream wrapper needed. Both follow the same pattern: create socket → bind to port → recvfrom/sendto.
</details>

### Quiz 8: URL Parsing
**Q:** What is the cleanest way to parse and modify URL query parameters in Python?
<details><summary>Show Answer</summary>
Use `yarl.URL` (pip install yarl): `url = URL("https://api.com/users?page=2"); url = url.update_query(page="3", limit="10")`. Like Node.js's `new URL()` with `searchParams.set()`, but even cleaner — chainable methods and property access make it feel natural. For stdlib, use `urllib.parse.urlparse()` + `urlencode()`.
</details>

### Quiz 9: Compression
**Q:** Which Python modules provide the equivalent of Node.js's `zlib.gzip()`?
<details><summary>Show Answer</summary>
`gzip.open("file.gz", "wb")` for compressed files, or `zlib.compress(data)` for raw deflate compression. Python has ALL compression formats built-in: zlib, gzip, bz2, lzma (xz), and zstandard (third-party). Node.js only has zlib via stdlib; brotli/lz4 require pako package.
</details>

### Quiz 10: assert/Testing
**Q:** What is the Python equivalent of `assert.throws(() => { throw new Error("msg") }, Error)`?
<details><summary>Show Answer</summary>
```python
import pytest
def test_error():
    with pytest.raises(Error, match="msg"):
        raise Error("msg")
```
pytest.raises() catches exceptions during test execution and verifies both the type and optionally the message content. More powerful than Node.js's assert.throws because it supports regex matching on error messages!
</details>

### Quiz 11: EventEmitter
**Q:** How do you implement a pub/sub pattern in Python like Node.js's EventEmitter?
<details><summary>Show Answer</summary>
For async: use `asyncio.Queue` — producers put() events, consumers await get(). For sync: use the `blinker` package (`pip install blinker`) with signals. Neither is built into Python's stdlib, which is the gap — Node.js has EventEmitter built-in; Python requires either asyncio patterns or a third-party package for synchronous pub/sub.
</details>

### Quiz 12: Buffer/Binary
**Q:** How do you create and manipulate binary data in Python (like Node.js's Buffer)?
<details><summary>Show Answer</summary>
`bytes` (immutable): `b"hello"` or `bytes(16)` for zeroed buffer. `bytearray` (mutable): `bytearray(16)`. For packing/unpacking numbers: `struct.pack(">I", value)`. For memory-mapped files: `mmap.mmap()`. Python has native bytes/bytearray types built into the language — no separate Buffer class needed!
</details>

### Quiz 13: timers
**Q:** What is the Python equivalent of `setTimeout(cb, 5000)`?
<details><summary>Show Answer</summary>
Async version: `await asyncio.sleep(5)`. Sync (blocking) version: `time.sleep(5)`. Unlike Node.js's setTimeout which is async by default in the event loop context, Python gives you a choice: asyncio for non-blocking or time for blocking. Like having both setInterval and process.nextTick as separate concepts!
</details>

### Quiz 14: util functions
**Q:** What Python equivalent replaces Node.js's `util.deepEqual(a, b)`?
<details><summary>Show Answer</summary>
Python's built-in `==` operator does deep equality by default for dicts and lists! No separate utility needed. `{"a": [1,2]} == {"a": [1,2]}` returns True — same as lodash's _.isEqual() or Node.js's deepStrictEqual(). Python doesn't need a separate deepEqual function because the language handles it natively!
</details>

### Quiz 15: Module Resolution
**Q:** How does Python handle module caching? (Like `require.cache` in Node.js)
<details><summary>Show Answer</summary>
Python uses `sys.modules` — a dictionary mapping module names to loaded module objects. Like Node.js's require.cache, this prevents duplicate loading. Importing the same module twice returns the exact same object reference. Access with `sys.modules["module_name"]`. Both ecosystems use this pattern for circular import handling!
</details>

### Quiz 16: HTTP Migration
**Q:** How do you migrate an Express CORS middleware to FastAPI?
<details><summary>Show Answer</summary>
In Express: `app.use(cors())`. In FastAPI: `from fastapi.middleware.cors import CORSMiddleware; app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_credentials=True)`. Both set up CORS headers automatically on every response. Like how the middleware pattern translates directly between frameworks!
</details>

### Quiz 17: Zod → Pydantic
**Q:** What is the Pydantic v2 equivalent of `z.string().email().min(5).max(100)`?
<details><summary>Show Answer</summary>
`from pydantic import EmailStr, constr; email: constr(min_length=5, max_length=100) = EmailStr()`. Actually in Pydantic v2 you use `field_validator`: `@field_validator("email")` for custom validation, or just `email: EmailStr` with `constr()` for min/max constraints. Pydantic's type hints DO the validation — no separate schema object needed!
</details>

### Quiz 18: Package Ecosystem
**Q:** What is the PyPI equivalent of npm's scope (`@scope/package-name`)?
<details><summary>Show Answer</summary>
There IS NO equivalent in Python. Package names must be globally unique on PyPI. This is why package naming is more competitive and slower on PyPI — you can't namespace packages under your organization like TypeScript developers use @scope to avoid conflicts. Python relies on PEP 503 normalization (underscores/hyphens treated as equivalent) rather than scoping.
</details>

### Quiz 19: Performance
**Q:** How can you make Python's asyncio event loop match Node.js's performance?
<details><summary>Show Answer</summary>
Install and enable `uvloop` (pip install uvloop): `import uvloop; uvloop.install()`. This replaces Python's default asyncio selector with libuv — the same event loop library that Node.js uses! With uvloop, FastAPI benchmarks often exceed Express performance for equivalent workloads because asyncpg outperforms node-postgres and uvloop provides the native I/O layer.
</details>

### Quiz 20: Compression
**Q:** What Python package provides brotli compression (like pako in Node.js)?
<details><summary>Show Answer</summary>
`pip install brotli`. Like how pako provides brotli/lz4 in Node.js but zlib is only deflate/gzip; Python's stdlib has gzip/zlib/bz2/lzma, and brotli via a third-party package. The zstandard package (`pip install zstandard`) is also popular and often outperforms brotli in compression ratio and speed!
</details>

### Quiz 21: Logging
**Q:** How does Python's logging module compare to Node.js's winston/pino for structured logging?
<details><summary>Show Answer</summary>
Python's built-in `logging` module supports levels (DEBUG/INFO/WARNING/ERROR), formatters, handlers (console/file/network), and filters — like Winston out of the box. For JSON-structured logging (like pino's fast JSON output), use the `structlog` package which provides production-grade structured logging with 50+ context processors, like having Pino built into Python!
</details>

### Quiz 22: Module Resolution
**Q:** How do you implement a Node.js-style dynamic require in Python?
<details><summary>Show Answer</summary>
Use `importlib.import_module("module.path")`. This is the closest equivalent to `require('dynamic-module')`: `mod = importlib.import_module(package_path); result = mod.some_function()`. Like TypeScript's dynamic imports (`import('./module')`) but synchronous. For async dynamic loading, wrap in asyncio.to_thread().
</details>

### Quiz 23: Querystring
**Q:** What is the Python equivalent of `URLSearchParams` for manipulating query strings?
<details><summary>Show Answer</summary>
```python
from urllib.parse import urlencode, parse_qs
params = {"page": "1", "limit": "20"}
url_string = urlencode(params)  # "page=1&limit=20"
parsed = parse_qs(url_string)   # {"page": ["1"], "limit": ["20"]}
```
Or use `yarl.URL` for a cleaner object-oriented API: `url.update_query(page="2")`. Both work like Node.js's URLSearchParams.set/delete/get methods!
</details>

### Quiz 24: Signal Handling
**Q:** How do you handle SIGTERM/SIGINT in Python (like `process.on('SIGTERM', cb)`)?
<details><summary>Show Answer</summary>
```python
import signal, sys
def shutdown(signum, frame):
    print(f"Received {signum} — cleaning up...")
    sys.exit(0)

signal.signal(signal.SIGTERM, shutdown)  # Like process.on('SIGTERM', cb)!
signal.signal(signal.SIGINT, shutdown)   # Like process.on('SIGINT', cb)!
```
Signal handling in Python is almost identical to Node.js — register a callback with signal.signal() and it fires when the OS sends that signal!
</details>

### Quiz 25: Worker/Thread
**Q:** How do you parallelize CPU-bound work in Python (like Node.js's worker_threads)?
<details><summary>Show Answer</summary>
Use `concurrent.futures.ProcessPoolExecutor` for true parallelism (bypasses GIL):
```python
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor() as executor:
    results = list(executor.map(cpu_intensive_func, data_chunks))
```
Unlike Node.js's worker_threads which creates separate V8 instances, Python uses separate processes with the multiprocessing module. The GIL prevents true parallelism in threading for CPU-bound work, which is why ProcessPoolExecutor (separate processes) is needed!
</details>

---

## 28. Exercises with Solutions (15+)

### Exercise 1: Complete File Manager with pathlib
**Task:** Create a file manager that handles every Node.js fs operation using Python's pathlib.

<details><summary>Solution</summary>

```python
from pathlib import Path
import json, shutil, os

def file_manager(root_dir: str):
    """Complete file management operations — replaces EVERY Node.js fs method!"""
    base = Path(root_dir)

    # === Read JSON (replaces fs.promises.readFile + JSON.parse) ===
    config = json.loads((base / "config.json").read_text())

    # === Write JSON (replaces fs.promises.writeFile with JSON.stringify) ===
    (base / "output.json").write_text(json.dumps({"status": "ok"}, indent=2))

    # === List directory with stats (replaces readdir + stat) ===
    for item in base.iterdir():
        stat = item.stat()
        print(f"{item.name}: {stat.st_size} bytes, {'dir' if item.is_dir() else 'file'}")

    # === Recursive glob (replaces glob library) ===
    py_files = list(base.rglob("*.py"))  # All .py files recursively!

    # === Create/destroy directories (replaces mkdir/rm) ===
    (base / "new" / "nested" / "dir").mkdir(parents=True, exist_ok=True)
    shutil.rmtree(base / "temp", ignore_errors=True)

    # === Copy/move/rename (replaces fs.copyFile + mv) ===
    shutil.copy2(base / "source.txt", base / "dest.txt")  # copy with metadata!
    shutil.move(base / "old/", base / "new-location/")     # move/rename!

    # === File existence checks ===
    if (base / "data.txt").exists(): ...        # isFile?
    if base.is_dir(): ...                         # isDir?

    # === Read line-by-line streaming (replaces createReadStream) ===
    count = 0
    with open(base / "large.csv") as f:
        for line in f:          # Streams line-by-line! Zero memory overhead!
            count += 1
    print(f"Total lines: {count}")
```

</details>

### Exercise 2: Complete HTTP Client with requests/httpx
**Task:** Create an HTTP client that handles every common HTTP operation.

<details><summary>Solution</summary>

```python
import requests
import httpx                        # pip install httpx — async!
from typing import Optional

def sync_api_client(base_url: str, token: Optional[str] = None) -> dict:
    """Complete synchronous HTTP client replacing axios/node-fetch/http module!"""
    headers = {}
    if token:
        headers["Authorization"] = f"Bearer {token}"

    # GET with query params (one-liner!)
    response = requests.get(
        f"{base_url}/users",
        params={"page": 1, "limit": 20},   # Query parameters!
        headers=headers,
        timeout=10                          # Timeout in seconds!
    )
    response.raise_for_status()              # Raise on 4xx/5xx (like checking statusCode)
    return response.json()                   # Auto-parsed JSON!

    # POST with JSON body
    # response = requests.post(f"{base_url}/users", json={"name": "Alice"})

    # File upload
    # with open("photo.jpg", "rb") as f:
    #     requests.post(f"{base_url}/upload", files={"file": f})

    # Session with cookie persistence
    # session = requests.Session()
    # session.headers.update({"Authorization": f"Bearer {token}"})
    # session.get(f"{base_url}/profile")  # Uses same session!

def async_api_client(base_url: str) -> dict:
    """Async HTTP client (httpx) — like node-fetch with native async!"""
    async with httpx.AsyncClient(timeout=10.0) as client:
        response = await client.get(f"{base_url}/users")
        return response.json()
```

</details>

### Exercise 3: EventEmitter Replacement with asyncio.Queue
**Task:** Create a pub/sub event system replacing Node.js's EventEmitter.

<details><summary>Solution</summary>

```python
import asyncio

class EventBus:
    """Pub/sub event bus — like Node.js's EventEmitter!"""
    def __init__(self):
        self._queues = {}                        # Channel → asyncio.Queue
        self._running = False

    async def start(self):
        """Start the event dispatch loop (like starting the event loop!)"""
        self._running = True
        while self._running:
            tasks = []
            for channel, queue in self._queues.items():
                if not queue.empty():
                    msg = await queue.get()
                    tasks.append(self._dispatch(channel, msg))
            await asyncio.gather(*tasks) if tasks else asyncio.sleep(0.01)

    def on(self, channel: str, callback):
        """Subscribe to a channel (like emitter.on(event, cb))!"""
        if channel not in self._queues:
            self._queues[channel] = asyncio.Queue()
        self._queues[channel]._callback = callback  # Store the callback!

    def emit(self, channel: str, data):
        """Publish to a channel (like emitter.emit(event, data))!"""
        if channel in self._queues and not self._queues[channel].empty():
            asyncio.create_task(self._dispatch(channel, data))
        # Actually use queue.put for async!
```

</details>

### Exercise 4: DNS Resolution with dnspython
**Task:** Create a DNS query tool that handles all record types.

<details><summary>Solution</summary>

```python
import dns.resolver
import socket

def resolve_dns(hostname: str):
    """Complete DNS resolution tool — like Node.js's dns module!"""
    results = {}

    # A records
    a_records = dns.resolver.resolve(hostname, "A")
    results["A"] = [str(r) for r in a_records]

    # AAAA (IPv6)
    try:
        aaaa = dns.resolver.resolve(hostname, "AAAA")
        results["AAAA"] = [str(r) for r in aaaa]
    except dns.resolver.NoAnswer:
        results["AAAA"] = []

    # MX records
    try:
        mx = dns.resolver.resolve(hostname, "MX")
        results["MX"] = [(r.preference, str(r.exchange)) for r in mx]
    except dns.resolver.NoAnswer:
        results["MX"] = []

    # TXT records
    try:
        txt = dns.resolver.resolve(hostname, "TXT")
        results["TXT"] = [s.decode() for s in txt[0].strings]
    except dns.resolver.NoAnswer:
        results["TXT"] = []

    # NS records
    try:
        ns = dns.resolver.resolve(hostname, "NS")
        results["NS"] = [str(r.target) for r in ns]
    except dns.resolver.NoAnswer:
        results["NS"] = []

    # Reverse lookup
    try:
        hostname = socket.getfqdn(socket.gethostbyname(hostname))
        results["PTR"] = hostname
    except socket.gaierror:
        results["PTR"] = None

    return results
```

</details>

### Exercise 5: Compression Utility with All Formats
**Task:** Create a compression/decompression utility that supports all Python compression formats.

<details><summary>Solution</summary>

```python
import zlib, gzip, bz2, lzma
from typing import Union

def compress(data: bytes, format: str = "gzip") -> bytes:
    """Compress data in any format — like Node.js's zlib + pako!"""
    if format == "zlib":
        return zlib.compress(data)
    elif format == "bz2":
        return bz2.compress(data)
    elif format == "lzma":
        return lzma.compress(data)
    else:  # gzip (default)
        import io
        buf = io.BytesIO()
        with gzip.GzipFile(fileobj=buf, mode='wb') as f:
            f.write(data)
        return buf.getvalue()

def decompress(data: bytes, format: str = "gzip") -> bytes:
    """Decompress data — handles ALL formats Node.js can handle!"""
    if format == "zlib":
        return zlib.decompress(data)
    elif format == "bz2":
        return bz2.decompress(data)
    elif format == "lzma":
        return lzma.decompress(data)
    else:  # gzip (default)
        import io
        with gzip.GzipFile(fileobj=io.BytesIO(data), mode='rb') as f:
            return f.read()

# Like Node.js's zlib module but with MORE formats built-in!
```

</details>

### Exercise 6: UDP Server/Client
**Task:** Create a complete UDP server and client pair.

<details><summary>Solution</summary>

```python
import socket
import threading

def udp_server(port: int = 9999):
    """UDP server — like Node.js dgram.createSocket('udp4')!"""
    server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(("0.0.0.0", port))

    print(f"UDP Server listening on port {port}")

    while True:
        data, addr = server.recvfrom(4096)     # Receive message!
        print(f"From {addr}: {data.decode()}")

        # Respond (like server.send() in Node.js!)
        response = f"Pong! Received {len(data)} bytes".encode()
        server.sendto(response, addr)            # Send back to sender!

def udp_client(host: str = "127.0.0.1", port: int = 9999):
    """UDP client — like Node.js dgram.createSocket + send!"""
    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    message = b"Hello from Python UDP Client!"

    client.sendto(message, (host, port))       # Send to server!

    # Receive response
    data, _ = client.recvfrom(4096)
    print(f"Server responded: {data.decode()}")
    client.close()

# Start server in a thread (like Node.js's UDP server)!
server_thread = threading.Thread(target=udp_server, args=(9999,), daemon=True)
server_thread.start()
udp_client("127.0.0.1", 9999)                  # Send message and receive response!
```

</details>

### Exercise 7: URL Parser with yarl
**Task:** Create a URL manipulation tool with full Node.js URL class parity.

<details><summary>Solution</summary>

```python
from yarl import URL

class URLManager:
    """Complete URL parser/manipulator — like Node.js's new URL() + URLSearchParams!"""

    def __init__(self, url_string: str):
        self.url = URL(url_string)               # Like new URL(urlString) in Node.js!

    @property
    def protocol(self): return str(self.url.scheme) + ":"      # "https:"
    @property
    def hostname(self): return self.url.host                   # "api.example.com"
    @property
    def port(self): return self.url.port                       # 443 (or None!)
    @property
    def pathname(self): return str(self.url.path)              # "/api/users"
    @property
    def search(self): return f"?{self.url.query}"              # "?page=2&limit=10"
    @property
    def hash(self): return self.url.fragment                   # "" (empty if no hash!)

    def get_param(self, key: str) -> str | None:
        """Like url.searchParams.get() in Node.js!"""
        value = self.url.query.get(key)
        return value if value else None

    def set_param(self, key: str, value: str):
        """Like url.searchParams.set() — like modifying a hash map!"""
        self.url = self.url.with_query(**{**self.url.query, key: value})

    def delete_param(self, key: str):
        """Like url.searchParams.delete()!"""
        if key in self.url.query:
            new_query = {k: v for k, v in self.url.query.items() if k != key}
            self.url = self.url.with_query(**new_query)

    def append_param(self, key: str, value: str):
        """Like url.searchParams.append()!"""
        current = self.url.query.get_list(key, [])
        new_query = dict(self.url.query)
        new_query[key] = current + [value]  # Add to list!
        self.url = self.url.with_query(**new_query)

    def __str__(self): return str(self.url)     # Like url.toString()!

# Usage — exact same pattern as Node.js URL!
url_mgr = URLManager("https://api.example.com:8080/users?page=2&limit=10")
print(f"Protocol: {url_mgr.protocol}")          # "https:"
print(f"Host: {url_mgr.hostname}:{url_mgr.port}")  # "api.example.com:8080"
url_mgr.set_param("page", "3")                  # Modify query params!
url_mgr.delete_param("limit")                   # Remove a param!
print(str(url_mgr))                              # "https://api.example.com:8080/users?page=3"
```

</details>

### Exercise 8: Child Process Manager
**Task:** Create a process manager that handles every Node.js child_process pattern.

<details><summary>Solution</summary>

```python
import subprocess, asyncio

def run_command(cmd: list[str], timeout: int = 30) -> dict:
    """Run a shell command — like child_process.exec/execFile!"""
    result = subprocess.run(
        cmd, capture_output=True, text=True, timeout=timeout, check=False
    )
    return {
        "success": result.returncode == 0,
        "stdout": result.stdout.strip(),
        "stderr": result.stderr.strip(),
        "returncode": result.returncode
    }

async def stream_process(cmd: list[str]):
    """Stream process output line-by-line — like child_process.spawn!"""
    process = await asyncio.create_subprocess_exec(
        *cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE
    )

    # Stream stdout line by line (like Node.js's child.stdout.on('data')!)
    while True:
        line = await process.stdout.readline()
        if not line: break
        print(f"[STDOUT] {line.decode().strip()}")

    await process.wait()                         # Like child.on('close', cb)!
    return process.returncode

def fork_worker(function, args=()):
    """Fork a new Python process — like child_process.fork()!"""
    import multiprocessing
    def worker_target():
        result = function(*args)
        print(f"Worker result: {result}")

    worker = multiprocessing.Process(target=worker_target)
    worker.start()
    worker.join()                                # Wait for completion!
    return worker.exitcode

# Usage!
out = run_command(["git", "log", "--oneline", "-5"])  # Like exec('git log...')!
rc = await stream_process(["python", "-c", "print('hello')"])  # Spawn + stream!
```

</details>

### Exercise 12: Express → FastAPI Complete Migration
**Task:** Migrate the following Express app to FastAPI (every route, middleware, and validation pattern).

<details><summary>Solution</summary>

See the complete Express → FastAPI migration in Section 21 above. Key mappings:
- `express()` → `FastAPI()`
- `app.use(cors())` → CORSMiddleware
- `app.get('/route', handler)` → `@app.get('/route') async def handler():`
- `req.query.page` → `page: int = Query(default=1)`
- `z.object({}).safeParse(req.body)` → Pydantic model as function parameter
- `app.use(middleware)` → FastAPI dependencies with `Depends()`
- `app.listen(3000)` → `uvicorn.run(app, port=3000)`

</details>

### Exercise 13: Complete Stream Processing Pipeline
**Task:** Create a file processing pipeline that reads, transforms, and writes data using Python's streaming capabilities.

<details><summary>Solution</summary>

```python
from pathlib import Path
import json, gzip

def stream_processing_pipeline(input_path: str, output_path: str):
    """Complete streaming pipeline — like Node.js createReadStream + transform + createWriteStream!"""
    input_file = Path(input_path)
    output_file = Path(output_path)

    def transform_csv_to_json(csv_line: str) -> dict | None:
        """Transform each CSV row to a JSON object (like a Transform stream!)."""
        fields = csv_line.strip().split(",")
        if len(fields) < 2: return None  # Skip malformed rows!
        return {"id": int(fields[0]), "name": fields[1], "active": True}

    with input_file.open() as infile, output_file.open("w") as outfile:
        for i, line in enumerate(infile):     # Streaming read! Like Node's readable stream!
            if i == 0: continue               # Skip header row!
            record = transform_csv_to_json(line)  # Transform (like a Transform stream!)
            if record:
                json.dump(record, outfile)      # Write transformed output!
                outfile.write("\n")

# Alternative: use gzip for compressed input/output!
with gzip.open("input.csv.gz", "rt") as infile:     # Auto-decompress while reading!
    for line in infile:                               # Stream decompressed data!
        process_line(line)                            # Process each line!
```

</details>

### Exercise 14: Custom Linter Configuration
**Task:** Create a complete ruff configuration that enforces the same standards as TypeScript's ESLint + Prettier.

<details><summary>Solution</summary>

See Section 5 above for the complete ruff configuration. Key settings:
```toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "W", "F", "I", "N", "UP", "B", "SIM", "RUF", "C90", "T20", "S"]
ignore = ["E501"]  # Let the formatter handle line length!

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```
This configuration is like TypeScript's ESLint (all the rules) + Prettier (formatting) — but in ONE tool that runs 10-100x faster!

</details>

### Exercise 15: Complete CI/CD Pipeline
**Task:** Create a comprehensive GitHub Actions pipeline that covers all Python development stages.

<details><summary>Solution</summary>

See Section 9 above for the complete CI/CD pipeline. Key stages:
1. **Lint** (fastest — runs first, catches 80% of issues instantly)
2. **Type Check** (only if lint passes — mypy --strict)
3. **Test + Coverage** (pytest on multiple Python versions with coverage measurement)
4. **Build + Publish** (only on tags — build sdist/wheel, publish to PyPI via API token)

</details>

---

## Next

> **Next:** [Module 17 — Final Course Summary & TypeScript-to-Python Cheat Sheet](./17-final-summary.md)

---

*This module covers EVERY Node.js core module, every popular npm package, and complete migration patterns to Python. The key insight: TypeScript's entire stdlib + ecosystem maps to a combination of Python's standard library (pathlib, os, subprocess, hashlib, logging) + well-maintained third-party packages (requests, httpx, FastAPI, Pydantic, ruff, pytest). Every concept has been exhaustively documented with side-by-side code examples, making migration from TypeScript to Python as straightforward as following a reference table.*
