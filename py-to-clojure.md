# From Python to Clojure: A Comprehensive Guide

## Table of Contents

1. [Mental Model Shift](#mental-model-shift)
2. [The Environment](#the-environment)
3. [Basic Syntax](#basic-syntax)
4. [Data Types](#data-types)
5. [Variables and Binding](#variables-and-binding)
6. [Functions](#functions)
7. [Control Flow](#control-flow)
8. [Collections](#collections)
9. [Sequences and Lazy Evaluation](#sequences-and-lazy-evaluation)
10. [Destructuring](#destructuring)
11. [Namespaces and Modules](#namespaces-and-modules)
12. [Macros](#macros)
13. [Concurrency](#concurrency)
14. [Protocols and Multimethods](#protocols-and-multimethods)
15. [Error Handling](#error-handling)
16. [Java Interop](#java-interop)
17. [I/O and Side Effects](#io-and-side-effects)
18. [Testing](#testing)
19. [Build Tools and Dependencies](#build-tools-and-dependencies)
20. [Common Patterns and Idioms](#common-patterns-and-idioms)
21. [Transducers](#transducers)
22. [core.async](#coreasync)
23. [Debugging and REPL](#debugging-and-repl)
24. [Style and Conventions](#style-and-conventions)

---

## 1. Mental Model Shift

This is the most important section. Before writing a single line, you need to understand how Clojure differs fundamentally from Python.

### Python thinks in objects and mutation. Clojure thinks in values and transformation.

In Python, your program is a sequence of instructions that mutate state. Objects have identity and you change them over time. In Clojure, your program is a series of **transformations on immutable values**. Data doesn't change — you produce new data from old data.

```python
# Python - mutation is normal
my_list = [1, 2, 3]
my_list.append(4)      # mutates my_list
my_list[0] = 99        # mutates my_list
# my_list is now [99, 2, 3, 4]
```

```clojure
;; Clojure - values never change
(def my-list [1 2 3])
(conj my-list 4)       ; → [1 2 3 4]  (new vector, my-list unchanged)
(assoc my-list 0 99)   ; → [99 2 3]   (new vector, my-list unchanged)
;; my-list is still [1 2 3]
```

### The Three Pillars

**1. Immutability by default** — Data structures cannot be changed. All operations return new data structures. This eliminates an entire class of bugs.

**2. Pure functions preferred** — Functions that take values and return values, with no side effects. The same input always produces the same output. Programs become easier to reason about, test, and parallelize.

**3. Explicit side effects** — Mutation exists but is explicit and controlled through reference types (`atom`, `ref`, `agent`). You always know when something is changing state.

### Clojure is a Lisp on the JVM

Clojure compiles to JVM bytecode, which means:

- You get access to the entire Java ecosystem
- Performance is close to Java
- Startup time is slow (JVM warmup) — mitigated by tools like `clj` with socket REPL
- ClojureScript compiles to JavaScript for browser/Node.js use

### Everything is Data

In Clojure, code is data. This is the core Lisp insight. A function call `(+ 1 2)` is literally a list containing the symbol `+` and the numbers `1` and `2`. This makes metaprogramming (macros) natural and powerful.

### Persistent Data Structures

Clojure's immutable collections are **structurally shared** — when you "change" a collection, the new version shares most of its structure with the old version. This makes immutability efficient, not slow:

```
Original: [1 2 3 4 5]
After (assoc v 2 99): [1 2 99 4 5]

These two vectors share nodes in memory — only the changed path is new.
O(log32 n) for most operations, effectively O(1) in practice.
```

---

## 2. The Environment

### Installation

```bash
# Install Clojure CLI tools (recommended)
# macOS
brew install clojure/tools/clojure

# Linux
curl -L -O https://github.com/clojure/brew-install/releases/latest/download/linux-install.sh
chmod +x linux-install.sh
sudo ./linux-install.sh

# Verify
clj --version
```

### The REPL

The Clojure REPL (Read-Eval-Print Loop) is central to the development workflow. Unlike Python's REPL which is mostly for quick tests, Clojure developers write entire programs interactively through the REPL:

```bash
# Start a REPL
clj

# You'll see:
# Clojure 1.11.1
# user=>

# Evaluate expressions:
user=> (+ 1 2 3)
6
user=> (str "Hello, " "World!")
"Hello, World!"
user=> (def x 42)
#'user/x
user=> x
42
```

### Project Structure

```bash
# Create a new project with deps.edn (modern approach)
# deps.edn is like Python's pyproject.toml + setup.cfg

my-project/
├── deps.edn           # dependencies and build config
├── src/
│   └── my_project/   # note: hyphens in ns → underscores in dirs
│       └── core.clj  # main namespace
├── test/
│   └── my_project/
│       └── core_test.clj
└── resources/         # static files, config
```

```clojure
;; deps.edn
{:paths ["src" "resources"]
 :deps {org.clojure/clojure {:mvn/version "1.11.1"}}
 :aliases
 {:test {:extra-paths ["test"]
         :extra-deps {io.github.cognitect-labs/test-runner
                      {:git/tag "v0.5.1" :git/sha "dfb30dd"}}}
  :dev {:extra-deps {nrepl/nrepl {:mvn/version "1.0.0"}}}}}
```

Python equivalent:

```toml
# pyproject.toml
[build-system]
requires = ["setuptools"]

[project]
name = "my-project"
version = "0.1.0"
dependencies = []

[project.optional-dependencies]
test = ["pytest"]
dev = ["ipython"]
```

### Leiningen (Alternative Build Tool)

```bash
# Leiningen is the older, also widely used build tool
# Like a combination of pip, virtualenv, and setuptools

lein new app my-project
cd my-project
lein repl        # start REPL
lein test        # run tests
lein run         # run main
lein uberjar     # build standalone JAR
```

```clojure
;; project.clj (Leiningen)
(defproject my-project "0.1.0-SNAPSHOT"
  :description "My Clojure project"
  :dependencies [[org.clojure/clojure "1.11.1"]]
  :main my-project.core)
```

### Editor Setup

The best Clojure editors have REPL integration where you evaluate code directly from your editor:

- **Emacs + CIDER**: the gold standard
- **VS Code + Calva**: most accessible for newcomers
- **IntelliJ + Cursive**: best IDE experience
- **Neovim + Conjure**: for vim users

All of them connect to a running nREPL server and let you:

- Evaluate the expression under the cursor
- Evaluate the whole file
- Inspect values inline
- Navigate to definitions

---

## 3. Basic Syntax

### Everything is a Function Call (Almost)

```python
# Python - different syntax for different constructs
result = add(1, 2)      # function call
x = 5                   # assignment
if x > 3: ...           # statement
for i in range(5): ...  # statement
```

```clojure
;; Clojure - everything is an expression in prefix form
(add 1 2)               ; function call
(def x 5)               ; var definition
(if (> x 3) ...)        ; expression (returns a value)
(dotimes [i 5] ...)     ; expression
```

### Prefix Notation

All operators are functions called in prefix form:

```python
# Python - infix operators
1 + 2 + 3 + 4
x > 3 and y < 10
not True
```

```clojure
;; Clojure - prefix, can take multiple args
(+ 1 2 3 4)          ; → 10
(and (> x 3) (< y 10))
(not true)
```

Arithmetic never has ambiguity:

```python
2 + 3 * 4    # = 14 (need to know precedence)
```

```clojure
(+ 2 (* 3 4))    ; = 14, obviously
(* (+ 2 3) 4)    ; = 20, obviously
```

### Literals

```clojure
;; Numbers
42          ; Long (64-bit integer)
42N         ; BigInteger (arbitrary precision)
3.14        ; Double
3.14M       ; BigDecimal (exact)
22/7        ; Ratio (exact fraction!)
0xff        ; hex → 255
2r1010      ; binary → 10
8r17        ; octal → 15

;; Strings
"hello"
"line1\nline2"
"tab\there"

;; Characters
\a          ; the character 'a'
\newline    ; newline character
\space      ; space character
\tab        ; tab character

;; Booleans
true
false

;; Nil
nil         ; null / None / null pointer

;; Keywords (like Ruby symbols or Elixir atoms)
:name
:user/name  ; namespaced keyword
::name      ; auto-namespace to current ns

;; Symbols
foo
my-func
+           ; + is just a symbol that refers to a function
```

### Keywords: A Critical Clojure Concept

Keywords have no Python equivalent but are extremely common in Clojure. They are like strings that evaluate to themselves and are used as map keys, option flags, and enumerations:

```clojure
:name          ; evaluates to :name
:status        ; evaluates to :status
:ok            ; common as a status value

;; Keywords are functions that look themselves up in maps!
(def person {:name "Alice" :age 30})
(:name person)     ; → "Alice"  (keyword as function)
(:age person)      ; → 30
(:city person "NYC") ; → "NYC"  (default value)
```

### Comments

```clojure
;; Double semicolon for top-level comments (conventional)
; Single semicolon for inline comments

;; Line comment
(def x 5)   ; x is five

;; Block comment (form comment — comments out an entire expression)
#_(this entire form is ignored
   even if it spans multiple lines)

;; Rich comment block — code you keep but don't ship
;; Convention for REPL experiments at the bottom of files
(comment
  (start-server! {:port 3000})
  (stop-server!)
  (run-tests))
```

### Whitespace and Formatting

Commas are whitespace in Clojure:

```clojure
;; These are identical:
{:a 1, :b 2, :c 3}
{:a 1 :b 2 :c 3}

;; Commas sometimes aid readability in maps:
{:name "Alice", :age 30, :city "NYC"}
```

---

## 4. Data Types

### Type Comparison Table

| Python         | Clojure                 | Notes                      |
| -------------- | ----------------------- | -------------------------- |
| `int`          | `Long` / `BigInteger`   | `42` or `42N`              |
| `float`        | `Double` / `BigDecimal` | `3.14` or `3.14M`          |
| `complex`      | no built-in             | use library                |
| `str`          | `String`                | immutable, Java strings    |
| `bool`         | `Boolean`               | `true`/`false`             |
| `None`         | `nil`                   | also falsy                 |
| `list`         | `vector` / `list`       | `[1 2 3]` / `(list 1 2 3)` |
| `tuple`        | `vector`                | `[1 2 3]`                  |
| `dict`         | `map`                   | `{:a 1 :b 2}`              |
| `set`          | `set`                   | `#{1 2 3}`                 |
| `frozenset`    | `set`                   | all sets are immutable     |
| class instance | `record` / `map`        |                            |
| `bytes`        | `byte-array`            | Java byte array            |

### Numbers

```clojure
;; Integer arithmetic is exact
(+ 1000000000 2000000000)  ; → 3000000000 (no overflow with N suffix)

;; Use N for arbitrary precision
(* 1000000000N 1000000000N) ; → 1000000000000000000N

;; Ratios are exact fractions (no floating point error!)
(/ 1 3)         ; → 1/3  (not 0.333...)
(+ 1/3 2/3)     ; → 1    (exact!)
(* 3 1/3)       ; → 1

;; Coerce to float when needed
(double 1/3)    ; → 0.3333333333333333

;; Math functions
(Math/abs -5)       ; → 5       (Java interop)
(Math/sqrt 16.0)    ; → 4.0
(Math/pow 2 10)     ; → 1024.0
(max 1 2 3)         ; → 3
(min 1 2 3)         ; → 1
(mod 10 3)          ; → 1
(quot 10 3)         ; → 3   (truncating division)
(rem 10 3)          ; → 1   (remainder, sign follows dividend)
(inc 5)             ; → 6   (increment)
(dec 5)             ; → 4   (decrement)
(zero? 0)           ; → true
(pos? 1)            ; → true
(neg? -1)           ; → true
(even? 4)           ; → true
(odd? 3)            ; → true
```

### Strings

```clojure
;; String operations
(count "hello")                 ; → 5      (like len())
(str "hello" " " "world")       ; → "hello world"  (concat)
(str 42)                        ; → "42"   (any value to string)
(subs "hello" 1 3)              ; → "el"   (like "hello"[1:3])
(.toUpperCase "hello")          ; → "HELLO" (Java method)
(.toLowerCase "HELLO")          ; → "hello"
(clojure.string/upper-case "hello")   ; → "HELLO"  (Clojure wrapper)
(clojure.string/lower-case "HELLO")   ; → "hello"
(clojure.string/trim "  hello  ")     ; → "hello"
(clojure.string/split "a,b,c" #",")   ; → ["a" "b" "c"]
(clojure.string/join ", " ["a" "b"])  ; → "a, b"
(clojure.string/starts-with? "hello" "hel") ; → true
(clojure.string/ends-with? "hello" "lo")    ; → true
(clojure.string/includes? "hello" "ell")    ; → true
(clojure.string/replace "hello" "l" "r")    ; → "herro"
(clojure.string/blank? "")              ; → true
(clojure.string/blank? "  ")            ; → true

;; Require the namespace first:
(require '[clojure.string :as str])
;; Then use:
(str/upper-case "hello")
(str/split "a,b,c" #",")
```

### String Formatting

```python
# Python
f"Hello, {name}! You are {age} years old."
"Hello, {}!".format(name)
"Hello, %s!" % name
```

```clojure
;; format (Java's String.format)
(format "Hello, %s! You are %d years old." name age)

;; str for concatenation
(str "Hello, " name "! You are " age " years old.")

;; Clojure has no f-string equivalent built-in
;; Libraries like clojure.pprint or stencil fill this gap
;; Or use format:
(format "%.2f" 3.14159)  ; → "3.14"
```

### Characters

```clojure
\a          ; character literal
\A          ; uppercase A
\1          ; character '1'
\space      ; space
\newline    ; newline
\tab        ; tab

(char 65)   ; → \A  (from int)
(int \A)    ; → 65  (to int)
(char? \a)  ; → true
```

### Booleans and Truthiness

Critical difference from Python:

```clojure
;; In Clojure, ONLY nil and false are falsy
;; Everything else (including 0, "", []) is truthy!

(if 0   "yes" "no")   ; → "yes"  (unlike Python!)
(if ""  "yes" "no")   ; → "yes"  (unlike Python!)
(if []  "yes" "no")   ; → "yes"  (unlike Python!)
(if nil "yes" "no")   ; → "no"
(if false "yes" "no") ; → "no"

;; This matches Emacs Lisp behavior
;; Python: 0, "", [], {}, 0.0, set() are all falsy
;; Clojure: only nil and false are falsy
```

---

## 5. Variables and Binding

### Vars (Global Definitions)

```python
# Python
x = 42
MY_CONSTANT = "hello"
message = "initial"
message = "changed"  # reassignment is normal
```

```clojure
;; def: create a var (global in a namespace)
(def x 42)
(def my-constant "hello")

;; Vars are not variables — redefining is possible but discouraged
;; Use def only for things that are truly global and stable
(def message "initial")
;; Redefining is possible but signals a design smell:
(def message "changed")   ; works but avoid this pattern
```

### Local Bindings with let

```python
# Python - local variables in functions are automatic
def foo():
    x = 10
    y = 20
    return x + y
```

```clojure
;; let: create local bindings (immutable)
(let [x 10
      y 20]
  (+ x y))   ; → 30

;; let bindings are sequential (like Python's walrus in a chain):
(let [x 10
      y (* x 2)    ; can reference x
      z (+ x y)]  ; can reference x and y
  z)   ; → 30

;; Unlike Python, you can't reassign a let binding:
(let [x 10]
  (def x 20)   ; BAD - don't do this
  x)           ; → 20 but this is terrible style
```

### Destructuring in let

```python
# Python
a, b, c = [1, 2, 3]
first, *rest = [1, 2, 3, 4]
x, y = point  # if point is a tuple
```

```clojure
;; Sequential destructuring (vectors/lists)
(let [[a b c] [1 2 3]]
  (+ a b c))   ; → 6

(let [[first & rest] [1 2 3 4]]
  [first rest])   ; → [1 (2 3 4)]

(let [[a _ c] [1 2 3]]   ; _ is convention for "ignore"
  [a c])   ; → [1 3]

;; Map destructuring
(let [{:keys [name age]} {:name "Alice" :age 30}]
  (str name " is " age))   ; → "Alice is 30"

;; With defaults
(let [{:keys [name age]
       :or {age 18}} {:name "Bob"}]
  age)   ; → 18 (default)

;; Nested
(let [{:keys [name]
       {:keys [city]} :address}
     {:name "Alice" :address {:city "NYC"}}]
  (str name " lives in " city))
```

### Atoms: Managed Mutable State

When you need state that changes over time, use an `atom`:

```python
# Python - mutable state is the default
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
```

```clojure
;; Clojure - explicit mutable state with atom
(def counter (atom 0))

;; Read the current value with @
@counter          ; → 0

;; Update with swap! (applies a function to current value)
(swap! counter inc)    ; → 1
(swap! counter + 5)    ; → 6
(swap! counter (fn [n] (* n 2)))  ; → 12

;; Reset to a specific value
(reset! counter 0)     ; → 0
@counter               ; → 0

;; swap! is atomic — safe for concurrent use
;; It may retry if another thread changes the value simultaneously
```

### The Four Reference Types

| Type    | Use case           | Coordination     | Async        |
| ------- | ------------------ | ---------------- | ------------ |
| `atom`  | independent state  | uncoordinated    | synchronous  |
| `ref`   | coordinated state  | STM transactions | synchronous  |
| `agent` | independent state  | uncoordinated    | asynchronous |
| `var`   | thread-local state | per-thread       | synchronous  |

---

## 6. Functions

### Defining Functions

```python
def greet(name, greeting="Hello"):
    """Greet someone."""
    return f"{greeting}, {name}!"
```

```clojure
(defn greet
  "Greet someone."
  ([name] (greet name "Hello"))           ; 1-arg version
  ([name greeting]                         ; 2-arg version
   (str greeting ", " name "!")))

(greet "Alice")           ; → "Hello, Alice!"
(greet "Bob" "Hi")        ; → "Hi, Bob!"
```

### Parameter Variations

```clojure
;; Required parameters
(defn foo [a b c] ...)

;; Variadic (rest) parameters
(defn foo [a b & more] ...)
;; more is a seq of remaining args (like *args in Python)

(defn sum [& nums]
  (apply + nums))

(sum 1 2 3 4 5)   ; → 15

;; Keyword arguments (via destructuring)
(defn create-user [{:keys [name age email]
                    :or {age 18 email "none"}}]
  {:name name :age age :email email})

(create-user {:name "Alice" :age 30 :email "alice@example.com"})
(create-user {:name "Bob"})   ; age=18, email="none" by default
```

### Anonymous Functions

```python
# Python
square = lambda x: x * x
add = lambda x, y: x + y
```

```clojure
;; Full syntax
(fn [x] (* x x))
(fn [x y] (+ x y))

;; Shorthand syntax #() — very common
#(* % %)         ; single arg: % refers to it
#(+ %1 %2)       ; multiple args: %1, %2, %3, etc.
#(apply + %&)    ; %& is the rest args

;; Examples:
(map #(* % %) [1 2 3 4 5])     ; → (1 4 9 16 25)
(filter #(> % 2) [1 2 3 4 5])  ; → (3 4 5)
(reduce #(+ %1 %2) [1 2 3])    ; → 6

;; Assign to a var
(def square #(* % %))
(square 5)   ; → 25
```

### Higher-Order Functions

```python
# Python
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
total = sum(numbers)
```

```clojure
(def numbers [1 2 3 4 5])

(map #(* % %) numbers)          ; → (1 4 9 16 25)
(filter even? numbers)          ; → (2 4)
(reduce + numbers)              ; → 15
(reduce + 0 numbers)            ; → 15 (with initial value)

;; These return lazy sequences, not concrete lists
;; Usually that's fine — they behave like sequences

;; apply: spread a collection as arguments
(apply + [1 2 3 4 5])   ; → 15  (like Python's sum(*args))
(apply str ["a" "b" "c"]) ; → "abc"

;; comp: function composition
(def inc-and-double (comp #(* % 2) inc))
(inc-and-double 3)   ; → 8  (* (inc 3) 2)

;; partial: partial application
(def add5 (partial + 5))
(add5 3)   ; → 8

;; Python equivalent:
;; from functools import partial
;; add5 = partial(operator.add, 5)
```

### Closures

```python
def make_adder(n):
    def adder(x):
        return x + n
    return adder

add5 = make_adder(5)
add5(3)  # → 8
```

```clojure
(defn make-adder [n]
  (fn [x] (+ x n)))   ; closes over n

(def add5 (make-adder 5))
(add5 3)   ; → 8

;; Closures over mutable atoms:
(defn make-counter []
  (let [count (atom 0)]
    {:increment #(swap! count inc)
     :decrement #(swap! count dec)
     :value     #(deref count)}))

(def c (make-counter))
((:increment c))   ; → 1
((:increment c))   ; → 2
((:value c))       ; → 2
```

### Threading Macros: The Pipeline Pattern

This is one of the most important Clojure idioms. It replaces nested function calls with a readable pipeline, similar to Python's method chaining:

```python
# Python - nested calls (read inside-out)
result = sorted(filter(lambda x: x > 2, map(lambda x: x * 2, [1, 2, 3, 4, 5])))

# Python - method chaining (more readable)
result = (pd.DataFrame(data)
          .query("value > 2")
          .assign(doubled=lambda df: df.value * 2)
          .sort_values("doubled"))
```

```clojure
;; Without threading — nested, read inside-out
(sort (filter #(> % 2) (map #(* % 2) [1 2 3 4 5])))

;; With -> (thread-first): inserts as FIRST argument
(-> [1 2 3 4 5]
    (map #(* % 2) ,,,)   ; doesn't work well here
    ...)

;; With ->> (thread-last): inserts as LAST argument — use for sequences
(->> [1 2 3 4 5]
     (map #(* % 2))       ; → (2 4 6 8 10)
     (filter #(> % 4))    ; → (6 8 10)
     (sort)               ; → (6 8 10)
     (take 2))            ; → (6 8)
; Reads top to bottom like a pipeline!

;; -> (thread-first) — use for object-style operations
(-> "hello world"
    (clojure.string/upper-case)    ; "HELLO WORLD"
    (clojure.string/split #" ")   ; ["HELLO" "WORLD"]
    (first))                       ; "HELLO"

;; as-> for mixed threading
(as-> 5 x
  (* x 2)     ; x = 10
  (+ x 3)     ; x = 13
  (str "Result: " x))  ; → "Result: 13"

;; some-> (short-circuits on nil)
(some-> {:user {:name "Alice"}}
        :user
        :name
        clojure.string/upper-case)  ; → "ALICE"

(some-> {:user nil}
        :user
        :name
        clojure.string/upper-case)  ; → nil (short-circuits at :user)
```

---

## 7. Control Flow

### If and Its Relatives

```python
if x > 0:
    print("positive")
elif x < 0:
    print("negative")
else:
    print("zero")
```

```clojure
;; if: exactly two branches, returns a value
(if (> x 0)
  "positive"
  "not positive")

;; if with side effects needs do (like progn in Emacs Lisp)
(if (> x 0)
  (do (println "positive")
      (log-it x)
      "positive")
  "not positive")

;; cond: multiple branches (like elif chain)
(cond
  (> x 0) "positive"
  (< x 0) "negative"
  :else   "zero")       ; :else is just a truthy keyword, convention

;; condp: cond with a shared predicate/value
(condp = status
  :ok      "Success"
  :error   "Failed"
  :pending "Waiting"
  "Unknown")             ; default (no test)

;; case: constant dispatch (like switch, must be compile-time constants)
(case day
  :monday    "Start of week"
  :friday    "End of week"
  (:saturday :sunday) "Weekend"  ; multiple values → same result
  "Midweek")            ; default

;; when: if without else (returns nil if false)
(when (> x 0)
  (println "positive")
  (* x 2))              ; returns value of last expression

;; when-not: inverted when
(when-not (empty? coll)
  (first coll))

;; if-let: bind and test in one
(if-let [user (find-user id)]
  (str "Found: " (:name user))
  "User not found")

;; when-let: like if-let without else
(when-let [result (expensive-operation)]
  (process result))
```

### Loop and Recur

Clojure does not have `for` loops that mutate a variable. Instead it has `loop`/`recur` for explicit recursion, and sequence operations for most iteration:

```python
# Python imperative loop
result = 0
for i in range(1, 11):
    result += i
```

```clojure
;; Idiomatic: use sequence functions
(reduce + (range 1 11))   ; → 55

;; loop/recur: explicit tail recursion (when you need manual control)
(loop [i 1
       result 0]
  (if (> i 10)
    result
    (recur (inc i) (+ result i))))   ; → 55

;; recur jumps back to the loop binding with new values
;; It's a TCO (tail call optimization) construct
;; The JVM doesn't support TCO natively so recur is how you do it

;; Recursive function with recur
(defn factorial [n]
  (loop [n n acc 1]
    (if (zero? n)
      acc
      (recur (dec n) (* acc n)))))

(factorial 10)   ; → 3628800
```

### For (Sequence Comprehension)

```python
# Python list comprehension
squares = [x**2 for x in range(10) if x % 2 == 0]
pairs = [(x, y) for x in [1,2,3] for y in [4,5,6]]
```

```clojure
;; Clojure for is a sequence comprehension (returns lazy seq)
(for [x (range 10)
      :when (even? x)]
  (* x x))
; → (0 4 16 36 64)

;; Multiple bindings → cartesian product (like nested for in Python)
(for [x [1 2 3]
      y [4 5 6]]
  [x y])
; → ([1 4] [1 5] [1 6] [2 4] [2 5] [2 6] [3 4] [3 5] [3 6])

;; With :let and :while
(for [x (range 10)
      :let [squared (* x x)]
      :while (< squared 50)]   ; stop when condition fails
  squared)
; → (0 1 4 9 16 25 36 49)

;; for is LAZY — use doall to force evaluation
(doall (for [x (range 3)] (println x)))
```

### doseq: Iteration for Side Effects

```python
for item in collection:
    print(item)
```

```clojure
;; doseq: like for but for side effects, returns nil
(doseq [item collection]
  (println item))

;; Multiple bindings
(doseq [x [1 2 3]
        y [:a :b]]
  (println x y))
```

---

## 8. Collections

Clojure's four core collection types are all immutable and persistent.

### Vectors

Vectors are Clojure's sequential, indexed collection. Unlike Python lists they're immutable but structurally shared:

```python
# Python list
lst = [1, 2, 3, 4, 5]
lst[2]        # 3
lst[1:4]      # [2, 3, 4]
lst.append(6) # mutates
```

```clojure
;; Vector literal
(def v [1 2 3 4 5])

;; Access
(v 2)          ; → 3   (vector is a function of its index)
(get v 2)      ; → 3   (safer, returns nil for out-of-bounds)
(get v 10)     ; → nil (no exception)
(get v 10 :default) ; → :default
(first v)      ; → 1
(last v)       ; → 5
(rest v)       ; → (2 3 4 5)   as seq
(next v)       ; → (2 3 4 5)   as seq (nil if empty, rest returns ())
(nth v 2)      ; → 3
(subvec v 1 4) ; → [2 3 4]   like v[1:4]

;; "Modification" (returns new vector)
(conj v 6)          ; → [1 2 3 4 5 6]   append
(assoc v 2 99)      ; → [1 2 99 4 5]    "update" index
(pop v)             ; → [1 2 3 4]       remove last
(update v 2 inc)    ; → [1 2 4 4 5]    apply fn to element

;; Info
(count v)       ; → 5   (like len())
(empty? v)      ; → false
(seq v)         ; → (1 2 3 4 5) as a seq

;; Build
(vec (range 5))         ; → [0 1 2 3 4]
(vector 1 2 3)          ; → [1 2 3]
(into [] (range 5))     ; → [0 1 2 3 4]
(mapv #(* % 2) [1 2 3]) ; → [2 4 6]  (map that returns vector)
```

### Lists

Clojure lists are singly-linked lists. They're rarely used as data structures (use vectors instead) but are the fundamental structure of code:

```clojure
;; List literal (must quote to avoid evaluation as function call)
'(1 2 3 4 5)
(list 1 2 3 4 5)

;; Access
(first '(1 2 3))   ; → 1
(rest '(1 2 3))    ; → (2 3)
(second '(1 2 3))  ; → 2

;; "Modification"
(conj '(1 2 3) 0)  ; → (0 1 2 3)  prepends! (vs vector which appends)

;; When to use list vs vector:
;; - vector []: when you need indexed access, building data
;; - list (): when building code (macros), or need efficient prepend
```

### Maps

Maps are Clojure's primary key-value structure, like Python dicts but immutable:

```python
# Python
d = {"name": "Alice", "age": 30}
d["name"]           # "Alice"
d.get("city", "NYC") # "NYC" (default)
d["email"] = "..."  # mutate
del d["age"]        # mutate
"name" in d         # True
d.keys()
d.values()
d.items()
```

```clojure
(def person {:name "Alice" :age 30})

;; Access (three ways)
(get person :name)       ; → "Alice"  (safe, returns nil if missing)
(person :name)           ; → "Alice"  (map as function)
(:name person)           ; → "Alice"  (keyword as function — most idiomatic)

;; With default
(get person :city "NYC")  ; → "NYC"
(:city person "NYC")      ; → "NYC"

;; "Modification" (returns new map)
(assoc person :email "alice@example.com")  ; → new map with :email
(dissoc person :age)                        ; → new map without :age
(merge {:a 1} {:b 2} {:a 99})              ; → {:a 99 :b 2}
(assoc-in person [:address :city] "NYC")   ; → nested assoc
(update person :age inc)                    ; → apply fn to value
(update-in person [:stats :score] + 10)    ; → nested update

;; Query
(contains? person :name)   ; → true  (key exists)
(keys person)              ; → (:name :age)
(vals person)              ; → ("Alice" 30)
(count person)             ; → 2

;; Build
(hash-map :a 1 :b 2)      ; → {:a 1 :b 2}
(zipmap [:a :b :c] [1 2 3]) ; → {:a 1 :b 2 :c 3}
(into {} [[:a 1] [:b 2]])   ; → {:a 1 :b 2}

;; Nested access
(get-in {:user {:name "Alice" :scores [10 20 30]}} [:user :scores 1])
; → 20
```

### Sets

```python
# Python
s = {1, 2, 3, 4}
s.add(5)           # mutate
s.remove(1)        # mutate
3 in s             # True
s1 & s2            # intersection
s1 | s2            # union
s1 - s2            # difference
```

```clojure
;; Set literal
#{1 2 3 4}

;; "Modification" (returns new set)
(conj #{1 2 3} 4)      ; → #{1 2 3 4}
(disj #{1 2 3} 2)      ; → #{1 3}

;; Query
(contains? #{1 2 3} 2)  ; → true
(#{1 2 3} 2)            ; → 2  (set as function — returns element or nil)

;; Set operations
(require '[clojure.set :as set])
(set/union #{1 2 3} #{3 4 5})        ; → #{1 2 3 4 5}
(set/intersection #{1 2 3} #{2 3 4}) ; → #{2 3}
(set/difference #{1 2 3} #{2 3})     ; → #{1}
(set/subset? #{1 2} #{1 2 3})        ; → true

;; Build
(set [1 2 2 3 3 3])     ; → #{1 2 3}  (deduplication!)
(into #{} [1 2 2 3])    ; → #{1 2 3}
```

### Nested Data Manipulation

This is where Clojure really shines — working with nested immutable data:

```clojure
(def state
  {:users [{:name "Alice" :score 10}
           {:name "Bob"   :score 20}]
   :config {:max-score 100
            :active true}})

;; Deep access
(get-in state [:users 0 :name])   ; → "Alice"
(get-in state [:config :max-score]) ; → 100

;; Deep update
(update-in state [:users 0 :score] + 5)
;; → {:users [{:name "Alice" :score 15} ...] :config ...}

(assoc-in state [:config :debug] true)
;; → adds :debug key to :config

;; update with function
(update state :users
        (fn [users]
          (map #(update % :score inc) users)))
```

---

## 9. Sequences and Lazy Evaluation

### The Sequence Abstraction

One of Clojure's most powerful features. Almost all collection operations return **lazy sequences** — they're not computed until needed:

```python
# Python generators are similar
def squares():
    for i in range(1000000):
        yield i * i

# takes memory for only one element at a time
first_five = list(itertools.islice(squares(), 5))
```

```clojure
;; Infinite lazy sequences — memory efficient
(def naturals (iterate inc 0))       ; 0, 1, 2, 3, ... forever
(def powers-of-2 (iterate #(* % 2) 1)) ; 1, 2, 4, 8, ...

(take 5 naturals)         ; → (0 1 2 3 4)
(take 10 powers-of-2)     ; → (1 2 4 8 16 32 64 128 256 512)

;; range: like Python's range (lazy)
(range)           ; 0, 1, 2, 3 ... forever
(range 5)         ; → (0 1 2 3 4)
(range 2 10)      ; → (2 3 4 5 6 7 8 9)
(range 0 10 2)    ; → (0 2 4 6 8)  with step

;; Sequence operations (all return lazy seqs)
(map f coll)          ; transform every element
(filter pred coll)    ; keep elements matching pred
(remove pred coll)    ; remove elements matching pred
(take n coll)         ; first n elements
(drop n coll)         ; all but first n elements
(take-while pred coll); take while pred is true
(drop-while pred coll); drop while pred is true
(partition n coll)    ; split into chunks of n
(partition-by f coll) ; split on function changes
(group-by f coll)     ; group into map by function
(sort-by f coll)      ; sort by key function
(frequencies coll)    ; count occurrences
(distinct coll)       ; remove duplicates
(flatten coll)        ; flatten nested collections
(interpose sep coll)  ; insert sep between elements
(interleave c1 c2)    ; interleave two collections
(concat c1 c2 c3)     ; concatenate collections
(mapcat f coll)       ; map then concatenate (like flatMap)
(zipmap keys vals)    ; create map from key/val seqs
```

### Realizing Lazy Sequences

```clojure
;; Lazy sequences are computed on demand
;; They must be "realized" to get a concrete value

;; These force realization:
(doall (map println [1 2 3]))  ; realize and keep
(dorun (map println [1 2 3]))  ; realize but discard (for side effects)
(vec (map inc [1 2 3]))        ; → [2 3 4]
(into [] (map inc [1 2 3]))    ; → [2 3 4]

;; WARNING: don't hold onto the head of a lazy seq
;; This causes memory issues (lazy seq gets materialized in memory):
(let [s (range 1000000)]
  (last s))   ; fine — s is not held
```

### Common Sequence Patterns

```python
# Python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# sum of squares of even numbers
sum(x**2 for x in numbers if x % 2 == 0)

# group by even/odd
from itertools import groupby
sorted_nums = sorted(numbers, key=lambda x: x % 2 == 0)
groups = {k: list(v) for k, v in groupby(sorted_nums, lambda x: x % 2 == 0)}

# running total
from itertools import accumulate
list(accumulate(numbers))
```

```clojure
(def numbers (range 1 11))

;; sum of squares of even numbers
(->> numbers
     (filter even?)
     (map #(* % %))
     (reduce +))
; → 220

;; group by even/odd
(group-by even? numbers)
; → {false [1 3 5 7 9], true [2 4 6 8 10]}

;; running total (reductions = Python's accumulate)
(reductions + numbers)
; → (1 3 6 10 15 21 28 36 45 55)

;; Frequency count
(frequencies ["a" "b" "a" "c" "b" "a"])
; → {"a" 3, "b" 2, "c" 1}

;; Partition into pairs
(partition 2 [1 2 3 4 5 6])
; → ((1 2) (3 4) (5 6))

;; Sliding window of size 3
(partition 3 1 [1 2 3 4 5])
; → ((1 2 3) (2 3 4) (3 4 5))
```

---

## 10. Destructuring

Clojure's destructuring is more powerful than Python's tuple unpacking:

```python
# Python
a, b, c = [1, 2, 3]
first, *rest = [1, 2, 3, 4]
{name, age} = person  # doesn't exist in Python
```

```clojure
;; In let, function params, for, doseq, etc.

;; Sequential destructuring
(let [[a b c] [1 2 3]]
  [a b c])   ; → [1 2 3]

;; With rest
(let [[first & rest] [1 2 3 4 5]]
  {:first first :rest rest})
; → {:first 1, :rest (2 3 4 5)}

;; Ignore positions
(let [[_ _ third] [1 2 3]]
  third)   ; → 3

;; Nested
(let [[[a b] c] [[1 2] 3]]
  [a b c])   ; → [1 2 3]

;; Map destructuring
(let [{:keys [name age]} {:name "Alice" :age 30}]
  (str name " is " age))

;; With rename
(let [{n :name a :age} {:name "Alice" :age 30}]
  [n a])   ; → ["Alice" 30]

;; With defaults
(let [{:keys [name age]
       :or {age 18}} {:name "Bob"}]
  [name age])   ; → ["Bob" 18]

;; Capture the whole map too
(let [{:keys [name] :as person} {:name "Alice" :age 30}]
  [name person])
; → ["Alice" {:name "Alice" :age 30}]

;; In function parameters
(defn greet [{:keys [name title]
              :or {title "Dr."}}]
  (str title " " name))

(greet {:name "Smith" :title "Prof."})   ; → "Prof. Smith"
(greet {:name "Jones"})                   ; → "Dr. Jones"
```

---

## 11. Namespaces and Modules

### Namespace Basics

```python
# Python
# file: myproject/utils.py
def helper():
    return 42

# file: myproject/core.py
from myproject.utils import helper
from myproject import utils
import myproject.utils as utils
```

```clojure
;; file: src/myproject/utils.clj
(ns myproject.utils
  "Utility functions for myproject.")

(defn helper []
  42)

;; file: src/myproject/core.clj
(ns myproject.core
  (:require [myproject.utils :as utils]
            [myproject.utils :refer [helper]]  ; import specific fn
            [clojure.string :as str]
            [clojure.set :refer [union intersection]]))

(utils/helper)    ; → 42  (qualified reference)
(helper)          ; → 42  (unqualified, from :refer)
(str/upper-case "hello")  ; → "HELLO"
```

### Namespace Forms

```clojure
(ns my.namespace
  "Docstring for the namespace."

  ;; Import Java classes
  (:import [java.util Date UUID]
           [java.io File])

  ;; Require Clojure namespaces
  (:require [clojure.string :as str]
            [clojure.set :as set]
            [some.library :refer [func1 func2]]
            [other.library :refer :all]))   ; import everything (avoid)

;; In REPL, require at any time:
(require '[clojure.string :as str])
(require '[my.namespace :reload])   ; reload after changes
```

### The ns-map and Vars

```clojure
;; Every var is fully qualified by namespace
;; clojure.core/+ is the full name of +
;; user/my-var is the full name of my-var in the user namespace

;; Get current namespace
*ns*

;; List everything in a namespace
(ns-publics 'clojure.string)    ; public vars
(ns-interns 'clojure.string)    ; all vars including private

;; Private vars (not exported)
(defn- private-helper []        ; defn- creates private fn
  "This is not in ns-publics")
```

---

## 12. Macros

Macros are code transformations at compile time. Clojure macros are more powerful and natural than Python decorators:

### Why Macros

```python
# Python - you cannot define new syntax
# This is impossible in Python:
with-retry 3:
    risky_operation()

# Python decorators work on functions only
@retry(times=3)
def risky_operation():
    ...
```

```clojure
;; Clojure - you can define any syntax
(defmacro with-retry [n & body]
  `(loop [attempts# ~n]
     (when (pos? attempts#)
       (try
         ~@body
         (catch Exception e#
           (if (= attempts# 1)
             (throw e#)
             (recur (dec attempts#))))))))

;; Now use it like a language construct:
(with-retry 3
  (risky-operation)
  (do-more-work))
```

### The Backquote Template

```clojure
;; Like Python f-strings but for code
;; ` (backtick) = template
;; ~ (tilde) = unquote (insert value)
;; ~@ (tilde-at) = unquote-splicing (insert sequence)

(defmacro my-and [x y]
  `(if ~x ~y false))

;; Expands to:
;; (if x-value y-value false)

;; Auto-gensym (# suffix) prevents variable capture
(defmacro with-timing [& body]
  `(let [start# (System/nanoTime)
         result# (do ~@body)
         elapsed# (/ (- (System/nanoTime) start#) 1e6)]
     (println (format "Elapsed: %.2f ms" elapsed#))
     result#))

;; start#, result#, elapsed# get unique names like start__1234__auto__
```

### Checking Macro Expansion

```clojure
;; macroexpand-1: expand one level
(macroexpand-1 '(when true (println "yes")))
; → (if true (do (println "yes")))

;; macroexpand: expand until no more macros
(macroexpand '(-> x (inc) (str)))
; → (str (inc x))

;; clojure.walk/macroexpand-all: deep expansion
(require '[clojure.walk :as walk])
(walk/macroexpand-all '(doto (java.util.ArrayList.) (.add 1) (.add 2)))
```

### Common Macro Patterns

```clojure
;; Define a timing macro (Python has timeit, this is more flexible)
(defmacro time-it [& body]
  `(let [start# (System/currentTimeMillis)]
     (let [result# (do ~@body)]
       (println "Time:" (- (System/currentTimeMillis) start#) "ms")
       result#)))

;; Define a debug macro that prints name and value
(defmacro dbg [x]
  `(let [v# ~x]
     (println (str '~x " => " v#))
     v#))

(dbg (* 2 3))
; prints: (* 2 3) => 6
; returns: 6

;; unless (because Clojure doesn't have it built in)
(defmacro unless [condition & body]
  `(when (not ~condition)
     ~@body))
```

---

## 13. Concurrency

This is where Clojure truly distinguishes itself from Python. Python's GIL makes true parallelism difficult; Clojure was designed from the ground up for concurrency.

### Atoms (Independent State)

```clojure
(def counter (atom 0))

;; swap! is atomic and retry-safe
(swap! counter inc)

;; Concurrent updates are safe
(def workers
  (repeatedly 10 #(future (dotimes [_ 1000] (swap! counter inc)))))

(doseq [w workers] @w)  ; wait for all
@counter   ; → 10000 (always, no race conditions)
```

### Futures (Asynchronous Execution)

```python
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor() as executor:
    future = executor.submit(expensive_computation)
    result = future.result()
```

```clojure
;; future: run in a thread pool, returns a future
(def f (future (Thread/sleep 2000) (+ 1 2)))

;; @ (deref) blocks until ready
@f         ; → 3  (blocks if not done)

;; with timeout
(deref f 3000 :timeout)   ; wait max 3 seconds, return :timeout if timeout

;; realized? checks without blocking
(realized? f)   ; → true/false

;; Run multiple things in parallel
(defn parallel-fetch [urls]
  (->> urls
       (map #(future (fetch-url %)))
       (map deref)))   ; deref blocks, but fetches run in parallel
```

### Promises

```python
# Python - like Future but you set the value manually
import concurrent.futures
p = concurrent.futures.Future()
p.set_result(42)
p.result()  # → 42
```

```clojure
(def p (promise))

;; In another thread:
(future (deliver p (compute-something)))

;; Wait for the value
@p   ; blocks until delivered
```

### Refs and Software Transactional Memory (STM)

For coordinating changes to multiple pieces of state atomically:

```python
# Python - no built-in STM
# You'd use locks and hope you get it right:
import threading
lock = threading.Lock()
with lock:
    account1.balance -= 100
    account2.balance += 100
```

```clojure
;; Clojure STM - atomic coordinated updates
(def account1 (ref 1000))
(def account2 (ref 500))

;; dosync: all changes inside are atomic
(defn transfer [from to amount]
  (dosync
    (when (>= @from amount)
      (alter from - amount)
      (alter to   + amount)
      true)))

(transfer account1 account2 200)
;; Both refs updated atomically, or neither changed
@account1   ; → 800
@account2   ; → 700

;; alter: apply function to ref value
;; ref-set: set ref to specific value
;; commute: like alter but more relaxed (order doesn't matter)
```

### Agents (Asynchronous State)

```clojure
(def agent-state (agent {:count 0 :log []}))

;; send: dispatch to thread pool (non-blocking)
(send agent-state update :count inc)
(send agent-state update :log conj "event-1")

;; send-off: for blocking operations (uses different thread pool)
(send-off agent-state (fn [state]
                         (Thread/sleep 1000)
                         (assoc state :done true)))

;; await: block until all sent actions are processed
(await agent-state)
@agent-state   ; → {:count 1 :log ["event-1"] :done true}
```

### core.async Channels (Brief Preview)

```clojure
(require '[clojure.core.async :as async])

;; Channels: like Go channels or Python asyncio queues
(def ch (async/chan 10))   ; buffered channel

;; Put value onto channel
(async/go (async/>! ch "hello"))

;; Take value from channel
(async/go (println (async/<! ch)))

;; Pipeline
(let [in  (async/chan 10)
      out (async/chan 10)]
  (async/pipeline 4 out (map inc) in)
  (async/onto-chan! in (range 10))
  (async/<!! (async/into [] out)))
; → [1 2 3 4 5 6 7 8 9 10]
```

---

## 14. Protocols and Multimethods

### Protocols (like Python Abstract Base Classes)

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass

    @abstractmethod
    def area(self):
        pass

class Circle(Drawable):
    def __init__(self, radius):
        self.radius = radius

    def draw(self):
        print(f"Drawing circle r={self.radius}")

    def area(self):
        return 3.14159 * self.radius ** 2
```

```clojure
;; Define protocol (interface)
(defprotocol Drawable
  "Things that can be drawn."
  (draw [this] "Draw this shape.")
  (area [this] "Calculate area."))

;; Implement with defrecord
(defrecord Circle [radius]
  Drawable
  (draw [this]
    (println (str "Drawing circle r=" radius)))
  (area [this]
    (* Math/PI radius radius)))

(defrecord Rectangle [width height]
  Drawable
  (draw [this]
    (println (str "Drawing rect " width "x" height)))
  (area [this]
    (* width height)))

;; Usage
(def c (->Circle 5))
(draw c)          ; → "Drawing circle r=5"
(area c)          ; → 78.539...

;; Extend existing types (like Python monkey-patching but structured)
(extend-protocol Drawable
  String
  (draw [s] (println s))
  (area [s] 0))

(draw "hello")   ; → "hello"
```

### Multimethods (Open Dispatch)

More flexible than protocols — dispatch on anything, not just type:

```python
# Python - usually done with if/elif or dict dispatch
def speak(animal):
    if animal["type"] == "dog":
        return "woof"
    elif animal["type"] == "cat":
        return "meow"
    else:
        return "..."
```

```clojure
;; Define dispatch function
(defmulti speak :type)   ; dispatch on :type key

;; Define methods
(defmethod speak :dog [animal]
  "woof")

(defmethod speak :cat [animal]
  "meow")

(defmethod speak :default [animal]
  "...")

;; Usage
(speak {:type :dog :name "Rex"})   ; → "woof"
(speak {:type :cat :name "Whiskers"}) ; → "meow"

;; Dispatch can be any function:
(defmulti process-shape (fn [shape] (:type shape)))

;; Dispatch on multiple values:
(defmulti collision-handler
  (fn [a b] [(:type a) (:type b)]))

(defmethod collision-handler [:circle :circle] [a b]
  (resolve-circle-circle a b))

(defmethod collision-handler [:circle :rect] [a b]
  (resolve-circle-rect a b))
```

### Records vs Maps

```clojure
;; defrecord: named type with fields (like a Python class for data)
(defrecord Person [name age email])

;; Create
(->Person "Alice" 30 "alice@example.com")  ; positional
(map->Person {:name "Alice" :age 30 :email "alice@example.com"}) ; from map

;; Records implement the map interface
(def alice (->Person "Alice" 30 "alice@example.com"))
(:name alice)         ; → "Alice"
(assoc alice :age 31) ; → new Person with age=31
(keys alice)          ; → (:name :age :email)

;; Check type
(instance? Person alice)   ; → true
(= (->Person "A" 1 "") (->Person "A" 1 ""))  ; → true (value equality)
```

---

## 15. Error Handling

### Try/Catch

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, IOError) as e:
    print(f"Other: {e}")
else:
    print("Success!")
finally:
    cleanup()
```

```clojure
(try
  (risky-operation)
  (catch clojure.lang.ExceptionInfo e
    ;; ExceptionInfo is Clojure's structured exception
    (println "Clojure error:" (ex-message e))
    (println "Data:" (ex-data e)))
  (catch java.lang.NumberFormatException e
    (println "Number format error:" (.getMessage e)))
  (catch Exception e
    (println "General error:" (.getMessage e)))
  (finally
    (cleanup)))
```

### Throwing Exceptions

```python
raise ValueError("something went wrong")
raise RuntimeError({"code": 404, "message": "not found"})
```

```clojure
;; throw: like Python's raise
(throw (Exception. "something went wrong"))
(throw (RuntimeException. "bad state"))

;; ex-info: Clojure's structured exception (recommended)
(throw (ex-info "Something went wrong"
                {:code 404
                 :message "not found"
                 :data some-value}))

;; Catch and inspect ex-info:
(try
  (throw (ex-info "Not found" {:code 404}))
  (catch clojure.lang.ExceptionInfo e
    (ex-message e)   ; → "Not found"
    (ex-data e)))    ; → {:code 404}
```

### Error Handling Patterns

```clojure
;; Option 1: Return nil on failure
(defn safe-divide [a b]
  (when (not= b 0)
    (/ a b)))

(safe-divide 10 2)   ; → 5
(safe-divide 10 0)   ; → nil

;; Option 2: Return {:ok value} or {:error message}
(defn parse-int [s]
  (try
    {:ok (Integer/parseInt s)}
    (catch NumberFormatException e
      {:error (str "Not a number: " s)})))

(parse-int "42")    ; → {:ok 42}
(parse-int "abc")   ; → {:error "Not a number: abc"}

;; Option 3: Use the `either` pattern from functional programming
;; Libraries like cats or failjure provide this

;; Option 4: Nilable with some->
(some-> (parse-user-id request)
        find-user
        :email
        send-email)
;; Short-circuits if any step returns nil
```

---

## 16. Java Interop

Clojure has seamless Java interop, giving access to the entire Java ecosystem:

```python
# Python - no Java interop (obviously)
# But the concepts map to:
# - import java.util.Date  →  calling obj.method()
# - new Date()             →  constructors
```

```clojure
;; Import in ns declaration
(ns my.ns
  (:import [java.util Date HashMap ArrayList]
           [java.io File]))

;; Create objects (ClassName.)
(Date.)                      ; new Date()
(HashMap.)                   ; new HashMap()
(ArrayList. [1 2 3])         ; new ArrayList with collection

;; Call methods (.methodName object args)
(.toUpperCase "hello")       ; "hello".toUpperCase()  → "HELLO"
(.substring "hello" 1 3)     ; "hello".substring(1, 3) → "el"
(.length "hello")            ; "hello".length() → 5
(.get (HashMap.) "key")      ; hashmap.get("key")

;; Static methods (ClassName/method)
(System/currentTimeMillis)   ; System.currentTimeMillis()
(Math/sqrt 16.0)             ; Math.sqrt(16.0)
(Integer/parseInt "42")      ; Integer.parseInt("42")

;; Static fields
Math/PI                      ; Math.PI → 3.14159...
Integer/MAX_VALUE            ; Integer.MAX_VALUE

;; doto: call multiple methods on same object
(doto (ArrayList.)
  (.add 1)
  (.add 2)
  (.add 3))
; → ArrayList [1 2 3]

;; .. (dot dot): chain method calls
(.. "hello world"
    toUpperCase
    (substring 0 5))
; → "HELLO"

;; Type checking
(instance? String "hello")   ; → true
(type "hello")               ; → java.lang.String
(class "hello")              ; → java.lang.String

;; Access fields
(.-x (Point. 3 4))           ; access public field x of Point
```

---

## 17. I/O and Side Effects

### Printing

```python
print("Hello")
print(f"Value: {x}")
import pprint
pprint.pprint(complex_obj)
```

```clojure
(println "Hello")                ; prints with newline
(print "No newline")             ; prints without newline
(prn "With quotes")              ; prints readable form (with quotes for strings)
(pr "readable")                  ; pr without newline
(pprint/pprint complex-obj)      ; pretty print

;; Format to string
(format "Value: %d" x)          ; → string

;; Print to stderr
(binding [*out* *err*]
  (println "Error message"))
```

### File I/O

```python
# Python
with open("file.txt", "r") as f:
    content = f.read()
    lines = f.readlines()

with open("output.txt", "w") as f:
    f.write("hello\n")
```

```clojure
(require '[clojure.java.io :as io])

;; Read entire file
(slurp "file.txt")                         ; → string
(slurp (io/resource "config.edn"))         ; from classpath

;; Read lines
(with-open [rdr (io/reader "file.txt")]
  (doall (line-seq rdr)))                  ; → seq of lines

;; Write to file
(spit "output.txt" "hello\n")              ; overwrite
(spit "output.txt" "more\n" :append true) ; append

;; Copy files
(io/copy (io/file "src.txt") (io/file "dst.txt"))

;; Check existence
(.exists (io/file "myfile.txt"))   ; → true/false

;; List directory
(->> (io/file ".")
     .listFiles
     (map #(.getName %)))
```

### Parsing EDN (Clojure's Data Format)

EDN (Extensible Data Notation) is like JSON but richer — it's Clojure's native data format:

```python
import json
data = json.loads('{"name": "Alice", "age": 30}')
```

```clojure
(require '[clojure.edn :as edn])

;; Parse EDN string
(edn/read-string "{:name \"Alice\" :age 30}")
; → {:name "Alice" :age 30}

;; Parse from file
(edn/read-string (slurp "config.edn"))

;; Write EDN
(pr-str {:name "Alice" :age 30})
; → "{:name \"Alice\", :age 30}"

;; JSON via cheshire library
(require '[cheshire.core :as json])
(json/parse-string "{\"name\": \"Alice\"}" true)  ; true = keyword keys
; → {:name "Alice"}
(json/generate-string {:name "Alice" :age 30})
; → "{\"name\":\"Alice\",\"age\":30}"
```

---

## 18. Testing

### clojure.test

```python
import pytest

def test_addition():
    assert add(2, 3) == 5

def test_greeting():
    assert greet("Alice") == "Hello, Alice!"

@pytest.mark.parametrize("input,expected", [
    ("42", 42),
    ("-1", -1),
    ("0", 0),
])
def test_parse_int(input, expected):
    assert parse_int(input) == expected
```

```clojure
(ns my.core-test
  (:require [clojure.test :refer [deftest testing is are]]
            [my.core :as sut]))  ; sut = system under test

(deftest test-addition
  (is (= (sut/add 2 3) 5))
  (is (not= (sut/add 2 3) 6))
  (is (= (sut/add 0 0) 0)))

(deftest test-greeting
  (is (= (sut/greet "Alice") "Hello, Alice!"))
  (is (string? (sut/greet "Bob"))))

;; testing: sub-groups within a deftest
(deftest test-user-operations
  (testing "creating users"
    (is (= (:name (sut/create-user "Alice")) "Alice")))
  (testing "validating users"
    (is (sut/valid-user? {:name "Alice" :age 30}))
    (is (not (sut/valid-user? {})))))

;; are: like parametrize in pytest
(deftest test-parse-int
  (are [input expected]
       (= (sut/parse-int input) expected)
    "42"   42
    "-1"  -1
    "0"    0))

;; Test for exceptions
(deftest test-error
  (is (thrown? Exception (sut/divide-by-zero)))
  (is (thrown-with-msg? Exception #"divide by zero"
                        (sut/divide-by-zero))))

;; Run tests
;; From REPL:
(clojure.test/run-tests 'my.core-test)

;; From command line:
;; clj -M:test
```

### Property-Based Testing with test.check

```python
# Python - hypothesis
from hypothesis import given
from hypothesis import strategies as st

@given(st.lists(st.integers()))
def test_reverse_reverse(lst):
    assert list(reversed(list(reversed(lst)))) == lst
```

```clojure
;; deps.edn:
;; org.clojure/test.check {:mvn/version "1.1.1"}

(require '[clojure.test.check :as tc]
         '[clojure.test.check.generators :as gen]
         '[clojure.test.check.properties :as prop])

(def reverse-reverse-prop
  (prop/for-all [lst (gen/vector gen/int)]
    (= (reverse (reverse lst)) lst)))

(tc/quick-check 1000 reverse-reverse-prop)
; → {:result true, :num-tests 1000, ...}

;; Integrated with clojure.test:
(require '[clojure.test.check.clojure-test :refer [defspec]])

(defspec reverse-is-involution 1000
  (prop/for-all [v (gen/vector gen/int)]
    (= v (vec (reverse (reverse v))))))
```

---

## 19. Build Tools and Dependencies

### deps.edn in Depth

```clojure
;; deps.edn — the modern approach

{;; Source paths
 :paths ["src" "resources"]

 ;; Maven dependencies
 :deps {org.clojure/clojure          {:mvn/version "1.11.1"}
        org.clojure/core.async        {:mvn/version "1.6.673"}
        cheshire/cheshire             {:mvn/version "5.11.0"}
        ring/ring-core                {:mvn/version "1.10.0"}
        metosin/malli                 {:mvn/version "0.13.0"}}

 :aliases
 {;; Dev alias — extra deps for development
  :dev
  {:extra-deps {nrepl/nrepl {:mvn/version "1.0.0"}
                cider/cider-nrepl {:mvn/version "0.42.1"}}
   :extra-paths ["dev"]}

  ;; Test alias
  :test
  {:extra-paths ["test"]
   :extra-deps {io.github.cognitect-labs/test-runner
                {:git/tag "v0.5.1"
                 :git/sha "dfb30dd7f88a71fa3f5d9d0d3c3a6dcccddc2bfb"}}}

  ;; Run tests
  :run-tests
  {:main-opts ["-m" "cognitect.test-runner"
               "-d" "test"]}

  ;; Uber jar
  :uberjar
  {:replace-deps {com.github.seancorfield/depstar {:mvn/version "2.1.303"}}
   :exec-fn hf.depstar/uberjar
   :exec-args {:aot true :main-class my.core}}

  ;; Start REPL with nREPL server
  :nrepl
  {:extra-deps {nrepl/nrepl {:mvn/version "1.0.0"}}
   :main-opts ["-m" "nrepl.cmdline" "--interactive"]}

  ;; Git dependency
  :some-lib
  {:extra-deps
   {io.github.some/library
    {:git/url "https://github.com/some/library"
     :git/sha "abc1234def5678"}}}}}
```

### Common Libraries

| Category        | Library                   | Python Equivalent    |
| --------------- | ------------------------- | -------------------- |
| HTTP Client     | `clj-http`                | `requests`           |
| HTTP Server     | `ring` + `compojure`      | `flask` / `django`   |
| JSON            | `cheshire`                | `json`               |
| SQL             | `next.jdbc`               | `sqlalchemy`         |
| Data validation | `malli` or `spec`         | `pydantic`           |
| Testing         | `clojure.test`            | `pytest`             |
| Async           | `core.async`              | `asyncio`            |
| Logging         | `timbre`                  | `logging`            |
| Config          | `aero`                    | `python-dotenv`      |
| Web scraping    | `enlive`                  | `beautifulsoup4`     |
| CLI             | `tools.cli`               | `argparse` / `click` |
| Dates           | `clj-time` or `java.time` | `datetime`           |

---

## 20. Common Patterns and Idioms

### Data-Oriented Programming

Clojure favors plain maps and vectors over custom types:

```python
# Python OOP approach
class User:
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

    def to_dict(self):
        return {"name": self.name, "age": self.age}
```

```clojure
;; Clojure data-oriented approach — just use maps
(def user {:name "Alice" :age 30 :email "alice@example.com"})

;; Functions operate on maps
(defn user-display-name [{:keys [name age]}]
  (str name " (" age ")"))

(user-display-name user)   ; → "Alice (30)"

;; Build pipelines that transform data
(defn process-users [users]
  (->> users
       (filter #(>= (:age %) 18))
       (map #(select-keys % [:name :email]))
       (sort-by :name)))
```

### The Update Pattern

```python
# Python - often mutate
user.age = user.age + 1
user.scores.append(100)
```

```clojure
;; Clojure - produce new values
(update user :age inc)
(update user :scores conj 100)
(update-in user [:address :city] str/upper-case)
```

### Memoization

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

```clojure
;; Built-in memoize
(def fibonacci
  (memoize
    (fn [n]
      (if (<= n 1)
        n
        (+ (fibonacci (- n 1))
           (fibonacci (- n 2)))))))

(fibonacci 40)   ; fast — cached
```

### Spec: Data Validation and Generation

```python
# Python - pydantic
from pydantic import BaseModel, validator

class User(BaseModel):
    name: str
    age: int

    @validator('age')
    def age_must_be_positive(cls, v):
        if v < 0:
            raise ValueError('must be positive')
        return v
```

```clojure
(require '[clojure.spec.alpha :as s])

;; Define specs
(s/def ::name string?)
(s/def ::age (s/and int? pos?))
(s/def ::email (s/and string? #(re-matches #".+@.+" %)))

(s/def ::user
  (s/keys :req-un [::name ::age]
          :opt-un [::email]))

;; Validate
(s/valid? ::user {:name "Alice" :age 30})   ; → true
(s/valid? ::user {:name "Bob" :age -1})     ; → false

;; Explain what's wrong
(s/explain ::user {:age -1})
;; Prints: val: {:age -1} fails spec: :my.ns/user
;;         at: [:name] predicate: clojure.core/string?

;; Generate test data!
(require '[clojure.spec.gen.alpha :as gen])
(gen/sample (s/gen ::user) 3)
; → ({:name "a" :age 1} {:name "bc" :age 3} ...)

;; Instrument functions
(s/fdef greet
  :args (s/cat :name ::name)
  :ret string?)

(require '[clojure.spec.test.alpha :as stest])
(stest/instrument `greet)   ; now greet validates args at runtime
```

---

## 21. Transducers

Transducers are composable, reusable transformations that work independently of the data source. They're more efficient than chaining map/filter because they avoid creating intermediate sequences:

```python
# Python - each step creates an intermediate list
result = list(filter(lambda x: x % 2 == 0,
                     map(lambda x: x * 2,
                         range(1000000))))
```

```clojure
;; Without transducers - creates intermediate lazy seqs
(->> (range 1000000)
     (map #(* % 2))
     (filter even?)
     (take 5))

;; With transducers - single pass, no intermediates
(def xf
  (comp (map #(* % 2))    ; note: no collection argument!
        (filter even?)
        (take 5)))

;; Apply to different sources
(transduce xf conj [] (range 1000000))   ; → [0 4 8 12 16]
(transduce xf + 0 (range 1000000))       ; → 40 (sum)
(sequence xf (range 1000000))            ; → lazy seq

;; Transducers work on channels too!
(async/chan 10 xf)   ; channel that transforms values as they pass through

;; Creating custom transducers
(defn my-map [f]
  (fn [rf]                    ; returns a reducing function
    (fn
      ([] (rf))               ; arity 0: init
      ([result] (rf result))  ; arity 1: completion
      ([result input]         ; arity 2: step
       (rf result (f input))))))
```

---

## 22. core.async

`core.async` brings Go-style concurrency with channels and goroutines (called `go` blocks):

```python
# Python asyncio
import asyncio

async def producer(queue):
    for i in range(5):
        await queue.put(i)
        await asyncio.sleep(0.1)

async def consumer(queue):
    while True:
        item = await queue.get()
        print(f"Got: {item}")

async def main():
    queue = asyncio.Queue()
    await asyncio.gather(producer(queue), consumer(queue))
```

```clojure
(require '[clojure.core.async :as async
           :refer [go >! <! >!! <!! chan close! timeout]])

;; Channels
(def ch (chan))           ; unbuffered
(def ch (chan 10))        ; buffered with size 10
(def ch (chan 10 (map inc)))  ; buffered with transducer

;; go blocks: lightweight "goroutines" (run on thread pool)
(go
  (dotimes [i 5]
    (<! (timeout 100))    ; non-blocking wait 100ms
    (>! ch i)))           ; put value onto channel

;; Take from channel
(go
  (loop []
    (when-let [v (<! ch)]
      (println "Got:" v)
      (recur))))

;; Blocking operations (outside go blocks)
(>!! ch "value")          ; blocking put
(<!! ch)                  ; blocking take

;; Select: wait for first ready channel (like Go's select)
(let [c1 (chan) c2 (chan)]
  (go (>! c1 "one"))
  (go (>! c2 "two"))
  (async/alt!
    c1 ([v] (println "From c1:" v))
    c2 ([v] (println "From c2:" v))))

;; Pipeline: parallel processing
(let [in  (chan 10)
      out (chan 10)]
  (async/pipeline-async
   4                          ; parallelism
   out
   (fn [x result-chan]        ; async processing fn
     (go
       (<! (timeout (rand-int 100)))  ; simulate async work
       (>! result-chan (* x x))
       (close! result-chan)))
   in)

  (async/onto-chan! in (range 10))
  (async/<!! (async/into [] out)))
```

---

## 23. Debugging and REPL

### The REPL-Driven Workflow

This is fundamentally different from Python's edit-run-debug cycle:

```python
# Python workflow
# 1. Write code in file
# 2. Run script / test
# 3. See error
# 4. Add print statements
# 5. Repeat
```

```clojure
;; Clojure workflow
;; 1. Start REPL connected to your editor
;; 2. Write function
;; 3. Evaluate it in place (C-c C-e in CIDER, Ctrl+Enter in Calva)
;; 4. Try it immediately in REPL
;; 5. Modify and re-evaluate
;; No restart needed — changes are live
```

### Inspecting Values

```clojure
;; Print and return (like Python's print but returns value)
(defmacro spy [x]
  `(let [v# ~x]
     (println "=>" '~x "=>" v#)
     v#))

(spy (* 2 3))   ; prints: => (* 2 3) => 6, returns 6

;; tap> and add-tap (Clojure 1.10+)
;; Like a pub/sub for debug values
(add-tap (fn [v] (println "TAP:" v)))
(tap> {:user "Alice" :action "login"})   ; → TAP: {:user Alice, :action login}

;; Clojure inspector
(require '[clojure.inspector :as inspector])
(inspector/inspect-tree {:a {:b {:c 42}}})   ; opens Swing GUI

;; pp/pprint for pretty printing
(require '[clojure.pprint :as pp])
(pp/pprint {:name "Alice" :scores [1 2 3] :address {:city "NYC"}})
```

### CIDER Debugger (Emacs)

```clojure
;; Add #break before any expression in source
(defn factorial [n]
  (if (zero? n)
    1
    (* n #break (factorial (dec n)))))
;; Evaluating this function will pause at #break

;; Debugger commands:
;; n = next step
;; c = continue
;; i = inspect value
;; e = eval expression in current context
;; q = quit
```

### Common REPL Operations

```clojure
;; Documentation
(doc map)              ; show docstring
(doc clojure.string/split)
(find-doc "reduce")    ; search docs by string

;; Source
(source map)           ; show source code!
(source defn)

;; Apropos: find functions by pattern
(apropos "filter")     ; → (filter filterv keep keep-indexed ...)

;; Reflection
(class "hello")        ; → java.lang.String
(type [1 2 3])         ; → clojure.lang.PersistentVector
(meta #'map)           ; metadata on the var
(dir clojure.string)   ; list all public vars in namespace

;; Reload namespace
(require '[my.namespace :reload])
(require '[my.namespace :reload-all])  ; reload and all dependencies

;; Stacktrace
(pst)                  ; print last stacktrace
(pst 10)               ; print last 10 frames
```

---

## 24. Style and Conventions

### Naming Conventions

```clojure
;; Functions and variables: kebab-case (hyphen-separated)
(defn calculate-user-score [user] ...)
(def default-timeout 5000)

;; Predicates end in ?
(defn valid? [x] ...)
(defn empty? [coll] ...)

;; Destructive/side-effecting functions end in !
(defn reset! [atom val] ...)      ; built-in example
(defn save-to-db! [user] ...)     ; your convention

;; Private: use defn- or metadata
(defn- internal-helper [] ...)
(def ^:private my-private-var 42)

;; Dynamic vars (rebindable): *earmuffs*
(def ^:dynamic *debug-mode* false)
(def ^:dynamic *current-user* nil)

;; Constants: just regular defs (no special convention)
(def max-retries 3)

;; Namespace aliases: short and meaningful
(require '[clojure.string :as str])
(require '[clojure.set :as set])
(require '[clojure.java.io :as io])
(require '[clojure.core.async :as async])
```

### Formatting

```clojure
;; Closing parens stay on the last line (same as Emacs Lisp)
;; NOT on their own lines

;; GOOD:
(defn foo [x y]
  (let [result (+ x y)]
    (* result 2)))

;; BAD (C-style):
(defn foo [x y]
  (let [result (+ x y)]
    (* result 2)
  )
)

;; Indent with 2 spaces
;; Align map literals
{:name  "Alice"
 :age   30
 :email "alice@example.com"}

;; Threading macros: each step on its own line
(->> users
     (filter active?)
     (map :email)
     (sort)
     (take 10))

;; Use cljfmt or zprint for automatic formatting
```

### When to Use What

```
Use def for:        global stable values and functions
Use let for:        local bindings within a function
Use atom for:       state that changes over time (single threaded view)
Use ref for:        coordinated changes across multiple values (STM)
Use agent for:      async state updates

Use map for:        transforming every element
Use filter for:     keeping elements matching a predicate
Use reduce for:     aggregating a sequence to a single value
Use for for:        comprehension with multiple bindings
Use doseq for:      side effects over a sequence

Use vector [] for:  ordered sequences, indexed access
Use list () for:    code representation, stack-like use
Use map {} for:     key-value data
Use set #{} for:    unique values, membership testing

Use protocol for:   defining interfaces across types
Use multimethod for: dispatch on arbitrary values
Use record for:     named structured data with known fields

Use ->> for:        sequence pipelines (data flows through last position)
Use -> for:         object-style chains (data in first position)
Use as-> for:       mixed threading
```

### The Functional Style

```clojure
;; AVOID: imperative style
(defn process-orders [orders]
  (let [result (atom [])]
    (doseq [order orders]
      (when (= (:status order) :pending)
        (swap! result conj (process-order order))))
    @result))

;; PREFER: functional style
(defn process-orders [orders]
  (->> orders
       (filter #(= (:status %) :pending))
       (map process-order)))

;; AVOID: nested ifs
(defn categorize [n]
  (if (< n 0)
    :negative
    (if (= n 0)
      :zero
      (if (< n 10)
        :small
        :large))))

;; PREFER: cond
(defn categorize [n]
  (cond
    (< n 0)  :negative
    (= n 0)  :zero
    (< n 10) :small
    :else    :large))
```

---

## Quick Reference Card

### Python → Clojure Cheat Sheet

| Python                    | Clojure                          |
| ------------------------- | -------------------------------- |
| `x = 5`                   | `(def x 5)`                      |
| `x = y = 0`               | `(def x 0) (def y 0)`            |
| local `x = 5`             | `(let [x 5] ...)`                |
| `def f(x): return x*2`    | `(defn f [x] (* x 2))`           |
| `lambda x: x*2`           | `#(* % 2)` or `(fn [x] (* x 2))` |
| `f(x)`                    | `(f x)`                          |
| `f(*args)`                | `(apply f args)`                 |
| `if a: b else: c`         | `(if a b c)`                     |
| `a and b`                 | `(and a b)`                      |
| `a or b`                  | `(or a b)`                       |
| `not a`                   | `(not a)`                        |
| `a == b`                  | `(= a b)`                        |
| `a is b`                  | `(identical? a b)`               |
| `print(x)`                | `(println x)`                    |
| `[1,2,3]`                 | `[1 2 3]`                        |
| `(1,2,3)`                 | `[1 2 3]` (use vector)           |
| `{"a":1}`                 | `{:a 1}`                         |
| `{1,2,3}`                 | `#{1 2 3}`                       |
| `lst[0]`                  | `(first lst)` or `(lst 0)`       |
| `lst[-1]`                 | `(last lst)`                     |
| `lst[1:]`                 | `(rest lst)`                     |
| `lst[a:b]`                | `(subvec lst a b)`               |
| `len(lst)`                | `(count lst)`                    |
| `x in lst`                | `(some #{x} lst)`                |
| `lst.append(x)`           | `(conj lst x)`                   |
| `d["k"]`                  | `(:k d)` or `(get d :k)`         |
| `d.get("k",v)`            | `(get d :k v)`                   |
| `d["k"] = v`              | `(assoc d :k v)`                 |
| `del d["k"]`              | `(dissoc d :k)`                  |
| `"k" in d`                | `(contains? d :k)`               |
| `map(f, lst)`             | `(map f lst)`                    |
| `filter(f, lst)`          | `(filter f lst)`                 |
| `sum(lst)`                | `(reduce + lst)`                 |
| `sorted(lst)`             | `(sort lst)`                     |
| `zip(a, b)`               | `(map vector a b)`               |
| `enumerate(lst)`          | `(map-indexed vector lst)`       |
| `[f(x) for x in xs]`      | `(for [x xs] (f x))`             |
| `[x for x in xs if p(x)]` | `(filter p xs)`                  |
| `try/except`              | `(try ... (catch ...))`          |
| `try/finally`             | `(try ... (finally ...))`        |
| `raise Exception(m)`      | `(throw (ex-info m {}))`         |
| `import mod`              | `(require '[mod ...])`           |
| `from mod import f`       | `(require '[mod :refer [f]])`    |
| `True`/`False`            | `true`/`false`                   |
| `None`                    | `nil`                            |
| `@dataclass`              | `defrecord`                      |
| `threading.Thread`        | `future`                         |
| `threading.Lock`          | `atom` or `ref`                  |
| `functools.partial`       | `partial`                        |
| `functools.reduce`        | `reduce`                         |
| `itertools.chain`         | `concat`                         |
| `itertools.islice`        | `take` / `drop`                  |
| `itertools.groupby`       | `group-by`                       |

---

## Where to Go From Here

1. **[Clojure.org](https://clojure.org)** — the official site with guides and reference.

2. **[Clojure for the Brave and True](https://www.braveclojure.com)** — the best free book for beginners, written with humor.

3. **[ClojureDocs](https://clojuredocs.org)** — documentation with community examples for every function.

4. **[4Clojure / Exercism](https://exercism.org/tracks/clojure)** — practice problems specifically for Clojure.

5. **[Clojurians Slack](https://clojurians.net)** — the main community chat. The `#beginners` channel is welcoming.

6. **Build something real** — pick a small project: a web scraper, a CLI tool, a small API. The [Clojure Cookbook](https://github.com/clojure-cookbook/clojure-cookbook) has recipes for common tasks.

7. **Learn the REPL workflow properly** — set up CIDER or Calva and force yourself to evaluate code from the editor instead of running scripts. This changes how you think about programming.

8. **Read [The Joy of Clojure](https://www.manning.com/books/the-joy-of-clojure)** — after you've written some code, this book explains the _why_ behind Clojure's design decisions.

9. **Study Rich Hickey's talks** — _Simple Made Easy_, _The Value of Values_, and _Hammock Driven Development_ on YouTube. These explain the philosophy that makes Clojure coherent rather than arbitrary.
