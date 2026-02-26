# Java 8 → 25: Complete Feature Guide

---

## Java Version Timeline & Features

### Java 8 (2014) - The Big One
```java
// Lambdas
Runnable r = () -> System.out.println("Hello");
Comparator<String> c = (a, b) -> a.compareTo(b);

// Method References
List<String> names = List.of("Alice", "Bob");
names.forEach(System.out::println);          // instance method ref
names.stream().map(String::toUpperCase);     // unbound method ref
names.stream().filter(Objects::nonNull);     // static method ref

// Streams
List<Integer> result = IntStream.rangeClosed(1, 10)
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .boxed()
    .collect(Collectors.toList());

// Optional
Optional<String> name = Optional.of("Alice");
String value = name
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .orElse("default");

// Default & Static interface methods
interface Greeter {
    String greet(String name);

    default String greetLoudly(String name) {
        return greet(name).toUpperCase();
    }

    static Greeter formal() {
        return name -> "Good day, " + name;
    }
}

// Date/Time API
LocalDate date = LocalDate.now();
LocalDateTime dt = LocalDateTime.of(2024, Month.JANUARY, 15, 10, 30);
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
Duration duration = Duration.between(dt, LocalDateTime.now());
```

---

### Java 9 (2017) - Modules & Collections
```java
// Module System (module-info.java)
module com.myapp {
    requires java.net.http;
    requires com.google.gson;
    exports com.myapp.api;
    opens com.myapp.model to com.google.gson;
}

// Collection Factory Methods
List<String> list = List.of("a", "b", "c");           // immutable
Set<Integer> set = Set.of(1, 2, 3);                   // immutable
Map<String, Integer> map = Map.of("a", 1, "b", 2);   // immutable
Map<String, Integer> map2 = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);

// Stream improvements
Stream.of(1, 2, null, 3)
    .takeWhile(n -> n != null)    // NEW
    .dropWhile(n -> n < 2)        // NEW
    .collect(Collectors.toList());

Stream.iterate(0, n -> n < 10, n -> n + 1)  // NEW: with predicate

// Optional improvements
optional.ifPresentOrElse(
    value -> System.out.println(value),
    () -> System.out.println("empty")
);
optional.stream()  // Optional -> Stream

// Private interface methods
interface MyInterface {
    default void publicMethod() {
        helper();  // can call private
    }

    private void helper() {
        System.out.println("shared logic");
    }
}
```

---

### Java 10 (2018) - Local Variable Type Inference
```java
// var keyword (local variables only)
var list = new ArrayList<String>();        // inferred: ArrayList<String>
var map = new HashMap<String, Integer>();  // inferred: HashMap<String, Integer>
var name = "Alice";                        // inferred: String

// Works in for loops
for (var entry : map.entrySet()) {
    System.out.println(entry.getKey() + "=" + entry.getValue());
}

// Works in try-with-resources
try (var stream = Files.newInputStream(path)) {
    // ...
}

// Does NOT work:
// var x;              // no initializer
// var x = null;       // can't infer from null
// var x = {1,2,3};   // array initializer not allowed
// class fields        // only local variables
```

---

### Java 11 (2018) - LTS
```java
// String methods
"  hello  ".strip();           // Unicode-aware trim
"  hello  ".stripLeading();
"  hello  ".stripTrailing();
"".isBlank();                  // true
"line1\nline2".lines()         // Stream<String>
    .collect(Collectors.toList());
"ha".repeat(3);                // "hahaha"

// var in lambda parameters (for annotations)
list.stream()
    .filter((@NotNull var s) -> s.length() > 3)
    .collect(Collectors.toList());

// HTTP Client (finalized from Java 9 incubator)
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Content-Type", "application/json")
    .GET()
    .build();

// Synchronous
HttpResponse<String> response = client.send(
    request, HttpResponse.BodyHandlers.ofString()
);
System.out.println(response.statusCode());
System.out.println(response.body());

// Async
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);

// Files utility methods
Path path = Path.of("/tmp/test.txt");
String content = Files.readString(path);
Files.writeString(path, "Hello World");
```

---

### Java 14 (2020) - Switch Expressions & Helpful NPE
```java
// Switch Expressions (finalized)
// Old style was statement, this is EXPRESSION (returns value)
String result = switch (day) {
    case MONDAY, TUESDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Other";
};

// With yield for multi-line cases
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    default -> {
        String s = day.toString();
        yield s.length();  // yield returns value from block
    }
};

// Helpful NullPointerException messages
// Old: "NullPointerException"
// New: "Cannot invoke String.length() because 'name' is null"
String name = null;
name.length();  // Now tells you EXACTLY what was null

// Records (PREVIEW in 14, final in 16)
record Point(int x, int y) {}
```

---

### Java 15 (2020) - Text Blocks
```java
// Text Blocks (finalized)
String json = """
        {
            "name": "Alice",
            "age": 30,
            "active": true
        }
        """;

String html = """
        <html>
            <body>
                <p>Hello %s</p>
            </body>
        </html>
        """.formatted("World");

// Indentation is stripped based on closing """
// The closing """ position matters:
String a = """
    hello
    """;   // "hello\n" - indent stripped

// Text block methods
String stripped = """
    line1
    line2
    """.stripIndent();

String escaped = "hello\nworld".translateEscapes();
```

---

### Java 16 (2021) - Records & Pattern Matching
```java
// Records (FINAL) - Immutable data carriers
record Person(String name, int age) {
    // Compact canonical constructor (validation)
    Person {
        Objects.requireNonNull(name);
        if (age < 0) throw new IllegalArgumentException("Age negative");
        name = name.trim();  // can normalize in compact constructor
    }

    // Custom constructor
    Person(String name) {
        this(name, 0);
    }

    // Additional methods
    boolean isAdult() {
        return age >= 18;
    }

    // Static members allowed
    static Person unknown() {
        return new Person("Unknown", -1);
    }
}

// Records auto-generate: constructor, getters, equals, hashCode, toString
Person p = new Person("Alice", 30);
p.name();   // "Alice"  (getter = field name, no "get" prefix)
p.age();    // 30

// Records implement interfaces
interface Printable { void print(); }
record LogEntry(String message, Instant time) implements Printable {
    public void print() { System.out.println(time + ": " + message); }
}

// Generic records
record Pair<A, B>(A first, B second) {}
Pair<String, Integer> pair = new Pair<>("hello", 42);

// instanceof Pattern Matching (FINAL)
Object obj = "Hello World";

// Old way:
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// New way:
if (obj instanceof String s) {
    System.out.println(s.length());  // s is scoped here
}

// With conditions (guarded):
if (obj instanceof String s && s.length() > 5) {
    System.out.println("Long string: " + s);
}
```

---

### Java 17 (2021) - LTS - Sealed Classes
```java
// Sealed Classes - restrict which classes can extend/implement
public sealed class Shape
    permits Circle, Rectangle, Triangle {}

public final class Circle extends Shape {
    private final double radius;
    Circle(double radius) { this.radius = radius; }
    double radius() { return radius; }
}

public final class Rectangle extends Shape {
    private final double width, height;
    Rectangle(double w, double h) { this.width = w; this.height = h; }
}

// Non-sealed allows further extension
public non-sealed class Triangle extends Shape {
    // anyone can extend Triangle
}

// Sealed interfaces
public sealed interface Expr
    permits Num, Add, Mul {}

record Num(int value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}

// Calculate expression (preview of what pattern matching enables)
int eval(Expr expr) {
    if (expr instanceof Num n) return n.value();
    if (expr instanceof Add a) return eval(a.left()) + eval(a.right());
    if (expr instanceof Mul m) return eval(m.left()) * eval(m.right());
    throw new AssertionError("unreachable");
}
```

---

### Java 21 (2023) - LTS - The BIG Release
```java
// ===== VIRTUAL THREADS =====
// Lightweight threads managed by JVM, not OS
// Millions of virtual threads vs thousands of OS threads

// Old way (OS thread per request - expensive)
Thread osThread = new Thread(() -> handleRequest());
osThread.start();

// Virtual thread
Thread vThread = Thread.ofVirtual().start(() -> handleRequest());

// Virtual thread per task executor (best for web servers)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {
        executor.submit(() -> {
            Thread.sleep(1000);  // blocks virtual thread, not OS thread
            return fetchFromDatabase();
        });
    }
}

// Structured Concurrency (Preview in 21)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser(id));
    Future<Order> order = scope.fork(() -> fetchOrder(id));

    scope.join();
    scope.throwIfFailed();

    return new Response(user.resultNow(), order.resultNow());
}

// ===== PATTERN MATCHING FOR SWITCH (FINAL) =====
sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double w, double h) implements Shape {}
record Triangle(double base, double height) implements Shape {}

double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.w() * r.h();
        case Triangle t  -> 0.5 * t.base() * t.height();
    };  // compiler knows all cases covered (sealed!) - no default needed
}

// Guarded patterns
String classify(Object obj) {
    return switch (obj) {
        case Integer i when i < 0  -> "negative int";
        case Integer i when i == 0 -> "zero";
        case Integer i             -> "positive int";
        case String s when s.isBlank() -> "blank string";
        case String s              -> "string: " + s;
        case null                  -> "null";   // can handle null!
        default                    -> "other";
    };
}

// ===== RECORD PATTERNS =====
// Destructure records in pattern matching
record Point(int x, int y) {}
record Line(Point start, Point end) {}

Object obj = new Line(new Point(1, 2), new Point(3, 4));

// Old way
if (obj instanceof Line l) {
    Point start = l.start();
    int x = start.x();
}

// Record pattern - destructures inline
if (obj instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
    System.out.println("from " + x1 + "," + y1 + " to " + x2 + "," + y2);
}

// In switch
String describe(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == y -> "on diagonal: " + x;
        case Point(int x, int y)             -> "point at " + x + "," + y;
        case Line(Point(var x, _), Point(var x2, _)) -> "horizontal line";
        default -> "unknown";
    };
}

// ===== SEQUENCED COLLECTIONS =====
// New interfaces: SequencedCollection, SequencedSet, SequencedMap
List<String> list = new ArrayList<>(List.of("a", "b", "c"));

list.getFirst();         // "a"
list.getLast();          // "c"
list.addFirst("z");      // ["z", "a", "b", "c"]
list.addLast("z");       // append
list.removeFirst();
list.removeLast();
list.reversed();         // reversed VIEW of collection

// Works on Deque, LinkedList, etc.
LinkedList<Integer> deque = new LinkedList<>(List.of(1, 2, 3));
deque.getFirst();
deque.reversed().forEach(System.out::println);  // 3, 2, 1

// SequencedMap
LinkedHashMap<String, Integer> map = new LinkedHashMap<>();
map.put("a", 1); map.put("b", 2); map.put("c", 3);
map.firstEntry();   // Map.Entry("a", 1)
map.lastEntry();    // Map.Entry("c", 3)
map.reversed();     // reversed view
```

---

### Java 22-24 (2024) - Incremental Improvements
```java
// ===== UNNAMED VARIABLES (Java 22, final) =====
// Use _ when you don't need the variable

// Old
try {
    riskyOperation();
} catch (Exception e) {   // e is never used
    log("failed");
}

// New
try {
    riskyOperation();
} catch (Exception _) {   // clearly intentional ignore
    log("failed");
}

// In patterns
switch (shape) {
    case Circle(var _) -> "a circle";   // don't need the radius
    case Rectangle(var w, var _) -> "width: " + w;
}

// In for loops
for (var _ : list) {
    count++;
}

// ===== UNNAMED CLASSES (Java 21 preview, Java 23 evolving) =====
// Simple scripts without boilerplate
// (see scripting section below)

// ===== STRING TEMPLATES (Preview Java 21, withdrawn/redesigned) =====
// Note: was previewed then withdrawn for redesign - NOT in Java 25 yet
// Tentative future syntax:
// String name = "Alice";
// String msg = STR."Hello \{name}";   // may look different when finalized

// ===== STREAM GATHERERS (Java 22 preview, Java 24 final) =====
// Custom intermediate stream operations
import java.util.stream.Gatherers;

// Built-in gatherers
List<List<Integer>> windows = Stream.of(1,2,3,4,5)
    .gather(Gatherers.windowFixed(3))    // [[1,2,3],[4,5]]
    .toList();

List<List<Integer>> sliding = Stream.of(1,2,3,4,5)
    .gather(Gatherers.windowSliding(3))  // [[1,2,3],[2,3,4],[3,4,5]]
    .toList();

Stream.of(1,2,3,4,5,6)
    .gather(Gatherers.fold(() -> 0, Integer::sum))  // running fold
    .toList();

Stream.of("a","b","c","d")
    .gather(Gatherers.scan(() -> "", (acc, s) -> acc + s))  // running concat
    .toList();  // ["a", "ab", "abc", "abcd"]

// Custom gatherer
Gatherer<Integer, ?, Integer> runningSum = Gatherer.of(
    () -> new int[]{0},           // initializer
    (state, element, downstream) -> {
        state[0] += element;
        return downstream.push(state[0]);
    }
);

Stream.of(1,2,3,4,5)
    .gather(runningSum)
    .toList();  // [1, 3, 6, 10, 15]

// ===== STRUCTURED CONCURRENCY (Java 21 preview -> 25 final?) =====
try (var scope = new StructuredTaskScope<String>()) {
    var t1 = scope.fork(() -> callServiceA());
    var t2 = scope.fork(() -> callServiceB());
    scope.join();

    String result = t1.get() + t2.get();
}

// ===== SCOPED VALUES (Java 21 preview -> evolving) =====
// Better alternative to ThreadLocal, works with virtual threads
static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.where(CURRENT_USER, user)
    .run(() -> {
        processRequest();  // can access CURRENT_USER.get() here
    });

// In nested method
void processRequest() {
    User user = CURRENT_USER.get();  // always available in this scope
    auditLog(user, "processed request");
}
```

---

### Java 25 (2025) - LTS
```java
// ===== PRIMITIVE TYPES IN GENERICS (Project Valhalla - if finalized) =====
// Historically generics only worked with reference types
// List<int> was impossible

// Future (tentative):
ArrayList<int> intList = new ArrayList<>();   // no boxing!
HashMap<String, double> scores = new HashMap<>();

// ===== VALUE CLASSES (Project Valhalla) =====
// Identity-free objects - pure data, no object identity
// Stored inline, not on heap as separate objects

value class Complex {
    double real;
    double imaginary;

    Complex(double real, double imaginary) {
        this.real = real;
        this.imaginary = imaginary;
    }

    Complex add(Complex other) {
        return new Complex(real + other.real, imaginary + other.imaginary);
    }
}
// NOTE: Java 25 exact features depend on final release - Valhalla features
// may still be in preview. Check JEPs for confirmed features.

// ===== FLEXIBLE CONSTRUCTOR BODIES (Java 25 likely final) =====
class Parent {
    int value;
    Parent(int value) { this.value = value; }
}

class Child extends Parent {
    Child(String s) {
        // Old: super() had to be FIRST statement
        // New: can do statements before super() as long as
        //      you don't access 'this'
        var parsed = Integer.parseInt(s);  // NEW: allowed before super()
        super(parsed);
        // now can use this
    }
}

// ===== MODULE IMPORT DECLARATIONS (Java 23 preview -> 25?) =====
// Import entire module at once
import module java.base;      // imports all exported packages
import module java.net.http;  // HttpClient etc all available

// ===== SIMPLE SOURCE FILES / UNNAMED CLASSES (evolving) =====
// See scripting section
```

---

## Pattern Matching - Complete Picture

```java
// The evolution of pattern matching across versions:

// Java 14-15: instanceof (preview)
// Java 16: instanceof (final)
// Java 17: switch (preview)
// Java 21: switch (final) + record patterns (final)

// All patterns in one example:
sealed interface Expr permits Num, Add, Mul, Neg {}
record Num(double value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}
record Neg(Expr expr) implements Expr {}

double eval(Expr expr) {
    return switch (expr) {
        // Record pattern with destructuring
        case Num(var v)              -> v;

        // Nested record patterns
        case Add(Num(var a), Num(var b)) -> a + b;        // specialized
        case Add(var l, var r)           -> eval(l) + eval(r);  // general

        case Mul(var l, var r)       -> eval(l) * eval(r);

        // Guarded pattern
        case Neg(Num(var v)) when v == 0 -> 0;            // -0 = 0
        case Neg(var e)              -> -eval(e);
    };
    // No default needed! Compiler verifies all sealed cases covered
}

// Usage:
eval(new Add(new Num(3), new Mul(new Num(4), new Num(5))));  // 23.0
```

---

## Java Scripting Complete Guide

### Setup & Basics

```bash
# Check your Java version
java --version

# Java 11+ supports single-file source execution
java Hello.java          # compile + run, no javac needed
java Hello.java arg1 arg2  # with arguments

# Shebang scripts (Unix/Mac/Linux)
#!/usr/bin/java --source 21
# or find java dynamically:
#!/usr/bin/env -S java --source 21
```

### Simple Script (Java 11+)

```java
#!/usr/bin/env -S java --source 21
// File: hello.java (make executable: chmod +x hello.java)

// Java 21+ unnamed classes: no class/main boilerplate needed (preview)
// Java 25: likely finalized

void main() {
    System.out.println("Hello from Java script!");

    var args_note = """
        NOTE: In unnamed classes you can define methods
        and call them directly without wrapping in a class
        """;

    greet("World");
}

void greet(String name) {
    System.out.printf("Hello, %s!%n", name);
}
```

### Old Style Single-File (Java 11)

```java
#!/usr/bin/env -S java --source 11
// File: OldStyle.java

public class OldStyle {
    public static void main(String[] args) {
        System.out.println("Old style - needs class + main");

        // But all Java features available!
        var list = java.util.List.of("a", "b", "c");
        list.stream()
            .map(String::toUpperCase)
            .forEach(System.out::println);
    }
}
```

### Real Script Examples

```java
#!/usr/bin/env -S java --source 21
///usr/bin/java --source 21 "$0" "$@"; exit $?
// The above line is an alternative shebang for some systems

import java.net.http.*;
import java.net.URI;
import java.nio.file.*;
import java.time.*;
import java.util.*;
import java.util.stream.*;

/**
 * Practical script: Fetch JSON from API and process it
 * Run: chmod +x fetch.java && ./fetch.java https://api.example.com
 */
void main(String[] args) throws Exception {
    if (args.length == 0) {
        System.err.println("Usage: fetch.java <url>");
        System.exit(1);
    }

    var url = args[0];
    var response = fetch(url);

    System.out.println("Status: " + response.statusCode());
    System.out.println("Body length: " + response.body().length());
    System.out.println(response.body());
}

HttpResponse<String> fetch(String url) throws Exception {
    var client = HttpClient.newHttpClient();
    var request = HttpRequest.newBuilder()
        .uri(URI.create(url))
        .header("Accept", "application/json")
        .timeout(Duration.ofSeconds(30))
        .build();
    return client.send(request, HttpResponse.BodyHandlers.ofString());
}
```

```java
#!/usr/bin/env -S java --source 21
// File: process_csv.java - Process a CSV file

import java.nio.file.*;
import java.util.*;
import java.util.stream.*;

record Person(String name, int age, String city) {
    static Person fromCsv(String line) {
        var parts = line.split(",");
        return new Person(parts[0].trim(),
                         Integer.parseInt(parts[1].trim()),
                         parts[2].trim());
    }
}

void main(String[] args) throws Exception {
    var file = args.length > 0 ? args[0] : "people.csv";

    var people = Files.lines(Path.of(file))
        .skip(1)  // skip header
        .filter(line -> !line.isBlank())
        .map(Person::fromCsv)
        .toList();

    // Stats using streams
    var avgAge = people.stream()
        .mapToInt(Person::age)
        .average()
        .orElse(0);

    var byCity = people.stream()
        .collect(Collectors.groupingBy(Person::city, Collectors.counting()));

    System.out.println("Total: " + people.size());
    System.out.printf("Average age: %.1f%n", avgAge);
    System.out.println("By city: " + byCity);

    // Find oldest per city
    people.stream()
        .collect(Collectors.groupingBy(
            Person::city,
            Collectors.maxBy(Comparator.comparingInt(Person::age))
        ))
        .forEach((city, person) ->
            System.out.println(city + ": " + person.map(Person::name).orElse("none"))
        );
}
```

```java
#!/usr/bin/env -S java --source 21
// File: watch.java - File watcher script

import java.nio.file.*;
import java.nio.file.attribute.*;

void main(String[] args) throws Exception {
    var dir = Path.of(args.length > 0 ? args[0] : ".");
    System.out.println("Watching: " + dir.toAbsolutePath());

    try (var watchService = FileSystems.getDefault().newWatchService()) {
        dir.register(watchService,
            StandardWatchEventKinds.ENTRY_CREATE,
            StandardWatchEventKinds.ENTRY_MODIFY,
            StandardWatchEventKinds.ENTRY_DELETE
        );

        while (true) {
            var key = watchService.take();  // blocks

            for (var event : key.pollEvents()) {
                var kind = event.kind();
                var filename = (Path) event.context();
                System.out.printf("[%s] %s: %s%n",
                    java.time.LocalTime.now(), kind.name(), filename);
            }

            if (!key.reset()) break;
        }
    }
}
```

### Script with Dependencies (JBang)

```bash
# JBang - the real solution for Java scripting with deps
# Install: curl -Ls https://sh.jbang.dev | bash -s - app setup
```

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.google.code.gson:gson:2.10.1
//DEPS org.slf4j:slf4j-simple:2.0.9

import com.google.gson.*;
import java.net.http.*;
import java.net.URI;
import java.util.*;

// With JBang: automatic dependency download!
// Run: jbang script.java  OR  ./script.java (after chmod +x)

void main() throws Exception {
    var client = HttpClient.newHttpClient();
    var request = HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/todos/1"))
        .build();

    var response = client.send(request, HttpResponse.BodyHandlers.ofString());

    var gson = new GsonBuilder().setPrettyPrinting().create();
    var json = gson.fromJson(response.body(), JsonObject.class);

    System.out.println("Title: " + json.get("title").getAsString());
    System.out.println("Done: " + json.get("completed").getAsBoolean());
}
```

### Passing Arguments & Environment

```java
#!/usr/bin/env -S java --source 21
import java.util.*;

void main(String[] args) {
    // Command line arguments
    System.out.println("Args: " + Arrays.toString(args));

    // Environment variables
    var home = System.getenv("HOME");
    var path = System.getenv().getOrDefault("MY_VAR", "default");

    // System properties (set with -D)
    // Run: java --source 21 script.java -Dmy.prop=value
    var prop = System.getProperty("my.prop", "not set");

    // Exit codes
    if (args.length == 0) {
        System.err.println("No args provided");
        System.exit(1);
    }
}
```

### Script with JVM Flags

```java
#!/usr/bin/env -S java --source 21 --enable-preview -Xmx512m
// Multiple flags in shebang
// --enable-preview: use preview features
// -Xmx512m: limit heap

void main() {
    // preview features available
    long mem = Runtime.getRuntime().maxMemory();
    System.out.println("Max memory: " + mem / 1024 / 1024 + "MB");
}
```

---

## Quick Reference: What to Use When

```
DATA CONTAINERS       → records
RESTRICT HIERARCHY    → sealed classes + permits
SAFE NULLS            → Optional
TYPE CHECKING         → instanceof pattern matching
COMPLEX BRANCHING     → pattern matching switch
CONCURRENT I/O        → virtual threads
SHARED STATE/THREADS  → scoped values (not ThreadLocal)
FUNCTIONAL PIPELINES  → streams + lambdas + method refs
CUSTOM STREAM OPS     → gatherers (Java 24+)
ORDERED COLLECTIONS   → sequenced collections API
SIMPLE SCRIPTS        → unnamed classes + void main()
SCRIPTS + DEPS        → JBang
```

---

## Key Mindset Shifts Since 2018

```
❌ Mutable POJOs with getters/setters   → ✅ Records
❌ class + main boilerplate for scripts → ✅ void main() / unnamed classes
❌ ThreadPoolExecutor for concurrency   → ✅ Virtual threads
❌ ThreadLocal for context passing      → ✅ ScopedValue
❌ null returns                         → ✅ Optional
❌ instanceof + cast                    → ✅ instanceof pattern matching
❌ complex if/else type dispatch        → ✅ pattern matching switch
❌ custom stream workarounds            → ✅ gatherers
❌ verbose switch statements            → ✅ switch expressions with ->
```

# Java 8→25: Before & After Examples

---

## Java 8: Lambdas & Method References

### Anonymous Classes → Lambdas

```java
// ❌ BEFORE (Java 7)
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("clicked");
    }
});

new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("running");
    }
}).start();
```

```java
// ✅ AFTER (Java 8)
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

Collections.sort(names, (a, b) -> a.compareTo(b));
// or even shorter:
names.sort(String::compareTo);

button.addActionListener(e -> System.out.println("clicked"));

new Thread(() -> System.out.println("running")).start();
```

---

### For Loops → Streams

```java
// ❌ BEFORE
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave");
List<String> result = new ArrayList<>();

for (String name : names) {
    if (name.length() > 3) {
        result.add(name.toUpperCase());
    }
}
Collections.sort(result);
```

```java
// ✅ AFTER
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

---

### Null Checks → Optional

```java
// ❌ BEFORE
public String getUpperCaseCity(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            String city = address.getCity();
            if (city != null) {
                return city.toUpperCase();
            }
        }
    }
    return "UNKNOWN";
}

// Caller
String city = getUpperCaseCity(user);
if (city != null) {
    System.out.println(city);
}
```

```java
// ✅ AFTER
public Optional<String> getUpperCaseCity(User user) {
    return Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .map(String::toUpperCase);
}

// Caller
getUpperCaseCity(user).ifPresentOrElse(
    System.out::println,
    () -> System.out.println("UNKNOWN")
);

// Or with default
String city = getUpperCaseCity(user).orElse("UNKNOWN");
```

---

### Manual Aggregations → Stream Collectors

```java
// ❌ BEFORE
List<Person> people = getPeople();

// Group by department
Map<String, List<Person>> byDept = new HashMap<>();
for (Person p : people) {
    String dept = p.getDepartment();
    if (!byDept.containsKey(dept)) {
        byDept.put(dept, new ArrayList<>());
    }
    byDept.get(dept).add(p);
}

// Count per department
Map<String, Integer> countByDept = new HashMap<>();
for (Map.Entry<String, List<Person>> entry : byDept.entrySet()) {
    countByDept.put(entry.getKey(), entry.getValue().size());
}

// Average salary per department
Map<String, Double> avgSalaryByDept = new HashMap<>();
for (Map.Entry<String, List<Person>> entry : byDept.entrySet()) {
    double total = 0;
    for (Person p : entry.getValue()) {
        total += p.getSalary();
    }
    avgSalaryByDept.put(entry.getKey(), total / entry.getValue().size());
}
```

```java
// ✅ AFTER
List<Person> people = getPeople();

Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));

Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.counting()
    ));

Map<String, Double> avgSalaryByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.averagingDouble(Person::getSalary)
    ));
```

---

## Java 9: Collection Factories

```java
// ❌ BEFORE
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
list = Collections.unmodifiableList(list);

Set<String> set = new HashSet<>();
set.add("x");
set.add("y");
set = Collections.unmodifiableSet(set);

Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
map.put("three", 3);
map = Collections.unmodifiableMap(map);
```

```java
// ✅ AFTER
List<String> list = List.of("a", "b", "c");
Set<String> set  = Set.of("x", "y");
Map<String, Integer> map = Map.of(
    "one", 1,
    "two", 2,
    "three", 3
);

// More than 10 entries use ofEntries
Map<String, Integer> big = Map.ofEntries(
    Map.entry("one",   1),
    Map.entry("two",   2),
    Map.entry("three", 3),
    Map.entry("four",  4)
    // ... unlimited entries
);
```

---

## Java 10: var

```java
// ❌ BEFORE - Redundant type declarations
ArrayList<Map<String, List<Integer>>> data = new ArrayList<Map<String, List<Integer>>>();
Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
HttpURLConnection connection = (HttpURLConnection) url.openConnection();

for (Map.Entry<String, List<Person>> entry : groupedPeople.entrySet()) {
    String department = entry.getKey();
    List<Person> employees = entry.getValue();
    // ...
}
```

```java
// ✅ AFTER
var data = new ArrayList<Map<String, List<Integer>>>();
var iterator = map.entrySet().iterator();
var connection = (HttpURLConnection) url.openConnection();

for (var entry : groupedPeople.entrySet()) {
    var department = entry.getKey();
    var employees  = entry.getValue();
    // ...
}

// Especially useful with streams
var result = people.stream()
    .filter(p -> p.getAge() > 18)
    .collect(Collectors.groupingBy(Person::getDepartment));
```

---

## Java 11: String Methods & HTTP Client

### String Methods

```java
// ❌ BEFORE
String s = "  hello  ";

// Check blank
boolean blank = s.trim().isEmpty();  // trim doesn't handle unicode whitespace

// Split to lines
String[] lines = text.split("\\r?\\n");

// Repeat
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 3; i++) sb.append("ha");
String repeated = sb.toString();

// Strip (had to use Apache Commons or similar)
String stripped = s.trim();  // not unicode-aware
```

```java
// ✅ AFTER
String s = "  hello  ";

boolean blank   = s.isBlank();         // unicode-aware
String stripped = s.strip();           // unicode-aware
String leading  = s.stripLeading();
String trailing = s.stripTrailing();

List<String> lines = text.lines().toList();

String repeated = "ha".repeat(3);      // "hahaha"
```

### HTTP Client

```java
// ❌ BEFORE (HttpURLConnection - painful)
URL url = new URL("https://api.example.com/users");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setRequestProperty("Accept", "application/json");
conn.setConnectTimeout(10000);

int status = conn.getResponseCode();

BufferedReader reader = new BufferedReader(
    new InputStreamReader(conn.getInputStream())
);
StringBuilder response = new StringBuilder();
String line;
while ((line = reader.readLine()) != null) {
    response.append(line);
}
reader.close();
conn.disconnect();

String body = response.toString();
```

```java
// ✅ AFTER
var client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(10))
    .build();

var request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Accept", "application/json")
    .GET()
    .build();

var response = client.send(request, HttpResponse.BodyHandlers.ofString());
String body = response.body();
int status   = response.statusCode();

// POST with body
var postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("""
        {"name": "Alice", "age": 30}
        """))
    .build();

// Async - doesn't block
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println)
    .join();
```

---

## Java 14: Switch Expressions

```java
// ❌ BEFORE - Switch statement, fall-through bugs waiting to happen
String result;
switch (day) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        result = "Weekday";
        break;           // forget this = bug
    case SATURDAY:
    case SUNDAY:
        result = "Weekend";
        break;
    default:
        result = "Unknown";
}

// Calculating with switch was painful
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        numLetters = 8;
        break;
    case WEDNESDAY:
        numLetters = 9;
        break;
    default:
        throw new IllegalArgumentException(day.toString());
}
```

```java
// ✅ AFTER - Switch expression, returns a value, no fall-through
String result = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY                             -> "Weekend";
};  // no default needed if all enum cases covered

int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY    -> 6;
    case TUESDAY                   -> 7;
    case THURSDAY, SATURDAY        -> 8;
    case WEDNESDAY                 -> 9;
};

// Multi-line with yield
int result2 = switch (day) {
    case MONDAY -> 1;
    default -> {
        System.out.println("processing: " + day);
        var value = day.ordinal() * 2;
        yield value;   // return value from block
    }
};
```

---

## Java 15: Text Blocks

```java
// ❌ BEFORE - String escaping nightmare
String json = "{\n" +
    "    \"name\": \"Alice\",\n" +
    "    \"age\": 30,\n" +
    "    \"roles\": [\"admin\", \"user\"]\n" +
    "}";

String html = "<html>\n" +
    "    <body>\n" +
    "        <h1>Hello " + name + "</h1>\n" +
    "        <p>Welcome back</p>\n" +
    "    </body>\n" +
    "</html>";

String sql = "SELECT u.name, u.email, o.total " +
    "FROM users u " +
    "JOIN orders o ON u.id = o.user_id " +
    "WHERE u.active = true " +
    "AND o.total > 100 " +
    "ORDER BY o.total DESC";
```

```java
// ✅ AFTER
String json = """
        {
            "name": "Alice",
            "age": 30,
            "roles": ["admin", "user"]
        }
        """;

String html = """
        <html>
            <body>
                <h1>Hello %s</h1>
                <p>Welcome back</p>
            </body>
        </html>
        """.formatted(name);

String sql = """
        SELECT u.name, u.email, o.total
        FROM users u
        JOIN orders o ON u.id = o.user_id
        WHERE u.active = true
        AND o.total > 100
        ORDER BY o.total DESC
        """;
```

---

## Java 16: Records

```java
// ❌ BEFORE - Boilerplate data class
public final class Person {
    private final String name;
    private final int age;
    private final String email;

    public Person(String name, int age, String email) {
        Objects.requireNonNull(name, "name required");
        if (age < 0) throw new IllegalArgumentException("age negative");
        this.name  = name;
        this.age   = age;
        this.email = email;
    }

    public String getName()  { return name; }
    public int    getAge()   { return age; }
    public String getEmail() { return email; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person p = (Person) o;
        return age == p.age &&
            Objects.equals(name,  p.name) &&
            Objects.equals(email, p.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }

    @Override
    public String toString() {
        return "Person[name=" + name +
               ", age=" + age +
               ", email=" + email + "]";
    }
}
```

```java
// ✅ AFTER - All of the above in ONE line
record Person(String name, int age, String email) {
    // Compact constructor for validation only
    Person {
        Objects.requireNonNull(name, "name required");
        if (age < 0) throw new IllegalArgumentException("age negative");
        email = email.toLowerCase();  // normalize
    }
}

// Usage is identical but accessors have no "get" prefix
Person p = new Person("Alice", 30, "Alice@Example.com");
p.name();   // "Alice"
p.age();    // 30
p.email();  // "alice@example.com"  (normalized)

// equals, hashCode, toString all work automatically
System.out.println(p); // Person[name=Alice, age=30, email=alice@example.com]

// Nesting records - great for API responses
record Address(String street, String city, String country) {}
record Order(String id, double total) {}
record UserProfile(Person person, Address address, List<Order> orders) {}
```

---

## Java 16: instanceof Pattern Matching

```java
// ❌ BEFORE - Cast after check (redundant)
Object obj = getShape();

if (obj instanceof Circle) {
    Circle c = (Circle) obj;        // redundant cast
    double area = Math.PI * c.getRadius() * c.getRadius();
    System.out.println("Circle area: " + area);
} else if (obj instanceof Rectangle) {
    Rectangle r = (Rectangle) obj; // redundant cast
    double area = r.getWidth() * r.getHeight();
    System.out.println("Rectangle area: " + area);
} else if (obj instanceof String) {
    String s = (String) obj;        // redundant cast
    System.out.println("String length: " + s.length());
}
```

```java
// ✅ AFTER - Pattern variable declared inline
Object obj = getShape();

if (obj instanceof Circle c) {
    double area = Math.PI * c.radius() * c.radius();
    System.out.println("Circle area: " + area);
} else if (obj instanceof Rectangle r) {
    double area = r.width() * r.height();
    System.out.println("Rectangle area: " + area);
} else if (obj instanceof String s && s.length() > 5) {
    // guard condition using the pattern variable directly
    System.out.println("Long string: " + s);
}

// Great for equals() implementations
@Override
public boolean equals(Object obj) {
    // ❌ BEFORE
    // if (!(obj instanceof Person)) return false;
    // Person other = (Person) obj;
    // return this.name.equals(other.name);

    // ✅ AFTER
    return obj instanceof Person other
        && this.name.equals(other.name)
        && this.age == other.age;
}
```

---

## Java 17: Sealed Classes

```java
// ❌ BEFORE - No way to restrict who implements your interface
// Anyone can add a new Shape anywhere in the codebase
public interface Shape {
    double area();
}

// SomeRandomDeveloper.java in another package
public class Hexagon implements Shape { ... }   // you had no idea
public class Blob     implements Shape { ... }  // this exists now

// Processing was risky - you could never know all subtypes
double process(Shape shape) {
    if (shape instanceof Circle) { ... }
    else if (shape instanceof Rectangle) { ... }
    else {
        throw new RuntimeException("Unknown shape: " + shape);
        // this could happen at runtime!
    }
}
```

```java
// ✅ AFTER - Sealed: you control ALL permitted subtypes
public sealed interface Shape
    permits Circle, Rectangle, Triangle {}

// These MUST be in the same package/module
public record Circle(double radius)         implements Shape {}
public record Rectangle(double w, double h) implements Shape {}
public record Triangle(double b, double h)  implements Shape {}

// Compiler knows ALL possible types
// No default needed - compiler verifies completeness
double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.w() * r.h();
        case Triangle t  -> 0.5 * t.b() * t.h();
    };
    // If you add Diamond to permits but forget here = COMPILE ERROR
    // Can't happen at runtime anymore
}
```

---

## Java 21: Pattern Matching for Switch

```java
// ❌ BEFORE - Chained instanceof checks
Object obj = getData();
String description;

if (obj instanceof Integer i) {
    if (i < 0) {
        description = "negative: " + i;
    } else if (i == 0) {
        description = "zero";
    } else {
        description = "positive: " + i;
    }
} else if (obj instanceof String s) {
    if (s.isBlank()) {
        description = "blank string";
    } else {
        description = "string: " + s.toUpperCase();
    }
} else if (obj instanceof List<?> list) {
    description = "list of size " + list.size();
} else if (obj == null) {
    description = "null value";
} else {
    description = "unknown: " + obj.getClass().getName();
}
```

```java
// ✅ AFTER - Pattern matching switch
String description = switch (obj) {
    case Integer i when i < 0  -> "negative: " + i;
    case Integer i when i == 0 -> "zero";
    case Integer i             -> "positive: " + i;
    case String s  when s.isBlank() -> "blank string";
    case String s               -> "string: " + s.toUpperCase();
    case List<?> list           -> "list of size " + list.size();
    case null                   -> "null value";
    default                     -> "unknown: " + obj.getClass().getName();
};
```

---

## Java 21: Record Patterns

```java
// ❌ BEFORE - Manual destructuring
record Point(int x, int y) {}
record Line(Point start, Point end) {}
record Circle(Point center, double radius) {}

Object shape = getShape();

if (shape instanceof Line l) {
    Point start = l.start();   // manual extraction
    Point end   = l.end();
    int x1 = start.x();        // manual extraction again
    int y1 = start.y();
    int x2 = end.x();
    int y2 = end.y();
    double length = Math.sqrt(Math.pow(x2-x1, 2) + Math.pow(y2-y1, 2));
    System.out.println("Line length: " + length);
}

if (shape instanceof Circle c) {
    Point center = c.center();
    int cx = center.x();
    int cy = center.y();
    System.out.println("Circle at " + cx + "," + cy);
}
```

```java
// ✅ AFTER - Record patterns destructure inline
// Variables are declared and bound in one step
if (shape instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
    double length = Math.sqrt(Math.pow(x2-x1, 2) + Math.pow(y2-y1, 2));
    System.out.println("Line length: " + length);
}

if (shape instanceof Circle(Point(var cx, var cy), var r)) {
    System.out.println("Circle at " + cx + "," + cy + " r=" + r);
}

// Shines in switch
String describe(Object shape) {
    return switch (shape) {
        // Destructure and guard in one case
        case Circle(Point(var x, var y), var r) when r > 100
            -> "big circle at %d,%d".formatted(x, y);

        case Circle(Point(var x, var y), var r)
            -> "circle at %d,%d r=%.1f".formatted(x, y, r);

        case Line(Point(0, 0), Point(var x, var y))
            -> "line from origin to %d,%d".formatted(x, y);

        case Line(Point(var x1, var y1), Point(var x2, var y2))
            -> "line from %d,%d to %d,%d".formatted(x1, y1, x2, y2);

        default -> "unknown shape";
    };
}
```

---

## Java 21: Virtual Threads

```java
// ❌ BEFORE - Thread per request, expensive OS threads
// A typical web server handling 10,000 concurrent requests
// needed 10,000 OS threads = ~80GB RAM just for thread stacks

ExecutorService executor = Executors.newFixedThreadPool(200); // hard limit

for (int i = 0; i < 10_000; i++) {
    executor.submit(() -> {
        // This BLOCKS an OS thread while waiting for DB
        String result = database.query("SELECT ...");  // blocks for 50ms
        // Thread is sleeping, wasting OS resources
        return processResult(result);
    });
}
// Most requests queued waiting for a thread - high latency!
```

```java
// ✅ AFTER - Virtual threads, millions of lightweight threads
// JVM parks virtual threads during blocking, reuses OS threads

ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

for (int i = 0; i < 1_000_000; i++) {  // 1 MILLION tasks - no problem
    executor.submit(() -> {
        // Virtual thread is PARKED (not blocking OS thread) during wait
        String result = database.query("SELECT ...");
        return processResult(result);
    });
}

// Spring Boot / web frameworks just need one config change:
// spring.threads.virtual.enabled=true
// That's it - your whole app uses virtual threads

// Structured approach - tasks tied to scope lifetime
void handleRequest(String userId) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        // Fork concurrent tasks
        var userFuture    = scope.fork(() -> fetchUser(userId));
        var ordersFuture  = scope.fork(() -> fetchOrders(userId));
        var profileFuture = scope.fork(() -> fetchProfile(userId));

        scope.join();           // wait for all
        scope.throwIfFailed();  // propagate any errors

        // All results available
        var user    = userFuture.resultNow();
        var orders  = ordersFuture.resultNow();
        var profile = profileFuture.resultNow();

        return new Dashboard(user, orders, profile);
    }
    // scope auto-closed: if ANY task fails, others are cancelled
}
```

---

## Java 21: Sequenced Collections

```java
// ❌ BEFORE - Inconsistent first/last access across collection types
List<String> list = new ArrayList<>(List.of("a", "b", "c"));

// Get first/last from List
String first = list.get(0);
String last  = list.get(list.size() - 1);  // verbose

// Add to front of List
list.add(0, "z");  // not obvious

// First/last from Deque was different API
Deque<String> deque = new LinkedList<>(List.of("a", "b", "c"));
String dequeFirst = deque.peekFirst();   // different method name
String dequeLast  = deque.peekLast();

// Reversed iteration was painful
List<String> reversed = new ArrayList<>(list);
Collections.reverse(reversed);          // mutates! need a copy first
for (String s : reversed) { ... }
```

```java
// ✅ AFTER - Consistent API via SequencedCollection interface
List<String> list = new ArrayList<>(List.of("a", "b", "c"));

// Uniform first/last for ALL sequenced collections
String first = list.getFirst();   // "a"
String last  = list.getLast();    // "c"

list.addFirst("z");   // ["z", "a", "b", "c"]
list.addLast("z");    // append
list.removeFirst();
list.removeLast();

// Non-mutating reversed VIEW
for (String s : list.reversed()) {  // no copy needed
    System.out.println(s);
}

// Works the same on LinkedList, TreeSet, LinkedHashMap etc.
LinkedHashMap<String, Integer> map = new LinkedHashMap<>();
map.put("a", 1); map.put("b", 2); map.put("c", 3);

Map.Entry<String,Integer> firstEntry = map.firstEntry(); // ("a", 1)
Map.Entry<String,Integer> lastEntry  = map.lastEntry();  // ("c", 3)
map.reversed().forEach((k, v) -> System.out.println(k + "=" + v));
```

---

## Java 22: Unnamed Variables

```java
// ❌ BEFORE - Forced to name variables you never use
try {
    loadConfig();
} catch (IOException e) {     // e is never referenced
    log.warn("Config not found, using defaults");
}

// In pattern matching
switch (event) {
    case UserCreated(var user, var timestamp, var requestId) ->
        // only need user, but must declare all
        sendWelcomeEmail(user);
}

// In for loops
int count = 0;
for (String ignored : largeList) {   // "ignored" naming convention hack
    count++;
}

// Decomposing - only want key
for (Map.Entry<String, List<Order>> entry : ordersByUser.entrySet()) {
    String key = entry.getKey();  // value never used
    process(key);
}
```

```java
// ✅ AFTER - _ signals intentionally unused
try {
    loadConfig();
} catch (IOException _) {    // clearly intentional
    log.warn("Config not found, using defaults");
}

// Only destructure what you need
switch (event) {
    case UserCreated(var user, var _, var _) ->
        sendWelcomeEmail(user);
}

// Clean counting loop
int count = 0;
for (var _ : largeList) {
    count++;
}

// Multiple _ is fine (they're not variables, no name conflict)
for (var _ : list1) {
    for (var _ : list2) {  // would be a compile error with any real name
        count++;
    }
}
```

---

## Java 24: Stream Gatherers

```java
// ❌ BEFORE - Custom intermediate operations were impossible without hacks
// Had to collect, transform, re-stream, or write complex Spliterators

// Sliding window - had to use index tricks
List<Integer> nums = List.of(1, 2, 3, 4, 5);
List<List<Integer>> windows = new ArrayList<>();
for (int i = 0; i <= nums.size() - 3; i++) {
    windows.add(nums.subList(i, i + 3));
}

// Running total - had to use external state (ugly with streams)
List<Integer> runningTotal = new ArrayList<>();
int sum = 0;
for (int n : nums) {
    sum += n;
    runningTotal.add(sum);
}

// First n distinct by key - no clean stream way
Map<String, Person> seen = new LinkedHashMap<>();
for (Person p : people) {
    seen.putIfAbsent(p.department(), p);
    if (seen.size() == 3) break;
}
List<Person> firstThreeDepts = new ArrayList<>(seen.values());
```

```java
// ✅ AFTER - Gatherers as custom intermediate stream operations
import java.util.stream.Gatherers;

// Built-in: sliding windows
List<List<Integer>> sliding = Stream.of(1,2,3,4,5)
    .gather(Gatherers.windowSliding(3))
    .toList();
// [[1,2,3], [2,3,4], [3,4,5]]

// Built-in: fixed windows (non-overlapping)
List<List<Integer>> chunks = Stream.of(1,2,3,4,5)
    .gather(Gatherers.windowFixed(2))
    .toList();
// [[1,2], [3,4], [5]]

// Built-in: running scan (like running total)
List<Integer> running = Stream.of(1,2,3,4,5)
    .gather(Gatherers.scan(() -> 0, Integer::sum))
    .toList();
// [1, 3, 6, 10, 15]

// Built-in: fold (reduce but as a stream)
Stream.of(1,2,3,4,5)
    .gather(Gatherers.fold(() -> 0, Integer::sum))
    .findFirst();  // Optional[15]

// Custom gatherer - first N distinct by key
Gatherer<Person, ?, Person> firstDistinctByDept =
    Gatherers.fold(
        LinkedHashMap<String,Person>::new,
        (map, person) -> { map.putIfAbsent(person.department(), person); return map; }
    )
    // ... (custom gatherers can be more complex, this shows the concept)

// Real custom gatherer
Gatherer<Integer, ?, Integer> runningMax = Gatherer.of(
    () -> new int[]{Integer.MIN_VALUE},          // state
    (state, element, downstream) -> {
        state[0] = Math.max(state[0], element);
        return downstream.push(state[0]);        // emit running max
    }
);

Stream.of(3, 1, 4, 1, 5, 9, 2, 6)
    .gather(runningMax)
    .toList();
// [3, 3, 4, 4, 5, 9, 9, 9]
```

---

## Putting It All Together: A Complete Before/After

### Real-World: User Order Processing

```java
// ❌ BEFORE (Java 7 style)
public class OrderProcessor {

    public Map<String, Double> getTopSpendersByRegion(
            List<Order> orders, String status) {

        Map<String, List<Order>> byRegion = new HashMap<>();
        for (Order order : orders) {
            if (order.getStatus().equals(status) &&
                order.getUser() != null &&
                order.getUser().getRegion() != null) {

                String region = order.getUser().getRegion();
                if (!byRegion.containsKey(region)) {
                    byRegion.put(region, new ArrayList<>());
                }
                byRegion.get(region).add(order);
            }
        }

        Map<String, Double> topSpenders = new HashMap<>();
        for (Map.Entry<String, List<Order>> entry : byRegion.entrySet()) {
            String region = entry.getKey();
            List<Order> regionOrders = entry.getValue();

            Order topOrder = null;
            double maxTotal = 0;
            for (Order order : regionOrders) {
                if (order.getTotal() > maxTotal) {
                    maxTotal = order.getTotal();
                    topOrder = order;
                }
            }

            if (topOrder != null) {
                topSpenders.put(region, maxTotal);
            }
        }

        return topSpenders;
    }
}
```

```java
// ✅ AFTER (Modern Java)

// Data modeled as records
record User(String name, String region) {}
record Order(String id, User user, double total, String status) {}

// Sealed result type - explicit success/failure
sealed interface ProcessResult permits Success, Failure {}
record Success(Map<String, Double> data) implements ProcessResult {}
record Failure(String reason)            implements ProcessResult {}

public class OrderProcessor {

    public ProcessResult getTopSpendersByRegion(
            List<Order> orders, String status) {

        if (orders == null || orders.isEmpty()) {
            return new Failure("no orders provided");
        }

        var result = orders.stream()
            .filter(o -> status.equals(o.status()))
            .filter(o -> o.user() != null && o.user().region() != null)
            .collect(Collectors.groupingBy(
                o -> o.user().region(),
                Collectors.maxBy(Comparator.comparingDouble(Order::total))
            ))
            .entrySet().stream()
            .filter(e -> e.getValue().isPresent())
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                e -> e.getValue().get().total()
            ));

        return new Success(result);
    }
}

// Caller uses pattern matching on the sealed result
var processor = new OrderProcessor();
switch (processor.getTopSpendersByRegion(orders, "COMPLETED")) {
    case Success(var data) ->
        data.forEach((region, total) ->
            System.out.printf("%s: $%.2f%n", region, total));
    case Failure(var reason) ->
        System.err.println("Failed: " + reason);
}
```

---

## Quick Cheat Sheet

```
FEATURE                    JAVA    BEFORE              AFTER
─────────────────────────────────────────────────────────────────
Lambdas                    8       new Runnable(){}    () -> ...
Method refs                8       x -> x.method()     Class::method
Streams                    8       for loops           .stream().filter().map()
Optional                   8       null checks         Optional.ofNullable()
Collection factories       9       add(), add(), add() List.of(a, b, c)
var                        10      Type name = ...     var name = ...
String methods             11      s.trim().isEmpty()  s.isBlank()
HTTP Client                11      HttpURLConnection   HttpClient.newHttpClient()
Switch expression          14      statement+break     -> with return value
Text blocks                15      "line\n" + "..."    """..."""
Records                    16      POJO + boilerplate  record Point(int x, int y){}
instanceof pattern         16      cast after check    instanceof Type t
Sealed classes             17      open interfaces     sealed + permits
Switch pattern matching    21      if/else instanceof  switch with case Type t
Record patterns            21      manual extraction   case Point(var x, var y)
Virtual threads            21      new Thread()        Thread.ofVirtual()
Sequenced collections      21      list.get(size-1)    list.getLast()
Unnamed variables          22      IOException e       IOException _
Stream gatherers           24      external state hack .gather(Gatherers.*)
```
