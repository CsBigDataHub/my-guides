# From Python to Nim: A Comprehensive Guide

## Table of Contents

1. [Mental Model Shift](#mental-model-shift)
2. [The Environment](#the-environment)
3. [Basic Syntax](#basic-syntax)
4. [Data Types](#data-types)
5. [Variables and Binding](#variables-and-binding)
6. [Functions](#functions)
7. [Control Flow](#control-flow)
8. [Collections](#collections)
9. [Strings](#strings)
10. [Object-Oriented Programming](#object-oriented-programming)
11. [Generics](#generics)
12. [Templates and Macros](#templates-and-macros)
13. [Error Handling](#error-handling)
14. [Concurrency](#concurrency)
15. [Memory Management](#memory-management)
16. [C Interop](#c-interop)
17. [I/O and Files](#io-and-files)
18. [Modules and Packages](#modules-and-packages)
19. [Testing](#testing)
20. [Build System](#build-system)
21. [Common Patterns and Idioms](#common-patterns-and-idioms)
22. [Metaprogramming](#metaprogramming)
23. [Debugging and Profiling](#debugging-and-profiling)
24. [Style and Conventions](#style-and-conventions)

---

## 1. Mental Model Shift

This is the most important section. Before writing a single line, understand
how Nim differs fundamentally from Python.

### Python is dynamic and interpreted. Nim is static and compiled.

Python figures out types at runtime, catches errors at runtime, and runs
through an interpreter. Nim resolves types at compile time, catches most
errors before your program ever runs, and compiles to native machine code
or C/JavaScript.

```python
# Python - discovered at runtime
def add(a, b):
    return a + b

add("hello", 5)  # runs, then crashes with TypeError
```

```nim
# Nim - caught at compile time
proc add(a, b: int): int =
  a + b

add("hello", 5)  # compile error: type mismatch
```

### The Three Core Differences

**1. Static typing with inference** — Every value has a type known at
compile time. You don't always have to write the type (inference fills it
in) but the compiler always knows it. This eliminates an entire class of
runtime bugs.

**2. Manual-ish memory with automatic safety** — Nim uses a garbage
collector by default but gives you control when you need it. You can also
use destructors, move semantics, or manual allocation for performance.
No GIL means true parallelism.

**3. Zero-cost abstractions** — Templates, macros, inline functions, and
generics generate efficient native code. High-level code compiles to the
same machine code as hand-written C. You don't pay for abstractions you
use.

### Nim's Unusual Position

Nim sits in a unique spot in the language landscape:

- **Syntax like Python**: indentation-based, readable, expressive
- **Performance like C**: compiles to C then to native code
- **Metaprogramming like Lisp**: macros that operate on the AST
- **Safety like modern languages**: no null pointer surprises, bounds checking

```
Python:  easy to write, slow, dynamic
C:       fast, manual memory, painful
Rust:    fast, safe, steep learning curve
Nim:     fast, safe-ish, Python-like syntax, powerful macros
```

### Compilation Pipeline

```
your .nim file
      ↓
  Nim compiler
      ↓
  C source code   (or C++, or JavaScript)
      ↓
  C compiler (gcc/clang)
      ↓
  native binary
```

This means:

- Your code runs as fast as C
- You can call any C library trivially
- You can inspect the generated C if needed
- Startup time is near-instant (no interpreter)

---

## 2. The Environment

### Installation

```bash
# Using choosenim (recommended — like pyenv for Nim)
curl https://nim-lang.org/choosenim/init.sh -sSf | sh

# Or via package manager
brew install nim          # macOS
apt install nim           # Ubuntu/Debian (may be older version)
pacman -S nim             # Arch Linux

# Verify
nim --version
nimble --version          # package manager (like pip)
```

### Project Structure

```
myproject/
├── myproject.nimble      # project file (like pyproject.toml)
├── src/
│   └── myproject.nim     # main module
├── tests/
│   └── test_myproject.nim
└── README.md
```

```nim
# myproject.nimble
version     = "0.1.0"
author      = "Your Name"
description = "My Nim project"
license     = "MIT"
srcDir      = "src"
bin         = @["myproject"]

requires "nim >= 2.0.0"
requires "some_package >= 1.0.0"
```

Python equivalent:

```toml
# pyproject.toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["some_package>=1.0.0"]
```

### Nimble: The Package Manager

```bash
# Install a package (like pip install)
nimble install some_package

# Install project dependencies (like pip install -e .)
nimble install

# Create new project
nimble init myproject

# Run the project
nimble run

# Build release binary
nimble build -d:release

# Run tests
nimble test

# Search packages
nimble search http
```

### Compilation and Running

```bash
# Compile and run (like python script.py)
nim r myfile.nim

# Compile only
nim c myfile.nim           # compile to native binary
nim c -o:mybin myfile.nim  # specify output name

# Release build (optimized, no debug info)
nim c -d:release myfile.nim

# Compile to JavaScript
nim js myfile.nim

# Compile to C (then inspect the C)
nim c --genScript myfile.nim

# With extra flags
nim c -d:release --opt:speed -d:danger myfile.nim
# -d:danger: remove all runtime checks (max speed)
```

### Editors

- **VS Code + nim extension**: best for beginners
- **Emacs + nim-mode**: great integration
- **Neovim + nim LSP**: excellent
- All support nimlsp (Nim Language Server Protocol)

---

## 3. Basic Syntax

### Indentation-Based Like Python

Nim uses indentation for blocks, just like Python. The main difference is
that statement terminators and block markers differ:

```python
# Python
def greet(name):
    if name == "Alice":
        print("Hello, Alice!")
    else:
        print(f"Hello, {name}!")
```

```nim
# Nim
proc greet(name: string) =
  if name == "Alice":
    echo "Hello, Alice!"
  else:
    echo "Hello, " & name & "!"
```

Key syntax differences:

- `proc` instead of `def`
- Type annotations are required in signatures
- `=` after proc/if/for etc. (like a block opener)
- `echo` instead of `print`
- `&` for string concatenation
- No parentheses required in if/while conditions

### Statements and Expressions

```nim
# Most things are expressions in Nim
let x = if condition: 1 else: 2

let result = block:
  let a = compute1()
  let b = compute2()
  a + b   # last expression is the block's value

# Discarding values you don't use
discard someFunction()   # explicit discard required (prevents accidents)
```

### Comments

```nim
# Single line comment

#[
  Multi-line comment
  can span many lines
]#

## Doc comment for procedures and types (appears in docs)
proc foo() =
  ## This is a documentation comment.
  ## It will appear in generated HTML docs.
  discard
```

### Case Sensitivity and Identifier Rules

Nim has **partial case insensitivity**. This is unusual and important:

```nim
# These are ALL the same identifier in Nim:
myVariable
myvariable
my_variable
MyVariable
MY_VARIABLE
myVARIABLE

# The first underscore and first character case matter for style
# But internally Nim strips underscores and lowercases everything
# for comparison purposes

# Except: the FIRST character is case-sensitive
# myVar ≠ MyVar for public/private distinction

# Underscores within names are ignored:
my_proc == myProc  # same identifier!
```

This means you can call a `snake_case` C library function using
`camelCase` in your Nim code — they're the same.

### Semicolons and Line Continuation

```nim
# Semicolons optional (can use to put multiple statements on one line)
let x = 1; let y = 2; let z = 3

# Line continuation: indent the next line
let result = someReallyLongFunctionName(argument1,
                                        argument2,
                                        argument3)

# Or use parentheses for grouping
let result = (someReallyLongFunctionName(arg1, arg2) +
              anotherFunction(arg3))
```

---

## 4. Data Types

### Type Comparison Table

| Python      | Nim                    | Notes                         |
| ----------- | ---------------------- | ----------------------------- |
| `int`       | `int`                  | platform native size          |
| `int`       | `int8/16/32/64`        | explicit size variants        |
| `int`       | `uint/uint8/16/32/64`  | unsigned variants             |
| `float`     | `float` / `float32/64` |                               |
| `complex`   | no built-in            | use library                   |
| `str`       | `string`               | mutable, UTF-8                |
| `bool`      | `bool`                 | `true`/`false`                |
| `None`      | `nil`                  | only for ref types            |
| `bytes`     | `seq[byte]`            |                               |
| `list`      | `seq[T]`               | dynamic array                 |
| `tuple`     | `tuple`                | named or unnamed              |
| `dict`      | `Table[K,V]`           | from tables module            |
| `set`       | `HashSet[T]`           | from sets module              |
| `frozenset` | `set[T]`               | built-in bit set (small ints) |
| class       | `object`               |                               |
| enum        | `enum`                 |                               |

### Integer Types

```nim
# Default int is platform-native (64-bit on 64-bit systems)
var x: int = 42
var y = 42          # inferred as int

# Explicit sizes
var a: int8  = 127
var b: int16 = 32767
var c: int32 = 2147483647
var d: int64 = 9223372036854775807

# Unsigned
var u: uint = 42
var u8: uint8 = 255

# Literals
let decimal    = 1_000_000   # underscore separator
let hex        = 0xFF
let octal      = 0o77
let binary     = 0b1010_1010

# Arithmetic
echo 10 div 3    # → 3   integer division (not / which is float)
echo 10 mod 3    # → 1   modulo
echo 2 ^ 10      # → 1024  power

# Type conversion (explicit, no implicit int→float)
let f: float = float(42)
let i: int   = int(3.14)    # truncates
```

### Float Types

```nim
var x: float  = 3.14     # float64 by default
var y: float32 = 3.14f   # float32

# Scientific notation
let avogadro = 6.022e23

# Arithmetic
echo 10.0 / 3.0    # → 3.3333...
echo sqrt(16.0)    # → 4.0
echo pow(2.0, 10)  # → 1024.0

import std/math
echo PI            # → 3.14159...
echo sin(PI / 2)   # → 1.0
echo floor(3.7)    # → 3.0
echo ceil(3.2)     # → 4.0
```

### Booleans

```nim
var t: bool = true
var f: bool = false

# Operators
echo true and false    # → false
echo true or false     # → true
echo not true          # → false
echo true xor false    # → true

# Unlike Python, only bool works in conditions
# 0 is NOT false, "" is NOT false
if 0:          # compile error!
  echo "nope"

if 0 != 0:     # correct
  echo "nope"
```

### Characters

```nim
var c: char = 'A'
echo c.ord       # → 65  (like Python's ord())
echo char(65)    # → 'A' (like Python's chr())

# Char predicates (import strutils)
import std/strutils
echo isAlpha('A')    # → true
echo isDigit('5')    # → true
echo isSpace(' ')    # → true
echo toLowerAscii('A') # → 'a'
```

### Enumerations

```nim
type
  Direction = enum
    North, South, East, West

  Color = enum
    Red = 1
    Green = 2
    Blue = 4    # explicit values

var d: Direction = North
echo d           # → North
echo ord(d)      # → 0

# Iteration over enum
for dir in Direction:
  echo dir

# Range check
echo d in {North, South}  # → true (set of enums)

# String conversion
echo $d          # → "North"
```

### Tuples

```nim
# Unnamed tuple
var point: tuple[x, y: int] = (10, 20)
echo point.x    # → 10
echo point[0]   # → 10 (index access too)

# Named tuple
type Point = tuple[x, y: float]
var p: Point = (x: 1.0, y: 2.0)

# Tuple unpacking (like Python)
let (a, b) = (1, 2)
let (first, second, third) = ("one", "two", "three")

# Proc returning multiple values
proc divmod(a, b: int): tuple[quotient, remainder: int] =
  (a div b, a mod b)

let (q, r) = divmod(17, 5)
echo q    # → 3
echo r    # → 2

# Swap using tuple (like Python's a, b = b, a)
var x = 1
var y = 2
(x, y) = (y, x)
```

---

## 5. Variables and Binding

### let, var, and const

This is a critical difference from Python. Nim has three ways to bind a
name to a value, and mutability is explicit:

```python
# Python - everything is mutable by default
x = 42
x = 100    # fine, just rebind
```

```nim
# Nim - must declare mutability intent

# let: immutable binding (like Rust's let or Python's "final" convention)
let x = 42
x = 100    # COMPILE ERROR: cannot assign to immutable

# var: mutable binding (like Python's normal variables)
var y = 42
y = 100    # fine

# const: compile-time constant (evaluated by the compiler, not at runtime)
const MaxSize = 1024
const Pi = 3.14159265358979

# const can include compile-time computations:
const Doubled = MaxSize * 2   # computed at compile time
```

### Type Inference

```nim
# Type is inferred from the right-hand side
let x = 42           # int
let y = 3.14         # float
let s = "hello"      # string
let b = true         # bool
let lst = @[1,2,3]   # seq[int]

# Explicit type annotation when you need it
let z: float = 42    # 42 is converted to 42.0
let big: int64 = 42  # forces int64 instead of default int

# Check inferred type at compile time
static: echo typeof(x)   # prints "int" during compilation
```

### Variable Scoping

```nim
# Block scoping like most languages (NOT Python's function scoping)
block:
  let x = 10
  echo x    # fine

echo x    # ERROR: x is out of scope

# Named blocks (can break out of them)
block outer:
  for i in 0..10:
    if i == 5:
      break outer   # break out of the named block
  echo "This runs only if no break"

# Variable shadowing is allowed
let x = 10
block:
  let x = 20    # shadows outer x
  echo x        # → 20
echo x          # → 10
```

### Global Variables

```nim
# Module-level variables are global to the module
var globalCounter = 0

proc increment() =
  globalCounter += 1    # access global

# Thread-local globals
var threadLocalVar {.threadvar.}: int
```

---

## 6. Functions

### Procedure Declaration

```python
def greet(name: str, times: int = 1) -> str:
    """Greet someone."""
    return (f"Hello, {name}!\n") * times
```

```nim
proc greet(name: string, times: int = 1): string =
  ## Greet someone.
  result = ""
  for i in 0..<times:
    result &= "Hello, " & name & "!\n"
```

Key differences:

- `proc` keyword
- Types after parameter names with `:`
- Return type after `):`
- `result` is a magic implicit return variable
- Or use explicit `return`

### The `result` Variable

Nim has a special `result` variable that is the implicit return value:

```nim
proc factorial(n: int): int =
  result = 1              # result starts as default (0 for int)
  for i in 1..n:
    result *= i

# Equivalent using return:
proc factorial2(n: int): int =
  var acc = 1
  for i in 1..n:
    acc *= i
  return acc

# Single-expression form:
proc double(x: int): int = x * 2
```

### Parameter Passing

```nim
# By value (default) - copy is made
proc addOne(x: int): int =
  x + 1   # doesn't modify original

# By reference with var - can modify original
proc addOneInPlace(x: var int) =
  x += 1

var n = 5
addOneInPlace(n)
echo n    # → 6

# Immutable reference with lent (read-only, no copy)
proc printLen(s: lent string) =
  echo s.len

# This avoids copying large strings/objects

# openArray: accept any array-like (seq, array, string)
proc sumAll(nums: openArray[int]): int =
  for n in nums:
    result += n

sumAll([1, 2, 3])       # array
sumAll(@[1, 2, 3])      # seq
```

### Function Overloading

```python
# Python - no real overloading, use *args or type checks
def process(data):
    if isinstance(data, str):
        return data.upper()
    elif isinstance(data, int):
        return data * 2
```

```nim
# Nim - true overloading by parameter types
proc process(data: string): string =
  data.toUpperAscii()

proc process(data: int): int =
  data * 2

echo process("hello")   # → "HELLO"
echo process(5)         # → 10

# Overloading on return type is NOT allowed
# Overloading on argument count is allowed
proc foo(x: int): int = x
proc foo(x, y: int): int = x + y
```

### Named and Default Arguments

```nim
proc createUser(name: string,
                age: int = 18,
                email: string = "",
                active: bool = true): string =
  result = name & " (age: " & $age & ")"

# Call with named arguments (any order)
createUser(name = "Alice", age = 30)
createUser(age = 25, name = "Bob", email = "bob@example.com")
createUser("Charlie")   # uses all defaults
```

### Variadic Functions

```python
def sum_all(*args):
    return sum(args)
```

```nim
proc sumAll(args: varargs[int]): int =
  for arg in args:
    result += arg

echo sumAll(1, 2, 3, 4, 5)   # → 15

# varargs with converter
proc printAll(args: varargs[string, `$`]) =
  # `$` converts each arg to string
  for arg in args:
    echo arg

printAll(1, 2.5, "hello", true)   # all converted to string
```

### First-Class Functions

```python
# Python
def apply(f, x):
    return f(x)

apply(lambda x: x * 2, 5)
```

```nim
# Nim - proc type annotation
proc apply(f: proc(x: int): int, x: int): int =
  f(x)

apply(proc(x: int): int = x * 2, 5)   # → 10

# Type alias for cleaner code
type IntTransform = proc(x: int): int

proc applyTransform(f: IntTransform, x: int): int =
  f(x)

# Anonymous proc (like lambda)
let double = proc(x: int): int = x * 2
echo double(5)   # → 10

# Arrow syntax (shorter)
let triple = (x: int) => x * 3
echo triple(5)   # → 15
```

### Closures

```nim
proc makeAdder(n: int): proc(x: int): int =
  result = proc(x: int): int = x + n

let add5 = makeAdder(5)
echo add5(3)    # → 8

# Closures capturing mutable state
proc makeCounter(): proc(): int =
  var count = 0
  result = proc(): int =
    inc count
    count

let counter = makeCounter()
echo counter()   # → 1
echo counter()   # → 2
echo counter()   # → 3
```

### Methods (UFCS - Uniform Function Call Syntax)

Nim's most distinctive feature: **any procedure can be called as a
method** if its first argument matches:

```python
# Python - method syntax requires class definition
"hello".upper()
[1,2,3].append(4)
```

```nim
# Nim UFCS - any proc can be called as method
proc double(x: int): int = x * 2
proc shout(s: string): string = s.toUpperAscii() & "!"

echo 5.double()     # same as double(5) → 10
echo "hello".shout() # same as shout("hello") → "HELLO!"

# Method chaining
echo "hello".shout().len    # 6

# This is why Nim looks object-oriented even without classes
var s = "hello world"
echo s.split(" ").len    # → 2
echo s.toUpperAscii().strip()
```

### Iterators

```python
# Python generator
def range_gen(n):
    i = 0
    while i < n:
        yield i
        i += 1
```

```nim
# Nim iterator
iterator countUp(n: int): int =
  var i = 0
  while i < n:
    yield i
    inc i

for x in countUp(5):
  echo x   # 0 1 2 3 4

# Inline iterator (more efficient, cannot be passed as value)
iterator pairs[T](s: seq[T]): tuple[idx: int, val: T] =
  for i in 0..<s.len:
    yield (i, s[i])

for (i, v) in pairs(@["a", "b", "c"]):
  echo i, ": ", v
```

---

## 7. Control Flow

### If Expressions

```nim
# If as statement
if x > 0:
  echo "positive"
elif x < 0:
  echo "negative"
else:
  echo "zero"

# If as expression (must have else)
let label = if x > 0: "positive"
            elif x < 0: "negative"
            else: "zero"

# Inline
echo(if x > 0: "pos" else: "neg")
```

### Case (Switch)

```python
# Python 3.10+ match
match status:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case _:
        print("Other")
```

```nim
# Nim case - exhaustive for enums!
case status
of 200:
  echo "OK"
of 404:
  echo "Not Found"
of 301, 302:         # multiple values
  echo "Redirect"
of 500..599:         # range
  echo "Server Error"
else:
  echo "Other"

# Case as expression
let label = case status
  of 200: "OK"
  of 404: "Not Found"
  else: "Other"

# Case on enum (exhaustive — compiler error if you miss a case)
type Fruit = enum Apple, Banana, Cherry

case fruit
of Apple:  echo "apple"
of Banana: echo "banana"
of Cherry: echo "cherry"
# No else needed (and no else allowed — compiler ensures exhaustion)
```

### Loops

```python
# Python
for i in range(10):
    print(i)

for item in my_list:
    print(item)

i = 0
while i < 10:
    i += 1

for i, item in enumerate(my_list):
    print(i, item)
```

```nim
# Nim

# for with range
for i in 0..<10:    # 0 to 9 (exclusive end)
  echo i

for i in 0..10:     # 0 to 10 (inclusive end)
  echo i

for i in countdown(10, 0):   # 10 down to 0
  echo i

# for over collection
for item in mySeq:
  echo item

# for with index
for i, item in mySeq:
  echo i, ": ", item

# while
var i = 0
while i < 10:
  echo i
  inc i

# Loop control
for i in 0..100:
  if i == 5: continue    # skip
  if i == 10: break      # stop
  echo i

# Nested loop with named break
block outer:
  for i in 0..5:
    for j in 0..5:
      if i + j == 7:
        break outer   # break all the way out
```

### When: Compile-Time If

```nim
# when is evaluated at COMPILE TIME (like #ifdef in C)
when defined(windows):
  echo "Running on Windows"
elif defined(linux):
  echo "Running on Linux"
elif defined(macosx):
  echo "Running on macOS"

# Use for platform-specific code
when sizeof(int) == 8:
  echo "64-bit system"
else:
  echo "32-bit system"

# Custom defines (pass -d:flag to nim compiler)
when defined(debug):
  echo "Debug mode"

# Check Nim version
when NimVersion >= "2.0.0":
  echo "Nim 2.0+"
```

---

## 8. Collections

### Sequences (seq) — The Main Dynamic Array

```python
# Python list
lst = [1, 2, 3, 4, 5]
lst.append(6)
lst.pop()
lst[2]
lst[1:4]
len(lst)
```

```nim
import std/sequtils

# seq literal (@ prefix creates a seq from array literal)
var s: seq[int] = @[1, 2, 3, 4, 5]
var s2 = @[1, 2, 3]    # type inferred as seq[int]

# Access
echo s[0]          # → 1
echo s[^1]         # → 5  (last element, ^ means "from end")
echo s[^2]         # → 4  (second to last)
echo s[1..3]       # → @[2, 3, 4]  (slice, inclusive)
echo s[1..<3]      # → @[2, 3]     (slice, exclusive end)

# Modification
s.add(6)           # append
s.add(@[7, 8])     # append seq
s.insert(99, 2)    # insert at index 2
s.del(0)           # delete by index (replaces with last — fast)
s.delete(0)        # delete by index (shifts — preserves order)
discard s.pop()    # remove and return last

# Info
echo s.len         # length
echo s.high        # last valid index (len - 1)
echo s.low         # first valid index (always 0 for seq)

# Build
var built = newSeq[int](5)       # seq of 5 zeros
var built2 = newSeqWith(5, 0)    # seq of 5 zeros
var built3 = toSeq(1..10)        # → @[1,2,3,4,5,6,7,8,9,10]

# Functional operations (from sequtils)
echo s.mapIt(it * 2)             # → @[2,4,6,8,10]
echo s.filterIt(it > 2)          # → @[3,4,5]
echo s.foldl(a + b)              # → 15 (reduce/fold)
echo s.foldl(a + b, 0)           # → 15 (with initial value)
echo s.anyIt(it > 4)             # → true
echo s.allIt(it > 0)             # → true
echo s.countIt(it mod 2 == 0)    # count evens
echo s.deduplicate()             # remove duplicates
echo s.sorted()                  # sorted copy
echo s.reversed()                # reversed copy
echo s.zip(@["a","b","c"])       # zip two seqs
```

### Arrays: Fixed-Size, Stack-Allocated

```nim
# Array type: [Size, Type]
var arr: array[5, int] = [1, 2, 3, 4, 5]
var arr2 = [1, 2, 3, 4, 5]   # inferred

# Size is part of the type — array[5, int] ≠ array[6, int]
echo arr.len        # → 5
echo arr[0]         # → 1

# Arrays are value types (copied on assignment)
var copy = arr      # independent copy
copy[0] = 99
echo arr[0]         # → 1 (unchanged)

# 2D array
var matrix: array[3, array[3, int]]
matrix[0][0] = 1

# Array with custom index range
var custom: array[1..5, int]   # index from 1 to 5
custom[1] = 10
custom[5] = 50
```

### Tables (Hash Maps)

```python
# Python
d = {"alice": 30, "bob": 25}
d["charlie"] = 35
d.get("dave", 0)
"alice" in d
del d["bob"]
for k, v in d.items():
    print(k, v)
```

```nim
import std/tables

# Create
var t = initTable[string, int]()
var t2 = {"alice": 30, "bob": 25}.toTable   # from array of pairs

# Set/Get
t["charlie"] = 35
echo t["alice"]              # → 30 (raises KeyError if missing)
echo t.getOrDefault("dave", 0)  # → 0 (safe get)

# Query
echo t.hasKey("alice")       # → true
echo "bob" in t              # → true

# Delete
t.del("bob")

# Iterate
for k, v in t:
  echo k, ": ", v

for k in t.keys:
  echo k

for v in t.values:
  echo v

# Info
echo t.len

# OrderedTable: preserves insertion order
import std/tables
var ordered = initOrderedTable[string, int]()

# CountTable: count occurrences
var counts = toCountTable(@["a", "b", "a", "c", "a"])
echo counts["a"]   # → 3
counts.sort()      # sort by count
```

### Sets

```nim
# Built-in set: bit set for small integers/enums (very fast)
var s: set[char]
s.incl('a')
s.incl('b')
s.incl('c')
echo 'a' in s     # → true
s.excl('b')

# Set literals
var digits = {'0'..'9'}        # range of chars
var vowels = {'a','e','i','o','u'}

# Set operations
var evens = {0, 2, 4, 6, 8}
var primes = {2, 3, 5, 7}
echo evens * primes    # intersection: {2}
echo evens + primes    # union: {0,2,3,4,5,6,7,8}
echo evens - primes    # difference: {0,4,6,8}
echo evens <= {0..10}  # subset: true

# HashSet: for arbitrary types
import std/sets
var hs = initHashSet[string]()
hs.incl("hello")
hs.incl("world")
echo "hello" in hs    # → true

# From seq
var hs2 = @["a","b","c","a"].toHashSet   # deduplicates!
```

---

## 9. Strings

### Basic String Operations

```python
s = "Hello, World!"
len(s)
s.upper()
s.lower()
s[7:12]
s.split(", ")
", ".join(["a","b"])
s.strip()
s.startswith("Hello")
s.replace("o","0")
"World" in s
```

```nim
import std/strutils

var s = "Hello, World!"

echo s.len                         # → 13
echo s.toUpperAscii()              # → "HELLO, WORLD!"
echo s.toLowerAscii()              # → "hello, world!"
echo s[7..11]                      # → "World"
echo s.split(", ")                 # → @["Hello", "World!"]
echo @["a","b"].join(", ")         # → "a, b"
echo s.strip()                     # strip whitespace
echo s.startsWith("Hello")         # → true
echo s.endsWith("!")               # → true
echo s.replace("o", "0")          # → "Hell0, W0rld!"
echo "World" in s                  # → true
echo s.contains("World")           # → true
echo s.count("l")                  # → 3
echo s.find("World")               # → 7 (index or -1)
echo s.toLowerAscii().capitalizeAscii() # → "Hello, world!"

# Trim specific characters
echo "  hello  ".strip()
echo "xxhelloxx".strip({'x'})     # strip x chars

# Split by whitespace
echo "a  b  c".splitWhitespace()  # → @["a","b","c"]
```

### String Formatting

```python
# Python
f"Hello, {name}! You are {age} years old."
"Hello, {}!".format(name)
"%s is %d" % (name, age)
```

```nim
import std/strformat

# & prefix for format strings (like Python f-strings)
let name = "Alice"
let age = 30
echo &"Hello, {name}! You are {age} years old."

# Format with expressions
echo &"2 + 2 = {2 + 2}"
echo &"Pi = {3.14159:.2f}"    # format specifier
echo &"Hex: {255:#x}"         # → "Hex: 0xff"

# fmt macro (same as &)
import std/strformat
echo fmt"Hello, {name}!"

# String concatenation
echo "Hello, " & name & "!"   # & is concat operator

# $ converts anything to string
echo $42           # → "42"
echo $3.14         # → "3.14"
echo $true         # → "true"
echo $@[1,2,3]     # → "@[1, 2, 3]"

# sprintf-style (from strutils)
echo "$1 is $2 years old" % [name, $age]
```

### Multi-line Strings

```python
# Python
s = """
line one
line two
line three
"""
```

```nim
# Nim raw string (no escape processing)
var s = """
line one
line two
line three
"""

# Triple-quoted strings strip leading whitespace if indented
var s2 = """
  line one
  line two
  """.dedent()   # from strutils

# Raw string: r"" (backslashes are literal)
var path = r"C:\Users\Alice\Documents"

# Raw string with custom delimiter
var re = r"(\d+)\s+(\w+)"
```

### String Manipulation

```nim
import std/strutils

# Split and join
let words = "one two three".split(" ")
echo words.join("-")   # → "one-two-three"

# Parse numbers from strings
echo "42".parseInt()       # → 42
echo "3.14".parseFloat()   # → 3.14

# Check content
echo "42".isDigit()        # → false (checks each char)
echo "hello".isAlpha()     # → false (checks each char)
echo "abc123".allCharsInSet({'a'..'z','0'..'9'})  # → true

# Pad and align
echo "hi".alignLeft(10)    # → "hi        "
echo "hi".align(10)        # → "        hi"
echo "42".intToStr.align(5,'0')  # → "00042"

# Repeat
echo "ab".repeat(3)        # → "ababab"

# Indent
echo "hello\nworld".indent(2)  # adds 2 spaces to each line
```

### Regular Expressions

```nim
import std/re

# Compile regex
let pattern = re"\d+"

# Test
echo "hello42world".contains(pattern)  # → true

# Find
echo "hello42world".find(pattern)      # → 5 (position)

# Extract matches
let m = "hello 42 world 99".findAll(re"\d+")
echo m   # → @["42", "99"]

# Replace
echo "hello42world".replace(re"\d+", "NUM")  # → "helloNUMworld"

# Capture groups
var matches: array[3, string]
if "2024-01-15".match(re"(\d{4})-(\d{2})-(\d{2})", matches):
  echo matches[0]   # → "2024"
  echo matches[1]   # → "01"
  echo matches[2]   # → "15"
```

---

## 10. Object-Oriented Programming

### Object Types

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def distance_to_origin(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

    def __str__(self) -> str:
        return f"Point({self.x}, {self.y})"
```

```nim
import std/math

type
  Point = object
    x, y: float

# Methods via UFCS (procs with first arg of the type)
proc distanceToOrigin(p: Point): float =
  sqrt(p.x * p.x + p.y * p.y)

# $ operator for string conversion (like __str__)
proc `$`(p: Point): string =
  "Point(" & $p.x & ", " & $p.y & ")"

# Construction
let p = Point(x: 3.0, y: 4.0)
echo p.distanceToOrigin()   # → 5.0
echo p                      # → "Point(3.0, 4.0)"
echo $p                     # → "Point(3.0, 4.0)"
```

### Reference vs Value Types

```nim
# Object (value type) — stored on stack, copied on assignment
type
  Vector = object
    x, y: float

var v1 = Vector(x: 1.0, y: 2.0)
var v2 = v1        # v2 is a COPY
v2.x = 99.0
echo v1.x          # → 1.0 (unchanged)

# ref object (reference type) — stored on heap, like Python objects
type
  Node = ref object
    value: int
    next: Node     # can reference itself (recursive type)

var n1 = Node(value: 42)
var n2 = n1        # n2 points to the SAME object
n2.value = 99
echo n1.value      # → 99 (changed!)

# Check for nil
if n1 == nil:
  echo "empty"
```

### Inheritance

```python
class Animal:
    def __init__(self, name: str):
        self.name = name

    def speak(self) -> str:
        return "..."

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says Woof!"
```

```nim
type
  Animal = ref object of RootObj   # must inherit from RootObj for polymorphism
    name: string

  Dog = ref object of Animal
    breed: string

  Cat = ref object of Animal
    indoor: bool

# Method dispatch
proc speak(a: Animal): string =
  "..."

proc speak(d: Dog): string =
  d.name & " says Woof!"

proc speak(c: Cat): string =
  c.name & " says Meow!"

# Polymorphism (method overriding)
method describe(a: Animal): string {.base.} =
  "I am an animal named " & a.name

method describe(d: Dog): string =
  procCall describe(Animal(d)) & ", a dog of breed " & d.breed

# Use 'method' (not 'proc') for virtual dispatch
var animals: seq[Animal] = @[
  Dog(name: "Rex", breed: "German Shepherd"),
  Cat(name: "Whiskers", indoor: true),
]

for a in animals:
  echo a.describe()    # virtual dispatch based on runtime type

# Type checking
if animals[0] of Dog:
  let dog = Dog(animals[0])   # downcast
  echo dog.breed
```

### Operator Overloading

```python
class Vector:
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
```

```nim
type Vector2D = object
  x, y: float

# Operator overloading using backtick syntax
proc `+`(a, b: Vector2D): Vector2D =
  Vector2D(x: a.x + b.x, y: a.y + b.y)

proc `*`(v: Vector2D, s: float): Vector2D =
  Vector2D(x: v.x * s, y: v.y * s)

proc `==`(a, b: Vector2D): bool =
  a.x == b.x and a.y == b.y

proc `[]`(v: Vector2D, i: int): float =
  case i
  of 0: v.x
  of 1: v.y
  else: raise newException(IndexDefect, "Index out of range")

let a = Vector2D(x: 1.0, y: 2.0)
let b = Vector2D(x: 3.0, y: 4.0)
echo a + b     # → Vector2D(x: 4.0, y: 6.0)
echo a * 2.0   # → Vector2D(x: 2.0, y: 4.0)
echo a[0]      # → 1.0
```

---

## 11. Generics

Nim has powerful generics that are resolved at compile time with zero
runtime overhead:

```python
# Python generics are just type hints (not enforced at runtime)
from typing import TypeVar, Generic, List
T = TypeVar('T')

def first(lst: List[T]) -> T:
    return lst[0]
```

```nim
# Nim generics are real and checked at compile time
proc first[T](lst: seq[T]): T =
  lst[0]

echo first(@[1, 2, 3])      # → 1     (T = int)
echo first(@["a", "b"])     # → "a"   (T = string)

# Generic with constraint
proc maxOf[T: Ordinal](a, b: T): T =
  if a > b: a else: b

echo maxOf(3, 5)        # → 5
echo maxOf('a', 'z')    # → 'z'

# Generic type
type
  Stack[T] = object
    data: seq[T]

proc push[T](s: var Stack[T], val: T) =
  s.data.add(val)

proc pop[T](s: var Stack[T]): T =
  result = s.data[^1]
  s.data.setLen(s.data.len - 1)

proc isEmpty[T](s: Stack[T]): bool =
  s.data.len == 0

var intStack: Stack[int]
intStack.push(1)
intStack.push(2)
echo intStack.pop()   # → 2

# Concept: type constraint (like Rust traits or Haskell typeclasses)
type Addable = concept x, y
  x + y is type(x)

proc addTwo[T: Addable](a, b: T): T =
  a + b

echo addTwo(1, 2)         # → 3
echo addTwo(1.0, 2.0)     # → 3.0
echo addTwo("a", "b")     # → "ab"
```

---

## 12. Templates and Macros

### Templates: Type-Safe Text Substitution

Templates are like inline functions but operate at the AST level. They
have zero runtime overhead:

```nim
# Template: compile-time code generation
template twice(action: untyped) =
  action
  action

twice:
  echo "hello"
# Output:
# hello
# hello

# Template with parameters
template withLogging(name: string, body: untyped) =
  echo "Starting: " & name
  body
  echo "Finished: " & name

withLogging("database operation"):
  connectToDb()
  runQuery()
  disconnect()

# Template as expression
template max(a, b: untyped): untyped =
  (if a > b: a else: b)

echo max(3, 5)    # → 5
echo max("a","z") # → "z"  (works for any comparable type)
```

### Macros: Full AST Manipulation

Macros transform Nim code at compile time, giving you full control over
code generation:

```nim
import std/macros

# Simple macro that prints its argument's AST
macro inspect(x: untyped): untyped =
  echo x.treeRepr    # print AST during compilation
  x                  # return original expression

inspect(1 + 2 * 3)

# Macro that generates code
macro repeatN(n: static int, body: untyped): untyped =
  result = newStmtList()
  for i in 0..<n:
    result.add(body)

repeatN(3):
  echo "hello"
# Generates three echo statements at compile time

# Macro for domain-specific language
macro html(body: untyped): string =
  ## Tiny HTML DSL
  # ... AST transformation ...
  discard

# Check macro expansion
expandMacros:
  repeatN(3): echo "test"
```

---

## 13. Error Handling

### Exceptions

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, RuntimeError):
    print("Type or runtime error")
finally:
    cleanup()
```

```nim
try:
  let result = riskyOperation()
except ValueError as e:
  echo "Value error: " & e.msg
except TypeError, IOError:
  echo "Type or IO error"
except CatchableError as e:
  echo "Any catchable error: " & e.msg
finally:
  cleanup()

# Raise exceptions
raise newException(ValueError, "invalid input")
raise newException(IOError, "file not found")

# Custom exceptions
type
  DatabaseError = object of CatchableError
  ConnectionError = object of DatabaseError

raise newException(ConnectionError, "cannot connect to DB")
```

### Result Type Pattern (No Exceptions)

Modern Nim style often avoids exceptions for expected failures:

```nim
# Option[T] - value or nothing
import std/options

proc findUser(id: int): Option[string] =
  if id == 1:
    some("Alice")
  else:
    none(string)

let user = findUser(1)
if user.isSome:
  echo user.get()      # → "Alice"

echo user.get("unknown")   # → "Alice" (or default)

# Chaining with options
let upper = findUser(1).map(proc(s: string): string = s.toUpperAscii())
echo upper   # → Some("ALICE")

# Result[T, E] from the results library (like Rust's Result)
# nimble install results
import results

proc divide(a, b: float): Result[float, string] =
  if b == 0.0:
    err("Division by zero")
  else:
    ok(a / b)

let r = divide(10.0, 2.0)
if r.isOk:
  echo r.value   # → 5.0
else:
  echo r.error

# Pattern with ? operator (like Rust's ?)
proc computation(): Result[int, string] =
  let x = ? divide(10.0, 2.0)   # return Err early if failed
  ok(int(x * 2))
```

### Defect vs CatchableError

```nim
# Nim distinguishes two kinds of exceptions:

# CatchableError: expected, recoverable errors (catch these)
type
  MyError = object of CatchableError

# Defect: programming errors, bugs (don't catch these)
# IndexDefect, NilAccessDefect, OverflowDefect, etc.
# These indicate bugs in your code

# You can turn off runtime checks (and thus Defects) with -d:danger
# But CatchableErrors are always active
```

---

## 14. Concurrency

### Threads

```python
import threading

def worker(n):
    print(f"Worker {n}")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```

```nim
import std/threadpool

# spawn: run in thread pool (like Python's concurrent.futures)
proc worker(n: int) {.thread.} =
  echo "Worker " & $n

# FlowVar: like Python's Future
var tasks: seq[FlowVar[void]]
for i in 0..4:
  tasks.add(spawn worker(i))

# Wait for all
sync()

# With return values
proc computeSquare(n: int): int {.thread.} =
  n * n

let f1 = spawn computeSquare(5)
let f2 = spawn computeSquare(10)
echo ^f1    # ^ dereferences FlowVar (blocks if not done)
echo ^f2
```

### Channels

```nim
import std/channels

# Type-safe channels between threads
var ch: Channel[int]
ch.open()

# Send
ch.send(42)

# Receive (blocking)
let val = ch.recv()

# Non-blocking receive
var data: int
if ch.tryRecv(data):
  echo data

# Close
ch.close()

# Example: producer/consumer
var channel: Channel[string]
channel.open(10)   # buffer size 10

proc producer() {.thread.} =
  for i in 0..9:
    channel.send("item " & $i)
  channel.send("")   # sentinel

proc consumer() {.thread.} =
  while true:
    let msg = channel.recv()
    if msg == "": break
    echo "Got: " & msg

var t1, t2: Thread[void]
createThread(t1, producer)
createThread(t2, consumer)
joinThread(t1)
joinThread(t2)
```

### Async/Await

```python
import asyncio

async def fetch(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    results = await asyncio.gather(fetch(url1), fetch(url2))
```

```nim
import std/asyncdispatch
import std/asynchttpserver

# Async proc
proc fetchUrl(url: string): Future[string] {.async.} =
  # ... http request ...
  result = "response body"

# Await
proc main() {.async.} =
  let r1 = fetchUrl("http://example.com/1")
  let r2 = fetchUrl("http://example.com/2")

  # Run concurrently
  let (result1, result2) = await (r1, r2)
  echo result1
  echo result2

# Run the event loop
waitFor main()

# Async HTTP server
proc handleRequest(req: Request) {.async.} =
  await req.respond(Http200, "Hello, World!")

var server = newAsyncHttpServer()
waitFor server.serve(Port(8080), handleRequest)
```

---

## 15. Memory Management

This is where Nim gives you unique flexibility. Python hides memory entirely;
Nim lets you choose your memory strategy:

### Default: ARC/ORC (Automatic Reference Counting)

```nim
# Default in modern Nim: ARC (Automatic Reference Counting)
# or ORC (for cyclic structures)
# Compile with: nim c --gc:arc myfile.nim
#               nim c --gc:orc myfile.nim (handles cycles)

# No GIL! True parallelism possible.
# Deterministic destruction (destructors run immediately)
# No stop-the-world GC pauses
```

### Manual Memory Management

```nim
import std/system

# Allocate on heap manually
let p = alloc0(sizeof(int))   # alloc and zero
cast[ptr int](p)[] = 42
echo cast[ptr int](p)[]       # → 42
dealloc(p)                    # must free manually

# Typed allocation
let ip = create(int)          # allocate one int
ip[] = 42
echo ip[]                     # → 42
dealloc(ip)

# Stack allocation (automatic, no heap)
var local: int = 42           # on stack, freed when scope exits
```

### Custom Allocators

```nim
# Memory pool for performance-critical code
import std/memfiles

# mmap a file into memory
let mf = memfiles.open("bigfile.dat")
# work with mf.mem pointer directly
mf.close()
```

### Destructors and Move Semantics

```nim
type
  Resource = object
    handle: int

proc `=destroy`(r: var Resource) =
  ## Called automatically when Resource goes out of scope
  if r.handle != 0:
    closeHandle(r.handle)
    echo "Resource " & $r.handle & " destroyed"

proc `=copy`(dst: var Resource, src: Resource) =
  ## Called on copy — you can make this a compile error
  {.error: "Resource cannot be copied".}

proc `=sink`(dst: var Resource, src: Resource) =
  ## Move semantics — transfer ownership
  dst.handle = src.handle

block:
  let r = Resource(handle: 42)
  # use r...
# r goes out of scope here, destructor called automatically
```

---

## 16. C Interop

One of Nim's greatest strengths — calling C libraries is trivial:

```python
# Python - requires ctypes or cffi, painful
import ctypes
lib = ctypes.CDLL("./mylib.so")
lib.myfunction.argtypes = [ctypes.c_int]
lib.myfunction.restype = ctypes.c_int
```

```nim
# Nim - just declare the binding
proc myFunction(x: cint): cint {.importc: "myFunction", header: "mylib.h".}

# Or link directly
proc strlen(s: cstring): cint {.importc, header: "<string.h>".}

# Call it
let s: cstring = "hello"
echo strlen(s)   # → 5

# Structs
type
  CPoint {.importc: "Point", header: "mylib.h".} = object
    x, y: cdouble

# Enums from C
type
  CDirection {.importc: "Direction", header: "mylib.h".} = enum
    North = 0, South, East, West

# Pragmas for binding
proc malloc(size: csize_t): pointer {.importc, header: "<stdlib.h>".}
proc free(p: pointer) {.importc, header: "<stdlib.h>".}

# Link a library
{.link: "mylib.a".}
# or
{.passl: "-lmylib".}

# Inline C code
{.emit: """
int myInlineCFunc(int x) {
    return x * 2;
}
""".}
```

---

## 17. I/O and Files

### Console I/O

```python
print("Hello")
name = input("Enter name: ")
print(f"Hello, {name}", end="")
```

```nim
echo "Hello"                       # print with newline
stdout.write("Hello")              # print without newline
let name = readLine(stdin)         # read a line
echo "Hello, " & name

# Formatted output
import std/strformat
echo &"Name: {name}, Age: {30}"

# stderr
stderr.writeLine("Error message")
```

### File I/O

```python
# Read entire file
with open("file.txt") as f:
    content = f.read()

# Read lines
with open("file.txt") as f:
    for line in f:
        print(line.strip())

# Write
with open("output.txt", "w") as f:
    f.write("hello\n")
```

```nim
import std/[os, io, streams]

# Read entire file
let content = readFile("file.txt")

# Write entire file
writeFile("output.txt", "hello\n")

# Read lines
for line in lines("file.txt"):
  echo line

# File streams (for large files or more control)
let f = open("file.txt", fmRead)
defer: f.close()    # like Python's 'with' — closes on scope exit

var line: string
while f.readLine(line):
  echo line

# Write with stream
let out = open("output.txt", fmWrite)
defer: out.close()
out.writeLine("Hello")
out.write("no newline")

# Append
let app = open("log.txt", fmAppend)
defer: app.close()
app.writeLine("new log entry")
```

### Path Operations

```python
import os
from pathlib import Path

path = Path("dir/subdir/file.txt")
path.parent       # Path("dir/subdir")
path.name         # "file.txt"
path.stem         # "file"
path.suffix       # ".txt"
os.path.exists(path)
os.makedirs(path.parent, exist_ok=True)
```

```nim
import std/os

let path = "dir/subdir/file.txt"

echo path.parentDir()    # → "dir/subdir"
echo path.lastPathPart() # → "file.txt"
echo path.splitFile().name  # → "file"
echo path.splitFile().ext   # → ".txt"
echo path.extractFilename() # → "file.txt"

echo fileExists(path)        # → true/false
echo dirExists("dir")        # → true/false

createDir("dir/subdir")         # like os.makedirs
createDir("dir/subdir")         # doesn't error if exists
removeFile("file.txt")
removeDir("dir")
copyFile("src.txt", "dst.txt")
moveFile("old.txt", "new.txt")

# Path joining
let full = "dir" / "subdir" / "file.txt"  # / is path join operator
echo full    # → "dir/subdir/file.txt"

# Current directory
echo getCurrentDir()
setCurrentDir("/tmp")

# List directory
for f in walkDir("."):
  echo f.path, " (", f.kind, ")"

# Recursive walk
for f in walkDirRec("src"):
  echo f
```

---

## 18. Modules and Packages

### Creating and Using Modules

```python
# mymodule.py
def helper():
    return 42

__all__ = ['helper']
```

```nim
# mymodule.nim
proc helper*(): int =   # * makes it exported (public)
  42

type
  PublicType* = object  # exported
    value*: int         # field exported

  PrivateType = object  # NOT exported
    secret: string
```

### Importing

```python
import mymodule
from mymodule import helper
from mymodule import helper as h
import mymodule as mm
from mymodule import *
```

```nim
import mymodule              # import all exported names
import mymodule as mm        # with alias
from mymodule import helper  # specific import
from mymodule import nil     # import but require full qualification
                             # must write mymodule.helper()

# Multiple imports
import std/[os, strutils, sequtils, tables]

# Selective import with except
import mymodule except privateHelper

# Include (textual inclusion, like C's #include)
include myfile    # pastes the file content (rarely used)
```

### Standard Library Organization

```nim
# std/ prefix for standard library
import std/strutils    # string utilities
import std/sequtils    # sequence utilities
import std/tables      # hash tables
import std/sets        # hash sets
import std/os          # operating system
import std/math        # math functions
import std/json        # JSON
import std/re          # regular expressions
import std/httpclient  # HTTP client
import std/asyncdispatch # async
import std/options     # Option type
import std/algorithm   # sorting, etc.
import std/parseutils  # parsing utilities
import std/strformat   # string formatting (&"")
import std/times       # date/time
import std/monotimes   # monotonic time
import std/hashes      # hashing
import std/deques      # deque data structure
import std/heapqueue   # priority queue
import std/sugar       # syntactic sugar (=>, capture, etc.)
import std/enumerate   # enumerate iterator
```

---

## 19. Testing

### Built-in unittest

```python
import unittest

class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)

    def test_divide_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            divide(10, 0)
```

```nim
import std/unittest

suite "Math operations":
  test "addition":
    check add(2, 3) == 5
    check add(-1, 1) == 0

  test "division":
    check 10 div 2 == 5

  test "divide by zero raises":
    expect(DivByZeroDefect):
      discard 10 div 0

  setup:
    # Runs before each test
    echo "Setting up..."

  teardown:
    # Runs after each test
    echo "Tearing down..."

# Run with: nim r tests/mytest.nim
# Or: nimble test

# Assertions
check someValue == expected
check someValue != unexpected
check someCondition
check someString.contains("substring")
require someValue != nil   # stops test on failure (unlike check)
```

### Testing Patterns

```nim
import std/unittest

# Test with message
check(result == 42, "Expected 42 but got " & $result)

# Multiple assertions in one check using and
check(x > 0 and x < 100)

# Skip a test
test "not implemented yet":
  skip()

# Test that something compiles (or doesn't)
static:
  doAssert sizeof(int32) == 4

# Benchmark within tests (rough)
import std/monotimes
test "performance":
  let start = getMonoTime()
  myExpensiveOperation()
  let elapsed = getMonoTime() - start
  check elapsed.inMilliseconds < 100
```

---

## 20. Build System

### nimble Tasks

```nim
# In your .nimble file

task test, "Run tests":
  exec "nim r tests/main.nim"

task release, "Build release binary":
  exec "nim c -d:release -o:myapp src/main.nim"

task clean, "Remove build artifacts":
  rmFile "myapp"
  rmDir "nimcache"

task bench, "Run benchmarks":
  exec "nim c -d:release -r benchmarks/bench.nim"
```

```bash
nimble test      # run test task
nimble release   # run release task
nimble bench     # run bench task
```

### Compile-Time Configuration

```nim
# Pass -d: flags to control compilation
nim c -d:release myapp.nim       # optimized
nim c -d:debug myapp.nim         # with debug info
nim c -d:ssl myapp.nim           # enable SSL
nim c -d:danger myapp.nim        # maximum speed (remove all checks)
nim c -d:myCustomFlag myapp.nim  # custom flag

# In code, check flags:
when defined(ssl):
  import openssl
  # SSL-specific code

when not defined(release):
  echo "Debug build"
```

---

## 21. Common Patterns and Idioms

### Functional Style with sequtils

```python
# Python
numbers = list(range(1, 11))
result = sorted(filter(lambda x: x % 2 == 0,
                       map(lambda x: x ** 2, numbers)))
```

```nim
import std/[sequtils, algorithm]

let numbers = toSeq(1..10)

let result = numbers
  .mapIt(it * it)
  .filterIt(it mod 2 == 0)
  .sorted()

echo result   # → @[4, 16, 36, 64, 100]

# More sequtils
echo numbers.foldl(a + b)           # sum → 55
echo numbers.any(proc(x:int):bool = x > 9)  # → true
echo numbers.all(proc(x:int):bool = x > 0)  # → true
echo numbers.zip(@["a","b","c"])     # zip
echo numbers[0..4].product()         # product of first 5
```

### String Processing Pipeline

```nim
import std/[strutils, sequtils]

let csv = "alice,30,nyc\nbob,25,la\ncharlie,35,chicago"

let records = csv
  .splitLines()
  .mapIt(it.split(","))
  .filterIt(it.len == 3)
  .mapIt((name: it[0], age: it[1].parseInt(), city: it[2]))

for r in records:
  echo &"{r.name} is {r.age} from {r.city}"
```

### Memoization

```nim
import std/tables

proc memoize[T, R](f: proc(x: T): R): proc(x: T): R =
  var cache = initTable[T, R]()
  result = proc(x: T): R =
    if x notin cache:
      cache[x] = f(x)
    cache[x]

var fibonacci: proc(n: int): int
fibonacci = memoize(proc(n: int): int =
  if n <= 1: n
  else: fibonacci(n-1) + fibonacci(n-2))

echo fibonacci(40)   # fast!
```

### Type State Pattern

```nim
# Encode state in the type system to prevent invalid operations
type
  Unopened = distinct string
  Opened   = distinct string
  Closed   = distinct string

proc open(f: Unopened): Opened =
  Opened(string(f))

proc read(f: Opened): string =
  # can only read Opened files
  readFile(string(f))

proc close(f: Opened): Closed =
  Closed(string(f))

let path = Unopened("myfile.txt")
let opened = path.open()
let content = opened.read()   # fine
let closed = opened.close()
# closed.read()   # COMPILE ERROR: read requires Opened, not Closed
```

### Builder Pattern

```nim
type
  QueryBuilder = object
    table: string
    conditions: seq[string]
    limit: int
    orderBy: string

proc newQuery(table: string): QueryBuilder =
  QueryBuilder(table: table, limit: -1)

proc where(q: QueryBuilder, cond: string): QueryBuilder =
  result = q
  result.conditions.add(cond)

proc orderBy(q: QueryBuilder, col: string): QueryBuilder =
  result = q
  result.orderBy = col

proc limit(q: QueryBuilder, n: int): QueryBuilder =
  result = q
  result.limit = n

proc build(q: QueryBuilder): string =
  result = "SELECT * FROM " & q.table
  if q.conditions.len > 0:
    result &= " WHERE " & q.conditions.join(" AND ")
  if q.orderBy != "":
    result &= " ORDER BY " & q.orderBy
  if q.limit > 0:
    result &= " LIMIT " & $q.limit

let query = newQuery("users")
  .where("age > 18")
  .where("active = true")
  .orderBy("name")
  .limit(10)
  .build()

echo query
# → SELECT * FROM users WHERE age > 18 AND active = true ORDER BY name LIMIT 10
```

---

## 22. Metaprogramming

### Compile-Time Evaluation (static)

```nim
# Run code at compile time
const fib10 = block:
  var a, b = 1
  for i in 0..<8:
    (a, b) = (b, a + b)
  b

echo fib10   # computed at compile time, no runtime cost

# static: keyword forces compile-time evaluation
proc isEven(n: static int): bool =
  n mod 2 == 0

const check = isEven(4)   # true, computed at compile time
```

### Type Information at Compile Time

```nim
import std/typetraits

# Get type information
echo name(int)          # → "int"
echo name(seq[string])  # → "seq[string]"

# Check type properties
echo isOrdinal(int)     # → true
echo isOrdinal(float)   # → false
echo genericParams(seq[int])  # → (int)

# sizeof at compile time
static:
  echo sizeof(int)      # → 8 (on 64-bit)
  echo sizeof(float32)  # → 4
  echo sizeof(bool)     # → 1

# Generic type checking
proc printTypeInfo[T](x: T) =
  echo "Type: " & name(T)
  echo "Size: " & $sizeof(T)
  when T is SomeNumber:
    echo "Is a number"
  when T is string:
    echo "Is a string"
```

### Macro for DSL Creation

```nim
import std/macros

# Create a simple test DSL
macro describe(name: string, body: untyped): untyped =
  result = newStmtList()
  result.add(newCall("echo", newStrLitNode("Suite: " & name.strVal)))
  result.add(body)

macro it(name: string, body: untyped): untyped =
  result = newStmtList()
  let testName = "  Test: " & name.strVal
  result.add(newCall("echo", newStrLitNode(testName)))
  result.add(body)

# Use the DSL
describe "my module":
  it "does something":
    assert 1 + 1 == 2

  it "handles edge cases":
    assert @[].len == 0
```

---

## 23. Debugging and Profiling

### Debug Output

```nim
# echo: basic print
echo "value = ", x

# debugEcho: only in debug builds
debugEcho "Debug: " & $x

# repr: detailed string representation
echo repr(myObject)   # like Python's repr()

# Custom $ for your types
proc `$`(p: Point): string =
  "Point(" & $p.x & ", " & $p.y & ")"

# stackTrace: print current stack trace
writeStackTrace()

# Assertions
assert x > 0, "x must be positive"
doAssert x > 0, "x must be positive"  # survives -d:release

# dump: print name = value (from sugar module)
import std/sugar
dump x           # prints: x = 42
dump x + y       # prints: x + y = 77
```

### Compile-Time Debugging

```nim
# Print during compilation
static: echo "This prints at compile time"

# Print AST
import std/macros
macro debugAST(x: untyped): untyped =
  echo x.treeRepr   # print AST to stdout during compile
  x

debugAST(1 + 2 * 3)

# expandMacros: see what a macro generates
expandMacros:
  myMacro(someArgs)
```

### Profiling

```bash
# Compile with profiling
nim c --profiler:on --stackTrace:on myapp.nim

# Run and get profile data
./myapp

# View profile
nim c --profiler:on --stackTrace:on -d:memProfiler myapp.nim
```

```nim
# Manual timing
import std/monotimes

let start = getMonoTime()
myOperation()
let elapsed = getMonoTime() - start
echo "Elapsed: " & $elapsed.inMilliseconds & "ms"

# Or use times
import std/times
let t0 = cpuTime()
myOperation()
echo "CPU time: " & $(cpuTime() - t0)
```

---

## 24. Style and Conventions

### Naming Conventions

```nim
# Types: PascalCase
type
  UserAccount = object
  HttpClient = ref object
  DatabaseError = object of CatchableError

# Procedures and variables: camelCase
proc getUserById(id: int): User = ...
let maxRetries = 3
var currentUser: User

# Constants: either camelCase or UPPER_CASE (both are acceptable)
const maxConnections = 100
const MAX_CONNECTIONS = 100   # also seen

# Exported symbols have *
proc publicProc*() = ...
type PublicType* = object
var exportedVar*: int

# Private symbols have no *
proc privateHelper() = ...

# Discard marker for intentionally unused
discard someProc()

# Generic type parameters: T, U, K, V, or descriptive
proc foo[T: SomeNumber](x: T): T = ...
proc lookup[K, V](t: Table[K,V], key: K): Option[V] = ...
```

### Code Style

```nim
# 2-space indentation (standard)

# Procedures: space before first param if long
proc myLongProcedureName(
    firstParam: string,
    secondParam: int,
    thirdParam: bool = false): string =
  discard

# Opening brace on same line (for object types)
type
  MyObject = object
    field1: int
    field2: string

# Import grouping: stdlib first, then third-party, then local
import std/[os, strutils, tables]
import someThirdPartyLib
import myLocalModule

# Prefer named tuples over positional
proc getSize(): tuple[width, height: int] =
  (width: 800, height: 600)
# Better than: proc getSize(): tuple[int, int]

# Use result variable for clarity in complex procs
proc buildReport(data: seq[int]): string =
  result = "Report:\n"
  for item in data:
    result &= "  - " & $item & "\n"
```

### When to Use What

```
let:         value won't change (prefer this by default)
var:         value needs to change
const:       compile-time constant
static let:  computed at compile time, stored as constant

proc:        regular procedure (most things)
func:        pure function (no side effects, no I/O)
method:      virtual dispatch (inheritance hierarchies)
template:    compile-time code reuse, zero overhead
macro:       AST transformation, DSLs
iterator:    produce sequences of values

object:      value type (stack, copied)
ref object:  reference type (heap, shared)

seq[T]:      dynamic array (most common collection)
array[N,T]:  fixed-size array (stack, for performance)
Table[K,V]:  hash map (when you need arbitrary keys)
set[T]:      bit set (for enums and small ints, very fast)
HashSet[T]:  hash set (for arbitrary types)

{.importc.}: bind a C function
{.emit.}:    inject raw C code
{.push.}/{.pop.}: apply pragmas to a block
{.thread.}:  mark proc as thread-safe entry point
{.async.}:   async procedure
{.inline.}:  hint to inline the procedure
```

---

## Quick Reference Card

### Python → Nim Cheat Sheet

| Python                 | Nim                           |
| ---------------------- | ----------------------------- |
| `x = 5`                | `var x = 5` or `let x = 5`    |
| `x: int = 5`           | `var x: int = 5`              |
| `def f(x): return x*2` | `proc f(x: int): int = x * 2` |
| `lambda x: x*2`        | `proc(x: int): int = x * 2`   |
| `lambda x: x*2`        | `(x: int) => x * 2` (sugar)   |
| `f(x)`                 | `f(x)` or `x.f()` (UFCS)      |
| `f(*args)`             | `f(args)` (varargs)           |
| `print(x)`             | `echo x`                      |
| `f"Hello {name}"`      | `&"Hello {name}"`             |
| `str(x)`               | `$x`                          |
| `len(x)`               | `x.len`                       |
| `range(n)`             | `0..<n`                       |
| `range(a,b)`           | `a..<b`                       |
| `[1,2,3]`              | `@[1,2,3]` (seq)              |
| `[1,2,3]`              | `[1,2,3]` (array)             |
| `lst[0]`               | `lst[0]`                      |
| `lst[-1]`              | `lst[^1]`                     |
| `lst[1:4]`             | `lst[1..3]`                   |
| `lst.append(x)`        | `lst.add(x)`                  |
| `{"a":1}`              | `{"a":1}.toTable`             |
| `d["k"]`               | `d["k"]`                      |
| `d.get("k",v)`         | `d.getOrDefault("k",v)`       |
| `"k" in d`             | `d.hasKey("k")`               |
| `{1,2,3}`              | `{1,2,3}` (bit set)           |
| `if a: b else: c`      | `if a: b else: c`             |
| `a and b`              | `a and b`                     |
| `a or b`               | `a or b`                      |
| `not a`                | `not a`                       |
| `a == b`               | `a == b`                      |
| `a is b`               | `a == b` (for values)         |
| `True/False`           | `true/false`                  |
| `None`                 | `nil`                         |
| `x ** 2`               | `x ^ 2` or `x * x`            |
| `10 // 3`              | `10 div 3`                    |
| `10 % 3`               | `10 mod 3`                    |
| `try/except`           | `try/except`                  |
| `raise Exception(m)`   | `raise newException(E, m)`    |
| `import mod`           | `import mod`                  |
| `from mod import f`    | `from mod import f`           |
| `# comment`            | `# comment`                   |
| `"""docstring"""`      | `## doc comment`              |
| `@dataclass`           | `type X = object`             |
| `isinstance(x,T)`      | `x of T` or `x is T`          |
| `type(x)`              | `typeof(x)`                   |
| `hasattr(x,"f")`       | `compiles(x.f)`               |
| `threading.Thread`     | `spawn` (threadpool)          |
| `async def`            | `proc ... {.async.}`          |
| `await`                | `await`                       |

| Python                        | Nim                              |
| ----------------------------- | -------------------------------- |
| `x = 5`                       | `var x = 5` (mutable)            |
| `x = 5`                       | `let x = 5` (immutable)          |
| `X = 5` (constant convention) | `const X = 5` (compile-time)     |
| `x: int = 5`                  | `var x: int = 5`                 |
| `x: Final[int] = 5`           | `let x: int = 5`                 |
| `True` / `False`              | `true` / `false`                 |
| `None`                        | `nil`                            |
| `type(x)`                     | `typeof(x)`                      |
| `isinstance(x, T)`            | `x of T` (ref types) or `x is T` |
| `hasattr(x, "f")`             | `compiles(x.f)`                  |
| `id(x)`                       | `cast[int](x.addr)`              |
| `sys.maxsize`                 | `high(int)`                      |
| `float('inf')`                | `Inf`                            |
| `float('nan')`                | `NaN`                            |
| `int` (arbitrary precision)   | `Int` (from std/bigints)         |

#### Arithmetic and Comparison

| Python                | Nim                                     |
| --------------------- | --------------------------------------- |
| `a + b`               | `a + b`                                 |
| `a - b`               | `a - b`                                 |
| `a * b`               | `a * b`                                 |
| `a / b` (float div)   | `a / b` (float)                         |
| `a // b` (int div)    | `a div b`                               |
| `a % b`               | `a mod b`                               |
| `a ** b`              | `a ^ b` or `pow(a,b)`                   |
| `abs(x)`              | `abs(x)`                                |
| `round(x)`            | `round(x)`                              |
| `a == b`              | `a == b`                                |
| `a != b`              | `a != b`                                |
| `a < b`               | `a < b`                                 |
| `a > b`               | `a > b`                                 |
| `a <= b`              | `a <= b`                                |
| `a >= b`              | `a >= b`                                |
| `a is b`              | `a == b` (values) or `a.addr == b.addr` |
| `a is not b`          | `a != b`                                |
| `a and b`             | `a and b`                               |
| `a or b`              | `a or b`                                |
| `not a`               | `not a`                                 |
| `a & b` (bitwise)     | `a and b` (bitwise on int)              |
| `a \| b` (bitwise)    | `a or b` (bitwise on int)               |
| `a ^ b` (bitwise xor) | `a xor b`                               |
| `~a` (bitwise not)    | `not a`                                 |
| `a << n`              | `a shl n`                               |
| `a >> n`              | `a shr n`                               |
| `1 < x < 10`          | `x in 1..<10` or `1 < x and x < 10`     |

#### Strings

| Python               | Nim                                 |
| -------------------- | ----------------------------------- |
| `"hello"`            | `"hello"`                           |
| `'hello'`            | `"hello"` (no single-quote strings) |
| `r"\n"` (raw)        | `r"\n"` (raw)                       |
| `b"bytes"`           | `"bytes"` cast to `seq[byte]`       |
| `f"Hi {name}"`       | `&"Hi {name}"` (strformat)          |
| `"a" + "b"`          | `"a" & "b"`                         |
| `"x" * 3`            | `"x".repeat(3)`                     |
| `len(s)`             | `s.len`                             |
| `s[0]`               | `s[0]` (returns `char`)             |
| `s[-1]`              | `s[^1]`                             |
| `s[1:4]`             | `s[1..3]`                           |
| `s[1:]`              | `s[1..^1]`                          |
| `s[:4]`              | `s[0..3]`                           |
| `s.upper()`          | `s.toUpperAscii()`                  |
| `s.lower()`          | `s.toLowerAscii()`                  |
| `s.strip()`          | `s.strip()`                         |
| `s.split(",")`       | `s.split(",")`                      |
| `",".join(lst)`      | `lst.join(",")`                     |
| `s.replace(a,b)`     | `s.replace(a,b)`                    |
| `s.startswith(p)`    | `s.startsWith(p)`                   |
| `s.endswith(p)`      | `s.endsWith(p)`                     |
| `s.find(sub)`        | `s.find(sub)`                       |
| `s.count(sub)`       | `s.count(sub)`                      |
| `s.format(...)`      | `&"..."` or `format(...)`           |
| `s.encode("utf-8")`  | `s.toBytes()`                       |
| `str(x)`             | `$x`                                |
| `ord('a')`           | `ord('a')`                          |
| `chr(65)`            | `char(65)`                          |
| `s in text`          | `s in text` or `text.contains(s)`   |
| `re.match(p, s)`     | `s.match(re(p))`                    |
| `re.sub(p, r, s)`    | `s.replace(re(p), r)`               |
| `textwrap.dedent(s)` | `s.dedent()`                        |

#### Collections — Lists / Sequences

| Python                     | Nim                                  |
| -------------------------- | ------------------------------------ |
| `[1, 2, 3]`                | `@[1, 2, 3]` (seq)                   |
| `[1, 2, 3]`                | `[1, 2, 3]` (fixed array)            |
| `list()`                   | `newSeq[T]()`                        |
| `[None]*n`                 | `newSeq[T](n)`                       |
| `lst[0]`                   | `lst[0]`                             |
| `lst[-1]`                  | `lst[^1]`                            |
| `lst[1:4]`                 | `lst[1..3]`                          |
| `lst[::2]`                 | `lst.filterIt(i mod 2==0)` (or loop) |
| `len(lst)`                 | `lst.len`                            |
| `lst.append(x)`            | `lst.add(x)`                         |
| `lst.extend(lst2)`         | `lst.add(lst2)`                      |
| `lst.insert(i,x)`          | `lst.insert(x, i)`                   |
| `lst.pop()`                | `lst.pop()`                          |
| `lst.pop(i)`               | `lst.delete(i)`                      |
| `lst.remove(x)`            | `lst.del(lst.find(x))`               |
| `lst.index(x)`             | `lst.find(x)`                        |
| `x in lst`                 | `x in lst`                           |
| `lst.count(x)`             | `lst.count(x)`                       |
| `lst.sort()`               | `lst.sort()`                         |
| `sorted(lst)`              | `lst.sorted()`                       |
| `lst.reverse()`            | `lst.reverse()`                      |
| `reversed(lst)`            | `lst.reversed()`                     |
| `lst.copy()`               | `lst` (seq assignment copies)        |
| `lst + lst2`               | `lst & lst2` or `concat(lst, lst2)`  |
| `list(map(f,lst))`         | `lst.mapIt(f(it))`                   |
| `list(filter(f,lst))`      | `lst.filterIt(f(it))`                |
| `sum(lst)`                 | `lst.foldl(a+b)`                     |
| `any(f(x) for x in lst)`   | `lst.anyIt(f(it))`                   |
| `all(f(x) for x in lst)`   | `lst.allIt(f(it))`                   |
| `enumerate(lst)`           | `lst.pairs` or `for i, v in lst`     |
| `zip(a, b)`                | `zip(a, b)`                          |
| `list(set(lst))`           | `lst.deduplicate()`                  |
| `[x**2 for x in lst]`      | `lst.mapIt(it*it)`                   |
| `[x for x in lst if p(x)]` | `lst.filterIt(p(it))`                |
| `functools.reduce(f,lst)`  | `lst.foldl(f(a,b))`                  |
| `itertools.chain(a,b)`     | `concat(a, b)`                       |
| `itertools.islice(it,n)`   | `lst.take(n)` (from sequtils)        |
| `itertools.groupby(lst,f)` | `lst.groupBy(f)` (from sequtils)     |
| `collections.deque`        | `Deque[T]` (from std/deques)         |
| `heapq`                    | `HeapQueue[T]` (from std/heapqueue)  |

#### Collections — Tuples

| Python                       | Nim                                             |
| ---------------------------- | ----------------------------------------------- |
| `(1, 2, 3)`                  | `(1, 2, 3)`                                     |
| `(1,)` (single)              | `(1,)`                                          |
| `t[0]`                       | `t[0]`                                          |
| `a, b = t`                   | `let (a, b) = t`                                |
| `a, *rest = t`               | `let (a, rest) = (t[0], t[1..^1])`              |
| `namedtuple('P', ['x','y'])` | `tuple[x,y: int]` or `type P = tuple[x,y: int]` |
| `t._replace(x=1)`            | `(x: 1, y: t.y)`                                |

#### Collections — Dictionaries / Tables

| Python                    | Nim                          |
| ------------------------- | ---------------------------- |
| `{"a": 1}`                | `{"a": 1}.toTable`           |
| `dict()`                  | `initTable[K,V]()`           |
| `d[k]`                    | `d[k]`                       |
| `d.get(k)`                | `d.getOrDefault(k)`          |
| `d.get(k, v)`             | `d.getOrDefault(k, v)`       |
| `d[k] = v`                | `d[k] = v`                   |
| `del d[k]`                | `d.del(k)`                   |
| `k in d`                  | `d.hasKey(k)` or `k in d`    |
| `k not in d`              | `not d.hasKey(k)`            |
| `d.keys()`                | `d.keys`                     |
| `d.values()`              | `d.values`                   |
| `d.items()`               | `d.pairs`                    |
| `len(d)`                  | `d.len`                      |
| `d.update(d2)`            | `d.merge(d2)` or manual loop |
| `d.pop(k)`                | `d.del(k)` (no return)       |
| `d.setdefault(k,v)`       | `d.mgetOrPut(k, v)`          |
| `{k:v for k,v in pairs}`  | `collect` (from sugar)       |
| `collections.OrderedDict` | `OrderedTable[K,V]`          |
| `collections.Counter`     | `CountTable[T]`              |
| `collections.defaultdict` | `toTable` + `mgetOrPut`      |

#### Collections — Sets

| Python         | Nim                                               |
| -------------- | ------------------------------------------------- |
| `{1, 2, 3}`    | `{1, 2, 3}` (bit set for small ints)              |
| `set()`        | `initHashSet[T]()`                                |
| `s.add(x)`     | `s.incl(x)`                                       |
| `s.remove(x)`  | `s.excl(x)`                                       |
| `x in s`       | `x in s`                                          |
| `s1 & s2`      | `s1 * s2` (intersection)                          |
| `s1 \| s2`     | `s1 + s2` (union)                                 |
| `s1 - s2`      | `s1 - s2` (difference)                            |
| `s1 ^ s2`      | `s1 -+- s2` (symmetric diff)                      |
| `s1 <= s2`     | `s1 <= s2` (subset)                               |
| `s1 < s2`      | `s1 < s2` (proper subset)                         |
| `len(s)`       | `s.len` or `card(s)`                              |
| `frozenset(s)` | `set[T]` (all Nim sets are immutable value types) |

#### Functions

| Python                   | Nim                               |
| ------------------------ | --------------------------------- |
| `def f(x): return x`     | `proc f(x: int): int = x`         |
| `def f(x) -> int:`       | `proc f(x: int): int =`           |
| `def f(x=5):`            | `proc f(x: int = 5) =`            |
| `def f(*args):`          | `proc f(args: varargs[T]) =`      |
| `def f(**kw):`           | (use object/tuple parameter)      |
| `lambda x: x*2`          | `proc(x: int): int = x*2`         |
| `lambda x: x*2`          | `(x: int) => x*2` (sugar)         |
| `f(*lst)`                | `f(lst)` for varargs              |
| `f(**d)`                 | no direct equivalent              |
| `functools.partial(f,x)` | manual wrapper or template        |
| `functools.lru_cache`    | manual `Table` cache              |
| `@decorator`             | `{.pragma.}` or template wrapping |
| `yield x` (generator)    | `yield x` (iterator)              |
| `next(gen)`              | iterate with `for`                |
| `return`                 | `return` or set `result`          |
| implicit `return None`   | implicit `return result`          |

#### Control Flow

| Python                       | Nim                            |
| ---------------------------- | ------------------------------ |
| `if a: ...`                  | `if a: ...`                    |
| `elif b: ...`                | `elif b: ...`                  |
| `else: ...`                  | `else: ...`                    |
| `x if c else y`              | `if c: x else: y` (expression) |
| `match x:`                   | `case x`                       |
| `case 1:`                    | `of 1:`                        |
| `case _:`                    | `else:`                        |
| `for x in lst:`              | `for x in lst:`                |
| `for i,x in enumerate(lst):` | `for i,x in lst:`              |
| `for k,v in d.items():`      | `for k,v in d:`                |
| `while cond:`                | `while cond:`                  |
| `break`                      | `break`                        |
| `continue`                   | `continue`                     |
| `pass`                       | `discard`                      |
| `range(n)`                   | `0..<n`                        |
| `range(a,b)`                 | `a..<b`                        |
| `range(a,b,step)`            | `countup(a,b,step)`            |
| `range(b,a,-1)`              | `countdown(b,a)`               |
| no labeled break             | `block label:` + `break label` |

#### Object-Oriented

| Python                       | Nim                                      |
| ---------------------------- | ---------------------------------------- |
| `class Foo:`                 | `type Foo = object`                      |
| `class Foo(Bar):`            | `type Foo = object of Bar`               |
| `class Foo(Bar): # ref`      | `type Foo = ref object of Bar`           |
| `def __init__(self):`        | `proc init(f: var Foo) =`                |
| `def method(self):`          | `proc method(f: Foo) =` (UFCS)           |
| `def __str__(self):`         | `proc \`$\`(f: Foo): string =`           |
| `def __eq__(self,o):`        | `proc \`==\`(a,b: Foo): bool =`          |
| `def __lt__(self,o):`        | `proc \`<\`(a,b: Foo): bool =`           |
| `def __add__(self,o):`       | `proc \`+\`(a,b: Foo): Foo =`            |
| `def __len__(self):`         | `proc len(f: Foo): int =`                |
| `def __getitem__(self,i):`   | `proc \`[]\`(f: Foo, i: int): T =`       |
| `def __setitem__(self,i,v):` | `proc \`[]=\`(f: var Foo, i:int, v:T) =` |
| `def __contains__(self,x):`  | `proc \`in\`(x: T, f: Foo): bool =`      |
| `def __iter__(self):`        | `iterator items(f: Foo): T =`            |
| `self.x`                     | `f.x`                                    |
| `super().__init__()`         | `procCall init(Base(self))`              |
| `isinstance(x, T)`           | `x of T` (ref objects)                   |
| `@staticmethod`              | `proc` (no self param)                   |
| `@classmethod`               | (no direct equiv, use module-level proc) |
| `@property`                  | proc with same name as field (UFCS)      |
| `@abstractmethod`            | `{.base.}` pragma on method              |
| `__slots__`                  | default (Nim objects have no dict)       |
| `dataclasses.dataclass`      | `type Foo = object` with fields          |
| `NamedTuple`                 | `tuple[field: Type]`                     |

#### Error Handling

| Python                  | Nim                                       |
| ----------------------- | ----------------------------------------- |
| `try: ...`              | `try: ...`                                |
| `except E as e:`        | `except E as e:`                          |
| `except (E1, E2):`      | `except E1, E2:`                          |
| `except Exception:`     | `except CatchableError:`                  |
| `except:` (bare)        | `except Exception:`                       |
| `else:` (no exception)  | (no direct equiv)                         |
| `finally:`              | `finally:`                                |
| `raise Exception(msg)`  | `raise newException(E, msg)`              |
| `raise` (re-raise)      | `raise` (re-raise current)                |
| `raise ValueError(msg)` | `raise newException(ValueError, msg)`     |
| `Exception`             | `CatchableError`                          |
| `RuntimeError`          | `CatchableError`                          |
| `ValueError`            | `ValueError`                              |
| `TypeError`             | `TypeError`                               |
| `IndexError`            | `IndexDefect`                             |
| `KeyError`              | `KeyError`                                |
| `IOError`               | `IOError`                                 |
| `OSError`               | `OSError`                                 |
| `OverflowError`         | `OverflowDefect`                          |
| `ZeroDivisionError`     | `DivByZeroDefect`                         |
| `AttributeError`        | `FieldDefect`                             |
| `assert cond`           | `assert cond`                             |
| `assert cond, msg`      | `assert cond, msg`                        |
| custom exceptions       | `type MyError = object of CatchableError` |
| context manager `with`  | `defer:` for cleanup                      |

#### Modules

| Python                       | Nim                            |
| ---------------------------- | ------------------------------ |
| `import module`              | `import module`                |
| `import module as m`         | `import module as m`           |
| `from module import f`       | `from module import f`         |
| `from module import *`       | `import module` (all exported) |
| `from module import f as g`  | `from module import f as g`    |
| `__all__ = [...]`            | `*` suffix on exported symbols |
| `if __name__ == "__main__":` | `when isMainModule:`           |
| `importlib.reload(m)`        | (recompile)                    |
| `sys.path.append(p)`         | `--path:p` compile flag        |
| `__file__`                   | `currentSourcePath()`          |
| `__doc__`                    | `bindSym("f").getImpl...`      |

#### Type System

| Python            | Nim                                    |
| ----------------- | -------------------------------------- |
| `Optional[T]`     | `Option[T]` (std/options)              |
| `Union[A,B]`      | (use object variants)                  |
| `List[T]`         | `seq[T]`                               |
| `Dict[K,V]`       | `Table[K,V]`                           |
| `Tuple[A,B]`      | `tuple[a: A, b: B]`                    |
| `Callable[[A],B]` | `proc(a: A): B`                        |
| `TypeVar('T')`    | `[T]` generic param                    |
| `Protocol`        | `concept`                              |
| `cast(T, x)`      | `T(x)` (safe) or `cast[T](x)` (unsafe) |
| `Any`             | (avoid; use generics or `RootObj`)     |

#### Concurrency

| Python                       | Nim                        |
| ---------------------------- | -------------------------- |
| `threading.Thread(target=f)` | `spawn f()` (threadpool)   |
| `t.start()`                  | (spawn starts immediately) |
| `t.join()`                   | `sync()`                   |
| `threading.Lock()`           | `Lock` (std/locks)         |
| `lock.acquire()`             | `lock.acquire()`           |
| `lock.release()`             | `lock.release()`           |
| `with lock:`                 | `withLock(lock):`          |
| `queue.Queue()`              | `Channel[T]`               |
| `q.put(x)`                   | `ch.send(x)`               |
| `q.get()`                    | `ch.recv()`                |
| `concurrent.futures.Future`  | `FlowVar[T]`               |
| `async def f():`             | `proc f() {.async.}`       |
| `await x`                    | `await x`                  |
| `asyncio.run(f())`           | `waitFor f()`              |
| `asyncio.gather(a,b)`        | `await (a, b)`             |
| `asyncio.sleep(n)`           | `await sleepAsync(n)`      |

#### I/O

| Python                      | Nim                                   |
| --------------------------- | ------------------------------------- |
| `print(x)`                  | `echo x`                              |
| `print(x, end="")`          | `stdout.write($x)`                    |
| `print(x, file=sys.stderr)` | `stderr.writeLine($x)`                |
| `input("prompt: ")`         | `readLine(stdin)`                     |
| `open("f","r")`             | `open("f", fmRead)`                   |
| `open("f","w")`             | `open("f", fmWrite)`                  |
| `open("f","a")`             | `open("f", fmAppend)`                 |
| `f.read()`                  | `readFile("f")` (whole file)          |
| `f.readlines()`             | `lines("f")` (iterator)               |
| `f.write(s)`                | `f.write(s)`                          |
| `f.close()`                 | `f.close()` or `defer: f.close()`     |
| `with open(...) as f:`      | `let f = open(...); defer: f.close()` |
| `os.path.join(a,b)`         | `a / b`                               |
| `os.path.exists(p)`         | `fileExists(p)` or `dirExists(p)`     |
| `os.makedirs(p)`            | `createDir(p)`                        |
| `os.remove(p)`              | `removeFile(p)`                       |
| `os.getcwd()`               | `getCurrentDir()`                     |
| `os.listdir(p)`             | `walkDir(p)`                          |
| `shutil.copy(a,b)`          | `copyFile(a, b)`                      |
| `shutil.move(a,b)`          | `moveFile(a, b)`                      |
| `pathlib.Path(p).stem`      | `p.splitFile().name`                  |
| `pathlib.Path(p).suffix`    | `p.splitFile().ext`                   |
| `pathlib.Path(p).parent`    | `p.parentDir()`                       |
| `json.loads(s)`             | `parseJson(s)`                        |
| `json.dumps(x)`             | `$x` (for JsonNode)                   |

#### Standard Library Mapping

| Python module  | Nim equivalent                              |
| -------------- | ------------------------------------------- |
| `os`           | `std/os`                                    |
| `sys`          | `std/os`, compile-time flags                |
| `math`         | `std/math`                                  |
| `random`       | `std/random`                                |
| `re`           | `std/re` or `std/nre` (PCRE)                |
| `json`         | `std/json`                                  |
| `datetime`     | `std/times`                                 |
| `time`         | `std/times`, `std/monotimes`                |
| `pathlib`      | `std/os` (path functions)                   |
| `io`           | `std/streams`, `std/io`                     |
| `collections`  | `std/tables`, `std/deques`, `std/heapqueue` |
| `itertools`    | `std/sequtils`, `std/sugar`                 |
| `functools`    | `std/sugar`, manual closures                |
| `typing`       | built-in type system                        |
| `dataclasses`  | `type X = object`                           |
| `abc`          | `concept`, `{.base.}`                       |
| `threading`    | `std/threadpool`, `std/locks`               |
| `asyncio`      | `std/asyncdispatch`                         |
| `subprocess`   | `std/osproc`                                |
| `socket`       | `std/net`, `std/asyncnet`                   |
| `http.client`  | `std/httpclient`                            |
| `http.server`  | `std/asynchttpserver`                       |
| `urllib.parse` | `std/uri`                                   |
| `hashlib`      | `std/md5`, `std/sha1`, `std/hashes`         |
| `base64`       | `std/base64`                                |
| `csv`          | manual or `std/parsecsv`                    |
| `sqlite3`      | `db_sqlite` (std) or `norm`                 |
| `logging`      | `std/logging`                               |
| `unittest`     | `std/unittest`                              |
| `argparse`     | `std/parseopt` or `cligen` package          |
| `pprint`       | `std/json` (pretty)                         |
| `copy`         | value semantics (just assign)               |
| `struct`       | `std/marshal` or cast                       |
| `ctypes`       | `{.importc.}` pragma                        |

---

## Bonus Section: Patterns That Don't Map Directly

Some Python patterns require a different approach in Nim. This section covers the most common transitions that trip up Python developers.

### Python's Duck Typing → Nim's Concepts

```python
# Python - duck typing, checked at runtime
def process(obj):
    return obj.serialize()   # works for any object with .serialize()
```

```nim
# Nim - concepts checked at compile time
type Serializable = concept x
  x.serialize() is string

proc process(obj: Serializable): string =
  obj.serialize()

# Any type with a serialize() proc works automatically
type
  User = object
    name: string

  Document = object
    content: string

proc serialize(u: User): string = u.name
proc serialize(d: Document): string = d.content

echo process(User(name: "Alice"))       # works
echo process(Document(content: "Hi"))  # works
```

### Python's `*args/**kwargs` → Nim's Alternatives

```python
# Python - flexible argument passing
def log(level, *messages, **context):
    print(f"[{level}]", *messages, **context)

log("INFO", "hello", "world", timestamp=True)
```

```nim
# Nim approach 1: varargs for homogeneous args
proc log(level: string, messages: varargs[string]) =
  echo "[" & level & "] " & messages.join(" ")

log("INFO", "hello", "world")

# Nim approach 2: object for named options
type LogOptions = object
  timestamp: bool
  color: bool

proc log(level: string,
         message: string,
         opts: LogOptions = LogOptions()) =
  let prefix = if opts.timestamp: "[$1]" % $now() else: ""
  echo prefix & "[" & level & "] " & message

log("INFO", "hello", LogOptions(timestamp: true))
```

### Python's `None` Returns → Nim's Option Type

```python
# Python - return None to signal "not found"
def find_user(id: int) -> Optional[User]:
    if id in database:
        return database[id]
    return None

user = find_user(42)
if user is not None:
    print(user.name)
```

```nim
import std/options

proc findUser(id: int): Option[User] =
  if id in database:
    some(database[id])
  else:
    none(User)

let user = findUser(42)

# Option forces you to handle the missing case
if user.isSome:
  echo user.get().name

# Or use map/get with default
echo user.map(proc(u: User): string = u.name).get("unknown")

# Or use pattern matching style
case user.isSome
of true:  echo user.get().name
of false: echo "not found"
```

### Python's `with` Statement → Nim's `defer`

```python
# Python
with open("file.txt") as f:
    data = f.read()
# file is closed here automatically

with db.connection() as conn:
    conn.execute("SELECT ...")
# connection closed here
```

```nim
# Nim - defer executes when scope exits (normally or via exception)
proc readFile(path: string): string =
  let f = open(path, fmRead)
  defer: f.close()    # guaranteed cleanup
  result = f.readAll()

# Multiple defers execute in reverse order (like a stack)
proc complexOperation() =
  let a = acquireResourceA()
  defer: releaseA(a)

  let b = acquireResourceB()
  defer: releaseB(b)    # released before a

  doWork(a, b)
  # b released here, then a released
```

### Python's List Comprehensions → Nim's `collect`

```python
# Python
squares = [x**2 for x in range(10) if x % 2 == 0]
lookup = {v: i for i, v in enumerate(words)}
```

```nim
import std/sugar

# collect macro - most Pythonic
let squares = collect:
  for x in 0..<10:
    if x mod 2 == 0:
      x * x
# → @[0, 4, 16, 36, 64]

# For tables
let lookup = collect(initTable()):
  for i, v in words:
    {v: i}

# Or the functional style (also common)
import std/sequtils
let squares2 = toSeq(0..<10)
  .filterIt(it mod 2 == 0)
  .mapIt(it * it)
```

### Python's `__repr__` and `__str__` → Nim's `$`

```python
class Point:
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"
```

```nim
type Point = object
  x, y: float

# $ is used for both repr and str in Nim
proc `$`(p: Point): string =
  "Point(x=" & $p.x & ", y=" & $p.y & ")"

# echo uses $ automatically
let p = Point(x: 1.0, y: 2.0)
echo p              # → Point(x=1.0, y=2.0)
echo $p             # → Point(x=1.0, y=2.0)
echo "Point: " & $p # → Point: Point(x=1.0, y=2.0)

# repr() equivalent - Nim has a built-in repr
echo repr(p)        # → low-level representation
```

### Python's Dynamic Attributes → Nim's Object Variants

```python
# Python - shapes can have different attributes
class Shape:
    def __init__(self, kind, **kwargs):
        self.kind = kind
        for k, v in kwargs.items():
            setattr(self, k, v)

circle = Shape("circle", radius=5.0)
rect   = Shape("rect", width=10.0, height=5.0)
```

```nim
# Nim - object variants (discriminated unions)
type
  ShapeKind = enum
    Circle, Rectangle, Triangle

  Shape = object
    case kind: ShapeKind    # discriminator field
    of Circle:
      radius: float
    of Rectangle:
      width, height: float
    of Triangle:
      base, triHeight: float

let circle = Shape(kind: Circle, radius: 5.0)
let rect   = Shape(kind: Rectangle, width: 10.0, height: 5.0)

# Access is safe - compiler enforces checking kind first
proc area(s: Shape): float =
  case s.kind
  of Circle:    3.14159 * s.radius * s.radius
  of Rectangle: s.width * s.height
  of Triangle:  0.5 * s.base * s.triHeight

echo circle.area()   # → 78.53...
echo rect.area()     # → 50.0
```

### Python's Decorators → Nim's Templates and Macros

```python
# Python decorator
import functools
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.3f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

```nim
import std/times

# Template approach (wraps any code block)
template timed(name: string, body: untyped): untyped =
  let start = now()
  body
  let elapsed = now() - start
  echo name & " took " & $elapsed.inMilliseconds & "ms"

timed("slow operation"):
  sleep(1000)

# For wrapping specific proc types, use a macro
import std/macros

macro timedProc(p: untyped): untyped =
  # AST transformation that wraps a proc with timing
  let procName = p[0]
  let body = p[^1]
  p[^1] = quote do:
    let start = now()
    `body`
    echo `procName`.repr & " took " & $(now() - start).inMilliseconds & "ms"
  p

proc slowFunction() {.timedProc.} =
  sleep(1000)
```

---

## Common Gotchas for Python Developers

### 1. Integer Division

```python
# Python 3 - / always gives float
10 / 3    # → 3.3333...
10 // 3   # → 3
```

```nim
# Nim - / gives float only if operands are float
10 / 3      # COMPILE ERROR (ambiguous) or float in some contexts
10 div 3    # → 3   integer division
10.0 / 3.0  # → 3.333...
```

### 2. String Concatenation Type

```python
# Python - implicit conversion
print("Value: " + str(42))  # or use f-strings
```

```nim
# Nim - explicit conversion required
echo "Value: " & $42    # $ converts to string
echo "Value: " & $true  # works for any type
```

### 3. Truthiness

```python
# Python - many things are falsy
if not []:      # True
if not 0:       # True
if not "":      # True
if not {}:      # True
```

```nim
# Nim - ONLY nil and false are falsy
# 0, "", @[], and {} are all TRUE
if @[].len == 0:   # correct way
if s == "":        # correct way
if n == 0:         # correct way
```

### 4. No Implicit Return of Last Expression

```python
# Python (in lambdas, expression = return value)
f = lambda x: x * 2   # implicit return
```

```nim
# Nim proc - use result or return explicitly
proc double(x: int): int =
  result = x * 2   # use result

proc double2(x: int): int =
  x * 2   # single expression form - this DOES work

proc double3(x: int): int =
  return x * 2   # explicit return
```

### 5. Mutable Default Arguments

```python
# Python famous gotcha
def append_to(lst=[]):
    lst.append(1)
    return lst

append_to()  # [1]
append_to()  # [1, 1] ← surprising!
```

```nim
# Nim - default arguments are always fresh (no gotcha)
proc appendTo(lst: seq[int] = @[]): seq[int] =
  result = lst
  result.add(1)

discard appendTo()   # @[1]
discard appendTo()   # @[1] ← always fresh copy
```

### 6. Variable Capture in Closures

```python
# Python - captures by reference
fns = [lambda: i for i in range(3)]
fns[0]()  # → 2 (captures i, not value of i at creation)
fns[1]()  # → 2
fns[2]()  # → 2
```

```nim
# Nim - captures by value (with lexical binding)
var fns: seq[proc(): int]
for i in 0..<3:
  let captured = i     # capture the current value
  fns.add(proc(): int = captured)

echo fns[0]()   # → 0
echo fns[1]()   # → 1
echo fns[2]()   # → 2
```

### 7. Method vs Proc Dispatch

```python
# Python - all method calls are virtual
class Animal:
    def speak(self): return "..."

class Dog(Animal):
    def speak(self): return "Woof"

animals = [Dog(), Animal()]
for a in animals:
    a.speak()   # always calls the right one
```

```nim
# Nim - proc: static dispatch (by declared type)
#       method: dynamic dispatch (by runtime type)
type
  Animal = ref object of RootObj
  Dog = ref object of Animal

proc speak(a: Animal): string = "..."       # static dispatch
method speak2(a: Animal): string {.base.} = "..."  # dynamic dispatch
method speak2(d: Dog): string = "Woof"

var animals: seq[Animal] = @[Dog(), Animal()]

for a in animals:
  echo a.speak()    # always "..." (proc, static dispatch)
  echo a.speak2()   # "Woof" then "..." (method, dynamic dispatch)
```

### 8. Copying vs Referencing

```python
# Python - everything is a reference
a = [1, 2, 3]
b = a           # b references same list
b.append(4)
print(a)        # [1, 2, 3, 4] — modified!
```

```nim
# Nim - sequences are value types (copied on assignment)
var a = @[1, 2, 3]
var b = a           # b is a COPY
b.add(4)
echo a    # @[1, 2, 3]  — unchanged!
echo b    # @[1, 2, 3, 4]

# ref objects behave like Python (reference semantics)
type Node = ref object
  value: int

var n1 = Node(value: 1)
var n2 = n1        # n2 references same object
n2.value = 99
echo n1.value      # → 99 — same object!
```

---

## Where to Go From Here

1. **[nim-lang.org](https://nim-lang.org)** — official site with downloads,
   news, and links to all resources.

2. **[Nim Manual](https://nim-lang.org/docs/manual.html)** — the authoritative
   language reference. Bookmark it from day one. It is dense but precise.

3. **[Nim Standard Library Docs](https://nim-lang.org/docs/lib.html)** —
   documentation for every stdlib module. The search is fast and the
   examples are practical.

4. **[Nim by Example](https://nim-by-example.github.io)** — short focused code
   examples for each language feature. Best for quick lookup when you know
   what you want but not the syntax.

5. **[Nim in Action](https://www.manning.com/books/nim-in-action)** — the main
   published book. Slightly dated on some APIs but excellent for building
   mental models.

6. **[Nimble Package Directory](https://nimble.directory)** — browse all
   published packages. Before writing something yourself, check if it
   exists.

7. **[Nim Forum](https://forum.nim-lang.org)** — the primary community forum.
   Search before asking — many Python-to-Nim questions are already
   answered in detail.

8. **[Nim Discord](https://discord.gg/nim)** — real-time chat. The
   `#beginners` channel is welcoming and fast-moving.

9. **[Awesome Nim](https://github.com/xflywind/awesome-nim)** — curated list
   of Nim libraries, tools, and resources by category.

10. **Read stdlib source code** — Nim's standard library is written in Nim and
    is a gold mine of idiomatic patterns. Find it in
    `~/.choosenim/toolchains/nim-VERSION/lib/`. Start with `sequtils.nim`
    and `strutils.nim` which are readable and instructive.

11. **First projects to build** — start with things you've already written in
    Python to directly compare:
    - A CLI tool using `std/parseopt` or the `cligen` package
    - A JSON processor using `std/json` and `std/httpclient`
    - A file renamer or organizer using `std/os`
    - A small TCP server using `std/asyncnet`
      These cover the majority of real-world Nim usage.

12. **Learn the memory model properly** — read the official blog posts on
    ARC and ORC. Understanding `=destroy`, `=copy`, and `=sink` hooks
    will make you a much better Nim programmer and explain why Nim code
    is fast. The post _"Nim 1.4 brings even more innovations"_ and
    _"Introducing ARC"_ on nim-lang.org are the best starting points.

13. **Study a real Nim project** — reading production Nim code is
    invaluable. Good examples include:
    - [Nimbus](https://github.com/status-im/nimbus-eth1) — Ethereum client
    - [Karax](https://github.com/karaxnim/karax) — frontend web framework
    - [Norm](https://github.com/moigagoo/norm) — ORM library
    - [Prologue](https://github.com/planety/prologue) — web framework
    - [Nimpy](https://github.com/yglukhov/nimpy) — Python interop (ironic but useful!)

14. **The Zen of Nim** — Nim's design philosophy rewards you for thinking
    about:
    - _Efficiency_: default to stack allocation, avoid heap where possible
    - _Expressiveness_: UFCS, templates, and macros let you build abstractions
      that cost nothing at runtime
    - _Correctness_: let the type system and compiler catch mistakes, not
      runtime crashes
    - _Pragmatism_: C interop is trivial, deploy as a single binary, target
      any platform
