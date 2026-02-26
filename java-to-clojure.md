Reasoning: The user wants a comprehensive guide for transitioning from Java to Clojure. They want to cover:

1. Java 8+ features (lambdas, streams, optional, method references `::`, functional interfaces)
2. Java 11 features (var, new string methods, etc.)
3. Java 17 features (records, sealed classes, pattern matching for instanceof, text blocks)
4. Java 21 features (virtual threads, pattern matching for switch, record patterns, sequenced collections)
5. Java 25 features (value classes, primitive types in generics, etc.)

And how these map to Clojure equivalents.

I'm organizing this guide to start with the fundamental paradigm shift from object-oriented to functional programming, then move through the development environment setup before diving into syntax, data structures, and how each Java feature translates to Clojure idioms.

# From Java to Clojure: A Comprehensive Guide

## Table of Contents

1. [Mental Model Shift](#mental-model-shift)
2. [The Environment](#the-environment)
3. [Basic Syntax](#basic-syntax)
4. [Data Types](#data-types)
5. [Variables and Binding](#variables-and-binding)
6. [Functions and Lambdas](#functions-and-lambdas)
7. [Control Flow](#control-flow)
8. [Collections](#collections)
9. [Streams API to Sequences](#streams-api-to-sequences)
10. [Java 8 Functional Interfaces](#java-8-functional-interfaces)
11. [Optional to Nil Handling](#optional-to-nil-handling)
12. [Java Records to Clojure Records](#java-records-to-clojure-records)
13. [Sealed Classes and Pattern Matching](#sealed-classes-and-pattern-matching)
14. [Text Blocks and Strings](#text-blocks-and-strings)
15. [Concurrency](#concurrency)
16. [Object-Oriented to Protocols](#object-oriented-to-protocols)
17. [Error Handling](#error-handling)
18. [Modules and Namespaces](#modules-and-namespaces)
19. [Java Interop](#java-interop)
20. [Testing](#testing)
21. [Build Tools](#build-tools)
22. [Common Patterns and Idioms](#common-patterns-and-idioms)
23. [Java Version Feature Mapping](#java-version-feature-mapping)
24. [Style and Conventions](#style-and-conventions)

---

## 1. Mental Model Shift

This is the most important section. Before writing a single line of Clojure,
you need to understand how it differs fundamentally from Java — not just
syntactically, but philosophically.

### Java thinks in objects and mutation. Clojure thinks in values and transformation.

In Java, your program is a graph of objects that collaborate by mutating each
other's state. Objects have identity, lifecycle, and encapsulate behavior with
data. In Clojure, your program is a series of **transformations on immutable
values**. Data is open, functions are first-class, and state change is explicit
and controlled.

```java
// Java - mutation is the default
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.set(0, "Charlie");  // mutates
// names is now ["Charlie", "Bob"]
```

```clojure
;; Clojure - values never change
(def names ["Alice" "Bob"])
(conj names "Charlie")          ; → ["Alice" "Bob" "Charlie"]  (new vector)
(assoc names 0 "Charlie")       ; → ["Charlie" "Bob"]          (new vector)
;; names is still ["Alice" "Bob"]
```

### The Four Pillars of the Transition

**1. Immutability by default** — All data structures are immutable and
persistent (structurally shared). This eliminates entire categories of bugs
that plague Java codebases: race conditions, defensive copying, unexpected
aliasing, and complex object lifecycle management.

**2. Functions over objects** — Instead of `object.method()`, you write
`(function object)`. Behavior and data are separate. A map is just a map —
it doesn't know how to serialize itself or compare itself. Functions that
operate on maps are defined elsewhere.

**3. The sequence abstraction** — Clojure's equivalent of Java's Streams API,
but it permeates everything. All collections implement the same seq interface.
Every transformation — `map`, `filter`, `reduce` — works on everything.

**4. Lisp's code-as-data** — Code is data. A function call `(+ 1 2)` is
literally a list. This enables macros: compile-time code transformations
that let you extend the language itself.

### What You're Leaving Behind (and Why)

| Java concept                    | Why Clojure doesn't need it                |
| ------------------------------- | ------------------------------------------ |
| `class` as unit of organization | Namespaces + functions replace it          |
| `private`/`public`/`protected`  | Namespace-level visibility is sufficient   |
| Getters and setters             | Data is open; use maps directly            |
| `null` checks everywhere        | Nil-safe idioms and `some->` threading     |
| `synchronized`, `volatile`      | STM, atoms, agents handle concurrency      |
| Interface boilerplate           | Protocols are concise and retroactive      |
| Abstract classes                | Protocols + records replace them cleanly   |
| Inheritance hierarchies         | Composition and polymorphism via protocols |
| `instanceof` chains             | Multimethods dispatch on anything          |
| Checked exceptions              | Values-based error handling                |

### What Java Has Gotten More Clojure-Like

Java has been adopting functional programming patterns since Java 8. Each
major version brings Java closer to what Clojure has always been:

```
Java 8  (2014): Lambdas, Streams, Optional, method references
Java 11 (2018): var, improved APIs
Java 14 (2020): Records (preview)
Java 16 (2021): Records (final), instanceof pattern matching
Java 17 (2021): Sealed classes
Java 21 (2023): Virtual threads, pattern matching for switch, record patterns
Java 25 (2025): Value classes (Valhalla), primitive generics, simplified modules
```

This guide maps all of these Java features to their Clojure counterparts —
because in Clojure, these features aren't additions, they're the foundation.

---

## 2. The Environment

### Installation

```bash
# Install Clojure CLI tools
# macOS
brew install clojure/tools/clojure

# Linux
curl -L -O https://github.com/clojure/brew-install/releases/latest/download/linux-install.sh
chmod +x linux-install.sh
sudo ./linux-install.sh

# Windows
winget install ClojureTools

# Verify
clj --version

# Install Leiningen (alternative, also widely used)
# macOS/Linux
brew install leiningen

# Windows
# Download lein.bat from leiningen.org
```

### Project Structure

```
myproject/                       # like a Maven/Gradle project
├── deps.edn                     # like pom.xml or build.gradle
├── src/
│   └── myproject/
│       └── core.clj             # main namespace
├── test/
│   └── myproject/
│       └── core_test.clj
└── resources/                   # like src/main/resources
    └── config.edn
```

```clojure
;; deps.edn (like pom.xml but readable)
{:paths ["src" "resources"]
 :deps  {org.clojure/clojure {:mvn/version "1.11.1"}}

 :aliases
 {:dev   {:extra-deps  {nrepl/nrepl {:mvn/version "1.0.0"}}
          :extra-paths ["dev"]}

  :test  {:extra-paths ["test"]
          :extra-deps  {io.github.cognitect-labs/test-runner
                        {:git/tag "v0.5.1"
                         :git/sha "dfb30dd"}}}

  :build {:deps       {io.github.clojure/tools.build
                       {:git/tag "v0.9.4"
                        :git/sha "76b78fe"}}
          :ns-default build}}}
```

Java equivalent:

```xml
<!-- pom.xml -->
<project>
  <groupId>com.example</groupId>
  <artifactId>myproject</artifactId>
  <version>1.0.0</version>
  <dependencies>
    <dependency>
      <groupId>org.clojure</groupId>
      <artifactId>clojure</artifactId>
      <version>1.11.1</version>
    </dependency>
  </dependencies>
</project>
```

### The REPL: Your Primary Development Tool

This is the biggest workflow change from Java. In Java you write code,
compile, run, see output, fix, repeat. In Clojure you connect a REPL
to your running application and evaluate code interactively:

```bash
# Start a REPL
clj

# REPL session:
user=> (+ 1 2 3)
6
user=> (def greeting "Hello, World!")
#'user/greeting
user=> greeting
"Hello, World!"
user=> (map inc [1 2 3 4 5])
(2 3 4 5 6)
```

In practice, you use an editor integration (CIDER for Emacs, Calva for
VS Code, Cursive for IntelliJ) that lets you evaluate code directly from
the editor while the application is running. You never restart. You
never wait for compilation. Changes are live.

### Editor Setup

- **IntelliJ + Cursive** — most familiar to Java developers, full IDE
- **VS Code + Calva** — easiest to set up, very good for beginners
- **Emacs + CIDER** — the gold standard, most powerful REPL integration
- **Neovim + Conjure** — for vim users

---

## 3. Basic Syntax

### The Fundamental Difference: Prefix Notation

Every Java operation has special syntax. Clojure has one syntax for
everything: `(function arg1 arg2 ...)`.

```java
// Java - different syntax for different constructs
int result = add(1, 2);       // method call
int x = 5;                    // assignment
if (x > 3) { ... }            // statement
for (int i = 0; i < 5; i++){ }// loop
x + y + z;                    // infix operator
```

```clojure
;; Clojure - everything is a list starting with the function/operator
(add 1 2)                     ; function call
(def x 5)                     ; var definition
(if (> x 3) ...)              ; expression
(dotimes [i 5] ...)           ; loop
(+ x y z)                     ; operator (just a function)
```

### Operators Are Functions

In Java operators are special syntax. In Clojure they are regular functions
that can take any number of arguments:

```java
// Java
1 + 2 + 3 + 4 + 5      // = 15
x > 0 && y < 10         // boolean
!isValid                 // negate
```

```clojure
;; Clojure
(+ 1 2 3 4 5)           ; → 15
(and (> x 0) (< y 10))  ; boolean
(not is-valid)           ; negate

;; No precedence to remember
;; Java: 2 + 3 * 4 = 14 (must know precedence)
;; Clojure:
(+ 2 (* 3 4))   ; = 14, obviously
(* (+ 2 3) 4)   ; = 20, obviously
```

### Literals and Reader Syntax

```clojure
;; Numbers
42          ; Long
42N         ; BigInteger
3.14        ; Double
3.14M       ; BigDecimal (exact, like Java's BigDecimal)
22/7        ; Ratio — exact fraction, no floating point error!

;; Strings
"Hello, World!"
"multi\nline"

;; Characters (like Java's char literals)
\a          ; 'a' in Java
\A          ; 'A' in Java
\newline    ; '\n' in Java
\space      ; ' ' in Java

;; Booleans
true
false

;; Null
nil         ; Java's null

;; Keywords (no Java equivalent — like interned strings used as keys)
:name
:user/name  ; namespaced keyword
::name      ; auto-namespace to current ns

;; Symbols (refer to vars)
my-var      ; refers to a variable

;; Collections
[1 2 3]         ; vector   — like ArrayList but immutable
'(1 2 3)        ; list     — linked list
{:a 1 :b 2}     ; map      — like HashMap but immutable
#{1 2 3}        ; set      — like HashSet but immutable
```

### Comments

```clojure
; Single line comment (convention: 1 semicolon for inline)

;; Double semicolon for top-level comments (most common)

;;; Triple semicolon for section headers

#_ (this entire form is commented out
    regardless of how many lines it spans)

;; Rich comment block — keep REPL experiments in the file
;; None of this code is executed by the application
(comment
  (start-server! {:port 3000})
  (run-tests)
  (->> (get-users db)
       (filter active?)
       (map :email)))
```

---

## 4. Data Types

### Primitive Types

```java
// Java
int    x = 42;
long   y = 9999999999L;
double z = 3.14;
float  f = 3.14f;
char   c = 'A';
boolean b = true;
byte   by = 127;
short  s = 32767;
```

```clojure
;; Clojure - unified numeric tower
42          ; Long by default (Java's long)
42N         ; BigInteger (arbitrary precision)
3.14        ; Double
3.14M       ; BigDecimal
(float 3.14); Float (when you need float precision)
\A          ; Character
true        ; Boolean

;; Type hints for Java interop performance
(let [^long x 42
      ^double y 3.14]
  (* x y))

;; Checking types
(integer? 42)    ; → true
(float? 3.14)    ; → true
(string? "hi")   ; → true
(keyword? :foo)  ; → true
(nil? nil)       ; → true
```

### No Primitive/Reference Distinction

Java's primitive vs wrapper type distinction (`int` vs `Integer`,
`long` vs `Long`) is a constant source of bugs and boilerplate:

```java
// Java - painful
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 100);  // autoboxing: int → Integer
int score = scores.get("Alice");  // unboxing: Integer → int
// NullPointerException if key missing!
Integer val = scores.get("Bob");  // returns null, not 0
```

```clojure
;; Clojure - no boxing/unboxing, no NPE
(def scores {"Alice" 100})
(get scores "Alice")    ; → 100
(get scores "Bob")      ; → nil (no exception)
(get scores "Bob" 0)    ; → 0 (with default)
```

### The Ratio Type

Clojure has exact fractions — no floating point errors:

```java
// Java
System.out.println(0.1 + 0.2);      // → 0.30000000000000004
System.out.println(1.0/3 + 2.0/3);  // → 1.0 (lucky)
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
System.out.println(a.add(b));         // → 0.3 (needs explicit BigDecimal)
```

```clojure
;; Clojure
(+ 0.1 0.2)       ; → 0.30000000000000004 (doubles, same as Java)
(+ 1/10 2/10)     ; → 3/10  (exact ratio!)
(+ 1/3 2/3)       ; → 1     (exact!)
(* 3 1/3)         ; → 1     (exact!)
(double 1/3)      ; → 0.3333... (convert to double when needed)
```

---

## 5. Variables and Binding

### Vars: Global Definitions

```java
// Java
public static final int MAX_SIZE = 1024;
private String name = "default";
public int counter = 0;
counter++;  // mutation is normal
```

```clojure
;; Clojure
(def max-size 1024)           ; like static final
(def name "default")          ; naming a value

;; Vars are NOT variables in the Java sense
;; Redefining is possible but signals a design problem
(def counter 0)
(def counter 1)  ; possible but bad practice — use atom instead

;; For mutable state, use atom (explicit mutation)
(def counter (atom 0))
(swap! counter inc)    ; increment
@counter               ; read current value → 1
```

### let: Local Bindings

```java
// Java
int x = 10;
int y = x * 2;
int z = x + y;
System.out.println(z);
```

```clojure
;; Clojure - let creates immutable local bindings
(let [x 10
      y (* x 2)    ; can reference x immediately
      z (+ x y)]  ; can reference x and y
  (println z))    ; → 30

;; No reassignment — these are not variables, they are bindings
(let [x 10]
  (let [x 20]    ; shadowing is allowed
    (println x)) ; → 20
  (println x))   ; → 10
```

### The Four Reference Types

When you need mutable state, Clojure gives you explicit, controlled
mutation through reference types. Each has a specific use case:

```java
// Java - everything is mutable, concurrency is your problem
private int counter = 0;
private final List<String> items = new ArrayList<>();
private volatile boolean running = true;
// You have to manually handle thread safety everywhere
```

```clojure
;; Clojure - explicit, controlled mutation

;; atom: uncoordinated, synchronous (most common)
(def counter (atom 0))
(swap! counter inc)           ; apply function atomically
(reset! counter 0)            ; set to specific value
@counter                      ; deref to read

;; ref: coordinated via Software Transactional Memory
(def account-a (ref 1000))
(def account-b (ref 500))
(dosync                       ; atomic transaction
  (alter account-a - 100)
  (alter account-b + 100))

;; agent: asynchronous state
(def log-agent (agent []))
(send log-agent conj "event") ; async update

;; var: thread-local dynamic binding
(def ^:dynamic *current-user* nil)
(binding [*current-user* "Alice"]
  (do-something))             ; current-user is "Alice" in this thread
```

---

## 6. Functions and Lambdas

### Java Lambdas → Clojure Functions

Java 8 introduced lambdas as a way to pass behavior as values. In
Clojure, **functions have always been first-class values**:

```java
// Java 8+ lambda
Runnable r = () -> System.out.println("Hello");
Function<Integer, Integer> doubler = x -> x * 2;
BiFunction<Integer, Integer, Integer> adder = (x, y) -> x + y;
Predicate<String> isLong = s -> s.length() > 10;
Consumer<String> printer = s -> System.out.println(s);
Supplier<String> greeting = () -> "Hello!";

// Method reference (Java 8)
Function<String, String> upper = String::toUpperCase;
Function<String, Integer> len = String::length;
Consumer<String> printer2 = System.out::println;
Comparator<String> comp = String::compareTo;
```

```clojure
;; Clojure - functions are values, no special interface needed
(def r #(println "Hello"))
(def doubler #(* % 2))
(def adder #(+ %1 %2))
(def is-long #(> (count %) 10))
(def printer println)          ; functions ARE values, no wrapping
(def greeting #"Hello!")

;; Method references → just refer to the function
(def upper clojure.string/upper-case)  ; #'clojure.string/upper-case
(def len count)                         ; count is already a function
(def printer2 println)

;; Call them
(r)                    ; Hello
(doubler 5)            ; → 10
(adder 3 4)            ; → 7
(is-long "hello")      ; → false
(printer "test")       ; test
(greeting)             ; → "Hello!"
```

### The `::` Method Reference vs Clojure's `#'`

Java 8's `::` syntax references methods as functions. Clojure has always
had this but uses `#'` or just the name:

```java
// Java 8 :: method reference
// Instance method reference
Function<String, String> upper = String::toUpperCase;
// means: s -> s.toUpperCase()

// Static method reference
Function<String, Integer> parser = Integer::parseInt;
// means: s -> Integer.parseInt(s)

// Constructor reference
Supplier<ArrayList<String>> listFactory = ArrayList::new;
// means: () -> new ArrayList<>()

// Bound instance method reference
String prefix = "Hello, ";
Function<String, String> greeter = prefix::concat;
// means: s -> prefix.concat(s)
```

```clojure
;; Clojure equivalents

;; Instance method → just a function call
(def upper clojure.string/upper-case)
;; Or inline: #(.toUpperCase %)

;; Static method → refer to the function
(def parser #(Integer/parseInt %))
;; Or: (partial Integer/parseInt)

;; Constructor → use the record constructor or (->Type ...)
(def list-factory #(java.util.ArrayList.))
;; Or for Clojure: (fn [] [])

;; Bound instance → partial application
(def greeter (partial str "Hello, "))
(greeter "Alice")   ; → "Hello, Alice"

;; The #' reader macro gives you the Var (like a reference to the function)
#'clojure.string/upper-case    ; the Var, not the value
clojure.string/upper-case       ; the function value itself
```

### Defining Functions

```java
// Java
public static int add(int a, int b) {
    return a + b;
}

public static String greet(String name) {
    return "Hello, " + name + "!";
}

// Overloaded
public static int sum(int... nums) {
    return Arrays.stream(nums).sum();
}
```

```clojure
;; Clojure
(defn add [a b]
  (+ a b))

(defn greet [name]
  (str "Hello, " name "!"))

;; Variadic
(defn sum [& nums]
  (apply + nums))

(sum 1 2 3 4 5)   ; → 15

;; Multiple arities (like overloading)
(defn greet
  ([]      "Hello, World!")
  ([name]  (str "Hello, " name "!"))
  ([name title] (str "Hello, " title " " name "!")))

(greet)              ; → "Hello, World!"
(greet "Alice")      ; → "Hello, Alice!"
(greet "Smith" "Dr") ; → "Hello, Dr Smith!"
```

### Higher-Order Functions

```java
// Java 8+
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

// map
numbers.stream()
       .map(x -> x * x)
       .collect(Collectors.toList());

// filter
numbers.stream()
       .filter(x -> x % 2 == 0)
       .collect(Collectors.toList());

// reduce
numbers.stream()
       .reduce(0, Integer::sum);

// forEach (consumer)
numbers.forEach(System.out::println);
```

```clojure
;; Clojure
(def numbers [1 2 3 4 5])

;; map
(map #(* % %) numbers)          ; → (1 4 9 16 25)

;; filter
(filter even? numbers)          ; → (2 4)

;; reduce
(reduce + 0 numbers)            ; → 15
(reduce + numbers)              ; → 15

;; for each (side effects)
(doseq [n numbers]
  (println n))

;; comp: function composition (like Java's Function.compose/andThen)
(def inc-then-double (comp #(* % 2) inc))
(inc-then-double 3)   ; → 8

;; partial: partial application (no direct Java equivalent)
(def add5 (partial + 5))
(add5 3)   ; → 8
```

### Threading Macros: Replacing Method Chaining

Java's method chaining and Stream API pipelines are replaced by
threading macros which are often more readable:

```java
// Java Stream pipeline
List<String> result = employees.stream()
    .filter(e -> e.getDepartment().equals("Engineering"))
    .map(Employee::getName)
    .sorted()
    .limit(10)
    .collect(Collectors.toList());
```

```clojure
;; Clojure with ->> (thread-last)
(def result
  (->> employees
       (filter #(= (:department %) "Engineering"))
       (map :name)
       (sort)
       (take 10)
       (vec)))

;; ->> inserts the previous result as the LAST argument
;; So (filter pred coll) receives employees as coll

;; -> (thread-first) for object-style chaining
(-> "hello world"
    clojure.string/upper-case     ; "HELLO WORLD"
    (clojure.string/replace " " "-")  ; "HELLO-WORLD"
    clojure.string/reverse)       ; "DLROW-OLLEH"

;; Mixing with as->
(as-> employees x
  (filter #(= (:dept %) "Eng") x)
  (group-by :level x)
  (map (fn [[level emps]] [level (count emps)]) x))
```

---

## 7. Control Flow

### If and Conditionals

```java
// Java
if (x > 0) {
    return "positive";
} else if (x < 0) {
    return "negative";
} else {
    return "zero";
}

// Ternary (expression form)
String label = x > 0 ? "positive" : "non-positive";
```

```clojure
;; Clojure - if is an EXPRESSION, not a statement
(if (> x 0)
  "positive"
  "non-positive")

;; cond for if/else if/else (like Java's switch expression in Java 14+)
(cond
  (> x 0) "positive"
  (< x 0) "negative"
  :else   "zero")

;; when: if without else (returns nil if false)
(when (> x 0)
  (log "positive")
  (* x 2))

;; if-let: bind and test in one
(if-let [user (find-user id)]
  (str "Found: " (:name user))
  "User not found")

;; Both branches required for if
;; Use when/when-not for single-branch conditions
(when-not (empty? items)
  (process items))
```

### Java Switch → Clojure case/cond

```java
// Java switch statement (classic)
switch (day) {
    case MONDAY:
        return "Start of week";
    case FRIDAY:
        return "End of week";
    case SATURDAY: case SUNDAY:
        return "Weekend";
    default:
        return "Midweek";
}

// Java 14+ switch expression
String label = switch (day) {
    case MONDAY -> "Start of week";
    case FRIDAY -> "End of week";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};

// Java 21 pattern matching switch
String result = switch (obj) {
    case Integer i when i > 0 -> "Positive int: " + i;
    case Integer i            -> "Non-positive int: " + i;
    case String s             -> "String: " + s;
    case null                 -> "null";
    default                   -> "Other: " + obj;
};
```

```clojure
;; Clojure case (for constant dispatch)
(case day
  :monday  "Start of week"
  :friday  "End of week"
  (:saturday :sunday) "Weekend"  ; multiple values
  "Midweek")                     ; default

;; cond for pattern-like matching
(cond
  (and (integer? obj) (pos? obj)) (str "Positive int: " obj)
  (integer? obj)                  (str "Non-positive int: " obj)
  (string? obj)                   (str "String: " obj)
  (nil? obj)                      "null"
  :else                           (str "Other: " obj))

;; core.match for Java 21-style pattern matching
(require '[clojure.core.match :refer [match]])

(match obj
  (x :guard integer? :guard pos?)  (str "Positive int: " x)
  (x :guard integer?)              (str "Non-positive int: " x)
  (x :guard string?)               (str "String: " x)
  nil                              "null"
  x                                (str "Other: " x))
```

### Loops

```java
// Java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

for (String item : list) {
    System.out.println(item);
}

while (condition) {
    doSomething();
}

// Java 8+
list.forEach(System.out::println);
IntStream.range(0, 10).forEach(System.out::println);
```

```clojure
;; Clojure

;; dotimes: n iterations
(dotimes [i 10]
  (println i))

;; doseq: for each (side effects)
(doseq [item list]
  (println item))

;; while: explicit while loop
(while condition
  (do-something))

;; Functional: use sequence operations (preferred)
(doseq [i (range 10)] (println i))
(run! println list)   ; like forEach

;; loop/recur: explicit recursion (Clojure's answer to imperative loops)
(loop [i 0
       acc 0]
  (if (>= i 10)
    acc
    (recur (inc i) (+ acc i))))
; → 45  (sum of 0..9)

;; Most "loops" in Clojure are actually sequence operations:
(reduce + (range 10))   ; → 45
```

---

## 8. Collections

### Java Collections Hierarchy → Clojure's Four Types

```java
// Java - complex hierarchy
List<String> list = new ArrayList<>();      // mutable
List<String> linked = new LinkedList<>();   // mutable
Map<String,Integer> map = new HashMap<>();  // mutable
Set<String> set = new HashSet<>();          // mutable
Deque<String> deque = new ArrayDeque<>();

// Java 9+ immutable factory methods
List<String> immutable = List.of("a", "b", "c");
Map<String,Integer> immutableMap = Map.of("a", 1, "b", 2);
Set<String> immutableSet = Set.of("a", "b", "c");
```

```clojure
;; Clojure - four core types, all immutable by default
["a" "b" "c"]           ; vector   (indexed access, O(log32 n))
'("a" "b" "c")          ; list     (prepend, stack)
{:a 1 :b 2}             ; map      (hash map)
#{"a" "b" "c"}          ; set      (hash set)

;; No mutable versions needed — you produce new collections
;; Structural sharing makes this efficient
```

### Vectors (replacing ArrayList)

```java
// Java ArrayList
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
list.get(0);            // 1
list.get(list.size()-1);// 5
list.add(6);            // mutates
list.set(2, 99);        // mutates
list.remove(0);         // mutates
list.size();            // 5 (after remove)
list.contains(3);       // true
list.indexOf(3);        // index
Collections.sort(list); // mutates
```

```clojure
;; Clojure vector
(def v [1 2 3 4 5])
(v 0)           ; → 1    (vector is a function of its index)
(get v 0)       ; → 1    (safer, returns nil for out-of-range)
(first v)       ; → 1
(last v)        ; → 5
(conj v 6)      ; → [1 2 3 4 5 6]  (new vector)
(assoc v 2 99)  ; → [1 2 99 4 5]   (new vector)
(subvec v 1 4)  ; → [2 3 4]
(count v)       ; → 5
(contains? v 2) ; → true (checks index, not value — use some for value)
(some #{3} v)   ; → 3 (truthy) or nil if not found
(sort v)        ; → (1 2 3 4 5) (new sorted seq)
(v 0)           ; → 1 (direct index access is O(log32 n) ≈ O(1))
(get v 10)      ; → nil (no exception like Java's IndexOutOfBoundsException)
```

### Maps (replacing HashMap)

```java
// Java HashMap
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 100);
scores.put("Bob", 85);
scores.get("Alice");            // 100
scores.getOrDefault("Dave", 0); // 0
scores.containsKey("Alice");    // true
scores.remove("Bob");
scores.size();
for (Map.Entry<String,Integer> e : scores.entrySet()) {
    System.out.println(e.getKey() + ": " + e.getValue());
}

// Java 9+ Map.of
Map<String,Integer> immutable = Map.of("Alice", 100, "Bob", 85);

// Java 8 compute methods
scores.compute("Alice", (k, v) -> v == null ? 1 : v + 1);
scores.merge("Alice", 1, Integer::sum);
```

```clojure
;; Clojure map
(def scores {"Alice" 100 "Bob" 85})
;; or with keyword keys (more common):
(def scores {:alice 100 :bob 85})

;; Access
(scores :alice)          ; → 100  (map as function)
(:alice scores)          ; → 100  (keyword as function — idiomatic)
(get scores :alice)      ; → 100  (explicit get)
(get scores :dave 0)     ; → 0    (with default)

;; "Modification" (returns new map)
(assoc scores :charlie 92)   ; → new map with :charlie
(dissoc scores :bob)          ; → new map without :bob
(merge scores {:dave 78 :eve 95}) ; → merged map

;; Query
(contains? scores :alice)    ; → true
(keys scores)                ; → (:alice :bob)
(vals scores)                ; → (100 85)
(count scores)               ; → 2

;; Iteration
(doseq [[k v] scores]
  (println (name k) ":" v))

;; update: apply function to a value
(update scores :alice + 5)        ; → {:alice 105 :bob 85}
(update scores :alice inc)        ; → {:alice 101 :bob 85}

;; assoc-in, get-in, update-in: nested operations
(def data {:user {:name "Alice" :scores [10 20 30]}})
(get-in data [:user :name])              ; → "Alice"
(get-in data [:user :scores 1])          ; → 20
(assoc-in data [:user :name] "Bob")      ; → nested update
(update-in data [:user :scores] conj 40) ; → append to nested vec
```

### Sets (replacing HashSet)

```java
// Java HashSet
Set<String> set = new HashSet<>(Arrays.asList("a", "b", "c"));
set.add("d");
set.remove("a");
set.contains("b");     // true
set.size();
// Set operations
Set<String> union = new HashSet<>(set1);
union.addAll(set2);
Set<String> intersection = new HashSet<>(set1);
intersection.retainAll(set2);
```

```clojure
;; Clojure set
(def s #{:a :b :c})

;; "Modification"
(conj s :d)          ; → #{:a :b :c :d}
(disj s :a)          ; → #{:b :c}

;; Query
(contains? s :b)     ; → true
(s :b)               ; → :b  (set as function, returns element or nil)
(count s)            ; → 3

;; Set operations (clojure.set)
(require '[clojure.set :as set])
(set/union #{1 2 3} #{3 4 5})        ; → #{1 2 3 4 5}
(set/intersection #{1 2 3} #{2 3 4}) ; → #{2 3}
(set/difference #{1 2 3} #{2 3})     ; → #{1}
(set/subset? #{1 2} #{1 2 3})        ; → true

;; Build from other collection (deduplicates)
(set [1 2 2 3 3 3])   ; → #{1 2 3}
```

---

## 9. Streams API to Sequences

Java 8's Stream API was a huge improvement. Clojure's sequence operations
are richer, more composable, and work on all collections natively.

### Mapping Between Stream and Seq Operations

```java
// Java 8+ Stream
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// map
numbers.stream().map(x -> x * x).collect(toList())

// filter
numbers.stream().filter(x -> x % 2 == 0).collect(toList())

// reduce
numbers.stream().reduce(0, Integer::sum)

// flatMap
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));
nested.stream().flatMap(Collection::stream).collect(toList())

// distinct
numbers.stream().distinct().collect(toList())

// sorted
numbers.stream().sorted().collect(toList())
numbers.stream().sorted(Comparator.reverseOrder()).collect(toList())

// limit/skip
numbers.stream().limit(5).collect(toList())
numbers.stream().skip(3).collect(toList())

// count
numbers.stream().filter(even).count()

// anyMatch/allMatch/noneMatch
numbers.stream().anyMatch(x -> x > 5)
numbers.stream().allMatch(x -> x > 0)
numbers.stream().noneMatch(x -> x > 100)

// findFirst
numbers.stream().filter(even).findFirst()

// min/max
numbers.stream().max(Integer::compareTo)
numbers.stream().min(Integer::compareTo)

// groupingBy
employees.stream().collect(Collectors.groupingBy(Employee::getDept))

// toMap
employees.stream().collect(Collectors.toMap(
    Employee::getId, e -> e))

// joining
names.stream().collect(Collectors.joining(", ", "[", "]"))

// Collectors.counting
employees.stream().collect(
    Collectors.groupingBy(Employee::getDept, Collectors.counting()))

// Parallel streams
numbers.parallelStream().map(heavyComputation).collect(toList())
```

```clojure
;; Clojure sequences
(def numbers (range 1 11))

;; map
(map #(* % %) numbers)          ; → (1 4 9 16 25 36 49 64 81 100)

;; filter
(filter even? numbers)          ; → (2 4 6 8 10)

;; reduce
(reduce + 0 numbers)            ; → 55

;; mapcat (flatMap)
(mapcat identity [[1 2] [3 4]]) ; → (1 2 3 4)

;; distinct
(distinct [1 2 2 3 3 3])        ; → (1 2 3)

;; sort
(sort numbers)                  ; → (1 2 3 ... 10)
(sort > numbers)                ; → (10 9 8 ... 1)  (reverse)
(sort-by :age employees)        ; → sorted by :age field

;; take/drop (limit/skip)
(take 5 numbers)                ; → (1 2 3 4 5)
(drop 3 numbers)                ; → (4 5 6 7 8 9 10)

;; count after filter
(count (filter even? numbers))  ; → 5

;; any?/every?/not-any?
(some even? numbers)            ; → 2  (first truthy value, like anyMatch)
(every? pos? numbers)           ; → true
(not-any? neg? numbers)         ; → true

;; first after filter (findFirst)
(first (filter even? numbers))  ; → 2

;; min/max
(apply max numbers)             ; → 10
(apply min numbers)             ; → 1
(reduce max numbers)            ; → 10

;; group-by (groupingBy)
(group-by :dept employees)      ; → {"Eng" [...], "Sales" [...]}

;; index-by (toMap)
(into {} (map (juxt :id identity) employees))
;; or:
(reduce #(assoc %1 (:id %2) %2) {} employees)

;; join strings
(clojure.string/join ", " names)        ; → "Alice, Bob, Charlie"
(str "[" (clojure.string/join ", " names) "]")

;; frequencies (Collectors.counting after groupBy)
(frequencies (map :dept employees))     ; → {"Eng" 5, "Sales" 3}

;; Parallel: pmap (parallel map)
(pmap heavy-computation numbers)        ; runs in parallel
```

### Lazy Sequences: Going Beyond Streams

Clojure's sequences are lazy by default. Unlike Java Streams (which
are one-shot), Clojure lazy seqs can be stored, shared, and consumed
multiple times:

```java
// Java Stream - one-shot, cannot be reused
Stream<Integer> s = numbers.stream().filter(x -> x > 5);
s.forEach(System.out::println);
s.forEach(System.out::println); // throws IllegalStateException!
```

```clojure
;; Clojure lazy seq - reusable
(def filtered (filter #(> % 5) numbers))
(println filtered)   ; prints (6 7 8 9 10)
(println filtered)   ; prints (6 7 8 9 10) — works again!
(count filtered)     ; → 5
(first filtered)     ; → 6

;; Infinite sequences
(def naturals (iterate inc 0))            ; 0, 1, 2, 3 ... forever
(def fibs (map first (iterate (fn [[a b]] [b (+ a b)]) [0 1])))

(take 10 naturals)   ; → (0 1 2 3 4 5 6 7 8 9)
(take 10 fibs)       ; → (0 1 1 2 3 5 8 13 21 34)
```

---

## 10. Java 8 Functional Interfaces

Java 8 introduced functional interfaces to enable lambdas. Clojure
has always had first-class functions, so there are no interfaces needed:

```java
// Java 8 functional interfaces and their usage
import java.util.function.*;

// Function<T,R>: T → R
Function<String, Integer> length = String::length;
Function<String, String> upper = String::toUpperCase;
Function<String, String> greet = upper.andThen(s -> "Hello, " + s);

// BiFunction<T,U,R>: T,U → R
BiFunction<Integer, Integer, Integer> add = Integer::sum;

// Predicate<T>: T → boolean
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
Predicate<String> longAndNotEmpty = isEmpty.negate()
    .and(s -> s.length() > 5);

// Consumer<T>: T → void
Consumer<String> print = System.out::println;
Consumer<String> printUpper = print.compose(String::toUpperCase);

// Supplier<T>: () → T
Supplier<List<String>> listFactory = ArrayList::new;
Supplier<String> greeting = () -> "Hello, World!";

// UnaryOperator<T>: T → T
UnaryOperator<Integer> increment = x -> x + 1;
UnaryOperator<String> shout = s -> s.toUpperCase() + "!";

// BinaryOperator<T>: T,T → T
BinaryOperator<Integer> sum = Integer::sum;
BinaryOperator<String> concat = String::concat;

// Comparator<T>
Comparator<String> byLength = Comparator.comparingInt(String::length);
Comparator<String> byLengthThenAlpha = byLength.thenComparing(Comparator.naturalOrder());
```

```clojure
;; Clojure - no wrapper interfaces, just functions

;; Function<String, Integer>
(def length count)
(def upper clojure.string/upper-case)
;; andThen = comp (note reversed order)
(def greet (comp #(str "Hello, " %) clojure.string/upper-case))

;; BiFunction<Integer, Integer, Integer>
(def add +)

;; Predicate<String>
(def is-empty empty?)
(def is-not-empty (complement empty?))  ; .negate()
(def long-and-not-empty
  (every-pred (complement empty?) #(> (count %) 5)))  ; .and()

;; Consumer<String>
(def print-val println)
;; compose in consumer direction
(defn print-upper [s] (println (clojure.string/upper-case s)))

;; Supplier<T>
(def list-factory #(vector))
(def greeting #"Hello, World!")

;; UnaryOperator<T>
(def increment inc)
(def shout #(str (clojure.string/upper-case %) "!"))

;; BinaryOperator<T>
(def sum +)
(def concat-str str)

;; Comparator
(def by-length #(compare (count %1) (count %2)))
(sort by-length ["hello" "hi" "hey" "howdy"])

;; Built-in comparator utilities
(sort-by count ["hello" "hi" "hey"])          ; sort by length
(sort-by (juxt count identity) ["b" "aa" "a"]) ; sort by length then alpha

;; comp: function composition (like andThen but in math order)
(def inc-then-double (comp #(* % 2) inc))   ; first inc, then double
(inc-then-double 3)   ; → 8

;; Every-pred: like Predicate.and()
(def valid-user?
  (every-pred map?
              #(contains? % :name)
              #(pos? (:age %))))

;; Some-fn: like Predicate.or()
(def number-or-string?
  (some-fn number? string?))
```

### Java 8 Collectors → Clojure into/reduce

```java
// Java Collectors
stream.collect(Collectors.toList())
stream.collect(Collectors.toSet())
stream.collect(Collectors.toMap(k, v))
stream.collect(Collectors.joining(", "))
stream.collect(Collectors.groupingBy(f))
stream.collect(Collectors.counting())
stream.collect(Collectors.summingInt(f))
stream.collect(Collectors.partitioningBy(pred))
stream.collect(Collectors.toUnmodifiableList())
```

```clojure
;; Clojure equivalents
(into [] seq)                     ; toList
(into #{} seq)                    ; toSet
(into {} (map (juxt k v) seq))    ; toMap
(clojure.string/join ", " seq)    ; joining
(group-by f seq)                  ; groupingBy
(count seq)                       ; counting
(reduce + (map f seq))            ; summingInt
(group-by pred seq)               ; partitioningBy (true/false groups)
(vec seq)                         ; toUnmodifiableList (vecs are immutable)

;; More idiomatic:
(transduce
  (comp (filter active?) (map :name))
  conj
  []
  employees)   ; one-pass transformation, no intermediate collections
```

---

## 11. Optional to Nil Handling

Java 8's `Optional` was introduced to avoid NullPointerException. Clojure
handles this more naturally with nil and its rich nil-safe idioms:

```java
// Java Optional
Optional<User> user = findUser(id);
user.isPresent();
user.get();                              // throws if empty
user.orElse(defaultUser);
user.orElseGet(() -> createUser(id));
user.orElseThrow(() -> new NoSuchElementException());
user.map(User::getName);
user.flatMap(u -> findAddress(u.getId()));
user.filter(u -> u.isActive());
user.ifPresent(u -> System.out.println(u.getName()));

// Java 9+ Optional
user.ifPresentOrElse(
    u -> System.out.println(u.getName()),
    () -> System.out.println("No user"));
user.or(() -> Optional.of(defaultUser));
user.stream(); // Optional as Stream
```

```clojure
;; Clojure nil handling - simpler and pervasive

;; Functions return nil for missing values
(def user (find-user id))  ; returns nil if not found

;; nil is falsy, so if/when work naturally
(if user
  (:name user)
  default-user)

(when user
  (println (:name user)))

;; some->: nil-safe threading (like Optional.map chain)
(some-> user
        :name
        clojure.string/upper-case)
;; If user is nil, stops and returns nil
;; If :name is nil, stops and returns nil
;; Otherwise returns the uppercased name

;; some->>: nil-safe thread-last
(some->> users
          (filter active?)
          first
          :email)

;; or: like orElse
(or user default-user)
(or (:name user) "Anonymous")

;; if-some: bind only if non-nil
(if-some [name (:name user)]
  (println name)
  (println "No name"))

;; when-some
(when-some [name (:name user)]
  (process name))

;; fnil: wrap function to handle nil with default
(def safe-inc (fnil inc 0))
(safe-inc nil)   ; → 1 (treats nil as 0)
(safe-inc 5)     ; → 6

;; get with default (like orElse)
(get user :name "Anonymous")

;; Using clojure.core Option-like patterns
(require '[clojure.core :refer [some? nil?]])
(some? user)   ; → true if not nil
(nil? user)    ; → true if nil
```

---

## 12. Java Records to Clojure Records

Java 14-16 introduced Records as immutable data carriers. Clojure has
always had records, and idiomatic Clojure often just uses plain maps:

```java
// Java 16+ Record
record Point(double x, double y) {
    // Compact canonical constructor
    Point {
        if (x < 0 || y < 0) throw new IllegalArgumentException("Negative");
    }

    // Custom methods
    double distanceTo(Point other) {
        return Math.sqrt(
            Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
    }

    static Point origin() { return new Point(0, 0); }
}

// Java record features:
// - Immutable fields
// - Auto-generated constructor, getters, equals, hashCode, toString
// - Can implement interfaces
// - Can have compact constructors
// - Can have static fields

record User(String name, int age, String email) implements Comparable<User> {
    @Override
    public int compareTo(User other) {
        return this.name.compareTo(other.name);
    }
}

Point p = new Point(3.0, 4.0);
p.x();          // 3.0
p.y();          // 4.0
p.toString();   // "Point[x=3.0, y=4.0]"
```

```clojure
;; Clojure defrecord
(defrecord Point [x y])

;; Construction
(->Point 3.0 4.0)              ; positional
(map->Point {:x 3.0 :y 4.0})  ; from map

;; Access
(def p (->Point 3.0 4.0))
(:x p)     ; → 3.0
(:y p)     ; → 4.0

;; Records implement map interface
(keys p)   ; → (:x :y)
(assoc p :z 0.0)  ; → new Point? No — assoc on record returns map

;; Custom methods via protocols
(defprotocol Geometric
  (distance-to [this other])
  (origin [_]))

(defrecord Point [x y]
  Geometric
  (distance-to [this other]
    (Math/sqrt (+ (Math/pow (- (:x this) (:x other)) 2)
                  (Math/pow (- (:y this) (:y other)) 2))))
  (origin [_]
    (->Point 0.0 0.0)))

;; Validation in factory function
(defn make-point [x y]
  (when (or (neg? x) (neg? y))
    (throw (ex-info "Negative coordinates" {:x x :y y})))
  (->Point x y))

;; Idiomatic alternative: just use plain maps!
;; For simple data, maps are often better than records
(def p {:x 3.0 :y 4.0})
(:x p)   ; → 3.0

;; Clojure records vs maps:
;; Records: fixed fields, faster field access, participate in protocols
;; Maps: flexible, easy to evolve, no type definition needed
```

---

## 13. Sealed Classes and Pattern Matching

Java 17 introduced sealed classes and Java 21 completed pattern matching
for switch. Clojure handles these via protocols, multimethods, and
`core.match`:

```java
// Java 17 Sealed Classes
public sealed interface Shape
    permits Circle, Rectangle, Triangle {}

public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height) implements Shape {}

// Java 21 Pattern Matching for Switch
double area(Shape shape) {
    return switch (shape) {
        case Circle c           -> Math.PI * c.radius() * c.radius();
        case Rectangle r        -> r.width() * r.height();
        case Triangle t         -> 0.5 * t.base() * t.height();
    }; // exhaustive — compiler checks all cases!
}

// Java 21 Guarded patterns
String describe(Object obj) {
    return switch (obj) {
        case Integer i when i < 0  -> "negative integer: " + i;
        case Integer i when i == 0 -> "zero";
        case Integer i             -> "positive integer: " + i;
        case String s when s.isEmpty() -> "empty string";
        case String s              -> "string: " + s;
        case null                  -> "null";
        default                    -> "other: " + obj;
    };
}

// Java 21 Record Patterns
void printPoint(Object obj) {
    if (obj instanceof Point(double x, double y)) {
        System.out.printf("x=%f, y=%f%n", x, y);
    }
}
```

```clojure
;; Clojure: multimethods for open dispatch (most flexible)

;; Define dispatch function
(defmulti area :shape-type)

;; Define methods per type
(defmethod area :circle [{:keys [radius]}]
  (* Math/PI radius radius))

(defmethod area :rectangle [{:keys [width height]}]
  (* width height))

(defmethod area :triangle [{:keys [base height]}]
  (* 0.5 base height))

;; No exhaustiveness check, but open for extension
(area {:shape-type :circle :radius 5.0})     ; → 78.53...
(area {:shape-type :rectangle :width 4 :height 5}) ; → 20

;; Clojure: protocols for closed-set dispatch (sealed-class equivalent)
(defprotocol Shape
  (area [shape])
  (perimeter [shape])
  (describe [shape]))

(defrecord Circle [radius]
  Shape
  (area [_] (* Math/PI radius radius))
  (perimeter [_] (* 2 Math/PI radius))
  (describe [_] (str "Circle with radius " radius)))

(defrecord Rectangle [width height]
  Shape
  (area [_] (* width height))
  (perimeter [_] (* 2 (+ width height)))
  (describe [_] (str "Rectangle " width "x" height)))

;; Polymorphic dispatch
(def shapes [(->Circle 5) (->Rectangle 4 6)])
(map area shapes)      ; → (78.53... 24)
(map describe shapes)  ; → ("Circle with radius 5" "Rectangle 4x6")

;; core.match for Java 21-style pattern matching
(require '[clojure.core.match :refer [match]])

(defn describe-value [obj]
  (match [obj]
    [(:or nil)]                         "null"
    [(n :guard #(and (integer? %) (neg? %)))] (str "negative integer: " n)
    [(0)]                               "zero"
    [(n :guard integer?)]              (str "positive integer: " n)
    [("")]                              "empty string"
    [(s :guard string?)]               (str "string: " s)
    :else                              (str "other: " obj)))

;; Destructuring patterns (like Java 21 record patterns)
(defn process-point [obj]
  (when (and (map? obj) (:x obj) (:y obj))
    (let [{:keys [x y]} obj]
      (printf "x=%.1f, y=%.1f%n" x y))))

;; or with core.match:
(match [obj]
  [{:x x :y y}] (printf "x=%.1f, y=%.1f%n" x y))
```

---

## 14. Text Blocks and Strings

Java 13-15 introduced Text Blocks (triple-quoted strings). Clojure has
always had multi-line strings:

```java
// Java 14+ Text Blocks
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;

String html = """
    <html>
        <body>
            <p>Hello, %s!</p>
        </body>
    </html>
    """.formatted(name);

String sql = """
    SELECT u.name, u.email
    FROM users u
    WHERE u.active = true
      AND u.department = ?
    ORDER BY u.name
    """;

// Java String methods (added over time)
// Java 11
"  hello  ".strip();         // trim (Unicode-aware)
"  ".isBlank();              // true
"a\nb\nc".lines();          // Stream<String>
"hello".repeat(3);           // "hellohellohello"
// Java 12
"hello".indent(4);
// Java 15
"hello".stripIndent();
"hello\n".stripTrailing();
```

```clojure
;; Clojure multi-line strings (always worked this way)
(def json
  "{
  \"name\": \"Alice\",
  \"age\": 30
}")

;; Or without escaping using raw strings via (str) or heredoc patterns
(def sql
  "SELECT u.name, u.email
FROM users u
WHERE u.active = true
  AND u.department = ?
ORDER BY u.name")

;; String formatting
(def html
  (format "<html><body><p>Hello, %s!</p></body></html>" name))

;; String interpolation (via clojure.string or strformat lib)
;; Standard: use str and format
(str "Hello, " name "!")
(format "Hello, %s! You are %d years old." name age)

;; clojure.string functions
(require '[clojure.string :as str])
(str/trim "  hello  ")       ; strip
(str/blank? "  ")            ; true
(str/split-lines "a\nb\nc")  ; ["a" "b" "c"]
(str/join "\n" lines)        ; join
(apply str (repeat 3 "hello")) ; "hellohellohello"

;; For complex string templates, use Selmer or Stencil library
;; Or hiccup for HTML generation (far superior to string templates)
(require '[hiccup.core :refer [html]])
(html [:html [:body [:p (str "Hello, " name "!")]]])
```

---

## 15. Concurrency

This is where Clojure most dramatically outshines Java. Java's concurrency
model evolved from raw threads to Futures to CompletableFuture to virtual
threads (Java 21). Clojure's was designed right from the start.

### Java Threads → Clojure's Reference Types

```java
// Java threading evolution

// Java 1: Raw threads (dangerous)
Thread t = new Thread(() -> doWork());
t.start();
t.join();

// Java 5: synchronized locks and volatile
private synchronized void increment() { count++; }
private volatile boolean running = true;

// Java 5: Atomic types
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
counter.compareAndSet(expected, updated);

// Java 5: Concurrent collections
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

// Java 5: Executor Service
ExecutorService pool = Executors.newFixedThreadPool(10);
Future<Integer> future = pool.submit(() -> computeSomething());
Integer result = future.get();

// Java 8: CompletableFuture
CompletableFuture<String> cf = CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(String::toUpperCase)
    .thenCombine(
        CompletableFuture.supplyAsync(() -> fetchMoreData()),
        (a, b) -> a + b)
    .exceptionally(ex -> "Error: " + ex.getMessage());
cf.join();

// Java 21: Virtual Threads (Project Loom)
Thread.ofVirtual().start(() -> handleRequest());
try (ExecutorService vpool = Executors.newVirtualThreadPerTaskExecutor()) {
    vpool.submit(() -> doWork());
}
// Structured concurrency (Java 21)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> u1 = scope.fork(() -> fetchUser(id1));
    Future<String> u2 = scope.fork(() -> fetchUser(id2));
    scope.join().throwIfFailed();
    return u1.get() + u2.get();
}
```

```clojure
;; Clojure concurrency — designed from the start

;; Atom: like AtomicReference but more powerful
(def counter (atom 0))
(swap! counter inc)              ; like incrementAndGet()
(swap! counter + 5)              ; apply any function
(compare-and-set! counter 5 10)  ; like compareAndSet()
@counter                         ; deref to read

;; Ref + STM: like synchronized but composable
;; Transfer money: either both change or neither does
(def account-a (ref 1000))
(def account-b (ref 500))

(defn transfer! [from to amount]
  (dosync
    (when (>= @from amount)
      (alter from - amount)
      (alter to + amount)
      true)))

(transfer! account-a account-b 200)
;; Completely safe, no deadlocks possible

;; Future: like CompletableFuture but simpler
(def f (future (fetch-data)))    ; starts immediately in thread pool
@f                               ; deref blocks until ready
(deref f 3000 :timeout)         ; with timeout
(realized? f)                    ; check without blocking

;; Run many things in parallel
(defn parallel-fetch [ids]
  (->> ids
       (map #(future (fetch-user %)))  ; all start in parallel
       (map deref)))                   ; collect results

;; Promise: like CompletableFuture.completedFuture (you set the value)
(def p (promise))
(future (Thread/sleep 1000) (deliver p "result"))
@p   ; blocks until delivered

;; Agent: asynchronous updates
(def state (agent {:count 0 :items []}))
(send state update :count inc)       ; async, returns immediately
(send state update :items conj "x")  ; queued after previous
(await state)                        ; wait for all pending actions
@state                               ; read current value

;; core.async: like Java 21's StructuredConcurrency + Go-style channels
(require '[clojure.core.async :as async])

(let [ch1 (async/chan)
      ch2 (async/chan)]
  (async/go (async/>! ch1 (fetch-user 1)))
  (async/go (async/>! ch2 (fetch-user 2)))
  ;; Wait for both (like StructuredTaskScope)
  (let [u1 (async/<!! ch1)
        u2 (async/<!! ch2)]
    [u1 u2]))

;; Virtual threads via Java interop (Clojure on Java 21)
(Thread/ofVirtual)
(.start (.. Thread ofVirtual (name "my-vthread")
            (unstarted #(do-work))))

;; Pmap: parallel map using threads
(pmap heavy-computation (range 1000))

;; Reducers: parallel data processing
(require '[clojure.core.reducers :as r])
(->> (range 1000000)
     (r/filter even?)
     (r/map #(* % %))
     (r/fold +))   ; parallel fold
```

### Java 21 Structured Concurrency → core.async

```java
// Java 21 Structured Concurrency
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var userFuture  = scope.fork(() -> findUser(userId));
    var orderFuture = scope.fork(() -> findOrders(userId));
    var addrFuture  = scope.fork(() -> findAddress(userId));

    scope.join().throwIfFailed();

    return buildProfile(
        userFuture.get(),
        orderFuture.get(),
        addrFuture.get());
}
```

```clojure
;; Clojure equivalent with core.async
(require '[clojure.core.async :refer [go <!! chan]])

(defn build-user-profile [user-id]
  (let [user-ch  (go (find-user user-id))
        order-ch (go (find-orders user-id))
        addr-ch  (go (find-address user-id))]
    ;; All three run concurrently, collect when all done
    (build-profile (<!! user-ch)
                   (<!! order-ch)
                   (<!! addr-ch))))

;; Or with futures (simpler for this case)
(defn build-user-profile [user-id]
  (let [user-f  (future (find-user user-id))
        order-f (future (find-orders user-id))
        addr-f  (future (find-address user-id))]
    (build-profile @user-f @order-f @addr-f)))
```

---

## 16. Object-Oriented to Protocols

Java uses classes and interfaces for polymorphism. Clojure uses protocols
(like interfaces) and multimethods (more flexible):

```java
// Java interface + implementations
public interface Serializable {
    String toJson();
    static <T> T fromJson(String json, Class<T> type);
}

public interface Printable {
    void print();
    default void printWithBorder() {
        System.out.println("---");
        print();
        System.out.println("---");
    }
}

public class User implements Serializable, Printable {
    private final String name;
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toJson() {
        return "{\"name\":\"" + name + "\",\"age\":" + age + "}";
    }

    @Override
    public void print() {
        System.out.println(name + " (age: " + age + ")");
    }
}
```

```clojure
;; Clojure protocols (like Java interfaces)
(defprotocol Serializable
  (to-json [this])
  (to-map  [this]))

(defprotocol Printable
  (print-it [this])
  (print-with-border [this]))   ; can have default implementations via defn

;; Provide default implementations separately
(defn print-with-border [x]
  (println "---")
  (print-it x)
  (println "---"))

;; Implement with defrecord
(defrecord User [name age]
  Serializable
  (to-json [this]
    (format "{\"name\":\"%s\",\"age\":%d}" name age))
  (to-map [this]
    {:name name :age age})

  Printable
  (print-it [this]
    (println (str name " (age: " age ")")))
  (print-with-border [this]
    (println "---")
    (println (str name " (age: " age ")"))
    (println "---")))

(def alice (->User "Alice" 30))
(to-json alice)          ; → "{\"name\":\"Alice\",\"age\":30}"
(print-it alice)         ; → Alice (age: 30)

;; Retroactive extension (impossible in Java!)
;; Add protocol to EXISTING types without modifying them
(extend-protocol Serializable
  String
  (to-json [s] (str "\"" s "\""))
  (to-map  [s] {:value s})

  Long
  (to-json [n] (str n))
  (to-map  [n] {:value n}))

(to-json "hello")    ; → "\"hello\""
(to-json 42)         ; → "42"

;; Multimethods: dispatch on ANYTHING (not just type)
(defmulti process-event :event-type)

(defmethod process-event :user-created [{:keys [user-id email]}]
  (send-welcome-email user-id email))

(defmethod process-event :order-placed [{:keys [order-id amount]}]
  (charge-payment order-id amount))

(defmethod process-event :default [event]
  (log-unknown-event event))

;; Dispatch on multiple values
(defmulti speak (fn [animal mood] [(:type animal) mood]))

(defmethod speak [:dog :happy] [animal _] "Woof woof!")
(defmethod speak [:dog :angry] [animal _] "GRRR!")
(defmethod speak [:cat :happy] [animal _] "Purr...")
(defmethod speak [:cat :angry] [animal _] "HISSS!")
```

### Inheritance vs Composition

```java
// Java inheritance (often leads to fragile hierarchies)
class Animal {
    protected String name;
    public void breathe() { }
    public void eat() { }
}

class Pet extends Animal {
    public void beLovedBy(Human h) { }
}

class Dog extends Pet {
    public void fetch() { }
    public void bark() { }
}

class ServiceDog extends Dog {
    public void guide() { }
}
// ServiceDog inherits from Animal, Pet, Dog
// Deep hierarchy is rigid and hard to change
```

```clojure
;; Clojure: composition via protocols (like Java interfaces but retroactive)
;; No inheritance — compose behaviors

(defprotocol Breathing  (breathe [this]))
(defprotocol Eating     (eat [this food]))
(defprotocol Fetching   (fetch [this item]))
(defprotocol Guiding    (guide [this person]))

(defrecord Dog [name breed]
  Breathing
  (breathe [_] (println name "breathes"))
  Eating
  (eat [_ food] (println name "eats" food))
  Fetching
  (fetch [_ item] (println name "fetches" item)))

(defrecord ServiceDog [name breed certified]
  Breathing
  (breathe [_] (println name "breathes"))
  Eating
  (eat [_ food] (println name "eats" food))
  Fetching
  (fetch [_ item] (println name "fetches" item))
  Guiding
  (guide [_ person] (println name "guides" person)))

;; Or use maps and functions — no types needed at all!
(defn make-dog [name breed]
  {:name name :breed breed :type :dog})

(defn dog-fetch [dog item]
  (println (:name dog) "fetches" item))
```

---

## 17. Error Handling

```java
// Java exceptions
try {
    int result = divide(a, b);
    process(result);
} catch (ArithmeticException e) {
    log.error("Division failed", e);
    return -1;
} catch (ProcessingException | ValidationException e) {
    throw new RuntimeException("Processing failed", e);
} finally {
    cleanup();
}

// Java checked exceptions (require declaration)
public void readFile(String path) throws IOException {
    // ...
}

// Java throw
throw new IllegalArgumentException("Invalid input: " + input);

// Custom exceptions
public class BusinessException extends RuntimeException {
    private final String code;
    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
}
```

```clojure
;; Clojure exceptions — similar to Java but no checked exceptions

(try
  (let [result (divide a b)]
    (process result))
  (catch ArithmeticException e
    (log/error "Division failed" e)
    -1)
  (catch Exception e
    (throw (RuntimeException. "Processing failed" e)))
  (finally
    (cleanup)))

;; throw
(throw (IllegalArgumentException. "Invalid input"))
(throw (ex-info "Invalid input"     ; Clojure-idiomatic: ex-info with data
                {:input input
                 :code "INVALID_INPUT"
                 :type :validation-error}))

;; Catch and inspect ex-info
(try
  (process user-input)
  (catch clojure.lang.ExceptionInfo e
    (let [data (ex-data e)]
      (case (:type data)
        :validation-error (str "Invalid: " (:input data))
        :not-found "Resource not found"
        (throw e)))))

;; ex-info is idiomatic Clojure — no need to define exception classes
;; The data map carries all relevant context

;; No checked exceptions — no throws declarations
;; Functions return nil or throw unchecked exceptions

;; Functional error handling (preferred for expected failures)
;; Return values that encode success or failure

;; Option 1: return nil for failure
(defn safe-divide [a b]
  (when (not= b 0) (/ a b)))

;; Option 2: return map with :ok/:error
(defn parse-user [data]
  (if (valid-user? data)
    {:ok (create-user data)}
    {:error "Invalid user data" :data data}))

;; Option 3: use the 'failjure' or 'either' library
;; (cognitect.anomalies is common in production Clojure)
(require '[cognitect.anomalies :as anom])

(defn find-user [id]
  (if-let [user (db/get-user id)]
    user
    {::anom/category ::anom/not-found
     ::anom/message  (str "User not found: " id)}))
```

---

## 18. Modules and Namespaces

Java 9+ introduced the module system. Clojure uses namespaces:

```java
// Java 9+ module-info.java
module com.example.myapp {
    requires java.net.http;
    requires com.fasterxml.jackson.databind;
    exports com.example.myapp.api;
    exports com.example.myapp.model;
    // com.example.myapp.internal is not exported
}

// Java package + class
package com.example.myapp.service;

import com.example.myapp.model.User;
import com.example.myapp.repository.UserRepository;
import java.util.Optional;

public class UserService {
    private final UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;
    }

    public Optional<User> findUser(long id) {
        return repo.findById(id);
    }
}

// Java 25: Simplified Module Imports (JEP 476)
import module java.net.http;   // imports all exported packages from module
import module java.base;
```

```clojure
;; Clojure namespace (ns)
(ns com.example.myapp.service
  "User service namespace."

  (:require [com.example.myapp.model :as model]
            [com.example.myapp.repository :as repo]
            [clojure.string :as str])

  (:import [java.time Instant]
           [java.util UUID]))

;; Functions in this namespace
(defn find-user [id]
  (repo/find-by-id id))

(defn create-user! [user-data]
  (let [user (model/make-user user-data)]
    (repo/save! user)))

;; Private functions (not accessible from other namespaces)
(defn- validate-user [user]
  (and (:name user) (:email user)))

;; Requiring namespaces
(ns my.app.core
  (:require
    ;; with alias
    [clojure.string :as str]
    ;; refer specific functions
    [clojure.set :refer [union intersection]]
    ;; refer all (avoid in production)
    [my.util :refer :all]
    ;; import Java classes
    )
  (:import
    [java.util Date UUID]
    [java.time LocalDateTime ZoneId]))

;; Dynamic require (like Java's Class.forName)
(require '[some.namespace :as ns])

;; Namespace introspection
(ns-publics 'clojure.string)    ; all public vars
(ns-interns 'my.namespace)      ; all vars including private
(dir clojure.string)            ; list public vars in REPL
```

---

## 19. Java Interop

Clojure runs on the JVM and has seamless Java interoperability:

```java
// Java code you want to call from Clojure
import java.util.regex.Pattern;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.net.URI;
import java.net.http.*;
```

```clojure
;; Calling Java from Clojure

;; Creating objects (ClassName. is the constructor)
(java.util.Date.)                    ; new Date()
(java.util.ArrayList.)               ; new ArrayList()
(java.util.HashMap. {"a" 1})        ; new HashMap(map)
(java.util.UUID/randomUUID)          ; UUID.randomUUID() (static)

;; Calling instance methods (.method object args)
(.toUpperCase "hello")               ; "hello".toUpperCase()
(.length "hello")                    ; "hello".length()
(.substring "hello" 1 3)            ; "hello".substring(1, 3)
(.getTime (java.util.Date.))        ; new Date().getTime()

;; Static methods (Class/method args)
(System/currentTimeMillis)           ; System.currentTimeMillis()
(Math/sqrt 16.0)                     ; Math.sqrt(16.0)
(Integer/parseInt "42")             ; Integer.parseInt("42")
(System/getenv "PATH")              ; System.getenv("PATH")

;; Static fields
Math/PI                              ; Math.PI
Integer/MAX_VALUE                    ; Integer.MAX_VALUE

;; Method chaining (..)
(.. "hello world"
    toUpperCase
    (substring 0 5))                 ; "HELLO"

;; doto: call multiple methods on same object
(doto (java.util.ArrayList.)
  (.add "one")
  (.add "two")
  (.add "three"))                    ; returns the ArrayList

;; Java 11+ HttpClient
(let [client  (java.net.http.HttpClient/newHttpClient)
      request (.. java.net.http.HttpRequest
                  newBuilder
                  (uri (java.net.URI/create "https://example.com"))
                  build)
      response (.send client request
                       (java.net.http.HttpResponse$BodyHandlers/ofString))]
  (.body response))

;; Type casting and checking
(instance? String "hello")           ; true
(cast String "hello")                ; type cast
(type "hello")                       ; java.lang.String

;; Implementing Java interfaces
;; proxy: implement Java interface at runtime
(def my-runnable
  (proxy [Runnable] []
    (run [] (println "Running!"))))

(.start (Thread. my-runnable))

;; reify: efficient interface implementation (preferred)
(def my-comparator
  (reify java.util.Comparator
    (compare [this a b]
      (compare (count a) (count b)))))

;; Implement multiple interfaces
(def handler
  (reify
    java.util.function.Function
    (apply [this x] (* x 2))

    java.util.function.Predicate
    (test [this x] (even? x))))

;; Annotations via metadata
(defn ^{:tag String} get-name [^{:tag User} user]
  (.getName user))
```

---

## 20. Testing

```java
// Java JUnit 5
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

@DisplayName("User Service Tests")
class UserServiceTest {

    private UserRepository mockRepo;
    private UserService service;

    @BeforeEach
    void setUp() {
        mockRepo = mock(UserRepository.class);
        service = new UserService(mockRepo);
    }

    @AfterEach
    void tearDown() {
        // cleanup
    }

    @Test
    @DisplayName("Should find user by id")
    void testFindUser() {
        var user = new User(1L, "Alice", "alice@example.com");
        when(mockRepo.findById(1L)).thenReturn(Optional.of(user));

        var result = service.findUser(1L);

        assertThat(result).isPresent();
        assertThat(result.get().getName()).isEqualTo("Alice");
        verify(mockRepo).findById(1L);
    }

    @ParameterizedTest
    @ValueSource(strings = {"", " ", "  "})
    void testBlankNames(String name) {
        assertThatThrownBy(() -> new User(1L, name, "a@b.com"))
            .isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    void testAsyncOperation() {
        assertTimeout(Duration.ofSeconds(1), () -> {
            service.asyncOperation().get();
        });
    }
}
```

```clojure
;; Clojure clojure.test
(ns myapp.service-test
  (:require [clojure.test :refer [deftest testing is are
                                  use-fixtures run-tests]]
            [myapp.service :as sut]   ; sut = system under test
            [myapp.repository :as repo]))

;; Fixtures (like @BeforeEach/@AfterEach)
(defn with-test-db [f]
  (let [db (create-test-db)]
    (try
      (f)    ; run the test
      (finally
        (drop-test-db db)))))

(use-fixtures :each with-test-db)

;; Basic test
(deftest find-user-test
  (testing "should find user by id"
    (let [user {:id 1 :name "Alice" :email "alice@example.com"}
          repo (reify repo/UserRepo
                 (find-by-id [_ id] (when (= id 1) user)))]
      (is (= user (sut/find-user repo 1)))
      (is (nil? (sut/find-user repo 999)))))

  (testing "should return nil for missing user"
    (is (nil? (sut/find-user stub-repo 999)))))

;; Parameterized tests with are
(deftest validate-name-test
  (are [name valid?]
       (= valid? (sut/valid-name? name))
    ""          false
    " "         false
    "Alice"     true
    "Bob Smith" true
    nil         false))

;; Test exceptions
(deftest division-test
  (is (thrown? ArithmeticException (/ 1 0)))
  (is (thrown-with-msg? Exception #"Invalid"
                        (sut/parse-user {}))))

;; Test with test.check (property-based, like JUnit's parameterized + random)
(require '[clojure.test.check :as tc]
         '[clojure.test.check.generators :as gen]
         '[clojure.test.check.properties :as prop]
         '[clojure.test.check.clojure-test :refer [defspec]])

(defspec user-roundtrip 1000
  (prop/for-all [name gen/string-alphanumeric
                 age  (gen/choose 0 120)]
    (let [user {:name name :age age}]
      (= user (sut/deserialize (sut/serialize user))))))

;; Mocking with with-redefs (simple, built-in)
(deftest email-test
  (with-redefs [send-email (fn [to body] {:sent true :to to})]
    (let [result (sut/notify-user {:email "alice@example.com"})]
      (is (:sent result))
      (is (= "alice@example.com" (:to result))))))

;; Run tests
;; From REPL:
;; (run-tests 'myapp.service-test)
;; (run-all-tests)
;; From command line:
;; clj -M:test
```

---

## 21. Build Tools

### Maven/Gradle → deps.edn / Leiningen

```xml
<!-- Maven pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.clojure</groupId>
        <artifactId>clojure</artifactId>
        <version>1.11.1</version>
    </dependency>
    <dependency>
        <groupId>ring</groupId>
        <artifactId>ring-core</artifactId>
        <version>1.10.0</version>
    </dependency>
    <dependency>
        <groupId>cheshire</groupId>
        <artifactId>cheshire</artifactId>
        <version>5.11.0</version>
    </dependency>
</dependencies>
```

```clojure
;; deps.edn equivalent
{:deps {org.clojure/clojure  {:mvn/version "1.11.1"}
        ring/ring-core        {:mvn/version "1.10.0"}
        cheshire/cheshire     {:mvn/version "5.11.0"}}}
```

```groovy
// Gradle build.gradle
dependencies {
    implementation 'org.clojure:clojure:1.11.1'
    testImplementation 'junit:junit:4.13'
}
```

### build.clj (Clojure Build Tool)

```clojure
;; build.clj — like Gradle's build.gradle but in Clojure
(ns build
  (:require [clojure.tools.build.api :as b]))

(def lib 'com.example/myapp)
(def version "1.0.0")
(def class-dir "target/classes")
(def basis (b/create-basis {:project "deps.edn"}))

(defn clean [_]
  (b/delete {:path "target"}))

(defn compile-java [_]
  (b/javac {:src-dirs  ["java"]
            :class-dir class-dir
            :basis     basis}))

(defn jar [_]
  (b/write-pom {:class-dir class-dir
                :lib       lib
                :version   version
                :basis     basis
                :src-dirs  ["src"]})
  (b/copy-dir {:src-dirs  ["src" "resources"]
               :target-dir class-dir})
  (b/jar {:class-dir class-dir
          :jar-file  (format "target/%s-%s.jar"
                              (name lib) version)}))

(defn uber [_]
  (clean nil)
  (b/copy-dir {:src-dirs ["src" "resources"]
               :target-dir class-dir})
  (b/compile-clj {:basis     basis
                  :src-dirs  ["src"]
                  :class-dir class-dir})
  (b/uber {:class-dir class-dir
           :uber-file "target/myapp.jar"
           :basis     basis
           :main      'myapp.core}))
```

### Leiningen (project.clj) — Alternative to deps.edn

```clojure
;; project.clj (like a more opinionated Maven pom.xml)
(defproject com.example/myapp "1.0.0"
  :description "My Clojure Application"
  :url "https://example.com"
  :license {:name "MIT" :url "https://opensource.org/licenses/MIT"}

  :dependencies [[org.clojure/clojure "1.11.1"]
                 [ring/ring-core "1.10.0"]
                 [ring/ring-jetty-adapter "1.10.0"]
                 [compojure "1.7.0"]
                 [cheshire "5.11.0"]]

  :main ^:skip-aot myapp.core

  :source-paths ["src"]
  :test-paths ["test"]
  :resource-paths ["resources"]

  :profiles {:dev  {:dependencies [[ring/ring-mock "0.4.0"]]
                    :plugins      [[lein-ring "0.12.6"]]}
             :uberjar {:aot :all}}

  :ring {:handler myapp.core/app}
  :jvm-opts ["-server" "-Xmx512m"]

  :aliases {"test"    ["test"]
            "build"   ["uberjar"]
            "deploy"  ["deploy" "clojars"]})
```

### Common Library Mapping

| Java ecosystem    | Clojure equivalent                |
| ----------------- | --------------------------------- |
| Spring Boot       | Integrant, Mount, Component       |
| Spring MVC        | Ring + Reitit/Compojure           |
| Spring Security   | Buddy                             |
| Spring Data / JPA | next.jdbc, HoneySQL, Duct         |
| Jackson           | Cheshire, data.json               |
| Lombok            | defrecord (no boilerplate needed) |
| MapStruct         | Simple map transforms             |
| Guava             | clojure.core + standard lib       |
| Apache Commons    | clojure.string, clojure.java.io   |
| SLF4J / Log4j     | Timbre                            |
| Micrometer        | Iapetos (Prometheus)              |
| JUnit 5           | clojure.test                      |
| Mockito           | with-redefs, reify                |
| AssertJ           | is, are in clojure.test           |
| Testcontainers    | clj-test-containers               |
| WireMock          | clj-wiremock                      |
| OpenAPI/Swagger   | Reitit spec integration           |
| Kafka client      | jackdaw, clj-kafka                |
| Redis client      | Carmine                           |
| MongoDB           | Monger                            |
| Elasticsearch     | Spandex                           |

---

## 22. Common Patterns and Idioms

### Replacing Java Design Patterns

Many Java design patterns exist because Java lacks features. In Clojure,
these patterns either disappear or become trivial:

```java
// Java Singleton (entire pattern)
public class DatabaseConnection {
    private static DatabaseConnection instance;
    private DatabaseConnection() { }
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}
```

```clojure
;; Clojure: just def a value
(def db-connection (create-db-connection config))
;; Done. def'd values are singletons by definition.
```

```java
// Java Builder Pattern (entire class + builder class)
User user = User.builder()
    .name("Alice")
    .age(30)
    .email("alice@example.com")
    .build();
```

```clojure
;; Clojure: just use a map
(def user {:name "Alice" :age 30 :email "alice@example.com"})

;; Or a factory function if you need validation
(defn make-user [{:keys [name age email]}]
  {:pre [(string? name) (pos? age) (string? email)]}
  {:name name :age age :email email :id (random-uuid)})
```

```java
// Java Strategy Pattern
interface SortStrategy {
    List<Integer> sort(List<Integer> data);
}
class BubbleSort implements SortStrategy { ... }
class QuickSort implements SortStrategy { ... }

class Sorter {
    private SortStrategy strategy;
    public Sorter(SortStrategy strategy) { this.strategy = strategy; }
    public List<Integer> sort(List<Integer> data) {
        return strategy.sort(data);
    }
}
```

```clojure
;; Clojure: functions ARE the strategy
(defn sort-data [data strategy]
  (strategy data))

(sort-data [3 1 4 1 5] sort)                    ; built-in sort
(sort-data [3 1 4 1 5] #(sort > %))             ; reverse sort
(sort-data users #(sort-by :age %))              ; sort by age
```

```java
// Java Observer Pattern (entire infrastructure)
interface Observer { void update(Event e); }
class EventBus {
    private Map<String, List<Observer>> listeners = new HashMap<>();
    public void subscribe(String event, Observer o) { ... }
    public void publish(String event, Event e) { ... }
}
```

```clojure
;; Clojure: core.async channels, or just use add-watch on atoms
(def state (atom {:events []}))

;; Watch: called whenever state changes
(add-watch state :event-logger
  (fn [key ref old-state new-state]
    (println "State changed:"
             (count (:events new-state)) "events")))

(swap! state update :events conj {:type :user-login :user "Alice"})
;; → "State changed: 1 events"
```

### Data Transformation Pipelines

The most common Clojure pattern, replacing Java's OOP service layers:

```java
// Java service layer (lots of classes, methods, mutation)
public class OrderProcessingService {
    public ProcessedOrder process(Order order) {
        Order validated = validator.validate(order);
        Order enriched = enricher.enrich(validated);
        Order priced = pricer.calculatePrice(enriched);
        return converter.toProcessed(priced);
    }
}
```

```clojure
;; Clojure: data pipeline (no classes, just functions)
(defn process-order [order]
  (->> order
       validate-order
       enrich-order
       calculate-price
       finalize-order))

;; Each step is a pure function: map → map
(defn validate-order [{:keys [items total] :as order}]
  (cond
    (empty? items) (throw (ex-info "Empty order" {:order order}))
    (neg? total)   (throw (ex-info "Negative total" {:order order}))
    :else          (assoc order :validated true)))

(defn calculate-price [order]
  (let [subtotal (->> (:items order)
                      (map #(* (:price %) (:quantity %)))
                      (reduce +))
        tax (* subtotal 0.1)]
    (assoc order
           :subtotal subtotal
           :tax tax
           :total (+ subtotal tax))))
```

### Dependency Injection → Function Parameters

```java
// Java Spring DI
@Service
public class UserService {
    @Autowired
    private UserRepository repo;
    @Autowired
    private EmailService email;
    @Autowired
    private AuditService audit;

    public User createUser(CreateUserRequest req) {
        User user = repo.save(new User(req));
        email.sendWelcome(user);
        audit.log("user-created", user);
        return user;
    }
}
```

```clojure
;; Clojure: just pass dependencies as parameters
(defn create-user!
  "Create a user. Dependencies passed explicitly."
  [db email-service audit-service user-data]
  (let [user (db/save! db (make-user user-data))]
    (email/send-welcome! email-service user)
    (audit/log! audit-service "user-created" user)
    user))

;; Or use a system map (Integrant/Component pattern)
(defn create-user!
  [{:keys [db email audit]} user-data]  ; destructure the system
  (let [user (db/save! db (make-user user-data))]
    (email/send-welcome! email user)
    (audit/log! audit "user-created" user)
    user))

;; Call with explicit deps (easy to test with mocks!)
(create-user! {:db test-db
               :email mock-email
               :audit no-op-audit}
              {:name "Alice" :email "alice@example.com"})
```

---

## 23. Java Version Feature Mapping

### Java 8 Features → Clojure

```java
// Java 8: Lambdas
list.stream().map(x -> x * 2).collect(toList());
// Clojure equivalent:
(map #(* % 2) lst)

// Java 8: Method References
list.stream().map(String::toUpperCase).collect(toList());
// Clojure equivalent:
(map clojure.string/upper-case lst)

// Java 8: Default interface methods
interface Drawable {
    void draw();
    default void drawWithBorder() { border(); draw(); border(); }
}
// Clojure equivalent:
(defprotocol Drawable
  (draw [this]))
(defn draw-with-border [d]
  (border)
  (draw d)
  (border))

// Java 8: Optional
Optional<String> opt = Optional.of("hello");
opt.map(String::toUpperCase).orElse("default");
// Clojure equivalent:
(some-> "hello" clojure.string/upper-case)

// Java 8: Stream API
// Already covered extensively above

// Java 8: CompletableFuture
CompletableFuture.supplyAsync(() -> fetchData())
                 .thenApply(String::toUpperCase);
// Clojure equivalent:
(future (clojure.string/upper-case (fetch-data)))

// Java 8: Date/Time API
LocalDate.now()
LocalDate.of(2024, 1, 15)
// Clojure equivalent:
(java.time.LocalDate/now)
(java.time.LocalDate/of 2024 1 15)
;; Or use clj-time or java-time library:
(require '[java-time :as t])
(t/local-date)
(t/local-date 2024 1 15)
```

### Java 11 Features → Clojure

```java
// Java 11: var in lambdas
list.stream().map((var s) -> s.toUpperCase()).collect(toList());
// Not relevant in Clojure — no type declarations in lambdas

// Java 11: New String methods
"  hello  ".strip();        // Clojure: (str/trim "  hello  ")
"  ".isBlank();             // Clojure: (str/blank? "  ")
"a\nb\nc".lines();          // Clojure: (str/split-lines "a\nb\nc")
"ha".repeat(3);             // Clojure: (apply str (repeat 3 "ha"))

// Java 11: HttpClient
HttpClient.newHttpClient()
          .send(request, BodyHandlers.ofString());
// Clojure: use clj-http or hato (wraps Java 11 HttpClient)
(require '[hato.client :as hc])
(hc/get "https://example.com" {:as :string})
```

### Java 14-17 Features → Clojure

```java
// Java 14+: Records
record Point(double x, double y) {}
// Clojure equivalent:
(defrecord Point [x y])
;; Or just a map: {:x 3.0 :y 4.0}

// Java 15+: Text Blocks
String json = """
    {"name": "Alice"}
    """;
// Clojure: strings are always multi-line
(def json "{\"name\": \"Alice\"}")

// Java 16+: instanceof pattern matching
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}
// Clojure:
(when (string? obj)
  (println (clojure.string/upper-case obj)))
;; Or with core.match destructuring

// Java 17: Sealed classes
sealed interface Shape permits Circle, Rectangle {}
// Clojure:
(defprotocol Shape (area [s]))
(defrecord Circle [radius]    Shape (area [_] (* Math/PI radius radius)))
(defrecord Rectangle [w h]   Shape (area [_] (* w h)))
```

### Java 21 Features → Clojure

```java
// Java 21: Virtual Threads
Thread.ofVirtual().start(() -> handleRequest(req));
ExecutorService vte = Executors.newVirtualThreadPerTaskExecutor();
// Clojure: future already uses thread pool; direct Java interop for vthreads
(let [vthread (.. Thread ofVirtual (start #(handle-request req)))]
  (.join vthread))

// Java 21: Sequenced Collections
List<String> list = List.of("a", "b", "c");
list.getFirst();   // "a"
list.getLast();    // "c"
list.reversed();   // reversed view
// Clojure always had these:
(first lst)   ; "a"
(last lst)    ; "c"
(reverse lst) ; reversed

// Java 21: Pattern matching for switch
String result = switch (obj) {
    case Integer i when i > 0 -> "positive: " + i;
    case String s             -> "string: " + s;
    default                   -> "other";
};
// Clojure with cond or core.match:
(cond
  (and (integer? obj) (pos? obj)) (str "positive: " obj)
  (string? obj)                   (str "string: " obj)
  :else                           "other")

// Java 21: Record Patterns
if (obj instanceof Point(double x, double y)) {
    System.out.printf("%.1f, %.1f%n", x, y);
}
// Clojure destructuring:
(when (instance? Point obj)
  (let [{:keys [x y]} obj]
    (printf "%.1f, %.1f%n" x y)))

// Or core.match:
(match [obj]
  [(p :guard #(instance? Point %))]
  (let [{:keys [x y]} p]
    (printf "%.1f, %.1f%n" x y)))

// Java 21: Structured Concurrency
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var f1 = scope.fork(() -> fetch1());
    var f2 = scope.fork(() -> fetch2());
    scope.join().throwIfFailed();
    return [f1.get(), f2.get()];
}
// Clojure:
(let [f1 (future (fetch1))
      f2 (future (fetch2))]
  [@f1 @f2])
```

### Java 25 Features → Clojure

```java
// Java 25: Value Classes (Project Valhalla, JEP 401)
// Value classes are inline, identity-free, immutable
value class Point(double x, double y) {}
// - No object identity (no == check)
// - Can be stored inline in arrays (no boxing)
// - Immutable

// In Clojure: all values have always been value-semantic
;; Maps, vectors, records — all value-based, no identity
(= {:x 1 :y 2} {:x 1 :y 2})  ; → true (value equality, always)
(identical? [1 2] [1 2])       ; → false (different objects)
(= [1 2] [1 2])                ; → true (value equality)

// Java 25: Primitive Types in Generics (Project Valhalla, JEP 402)
// ArrayList<int>  — no boxing!
// Map<int, String>
// Previously required Integer, Long, Double wrappers
// Clojure: never had boxing issues for collection operations
;; Clojure sequences already work uniformly on all types
;; For performance, use type hints: ^long, ^double

// Java 25: Flexible Constructor Bodies (JEP 482, finalized)
class Point {
    final double x, y;
    Point(double x, double y) {
        // Can now do work before super() call
        validate(x, y);         // previously illegal
        super();
        this.x = x; this.y = y;
    }
}
// Not relevant: Clojure constructors are much simpler

// Java 25: Module Import Declarations (JEP 476, finalized)
import module java.net.http;   // imports entire module's public API
import module java.base;
// Clojure ns declarations already work this way:
(ns myapp (:require [clojure.string :as str]  ; explicit is better
                    [clojure.set    :as set]))

// Java 25: Simplified Main (JEP 477, finalized)
// No class or public static void main needed
void main() {
    System.out.println("Hello!");
}
// Clojure: always been simple
(println "Hello!")          ; at top level — that's your main
;; or:
(defn -main [& args]
  (println "Hello!"))
```

---

## 24. Style and Conventions

### Naming Conventions

```clojure
;; Namespaces: dot-separated, matching directory structure
(ns com.example.myapp.user-service)

;; Functions and variables: kebab-case (hyphen-separated)
(defn get-user-by-id [id] ...)
(def default-timeout 5000)

;; Predicates end with ?
(defn active? [user] ...)
(defn valid-email? [s] ...)

;; Side-effecting functions (mutation, I/O) end with !
(defn save-user! [user] ...)
(defn send-email! [to body] ...)
(defn reset-state! [] ...)

;; Private functions: use defn-
(defn- internal-helper [] ...)

;; Dynamic vars (rebindable): *earmuffs*
(def ^:dynamic *current-user* nil)
(def ^:dynamic *db-connection* nil)

;; Protocol names: PascalCase (like Java interfaces)
(defprotocol UserRepository ...)
(defprotocol Serializable ...)

;; Record names: PascalCase (like Java classes)
(defrecord UserEntity [id name email] ...)

;; Constants: just regular defs (no SCREAMING_CASE needed)
(def max-retry-count 3)
(def default-page-size 20)

;; Namespace aliases: short and conventional
(require '[clojure.string :as str])
(require '[clojure.set    :as set])
(require '[clojure.java.io :as io])
(require '[clojure.core.async :as async])
```

### Code Organization

```clojure
;; Prefer flat namespaces over deep hierarchies
;; Java: com.example.app.service.impl.UserServiceImpl
;; Clojure: myapp.users  (the ns suffix tells you it's about users)

;; Typical Clojure application layout:
;; myapp.core       — entry point, system startup
;; myapp.db         — database operations
;; myapp.users      — user domain logic
;; myapp.orders     — order domain logic
;; myapp.http       — HTTP handlers
;; myapp.config     — configuration loading
;; myapp.util       — utility functions

;; Comment blocks in Clojure files:
;; 1. Namespace declaration
;; 2. Imports/requires
;; 3. Constants/config
;; 4. Private helpers
;; 5. Public API functions
;; 6. Commented REPL experiments at the bottom
(comment
  ;; These are experiments — not executed in production
  (start-system!)
  (stop-system!)
  (->> (get-all-users db)
       (filter active?)
       count))
```

### Formatting Style

```clojure
;; Closing parens at END of last line (Lisp style, NOT Allman style)

;; CORRECT:
(defn process-users [users]
  (let [active (filter active? users)
        sorted (sort-by :name active)]
    (map transform sorted)))

;; WRONG (never do this in Clojure):
(defn process-users [users]
  (let [active (filter active? users)
        sorted (sort-by :name active)
       ]                                 ; wrong
    (map transform sorted)
  )                                      ; wrong
)                                        ; wrong

;; Map formatting: align values for readability
{:name  "Alice"
 :age   30
 :email "alice@example.com"}

;; Threading: each step on its own line
(->> users
     (filter active?)
     (map :email)
     (filter valid-email?)
     (into #{}))

;; Function calls: one arg per line when they don't fit
(some-function-with-long-name
  first-argument
  second-argument
  third-argument)

;; Use cljfmt or zprint for automated formatting
```

### The Clojure Aesthetic: Data, Functions, Namespaces

The guiding principle Rich Hickey calls "Simple Made Easy":

```clojure
;; AVOID: OOP-style with mutable state and encapsulation
(defrecord UserService [db email-svc]
  Object
  ...)

;; PREFER: Functions that take data and return data
(defn find-active-users
  "Find all active users in the given database."
  [db]
  (->> (db/query db "SELECT * FROM users WHERE active = true")
       (map normalize-user)))

;; AVOID: Mixing side effects and computation
(defn process-and-save [user-data]
  (let [processed (complex-processing user-data)]
    (save-to-db! processed)          ; side effect mixed in
    (send-notification! processed)   ; another side effect
    processed))

;; PREFER: Pure core, explicit edges
(defn process-user [user-data]        ; pure
  (-> user-data
      validate
      enrich
      calculate-scores))

(defn handle-new-user! [user-data]    ; impure but explicit
  (let [processed (process-user user-data)]    ; pure computation
    (db/save! processed)                        ; explicit side effect
    (email/send-welcome! processed)             ; explicit side effect
    processed))
```

---

## Quick Reference Card

### Java → Clojure Cheat Sheet

#### Types and Variables

| Java                                     | Clojure                              |
| ---------------------------------------- | ------------------------------------ |
| `int x = 5`                              | `(def x 5)`                          |
| `final int x = 5`                        | `(def x 5)` (all defs are immutable) |
| `var x = 5` (Java 10+)                   | `(let [x 5] ...)`                    |
| `AtomicInteger n = new AtomicInteger(0)` | `(def n (atom 0))`                   |
| `n.incrementAndGet()`                    | `(swap! n inc)`                      |
| `n.get()`                                | `@n`                                 |
| `int`                                    | `Long` (default) or `int` with hint  |
| `long`                                   | `Long`                               |
| `double`                                 | `Double`                             |
| `boolean`                                | `Boolean`                            |
| `String`                                 | `String`                             |
| `null`                                   | `nil`                                |
| `true` / `false`                         | `true` / `false`                     |
| `Integer.MAX_VALUE`                      | `Integer/MAX_VALUE`                  |
| `(int) 3.14`                             | `(int 3.14)`                         |
| `instanceof`                             | `instance?` or `(type x)`            |

#### Functions and Lambdas

| Java                  | Clojure                               |
| --------------------- | ------------------------------------- |
| `x -> x * 2`          | `#(* % 2)`                            |
| `(x, y) -> x + y`     | `#(+ %1 %2)`                          |
| `() -> "hello"`       | `#"hello"` or `(fn [] "hello")`       |
| `String::toUpperCase` | `clojure.string/upper-case`           |
| `System.out::println` | `println`                             |
| `String::new`         | `->String` or `(fn [s] s)`            |
| `Function.andThen(g)` | `(comp g f)`                          |
| `Function.compose(g)` | `(comp f g)`                          |
| `Predicate.negate()`  | `(complement pred)`                   |
| `Predicate.and(p)`    | `(every-pred p1 p2)`                  |
| `Predicate.or(p)`     | `(some-fn p1 p2)`                     |
| `partial(f, a)`       | `(partial f a)`                       |
| `method(a, b, c)`     | `(method a b c)`                      |
| `obj.method(a)`       | `(method obj a)` or `(.method obj a)` |

#### Collections

| Java                        | Clojure                                   |
| --------------------------- | ----------------------------------------- |
| `new ArrayList<>()`         | `[]` or `(vector)`                        |
| `List.of(1,2,3)`            | `[1 2 3]`                                 |
| `new HashMap<>()`           | `{}`                                      |
| `Map.of("k",1)`             | `{:k 1}`                                  |
| `new HashSet<>()`           | `#{}`                                     |
| `Set.of(1,2,3)`             | `#{1 2 3}`                                |
| `list.get(0)`               | `(first lst)` or `(lst 0)`                |
| `list.get(list.size()-1)`   | `(last lst)`                              |
| `list.size()`               | `(count lst)`                             |
| `list.add(x)`               | `(conj lst x)`                            |
| `list.set(i, x)`            | `(assoc lst i x)`                         |
| `list.remove(i)`            | `(subvec lst 0 i) + (subvec lst (inc i))` |
| `list.contains(x)`          | `(some #{x} lst)`                         |
| `list.subList(a,b)`         | `(subvec lst a b)`                        |
| `Collections.sort(list)`    | `(sort lst)`                              |
| `Collections.reverse(list)` | `(reverse lst)`                           |
| `map.get("k")`              | `(get m :k)` or `(:k m)`                  |
| `map.getOrDefault("k",v)`   | `(get m :k v)`                            |
| `map.put("k",v)`            | `(assoc m :k v)`                          |
| `map.remove("k")`           | `(dissoc m :k)`                           |
| `map.containsKey("k")`      | `(contains? m :k)`                        |
| `map.keySet()`              | `(keys m)`                                |
| `map.values()`              | `(vals m)`                                |
| `map.entrySet()`            | `(seq m)` or iterate with `[k v]`         |

#### Streams → Sequences

| Java Streams                   | Clojure                            |
| ------------------------------ | ---------------------------------- |
| `stream.map(f)`                | `(map f coll)`                     |
| `stream.filter(p)`             | `(filter p coll)`                  |
| `stream.flatMap(f)`            | `(mapcat f coll)`                  |
| `stream.reduce(id, f)`         | `(reduce f id coll)`               |
| `stream.collect(toList())`     | `(vec coll)` or `(into [] coll)`   |
| `stream.collect(toSet())`      | `(set coll)` or `(into #{} coll)`  |
| `stream.forEach(f)`            | `(doseq [x coll] (f x))`           |
| `stream.count()`               | `(count coll)`                     |
| `stream.distinct()`            | `(distinct coll)`                  |
| `stream.sorted()`              | `(sort coll)`                      |
| `stream.sorted(cmp)`           | `(sort cmp coll)`                  |
| `stream.limit(n)`              | `(take n coll)`                    |
| `stream.skip(n)`               | `(drop n coll)`                    |
| `stream.anyMatch(p)`           | `(some p coll)`                    |
| `stream.allMatch(p)`           | `(every? p coll)`                  |
| `stream.noneMatch(p)`          | `(not-any? p coll)`                |
| `stream.findFirst()`           | `(first (filter p coll))`          |
| `stream.min(cmp)`              | `(apply min coll)`                 |
| `stream.max(cmp)`              | `(apply max coll)`                 |
| `stream.peek(f)`               | `(map (fn [x] (f x) x) coll)`      |
| `stream.takeWhile(p)`          | `(take-while p coll)`              |
| `stream.dropWhile(p)`          | `(drop-while p coll)`              |
| `Collectors.groupingBy(f)`     | `(group-by f coll)`                |
| `Collectors.joining(",")`      | `(clojure.string/join "," coll)`   |
| `Collectors.counting()`        | `(count coll)`                     |
| `Collectors.toMap(k,v)`        | `(into {} (map (juxt k v) coll))`  |
| `Collectors.partitioningBy(p)` | `(group-by #(boolean (p %)) coll)` |
| `stream.parallel()`            | `(pmap f coll)` or reducers        |
| `IntStream.range(a,b)`         | `(range a b)`                      |
| `Stream.iterate(seed,f)`       | `(iterate f seed)`                 |
| `Stream.of(1,2,3)`             | `[1 2 3]` or `(list 1 2 3)`        |

#### Control Flow

| Java                          | Clojure                      |
| ----------------------------- | ---------------------------- |
| `if (a) { b } else { c }`     | `(if a b c)`                 |
| `a ? b : c`                   | `(if a b c)`                 |
| `if (a) { b }`                | `(when a b)`                 |
| `switch (x) { case 1: ...; }` | `(case x 1 ...)`             |
| `switch (x) { case 1 -> ...}` | `(case x 1 ...)`             |
| `for (int i=0; i<n; i++)`     | `(dotimes [i n] ...)`        |
| `for (String s : list)`       | `(doseq [s list] ...)`       |
| `while (cond)`                | `(while cond ...)`           |
| `break`                       | `(reduced val)` in reduce    |
| `continue`                    | restructure with filter      |
| `return val`                  | last expression in body      |
| `throw new Exc(msg)`          | `(throw (ex-info msg data))` |
| `try { } catch (E e) { }`     | `(try ... (catch E e ...))`  |
| `finally { }`                 | `(finally ...)`              |

#### OOP → Clojure

| Java                       | Clojure                                   |
| -------------------------- | ----------------------------------------- |
| `class Foo { }`            | `(defrecord Foo [fields])` or map         |
| `new Foo(a, b)`            | `(->Foo a b)`                             |
| `new Foo()`                | `(map->Foo {})`                           |
| `obj.field`                | `(:field obj)`                            |
| `obj.setField(v)`          | `(assoc obj :field v)`                    |
| `interface Foo { }`        | `(defprotocol Foo ...)`                   |
| `class Foo implements Bar` | `(defrecord Foo [] Bar ...)`              |
| `extends`                  | `(defrecord Foo [] :extends Base)` (rare) |
| `abstract method`          | protocol method (no default impl)         |
| `default method`           | separate function                         |
| `@Override`                | method dispatch automatic                 |
| `static void main()`       | `(defn -main [& args] ...)`               |
| `System.out.println()`     | `(println ...)`                           |
| `String.format()`          | `(format ...)`                            |
| `Math.sqrt()`              | `(Math/sqrt ...)`                         |

#### Concurrency

| Java                               | Clojure                                             |
| ---------------------------------- | --------------------------------------------------- |
| `new Thread(() -> f())`            | `(future (f))`                                      |
| `thread.start()`                   | implicit (future starts immediately)                |
| `thread.join()`                    | `@future`                                           |
| `synchronized { }`                 | `(locking obj ...)`                                 |
| `volatile field`                   | `atom`                                              |
| `AtomicInteger`                    | `atom`                                              |
| `ConcurrentHashMap`                | `atom` holding a map                                |
| `CompletableFuture.supplyAsync(f)` | `(future (f))`                                      |
| `cf.thenApply(f)`                  | `(future (f @cf))`                                  |
| `cf.join()`                        | `@cf`                                               |
| `ExecutorService.submit(f)`        | `(future (f))`                                      |
| `Thread.ofVirtual().start(f)`      | Java interop `(.. Thread ofVirtual ...)`            |
| `new StructuredTaskScope()`        | `(let [f1 (future ...) f2 (future ...)] [@f1 @f2])` |

---

## Where to Go From Here

1. **[clojure.org/guides](https://clojure.org/guides/getting_started)** —
   the official guides are excellent. Start with "Getting Started" then
   read the "Learn Clojure" guide cover to cover.

2. **[Clojure for the Brave and True](https://www.braveclojure.com)** —
   the best free book, written with humor and practical Java-aware examples.

3. **[The Joy of Clojure](https://www.manning.com/books/the-joy-of-clojure)** —
   explains _why_ Clojure works the way it does. More important than any
   syntax guide.

4. **[ClojureDocs](https://clojuredocs.org)** — every standard library
   function documented with community examples. Bookmark this.

5. **[4Clojure / Exercism Clojure track](https://exercism.org/tracks/clojure)** —
   practice problems that teach idiomatic Clojure.

6. **[Clojurians Slack](https://clojurians.net)** — the main community.
   The `#beginners` channel is very welcoming to Java developers.

7. **Set up a proper REPL workflow** — install Calva (VS Code) or Cursive
   (IntelliJ) and force yourself to evaluate code from the editor. The
   REPL-driven workflow is the most important skill in Clojure. Don't skip it.

8. **Study Rich Hickey's talks** — these are mandatory:
   - _Simple Made Easy_ — the core philosophy behind Clojure
   - _The Value of Values_ — why immutability matters
   - _Are We There Yet?_ — identity vs state vs value
   - _Hammock Driven Development_ — designing systems

9. **Build a small web service** — use Ring + Reitit + next.jdbc.
   This is the most common Clojure stack and directly maps to your
   Spring Boot experience. You'll be surprised how little code it takes.

10. **Learn the data model deeply** — Clojure's persistent data structures,
    structural sharing, and the sequence abstraction are what make Clojure
    code idiomatic. Everything flows from understanding that maps, vectors,
    and sequences are the universal data model.

11. **Real Clojure projects to study** — reading production code is invaluable:
    - [Metabase](https://github.com/metabase/metabase) — analytics platform,
      large production Clojure codebase
    - [Jepsen](https://github.com/jepsen-io/jepsen) — distributed systems testing
    - [re-frame](https://github.com/day8/re-frame) — ClojureScript web framework
    - [Pedestal](https://github.com/pedestal/pedestal) — web framework
    - [Datomic](https://www.datomic.com) — Clojure-native database (closed source but influential)
