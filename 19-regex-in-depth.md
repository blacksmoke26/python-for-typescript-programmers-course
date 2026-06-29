# Module 19 — Regular Expressions In-Depth: Python `re` vs TypeScript RegExp

A complete, exhaustive reference to regular expressions in both Python and TypeScript — every metacharacter, flag, method, and edge case. Includes side-by-side comparisons, performance patterns, common pitfalls with fixes, references, quizzes, and key notes throughout. This module is designed as both a learning path (basic → advanced) and a daily reference for pattern matching.

## Table of Contents

- [1. Regex Landscape Overview](#1-regex-landscape-overview)
- [2. Python `re` Module — Complete API Reference](#2-python-re-module--complete-api-reference)
- [3. TypeScript `RegExp` — Complete API Reference](#3-typescript-regexp--complete-api-reference)
- [4. Metacharacters & Character Classes: Every Pattern Symbol](#4-metacharacters--character-classes-every-pattern-symbol)
- [5. Quantifiers, Groups & Assertions: The Building Blocks](#5-quantifiers-groups--assertions-the-building-blocks)
- [6. Python vs TypeScript Regex Feature Comparison Matrix](#6-python-vs-typescript-regex-feature-comparison-matrix)
- [7. Common Patterns Library: Ready-to-Use Recipes](#7-common-patterns-library-ready-to-use-recipes)
- [8. Advanced Regex Topics (Lookarounds, Branch Reset, Recursion)](#8-advanced-regex-topics-lookarounds-branch-reset-recursion)
- [9. Performance & Optimization](#9-performance--optimization)
- [10. Key Notes & Important Factors](#10-key-notes--important-factors)
- [11. References](#11-references)
- [12. Quizzes](#12-quizzes)
- [13. 🆕 Extended TS vs Python Examples Deep Dive](#13-new-extended-ts-vs-python-examples-deep-dive)
- [14. 🆕 Visual Pattern Breakdowns](#14-new-visual-pattern-breakdowns)
- [15. 🆕 Edge Cases & Gotchas](#15-new-edge-cases--gotchas)
- [16. 🆕 Advanced TypeScript Regex Patterns](#16-new-advanced-typescript-regex-patterns)
- [17. 🆕 Advanced Python `regex` Module Features](#17-new-advanced-python-regex-module-features)
- [18. 🆕 Real-World Production Recipes](#18-new-real-world-production-recipes)
- [19. 🆕 Massive Expanded Quizzes & Exercises](#19-new-massive-expanded-quizzes--exercises)

---

## 1. Regex Landscape Overview

### Why Regex Matters for TypeScript Developers

Regular expressions are one area where Python's `re` module and TypeScript's `RegExp` diverge significantly in features, behavior, and ergonomics:

| Aspect | Python (`re`) | TypeScript (`RegExp`) | Impact |
|--------|---------------|---------------------|--------|
| **Underlying engine** | Python regex engine (backtracking NFA) — consistent across versions | V8/JavaScript engines — ECMAScript Regex standard (2018+ features) | Different engine quirks, different feature support |
| **Raw string support** | `r"pattern"` — backslashes are literal 🎉 | No raw string for regex; must escape every `\`: `"\\d{3}"` 😫 | Python is much more readable! |
| **Named groups** | `(?P<name>...)` syntax | `(?'name'...)` or `(?<name>...)` — both work | Both support it, different syntax |
| **Non-capturing groups** | `(?:pattern)` | Same: `(?:pattern)` ✅ Identical! | Same across languages |
| **Lookarounds** | `(?=...)`, `(?!...)`, `(?<=...)`, `(?<!...)` | `(?=...)`, `(?!...)` — lookBEHIND must be fixed-width | TS limited in lookbehind support! |
| **Verbose mode** | `re.VERBOSE` / `re.X` — whitespace & comments ignored! | No equivalent (must escape spaces `\ `) | Python is much more readable for complex patterns |
| **Unicode handling** | `\w`, `\d`, `\s` match Unicode by default in py3 | `\w` matches Unicode by default | Same behavior ✅ |
| **Flag syntax** | Flags as constants: `re.IGNORECASE` | Inline flags: `(?i)`, `/pattern/i` flag | Both support both approaches |

### Complexity Legend

| Symbol | Meaning |
|--------|---------|
| 🟢 Basic | Straightforward, memorize quickly |
| 🟡 Intermediate | Understand nuances and edge cases |
| 🔴 Advanced | Subtle behavior, common pitfalls, deep-dive required |

---

## 2. Python `re` Module — Complete API Reference

### 2.1 `re` Module Lookup Table

| Function / Constant | 🟢/🟡/🔴 | Returns | Description |
|-------------------|-----------|---------|-------------|
| `re.search(pattern, string)` | 🟢 | `Match | None` | First match anywhere in string (like TS `.test()`) |
| `re.match(pattern, string)` | 🟡 | `Match | None` | Match ONLY at start of string (like anchored `^pattern`) |
| `re.fullmatch(pattern, string)` | 🟡 | `Match | None` | Entire string must match the pattern ✅ |
| `re.findall(pattern, string)` | 🟢 | `list[str]` | All non-overlapping matches as strings |
| `re.finditer(pattern, string)` | 🟡 | `Iterator[Match]` | All non-overlapping matches as Match objects (lazy!) |
| `re.sub(pattern, repl, string)` | 🟢 | `str` | Replace first match (or all with `count=0`) |
| `re.subn(pattern, repl, string)` | 🟡 | `(new_str, count)` | Like `.sub()` but also returns replacement count |
| `re.split(pattern, string)` | 🟢 | `list[str]` | Split string by pattern (like TS `.split()`) |
| `re.compile(pattern, flags)` | 🟢 | `Pattern` | Precompile regex for reuse (performance!) |

### 2.2 The Five Core Functions — Deep Comparison with TS

```python
import re

text = "Contact us at support@example.com or sales@example.org"

# === re.search() — like RegExp.test() in TS ===
# Returns FIRST match object anywhere in string
match = re.search(r"[a-z]+@[a-z]+\.[a-z]+", text)
print(match.group())          # "support@example.com" — matched text
print(match.start())          # 13 — start index
print(match.end())            # 28 — end index (exclusive!)

# TypeScript equivalent:
// const m = text.match(/[a-z]+@[a-z]+\.[a-z]+/);
// if (m) console.log(m[0]); // "support@example.com"

# === re.match() — matches ONLY at start! 🟡 ===
result = re.match(r"[a-z]+@", text)  # None! Because "@" isn't at position 0!
# To match from start, explicitly anchor with ^:
match = re.match(r"^Contact", text)   # Match! ✅

# TypeScript has no exact equivalent — use /^pattern/ for same behavior!
# const result = text.match(/^Contact/); // Same as Python's re.match()

# === re.fullmatch() — entire string must match 🟡 ===
result = re.fullmatch(r"\d{3}-\d{2}-\d{4}", "123-45-6789")  # Match! ✅
result = re.fullmatch(r"\d{3}-\d{2}-\d{4}", "call 123-45-6789")  # None! ❌

# TypeScript: use ^ and $ anchors manually:
// const m = text.match(/^\d{3}-\d{2}-\d{4}$/); // Same as fullmatch()

# === re.findall() — all non-overlapping matches 🟢 ===
emails = re.findall(r"[a-z]+@[a-z]+\.[a-z]+", text)
print(emails)  # ["support@example.com", "sales@example.org"]

# TypeScript: use global flag
// const emails = text.match(/[a-z]+@[a-z]+\.[a-z]+/g); // Same!

# === re.finditer() — lazy iterator over all matches 🟡 ===
for m in re.finditer(r"[a-z]+@", text):
    print(f"Found '{m.group()}' at position {m.start()}-{m.end()}")
# More memory-efficient than findall for large texts (lazy evaluation!)

# TypeScript equivalent: use a while loop with RegExp.exec()
# const regex = /[a-z]+@[a-z]+\.[a-z]+/g;
# let m;
# while ((m = regex.exec(text)) !== null) {  // Must check m !== null!
#   console.log(m[0], m.index);              // Same as match.group() and match.start()
# }

# === re.sub() — replacement (like TS .replace()) 🟢 ===
result = re.sub(r"\d", "*", "abc123def456")
print(result)  # "abc***def**" — replaces ALL digits with *

# With count parameter:
result = re.sub(r"\d", "*", "abc123def456", count=2)
print(result)  # "abc**3def**6" — only first 2 replacements

# TypeScript equivalent: "abc123def456".replace(/\d/g, "*")

# === re.split() — splitting by regex (like TS .split()) 🟢 ===
words = re.split(r"\s+", "hello   world\t! python")
print(words)  # ["hello", "world", "!", "python"] — splits on any whitespace!

# TypeScript equivalent: "hello   world\\t! python".split(/\s+/)

# Capture groups in split: groups are INCLUDED in the result
parts = re.split(r"(,|;)", "a,b;c,d")  # ["a", ",", "b", ";", "c,", ...]
# Groups appear in the output list — TypeScript's .split() discards them!
```

### 2.3 Flags / Options (Both Languages)

```python
import re

text = "Hello WORLD hello World"

# Flag: case-insensitive
re.search(r"hello", text)              # None — case-sensitive by default
re.search(r"hello", text, re.IGNORECASE)  # Match! ✅
# Same as: /hello/i in TS
result = re.findall(r"world", text, re.I)  # ["WORLD", "World"]

# Flag: multiline (^ and $ match line boundaries!)
multi_line = "first line\nsecond line\nthird line"
re.search(r"^second", multi_line)               # None — ^ matches start of STRING only!
re.search(r"^second", multi_line, re.MULTILINE)  # Match! ✅
# Same as: /second/m in TS (but also available as inline (?m))

# Flag: dotall (.) matches newlines too!
with_newline = "hello\nworld"
re.search(r"h.*d", with_newline)          # None — . doesn't match \n by default
re.search(r"h.*d", with_newline, re.DOTALL)  # Match! ✅ -> "hello\\nworl"
# Same as: /h.s/d in TS (or inline (?s))

# Flag: verbose (ignore whitespace & allow # comments!)
pattern = re.compile(r"""
    \+?       # Optional country code
    (\d{3})   # Area code
    [-.\s]?   # Optional separator
    (\d{3})   # First 3 digits
    [-.\s]?   # Optional separator
    (\d{4})   # Last 4 digits
""", re.VERBOSE)
# In TS: no equivalent — must write "\\+?(\\d{3})([-.\\s]?)?..." 😫

# Combining multiple flags:
re.search(r"pattern", text, re.IGNORECASE | re.MULTILINE | re.VERBOSE)
# Same as: /pattern/gims in TS (inline flags) or (?ims:pattern)
```

### 2.4 `re.compile()` — Precompilation for Performance 🟢

```python
# Compile once, reuse many times — saves repeated regex parsing!
PHONE_PATTERN = re.compile(
    r"^\+?(\d{1,3})[-.\s]?(\d{3})[-.\s]?(\d{3})[-.\s]?(\d{4})$"
)

phone_numbers = [
    "+1-555-123-4567",
    "555.987.6543",
    "(555) 234 5678",   # Won't match without extra patterns
]

for phone in phone_numbers:
    m = PHONE_PATTERN.match(phone)
    if m:
        print(m.groups())  # Extract groups directly!

# TypeScript equivalent (no compile step — JS engine caches automatically):
// const PHONE_RE = /^\+?(\d{1,3})[-.\s]?(\d{3})[-.\s]?(\d{3})[-.\s]?(\d{4})$/;
// TS/V8 handles cache automatically. Python needs explicit compile().

# When to use re.compile():
# - Pattern is used 5+ times → compile once ✓
# - In tight loops → compile outside the loop ✓
# - Complex patterns with many sub-patterns → save parse time ✓
```

### 2.5 The Match Object — Complete API Reference

| Method / Attribute | 🟢/🟡/🔴 | Returns | Description |
|-------------------|-----------|---------|-------------|
| `.group([n])` | 🟢 | `str` | Extracted text (0 = full match, 1+ = captured groups) |
| `.groups()` | 🟢 | `tuple[str, ...]` | All capture groups as a tuple |
| `.groupdict()` | 🟡 | `dict[str, str]` | Named capture groups as dict |
| `.start([n])` | 🟢 | `int` | Start index of group n (or full match if 0) |
| `.end([n])` | 🟢 | `int` | End index (exclusive) of group n |
| `.span([n])` | 🟡 | `(start, end)` | Tuple of (start, end) indices |
| `.expand(template)` | 🟡 | `str` | Expand backreferences in template string |

```python
import re

text = "John Smith, john@example.com, +1-555-123-4567"

# Named groups make extraction easy
pattern = re.compile(
    r"(?P<name>[A-Z][a-z]+ [A-Z][a-z]+),\s*"
    r"(?P<email>[a-z]+@[a-z]+\.[a-z]+),\s*"
    r"(?P<phone>\+1-\d{3}-\d{3}-\d{4})"
)

m = pattern.search(text)

# .group() — extract specific group
print(m.group())           # Full match: "John Smith, john@example.com, +1-555-123-4567"
print(m.group(0))          # Same as .group() — full match
print(m.group("name"))     # "John Smith" — named group access!

# .groups() — all groups as tuple
print(m.groups())  # ("John Smith", "john@example.com", "+1-555-123-4567")

# .groupdict() — named groups as dict
print(m.groupdict())   # {"name": "John Smith", "email": "john@example.com", "phone": "+1-555-123-4567"}

# .start() / .end() / .span() — positions
print(m.start("email"))  # 17 — start of email in original string
print(m.end("email"))    # 34 — end (exclusive)
print(m.span("phone"))   # (36, 50) — tuple form

# .expand() — replace backreferences in template string
template = "Name: {name}, Email: <{email}>"
print(m.expand(template))  # "Name: John Smith, Email: <john@example.com>"
# Similar to TS destructuring from match array.
```

### 2.6 Replacement Strings in `re.sub()` — Backreferences 🟡

```python
import re

text = "John Smith - jane doe"

# Using \\1, \\2 backreferences (in replacement string)
result = re.sub(r"([A-Z][a-z]+) ([A-Z][a-z]+)", r"\2, \1", text)
print(result)  # "Smith John - doe jane" — name swapped!

# Named group backreferences
result = re.sub(r"(?P<first>[A-Z][a-z]+) (?P<last>[A-Z][a-z]+)",
                r"\g<last>, \g<first>", text)
print(result)  # "Smith John - doe jane"

# f-string compatible (Python 3.12+): use a callable instead!
def swap_name(match):
    first, last = match.groups()
    return f"{last}, {first}"

result = re.sub(r"[A-Z][a-z]+ [A-Z][a-z]+", swap_name, text)
print(result)  # "Smith John - doe jane" — using callable for complex logic!

# In TypeScript (must use literal backreferences in replacement string):
// "John Smith - jane doe".replace(/([A-Z][a-z]+) ([A-Z][a-z]+)/g, "$2, $1")
```

### 2.7 `re.split()` — Advanced Splitting 🟡

```python
import re

text = "hello,world;foo|bar  baz\tqux"

# Split by multiple delimiters in one call!
parts = re.split(r"[,;\s|]+", text)
print(parts)  # ["hello", "world", "foo", "bar", "baz", "qux"]

# Split keeping the delimiter (using capturing groups):
text = "a-b-c-d"
result = re.split(r"(-)", text)     # Includes '-' in output!
print(result)  # ["a", "-", "b", "-", "c", "-", "d"]

# Max splits: limit number of splits
text = "one two three four five"
parts = re.split(r"\s+", text, maxsplit=2)
print(parts)  # ["one", "two", "three four five"] — only first 2 splits!

# TypeScript equivalent: use regex with capture groups manually
// text.split(/(-)/g); // Same behavior in TS!
```

---

## 3. TypeScript `RegExp` — Complete API Reference

### 3.1 TypeScript RegExp Constructor & Properties

| Property | 🟢/🟡/🔴 | Type | Description |
|----------|-----------|------|-------------|
| `.source` | 🟢 | `string` | The pattern text (without delimiters) |
| `.flags` | 🟢 | `string` | Active flags as a string (`"gim"`) |
| `.global` | 🟢 | `boolean` | Whether `g` flag is set |
| `.ignoreCase` | 🟢 | `boolean` | Whether `i` flag is set |
| `.multiline` | 🟢 | `boolean` | Whether `m` flag is set |
| `.dotAll` | 🟡 | `boolean` | Whether `s` flag is set (ES2018+) |
| `.sticky` | 🔴 | `boolean` | Whether `y` flag is set — only matches at lastIndex! |
| `.unicode` | 🟡 | `boolean` | Whether `u` flag is set — Unicode semantics |
| `.lastIndex` | 🔴 | `number` | Start position for next match (only with `g` or `y`) |

```typescript
// Constructor (both forms are equivalent):
const regex1 = new RegExp("abc", "gi");    // With flags as second arg
const regex2 = /abc/gi;                     // Literal syntax (preferred!)

// Properties:
const emailRe = /^[^\s@]+@[^\s@]+\.[^\s@]+$/i;
emailRe.source   // "/^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/" — pattern text
emailRe.flags    // "i" — active flags
emailRe.global   // false — no 'g' flag
```

### 3.2 RegExp Methods on String (The TS Developer's Way)

| String Method | 🟢/🟡/🔴 | Returns | Behavior |
|--------------|-----------|---------|----------|
| `str.match(pattern)` | 🟢 | `string[]|null` | All matches (if `g`) or first match with groups |
| `str.matchAll(pattern)` | 🟡 | `Iterable<Match>` | All non-overlapping matches as iterable (ES2020+) |
| `str.search(pattern)` | 🟢 | `number` | Index of first match (-1 if none) |
| `str.replace(pattern, repl)` | 🟢 | `string` | Replace (all if `g`, one without `g`) |
| `str.split(pattern)` | 🟡 | `string[]` | Split by regex (discards delimiters unlike Python!) |
| `str.includes(pattern)` | 🔴 | `boolean` | Does it literally include the string? NOT regex! |

```typescript
// === match() — with and without 'g' flag ===
const text = "Contact: john@example.com, jane@test.org";

// Without 'g': returns match array like JS/TS exec result
const m1 = text.match(/[a-z]+@[a-z]+\.[a-z]+/);  // No 'g'!
console.log(m1[0]);        // "john@example.com" — matched text
console.log(m1.index);     // 9 — start index (NOT called .start() in TS!)

// With 'g': returns ONLY the matched strings (no group info!)
const m2 = text.match(/[a-z]+@[a-z]+\.[a-z]+/g);
console.log(m2);            // ["john@example.com", "jane@test.org"]
// ⚠️ No way to get indices or groups from this form!

// === matchAll() — the modern, complete approach (ES2020+) ===
const matches = [...text.matchAll(/[a-z]+@[a-z]+\.[a-z]+/g)];
// Same as Python's re.finditer()!
matches.forEach((m) => {
    console.log(m[0], m.index);     // "john@example.com" 9
});

// === search() — like indexOf but with regex ===
const idx = text.search(/@/);        // 15 — position of first @
console.log(idx);                     // -1 if not found

// === replace() — basic and advanced ===
// Basic: replace ALL matches (need 'g' flag!)
text.replace(/@/g, "#");             // "Contact: john#example.com, jane#test.org"

// Replace with groups (same as Python's \\1, \\2):
const name = "John Smith";
name.replace(/([A-Z][a-z]+) ([A-Z][a-z]+)/, "$2, $1");  // "Smith John"

// Replace with a callback function:
"text 123 hello".replace(/\d+/g, (match) => `[${match}]`);  // "text [123] hello"

// === split() — like Python's re.split() but discards delimiters ===
"a,b;c,d".split(/[;,]/);   // ["a", "b", "c", "d"] — delimiters are GONE!
```

### 3.3 The `sticky` Flag (`y`) — Unique to TS/JS 🔴

```typescript
// sticky flag only matches at regex.lastIndex position!
const text = "Hello world";
const regex = /world/y;
regex.lastIndex = 6;              // Start searching at position 6
console.log(regex.test(text));    // true — matches exactly at position 6!

regex.lastIndex = 0;
console.log(regex.test(text));    // false — doesn't match at position 0!

// sticky is used in tokenizer implementations:
function tokenize(input: string) {
    const tokenRegex = /\s+|(\d+)|([a-z]+)/gy;
    let match;
    const tokens: string[] = [];
    while ((match = tokenRegex.exec(input)) !== null) {
        if (match[0].length > 0) tokens.push(match[1] || match[2]);
    }
    return tokens;  // ["hello", "123"] for "hello 123"
}

// ⚠️ sticky flag has a major caveat: lastIndex must be manually managed!
// If not at lastIndex, test() returns false and does NOT advance lastIndex.
```

### 3.4 Unicode Flag (`u`) — Important for Multilingual Text 🟡

```typescript
// Without 'u': treats string as UTF-16 code units (broken with emojis!)
"🎉🎊".length;           // 4 — counts surrogate pairs separately! ❌
/\x{1F389}/.test("🎉");  // false — \\x{} not valid without 'u' flag!

// With 'u': treats string as Unicode code points (correct!)
"🎉🎊".length;           // 2 — counts actual characters! ✅
/\p{Extended_Pictographic}/u.test("🎉"); // true — Unicode property escapes!
/u{1F389}/.test("🎉");         // true — direct code point!

// Python equivalent: re with u flag is automatic (Py3 strings are Unicode)
import re
re.search(r'\p{Extended_Pictographic}', '🎉')  # Need regex module, not stdlib re
# NOTE: Python's stdlib `re` doesn't support \p{} — use the third-party `regex` module!
```

---

## 4. Metacharacters & Character Classes: Every Pattern Symbol

### 4.1 Complete Metacharacter Reference Table

| Pattern | 🟢/🟡/🔴 | Matches | Python Notes | TypeScript Notes |
|---------|-----------|---------|-------------|-----------------|
| `.` | 🟢 | Any character EXCEPT newline | Use `re.DOTALL` to include `\n` | Use `s` flag (`/./s`) to include `\n` |
| `\d` | 🟢 | Digit `[0-9]` | Same as Unicode digits (3.12+) | Same — ASCII digits only! |
| `\D` | 🟢 | Non-digit | Same as `[^0-9]` | Same |
| `\w` | 🟢 | Word char `[a-zA-Z0-9_]` | Includes Unicode letters/digits in Py3.12+ | ASCII only (no accented chars!) |
| `\W` | 🟢 | Non-word char | Same as `[^a-zA-Z0-9_]` | Same |
| `\s` | 🟢 | Whitespace `[ \t\n\r\f\v]` | Same Unicode whitespace in Py3.12+ | ASCII whitespace only! |
| `\S` | 🟢 | Non-whitespace | `[^ \t\n\r\f\v]` | Same |
| `\b` | 🟢 | Word boundary | Between `\w` and `\W` (or start/end) | Same — depends on word char definition |
| `\B` | 🔴 | NOT a word boundary | Same as negative word boundary | Same |
| `^` | 🟡 | Start of string (or line with `m` flag) | Anchors to START only by default | Same |
| `$` | 🟡 | End of string (or line with `m` flag) | Anchors to END only by default | Same |
| `*` | 🟢 | 0 or more of preceding | Greedy by default! Use `*?` for lazy | Same behavior |
| `+` | 🟢 | 1 or more of preceding | Greedy by default! Use `+?` for lazy | Same |
| `?` | 🟢 | 0 or 1 of preceding | Greedy by default! Use `??` for lazy | Same |
| `{n}` | 🟢 | Exactly n repetitions | — | Same |
| `{n,}` | 🟢 | At least n repetitions | — | Same |
| `{n,m}` | 🟢 | Between n and m repetitions | — | Same |
| `[]` | 🟡 | Character class (any char in brackets) | Use `[^...]` for negation | Same syntax! |
| `()` | 🟢 | Capture group | Groups: 1,2,... Named: `(?P<name>...)` | Groups: `$1`, `$2`. Named: `(?<name>)` |
| `(?:...)` | 🟢 | Non-capturing group | Same in both languages ✅ | Same! |
| `|` | 🟢 | Alternation (OR) | Left-to-right — first match wins | Same behavior |
| `\\` | 🟢 | Literal backslash | Must double-escape: `"\\\\" `or use raw strings | Must escape: `"\\"` in string literal |

### 4.2 Character Classes Deep Dive

```python
import re

# === Basic character classes [] ===
re.search(r"[aeiou]", "hello")       # Match — finds first vowel
re.search(r"[^aeiou]", "hello")      # Match — finds consonant 'h' (negated class)
re.search(r"[A-Z][a-z]+", "Hello")   # Match — one uppercase followed by lowercase

# Range inside classes:
re.search(r"[0-9]{3}", "abc123def")  # Match — digits 0-9
re.search(r"[a-zA-Z]", "hello")      # Match — any letter
re.search(r"[\d\w]", "test_123")     # Match — digit or word char (combined)

# Unicode in character classes (Python 3.12+):
re.search(r"[\p{L}]+", "café résumé")  # Need 'regex' module, not stdlib re!
# stdlib re doesn't support \p{} — this is a key limitation!

# Special chars inside [] lose their metacharacter meaning:
re.search(r"[.*+?^${}()|[\]\\]", "hello.world")  # Matches the '.' literally
# Inside [], most metacharacters are literal EXCEPT: \, ^ (at start), - (not at end)
```

### 4.3 Python-Specific Character Escapes (stdlib `re` only) 🔴

```python
import re

# Python regex has extensions beyond ECMAScript standard:
# These work in Python's stdlib re but NOT in TypeScript!

re.search(r"\Ahello", "hello world")    # \A — matches START of string (not line)
re.search(r"world\Z", "hello world")    # \Z — matches END of string (not line)
re.search(r"world\z", "hello world")    # \z — same as \Z (end of string only)
re.search(r"\bhello\b", "hello world")  # \b — word boundary ✓
re.search(r"\bhello\b", "ahello world")  # No match — 'a' is a word char!

# Python's unique escapes:
re.search(r"\G", "hello")               # \G — matches at current position (0 or after last match)
re.search(r"(?P<name>pattern)", text)   # (?P<name>...) — named groups (TS uses (?<name>) instead!)

# ❌ Python's re does NOT support these (unlike TypeScript/ECMAScript 2018+):
# \p{L} — Unicode property escapes → use third-party 'regex' module!
# (?|pattern) — Branch reset groups → not supported in stdlib re!
# (?<name>pattern) — Alternative named group syntax → use (?P<name>...) instead!
```

### 4.4 TypeScript-Specific Metacharacters (ECMAScript 2018+) 🟡

```typescript
// ECMAScript 2018+ added features NOT in Python's stdlib re:

// Unicode property escapes (requires 'u' flag!)
/\p{Script=Latin}/u.test("café");       // true — Latin script chars
/\p{Number}/u.test("½");                // true — includes fractions!
/\P{Number}/u.test("a");                // \P = NOT (uppercase P)

// s flag (dotall) — already shown, but explicit:
/.s/u;  // Same as Python's re.DOTALL / re.S

// Named capturing groups — BOTH syntaxes work in TS!
/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
    .exec("2025-01-15")?.groups;  // { year: "2025", month: "01", day: "15" }

// Unicode escape sequences in character classes:
/[\\u{1F389}-\\u{1F9FF}]/su.test("🎉");  // Emoji range check! (Requires 's' + 'u')

// ❌ TypeScript regex does NOT support these (unlike Python):
// \A, \Z — Python-only anchors (TS uses ^ and $)
// \G — Position anchor not supported in TS regex!
// (?P<name>) — Python's named group syntax (use (?<name>) in TS!)
// re.VERBOSE mode — whitespace/comment ignoring not supported in TS!
```

---

## 5. Quantifiers, Groups & Assertions: The Building Blocks

### 5.1 Complete Quantifier Reference

| Pattern | Name | Matches | Greedy/Lazy | Python TS Equivalent |
|---------|------|---------|-------------|---------------------|
| `*` | Kleene star | 0 or more | ✅ Greedy by default, `*?` lazy | Both |
| `+` | Plus | 1 or more | ✅ Greedy, `+?` lazy | Both |
| `?` | Optional | 0 or 1 | ✅ Greedy, `??` lazy | Both |
| `{n}` | Exact count | Exactly n | N/A (fixed) | Both |
| `{n,}` | Min count | At least n | ✅ Greedy, `{n,}?` lazy | Both |
| `{n,m}` | Range | Between n and m | ✅ Greedy, `{n,m}?` lazy | Both |

```python
import re

# === Greedy vs Lazy (Minimal) Quantifiers 🔴 ===
# GREEDY (default): matches as MUCH as possible
re.sub(r"<.*>", "REPLACED", "<div><p>hello</p></div>")
# "REPLACED" — .* matched from FIRST < to LAST >! (catastrophic overreach)

# LAZY (minimal): matches as LITTLE as possible
re.sub(r"<.*?>", "REPLACED", "<div><p>hello</p></div>")
# "REPLACEDREPLACEDhelloREPLACEDREPLACED" — each tag replaced individually ✓

# All quantifiers have lazy variants:
"*"  → "*?"   "+ " → "+?"   "?" → "??"   "{n,m}" → "{n,m}?"
# These work identically in both Python and TypeScript!

# === Performance impact of greedy vs lazy 🔴 ===
# Greedy quantifiers can cause catastrophic backtracking:
text = "aaaaaaaaaaaaaaaaaaaa"  # 20 'a's — no closing tag
re.search(r"<.*>", text)      # ✅ Fast — .* matches nothing (no < found)
re.search(r"<.+?>", text)     # Still fast

# But this pattern is DANGEROUS:
text = "a" * 100 + "b"
re.search(r"^(a+)+b$", text)   # May take FOREVER — exponential backtracking!
# Both languages suffer from the same issue: NFA backtracking engine!
```

### 5.2 Groups & Backreferences Deep Dive

| Pattern | Name | Python Syntax | TypeScript Syntax | Notes |
|---------|------|--------------|-------------------|-------|
| Capture group | Named capture | `(?P<name>...)` | `(?<name>...)` | Extracts and names the captured text |
| Non-capturing group | Group without capture | `(?:...)` | Same: `(?:...)` | Groups logic, no memory overhead |
| Backreference in pattern | Reference captured group | `\1`, `\2`, `(?P=name)` | `\1`, `\2`, `\k<name>` | Must match EXACTLY what was captured |
| Backreference in replacement | Reference in replace string | `\1`, `\g<name>` | `$1`, `$2`, `$&`, `$'` | In replacement, not in pattern! |

```python
import re

text = "email: john@example.com | phone: 555-1234"

# Named groups — Python syntax
pattern = re.compile(
    r"(?P<name>[a-z]+)@(?P<domain>[a-z]+\.[a-z]+)"
)
m = pattern.search(text)
print(m.group("name"))      # "john"
print(m.group("domain"))    # "example.com"
print(m.groups())           # ("john", "example.com")

# Backreferences within the SAME pattern — enforce matching pairs!
text2 = "<div>Hello</div><span>World</span>"
# Match only correctly paired tags:
re.search(r"<(?P<tag>[a-z]+)>.*?</(?P=tag)>", text2)  # (?P=tag) = backreference ✓
# TypeScript equivalent: /<(?<tag>[a-z]+)>.*?<\k<tag>>/

# Non-capturing groups (no memory overhead, cleaner):
pattern = re.compile(r"(?:https?://)?([a-z]+\.[a-z]+)/path")
# The (?:...) doesn't appear in .groups() — only group 1 captures!

# Substitution with named groups:
result = re.sub(
    r"(?P<first>[A-Z][a-z]+) (?P<last>[A-Z][a-z]+)",
    r"\g<last>, \g<first>",
    "John Smith"
)
print(result)  # "Smith, John"

# Group numbering:
pattern = re.compile(r"(\d+)-(\w+)-(\d+)")
m = pattern.search("123-abc-456")
print(m.group(0))     # "123-abc-456" — full match
print(m.group(1))     # "123" — first capture group
print(m.group(2))     # "abc"
print(m.group(3))     # "456"

# ❌ Common pitfall: over-capturing!
re.findall(r".*", "hello")    # ['hello', ''] — matches empty at end too!
# Fix: use specific patterns instead of .*
```

### 5.3 Lookarounds — Zero-Width Assertions 🔴

| Pattern | Name | Meaning | Python ✅ | TypeScript ✅ |
|---------|------|---------|-----------|-------------|
| `(?=...)` | Positive lookahead | What follows MUST match | ✅ | ✅ |
| `(?!...)` | Negative lookahead | What follows must NOT match | ✅ | ✅ |
| `(?<=...)` | Positive lookbehind | Preceding text MUST match | ✅ (variable width!) | ⚠️ Fixed-width only! |
| `(?<!...)` | Negative lookbehind | Preceding text must NOT match | ✅ (variable width!) | ⚠️ Fixed-width only! |

```python
import re

# === Positive lookahead: assert what follows without consuming ===
# Extract price: number immediately followed by '$'
prices = re.findall(r"\d+(?=\$)", "$9.99 and $15.00")
print(prices)  # ["9", "15"] — dollars not included!

# TypeScript: same syntax works!
// /\d+(?=\$)/g.test("$9.99"); // Same behavior ✓

# === Negative lookahead: assert what does NOT follow ===
# Match 'foo' NOT followed by 'bar':
re.findall(r"foo(?!bar)", "foobar foobaz foo")  # ["foo", "foo"] — only non-'bar' foos

# Practical: match passwords with at least one digit (lookahead assertion):
has_digit = re.search(r".*\d", password)   # Simple version
has_digit_and_upper = re.search(r"(?=.*\d)(?=.*[A-Z]).{8,}", password)  # Both required!

# === Positive lookbehind: assert what precedes (must match!) ===
# Get the number in "500 items":
re.findall(r"(?<=\$)\d+", "$99 and $150")  # ["99", "150"]
re.findall(r"(?<=item\s)\d+", "500 items, 300 items")  # ["500", "300"]

# === Negative lookbehind: assert what does NOT precede ===
# Match numbers NOT preceded by ':':
re.findall(r"(?<!:)\b\d+\b", "price:500 tax:100 total:200 extra:999")  # ["extra"] ❌ wrong!
re.findall(r"\b(?<!:)\d+\b", "price:500 tax:100 total:200 extra:999")   # Better approach

# === Key difference: Python vs TypeScript lookbehind 🟡 ===
# Python: (?<=...) can use VARIABLE WIDTH patterns!
re.search(r"(?<=https://|http://|ftp://)", "ftp://example.com")  # ✅ Works in Python!

# TypeScript: lookbehind MUST be fixed-width!
// /(?<=(https://|http://|ftp:\/\/))/ /* ❌ Variable width — NOT allowed in TS regex! */
// Must rewrite as: /(https:\/\/|http:\/\/|ftp:\/\/)/

# === Common pitfall: Lookahead doesn't consume! ✅===
text = "foo bar baz"
re.search(r"foo\s+(?=bar)", text)  # Matches "foo " but only "foo " is consumed
rest = text[re.search(r"foo\s+(?=bar)", text).end():]  # "baz" — 'bar' was NOT consumed!
```

### 5.4 Alternation `|` — The OR Operator

```python
import re

# Basic alternation: match A or B
re.findall(r"cat|dog", "I have a cat and a dog")  # ["cat", "dog"]

# Alternation is LEFT-TO-RIGHT (first match wins!) — critical!
re.findall(r"longer|short", "long string")   # ["long"] — 'l' matched, stops at first alt
re.findall(r"short|longer", "long string")   # Still ["long"] — alternation matches the LEFT most first!

# ⚠️ Order matters for alternatives! Put longer patterns first:
colors = "red green blue yellow"
re.findall(r"red|yellow|green|blue|orange", colors)  # Works because 'red' doesn't overlap with others
# But: re.findall(r"r|red", "red") → ["r"] — WRONG! Always put longer alternatives FIRST!

# Grouped alternation:
re.findall(r"(?:https?|ftp)://[a-z]+", "Visit https://example.com or ftp://files.com")
# ["https://example.com", "ftp://files.com"] ✓

# TypeScript equivalent: same rules apply — left-to-right, order matters!
```

### 5.5 Anchors `^` and `$` — Position Assertions

| Pattern | Meaning | Default Behavior | With Multiline Flag |
|---------|---------|-----------------|-------------------|
| `^` | Start of string (or line) | Matches start of entire string | Matches start of EACH LINE |
| `$` | End of string (or line) | Matches end of entire string | Matches end of EACH LINE |
| `\A` | Absolute start (Python only) | Always matches very start | Same — never affected by `m` flag |
| `\Z` | Absolute end (Python only) | Always matches very end | Same — never affected by `m` flag |

```python
import re

text = "line 1\nline 2\nline 3"

# Default: ^ and $ match start/end of STRING, not lines!
re.findall(r"^line", text)          # ["line"] — only first line ✓
re.findall(r"line$", text)          # ["line"] — only last line ✓

# With re.MULTILINE (re.M): ^ and $ match START/END OF EACH LINE!
re.findall(r"^line", text, re.M)    # ["line", "line", "line"] — all 3 lines!
re.findall(r"line$", text, re.M)    # Same result ✓

# Python-only: \A and \Z are ALWAYS absolute (immune to m flag)!
re.search(r"\Aline", text)           # Match ✅
re.search(r"line\Z", text)           # Match ✅
re.search(r"^line", text, re.M)     # Still matches at start (coincidentally same)

# ❌ Common bug: $ doesn't match before \r in Windows line endings!
text_windows = "hello\r\nworld"
re.findall(r".+$", text_windows)    # ["world"] — but also matches empty string before \n!
# Fix: use r"\r?$" or ensure consistent line endings.

# TypeScript: same ^/$ behavior! (use /m flag for multiline)
// "line 1\\nline 2".match(/^line/gm);  // Same as Python's re.M!
```

---

## 6. Python vs TypeScript Regex Feature Comparison Matrix

### 6.1 Complete Feature Comparison Table

| Feature | Python `re` | TypeScript `RegExp` | Comments |
|---------|------------|---------------------|----------|
| Basic patterns (`\d`, `\w`, `.` etc.) | ✅ Same | ✅ Same | Identical behavior |
| Character classes `[]` | ✅ Same | ✅ Same | Identical |
| Quantifiers (`* + ? {n,m}`) | ✅ Same | ✅ Same | Identical |
| Lazy quantifiers (`*? +? ??`) | ✅ Yes | ✅ Yes | Identical |
| Groups `(?:...)` (non-capturing) | ✅ Same | ✅ Same | Identical |
| Named groups `(?P<name>...)` | ✅ Python syntax | ⚠️ Use `(?<name>...)` | Different syntax! |
| Backreferences `\1, \2` in pattern | ✅ Yes | ✅ Yes | Identical |
| Lookahead `(?=...)`, `(?!...)` | ✅ Variable width OK | ✅ Fixed-width OK | Same behavior |
| Lookbehind `(?<=...)`, `(?<!...)` | ✅ Variable width | ⚠️ **Fixed width only!** | Critical difference! |
| Anchors `^`, `$` | ✅ Both modes | ✅ Both modes | With `/m` flag |
| `\A`, `\Z` (absolute anchors) | ✅ Python only | ❌ Not supported | Python-only feature |
| `\G` (match at position) | ✅ Python only | ❌ Not supported | Python-only |
| Unicode property escapes `\p{}` | ⚠️ No (need `regex` module!) | ✅ Yes (with `u` flag) | TS has more Unicode support! |
| Unicode escape `\u{XXXX}` in class | ⚠️ Limited | ✅ Yes (with `s`, `u` flags) | TS is richer |
| Dotall (`.` matches newline) | `re.DOTALL` or inline `(?s)` | `s` flag or inline `(?s)` | Same concept, different API |
| Multiline (`^`/`$` per line) | `re.MULTILINE` or inline `(?m)` | `m` flag or inline `(?m)` | Same concept |
| Case-insensitive | `re.IGNORECASE` or inline `(?i)` | `i` flag or inline `(?i)` | Identical! ✅ |
| Verbose mode (ignore whitespace) | `re.VERBOSE` / inline `(?x)` | ❌ **Not supported!** | Python-only feature — huge readability win |
| Sticky flag (`y`) | ❌ Not supported | ✅ Yes | TS/JS unique — matches only at lastIndex |
| String methods: match, matchAll, search, replace, split | N/A (regex is a separate module) | ✅ Built-in on all strings | In TS, regex methods are on String, not RegExp object! |
| Raw string support | ✅ `r"pattern"` — NO double escaping! | ❌ Must manually escape: `"\\d{3}"` | Massive ergonomics win for Python |
| Replace with callback | ✅ `re.sub(pattern, callable, str)` | ✅ `"text".replace(regex, callback)` | Both support functional replacement ✓ |
| Capture group access in replace | `\1`, `\g<name>` (Python) / `$&`, `$1` | `$1`, `$2`, `$&`, `$'`, `` $`` `` (TS) | Different naming! |
| Pattern compilation | ✅ `re.compile(pattern)` for performance | ⚠️ V8 auto-caches; explicit compile not needed | Python needs it; TS handles it automatically |

### 6.2 Ergonomics: Raw Strings vs Regex Literals

```python
# === Python: raw strings make patterns READABLE 🎉 ===
# Email pattern in Python:
email_re = r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}$"
# Every \ is literal — no double-backslash madness!

# Phone number (complex):
phone_re = r"^(\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$"
```

```typescript
// === TypeScript: regex literals auto-handle one level of escaping ===
// Email pattern in TS:
const email_re = /^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}$/;
// Regex literal syntax \/pattern\/ — no string escaping needed!

// But with new RegExp() constructor (dynamic patterns), double-escape needed!
const dynamicEmail = new RegExp(`^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z]{2,}$`);
// Note: \\ . because string literal interprets \\ as single \.

// Phone number (complex):
const phone_re = /^(\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$/;
// Regex literal is clean! new RegExp() would need double-escape.

# Rule of thumb:
# Python: always use r"..." for patterns (no raw string for regex in TS, but /.../ works!)
# TypeScript: prefer /pattern/ literals over new RegExp() — cleaner and auto-don't double-escape!
```

---

## 7. Common Patterns Library: Ready-to-Use Recipes

### 7.1 Validation Patterns (Every Type)

| Pattern | What It Validates | Python | TypeScript | Notes |
|---------|------------------|--------|-----------|-------|
| **Email** | Valid email address | `r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}$"` | `/^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}$/` | RFC-ish; doesn't cover all valid emails |
| **Phone (US)** | US phone number | `r"^\+?1?[-.\s]?\(?(\d{3})\)?[-.\s]?(\d{3})[-.\s]?(\\d{4})$"` | Same with `/.../g` | Handles +1, 555-1234, (555) 123-4567 |
| **URL** | HTTP/HTTPS URL | `r"^(https?://)?([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})(/[^\s]*)?$"` | Same | Basic; for full RFC URLs use a library |
| **IPv4** | IPv4 address (1.0.0.0 - 255.255.255.255) | `r"^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$"` | Same | Complex look-ahead for 0-255 range |
| **Hex Color** | `#RGB` or `#RRGGBB` | `r"^#([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$"` | Same | Shorthand and full hex |
| **UUID** | UUID v1-v5 format | `r"^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"` | Same | Case-insensitive hex groups |
| **Date (ISO 8601)** | YYYY-MM-DD | `r"^\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\d|3[01])$"` | Same | Validates ranges but not calendar logic! |
| **Base64** | Valid Base64 string | `r"^[A-Za-z0-9+/]+={0,2}$"` | Same | Optional padding (=) |

```python
import re

# === Practical validation functions ===

def is_valid_email(email: str) -> bool:
    return bool(re.fullmatch(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}", email))

def is_valid_phone(phone: str) -> bool:
    return bool(re.fullmatch(r"^\+?1?[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$", phone))

def is_valid_ipv4(ip: str) -> bool:
    pattern = r"^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$"
    return bool(re.fullmatch(pattern, ip))

def is_valid_uuid(uuid: str) -> bool:
    return bool(re.fullmatch(r"[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}", uuid, re.I))

def is_valid_url(url: str) -> bool:
    pattern = r"^(https?://)?([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})(/[^\s]*)?$"
    return bool(re.fullmatch(pattern, url))

# Usage:
is_valid_email("user@example.com")   # True
is_valid_email("invalid@")            # False — no TLD domain!
is_valid_ipv4("192.168.1.1")         # True
is_valid_ipv4("999.999.999.999")     # False — each octet ≤ 255 ✓
```

### 7.2 Extraction Patterns

| Use Case | Python Pattern | TypeScript Equivalent | Result Example |
|----------|---------------|---------------------|---------------|
| **HTML tag extraction** | `r"<([^>]+)>(.*?)</\1>"` (with backreference) | Same in `/.../g` | `["div", "Hello"]` from `<div>Hello</div>` |
| **Markdown headers** | `r"^(#{1,6})\s+(.+)$"` (with `re.M`) | Same with `/gm` | Groups: (`###`, "Title") |
| **CSV fields** | `r"((?:[^,",]*)|(?:",.*?",))"` | Similar pattern | Fields of a CSV line |
| **JSON key-value pairs** | `r'"([^"]+)"\s*:\s*"([^"]*)"'` | Same (with `/g`) | `("name", "Alice")`, `("age", "30")` |
| **Log parsing** | `r"\[(?P<date>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] (?P<level>[A-Z]+): (?P<message>.+)"` | Named groups with `(?<name>)` syntax | Extracts date, level, message from log line |
| **IP address from text** | `r"\b(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\b"` | Same (with `/g`) | `["192.168.1.1", "10.0.0.1"]` from any text |
| **Git hash** | `r"\b[0-9a-f]{7,40}\b"` | Same with word boundaries | Short and long Git commit hashes |

```python
import re

# === Log line parsing example (named groups) ===
log_line = "[2025-01-15 14:30:45] ERROR: Connection timeout after 30s"
pattern = re.compile(
    r"\[(?P<date>\d{4}-\d{2}-\d{2} (?P<time>\d{2}:\d{2}:\d{2}))\] "
    r"(?P<level>[A-Z]+): (?P<message>.+)"
)

m = pattern.search(log_line)
if m:
    print(m.group("date"))   # "2025-01-15 14:30:45"
    print(m.group("time"))   # "14:30:45"
    print(m.group("level"))  # "ERROR"
    print(m.group("message"))# "Connection timeout after 30s"

# === CSV field extraction (with quoted fields) ===
csv_line = 'name,"Smith, John",age'
fields = re.findall(r'("(?:[^"\\]|\\.)*"|[^,]*)', csv_line)
print(fields)  # ['name', '"Smith, John"', 'age'] — handles commas inside quotes!
```

### 7.3 Sanitization / Transformation Patterns

| Use Case | Python Pattern | TypeScript Equivalent | Result Example |
|----------|---------------|---------------------|---------------|
| **Remove HTML tags** | `re.sub(r"<[^>]+>", "", html)` | `"text".replace(/<[^>]+>/g, "")` | Strips all HTML markup |
| **Slugify (URL-friendly)** | `re.sub(r"[^a-z0-9]+", "-", text.lower())` | Same with `.toLowerCase()` + replace | "Hello World!" → "hello-world" |
| **Mask credit card** | `re.sub(r"\d(?=\d{4})", "*", "1234567890123456")` | Same (with `/g`) | "***-***-***-3456" |
| **Normalize whitespace** | `re.sub(r"\s+", " ", text).strip()` | Same pattern + `.trim()` | Multiple spaces → single space |
| **Remove leading zeros** | `re.sub(r"^0+(?=\d)", "", "007")` | Same with `/g` | "007" → "7" |

```python
import re

def slugify(text: str) -> str:
    """Convert text to URL-friendly slug."""
    text = text.lower()                    # Lowercase
    text = re.sub(r"[^a-z0-9]+", "-", text)   # Non-alphanumeric → dash
    text = re.sub(r"-+", "-", text)            # Multiple dashes → single
    return text.strip("-")                      # Trim leading/trailing dashes

slugify("Hello World!  This is a test.")
# "hello-world-this-is-a-test" ✓

def mask_credit_card(card: str) -> str:
    """Mask all digits except last 4."""
    digits = re.sub(r"[^0-9]", "", card)  # Extract only digits
    return digits[:-4].replace(digits[:-4], "*" * len(digits[:-4])) + digits[-4:]

mask_credit_card("4111-1111-1111-1234")
# "****-****-****-1234" ✓
```

---

## 8. Advanced Regex Topics (Lookarounds, Branch Reset, Recursion)

### 8.1 Complex Lookaround Patterns — Real-World Use Cases 🔴

```python
import re

# === Password strength validation (multiple lookahead assertions) ===
def is_strong_password(password: str) -> bool:
    # Must have: 8+ chars, uppercase, lowercase, digit, special char
    patterns = [
        r".{8,}",              # Minimum 8 characters
        r"(?=.*[A-Z])",        # At least one uppercase
        r"(?=.*[a-z])",        # At least one lowercase
        r"(?=.*\d)",            # At least one digit
        r"(?=.*[@$!%*?&#])",   # At least one special char
    ]
    return all(re.search(p, password) for p in patterns)

is_strong_password("Abc123!@#")  # True ✓
is_strong_password("abc123")       # False — no uppercase or special ✓

# === Extract email from quoted string (lookahead + lookbehind) ===
text = 'Contact "john.doe@example.com" for info'
email_match = re.search(r'"(?=.*@.*\..{2,})([^"]+)"', text)  # Lookahead validates pattern before extracting

# === Match balanced parentheses using recursion (Python `regex` module only!) 🔴 ===
import regex  # Third-party: pip install regex

text = "(a + (b * c)) - d"
balanced = regex.search(r"\((?:[^()]+|\((?:[^()]+|\([^()]*\))*\))*\)", text)
# Or with the 'regex' module's actual recursion support:
balanced = regex.search(r"\((?:[^()]|(?R))*\)", text)  # (?R) = recurse entire pattern ✓

# === TypeScript doesn't support recursion at all in native RegExp! ===
// To do this in TS, you'd need a stack-based parser instead of regex.
```

### 8.2 Branching — The `(?|...)` Feature 🔴

```python
# Python's stdlib re does NOT support branch reset groups (?|...)!
# The 'regex' third-party module does:

import regex

text = "date: 2025-01-15 | time: 14:30:45"
# Branch reset: both branches use the SAME group number!
pattern = regex.compile(r"(?|(\d{4})-(\d{2})-(\d{2})|(\d{2}):(\d{2}):(\d{2}))")
m = pattern.search(text)
print(m.groups())  # ('2025', '01', '15') for date branch

# TypeScript: NO branch reset support. Use separate captures or post-process.
```

### 8.3 Conditional Patterns — If-Then in Regex 🔴

| Pattern | Name | Supported? | Python / TS |
|---------|------|-----------|-------------|
| `(?(name)yes|no)` | Conditional: if group exists, match yes; else no | ✅ Python `regex` module only | ❌ Neither stdlib supports this! |
| `(?(condition)yes|no)` | Check for backreference existence | ⚠️ Limited | ⚠️ Very limited / not supported |

```python
# Python's 'regex' module (third-party) supports conditional patterns:
import regex

text = "The price is $99 or 100 dollars"
price_pattern = regex.compile(r"(?:(?<dollar>\$)|(?<number>\d+))(?(dollar)\d{2}|\d+)")
# Match $99 OR 100, but not $ without digits, not number with $

# ❌ Python stdlib re does NOT support conditionals at all!
# ❌ TypeScript RegExp does NOT support conditionals at all!
# Solution: use a programming language's control flow instead of regex for this.
```

### 8.4 Possessive Quantifiers — Match and Never Give Back 🔴

| Pattern | Name | Supported? | Notes |
|---------|------|-----------|-------|
| `*+` | Possessive star | ✅ `regex` module only | Same as `*` but NEVER backtracks |
| `++` | Possessive plus | ✅ `regex` module only | Same as `+` but no backtrack |
| `{n,m}+` | Possessive range | ✅ `regex` module only | Same as `{n,m}` but no backtrack |

```python
# The 'regex' module: possessive quantifiers prevent catastrophic backtracking!
import regex

text = "a" * 1000 + "b"

# This pattern would take FOREVER with stdlib re (exponential backtracking):
# re.search(r"^(a+)+b$", text)

# With possessive quantifier — instant match, no backtracking:
regex.search(r"^(a++)*b$", text)  # Instant ✓ — 'a++' never gives back characters!

# TypeScript: NO possessive quantifier support. Use non-backtracking engine or rewrite logic.
```

---

## 9. Performance & Optimization

### 9.1 Performance Comparison: Python vs TypeScript Regex

| Metric | Python `re` | TypeScript `RegExp` | Notes |
|--------|-------------|---------------------|-------|
| **Pattern compilation** | Manual via `re.compile()` — V8 auto-caches | ✅ Automatic by V8 engine | In tight loops, always compile in Python; no action needed in TS |
| **Matching speed** | Fast for simple patterns (C-level NFA) | Very fast (V8 optimized JIT) | Both are C-optimized under the hood |
| **Complex pattern performance** | Can degrade with backtracking | Same — both use backtracking engines | Rewrite complex patterns as programmatic logic! |
| **Unicode handling speed** | Slower in Py3.12+ (full Unicode word chars) | Fast (UCD-based in V8) | For Unicode-heavy patterns, TS may be faster! |
| **Replace with callback** | `re.sub(pattern, fn, str)` — function call overhead | `"text".replace(regex, fn)` — same overhead | Callbacks are slow in BOTH — pre-build replacement string when possible! |

### 9.2 Common Performance Pitfalls & Fixes

```python
import re

# === PITFALL 1: Catastrophic Backtracking 🔴 ===
# Pattern like (a+)+b on a string of only 'a's causes exponential time!
text = "a" * 100 + "z"       # No 'b' at end — will cause backtracking!

# ❌ BAD: nested quantifiers create exponential backtracking
re.search(r"(a+)+", text)     # Can freeze!

# ✅ FIX: Use possessive or atomic grouping (via `regex` module):
import regex
regex.search(r"(a++)+", text)  # Instant — no backtrack possible!

# ✅ FIX: Rewrite the pattern to be unambiguous:
re.search(r"a+", text)         # Simple, unambiguous — always O(n)!

# === PITFALL 2: Unanchored . in tight loops 🔴 ===
for word in huge_dataset:
    re.search(r".*bad_pattern", word)   # Very slow! . can match anything.

# ✅ FIX: Anchor the pattern or use specific characters:
re.search(r"^.*bad_pattern$", word)     # ^$ anchors prevent runaway matches
re.search(r"pre[^a-z]*bad_pattern", word)  # [^a-z] is more constrained than .

# === PITFALL 3: Over-use of .* in findall ===
re.findall(r".*", huge_text)   # Returns every char + empty strings — very slow!

# ✅ FIX: Use specific character classes instead of .*:
re.findall(r"[a-z]+", huge_text)   # Much faster, no ambiguity!

# === PITFALL 4: Recompiling the same pattern in a loop ===
for line in lines:
    if re.search(r"\d{3}-\d{2}-\d{4}", line):  # Compiled every iteration! 😫
        ...

# ✅ FIX: Compile once, reuse:
SSN_PATTERN = re.compile(r"\d{3}-\d{2}-\d{4}")
for line in lines:
    if SSN_PATTERN.search(line):  # Reuses compiled pattern ✓
        ...
```

### 9.3 When to Use Regex vs String Methods

| Task | Use | Why |
|------|-----|-----|
| Simple substring check | `"abc" in text` | O(n), no regex overhead |
| Split by delimiter | `text.split(",")` or `re.split(r"\s+", text)` | If multi-character or complex pattern → regex; if simple literal → `.split()` |
| Find/replace one value | `text.replace("old", "new")` | Simpler, faster than regex for literal strings |
| Extract structured data (emails, phones, IDs) | ✅ Regex | Only regex can express flexible patterns |
| Validate format (email, phone, URL) | ✅ Regex or dedicated validator library | Regex is concise; libraries are more accurate |
| Case-insensitive search | `"text".lower().find("search")` or `re.search(r"text", "Text", re.I)` | For simple cases, `.lower()` is faster |

### 9.4 Python vs TS Regex Performance: Quick Benchmark Guide

```python
# === Python benchmark (use timeit!) ===
import timeit

pattern = re.compile(r"[a-z]+@[a-z]+\.[a-z]+")
test_text = "user@example.com domain.org sender@test.io"

timeit.timeit(lambda: pattern.findall(test_text), number=10000)
# ~ 0.5 seconds for 10K iterations (compiled pattern ✓)

# Without compile (slower):
timeit.timeit(lambda: re.findall(r"[a-z]+@[a-z]+\.[a-z]+", test_text), number=10000)
# ~ 0.8 seconds (pattern compiled every time! 😫)
```

---

## 10. Key Notes & Important Factors

### Critical Regex Differences from TypeScript You Must Know as a TS Dev

| Concept | TypeScript | Python | Common Mistake by TS Devs |
|---------|-----------|--------|--------------------------|
| **Raw string** | Regex literals `/pattern/` are clean; `new RegExp("...")` needs double-escape for `\d`, `\s`, etc. | ✅ Always use raw strings: `r"\d{3}"` — no double-backslash needed! | Writing `"\\d"` instead of `r"\d"` in Python (works but ugly) |
| **Lookbehind** | Fixed-width only: `(?<=\d)` ✓, `(?<=\d\d?)` ✗ (variable width) | Variable width allowed in stdlib `re`: `(?<=\d+)` ✓ | Trying variable-width lookbehinds — TS rejects them at compile time! |
| **Verbose mode** | ❌ Not supported | ✅ `re.VERBOSE` / inline `(?x)` — ignore whitespace & add comments! | Assuming TS supports it (it doesn't) |
| **Unicode \w, \s, \d** | ASCII only (no accented letters in `\w`) | Py3.12+: Unicode-aware (\w matches CJK, accented chars too!) | Assuming Python \w = same as TS \w — they differ! |
| **String methods vs module** | Methods on all strings: `str.match()`, `str.replace()` etc. | Separate module: `import re; re.search()`, `re.sub()`, etc. | Trying to use `.match()` on Python strings (doesn't exist!) |
| **Replace backreferences** | `$1`, `$2`, `$&` (full match), `$'` (after match), `` $`` ``(before) | `\1`, `\2`, `\g<name>`, `\g<number>` | Using `$1` in Python's `re.sub()` replacement string — wrong! Use `\1` instead! |
| **Sticky flag** | ✅ `y` flag — only matches at `lastIndex` position | ❌ Not supported | Looking for sticky behavior in Python |
| **Unicode \p{L} property escapes** | ✅ `/\\p{Extended_Pictographic}/u` (ES2018+) | ⚠️ Need third-party `regex` module (not stdlib re!) | Assuming Python's stdlib `re` supports `\p{}` — it doesn't! |
| **Branch reset groups** | ❌ Not supported | ⚠️ Third-party `regex` only (`(?|...)`) | Using in other languages' regex engines |
| **Conditional patterns** | ❌ Not supported | ⚠️ Third-party `regex` module only `(?(name)yes|no)` | Looking for this in stdlib re or TS RegExp |
| **Possessive quantifiers** | ❌ Not supported | ⚠️ Third-party `regex` module only (`*+`, `++`, `{n,m}+`) | Using as a fix for backtracking in other engines |

### Key Notes

1. **Always prefer Python's raw strings (`r"..."`) or TypeScript's regex literals (`/pattern/`)** — they eliminate double-escaping confusion. Writing `"\\d{3}"` (non-raw) is error-prone and hard to read.

2. **Lookbehind limitation in TypeScript**: TypeScript's lookbehind `(?<=...)` must be fixed-width. If you need variable-width lookbehinds, Python's stdlib `re` supports it natively; TS requires a different approach (regex or restructure logic).

3. **Named group syntax differs**: Python uses `(?P<name>...)`, TypeScript uses `(?<name>...)`. This is the #1 compatibility issue when sharing regex patterns between languages.

4. **Replace backreference syntax differs**: Python `\1` / `\g<name>`, TypeScript `$1` / `$&` / `` $' `` in replacement strings. Never mix them!

5. **Python's `\w`, `\s`, `\d` are Unicode-aware** (Py3.12+), while TypeScript's are ASCII-only. This means `[a-z]` won't match "é" in TS but will in Python if using the `regex` module with `\p{L}`.

6. **Catastrophic backtracking is a real threat** in both languages. Both use backtracking NFA engines. If your pattern has nested quantifiers like `(a+)+`, rewrite it — both Python and TS will suffer exponential time.

7. **Python `re` module methods operate on patterns, not strings**. In TypeScript, regex methods live on String objects (`.match()`, `.replace()`, etc.). The mental model is different: Python = `re.search(pattern, string)` vs TS = `string.match(regex)`.

8. **For production email/phone validation, consider using a library** rather than home-rolled regex. Regex can't fully validate RFC 5322 emails or international phone formats correctly.

### Utility Patterns Quick Reference

| Goal | Python Pattern | TypeScript Equivalent | Notes |
|------|---------------|---------------------|-------|
| Word boundary match | `\bword\b` | Same | Depends on \w definition (Unicode vs ASCII) |
| Match entire string | `re.fullmatch(pattern, text)` | `/^pattern$/` | Python has built-in method; TS needs anchors |
| Case-insensitive search | `re.search(r"pat", text, re.I)` | `text.match(/pat/i)` | Same concept, different API |
| Extract first match | `re.search(pattern, text).group()` | `text.match(regex)?.[0]` | Python returns object; TS returns array or null |
| Replace all matches | `re.sub(r"p", "r", text)` (all) / `.sub(r"p", "r", text, count=N)` (N times) | `"text".replace(/p/g, "r")` / `.replace("p", "r")` (first only) | Both support per-call vs global replacement |
| Split by complex pattern | `re.split(r"\s+", text)` | `"text".split(/\s+/)` | Python includes captured groups in split result; TS discards them! |

---

## 11. References

### Official Documentation

| Resource | URL | What It Covers |
|----------|-----|---------------|
| Python `re` docs | [docs.python.org/3/library/re.html](https://docs.python.org/3/library/re.html) | Complete regex API for Python's stdlib re module |
| Python `regex` (third-party) | [github.com/mrabarnett/mrab-regex](https://github.com/mrabarnett/mrab-regex) | Advanced features: atomic groups, conditionals, possessive quantifiers, recursion |
| TypeScript RegExp docs | [developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) | Complete ES2018+ regex feature reference |
| ECMAScript Regex Spec | [tc39.es/ecma262/multipage/text-processing.html#sec-patterns](https://tc39.es/ecma262/multipage/text-processing.html#sec-patterns) | Formal spec for TS regex engine behavior |
| Python PEP 616 (String Methods) | [peps.python.org/pep-0616/](https://peps.python.org/pep-0616/) | `str.removeprefix()`, etc. — related to string manipulation |
| "Regular Expressions" by Jan Goyvaerts | [regexone.com](https://regexone.com/) | Interactive regex tutorial (language-agnostic concepts) |
| Regex101.com | [regex101.com](https://regex101.com/) | Interactive regex tester with Python/JS modes |

### Key Comparisons to Read Further

- **Python vs JS regex differences**: [RegexBuddy's comparison page](https://www.regexbuddy.com/compare.html)
- **Catastrophic backtracking explained**: [What Every Programmer Should Know About Regular Expressions (regular-expressions.info)](https://www.regular-expressions.info/catastrophic.html)
- **Unicode in regex**: [Python 3.12 Unicode-aware regex changes](https://peps.python.org/pep-0616/)

---

## 12. Quizzes

### Quiz 1: Pattern Matching (Easy) 🟢

**Which Python pattern matches the same strings as the TypeScript regex `/^\d{3}-\d{2}-\d{4}$/`?**

A) `re.search(r"\d{3}-\d{2}-\d{4}", text)`
B) `re.search(r"^\\d{3}-\\d{2}-\\d{4}$", text)`
C) `re.search(r"^\d{3}-\d{2}-\d{4}$", text)`
D) `re.search(r"^\d{3}\-\d{2}\-\d{4}$", text)`

<details>
<summary><b>Click to reveal answer</b></summary>

**C** — In Python, `\d` inside a regular string needs double-escaping (`"\\d"`), but with raw strings `r"\d"` is correct. Pattern C uses raw string so backslashes are literal ✅.

B would also work (escaped in non-raw string), but C is the preferred Pythonic approach using `r""`.

A is wrong — missing `^` and `$` anchors, so it matches SSNs anywhere in the string (not just full-string match).

</details>

---

### Quiz 2: Lookahead & Lookbehind (Medium) 🟡

**What does each pattern output?**

<details>
<summary><b>Q1:</b> `re.findall(r"\d+(?=\$)", "$9.99 and $15")`</summary>

**["9", "15"]** — The lookahead `(?=\$)` asserts a dollar sign follows the digits but doesn't consume it. So only the digit characters are captured.

</details>

<details>
<summary><b>Q2:</b> What's the difference between Python and TS regex for this?</summary>

Python: `re.search(r"(?<=\$)\d+", "$99")`  — variable-width lookbehind ✅ (stdlib re allows it)
TS: `/(?<=\$)\d+/` — fixed-width lookbehind only ❌ in many JS engines

Python's stdlib `re` is more permissive with lookbehinds. TS/ECMAScript requires fixed-width.

</details>

<details>
<summary><b>Q3:</b> What does this output?</summary>

```python
import re
text = "abc123def456"
re.findall(r"(\d+)", text)  # What are the captured groups?
```

**[("123",), ("456",)]** — `findall()` returns captured group contents when groups are present. Without groups, it would return ["123", "456"] (same result but as strings). With groups, each item is a tuple of group captures!

</details>

---

### Quiz 3: Code Tracing (Hard) 🔴

**What does each snippet output?**

<details>
<summary><b>Q1:</b></summary>

```python
import re
text = "hello world hello"
result = re.sub(r"(?i)hello", "hi", text)
print(result)
```

**Answer: `hi world hi`** — The inline flag `(?i)` makes it case-insensitive. Both "hello" occurrences are replaced with "hi". The `(?i)` flag can be placed anywhere in the pattern (not just at start).

</details>

<details>
<summary><b>Q2:</b></summary>

```python
import re
text = "<div>Hello</div><span>World</span>"

# Pattern A:
result_a = re.search(r"<([a-z]+)>.*?</\1>", text)

# Pattern B:
result_b = re.search(r"<(?P<tag>[a-z]+)>.*?</(?P=tag)>", text)

print(result_a.group())   # Line A
print(result_b.group())   # Line B
```

**Answer (Line A): `<div>Hello</div>`** — `\1` backreferences the first capture group. It matches "div" and then asserts the closing tag is also "div" (not any other tag).

**Answer (Line B): Same result** — Named groups `(?P<tag>...)` and `(?P=tag)` achieve the same as `\1`. This is the more readable version for complex patterns.

</details>

<details>
<summary><b>Q3:</b></summary>

```typescript
// What does this output?
const text = "User: john_doe, email: John.Doe@Example.COM";

// Pattern 1:
console.log(text.match(/john\.doe@example\.com/i));

// Pattern 2:  
console.log(text.match(/John\.Doe@Example\.COM/));
```

**Answer (Pattern 1): `["John.Doe@Example.COM"]`** — Case-insensitive flag matches the email despite different casing. Returns the actual matched substring (not the pattern).

**Answer (Pattern 2): `["John.Doe@Example.COM"]`** — Exact case match also works because the text has that exact casing. Both patterns return the same result here!

</details>

<details>
<summary><b>Q4:</b></summary>

```python
import re

text = "apple,banana,cherry,date,elderberry"

result1 = re.split(r",", text)
result2 = re.split(r"(?<=,)", text)  # Lookbehind after each comma
result3 = re.split(r",(?=cherry)", text)  # Split before 'cherry' only

print(result1)   # Line A
print(result2)   # Line B
print(result3)   # Line C
```

**Answer (Line A): `["apple", "banana", "cherry", "date", "elderberry"]`** — Standard split by comma.

**Answer (Line B): `["apple,", "banana,", "cherry,", "date,", "elderberry"]`** — The lookbehind `(?<=,)` asserts a comma precedes the split position, effectively keeping the comma in the result.

**Answer (Line C): `["apple,banana,", "cherry,date,elderberry"]`** — Split only at the comma immediately before "cherry" (the lookahead `(?=cherry)` ensures it).

</details>

---

### Quiz 4: True/False (Quick Knowledge Check) 🟡

| # | Statement | True / False |
|---|----------|-------------|
| 1 | Python's `\w` matches Unicode letters by default in the stdlib `re` module. | — |
| 2 | TypeScript RegExp supports variable-width lookbehinds like `(?<=\d+)`. | — |
| 3 | In both Python and TS, `*?` means "lazy (minimal) match of 0 or more". | — |
| 4 | Python's `re.findall()` with capture groups returns a list of tuples. | — |
| 5 | `re.match()` in Python searches for the pattern anywhere in the string. | — |
| 6 | TypeScript's `/pattern/s` flag makes `.` match newline characters (dotall). | — |
| 7 | Both Python's `re.sub()` and TS's `str.replace()` accept a callback function as replacement. | — |
| 8 | The `\b` word boundary in both languages considers `_` as part of a word. | — |

<details>
<summary><b>Click to reveal all answers</b></summary>

1. **False** — In Python stdlib `re`, `\w` matches `[a-zA-Z0-9_]` (ASCII only). The third-party `regex` module or Python 3.12+ with Unicode flags extends this, but by default it's ASCII.
2. **False** — TypeScript/ECMAScript lookbehinds must be fixed-width. Use `(?<=\d)` not `(?<=\d+)`.
3. **True** ✅ — Lazy quantifiers work identically in both languages.
4. **True** ✅ — When groups are present, `findall()` returns tuples of group captures (not full matches).
5. **False** — `re.match()` only matches at the START of the string. Use `re.search()` for anywhere-in-string matching.
6. **True** ✅ — The `s` flag (dotall) makes `.` match newlines in both Python and TS.
7. **True** ✅ — Both support: Python's `re.sub(pattern, lambda m: ..., text)` and TS's `"text".replace(regex, (match) => ...)`.
8. **True** ✅ — `_` is considered a word character (`\w`) in both languages. `\b` sits between `\w` and `\W` boundaries.

</details>

---

### Quiz 5: Pattern Building Challenge (Hard) 🔴

**Write regex patterns (both Python and TypeScript) for each requirement:**

<details>
<summary><b>Q1:</b> Match a password that is 8-20 chars, has at least one uppercase, one lowercase, one digit.</summary>

```python
# Python:
r'^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)[A-Za-z\d]{8,20}$'

# TypeScript:
/^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)[A-Za-z\d]{8,20}$/
```

All three lookahead assertions must be at position 0 (start), then the character class matches the password.

</details>

<details>
<summary><b>Q2:</b> Extract only the domain from emails in text: "Email john@example.com and jane@test.org".</summary>

```python
# Python:
re.findall(r'@\K[a-z]+(?=\.)', "Email john@example.com and jane@test.org", re.IGNORECASE)
# But \K is not supported in Python stdlib! Use lookahead instead:

re.findall(r'@([a-z]+)\.', "Email john@example.com and jane@test.org", re.IGNORECASE)
# ["example", "test"]

# TypeScript:
"Email john@example.com and jane@test.org".match(/@([a-z]+)\./gi)?.map(m => m.slice(1))
// ["example", "test"]
```

Both extract the domain name before the dot using a capture group (non-capturing the @).

</details>

<details>
<summary><b>Q3:</b> Validate ISBN-10: 10 digits, last can be X.</summary>

```python
# Python:
r'^\d{9}[\dX]$'

# TypeScript:
/^\d{9}[\dX]$/
```

ISBN-10: exactly 10 characters, first 9 are digits, last is digit or 'X'. Simple and unambiguous — no backtracking risk!

</details>

---

> **Next:** This module covers every regex pattern, method, flag, and feature across both languages. Combine with [Module 17 — Data Types In-Depth](./17-data-types-in-depth.md) to see how regex integrates with string manipulation, and [Module 18 — Keywords Deep Dive](./18-keywords-deep-dive.md) for the broader language syntax landscape.

---

## 13. 🆕 Extended TS vs Python Examples Deep Dive

This section provides massive side-by-side code comparisons across every major regex operation. Each example shows exact Python and TypeScript equivalents with output, so you can transfer knowledge between languages effortlessly.

### 13.1 Email Parsing & Validation

```python
import re

# === Extract email from mixed text ===
text = "Email me at alice@example.com or bob@sub.domain.org for help"
emails = re.findall(r"[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}", text)
print(emails)  # ['alice@example.com', 'bob@sub.domain.org']

# === Validate with detailed feedback ===
def validate_email(email: str) -> dict:
    """Returns detailed email validation info."""
    pattern = re.compile(
        r"^(?P<local>[\w.-]+)@(?P<domain>[\w.-]+)\.(?P<tld>[a-zA-Z]{2,})$",
        re.VERBOSE
    )
    m = pattern.fullmatch(email)
    if not m:
        return {"valid": False, "error": "Invalid email format"}
    
    local_part = m.group("local")
    domain = m.group("domain")
    tld = m.group("tld")
    
    checks = {
        "valid": True,
        "local_length": len(local_part),
        "has_dots": "." in local_part,
        "starts_with_dot": local_part.startswith("."),  # invalid per RFC
        "domain_tld": tld,
    }
    if checks["starts_with_dot"]:
        checks["valid"] = False
        checks["error"] = "Local part cannot start with dot"
    return checks

print(validate_email("alice@example.com"))
# {'valid': True, 'local_length': 5, 'has_dots': False, ...}

print(validate_email(".invalid@test.org"))
# {'valid': False, 'error': 'Local part cannot start with dot'}
```

```typescript
// === Extract email from mixed text ===
const text = "Email me at alice@example.com or bob@sub.domain.org for help";
const emails = text.match(/[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}/g);
console.log(emails);  // ['alice@example.com', 'bob@sub.domain.org']

// === Validate with detailed feedback ===
function validateEmail(email: string): { valid: boolean; error?: string; localLength?: number; tld?: string } {
    const pattern = /^(?<local>[\w.-]+)@(?<domain>[\w.-]+)\.(?<tld>[a-zA-Z]{2,})$/;
    const m = email.match(pattern);
    
    if (!m) {
        return { valid: false, error: "Invalid email format" };
    }
    
    const localPart = m.groups?.local ?? "";
    const tld = m.groups?.tld ?? "";
    
    return {
        valid: !localPart.startsWith("."),
        localLength: localPart.length,
        tld: tld,
        error: localPart.startsWith(".") ? "Local part cannot start with dot" : undefined
    };
}

console.log(validateEmail("alice@example.com"));
// { valid: true, localLength: 5, tld: 'com' }

console.log(validateEmail(".invalid@test.org"));
// { valid: false, error: 'Local part cannot start with dot' }
```

### 13.2 URL Parsing & Component Extraction

```python
import re
from urllib.parse import urlparse

# === Extract all URLs from text (both http and https) ===
text = "Check https://example.com/path?q=1 or http://test.org/api/v2/docs"
urls = re.findall(r"https?://[\w.-]+(?:/[\w./%-]*)*(?:\?[\w=&%-]*)?(?:#[\w-]*)?", text)
print(urls)  # ['https://example.com/path?q=1', 'http://test.org/api/v2/docs']

# === Parse URL components (manual extraction) ===
def parse_url_components(url: str) -> dict:
    """Extract protocol, domain, path, query from URL."""
    pattern = re.compile(
        r"^(?P<proto>https?)://"
        r"(?P<user>[^:@/]+)?:?"
        r"(?P<pass>[^@/]*)?@"   # optional user:pass@
        r"(?P<host>[\w.-]+)"
        r"(?::(?P<port>\d+))?"  # optional port
        r"(?P<path>/[\w./%-]*)?"  # optional path
        r"(?:\?(?P<query>[\w=&%-]*))?$",  # optional query
        re.VERBOSE
    )
    m = pattern.fullmatch(url)
    if not m:
        return {"error": "Invalid URL format"}
    
    result = m.groupdict()
    # Fix: groupdict returns None for unmatched groups
    return {k: (v or "") for k, v in result.items()}

print(parse_url_components("https://user:pass@example.com:8080/path?q=1"))
# {'proto': 'https', 'user': 'user', 'pass': 'pass', 'host': 'example.com', 
#  'port': '8080', 'path': '/path', 'query': 'q=1'}

# === Validate HTTP status code in log ===
log_text = "GET /api 200 OK | POST /login 401 Unauthorized | GET /home 304 Not Modified"
status_codes = re.findall(r"\d{3}", log_text)
print(status_codes)  # ['200', '401', '304']

# Filter valid HTTP status (100-599):
valid_status = [code for code in status_codes if 100 <= int(code) < 600]
print(valid_status)  # ['200', '401', '304']
```

```typescript
// === Extract all URLs from text ===
const text = "Check https://example.com/path?q=1 or http://test.org/api/v2/docs";
const urls = text.match(/https?:\/\/[\w.-]+(?:\/[\w.\/%-]*)*(?:\?[\w=&%-]*)?(?:#[\w-]*)?/g);
console.log(urls);  // ['https://example.com/path?q=1', 'http://test.org/api/v2/docs']

// === Parse URL components (manual extraction) ===
function parseUrlComponents(url: string): Record<string, string> {
    const pattern = /^(?<proto>https?):\/(?<user>[^:@\/]+)?:?(?<pass>[^@\/]*)?@(?<host>[\w.-]+)(?::(?<port>\d+))?(?<path>\/[\w.\/%-]*)?(?:\?(?<query>[\w=&%-]*))?$/;
    const m = url.match(pattern);
    
    if (!m || !m.groups) {
        return { error: "Invalid URL format" };
    }
    
    // Convert groups object to regular object, defaulting None/null to ""
    return Object.fromEntries(
        Object.entries(m.groups).map(([k, v]) => [k, v ?? ""])
    );
}

console.log(parseUrlComponents("https://user:pass@example.com:8080/path?q=1"));
// { proto: 'https', user: 'user', pass: 'pass', host: 'example.com', port: '8080', path: '/path', query: 'q=1' }

// === Validate HTTP status code in log ===
const logText = "GET /api 200 OK | POST /login 401 Unauthorized | GET /home 304 Not Modified";
const statusCodes = logText.match(/\d{3}/g) ?? [];
console.log(statusCodes);  // ['200', '401', '304']

// Filter valid HTTP status (100-599):
const validStatus = statusCodes.filter(code => {
    const n = parseInt(code);
    return n >= 100 && n < 600;
});
console.log(validStatus);  // ['200', '401', '304']
```

### 13.3 JSON String Extraction (No Parser Needed for Simple Cases)

```python
import re

# === Extract key-value pairs from simple JSON ===
json_text = '{"name": "Alice", "age": 30, "active": true}'

# Match string values: "key": "value"
string_pairs = re.findall(r'"([\w]+)"\s*:\s*"([^"]*)"', json_text)
print(string_pairs)  # [('name', 'Alice'), ('age', '30'), ('active', 'true')]
dict(string_pairs)  # {'name': 'Alice', 'age': '30', 'active': 'true'}

# Match all quoted keys
keys = re.findall(r'"([\w]+)"\s*:', json_text)
print(keys)  # ['name', 'age', 'active']

# Extract numeric values
numbers = re.findall(r'"([\w]+)"\s*:\s*(-?\d+\.?\d*)', json_text)
print(numbers)  # [('age', '30')]

# === Extract nested JSON array items (simple case) ===
count = re.findall(r'\{(.*?)\}', '{"a":1}, {"b":2}')
print(count)  # ['"a":1', '"b":2'] — matches inner content of each {}

# WARNING: this fails for deeply nested JSON! Use a parser for complex cases.
```

```typescript
// === Extract key-value pairs from simple JSON ===
const jsonText = '{"name": "Alice", "age": 30, "active": true}';

// Match string values: "key": "value"
const stringPairs = jsonText.match(/"([\w]+)"\s*:\s*"([^"]*)"/g);
console.log(stringPairs);
// ['"name": "Alice"', '"age": "30"', '"active": "true"']

// Using matchAll for structured data:
const matches = [...jsonText.matchAll(/"([\w]+)"\s*:\s*"([^"]*)"/g)];
for (const [fullMatch, key, value] of matches) {
    console.log(`${key} -> ${value}`);
}
// name -> Alice
// age -> 30
// active -> true

// Match all quoted keys
const keys = jsonText.match(/"([\w]+)"\s*:/g)?.map(m => m.replace(/"|\s*:/g, ''));
console.log(keys);  // ['name', 'age', 'active']

// Extract numeric values
const numbers = [...jsonText.matchAll(/"([\w]+)"\s*:\s*(-?\d+\.?\d*)/g)];
console.log(numbers.map(m => [m[1], m[2]]));  // [['age', '30']]
```

### 13.4 HTML Parsing (Basic Extraction)

```python
import re

# === Extract all href attributes from anchor tags ===
html = '''
<a href="https://example.com" class="link">Home</a>
<a href="https://test.org/page?id=1" target="_blank">Test</a>
<img src="https://img.com/photo.jpg" alt="Photo">
'''

# Extract href URLs (handles both single and double quotes)
hrefs = re.findall(r'<a\s+[^>]*href=["\']([^"\']+)["\'][^>]*>', html, re.IGNORECASE)
print(hrefs)  # ['https://example.com', 'https://test.org/page?id=1']

# === Extract all image URLs ===
imgs = re.findall(r'<img[^>]+src=["\']([^"\']+)["\']', html, re.IGNORECASE)
print(imgs)  # ['https://img.com/photo.jpg']

# === Extract text content from HTML tags (basic) ===
texts = re.findall(r'><([^<]*)>', html)
print(texts)  # ['', 'Home', '', 'Test', '']

# === Get all CSS class names from HTML ===
classes = re.findall(r'class=["\']([^"\']+)["\']', html, re.IGNORECASE)
print(classes)  # ['link', '_blank']

# === Extract color values from CSS ===
css = "color: #ff5733; background-color: rgb(255, 87, 51); font-size: 14px"
hex_colors = re.findall(r"#[0-9a-fA-F]{6}", css)
print(hex_colors)  # ['#ff5733']
rbg_values = re.findall(r"rgb\((\d+),\s*(\d+),\s*(\d+)\)", css)
print(rbg_values)  # [('255', '87', '51')]
```

```typescript
// === Extract all href attributes from anchor tags ===
const html = `<a href="https://example.com" class="link">Home</a>
<a href="https://test.org/page?id=1" target="_blank">Test</a>
<img src="https://img.com/photo.jpg" alt="Photo">`;

// Extract href URLs (handles both single and double quotes)
const hrefs = html.match(/<a\s+[^>]*href=["']([^"']+)["'][^>]*>/gi)?.map(m => {
    return m.match(/href=["']([^"']+)["']/i)?.[1];
}).filter(Boolean);
console.log(hrefs);  // ['https://example.com', 'https://test.org/page?id=1']

// === Extract all image URLs ===
const imgs = html.match(/<img[^>]+src=["']([^"']+)["']/gi)?.map(m => {
    return m.match(/src=["']([^"']+)["']/i)?.[1];
}).filter(Boolean);
console.log(imgs);  // ['https://img.com/photo.jpg']

// === Extract text content from HTML tags (basic) ===
const texts = html.match(/><([^<]*)>/g)?.map(t => t.replace(/</?[^>]*>/g, ''));
console.log(texts.filter(Boolean));  // ['Home', 'Test']

// === Get all CSS class names from HTML ===
const classes = html.match(/class=["']([^"']+)["']/gi)?.map(m => m.replace(/["'class:=]/g, ''));
console.log(classes);  // ['link', '_blank']

// === Extract color values from CSS ===
const css = "color: #ff5733; background-color: rgb(255, 87, 51); font-size: 14px";
const hexColors = css.match(/#[0-9a-fA-F]{6}/g) ?? [];
console.log(hexColors);  // ['#ff5733']
const rgbValues = [...css.matchAll(/rgb\((\d+),\s*(\d+),\s*(\d+)\)/g)]
    .map(m => [m[1], m[2], m[3]]);
console.log(rgbValues);  // [['255', '87', '51']]
```

### 13.5 Date & Time Parsing

```python
import re
from datetime import datetime

text = "Meeting on 2025-01-15 at 14:30, or 2025/03/22 at 9:00 AM"

# ISO date (YYYY-MM-DD)
is_dates = re.findall(r"\d{4}-\d{2}-\d{2}", text)
print(is_dates)  # ['2025-01-15']

# US date (MM/DD/YYYY or MM-DD-YYYY)
us_dates = re.findall(r"(\d{1,2})[-/](\d{1,2})[-/](\d{4})", text)
print(us_dates)  # [('03', '22', '2025')]

# Time formats: 14:30, 9:00 AM, 09:00:00
times = re.findall(r"(\d{1,2}):(\d{2})(?::(\d{2}))?\s*(AM|PM)?", text, re.I)
print(times)  # [('14', '30', None, None), ('9', '00', None, 'AM')]

# === Complex: Extract datetime with flexible formats ===
def parse_flexible_datetime(text: str) -> list:
    """Extract all date-time combinations from text."""
    # Pattern matches: YYYY-MM-DD HH:MM or YYYY/MM/DD at H:MM AM/PM
    pattern = re.compile(
        r"(\d{4})[-/](\d{1,2})[-/](\d{1,2})"
        r"(?:\s+(\d{1,2}):(\d{2})(?::(\d{2}))?)?"
        r"(\s+(AM|PM))?(?:[Pp]M)?",
        re.IGNORECASE
    )
    results = []
    for m in pattern.finditer(text):
        year, month, day = int(m.group(1)), int(m.group(2)), int(m.group(3))
        hour = int(m.group(4) or 0)
        minute = int(m.group(5) or 0)
        second = int(m.group(6) or 0)
        ampm = (m.group(7) or "").upper().strip()
        
        # Handle AM/PM
        if ampm == "PM" and hour != 12:
            hour += 12
        elif ampm == "AM" and hour == 12:
            hour = 0
        
        results.append({
            "date": f"{year:04d}-{month:02d}-{day:02d}",
            "time_24h": f"{hour:02d}:{minute:02d}:{second:02d}"
        })
    return results

print(parse_flexible_datetime(text))
# [{'date': '2025-01-15', 'time_24h': '14:30:00'}, {'date': '2025-03-22', 'time_24h': '09:00:00'}]
```

```typescript
// === Date & Time Parsing ===
const text = "Meeting on 2025-01-15 at 14:30, or 2025/03/22 at 9:00 AM";

// ISO date (YYYY-MM-DD)
const isDates = text.match(/\d{4}-\d{2}-\d{2}/g);
console.log(isDates);  // ['2025-01-15']

// US date (MM/DD/YYYY or MM-DD-YYYY)
const usDates = [...text.matchAll(/(\d{1,2})[-/](\d{1,2})[-/](\d{4})/g)];
console.log(usDates.map(m => [m[1], m[2], m[3]]));  // [['03', '22', '2025']]

// Time formats: 14:30, 9:00 AM, 09:00:00
const times = [...text.matchAll(/(\d{1,2}):(\d{2})(?::(\d{2}))?\s*(AM|PM)?/gi)];
console.log(times.map(m => [m[1], m[2], m[3] ?? null, m[4]]));
// [['14', '30', null, undefined], ['9', '00', null, 'AM']]

// === Complex: Extract datetime with flexible formats ===
function parseFlexibleDatetime(text: string): Array<{date: string; time24h: string}> {
    const pattern = /(\d{4})[-/](\d{1,2})[-/](\d{1,2})(?:\s+(\d{1,2}):(\d{2})(?::(\d{2}))?)?(?:\s+(AM|PM))?/i;
    const results: Array<{date: string; time24h: string}> = [];
    
    for (const m of text.matchAll(pattern)) {
        const [_, yearStr, monthStr, dayStr, hourStr, minStr, secStr] = m;
        const year = parseInt(yearStr);
        const month = parseInt(monthStr);
        const day = parseInt(dayStr);
        let hour = parseInt(hourStr || "0");
        const minute = parseInt(minStr || "0");
        const second = parseInt(secStr || "0");
        const ampm = (m[6] || "").toUpperCase().trim();
        
        // Handle AM/PM
        if (ampm === "PM" && hour !== 12) hour += 12;
        else if (ampm === "AM" && hour === 12) hour = 0;
        
        results.push({
            date: `${year.toString().padStart(4, '0')}-${month.toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`,
            time24h: `${hour.toString().padStart(2, '0')}:${minute.toString().padStart(2, '0')}:${second.toString().padStart(2, '0')}`
        });
    }
    return results;
}

console.log(parseFlexibleDatetime(text));
// [{date: '2025-01-15', time24h: '14:30:00'}, {date: '2025-03-22', time24h: '09:00:00'}]
```

### 13.6 Semantic Version Parsing (SemVer)

```python
import re

# === SemVer pattern: major.minor.patch-prerelease+build ===
semver_pattern = re.compile(
    r"^(?P<major>0|[1-9]\d*)"
    r"\.(?P<minor>0|[1-9]\d*)"
    r"\.(?P<patch>0|[1-9]\d*)"
    r"(?:-(?P<pre>[0-9A-Za-z-]+))?(?:\+(?P<build>[0-9A-Za-z-]+))?",
    re.VERBOSE
)

def parse_semver(version: str) -> dict | None:
    """Parse a SemVer string into components."""
    m = semver_pattern.fullmatch(version)
    if not m:
        return None
    
    groups = m.groupdict()
    groups['major'] = int(groups['major'])
    groups['minor'] = int(groups['minor'])
    groups['patch'] = int(groups['patch'])
    return groups  # pre and build may be None

print(parse_semver("1.2.3"))           # {'major': 1, 'minor': 2, 'patch': 3, 'pre': None, 'build': None}
print(parse_semver("2.0.0-alpha.1"))   # includes pre-release
print(parse_semver("3.1.4+build.42"))   # includes build metadata
print(parse_semver("1.2"))              # None — patch required!

# === Extract SemVer from changelog text ===
changelog = "v1.0.0 (2024-01) -> v1.1.0 (2024-03) -> v2.0.0-beta.1 (2025-06)"
versions = [parse_semver(v.lstrip('v')) for v in re.findall(r'[\d]+\.[\d]+\.[\d]+(?:-[\w.]+)?(?:\+[\w.]+)?', changelog)]
print(versions)
# [{'major': 1, ...}, {'major': 1, 'minor': 1, ...}, {'major': 2, ...}]
```

```typescript
// === SemVer pattern: major.minor.patch-prerelease+build ===
const SEMVER_PATTERN = /^(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)(?:-(?P<pre>[0-9A-Za-z-]+))?(?:\+(?P<build>[0-9A-Za-z-]+))?$/;

interface SemVer {
    major: number;
    minor: number;
    patch: number;
    pre?: string;
    build?: string;
}

function parseSemVer(version: string): SemVer | null {
    const m = version.match(SEMVER_PATTERN);
    if (!m || !m.groups) return null;
    
    return {
        major: parseInt(m.groups.major),
        minor: parseInt(m.groups.minor),
        patch: parseInt(m.groups.patch),
        pre: m.groups.pre ?? undefined,
        build: m.groups.build ?? undefined
    };
}

console.log(parseSemVer("1.2.3"));           // { major: 1, minor: 2, patch: 3 }
console.log(parseSemVer("2.0.0-alpha.1"));   // { major: 2, minor: 0, patch: 0, pre: 'alpha.1' }
console.log(parseSemVer("3.1.4+build.42"));   // { major: 3, minor: 1, patch: 4, build: 'build.42' }
console.log(parseSemVer("1.2"));              // null — patch required!

// === Extract SemVer from changelog text ===
const changelog = "v1.0.0 (2024-01) -> v1.1.0 (2024-03) -> v2.0.0-beta.1 (2025-06)";
const versions = changelog.match(/\d+\.\d+\.\d+(?:-[\w.]+)?(?:\+[\w.]+)?/g)
    ?.map(v => parseSemVer(v.replace(/^v/, '')))
    .filter(Boolean);
console.log(versions);  // [SemVer objects for each version]
```

### 13.7 Git Diff / Log Parsing

```python
import re

# === Parse git log output ===
git_log = """
commit abc123def456
Author: Alice <alice@example.com>
Date:   Mon Jan 15 14:30:00 2025 +0000

    feat: add user authentication module

commit def789ghi012
Author: Bob <bob@test.org>
Date:   Tue Jan 16 09:15:00 2025 +0000

    fix: resolve login redirect bug
"""

# Extract commits
commits = re.findall(
    r"commit\s+([0-9a-f]{7,40})\s+"
    r"Author:\s+(.+?)\s+<(.+?)>\s+"
    r"Date:\s+(.+?)\s+\+\d{4}\s+"
    r"Date:\s+(.+?)[\r\n]{2}([\s\S]*?)$",
    git_log, re.MULTILINE
)
# Simpler approach:
commit_ids = re.findall(r"commit\s+([0-9a-f]+)", git_log)
authors = re.findall(r"Author:\s+(.+?)\s+<(.+?)>", git_log)
messages = re.findall(r"Date:\s+.+?[\r\n]{2}    (.+)", git_log, re.MULTILINE)

print("Commit IDs:", commit_ids)
# ['abc123def456', 'def789ghi012']
print("Authors:", authors)
# [('Alice', 'alice@example.com'), ('Bob', 'bob@test.org')]
print("Messages:", messages)
# ['feat: add user authentication module', 'fix: resolve login redirect bug']

# === Parse git diff output ===
git_diff = """
diff --git a/src/main.py b/src/main.py
index abc123..def456 100644
--- a/src/main.py
+++ b/src/main.py
@@ -10,3 +10,4 @@ def main():
+    print('hello')
-    print('world')
     return True
"""

# Extract file paths
files = re.findall(r"diff --git a/(.+) b/(.+)", git_diff)
print(files)  # [['src/main.py', 'src/main.py']]

# Extract added lines (+ prefix)
added_lines = re.findall(r"^\+(.*)$", git_diff, re.MULTILINE)
print(added_lines)  # ["    print('hello')"]

# Extract removed lines (- prefix)
removed_lines = re.findall(r"^-(.*)$", git_diff, re.MULTILINE)
print(removed_lines)  # ["    print('world')"]

# Extract hunk header (line numbers change)
hunks = re.findall(r"@@ -(\d+),(\d+) \+(\d+),(\d+) @@", git_diff)
print(hunks)  # [('10', '3', '10', '4')]
```

```typescript
// === Parse git log output ===
const gitLog = `
commit abc123def456
Author: Alice <alice@example.com>
Date:   Mon Jan 15 14:30:00 2025 +0000

    feat: add user authentication module

commit def789ghi012
Author: Bob <bob@test.org>
Date:   Tue Jan 16 09:15:00 2025 +0000

    fix: resolve login redirect bug
`;

const commitIds = (gitLog.match(/commit\s+([0-9a-f]+)/g) ?? [])
    .map(m => m.replace('commit ', ''));
const authors = [...gitLog.matchAll(/Author:\s+(.+?)\s+<(.+?)>/g)]
    .map(m => [m[1], m[2]]);
const messages = [...gitLog.matchAll(/^\s{4}(.+)$/gm)]
    .filter(m => m[1].includes(':'))  // filter to commit messages
    .map(m => m[1]);

console.log('Commit IDs:', commitIds);
// ['abc123def456', 'def789ghi012']
console.log('Authors:', authors);
// [['Alice', 'alice@example.com'], ['Bob', 'bob@test.org']]
console.log('Messages:', messages);
// ['feat: add user authentication module', 'fix: resolve login redirect bug']

// === Parse git diff output ===
const gitDiff = `diff --git a/src/main.py b/src/main.py
index abc123..def456 100644
--- a/src/main.py
+++ b/src/main.py
@@ -10,3 +10,4 @@ def main():
+    print('hello')
-    print('world')
     return True`;

const files = (gitDiff.match(/diff --git a\/(.+) b\/(.+)/g) ?? [])
    .map(m => m.replace(/diff --git a\/(.+) b\/.*/, '$1'));
console.log(files);  // ['src/main.py']

const added = (gitDiff.match(/^\+(.*)$/gm) ?? []).filter(l => l[1] !== '-');
console.log(added.map(l => l.slice(1)));  // ["    print('hello')"]

const removed = gitDiff.match(/^-([^-].*)$/gm);
console.log((removed ?? []).map(l => l.slice(1)));  // ["    print('world')"]

const hunks = (gitDiff.match(/@@ -(\d+),(\d+) \+(\d+),(\d+) @@/g) ?? [])
    .map(m => m.match(/@@ -(\d+),(\d+) \+(\d+),(\d+) @@/)!.slice(1));
console.log(hunks);  // [['10', '3', '10', '4']]
```

### 13.8 Log File Parsing (Complex Real-World)

```python
import re
from typing import TypedDict

class LogEntry(TypedDict):
    timestamp: str
    level: str
    module: str
    message: str

# Common log format:
# [2025-01-15 14:30:45.123] [ERROR] [myapp.auth] Connection timeout after 30s
def parse_log_line(line: str) -> LogEntry | None:
    """Parse a structured log line into components."""
    pattern = re.compile(
        r"\["
        r"(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}(?:\.\d+)?)"
        r"\] "
        r"\[(?P<level>[A-Z]+)\] "
        r"\[(?P<module>[\w.]+)\] "
        r"(?P<message>.+)",
        re.VERBOSE
    )
    m = pattern.match(line)
    if not m:
        return None
    
    return LogEntry(
        timestamp=m.group("timestamp"),
        level=m.group("level"),
        module=m.group("module"),
        message=m.group("message")
    )

# Parse multiple lines from a log file
log_data = """
[2025-01-15 14:30:45.123] [ERROR] [myapp.auth] Connection timeout after 30s
[2025-01-15 14:30:46.456] [INFO] [myapp.api] Retry attempt 1/3
[2025-01-15 14:30:47.789] [WARN] [myapp.db] Slow query: SELECT * FROM users (took 2.5s)
[2025-01-15 14:30:48.000] [DEBUG] [myapp.auth] Token refresh for user_id=12345
"""

entries = []
for line in log_data.strip().split("\n"):
    entry = parse_log_line(line)
    if entry:
        entries.append(entry)

# Filter only ERROR and WARN levels
alerts = [e for e in entries if e["level"] in ("ERROR", "WARN")]
for alert in alerts:
    print(f"[{alert['level']}] {alert['module']}: {alert['message']}")
# [ERROR] myapp.auth: Connection timeout after 30s
# [WARN] myapp.db: Slow query: SELECT * FROM users (took 2.5s)

# === Extract all unique log levels and their counts ===
level_counts = re.findall(r"\[(?P<level>[A-Z]+)\]", log_data)
from collections import Counter
counts = Counter(level_counts)
print(dict(counts))  # {'ERROR': 1, 'INFO': 1, 'WARN': 1, 'DEBUG': 1}

# === Extract all module names ===
modules = re.findall(r"\[(?P<module>\w+\.\w+)\]", log_data)
print(set(modules))  # {'myapp.auth', 'myapp.api', 'myapp.db'}
```

```typescript
// === Log file parsing (complex real-world) ===
interface LogEntry {
    timestamp: string;
    level: string;
    module: string;
    message: string;
}

const LOG_PATTERN = /^\[(?<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}(?:\.\d+)?)\] \[(?<level>[A-Z]+)\] \[(?<module>[\w.]+)\] (?<message>.+)$/;

function parseLogLine(line: string): LogEntry | null {
    const m = line.match(LOG_PATTERN);
    if (!m || !m.groups) return null;
    
    return {
        timestamp: m.groups.timestamp,
        level: m.groups.level,
        module: m.groups.module,
        message: m.groups.message
    };
}

const logData = `
[2025-01-15 14:30:45.123] [ERROR] [myapp.auth] Connection timeout after 30s
[2025-01-15 14:30:46.456] [INFO] [myapp.api] Retry attempt 1/3
[2025-01-15 14:30:47.789] [WARN] [myapp.db] Slow query: SELECT * FROM users (took 2.5s)
[2025-01-15 14:30:48.000] [DEBUG] [myapp.auth] Token refresh for user_id=12345
`;

const entries = logData.strip().split('\n')
    .map(line => parseLogLine(line))
    .filter((e): e is LogEntry => e !== null);

// Filter only ERROR and WARN levels
const alerts = entries.filter(e => e.level === 'ERROR' || e.level === 'WARN');
alerts.forEach(alert => {
    console.log(`[${alert.level}] ${alert.module}: ${alert.message}`);
});
// [ERROR] myapp.auth: Connection timeout after 30s
// [WARN] myapp.db: Slow query: SELECT * FROM users (took 2.5s)

// === Extract all unique log levels and their counts ===
const levelCounts = [...logData.matchAll(/\[(?<level>[A-Z]+)\]/g)]
    .map(m => m.groups?.level);

const counts: Record<string, number> = {};
levelCounts.forEach(level => {
    counts[level] = (counts[level] || 0) + 1;
});
console.log(counts);  // { ERROR: 1, INFO: 1, WARN: 1, DEBUG: 1 }

// === Extract all module names ===
const modules = [...logData.matchAll(/\[(?:[A-Z]+)\] \[(?<module>[\w.]+)\]/g)]
    .map(m => m.groups?.module)
    .filter(Boolean);
console.log(new Set(modules));  // {'myapp.auth', 'myapp.api', 'myapp.db'}
```

### 13.9 SQL Query Pattern Matching

```python
import re

# === Extract all table names from a SQL query ===
sql = "SELECT name, age FROM users WHERE id IN (SELECT user_id FROM orders WHERE status = 'active')"
tables = re.findall(r"FROM\s+([\w_]+)", sql, re.I)
print(tables)  # ['users', 'orders']

# === Extract all column names after SELECT and FROM/JOIN ===
columns_select = re.findall(r"SELECT\s+(.+?)\s+FROM", sql, re.I | re.DOTALL)
print(columns_select)  # ['name, age']

# === Validate SQL-like patterns ===
def validate_sql_select(sql: str) -> bool:
    """Basic validation: must have SELECT ... FROM pattern."""
    pattern = r"^\s*SELECT\s+[^;]+\s+FROM\s+[\w_]+\s*$"
    return bool(re.search(pattern, sql, re.I))

print(validate_sql_select("SELECT * FROM users"))  # True
print(validate_sql_select("INSERT INTO users VALUES (1)"))  # False

# === Extract SQL WHERE conditions ===
def extract_where_conditions(sql: str) -> list:
    """Extract individual conditions from WHERE clause."""
    m = re.search(r"WHERE\s+(.+?)(?:ORDER|GROUP|LIMIT|$)", sql, re.I | re.DOTALL)
    if not m:
        return []
    where_clause = m.group(1)
    # Split on AND (case-insensitive)
    conditions = re.split(r'\s+AND\s+', where_clause, flags=re.I)
    return [c.strip() for c in conditions]

sql_with_where = "SELECT * FROM users WHERE age > 18 AND status = 'active' ORDER BY name"
print(extract_where_conditions(sql_with_where))
# ['age > 18', "status = 'active'"]
```

```typescript
// === Extract all table names from a SQL query ===
const sql = "SELECT name, age FROM users WHERE id IN (SELECT user_id FROM orders WHERE status = 'active')";
const tables = sql.match(/FROM\s+([\w_]+)/gi)?.map(m => m.replace(/^FROM\s+/i, ''));
console.log(tables);  // ['users', 'orders']

// === Validate SQL-like patterns ===
function validateSqlSelect(sql: string): boolean {
    const pattern = /^\s*SELECT\s+[^;]+\s+FROM\s+[\w_]+\s*$/i;
    return pattern.test(sql);
}
console.log(validateSqlSelect("SELECT * FROM users"));  // true
console.log(validateSqlSelect("INSERT INTO users VALUES (1)"));  // false

// === Extract SQL WHERE conditions ===
function extractWhereConditions(sql: string): string[] {
    const m = sql.match(/WHERE\s+(.+?)(?:\s+ORDER|\s+GROUP|\s+LIMIT|$)/is);
    if (!m) return [];
    const whereClause = m[1];
    const conditions = whereClause.split(/\s+AND\s+/i);
    return conditions.map(c => c.trim());
}

const sqlWithWhere = "SELECT * FROM users WHERE age > 18 AND status = 'active' ORDER BY name";
console.log(extractWhereConditions(sqlWithWhere));
// ['age > 18', "status = 'active'"]
```

### 13.10 File Path & OS-Level Pattern Matching

```python
import re

# === Validate Windows file path ===
def is_valid_windows_path(path: str) -> bool:
    pattern = r"^[a-zA-Z]:\\(?:[^\\/:*?\"<>|]+\\)*[^\\/:*?\"<>|]*$"
    return bool(re.fullmatch(pattern, path))

print(is_valid_windows_path("C:\\Users\\Alice\\Documents\\file.txt"))  # True
print(is_valid_windows_path("D:\\invalid\\file*.txt"))  # False (* not allowed)

# === Validate Unix/POSIX file path ===
def is_valid_posix_path(path: str) -> bool:
    pattern = r"^(/[^/\0]*)+/[^/\0]*$"
    return bool(re.fullmatch(pattern, path))

print(is_valid_posix_path("/home/alice/docs/file.txt"))  # True
print(is_valid_posix_path("/invalid/path/with/null\x00char"))  # False

# === Extract all file extensions from a directory listing ===
dir_listing = """
file1.py
file2.ts
config.json
style.css
readme.md
package.json
Dockerfile
.gitignore
"""

ext_pattern = re.compile(r"^\S+\.(?P<ext>[a-z0-9]+)$", re.I)
extensions = set()
for line in dir_listing.strip().split("\n"):
    m = ext_pattern.match(line.strip())
    if m:
        extensions.add(m.group("ext"))
print(extensions)  # {'py', 'ts', 'json', 'css', 'md'}

# === Extract filenames (without directory) from full paths ===
def get_filename(path: str) -> str:
    """Extract filename from path (cross-platform)."""
    parts = re.split(r'[\\/]', path)
    return parts[-1]

print(get_filename("/home/alice/docs/file.txt"))   # 'file.txt'
print(get_filename("C:\\Users\\Alice\\file.txt"))   # 'file.txt'
```

```typescript
// === File Path & OS-Level Pattern Matching ===

// === Validate Unix/POSIX file path ===
function isValidPosixPath(path: string): boolean {
    const pattern = /^\/[^\0]*$/;
    return pattern.test(path);
}
console.log(isValidPosixPath("/home/alice/docs/file.txt"));  // true

// === Extract all file extensions from a directory listing ===
const dirListing = `file1.py\nfile2.ts\nconfig.json\nstyle.css\nreadme.md\npackage.json`;

const extPattern = /^\S+\.(?<ext>[a-z0-9]+)$/i;
const extensions = new Set<string>();
dirListing.split('\n').forEach(line => {
    const m = line.trim().match(extPattern);
    if (m?.groups) extensions.add(m.groups.ext);
});
console.log(extensions);  // {'py', 'ts', 'json', 'css', 'md'}

// === Extract filenames from full paths ===
function getFilename(path: string): string {
    const parts = path.split(/[\\/]/);
    return parts[parts.length - 1];
}
console.log(getFilename("/home/alice/docs/file.txt"));   // 'file.txt'
console.log(getFilename("C:\\Users\\Alice\\file.txt"));   // 'file.txt'
```

### 13.11 Color Format Conversion Patterns

```python
import re

def hex_to_rgb(hex_color: str) -> tuple[int, int, int] | None:
    """Convert #RRGGBB or #RGB to RGB tuple."""
    pattern = r"^#(?:(?P<full>[0-9a-fA-F]{6})|(?P<shorthand>[0-9a-fA-F]{3}))$"
    m = re.fullmatch(pattern, hex_color, re.I)
    if not m:
        return None
    
    full = m.group("full") or m.group("shorthand")
    # Expand shorthand: #fff -> #ffffff
    if m.group("shorthand"):
        f = "".join(c * 2 for c in full)
    else:
        f = full
    
    return (int(f[0:2], 16), int(f[2:4], 16), int(f[4:6], 16))

print(hex_to_rgb("#ff5733"))  # (255, 87, 51)
print(hex_to_rgb("#f53"))    # (255, 85, 51) — shorthand!
print(hex_to_rgb("invalid"))   # None

# === Match all CSS color formats in a stylesheet ===
css = """
color: #ff5733;
background-color: rgb(255, 87, 51);
border-color: rgba(0, 0, 0, 0.5);
text-shadow: hsl(240, 100%, 50%);
color: blue; /* named color */
"""

# Extract all hex colors
hex_colors = re.findall(r"#[0-9a-fA-F]{3,8}\b", css)
print(hex_colors)  # ['#ff5733', '#f533']  (if shorthand was there)

# Extract rgb/rgba/hsla values
rgb_colors = re.findall(
    r"(rgba?|hsla?)\(([^)]+)\)",
    css,
    re.I
)
for color_type, values in rgb_colors:
    print(f"{color_type}: {values}")
# rgba: 0, 0, 0, 0.5
# hsla: 240, 100%, 50%
```

```typescript
// === Color Format Conversion Patterns ===

function hexToRgb(hexColor: string): [number, number, number] | null {
    const match = hexColor.match(/^#(?:(?<full>[0-9a-fA-F]{6})|(?<shorthand>[0-9a-fA-F]{3}))$/i);
    if (!match || !match.groups) return null;
    
    let full = match.groups.full ?? match.groups.shorthand!;
    if (match.groups.shorthand) {
        full = [...full].map(c => c.repeat(2)).join('');
    }
    
    return [
        parseInt(full.slice(0, 2), 16),
        parseInt(full.slice(2, 4), 16),
        parseInt(full.slice(4, 6), 16)
    ];
}

console.log(hexToRgb("#ff5733"));  // [255, 87, 51]
console.log(hexToRgb("#f53"));     // [255, 85, 51] — shorthand!
console.log(hexToRgb("invalid"));   // null

// === Match all CSS color formats in a stylesheet ===
const css = `color: #ff5733;\nbackground-color: rgb(255, 87, 51);\nborder-color: rgba(0, 0, 0, 0.5);\ntext-shadow: hsl(240, 100%, 50%);`;

// Extract all hex colors
const hexColors = css.match(/#[0-9a-fA-F]{3,8}\b/g) ?? [];
console.log(hexColors);  // ['#ff5733']

// Extract rgb/rgba/hsla values
const rgbMatches = [...css.matchAll(/(rgba?|hsla?)\(([^)]+)\)/gi)];
rgbMatches.forEach(([full, type, values]) => {
    console.log(`${type}: ${values}`);
});
// rgba: 0, 0, 0, 0.5
// hsl: 240, 100%, 50%
```

### 13.12 Data Transformation & Formatting Patterns

```python
import re

def format_phone_number(phone: str, fmt: str = "international") -> str:
    """Format a phone number into different formats."""
    digits = re.sub(r"[^\d]", "", phone)
    
    if not (10 <= len(digits) <= 15):
        return phone  # Invalid: return as-is
    
    if fmt == "international":
        return f"+{digits}"
    elif fmt == "us":
        if len(digits) == 11 and digits[0] == '1':
            digits = digits[1:]  # strip country code
        return f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"
    elif fmt == "e164":
        return f"+{digits}"
    elif fmt == "grouped":
        groups = []
        for i in range(0, len(digits), 4):
            groups.append(digits[i:i+4])
        return ' '.join(groups)
    else:
        return phone

print(format_phone_number("1-555-123-4567", "us"))          # '(555) 123-4567'
print(format_phone_number("+44 20 7946 0958", "grouped"))   # '4420 7946 0958'
print(format_phone_number("(555) 123-4567", "international"))  # '+5551234567'

# === Convert camelCase to snake_case ===
def camel_to_snake(name: str) -> str:
    """Convert CamelCase or camelCase to snake_case."""
    # Insert underscore before uppercase letters, then lowercase
    s1 = re.sub(r'([A-Z]+)([A-Z][a-z])', r'\1_\2', name)
    return re.sub(r'([a-z\d])([A-Z])', r'\1_\2', s1).lower()

print(camel_to_snake("camelCase"))           # 'camel_case'
print(camel_to_snake("HTMLParser")           # 'html_parser'
print(camel_to_snake("simpleTest")          # 'simple_test'
print(camel_to_snake("getHTTPResponse")     # 'get_http_response'

# === Convert snake_case to camelCase ===
def snake_to_camel(name: str, capitalize_first: bool = False) -> str:
    """Convert snake_case to camelCase."""
    parts = name.split('_')
    if not capitalize_first:
        return parts[0] + ''.join(p.capitalize() for p in parts[1:])
    else:
        return ''.join(p.capitalize() for p in parts)

print(snake_to_camel("camel_case"))           # 'camelCase'
print(snake_to_camel("simple_test", True))   # 'SimpleTest'  (PascalCase!)
```

```typescript
// === Data Transformation & Formatting Patterns ===

function formatPhoneNumber(phone: string, fmt: "international" | "us" | "e164" | "grouped" = "international"): string {
    const digits = phone.replace(/\D/g, '');
    if (!(digits.length >= 10 && digits.length <= 15)) return phone;
    
    switch (fmt) {
        case "us": {
            let d = digits;
            if (d.length === 11 && d[0] === '1') d = d.slice(1);
            return `(${d.slice(0,3)}) ${d.slice(3,6)}-${d.slice(6)}`;
        }
        case "grouped": {
            const groups: string[] = [];
            for (let i = 0; i < d.length; i += 4) groups.push(d.slice(i, i + 4));
            return groups.join(' ');
        }
        default:
            return `+${digits}`;
    }
}

console.log(formatPhoneNumber("1-555-123-4567", "us"));          // '(555) 123-4567'
console.log(formatPhoneNumber("+44 20 7946 0958", "grouped"));   // '4420 7946 0958'

// === Convert camelCase to snake_case ===
function camelToSnake(name: string): string {
    const s1 = name.replace(/([A-Z]+)([A-Z][a-z])/g, '$1_$2');
    return name.replace(/([a-z\d])([A-Z])/g, '$1_$2').toLowerCase();
}

console.log(camelToSnake("camelCase"));           // 'camel_case'
console.log(camelToSnake("HTMLParser"));          // 'html_parser'
console.log(camelToSnake("simpleTest"));          // 'simple_test'
console.log(camelToSnake("getHTTPResponse"));     // 'get_http_response'

// === Convert snake_case to PascalCase ===
function snakeToCamel(name: string): string {
    return name.split('_')
        .map((p, i) => i === 0 ? p.toLowerCase() : p.charAt(0).toUpperCase() + p.slice(1))
        .join('');
}
console.log(snakeToCamel("camel_case"));  // 'camelCase'

// PascalCase variant:
function snakeToPascal(name: string): string {
    return name.split('_')
        .map(p => p.charAt(0).toUpperCase() + p.slice(1))
        .join('');
}
console.log(snakeToPascal("simple_test"));  // 'SimpleTest'
```

---

## 14. 🆕 Visual Pattern Breakdowns

This section provides deep visual breakdowns of complex regex patterns — showing exactly how each part matches, step by step.

### 14.1 Email Validation Pattern — Anatomy

```python
pattern = r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z]{2,})+$"
```

Visual breakdown:

```
^                                    Start of string
[a-zA-Z0-9_.+-]+                    Local part: one or more valid chars
                                     (letters, digits, . _ + -)
@                                   Literal @ symbol
[a-zA-Z0-9-]+                       Domain name: letters, digits, hyphens
(?:                                  Non-capturing group:
  \.[a-zA-Z]{2,}                   Dot followed by 2+ letters
)+                                   One or more such segments (.com, .co.uk)
$                                    End of string
```

Matching example: `user.name@example.com`:
| Position | Pattern Part | Matched? | Consumes |
|----------|-------------|----------|----------|
| 0        | `^`         | Yes      | (nothing) |
| 0-8      | `[a-zA-Z0-9_.+-]+` | Yes | "user.name" |
| 8        | `@`     | Yes      | "@" |
| 9-16     | `[a-zA-Z0-9-]+`    | Yes | "example" |
| 16-20    | `(?:\.[a-zA-Z]{2,})+` | Yes | ".com" |
| 20       | `$`              | Yes      | (nothing) |

### 14.2 IPv4 Validation Pattern — Step-by-Step

```python
pattern = r"^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$"
```

Visual breakdown of one octet `(?:25[0-5]|2[0-4]\d|[01]?\d\d?)`:

```
25[0-5]           Matches: 250-255
|                 OR
2[0-4]\d          Matches: 200-249
|                 OR
[01]?\d\d?        Matches: 0-199 (with optional leading zero)
```

Matching `192.168.1.1`:
| Octet | Pattern Matched | Result |
|-------|----------------|--------|
| 192   | `[01]?\d\d?` | Matches '192' (0 + 92) |
| 168   | `[01]?\d\d?` | Matches '168' (1 + 68) |
| 1     | `[01]?\d\d?` | Matches '1' (empty + 1) |
| 1     | `[01]?\d\d?` | Matches '1' (empty + 1) |

Matching `256.0.0.1`:
| Octet | Pattern Matched | Result |
|-------|----------------|--------|
| 256   | **FAILS all three alternatives** | No match! Correct! \u274c |

### 14.3 URL Pattern — Visual Walkthrough

```python
pattern = r"^(?P<protocol>https?):\\/(?{3}[^/]+)(?::(?P<port>\d+))?(?P<path>/[\w./%-]*)?(?:\?(?P<query>[\w=&%-]*))?$(?P<hash>[#\w-]*)?$"
```

For URL `https://user:pass@example.com:8080/path?q=1#section`:

```
https?                        Protocol: 'https'
:\\/                         Literal ://
[^@/]+                        User info: 'user:pass'  (simplified)
@                           Literal @
example.com                   Host
:8080                         Port
/path                         Path
?q=1                          Query
#section                      Hash
```

### 14.4 SemVer Pattern — Visual Walkthrough

```python
pattern = r"^(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)(?:-(?P<pre>[0-9A-Za-z-]+))?(?:\+(?P<build>[0-9A-Za-z-]+))?"
```

For version `2.0.0-alpha.1+build.42`:
| Group | Pattern | Matches |
|-------|---------|----------|
| major   | `0|[1-9]\d*` | "2" (matches `[1-9]\d*`) |
| minor   | `0|[1-9]\d*` | "0" (matches literal "0") |
| patch   | `0|[1-9]\d*` | "0" (matches literal "0") |
| pre     | `[0-9A-Za-z-]+` | "alpha.1" |
| build   | `[0-9A-Za-z-]+` | "build.42" |

For version `0.0.0`: All groups match the literal "0" alternative.
For version `10.0.0-beta+abc`: major matches `\d*`, pre = "beta", build = "abc".

### 14.5 Password Strength — Lookahead Chain Visualization

```python
pattern = r"^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&#]).{8,}$"
```

Visual flow for password `Abc123!@#`:

```
^                                    Start of string (position 0)
| └ (?=.*[A-Z])   Lookahead: scans entire string for [A-Z] -> finds 'A' ✓
├ (?=.*[a-z])   Lookahead: scans entire string for [a-z] -> finds 'bc' ✓
├ (?=.*\d)      Lookahead: scans entire string for \d -> finds '123' ✓
└ (?=.*[@$!%*?&#])  Lookahead: scans for special char -> finds '!@#' ✓
.{8,}                    Now match: 8+ chars -> 'Abc123!@#' (9 chars) ✓
$                        End of string ✓
```

### 14.6 Balanced HTML Tags — Why Regex Fails Here

**Attempted pattern (pseudo-code):**
```python
r"<([^>]+)>(.*?)</\1>"
```

For `<div><span>text</span></div>`:
| Step | Action | Result |
|------|--------|--------|
| 1 | `<` matches first `<` | Found open tag |
| 2 | `([^>]+)` captures "div" | Tag name found |
| 3 | `>` matches `>` | End of open tag |
| 4 | `(.*?)` matches minimally | Matches "<span>text</span></div>... wait! |
| 5 | `<\/\1>` checks for `</div>` | Finds it at end ✓ |

But what about nested tags? The pattern matches the **outermost** pair: `<div>...</div>` including the inner `<span>`.

For truly balanced HTML, you need a parser (like BeautifulSoup in Python or DOMParser in TypeScript), not regex. Regex cannot handle arbitrary nesting depth.

### 14.7 Possessive Quantifier — Visual Backtracking Prevention

**Without possessive (exponential backtracking):**
```
Pattern: ^(a+)+b$
Input:   aaaaaaaaaaaaaaaaaaaaaaz  (25 'a's, no 'b')

Step 1: (a+) matches all 25 'a's
Step 2: '+' group tries to match more but nothing left
Step 3: Looks for 'b' -- FAILS at position 25
Step 4: BACKTRACK! Give back one 'a' from inner (a+)
Step 5: Now (a+) has 24 'a's, outer + repeats...
Step 6: Repeat steps 2-5 with fewer 'a's each time!
... This creates 2^25 possible combinations! EXPONENTIAL TIME!
```

**With possessive quantifier (no backtracking):**
```
Pattern: ^(a++)*b$
Input:   aaaaaaaaaaaaaaaaaaaaaaz  (25 'a's, no 'b')

Step 1: (a++) matches all 25 'a' -- NEVER gives back!
Step 2: Looks for 'b' -- FAILS
Step 3: Outer * repeats... but inner ++ won't give back
Step 4: No more combinations to try ✓ INSTANT FAILURE!
```

---

## 15. 🆕 Edge Cases & Gotchas

This section catalogs every subtle edge case, gotcha, and cross-language difference that catches developers off guard.

### 15.1 String vs Pattern Escaping: The Double Escape Problem

```python
# === The most common regex bug source across both languages ===

# Python - raw string (CORRECT, readable):
pattern = r"\d{3}-\d{2}-\d{4}"    # Matches: 123-45-6789

# Python - non-raw string (WORKS but confusing):
pattern = "\\d{3}-\\d{2}-\\d{4}"   # Same result, double escaped!

# TypeScript - regex literal (CORRECT, clean):
pattern = /\d{3}-\d{2}-\d{4}/      # Clean, no escaping needed!

// TypeScript - string + new RegExp (THE PROBLEM):
pattern = new RegExp("\\d{3}-\\d{2}-\\d{4}")  // Must double-escape EVERY \!

// ⚠️ This is WRONG and won't match:
const bad = new RegExp("\d{3}");    // "d" followed by literal '{3}' -- NOT what you want!
```

### 15.2 findall() Behavior Trap in Python

```python
import re

text = "abc123def456"

# WITHOUT groups: returns list of full matches
print(re.findall(r"\d+", text))    # ['123', '456']

# WITH groups: returns list of TUPLES (each tuple = captured groups!)
print(re.findall(r"(\d+)", text))  # [('123',), ('456',)] -- tuples!

# With two groups: each item is a 2-tuple
print(re.findall(r"(\w+)(\d+)", "abc123 def456"))
# [('abc', '123'), ('def', '456')]

# FIX: use finditer() if you want full match objects:
for m in re.finditer(r"(\d+)", text):
    print(m.group())   # Use .group() to get the actual match string!
    print(m.groups())  # Tuple of captured groups
```

```typescript
// TypeScript has NO such trap! matchAll always returns consistent arrays:
const text = "abc123def456";

const m1 = [...text.matchAll(/\d+/g)];
console.log(m1);  // ['123', '456'] -- just strings!

const m2 = [...text.matchAll(/(\w+)(\d+)/g)];
console.log(m2[0]);  // ['abc123', 'abc', '123'] -- [full, group1, group2]
// Consistent array format -- no tuple vs string confusion!
```

### 15.3 Lookahead/Backreference Interaction

```python
import re

# === Lookahead doesn't consume characters! ===
text = "foo bar baz"
m = re.search(r"foo\s+(?=bar)", text)
print(m.group())     # 'foo ' -- lookahead not consumed!
print(text[m.end():])  # 'baz' -- everything after the match

# === But backreference in lookbehind DOES work! ===
text = "price: $100 and tax: $50"
dollars = re.findall(r"(?<=\$)\d+", text)
print(dollars)  # ['100', '50'] -- \$ asserts preceding, doesn't capture

# === Lookahead can be nested! ===
# Match word followed by number followed by another number
# (the first number must be even)
text = "item1 2 item2 3 item3 4"
pattern = re.compile(r"(\w+)(?=(\d+) (?:\d+))")
for m in pattern.finditer(text):
    print(m.group(), "->", m.groups())
# 'item2' -> ('2',)

# === Negative lookahead with boundaries ===
# Match words NOT followed by 'ing'
text = "running jumping walk run walked"
words = re.findall(r"\b\w+(?!ing)\b", text)
print(words)  # ['walk', 'run', 'walked'] -- not 'running' or 'jumping'
```

### 15.4 Word Boundary \b Gotchas

```python
import re

# === \b depends on \w definition! ===
# In Python (Unicode-aware in Py3.12+):
print(re.findall(r"\b\w+\b", "café résumé naïve"))
# ['café', 'résumé', 'naïve'] -- accented chars ARE word chars!

# === But underscore confuses \b! ===
print(re.findall(r"\bfoo\b", "_foo_bar"))   # [] -- _ is a word char!
print(re.findall(r"\bfoo\b", " foo bar"))    # ['foo'] -- space separates ✓

# \B (NOT word boundary):
print(re.findall(r"\Bfoo\B", "_foo_bar baz_foo_qux"))  # ['_foo_', '_foo_']
# Matches 'foo' NOT at word boundary on either side

# === Practical: extract whole words from camelCase ===
camel = "getHTTPResponse"
words = re.findall(r"[A-Z]?[a-z]+|[A-Z]+(?=[A-Z][a-z]|$)", camel)
print(words)  # ['get', 'HTTP', 'Response']
```

### 15.5 Multiline Flag Edge Cases

```python
import re

# === ^ and $ in multiline mode vs raw string mode ===
text = "line1\nline2\nline3"

# Default: ^ matches ONLY the start of the entire string
print(re.findall(r"^line", text))           # ['line'] -- only first line!

# With MULTILINE flag: ^ matches start of EACH line
print(re.findall(r"^line", text, re.MULTILINE))  # ['line', 'line', 'line']

# $ also changes in multiline mode:
print(re.findall(r"line$", text, re.MULTILINE))    # ['line', 'line', 'line']
print(re.findall(r"line$", text))                     # ['line'] -- only last line!

# === \A and \Z are ALWAYS absolute (unaffected by m flag) ===
print(re.search(r"\Alin", text).group())    # 'lin' -- always at string start!
print(re.search(r"e3\Z", text).group())     # 'e3' -- always at string end!

# === Common bug: $ in multiline doesn't match before \r ===
text_crlf = "line1\r\nline2\r\n"
print(re.findall(r"line$", text_crlf, re.MULTILINE))  # ['line'] - matches but position is tricky
# Fix for CRLF: use r"line\r?$" or normalize line endings first
```

### 15.6 Empty String Matches

```python
import re

# === .* matches empty string! ===
text = "hello"
m = re.findall(r".*", text)
print(m)  # ['hello', ''] -- the '' is the match at end-of-string!

# === + and ? also match empty in some edge cases ===
print(re.findall(r"a?b", "abacb"))  # ['ab', 'b'] -- '?' matched 0 'a's before second 'b'!

# === Avoid: use specific patterns instead of .* when possible ===
# BAD: matches everything including empty strings
re.findall(r".*", text)
# GOOD: match only what you want
re.findall(r"[a-z]+", text)

# TypeScript equivalent behavior:
// "hello".match(/.*/g);    // ['hello', '']
// "hello".match(/[a-z]+/g);  // ['hello'] -- no empty!
```

### 15.7 Group Numbering with Non-Capturing Groups

```python
import re

# Non-capturing groups (?:...) don't increment group numbers!
pattern = re.compile(r"(\d+)(?:-)(\w+)")
m = pattern.search("123-abc")
print(m.groups())  # ('123', 'abc') -- only 2 groups! The non-capturing one is invisible.

# Named and unnamed: numbering order follows opening paren!
pattern = re.compile(r"(?P<first>\d+)(?:(?P<middle>-)|_)(?P<last>\w+)")
m = pattern.search("123-abc")
print(m.groups())  # ('123', '-', 'abc') -- numbered in order of (, (, (
p.print(m.groupdict())  # {'first': '123', 'middle': '-', 'last': 'abc'}

# TypeScript: identical behavior with (?<name>) syntax!
```

### 15.8 Regex Module vs String Method Confusion

```python
# === CRITICAL: Python regex is MODULE-based, not method-based! ===
# WRONG in Python:
"hello world".search(r"\bworld\b")          # AttributeError!
"hello world".match(r"hello")               # This exists but only at START!

# CORRECT in Python:
import re
re.search(r"\bworld\b", "hello world")    # Match object or None
re.match(r"hello", "hello world")          # Match at start only!

# === TypeScript: regex methods ARE on strings! ===
// CORRECT in TypeScript:
"hello world".search(/\bworld\b/)           // 6 -- index of match
"hello world".match(/hello/)                // ['hello'] -- match array or null
"hello world".replace(/world/, "there")     // 'hello there'

// This is the #1 source of confusion for TS devs learning Python regex!
```

### 15.9 Replace Backreference Syntax Differences

```python
import re

# === Python replacement string syntax ===
text = "John Smith - Jane Doe"
print(re.sub(r"([A-Z][a-z]+) ([A-Z][a-z]+)", r"\2, \1", text))
# 'Smith, John - Doe, Jane'

# Named group in replacement:
print(re.sub(
    r"(?P<first>[A-Z][a-z]+) (?P<last>[A-Z][a-z]+)",
    r"\g<last>, \g<first>",
    text
))
# 'Smith, John - Doe, Jane'

# Using \g<number> for unambiguous reference:
text = "test123"
print(re.sub(r"(\d+)", r"num=\1_suffix", text))  # 'testnum=123_suffix'
```

```typescript
// === TypeScript replacement string syntax ===
const text = "John Smith - Jane Doe";
console.log(text.replace(/([A-Z][a-z]+) ([A-Z][a-z]+)/g, "$2, $1"));
// 'Smith, John - Doe, Jane'

// Special replacements:
text.replace(/(\w+)@(\w+)\.(\w+)/, "@$3:$2:$1");
// '@.com:example:john' -- $& = full match, $' = after match, $` = before match

console.log("john@example.com".replace(/(.+)$/g, "$&.uk"));  // 'john@example.com.uk'
// $& inserts the full matched text!
```

### 15.10 Zero-Width Matches & infinite Loops

```python
import re

# === \b is zero-width -- can cause infinite loops in replace! ===
text = "hello"
try:
    # This WILL loop forever because \b matches, replaces with 'X\b',
    # then \b matches again at new position... infinite loop!
    result = re.sub(r"\b", "X", text)  # Actually this works: XhXeIlLoX
except RecursionError:
    print("Infinite loop!")

# The real danger is in while loops with sticky-like behavior:
count = 0
while re.search(r"", text):  # Empty pattern matches at EVERY position!
    count += 1
    if count > 100: break     # Prevent infinite loop!
print(count)  # Would be len(text) + 1 for each position

# === FIX: always use non-empty patterns in loops! ===
count = 0
for m in re.finditer(r"", text):  # Even finditer with empty pattern...
    count += 1
print(count)  # len(text) + 1 = matches at each position + end
# Use specific patterns, not empty ones!
```

### 15.11 Unicode Handling Differences

```python
import re

# === Python's \w includes Unicode letters (Py3.12+) ===
print(re.findall(r"\w+", "café résumé naïve 日本語"))
# ['café', 'résumé', 'naïve', '日本語'] -- all word chars!

# === But \d is still ASCII digits ONLY in stdlib re! ===
# Arabic-Indic digits ٠١٢٣ (0-3) are NOT matched by \d in Python's re!
print(re.findall(r"\d+", "٠١٢٣"))  # [] -- empty! No ASCII digits found!

# === TypeScript: \w is ASCII only, even for Unicode text ===
// console.log("café résumé".match(/\w+/g));  // ['caf'] -- stops at accented char!
// This is a KEY difference between the two languages!

# === Fix for Python stdlib re: use third-party 'regex' module ===
# pip install regex
import regex
print(regex.findall(r"\p{L}+", "café résumé naïve 日本語"))
# ['café', 'résumé', 'naïve', '日本語'] -- all Unicode letters!

# TypeScript's \p{} works natively with the 'u' flag:
// /[\p{Script=Latin}]+/u.test("café");  // true
```

### 15.12 Named Group Capture in Different Contexts

```python
import re

# === Same named group multiple times: last capture wins ===
pattern = re.compile(r"(?(?P<proto>https?)://|(?P<proto>ftp)://)(?P<host>[\w.]+)")
text = "https://example.com"
m = pattern.search(text)
print(m.group("proto"))  # 'https'

# === Different groups with same name in alternation: only one is active! ===
pattern = re.compile(r"(?P<date>\d{4}-\d{2}-\d{2})|(?P<date>\d{2}/\d{2}/\d{4})")
text1 = "2025-01-15"
text2 = "01/15/2025"
m1 = pattern.search(text1)
m2 = pattern.search(text2)
print(m1.group("date"))  # '2025-01-15'
print(m2.group("date"))  # '01/15/2025' -- different group index, same name
# This is the "branch reset" behavior -- third-party 'regex' module handles this better!
```

---

## 16. 🆕 Advanced TypeScript Regex Patterns

This section covers advanced patterns and techniques specific to TypeScript/JavaScript that have no Python equivalent, plus modern ES2018+ features.

### 16.1 Unicode Property Escapes (ES2018+)

```typescript
// Requires 'u' flag for \p{} syntax

// Match only Latin script characters
/[\p{Script=Latin}]+/u.test("café");        // true
/[\p{Script=Latin}]+/u.test("日本語");       // false

// Match all emoji
/[\p{Extended_Pictographic}]+/u.test("🎉🎊"); // true

// Match whitespace in ALL Unicode scripts
/[\p{White_Space}]/gu.test("hello\u2003world");  // true (em space is Unicode whitespace)

// Negative property: NOT a letter
/[\P{L}]+/u.test("123!@#");  // true -- all non-letter characters

// Combined with character classes:
const emojiRegex = /[\p{Extended_Pictographic}\p{Emoji_Modifier}]/gu;
const text = "Hello 🎉 World 😊";
console.log([...text.matchAll(emojiRegex)]);  // ['🎉', '😊']

// Emoji range in character class:
const emojiRange = /[[\u{1F300}-\u{1F9FF}]|[\u{2600}-\u{26FF}]]/su;
console.log(emojiRange.test("🎉"));  // true
```

### 16.2 Lookbehind with Alternation (ES2018+)

```typescript
// ES2018 introduced fixed-width lookbehind -- a major feature!

// Fixed-width positive lookbehind
const text = "Price: $99, Tax: $15";
text.match(/(?<=\$)\d+/g);  // ['99', '15']

// Fixed-width negative lookbehind
text.match(/(?<!\$)\d+/g);  // Only numbers NOT preceded by $

// Variable-width lookbehind is NOT supported -- this fails:
// /(?<={https://|http://})example.com/u;  // SyntaxError in some engines!

// Workaround: capture the prefix instead
const urlPattern = /(https?:\/\/)?example\.com/;
const match = "https://example.com".match(urlPattern);
console.log(match?.[1]);  // 'https://' or undefined

// Alternative: use capturing groups to extract what lookbehind would have
const prefixPattern = /(?:^|(?<=[,;\s]))([a-zA-Z0-9.-]+)\.example\.com/;
```

### 16.3 Sticky Flag Advanced Patterns

```typescript
// The sticky flag (y) only matches at regex.lastIndex -- unique to JS/TS!

// Tokenizer with sticky: ensures we consume EVERY character in order
function tokenize(input: string): { type: string; value: string }[] {
    const patterns = [
        /\s+/y,                // whitespace (skip)
        /(?<=[=+\-*])y,       // operators (lookbehind for fixed-width)
        /[a-zA-Z_$][\w$]*/gy, // identifiers
        /\d+(?:\.\d+)?/gy     // numbers
    ];
    
    const tokens: { type: string; value: string }[] = [];
    let lastIndex = 0;
    
    while (lastIndex < input.length) {
        let matched = false;
        for (const pattern of patterns) {
            pattern.lastIndex = lastIndex;
            const match = pattern.exec(input);
            if (match && match.index === lastIndex) {
                if (pattern.source.includes('\s')) {
                    lastIndex = match[0].length;  // skip whitespace
                } else {
                    tokens.push({ type: 'token', value: match[0] });
                    lastIndex += match[0].length;
                }
                matched = true;
                break;
            }
        }
        if (!matched) {
            throw new Error(`Unexpected character at position ${lastIndex}: ${input[lastIndex]}`);
        }
    }
    return tokens;
}

// Tokenize "function add(a, b) { return a + b; }"
console.log(tokenize("function add(a, b) { return a + b; }"));
```

### 16.4 String.prototype.matchAll() Modern Pattern

```typescript
// matchAll is the modern replacement for looping with exec()
const text = "User: alice@example.com, Admin: bob@test.org";
const emailRegex = /[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}/g;

// Before ES2020 (verbose): let m; while ((m = regex.exec(text)) !== null)
// After ES2020 (clean):
for (const [fullMatch, localPart, domainPart] of text.matchAll(/([\w.-]+)@([\w.-]+)\.[a-zA-Z]{2,}/g)) {
    console.log(`${localPart}@${domainPart}`);  // 'alice@example', 'bob@test'
}

// Using destructuring for named access:
const entries = [...text.matchAll(/(?<user>[\w.-]+)@(?<domain>[\w.-]+)\.[a-zA-Z]{2,}/g)];
for (const { groups } of entries) {
    if (groups) {
        console.log(`${groups.user}@${groups.domain}`);  // 'alice@example', 'bob@test'
    }
}

// TypeScript type safety:
interface MatchResult {
    [0]: string;           // Full match
    [1]: string;           // First group
    index: number;
    input: string;
    groups: { user: string; domain: string } | undefined;
}
const typedMatches = [...text.matchAll(/(?<user>[\w.-]+)@(?<domain>[\w.-]+)\.[a-zA-Z]{2,}/g)] as MatchResult[];
```

### 16.5 RegExp.prototype.flags Property (ES2015+)

```typescript
// Check which flags are set on a regex:
const regex = /hello/gimuy;  // All possible flags!
console.log(regex.flags);      // 'gimuy' -- shows all active flags
console.log(regex.global);     // true
console.log(regex.ignoreCase); // true
console.log(regex.multiline);  // true
console.log(regex.unicode);    // true
console.log(regex.sticky);     // true

// Dynamically build regex with specific flags:
function buildRegex(pattern: string, flags: 'gi' | 'i' | 'm' | 'g' = '') {
    return new RegExp(pattern, flags);
}
const caseInsensitiveUrl = buildRegex('https?://[\\w.-]+', 'gi');
// Equivalent to: /https?:\/\/[^\s]+/gi
```

### 16.6 Regex with Template Literals and Dynamic Patterns

```typescript
// Building regex from user input or variables -- common pattern!
function buildEmailRegex(domain: string): RegExp {
    // Escape special regex characters in domain first!
    const escapedDomain = domain.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    return new RegExp(`^[\\w.-]+@${escapedDomain}$`);
}

const localEmailRe = buildEmailRegex('example.com');
console.log(localEmailRe.test('user@example.com'));  // true
console.log(localEmailRe.test('user@test.org'));     // false

// Dynamic pattern from user input (search functionality):
function createSearchPattern(term: string, caseInsensitive: boolean): RegExp {
    const escaped = term.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const flags = `g${caseInsensitive ? 'i' : ''}`;
    return new RegExp(escaped, flags);
}

const pattern = createSearchPattern('hello world', true);
console.log(pattern);  // /hello\s+world/gi
```

### 16.7 Regex to Validate Complex Data Formats

```typescript
// === IP Address (IPv4) Validation ===
const IPV4_PATTERN = /^((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$/;
console.log(IPV4_PATTERN.test('192.168.1.1'));   // true
console.log(IPV4_PATTERN.test('256.0.0.1'));     // false (octet > 255)
console.log(IPV4_PATTERN.test('1.2.3'));          // false (only 3 octets)

// === Credit Card Validation (Luhn check not in regex -- do it in code!) ===
// Regex only validates format: 16 digits with optional separators
const CREDIT_CARD_PATTERN = /^\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}$/;
console.log(CREDIT_CARD_PATTERN.test('4111-1111-1111-1234'));  // true
console.log(CREDIT_CARD_PATTERN.test('4111111111111234'));     // true (no separators)

// === SSN (Social Security Number) ===
const SSN_PATTERN = /^\d{3}-\d{2}-\d{4}$/;
console.log(SSN_PATTERN.test('123-45-6789'));  // true
console.log(SSN_PATTERN.test('123456789'));     // false (format requires dashes)

// === MAC Address ===
const MAC_PATTERN = /^([0-9a-fA-F]{2}[:-]){5}[0-9a-fA-F]{2}$/;
console.log(MAC_PATTERN.test('AA:BB:CC:DD:EE:FF'));  // true
console.log(MAC_PATTERN.test('aa-bb-cc-dd-ee-ff'));   // true (hyphen variant)

// === GitHub Username/Repo ===
const GITHUB_PATTERN = /^([a-zA-Z0-9](?:[a-zA-Z0-9]|-(?=[a-zA-Z0-9])){0,38}\/) ([a-zA-Z0-9_.-]+)$/;
console.log(GITHUB_PATTERN.test('facebook/react'));      // true
console.log(GITHUB_PATTERN.test('facebook/React.js'));   // false (. not allowed in org)

// === Phone Number (international E.164) ===
const PHONE_E164 = /^\+[1-9]\d{1,14}$/;
console.log(PHONE_E164.test('+15551234567'));  // true
console.log(PHONE_E164.test('15551234567'));    // false (must start with +)
```

### 16.8 Regex-Based State Machine Patterns

```typescript
// Use regex to validate a sequence of states -- a common parser technique!

// === Validate HTTP Method Sequence ===
const VALID_SEQUENCE = /^(GET|POST|PUT|DELETE|PATCH|HEAD|OPTIONS)(\s+(GET|POST|PUT|DELETE|PATCH|HEAD|OPTIONS))*$/;
console.log(VALID_SEQUENCE.test('GET POST DELETE'));  // true
console.log(VALID_SEQUENCE.test('GET TRACE'));        // false (TRACE not in list)

// === Validate Git Commit Message Format (Conventional Commits) ===
const COMMIT_FORMAT = /^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+$/;
console.log(COMMIT_FORMAT.test('feat(auth): add login'));         // true
console.log(COMMIT_FORMAT.test('feat: update docs'));              // true
console.log(COMMIT_FORMAT.test('random message'));                 // false

// === Validate JSON Key-Value structure (simplified) ===
const SIMPLE_JSON = /^\{(?:\s*"[^"]+"\s*:\s*(?:"[^"]*"|\d+|true|false|null)(?:,\s*)?)*\s*}$/;
console.log(SIMPLE_JSON.test('{"name": "Alice", "age": 30}'));     // true
console.log(SIMPLE_JSON.test('{invalid}'));                              // false
```

---

## 17. 🆕 Advanced Python `regex` Module Features

This section covers the third-party [`regex`](https://github.com/mrabarnett/mrb-regex) module which adds powerful features beyond stdlib `re`.

### 17.1 Installation & Import

```bash
pip install regex
```

```python
import regex  # NOT 're' -- this is the third-party module!
# Still use re.compile() for stdlib patterns that don't need advanced features
```

### 17.2 Atomic Grouping `(?>...)`

```python
import regex

# Atomic groups prevent backtracking once matched -- huge performance win!
pattern = regex.compile(r"(?>a+)b")
text = "aaaaaaaaaaaaaaaaaaaaaz"
print(pattern.search(text))  # None -- 'a++' consumed all, no 'b' found, NO BACKTRACK

# Compare with stdlib re (exponential backtracking):
import re
# re.search(r"(a+)b", text)  # Would hang on very long 'a' strings!

# Atomic groups are like possessive quantifiers but for group content!
pattern2 = regex.compile(r"(?>hello|hell|hel|he)world")
print(pattern2.search("helloworld"))    # 'helloworld' -- matches longest alternation, no backtrack

# Practical: fast XML tag matching without catastrophic backtracking
xml_pattern = regex.compile(r"<(?>(?:[^<>]+)<)*[^>]*>")
```

### 17.3 Named Group Variants & Overlapping Groups

```python
import regex

# The 'regex' module supports overlapping named groups!
pattern = regex.compile(
    r"(?P<date>\d{4}-\d{2}-\d{2})"
    r"|"
    r"(?P<datetime>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})"
)
text = "2025-01-15 14:30:45"
m = pattern.search(text)
print(m.group("date"))       # '2025-01-15' -- first branch matched
print(m.group("datetime"))   # '2025-01-15 14:30:45' -- also accessible!

# Branch reset groups (?|...):
# Both branches use the SAME group numbers!
pattern = regex.compile(r"(?|(\d{4})-(\d{2})-(\d{2})|(\d{2}):(\d{2}):(\d{2}))")
text = "2025-01-15"
m = pattern.search(text)
print(m.group(1))  # '2025' -- both branches would use group(1)
print(m.groups())  # ('2025', '01', '15')

# This is useful when you have multiple patterns that all produce the same output format!
```

### 17.4 Conditional Patterns `(?(name)yes|no)`

```python
import regex

# Conditional: if group matches, apply pattern A; otherwise pattern B
text = "Price: $99 or 100 dollars"
price_pattern = regex.compile(r"(?:(?P<dollars>\$)|(?P<number>\d+))(?(dollars)\d{2}|\d+)")
m = price_pattern.search(text)
print(m.group())  # '$99' -- the 'dollars' branch: dollar sign followed by 2 digits

# More complex: validate a URL with optional protocol
text = "example.com"
url_pattern = regex.compile(
    r"(?:(?P<proto>https?)://)?"
    r"(?P<host>[\w.-]+)"
    r"(?(proto)/.*)?$"
)
m = url_pattern.fullmatch(text)
print(m.group("proto"))   # None -- no protocol given
print(m.group("host"))    # 'example.com'
```

### 17.5 Possessive Quantifiers & Backtracking Prevention

```python
import regex

# Possessive quantifiers: match and NEVER give back (zero backtracking)
pattern = regex.compile(r"^(a++)*b$")
text = "a" * 1000 + "z"  # No 'b' at the end!
print(pattern.search(text))  # None -- instant failure, no exponential time!

# Compare with stdlib re:
import re
# re.search(r"^(a+)+b$", text)  # Would take exponentially longer as 'a' count grows!

# Other possessive quantifiers:
text2 = "abc"
print(regex.search(r"a++bc", text2))     # None -- 'a++' grabs all 'a's, won't backtrack for 'b'
print(regex.search(r"a+?bc", text2))     # Match! 'a+' takes only 1 'a', can give back
```

### 17.6 Recursive Patterns `(?R)`

```python
import regex

# Recursion: match nested structures of arbitrary depth!
def find_balanced_parens(text: str):
    """Find all balanced parenthesized groups, even nested."""
    # (?R) = recursive pattern (match entire regex again)
    pattern = regex.compile(r"\((?:[^()]+|(?R))*\)")
    return pattern.findall(text)

print(find_balanced_parens("(a + (b * c)) - (d / e)"))
# ['(a + (b * c))', '(d / e)'] -- correctly handles nested parens!

# Match XML/HTML-like tags with nesting:
def find_nested_tags(text: str):
    pattern = regex.compile(r"<(?P<tag>\w+)>(?:(?P>text)|<(?P=tag)>.*?</(?P=tag)>)*</(?P>tag>)")
    return pattern.findall(text)

# The 'regex' module is the ONLY way to do recursion in Python!
```

### 17.7 Advanced Unicode Support

```python
import regex

# Unicode property escapes (not available in stdlib re!):
text = "Hello 🎉 World 日本語 café"
print(regex.findall(r"\p{Extended_Pictographic}+", text))
# ['🎉'] -- all emoji!

print(regex.findall(r"\p{Script=Han}+", text))
# ['日本語'] -- all CJK characters!

# Unicode category matching:
text2 = "café 123 résumé αβγ"
print(regex.findall(r"\p{L}+", text2))  # All Unicode letters: ['café', 'résumé']
print(regex.findall(r"\p{N}+", text2))  # All Unicode numbers: ['123']

# Contrast with stdlib re:
import re
print(re.findall(r"\w+", "café résumé"))  # ['café', 'résumé'] -- works for letters!
# But \d only matches ASCII digits in stdlib re, not Unicode digits!
```

---

## 18. 🆕 Real-World Production Recipes

### 18.1 Password Validator (Production-Grade)

```python
import regex
from typing import TypedDict

class PasswordResult(TypedDict):
    valid: bool
    score: int  # 0-100
    errors: list[str]
    suggestions: list[str]

def validate_password(password: str) -> PasswordResult:
    """
    Production-grade password validator with detailed feedback.
    Uses the 'regex' module for advanced features.
    """
    errors: list[str] = []
    score = 50  # Start at baseline
    
    if len(password) < 8:
        errors.append("Password must be at least 8 characters")
        score -= 20
    elif len(password) >= 12:
        score += 10
    elif len(password) >= 16:
        score += 20
    
    # Complexity checks using regex (with possessive quantifiers for speed)
    has_lower = bool(regex.search(r"\p{Ll}+", password))
    has_upper = bool(regex.search(r"\p{Lu}+", password))
    has_digit = bool(regex.search(r"\p{Nd}+", password))
    has_special = bool(regex.search(r"[\p{P}\p{S}]+", password))
    
    if not has_lower:
        errors.append("Must contain at least one lowercase letter")
        score -= 10
    else:
        score += 5
    
    if not has_upper:
        errors.append("Must contain at least one uppercase letter")
        score -= 10
    else:
        score += 10
    
    if not has_digit:
        errors.append("Must contain at least one digit")
        score -= 10
    else:
        score += 10
    
    if not has_special:
        errors.append("Must contain at least one special character")
        score -= 10
    else:
        score += 15
    
    # Check for common patterns (weak passwords)
    common_passwords = regex.compile(
        r"^(password|123456|qwerty|abc123|letmein|welcome)(?i)$"
    )
    if common_passwords.fullmatch(password):
        score = max(0, score - 40)
        errors.append("This is a commonly used password")
    
    # Check for sequential characters
    sequential = regex.compile(r"(abc|bcd|cde|def|efg|fgh|ghi|hij|i jk|jkl|klm|lmn|mno|nop|opq|pqr|qrs|rst|stu|tuv|uvw|vwx|wxy|xza)")
    if sequential.search(password, regex.WHOLEMATCH):
        score -= 10
        errors.append("Avoid sequential characters")
    
    # Check for repeated characters
    repeated = regex.compile(r"(.)\1{2,}")  # 3+ consecutive identical chars
    if repeated.search(password):
        score -= 5
        errors.append("Avoid repeating the same character")
    
    suggestions: list[str] = []
    if not has_upper:
        suggestions.append("Add uppercase letters")
    if not has_digit:
        suggestions.append("Add numbers")
    if not has_special:
        suggestions.append("Add special characters (@$!%*?&)"
    
    return PasswordResult(
        valid=not errors and score >= 50,
        score=min(100, max(0, score)),
        errors=errors,
        suggestions=suggestions
    )
```

```typescript
// === Production-grade TypeScript password validator ===
interface PasswordResult {
    valid: boolean;
    score: number;
    errors: string[];
    suggestions: string[];
}

function validatePassword(password: string): PasswordResult {
    const errors: string[] = [];
    let score = 50;
    
    if (password.length < 8) {
        errors.push("Password must be at least 8 characters");
        score -= 20;
    } else if (password.length >= 16) {
        score += 20;
    }
    
    const checks = [
        { regex: /[\p{Lower}]+/u, name: "lowercase", penalty: 10 },
        { regex: /[\p{Upper}]+/u, name: "uppercase", bonus: 10 },
        { regex: /[\p{Nd}]+/u, name: "digit", bonus: 10 },
        { regex: /[\p{P}\p{S}]+/u, name: "special", bonus: 15 }
    ];
    
    for (const check of checks) {
        if (!check.regex.test(password)) {
            errors.push(`Must contain at least one ${check.name}`);
            score -= check.penalty || 0;
        } else {
            score += check.bonus || 0;
        }
    }
    
    const suggestions: string[] = [];
    checks.forEach(check => {
        if (!check.regex.test(password)) {
            suggestions.push(`Add ${check.name}`);
        }
    });
    
    return { valid: errors.length === 0 && score >= 50, score, errors, suggestions };
}
```

### 18.2 Log Aggregator with Multiple Format Support

```python
import regex
from typing import Generator

class LogAggregator:
    """Parse log lines in multiple formats and extract structured data."""
    
    # Define all supported log format patterns
    PATTERNS = [
        (
            "structured",  # [2025-01-15 14:30:45.123] [ERROR] [module] Message
            regex.compile(r"\[(?P<ts>\d{4}-\d{2}-\d{2}T?\d{2}:\d{2}:\d{2}(?:\.\d+)?)\]\s*\[(?P<level>[A-Z]+)\]\s*\[(?P<module>[^\]]+)\]\s*(?P<message>.+)")
        ),
        (
            "apache",  # 127.0.0.1 - - [15/Jan/2025:14:30:45 +0000] "GET /path HTTP/1.1" 200 1234
            regex.compile(r"(?P<ip>[\d.]+)\s+-\s+-\s+\[(?P<ts>[^\]]+)\]\s+\"(?P<method>\w+)\s+(?P<path>\S+)\s+HTTP/[\d.]\"\s+(?P<status>\d{3})\s+(?P<size>\d+)")
        ),
        (
            "simple",  # 2025-01-15 ERROR Message here
            regex.compile(r"(?P<ts>\d{4}-\d{2}-\d{2})\s+(?P<level>[A-Z]+)\s+(?P<message>.+)")
        )
    ]
    
    def parse_line(self, line: str) -> dict | None:
        """Try each pattern in order; return first match."""
        for name, pattern in self.PATTERNS:
            m = pattern.match(line)
            if m:
                return {
                    "format": name,
                    **m.groupdict()
                }
        return None
    
    def stream_logs(self, lines: list[str]) -> Generator[dict, None, None]:
        """Yield parsed log entries from a list of raw lines."""
        for line in lines:
            entry = self.parse_line(line.strip())
            if entry:
                yield entry
    
    def filter_by_level(self, entries: list[dict], levels: set[str]) -> list[dict]:
        """Filter log entries by log level."""
        return [e for e in entries if e.get("level", "") in levels]

# Usage:
aggregator = LogAggregator()
logs = [
    "[2025-01-15 14:30:45.123] [ERROR] [auth] Connection timeout",
    "192.168.1.1 - - [15/Jan/2025:14:30:45 +0000] \"GET /api HTTP/1.1\" 200 1234",
]
for log in aggregator.stream_logs(logs):
    print(log)
```

### 18.3 Markdown Parser (Regex-Based)

```python
import regex

class RegexMarkdownParser:
    """Parse markdown elements using regex patterns."""
    
    def parse(self, markdown: str) -> dict[str, str]:
        """Extract all markdown blocks from text."""
        result = {"headings": [], "bold": [], "italic": [], "links": [], "images": [], "code_blocks": []}
        
        # Headings
        for m in regex.finditer(r"^(?P<level>#{1,6})\s+(?P<text>.+)$", markdown, regex.M):
            result["headings"].append({
                "level": len(m.group("level")),
                "text": m.group("text")
            })
        
        # Bold
        for m in regex.finditer(r"(?<!\*)\*{2}(?P<text>[^*]+)\*{2}(?!\*)", markdown):
            result["bold"].append(m.group("text"))
        
        # Italic (single asterisk)
        for m in regex.finditer(r"(?<!\*)\*(?P<text>[^*]+)\*(?!\*)", markdown):
            if f"**{m.group('text')}**" not in markdown:  # Not bold
                result["italic"].append(m.group("text"))
        
        # Links [text](url)
        for m in regex.finditer(r"\[(?P<text>[^\]]+)\]\((?P<url>[^)]+)\)", markdown):
            result["links"].append({"text": m.group("text"), "url": m.group("url")})
        
        # Images ![alt](url)
        for m in regex.finditer(r"!\[(?P<alt>[^\]]+)\]\((?P<url>[^)]+)\)", markdown):
            result["images"].append({"alt": m.group("alt"), "url": m.group("url")})
        
        # Code blocks
        for m in regex.finditer(r"```(?P<lang>\w*)\n(?P<code>[^`]+)```", markdown, regex.S):
            result["code_blocks"].append({"language": m.group("lang") or "text", "code": m.group("code")})
        
        return result
    
    def to_html(self, markdown: str) -> str:
        """Convert markdown to HTML (simplified)."""
        html = markdown
        
        # Headings -> h1-h6
        for level in range(6, 0, -1):
            pattern = rf'^{\"#{level}\"}\s+(.+)$'
            repl = f'<h{level}>\\1</h{level}>'
            html = regex.sub(pattern, repl, html, flags=regex.M)
        
        # Bold -> strong
        html = regex.sub(r'\*\*(.+?)\*\*', r'<strong>\\1</strong>', html)
        
        # Italic -> em
        html = regex.sub(r'\*(.+?)\*', r'<em>\\1</em>', html)
        
        # Links -> <a>
        html = regex.sub(r'\[([^\]]+)\]\(([^)]+)\)', r'<a href="\\2">\\1</a>', html)
        
        return html
```

### 18.4 CSV Parser with Edge Cases (Quoted Fields, Escaped Quotes)

```python
import regex

def parse_csv_line(line: str) -> list[str]:
    """Parse a single CSV line handling quoted fields and escaped quotes."""
    # Quoted field: "field with \"escaped\" quotes"
    # Unquoted field: simple text without comma
    pattern = regex.compile(
        r'^'
        r'(?:(?P<quoted>"((?:[^"\\]|\\.)*)")'
        r'|'
        r'(?P<unquoted>[^,]*)'
        r')(?:,|$)'
        + ')*',
        regex.VERBOSE
    )
    
    fields = []
    for m in pattern.finditer(line):
        if m.group("quoted"):
            # Remove surrounding quotes and unescape \"
            field = m.group("quoted").strip('"').replace('\\"', '"')
            fields.append(field)
        elif m.group("unquoted"):
            fields.append(m.group("unquoted"))
    
    return fields

# Test with edge cases:
test_lines = [
    'name,age,notes',
    'Alice,30,"Likes coffee, tea, and code"',
    '\"Smith, John\",25,"Said \"hello\" to me"',
]

for line in test_lines:
    print(parse_csv_line(line))
# ['name', 'age', 'notes']
# ['Alice', '30', 'Likes coffee, tea, and code']
# ['Smith, John', '25', 'Said "hello" to me']
```

### 18.5 Diff / Patch Parser

```python
import regex
from typing import NamedTuple

class Hunk(NamedTuple):
    old_start: int
    old_count: int
    new_start: int
    new_count: int
    lines: list[tuple[str, str]]  # (type, content): type = '+' '-' or ' '

class DiffFile(NamedTuple):
    old_path: str | None
    new_path: str | None
    hunks: list[Hunk]

def parse_unified_diff(diff_text: str) -> list[DiffFile]:
    """Parse unified diff format into structured data."""
    files = []
    file_pattern = regex.compile(
        r'diff --git a/(.*) b/(.*)\n--- (.*)\n\+\+\+(.*)',
        regex.MULTILINE
    )
    
    for file_match in file_pattern.finditer(diff_text):
        hunks = []
        hunk_pattern = regex.compile(
            r'@@ -(\d+),(\d+) \+(\d+),(\d+) @@\s*'
            r'(?P<lines>(?:[ +-].*\n)*)',
            regex.MULTILINE
        )
        
        for hunk_match in hunk_pattern.finditer(file_match.group()):
            lines = []
            for line in hunk_match.group("lines").splitlines():
                if line.startswith('+'):
                    lines.append(('+', line[1:]))  # Added
                elif line.startswith('-'):
                    lines.append(('-', line[1:]))  # Removed
                else:
                    lines.append((' ', line[1:] if len(line) > 0 else ''))  # Context
            
            hunks.append(Hunk(
                old_start=int(hunk_match.group(1)),
                old_count=int(hunk_match.group(2)),
                new_start=int(hunk_match.group(3)),
                new_count=int(hunk_match.group(4)),
                lines=lines
            ))
        
        files.append(DiffFile(
            old_path=file_match.group(1),
            new_path=file_match.group(2),
            hunks=hunks
        ))
    
    return files
```

### 18.6 URL Router Pattern Matching (Express-like)

```python
import regex
from typing import Callable, Any

class Route:
    """A single route definition."""
    def __init__(self, method: str, path_pattern: str, handler: Callable):
        self.method = method.upper()
        self.handler = handler
        # Convert Express-style :param to named groups (?P<name>[^/]+)
        regex_pattern = re.sub(r':(\w+)', r'(?P<\1>[^/]+)', path_pattern)
        regex_pattern = f'^{regex_pattern}$'
        self.pattern = re.compile(regex_pattern)
    
    def match(self, url_path: str) -> dict[str, str] | None:
        m = self.pattern.match(url_path)
        return m.groupdict() if m else None

class SimpleRouter:
    """Simple Express-style router."""
    def __init__(self):
        self.routes: list[Route] = []
    
    def get(self, path: str, handler: Callable):  pass  # def
    def post(self, path: str, handler: Callable):  pass  # def
    def add_route(self, method: str, path: str, handler: Callable):
        self.routes.append(Route(method, path, handler))
    
    def resolve(self, method: str, url_path: str) -> tuple[Callable | None, dict[str, str]]:
        for route in self.routes:
            if route.method == method.upper():
                params = route.match(url_path)
                if params:
                    return route.handler, params
        return None, {}

# Usage:
router = SimpleRouter()
router.add_route("GET", "/users/:id", lambda req: f"User {req['params']['id']}")
router.add_route("POST", "/users", lambda req: "New user created")
handler, params = router.resolve("GET", "/users/42")
print(params)  # {'id': '42'}
```

---

## 19. 🆕 Massive Expanded Quizzes & Exercises

This section contains massive new quizzes and exercises, organized by difficulty level with progressively harder challenges.

### Quiz 6: Character Class Mastery (Easy) 🟢

**Which pattern correctly matches ONLY valid US zip codes (XXXXX or XXXXX-XXXX)?**

A) `r"\d{5}"`
B) `r"\d{5}(?:-\d{4})?"`
C) `r"^\d{5}$"`  
D) `r"^\d{5}(?:-\d{4})?$"`

<details>
<summary><b>Click to reveal answer</b></summary>

**D** — Must have anchors (`^` and `$`) AND optional extension `(?:-\d{4})?`. Pattern B matches zip codes embedded in other strings.

</details>

---

**Which pattern correctly matches a hex color (with or without #)?**

A) `r"#[0-9a-f]{6}"`
B) `r"#?[0-9a-fA-F]{3,6}"`
C) `r"^(?:#[0-9a-fA-F]{3}|#[0-9a-fA-F]{6})$"`
D) `r"#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$"`

<details>
<summary><b>Click to reveal answer</b></summary>

**C** — Handles both 3-digit and 6-digit hex colors, with optional `#`, anchored. Pattern B is too loose (allows 4-5 digits which aren't valid hex colors). Pattern D allows no `#` prefix.

</details>

---

### Quiz 7: Lookahead/Lookbehind Deep Dive (Medium) 🟡

**What does this output?**

```python
import re
text = "foo1bar2baz3qux"
result = re.findall(r"(\w+)(?=\d)", text)
print(result)
```

A) `['foo', 'bar', 'baz']`
B) `['foo1', 'bar2', 'baz3']`
C) `['foo', 'bar', 'baz', 'qux']`
D) `['foo1ba', 'r2ba', 'z3qu']`

<details>
<summary><b>Click to reveal answer</b></summary>

**A** — The lookahead `(?=\d)` checks for a digit WITHOUT consuming it. So each `\w+` stops right before the digit, capturing only the word characters before each number.

</details>

---

**What's the output of this Python code?**

```python
import re
text = "item1 item2 item3"
result = re.findall(r"\b(?<=item\s)\d+", text)
print(result)
```

A) `['item1', 'item2', 'item3']`  
B) `['1', '2', '3']`  
C) `['item ']`  
D) `[]`

<details>
<summary><b>Click to reveal answer</b></summary>

**B** — The lookbehind `(?<=item\s)` asserts that "item " precedes the match position. `\d+` then matches only the numbers. The lookbehind itself doesn't consume characters.

</details>

---

### Quiz 8: Code Tracing Challenges (Hard) 🔴

**What does this output?**

```python
import re

# Pattern with multiple capture groups and alternation
text = "foo123bar456baz"
pattern = re.compile(r"((foo)|(bar)|(baz))(\d+)")
m = pattern.search(text)

print(m.groups())       # Line 1
print(m.lastindex)      # Line 2
print(m.lastgroup)      # Line 3
```

<details>
<summary><b>Click to reveal answer</b></summary>

**Line 1:** `('foo', 'foo', None, None, '123')` -- groups() returns ALL capture groups in order: group 1 (full alternation), group 2 (foo branch - matched), group 3 (bar branch - not matched = None), group 4 (baz branch - not matched = None), group 5 (the digits).

**Line 2:** `5` -- last numbered group that participated in the match.

**Line 3:** `None` -- no named groups were used, so there's no last named group.

</details>

---

**What does this TypeScript code output?**

```typescript
const text = "Hello World! Hello TypeScript!";
const regex = /Hello/g;

console.log(text.match(regex));          // Line A
console.log(regex.test(text));           // Line B
console.log(regex.lastIndex);            // Line C
console.log(regex.test(text));           // Line D
```

<details>
<summary><b>Click to reveal answer</b></summary>

**Line A:** `['Hello', 'Hello']` -- matchAll with global flag returns all matches.

**Line B:** `true` -- first test() succeeds (found at position 0), advances lastIndex to 6.

**Line C:** `12` -- lastIndex is now at the second "Hello" position (after "Hello World! ").

**Line D:** `true` -- finds the second "Hello" at position 12, advances lastIndex again.

If you called test() a third time: `false` -- no more matches, resets lastIndex to 0.

</details>

---

### Quiz 9: Pattern Building Challenges (Hard) 🔴

**Write regex for each requirement:**

<details>
<summary><b>Q1:</b> Match phone numbers in ANY of these formats: +1-555-123-4567, (555) 123-4567, 555.123.4567</summary>

```python
# Python:
r'^(?:\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$'

// TypeScript:
/^(?:\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$/
```

This handles all three formats by making each separator optional and flexible.

</details>

<details>
<summary><b>Q2:</b> Extract the filename from a full path (cross-platform: /home/user/file.txt or C:\Users\file.txt)</summary>

```python
# Python:
r'[^/\\]+$'

// TypeScript:
/[^/\\]+$/
```

Simple! Match everything from the last slash or backslash to the end.

</details>

<details>
<summary><b>Q3:</b> Validate a GitHub issue number format: #12345 (just the hash + digits)</summary>

```python
# Python:
r'^#[0-9]+$'

// TypeScript:
/^#[0-9]+$/
```

GitHub uses sequential integer IDs with `#` prefix.

</details>

<details>
<summary><b>Q4:</b> Match balanced parentheses up to depth 3 (not infinite nesting!)</summary>

```python
# Python:
r'^\([^()]*\)(?:\([^()]*\))*$'

// TypeScript:
/^\([^()]*\)(?:\([^()]*\))*$/
```

This matches: `(a)`, `(a)(b)`, `((a))` -- but NOT deeply nested like `(((a)))`. For arbitrary nesting, you need the `regex` module's `(?R)` or a proper parser.

</details>

<details>
<summary><b>Q5:</b> Extract all CSS property names from a block: `color: red; font-size: 14px; margin: 0`</summary>

```python
# Python:
r'([a-z-]+):'

// TypeScript:
/([a-z-]+):/g
```

Property names are hyphenated lowercase identifiers followed by `:`.

</details>

<details>
<summary><b>Q6:</b> Match a UUID v4: 8-4-4-4-12 hex format with specific version/bits pattern</summary>

```python
# Python:
r'^[0-9a-f]{8}-[0-9a-f]{4}-4[a-f0-9]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$'

// TypeScript:
/^[0-9a-f]{8}-[0-9a-f]{4}-4[a-f0-9]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
```

UUID v4 specific rules: version 4 in the third group (`4`), variant bits in fourth group (`8`, `9`, `a`, or `b`).

</details>

<details>
<summary><b>Q7:</b> Match an HTTP URL with optional port, path, query params, and fragment</summary>

```python
# Python:
r'^(?P<proto>https?)://(?P<host>[^:/\s]+)(?::(?P<port>\d+))?(?P<path>/[^\s]*)?(?:\?(?P<query>[^\s]*))?(?:#(?P<hash>[^\s]*))?$'

// TypeScript:
/^(?<proto>https?):\/\/(?<host>[^:/\s]+)(?::(?<port>\d+))?(?<path>\/[^\s]*)?(?:\?(?<query>[^\s]*))?(?:#(?<hash>[^\s]*))?$/
```

Full URL validation with all components as named groups.

</details>

<details>
<summary><b>Q8:</b> Strip HTML tags (handling nested tags, attributes with `>` in values) -- the CORRECT way</summary>

```python
# Python: This pattern is still fragile for complex HTML!
# The correct production approach uses BeautifulSoup or DOMParser, NOT regex.
# But for simple cases:
r'<[^>]+>'

// TypeScript (same limitation):
/<[^>]+>/g
```

⚠️ **WARNING**: Regex CANNOT correctly parse arbitrary HTML. This pattern works for simple tags but fails on `>` inside attributes like `<img alt='a > b'>`. Use proper HTML parsers in production!

</details>

<details>
<summary><b>Q9:</b> Extract email local parts (before @) from emails in text, only if the domain is "example.com" or subdomains thereof</summary>

```python
# Python:
r'(?<=@)((?:[a-z0-9-]+\.)?example\.com)'
// TypeScript: Fixed-width lookbehind requires care!
r'@(?:[a-z0-9-]+\.)?example\.com"

// Actually, variable-width lookbehind only works in Python:
# Use lookahead instead for TS:
r'[a-zA-Z0-9_.+-]+(?=@(?:[a-zA-Z0-9-]+\.)?example\.com)'
```

This requires careful use of lookarounds to extract the local part while validating the domain.

</details>

<details>
<summary><b>Q10:</b> Match all lines in a multi-line string that contain an HTTP status code (3-digit number not embedded in other digits)</summary>

```python
# Python:
r'^.*\b(\d{3})\b.*$'  with re.MULTILINE flag
// TypeScript:
/^.*\b(\d{3})\b.*$/gm
```

This finds lines containing exactly a 3-digit number as a standalone value (not part of a larger number).

</details>

---

### Quiz 10: True/False Rapid-Fire 🟡

| # | Statement | True / False | Explanation |
|---|----------|-------------|-------------|
| 1 | Python's `re.findall(r"(\d+)", "abc123")` returns `["123"]`. | — | |
| 2 | TypeScript's `sticky` flag (`y`) is equivalent to anchoring with `^` in every way. | — | |
| 3 | `(?P<name>...)` in Python and `(?<name>...)` in TypeScript are functionally identical. | — | |
| 4 | Both Python's re and TS regex support variable-width lookbehinds natively. | — | |
| 5 | Possessive quantifiers exist in both Python's stdlib `re` and TypeScript RegExp. | — | |
| 6 | The pattern `(?:abc)` captures the matched text into group 1. | — | |
| 7 | In Python, `r"\d"` and `"\\d"` produce the same regex pattern string. | — | |
| 8 | TypeScript's `/pattern/s` flag makes `.` match newlines (dotall). | — | |
| 9 | `re.match()` in Python is equivalent to using `^` anchor in `re.search()`. | — | |
| 10 | Both languages treat `_` as a word character for `\b` boundaries. | — | |
| 11 | Python's `≥` pattern matches Unicode property escapes natively in stdlib re. | — | |
| 12 | In both languages, unescaped `(` and `)` inside character classes `[...]` are literal characters. | — | |

<details>
<summary><b>Click to reveal all answers</b></summary>

1. **False** -- Returns `[('123',)]` (a tuple with one string element) because there's a capture group. Without the group, it would be `["123"]`.

2. **False** -- Sticky only matches at `lastIndex`, while `^` matches at position 0 or start of each line (with `m` flag). Different semantics!

3. **True** -- Both create named capture groups, just different syntax conventions.

4. **False** -- Only Python's stdlib `re` supports variable-width lookbehinds. TypeScript requires fixed-width only.

5. **False** -- Neither supports possessive quantifiers natively. Python's third-party `regex` module does; TypeScript doesn't have them at all.

6. **False** -- `(?:abc)` is a NON-capturing group. It groups logic without capturing into any group number.

7. **True** -- Both produce the string `\d` which regex interprets as a digit class.

8. **True** -- The `s` flag (dotall) makes `.` match newlines.

9. **True** -- `re.match()` implicitly anchors at position 0, equivalent to prepending `^`.

10. **True** -- `_` is always considered a word character (`\w`) in both languages.

11. **False** -- Python's stdlib `re` does NOT support `≥` (Unicode property escapes). Use the third-party `regex` module.

12. **True** -- Inside `[...]`, most metacharacters lose their special meaning. Parentheses are literal inside character classes.

</details>

---

### Quiz 11: Cross-Language Pattern Translation Challenge 🟡🔴

For each TypeScript pattern, write the equivalent Python `re` pattern (or vice versa):

<details>
<summary><b>T1:</b> TS: `/[a-z]+@[a-z]+\.[a-z]{2,}/i` -- Write Python equivalent</summary>

```python
# Answer:
r'[a-z]+@[a-z]+\.[a-z]{2,}'
# With re.IGNORECASE flag:
re.findall(r'[a-z]+@[a-z]+\.[a-z]{2,}', text, re.IGNORECASE)
```

</details>

<details>
<summary><b>T2:</b> Python: `re.search(r'^\d{3}-\d{2}-\d{4}$', text)` -- Write TypeScript equivalent</summary>

```typescript
// Answer:
/^\d{3}-\d{2}-\d{4}$/.test(text)
```

</details>

<details>
<summary><b>T3:</b> TS: `/(?=https?:)[^\s]+/` -- Write Python equivalent</summary>

```python
# Answer:
r'(?=https?:)\S+'
# Note: \S is the same in both languages (non-whitespace)
```

</details>

<details>
<summary><b>T4:</b> Python: `re.sub(r'([A-Z])([a-z]+)', r'\1_\2', text)` -- Write TypeScript equivalent</summary>

```typescript
// Answer:
text.replace(/([A-Z])([a-z]+)/g, '$1_$2')
```

Note the different backreference syntax: `\1` in Python vs `$1` in TS replacement strings.

</details>

<details>
<summary><b>T5:</b> TypeScript: `/^(?:(?:https?|ftp):\/\/)?([a-zA-Z0-9.-]+)(:[\d]+)?(\/.*)?$/` -- Write Python equivalent</summary>

```python
# Answer:
r'^(?:(?:https?|ftp):\/\/)?([a-zA-Z0-9.-]+)(:\d+)?(/.*)?$'
```

Clean pattern for URL component extraction!

</details>

<details>
<summary><b>T6:</b> Python: `re.finditer(r'\b(?P<word>[a-z]+)\b', text)` -- Write TypeScript equivalent with proper structure</summary>

```typescript
// Answer:
const matches = [...text.matchAll(/\b(?<word>[a-z]+)\b/g)];
// Each match: [fullText, word: capturedGroup, index, input, groups: { word }]
```

Note: TypeScript's `matchAll` returns an array of match objects, each with a `groups` property for named captures.

</details>

---

### Quiz 12: Performance Challenge ⚡🔴

**Which pattern is FASTEST for extracting emails from a 10MB text file?**

A) `re.findall(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", huge_text)`
B) `re.compile(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}").findall(huge_text)`
C) `re.findall(r".*@.*", huge_text)`  
D) `re.finditer(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", huge_text)` -- process lazily

<details>
<summary><b>Click to reveal answer</b></summary>

**B or D** -- Both are fast because they use a compiled pattern. **B** is faster for small text (all results in memory). **D** is faster for massive text (lazy evaluation, no full list in memory).

A is slower because `re.findall()` compiles the pattern every time.

C is WRONG -- `.*@.*` matches everything with an @, including URLs, usernames, etc. Way too loose!

</details>

---

### Exercise 13: Build a Regex-Powered URL Shortener (Full Implementation)

Write a function that takes a long URL and generates a short code using regex:

<details>
<summary><b>Solution in Python</b></summary>

```python
import re
import base64
import hashlib

def validate_url(url: str) -> bool:
    pattern = r'^(https?:\/\/)?([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})(:[\d]+)?(\/[\w.-]*)*(\?[\w=&%_-]*)?(#[\w-]*)?$'
    return bool(re.fullmatch(pattern, url))

def short_url_to_code(url: str) -> str:
    # Validate first
    if not validate_url(url):
        raise ValueError("Invalid URL")
    
    # Generate a hash-based short code (Base62 encoded)
    hash_obj = hashlib.sha256(url.encode())
    short_hash = hash_obj.hexdigest()[:8]  # First 8 hex chars
    return short_hash.upper()

# Example:
url = "https://example.com/path/to/resource?id=123&name=test"
print(short_url_to_code(url))  # 'A3F7B2C1'
print(validate_url("not-a-url"))  # False
```

</details>

<details>
<summary><b>Solution in TypeScript</b></summary>

```typescript
function validateUrl(url: string): boolean {
    const pattern = /^(https?:\/\/)?([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})(:\d+)?(\/[\w.-]*)*(\?[\w=&%_-]*)?(#[\w-]*)?$/;
    return pattern.test(url);
}

function shortUrlToCode(url: string): string {
    if (!validateUrl(url)) {
        throw new Error("Invalid URL");
    }
    
    // Simple hash-based approach (same as Python version)
    let hash = 0;
    for (let i = 0; i < url.length; i++) {
        const char = url.charCodeAt(i);
        hash = ((hash << 5) - hash) + char;
        hash = hash & hash;  // Convert to 32-bit integer
    }
    
    return Math.abs(hash).toString(16).toUpperCase().padStart(8, '0').slice(0, 8);
}

console.log(shortUrlToCode("https://example.com/path/to/resource?id=123"));
// 'A3F7B2C1' (same as Python if hash algorithm matches)
```

</details>

---

### Exercise 14: Regex Performance Benchmark Suite

Compare regex patterns across languages and identify bottlenecks:

<details>
<summary><b>Benchmark: Email Extraction Speed</b></summary>

```python
# Python benchmark script
import re, timeit

test_text = " ".join([f"user{i}@example{i % 100}.com" for i in range(10000)])
compiled = re.compile(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}")

# Compiled pattern:
time_compiled = timeit.timeit(
    lambda: compiled.findall(test_text),
    number=100
)
print(f"Compiled: {time_compiled:.4f}s")

# Non-compiled:
time_raw = timeit.timeit(
    lambda: re.findall(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", test_text),
    number=100
)
print(f"Non-compiled: {time_raw:.4f}s")
print(f"Speedup: {time_raw/time_compiled:.1f}x")  # Typically 3-5x speedup!
```

```typescript
// TypeScript benchmark
const testText = Array.from({length: 10000}, (_, i) => `user${i}@example${i % 100}.com`).join(' ');
const emailRegex = /[a-zA-Z0-9_.+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g;

// First run (compilation):
const start1 = performance.now();
emailRegex.test(testText);  // forces compilation
for (let i = 0; i < 99; i++) emailRegex.exec(testText);
const end1 = performance.now();

// Second run (cached):
emailRegex.lastIndex = 0;
const start2 = performance.now();
for (let i = 0; i < 100; i++) {
    emailRegex.test(testText);  // V8 caches the compiled regex automatically
}
const end2 = performance.now();

console.log(`V8 auto-cached: ${end2 - start1}ms`);
// V8 handles regex caching internally -- no explicit compile() needed!
```

</details>

---

### Exercise 15: The Regex Golf Challenge 🏆

Write the SHORTEST pattern that matches each:

<details>
<summary><b>Challenge 1:</b> Match any valid IPv4 address (not validate, just match)</summary>

```python
# Python & TS share this simple version:
r'\b(?:\d{1,3}\.){3}\d{1,3}\b'
// TypeScript: /\b(?:\d{1,3}\.){3}\d{1,3}\b/
```

For validation (0-255 per octet), see Section 7.1 for the complete pattern.

</details>

<details>
<summary><b>Challenge 2:</b> Match any line that contains BOTH "error" and "timeout" (in any order)</summary>

```python
# Python:
r'^(?=.*error)(?=.*timeout).*$'  with re.IGNORECASE | re.MULTILINE
// TypeScript: /^(?=.*error)(?=.*timeout).*$/gi
```

Two positive lookaheads at position 0 check for both words independently.

</details>

<details>
<summary><b>Challenge 3:</b> Match balanced single-quoted strings (no nested quotes!)</summary>

```python
# Python:
r"'([^']*)'"
// TypeScript: /'([^']*)'/
```

Note: This doesn't handle escaped quotes within the string. For that, use `'((?:[^'\\]|\\.)*)'`.

</details>

<details>
<summary><b>Challenge 4:</b> Match a semver range like >=1.2.3 <2.0.0</summary>

```python
# Python:
r'^[><=]?\s*\d+\.\d+\.\d+(?:-[\w.-]+)?(?:\+[\w.-]+)?$'
// TypeScript: /^[><=]?\s*\d+\.\d+\.\d+(?:-[\w.-]+)?(?:\+[\w.-]+)?$/
```

</details>

<details>
<summary><b>Challenge 5:</b> Match CSS custom properties like --my-custom-prop: value</summary>

```python
# Python:
r'(--[a-z][a-z0-9-]*):\s*(.+?)(?=;|$)'
// TypeScript: /(--[a-z][a-z0-9-]*):\s*(.+?)(?=;|$)/g
```

Custom properties always start with `--` per the CSS spec.

</details>

---

## Summary Table: Python vs TypeScript Regex at a Glance

| Category | Python | TypeScript | Best For |
|----------|--------|-----------|----------|
| **Pattern syntax** | `r"pattern"` (raw strings) | `/pattern/flags` (regex literals) | Python raw strings are cleaner for complex patterns |
| **Compile step** | Manual (`re.compile()`) | Automatic (V8 caches) | TS is easier for simple use; Python wins for repeated use in loops |
| **Named groups** | `(?P<name>...)` | `(?<name>...)` | Both work well, different syntax |
| **Lookbehind** | Variable width OK | Fixed width only | Python is more flexible here |
| **Verbose mode** | ✅ `re.VERBOSE` | ❌ Not supported | Huge readability win for complex patterns in Python |
| **Unicode support** | Third-party `regex` module needed | ✅ Native with `u` flag (ES2018+) | TypeScript wins for Unicode property escapes |
| **String methods** | Separate `re` module | Built-in on all strings | Different mental models: `re.search()` vs `str.match()` |
| **Replace backrefs** | `\1`, `\g<name>` | `$1`, `$&`, `$'` | Never mix them between languages! |
| **Sticky flag** | ❌ Not supported | ✅ `y` flag (unique!) | Only in TypeScript for tokenizer-style matching |
| **Atomic groups** | Third-party `regex` module only | ❌ Not supported | Python's `regex` wins |
| **Recursive patterns** | Third-party `regex` module only | ❌ Not supported | Python's `regex` wins |
| **Possessive quantifiers** | Third-party `regex` module only | ❌ Not supported | Prevents catastrophic backtracking in Python |
| **Best use case** | Complex pattern building, log parsing | Quick string operations, tokenization | Both excel in their respective ecosystems |

---

## Quick Reference Card: Common Patterns

| Goal | Python (`re`) | TypeScript (`RegExp`) |
|------|--------------|---------------------|
| **Email match** | `r"[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}"` | `/[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}/g` |
| **URL match** | `r"https?://[^\s]+"` | `/https?:\/\/[^\s]+/g` |
| **Phone (US)** | `r"^\+?1?[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$"` | Same with `/.../` literals |
| **UUID** | `r"[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}"` | Same (add `re.I` / `i`) |
| **SemVer** | `r"^\d+\.\d+\.\d+(?:-[\w.]+)?(?:\+[\w.]+)?$"` | Same |
| **IPv4** | `r"^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$"` | Same |
| **Hex color** | `r"^#([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$"` | Same (add `re.I`) |
| **Date ISO** | `r"^\d{4}-\d{2}-\d{2}$"` | Same |
| **MAC addr** | `r"^([0-9a-fA-F]{2}[:-]){5}[0-9a-fA-F]{2}$"` | Same (add `re.I`) |
| **Slack ID** | `r"^U[A-Z0-9]{8,}"` | Same |
| **JWT token** | `r"^[\w-]+\.[\w-]+\.[\w-]+$"` | Same |
| **Git hash** | `r"\b[0-9a-f]{7,40}\b"` | Same |
| **Phone (E.164)** | `r"^\+[1-9]\d{1,14}$"` | Same |
| **ISO 8601 datetime** | `r"^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(?:\.\d+)?(?:Z|[+-]\d{2}:?\d{2})?$"` | Same |
| **CRAZY**: Password with lookaheads | `r"^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&#]).{8,}$"` | Same |

---

> **You've completed Module 19!** This is now a comprehensive v2 reference covering every regex pattern, method, flag, edge case, and production use case across both Python `re` and TypeScript RegExp. Use the summary tables for quick lookups, the exercises for practice, and the visual breakdowns to understand how patterns work step by step.
