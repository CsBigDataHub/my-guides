# Java → Babashka: A Practical Guide

Babashka is a native Clojure scripting runtime built on GraalVM. It starts in milliseconds (vs. seconds for the JVM), ships as a single binary, and lets you write Clojure where you would otherwise write Bash or Python scripts. This guide is structured the same way as the companion *Java → Clojure* guide — side-by-side comparisons for every concept.

> **Who this is for:** Java developers who want to replace shell scripts, build automation, CLI tools, or DevOps tooling with real Clojure syntax — without the JVM startup penalty.

---

## 1. What Is Babashka?

| Feature | JVM Clojure | Babashka |
|---|---|---|
| Startup time | ~1-3 seconds | ~10-50 ms |
| Memory footprint | ~100-300 MB | ~30-60 MB |
| Compilation | JVM bytecode | GraalVM native image |
| Interpreter engine | Full Clojure compiler | SCI (Small Clojure Interpreter) |
| Ideal use case | Long-running apps, services | Scripts, CLI tools, automation |
| `deps.edn` / `bb.edn` | Yes | Yes (subset) |
| Java interop | Full | Limited (allow-listed classes) |
| Macros | Full | Most work; some edge cases |
| AOT compilation | Yes | No (interpreted) |

Babashka runs the same Clojure syntax you know — it just doesn't compile to bytecode. It uses the SCI interpreter, meaning a small subset of the JVM's capabilities are available, but almost all day-to-day scripting needs are covered.

---

## 2. Installation

### Java (via Maven wrapper)
```xml
<!-- pom.xml — needs JDK installed separately -->
<dependencies>
  <dependency>
    <groupId>org.clojure</groupId>
    <artifactId>clojure</artifactId>
    <version>1.12.0</version>
  </dependency>
</dependencies>
```

### Babashka
```bash
# macOS
brew install babashka

# Linux (one-liner)
bash < <(curl -s https://raw.githubusercontent.com/babashka/babashka/master/install)

# Windows (scoop)
scoop install babashka

# Verify
bb --version
# babashka v1.3.191
```

---

## 3. Hello World

### Java
```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
// $ javac HelloWorld.java && java HelloWorld
```

### Babashka
```clojure
#!/usr/bin/env bb
;; hello.clj
(println "Hello, World!")
;; $ chmod +x hello.clj && ./hello.clj
;; OR: bb hello.clj
```

No class, no method, no compilation step. The shebang line `#!/usr/bin/env bb` makes it a self-contained executable script.

---

## 4. Project Structure

### Java (Maven)
```
my-app/
├── pom.xml
└── src/main/java/com/example/
    └── App.java
```

### Babashka (`bb.edn`)
```
my-scripts/
├── bb.edn           ← Babashka project config (like pom.xml, but minimal)
├── src/
│   └── utils.clj
└── scripts/
    └── deploy.clj
```

**`bb.edn`**
```clojure
{:paths ["src" "scripts"]
 :deps  {org.clojure/data.csv {:mvn/version "1.1.0"}}
 :tasks {:clean  {:doc "Remove build artifacts"
                  :task (shell "rm -rf target")}
         :deploy {:doc "Deploy to production"
                  :depends [:clean]
                  :task (load-file "scripts/deploy.clj")}}}
```

Run tasks with `bb <task-name>`. This replaces `make`, shell scripts, or Bash-based CI tasks.

---

## 5. Variables and Data Types

### Java
```java
// Primitives and objects
int count = 42;
double ratio = 3.14;
boolean active = true;
String name = "ck";
long bigNum = 1_000_000L;

// Immutable (effectively final)
final int MAX = 100;
```

### Babashka
```clojure
;; Everything is a value, everything is immutable by default
(def count 42)
(def ratio 3.14)
(def active? true)       ; convention: ? suffix for booleans
(def name "ck")
(def big-num 1000000)    ; arbitrary precision by default

;; Local bindings (equivalent to final local vars)
(let [x 10
      y 20
      z (+ x y)]
  (println z))           ; → 30
```

---

## 6. Collections

### Java
```java
// List (mutable)
List<String> names = new ArrayList<>(List.of("Alice", "Bob", "Carol"));
names.add("Dave");

// Map (mutable)
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);

// Set
Set<String> tags = new HashSet<>(Set.of("java", "clojure", "babashka"));
```

### Babashka
```clojure
;; Vector (ordered, indexed — like ArrayList but immutable)
(def names ["Alice" "Bob" "Carol"])
(conj names "Dave")          ; → ["Alice" "Bob" "Carol" "Dave"] (new vector)

;; Map (like HashMap but immutable)
(def scores {"Alice" 95 "Bob" 87})
(assoc scores "Carol" 92)    ; → new map with Carol added
(get scores "Alice")         ; → 95
(:name {:name "ck" :age 30}) ; → "ck"  (keywords as functions)

;; Set
(def tags #{"java" "clojure" "babashka"})
(contains? tags "clojure")   ; → true
(conj tags "graalvm")        ; → new set

;; List (linked list — prefer vectors for most uses)
(def items '(1 2 3))
```

---

## 7. Control Flow

### Java
```java
// if/else
if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else {
    System.out.println("C");
}

// switch
switch (day) {
    case "Monday": System.out.println("Start of week"); break;
    case "Friday": System.out.println("End of week"); break;
    default:       System.out.println("Midweek");
}
```

### Babashka
```clojure
;; if/cond
(if (>= score 90)
  (println "A")
  (println "Not A"))

;; cond — replaces if/else if chains
(cond
  (>= score 90) "A"
  (>= score 80) "B"
  (>= score 70) "C"
  :else         "F")

;; case — replaces switch (constant values only)
(case day
  "Monday" "Start of week"
  "Friday" "End of week"
  "Midweek")

;; when — like if with no else branch (for side effects)
(when active?
  (println "System is active")
  (start-service!))
```

---

## 8. Functions

### Java
```java
// Regular method
public static int add(int a, int b) {
    return a + b;
}

// With default parameter (overloading)
public static String greet(String name) {
    return greet(name, "Hello");
}
public static String greet(String name, String prefix) {
    return prefix + ", " + name + "!";
}

// Lambda
Function<Integer, Integer> doubler = x -> x * 2;
```

### Babashka
```clojure
;; Regular function
(defn add [a b]
  (+ a b))

;; Multi-arity (default parameters)
(defn greet
  ([name]        (greet name "Hello"))
  ([name prefix] (str prefix ", " name "!")))

(greet "ck")           ; → "Hello, ck!"
(greet "ck" "Hi")      ; → "Hi, ck!"

;; Variadic (like varargs)
(defn sum [& nums]
  (apply + nums))

(sum 1 2 3 4 5)        ; → 15

;; Anonymous function (lambda)
(def doubler (fn [x] (* 2 x)))

;; Short lambda syntax
(def doubler #(* 2 %))
(def add      #(+ %1 %2))
```

---

## 9. Loops and Iteration

### Java
```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);

// for-each
for (int n : nums) {
    System.out.println(n * 2);
}

// Stream map + filter
List<Integer> result = nums.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 10)
    .collect(Collectors.toList());

// for loop with index
for (int i = 0; i < nums.size(); i++) {
    System.out.println(i + ": " + nums.get(i));
}
```

### Babashka
```clojure
(def nums [1 2 3 4 5])

;; map (transforms each element)
(map #(* 2 %) nums)            ; → (2 4 6 8 10)

;; filter + map (pipeline)
(->> nums
     (filter even?)
     (map #(* 10 %)))          ; → (20 40)

;; doseq — for side effects (like for-each)
(doseq [n nums]
  (println (* 2 n)))

;; map-indexed — with index
(map-indexed (fn [i v] (str i ": " v)) nums)
; → ("0: 1" "1: 2" "2: 3" "3: 4" "4: 5")

;; reduce — fold/accumulate
(reduce + 0 nums)              ; → 15
(reduce (fn [acc x] (assoc acc x (* x x))) {} nums)
; → {1 1, 2 4, 3 9, 4 16, 5 25}

;; for comprehension (like nested loops)
(for [x [1 2 3]
      y [10 20]
      :when (not= x 2)]
  (* x y))
; → (10 20 30 60)
```

---

## 10. String Manipulation

### Java
```java
String s = "  Hello, World!  ";
s.trim();                      // "Hello, World!"
s.toLowerCase();               // "  hello, world!  "
s.toUpperCase();               // "  HELLO, WORLD!  "
s.replace("World", "Babashka");
s.contains("Hello");           // true
s.split(", ");                 // ["  Hello", "World!  "]
String.format("Name: %s", "ck"); // "Name: ck"
s.startsWith("  Hello");       // true
```

### Babashka
```clojure
(require '[clojure.string :as str])

(def s "  Hello, World!  ")
(str/trim s)                          ; → "Hello, World!"
(str/lower-case s)                    ; → "  hello, world!  "
(str/upper-case s)                    ; → "  HELLO, WORLD!  "
(str/replace s "World" "Babashka")    ; → "  Hello, Babashka!  "
(str/includes? s "Hello")             ; → true
(str/split s #", ")                   ; → ["  Hello" "World!  "]
(format "Name: %s" "ck")             ; → "Name: ck"
(str/starts-with? s "  Hello")        ; → true
(str/join ", " ["a" "b" "c"])         ; → "a, b, c"
(str/split-lines "line1\nline2")      ; → ["line1" "line2"]

;; String interpolation (format is the idiom)
(let [name "ck" age 30]
  (format "%s is %d years old" name age))
```

---

## 11. Exception Handling

### Java
```java
try {
    int result = Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.err.println("Parse error: " + e.getMessage());
} finally {
    System.out.println("Always runs");
}

// Throw
throw new IllegalArgumentException("Invalid input: " + value);
```

### Babashka
```clojure
;; try/catch/finally
(try
  (Integer/parseInt "abc")
  (catch NumberFormatException e
    (println "Parse error:" (.getMessage e)))
  (finally
    (println "Always runs")))

;; Throw
(throw (ex-info "Invalid input" {:value value}))

;; ex-info creates a Clojure-idiomatic exception with a data map
(try
  (throw (ex-info "Something went wrong" {:code 404 :url "/api/users"}))
  (catch clojure.lang.ExceptionInfo e
    (let [data (ex-data e)]
      (println "Error:" (ex-message e))
      (println "Code:" (:code data)))))

;; Useful helpers
(ex-message e)   ; → the message string
(ex-data e)      ; → the data map passed to ex-info
(ex-cause e)     ; → the cause exception
```

---

## 12. Command-Line Arguments

This is one of Babashka's core strengths — it replaces shell script argument parsing.

### Java
```java
public static void main(String[] args) {
    if (args.length < 1) {
        System.err.println("Usage: app <name>");
        System.exit(1);
    }
    String name = args[0];
    System.out.println("Hello, " + name);
}
```

### Babashka (raw args)
```clojure
#!/usr/bin/env bb

(let [[name & rest] *command-line-args*]
  (when-not name
    (println "Usage: script.clj <name>")
    (System/exit 1))
  (println "Hello," name))

;; $ bb script.clj ck
;; → Hello, ck
```

### Babashka with `clojure.tools.cli` (batteries included)
```clojure
#!/usr/bin/env bb
(ns deploy
  (:require [clojure.tools.cli :refer [parse-opts]]))

(def cli-options
  [["-e" "--env ENV"    "Environment (dev/staging/prod)"
    :default "dev"
    :validate [#{"dev" "staging" "prod"} "Must be dev, staging, or prod"]]
   ["-p" "--port PORT"  "Port number"
    :default 8080
    :parse-fn #(Integer/parseInt %)
    :validate [#(< 0 % 65536) "Must be a valid port"]]
   ["-v" "--verbose"    "Verbose output"]
   ["-h" "--help"       "Show this help"]])

(let [{:keys [options errors summary]} (parse-opts *command-line-args* cli-options)
      {:keys [env port verbose help]} options]
  (cond
    help   (do (println "Deploy script") (println summary) (System/exit 0))
    errors (do (println "Errors:") (run! println errors) (System/exit 1))
    :else  (do
             (when verbose (println "Deploying to" env "on port" port))
             (println (format "Deployed! env=%s port=%d" env port)))))

;; $ bb deploy.clj --env prod --port 9000 --verbose
```

### Babashka CLI (`babashka.cli`) — modern style
```clojure
(ns tasks
  (:require [babashka.cli :as cli]))

(def spec
  {:env  {:desc "Environment" :default "dev" :alias :e}
   :port {:desc "Port"        :default 8080  :alias :p :coerce :int}})

(let [{:keys [env port]} (cli/parse-opts *command-line-args* {:spec spec})]
  (println (format "env=%s port=%d" env port)))

;; $ bb tasks.clj --env prod --port 9000
;; $ bb tasks.clj -e prod -p 9000   ← short aliases work too
```

---

## 13. File I/O

File I/O is a first-class use case for Babashka scripts.

### Read a text file

#### Java
```java
String content = Files.readString(Path.of("config.txt"));
```

#### Babashka
```clojure
(slurp "config.txt")
```

### Write a text file

#### Java
```java
Files.writeString(Path.of("output.txt"), "Hello");
```

#### Babashka
```clojure
(spit "output.txt" "Hello")
```

### Append to a file

#### Java
```java
Files.writeString(Path.of("app.log"), "new line\n",
    StandardOpenOption.APPEND, StandardOpenOption.CREATE);
```

#### Babashka
```clojure
(spit "app.log" "new line\n" :append true)
```

### Read lines

#### Java
```java
List<String> lines = Files.readAllLines(Path.of("data.txt"));
```

#### Babashka
```clojure
(str/split-lines (slurp "data.txt"))
;; Or lazily with a reader:
(with-open [rdr (clojure.java.io/reader "data.txt")]
  (doall (line-seq rdr)))
```

### Copy a file

#### Java
```java
Files.copy(Path.of("src.txt"), Path.of("dst.txt"),
           StandardCopyOption.REPLACE_EXISTING);
```

#### Babashka
```clojure
(require '[babashka.fs :as fs])

(fs/copy "src.txt" "dst.txt" {:replace-existing true})
```

### Move / rename a file

#### Java
```java
Files.move(Path.of("old.txt"), Path.of("new.txt"),
           StandardCopyOption.REPLACE_EXISTING);
```

#### Babashka
```clojure
(fs/move "old.txt" "new.txt" {:replace-existing true})
```

### Delete a file

#### Java
```java
Files.deleteIfExists(Path.of("temp.txt"));
```

#### Babashka
```clojure
(fs/delete-if-exists "temp.txt")
```

### Check existence, file/directory predicates

#### Java
```java
Files.exists(Path.of("config.edn"));
Files.isDirectory(Path.of("src"));
Files.isRegularFile(Path.of("app.jar"));
```

#### Babashka
```clojure
(fs/exists? "config.edn")
(fs/directory? "src")
(fs/regular-file? "app.jar")
(fs/readable? "secret.txt")
(fs/extension "report.csv")   ; → "csv"
(fs/file-name "src/utils.clj") ; → "utils.clj"
```

### Create directories

#### Java
```java
Files.createDirectories(Path.of("data/archive/2026"));
```

#### Babashka
```clojure
(fs/create-dirs "data/archive/2026")
```

### List directory contents

#### Java
```java
try (Stream<Path> paths = Files.list(Path.of("."))) {
    paths.filter(Files::isRegularFile)
         .forEach(System.out::println);
}
```

#### Babashka
```clojure
;; List files in a directory
(fs/list-dir ".")

;; Filter: only .clj files
(->> (fs/list-dir "src")
     (filter #(= (fs/extension %) "clj")))

;; Recursive glob
(fs/glob "." "**/*.clj")   ; all .clj files recursively
(fs/glob "src" "*.{clj,edn}")
```

### Temporary files

#### Java
```java
Path tmp = Files.createTempFile("prefix-", ".txt");
```

#### Babashka
```clojure
(fs/create-temp-file {:prefix "prefix-" :suffix ".txt"})
(fs/create-temp-dir {:prefix "work-"})
```

### Walk a directory tree

#### Java
```java
Files.walkFileTree(Path.of("."), new SimpleFileVisitor<>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
        System.out.println(file);
        return FileVisitResult.CONTINUE;
    }
});
```

#### Babashka
```clojure
(fs/walk-file-tree "."
  {:visit-file (fn [path _attrs]
                 (println path)
                 :continue)})

;; Or simpler with glob
(doseq [f (fs/glob "." "**/*" {:file-only true})]
  (println f))
```

### `babashka.fs` quick reference

```clojure
(fs/path "a" "b" "c")          ; → Path: a/b/c
(fs/absolutize "relative.txt") ; → absolute path
(fs/normalize "a/../b/./c")    ; → b/c
(fs/parent "src/utils.clj")    ; → src
(fs/size "big-file.bin")       ; → file size in bytes
(fs/last-modified-time "f")    ; → FileTime
(str (fs/path (fs/home) ".config/bb/")) ; → /home/ck/.config/bb/
```

---

## 14. Shell / Process Execution

Babashka excels at replacing Bash glue code.

### Java
```java
ProcessBuilder pb = new ProcessBuilder("git", "log", "--oneline", "-5");
pb.inheritIO();
Process p = pb.start();
p.waitFor();
```

### Babashka (`babashka.process`)
```clojure
(require '[babashka.process :as p])

;; Run and inherit I/O (like calling from shell)
(p/shell "git log --oneline -5")

;; Capture output as string
(-> (p/shell {:out :string} "git rev-parse HEAD")
    :out
    str/trim)

;; With args as a vector (safe — no shell injection)
(p/shell {:out :string} "git" "log" "--oneline" "-5")

;; Pipe commands
(-> (p/process "ls -la")
    (p/process "grep .clj")
    (p/process {:out :string} "wc -l")
    deref
    :out
    str/trim)

;; Check exit code
(let [result (p/shell {:out :string :err :string :continue true} "npm test")]
  (if (zero? (:exit result))
    (println "Tests passed!")
    (do (println "Tests failed:" (:err result))
        (System/exit 1))))

;; Environment variables
(p/shell {:env (merge (into {} (System/getenv))
                      {"MY_VAR" "hello"})}
         "printenv MY_VAR")
```

---

## 15. JSON

### Java
```java
// Using Jackson
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(Map.of("name", "ck", "age", 30));
Map<String, Object> obj = mapper.readValue(json, Map.class);
```

### Babashka (cheshire — built in)
```clojure
(require '[cheshire.core :as json])

;; Clojure map → JSON string
(json/generate-string {:name "ck" :age 30})
; → {"name":"ck","age":30}

;; Pretty-print
(json/generate-string {:name "ck" :age 30} {:pretty true})

;; JSON string → Clojure map
(json/parse-string "{\"name\":\"ck\",\"age\":30}")
; → {"name" "ck", "age" 30}   ← string keys

;; With keyword keys (usually what you want)
(json/parse-string "{\"name\":\"ck\"}" true)
; → {:name "ck"}

;; Parse a JSON file
(json/parse-string (slurp "config.json") true)

;; Write JSON to file
(spit "output.json"
      (json/generate-string {:result "ok"} {:pretty true}))
```

---

## 16. REST API Clients

### Java
```java
HttpClient client = HttpClient.newHttpClient();

// GET
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://api.github.com/users/octocat"))
    .header("Accept", "application/vnd.github+json")
    .GET()
    .build();
HttpResponse<String> res = client.send(req, BodyHandlers.ofString());
System.out.println(res.statusCode());
System.out.println(res.body());
```

### Babashka (`babashka.http-client` — built in since v1.1)
```clojure
(require '[babashka.http-client :as http])

;; GET
(def response (http/get "https://api.github.com/users/octocat"
                        {:headers {"Accept" "application/vnd.github+json"}}))
(:status response)   ; → 200
(:body response)     ; → JSON string

;; GET with query params
(http/get "https://api.example.com/search"
          {:query-params {:q "babashka" :per_page 10}})

;; Parse JSON response in one shot
(-> (http/get "https://api.github.com/users/octocat"
              {:headers {"Accept" "application/vnd.github+json"}})
    :body
    (json/parse-string true)
    :public_repos)

;; POST JSON
(http/post "https://api.example.com/users"
           {:headers {"Content-Type" "application/json"
                      "Authorization" "Bearer TOKEN"}
            :body (json/generate-string {:name "ck" :role "developer"})})

;; POST form data
(http/post "https://api.example.com/login"
           {:form-params {:username "ck" :password "secret"}})

;; PUT
(http/put "https://api.example.com/users/1"
          {:headers {"Content-Type" "application/json"}
           :body (json/generate-string {:name "ck-updated"})})

;; DELETE
(http/delete "https://api.example.com/users/1"
             {:headers {"Authorization" "Bearer TOKEN"}})

;; Error handling
(let [response (http/get "https://api.example.com/missing"
                         {:throw false})]   ; ← don't throw on 4xx/5xx
  (if (= 200 (:status response))
    (:body response)
    {:error true :status (:status response)}))
```

### Full API wrapper pattern
```clojure
(ns github-api
  (:require [babashka.http-client :as http]
            [cheshire.core :as json]
            [clojure.string :as str]))

(def base-url "https://api.github.com")
(def token (System/getenv "GITHUB_TOKEN"))

(defn api-get [path & [params]]
  (let [resp (http/get (str base-url path)
                       {:headers {"Accept"        "application/vnd.github+json"
                                  "Authorization" (str "Bearer " token)
                                  "X-GitHub-Api-Version" "2022-11-28"}
                        :query-params (or params {})
                        :throw false})]
    (if (<= 200 (:status resp) 299)
      (json/parse-string (:body resp) true)
      (throw (ex-info "GitHub API error"
                      {:status (:status resp)
                       :body   (json/parse-string (:body resp) true)})))))

;; Usage
(api-get "/users/octocat")
(api-get "/repos/babashka/babashka/issues" {:state "open" :per_page 5})
```

### Download a file via HTTP
```clojure
(let [resp (http/get "https://example.com/report.csv" {:as :bytes})]
  (with-open [out (clojure.java.io/output-stream "report.csv")]
    (.write out ^bytes (:body resp))))
```

---

## 17. Environment Variables and Config

### Java
```java
String dbUrl = System.getenv("DATABASE_URL");
String port  = System.getProperty("server.port", "8080");
```

### Babashka
```clojure
;; Environment variables
(System/getenv "DATABASE_URL")

;; With a default
(or (System/getenv "PORT") "8080")

;; All env vars as a map
(into {} (System/getenv))

;; Read a .env file (simple key=value format)
(->> (str/split-lines (slurp ".env"))
     (remove #(str/starts-with? % "#"))
     (remove str/blank?)
     (map #(str/split % #"=" 2))
     (into {}))

;; Read EDN config (idiomatic Clojure)
(clojure.edn/read-string (slurp "config.edn"))
```

---

## 18. Tasks (Babashka's `make` / build tool)

Babashka tasks in `bb.edn` replace `Makefile`, shell scripts, and build-tool glue code.

```clojure
;; bb.edn
{:tasks
 {:requires ([babashka.fs :as fs]
             [clojure.string :as str])

  clean     {:doc  "Remove build artifacts"
             :task (fs/delete-tree "target")}

  test      {:doc  "Run tests"
             :task (shell "clojure -M:test")}

  build     {:doc     "Build uberjar"
             :depends [clean test]
             :task    (shell "clojure -T:build uber")}

  lint      {:doc  "Run linter"
             :task (shell "clj-kondo --lint src")}

  deploy    {:doc     "Deploy to production"
             :depends [build]
             :task    (do
                        (println "Deploying version" (System/getenv "VERSION"))
                        (shell "scp target/app.jar user@server:/opt/app/"))}

  ;; Run a Clojure function directly
  gen-docs  {:doc  "Generate API docs"
             :task (load-file "scripts/gen_docs.clj")}}}
```

```bash
bb tasks          # list all tasks with docs
bb clean          # run clean
bb build          # runs clean → test → build in order
bb deploy         # full pipeline
```

---

## 19. Testing

### Java (JUnit 5)
```java
@Test
void testAdd() {
    assertEquals(5, Calculator.add(2, 3));
}
```

### Babashka (`clojure.test` — built in)
```clojure
;; test/math_test.clj
(ns math-test
  (:require [clojure.test :refer [deftest is are testing run-tests]]))

(defn add [a b] (+ a b))

(deftest test-add
  (is (= 5 (add 2 3)))
  (is (= 0 (add 0 0)))
  (is (= -1 (add 1 -2))))

(deftest test-add-cases
  (are [a b expected] (= expected (add a b))
    2  3  5
    0  0  0
    -1 1  0))

(deftest test-with-context
  (testing "positive numbers"
    (is (pos? (add 1 2))))
  (testing "negative result"
    (is (neg? (add -5 1)))))

;; Run directly
(run-tests)

;; Or from CLI
;; bb test/math_test.clj
```

---

## 20. Namespaces and Requires

### Java
```java
import java.util.List;
import java.util.stream.Collectors;
import com.example.utils.StringHelper;
```

### Babashka
```clojure
(ns my.script
  (:require [clojure.string         :as str]
            [clojure.java.io        :as io]
            [clojure.edn            :as edn]
            [cheshire.core          :as json]
            [babashka.fs            :as fs]
            [babashka.process       :as p]
            [babashka.http-client   :as http]
            [clojure.tools.cli      :refer [parse-opts]]
            [clojure.data.csv       :as csv]))

;; Load a local file dynamically (like a utility module)
(load-file "src/utils.clj")
```

---

## 21. Built-in Namespaces (Batteries Included)

Babashka ships with these ready to use — no `deps.edn` needed:

| Namespace | Description |
|---|---|
| `clojure.string` | String utilities |
| `clojure.java.io` | File I/O (readers, writers, streams) |
| `clojure.edn` | EDN parsing |
| `clojure.data.csv` | CSV parsing and generation |
| `clojure.data.json` | JSON (also `cheshire` is available) |
| `cheshire.core` | Full-featured JSON encoding/decoding |
| `babashka.fs` | File system operations (modern, idiomatic) |
| `babashka.process` | Shell out / subprocesses |
| `babashka.http-client` | HTTP client (since v1.1) |
| `babashka.cli` | CLI argument parsing (modern style) |
| `clojure.tools.cli` | CLI argument parsing (classic style) |
| `clojure.test` | Unit testing |
| `clojure.zip` | Zipper (tree navigation) |
| `hiccup.core` | HTML generation |
| `selmer.parser` | Jinja-like templating |
| `sci.core` | The SCI interpreter itself (for embedding) |

---

## 22. Adding External Dependencies

### Java
```xml
<!-- pom.xml -->
<dependency>
  <groupId>org.clojure</groupId>
  <artifactId>data.csv</artifactId>
  <version>1.1.0</version>
</dependency>
```

### Babashka (via `bb.edn`)
```clojure
;; bb.edn
{:deps {org.clojure/data.csv  {:mvn/version "1.1.0"}
        io.github.clojure/tools.deps.graph {:mvn/version "1.1.90"}}}
```

### Babashka (inline, no project file)
```clojure
#!/usr/bin/env bb
(require '[babashka.deps :as deps])
(deps/add-deps '{:deps {camel-snake-kebab/camel-snake-kebab {:mvn/version "0.4.3"}}})
(require '[camel-snake-kebab.core :as csk])
(csk/->kebab-case "HelloWorld")   ; → :hello-world
```

---

## 23. CSV

### Java
```java
// No stdlib CSV — needs Apache Commons CSV
CSVParser parser = CSVFormat.DEFAULT.withFirstRecordAsHeader()
    .parse(new FileReader("data.csv"));
for (CSVRecord record : parser) {
    System.out.println(record.get("name") + ": " + record.get("score"));
}
```

### Babashka (built in)
```clojure
(require '[clojure.data.csv :as csv]
         '[clojure.java.io  :as io])

;; Read CSV
(with-open [rdr (io/reader "data.csv")]
  (doall (csv/read-csv rdr)))
; → [["name" "score"] ["Alice" "95"] ["Bob" "87"]]

;; With headers as maps
(defn csv->maps [path]
  (with-open [rdr (io/reader path)]
    (let [[header & rows] (csv/read-csv rdr)]
      (mapv (partial zipmap (map keyword header)) rows))))

(csv->maps "data.csv")
; → [{:name "Alice" :score "95"} {:name "Bob" :score "87"}]

;; Write CSV
(with-open [wtr (io/writer "output.csv")]
  (csv/write-csv wtr [["name" "score"]
                       ["Alice" "95"]
                       ["Bob" "87"]]))
```

---

## 24. Concurrency

### Java
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
List<Future<Integer>> futures = items.stream()
    .map(item -> pool.submit(() -> processItem(item)))
    .collect(Collectors.toList());
```

### Babashka
```clojure
;; pmap — parallel map (uses thread pool automatically)
(defn process-item [x] (* x x))

(pmap process-item (range 100))
; runs in parallel across available cores

;; future — async computation
(def result (future (expensive-calculation)))
(println @result)   ; blocks until done

;; Thread-safe state
(def counter (atom 0))
(swap! counter inc)     ; thread-safe increment
@counter                ; read current value

;; Multiple futures
(let [f1 (future (fetch-data "api/users"))
      f2 (future (fetch-data "api/orders"))]
  {:users  @f1
   :orders @f2})
```

---

## 25. Differences Between Babashka and Full Clojure

These are the key limitations to be aware of:

| Feature | JVM Clojure | Babashka |
|---|---|---|
| Custom macros | Fully supported | Mostly supported; some edge cases |
| `gen-class` / `deftype` | Yes | No `gen-class`; `defrecord`/`deftype` work |
| Java interop | All classes | Allow-listed classes only |
| `clojure.reflect` | Yes | No |
| AOT compilation | Yes | No |
| `clojure.spec` | Yes (via dep) | `babashka/spec.alpha` |
| Core.async | Full | `org.babashka/core.async` |
| Protocols | Yes | Yes |
| Transducers | Yes | Yes |
| Lazy sequences | Yes | Yes |
| Agents | Yes | No |
| `require` dynamic | Yes | Yes |
| Namespace reload | `(require ... :reload)` | Same |
| Thread locals | Yes | Limited |

### When to use full Clojure instead

- Building a long-running HTTP server or microservice
- Heavy Java interop (custom classes, reflection)
- Libraries that rely on AOT compilation or `gen-class`
- CPU-bound workloads running > a few minutes (JIT pays off)

### When Babashka is the right tool

- Shell scripts longer than ~30 lines
- CI/CD pipeline tasks (build, deploy, notify)
- DevOps tooling (Kubernetes YAML generation, config transforms)
- File processing scripts (CSV transforms, JSON reshaping)
- REST API calls and webhook handling
- Replacing Bash + `jq` + `curl` pipelines
- Cross-platform `Makefile` replacement via `bb.edn` tasks

---

## 26. Quick-Reference Cheat Sheet

### Babashka built-in namespaces (import cheat sheet)
```clojure
(:require [clojure.string         :as str]   ; str/trim, str/split, str/join ...
          [clojure.java.io        :as io]    ; io/reader, io/writer, io/file ...
          [clojure.edn            :as edn]   ; edn/read-string
          [clojure.data.csv       :as csv]   ; csv/read-csv, csv/write-csv
          [cheshire.core          :as json]  ; json/parse-string, json/generate-string
          [babashka.fs            :as fs]    ; fs/copy, fs/move, fs/glob, fs/exists? ...
          [babashka.process       :as p]     ; p/shell, p/process
          [babashka.http-client   :as http]  ; http/get, http/post, http/put, http/delete
          [babashka.cli           :as cli]   ; cli/parse-opts
          [clojure.tools.cli      :refer [parse-opts]])  ; classic CLI parsing
```

### File I/O
```clojure
(slurp "f")                           ; read whole file
(spit "f" content)                    ; write whole file
(spit "f" content :append true)       ; append
(fs/copy "src" "dst")                 ; copy
(fs/move "old" "new")                 ; move/rename
(fs/delete-if-exists "f")             ; delete
(fs/exists? "f")                      ; check existence
(fs/list-dir ".")                     ; list directory
(fs/glob "." "**/*.clj")              ; recursive glob
(fs/create-dirs "a/b/c")              ; mkdir -p
```

### HTTP
```clojure
(http/get url)                        ; GET
(http/get url {:query-params {:k v}}) ; GET + query params
(http/post url {:body (json/generate-string m) :headers {...}})
(http/put  url {:body ...})
(http/delete url)
(-> resp :body (json/parse-string true)) ; parse JSON response
```

### Shell
```clojure
(p/shell "cmd arg1 arg2")             ; run, inherit I/O
(-> (p/shell {:out :string} "cmd") :out str/trim) ; capture output
(p/shell {:continue true} "may-fail") ; don't throw on non-zero exit
```

### JSON
```clojure
(json/parse-string s true)            ; → Clojure map (keyword keys)
(json/generate-string m)              ; → JSON string
(json/generate-string m {:pretty true}) ; → pretty JSON
```

### Common patterns
```clojure
*command-line-args*                   ; CLI args as a vector
(System/getenv "VAR")                 ; environment variable
(System/exit 1)                       ; exit with code
(edn/read-string (slurp "cfg.edn"))   ; read EDN config
(->> data (filter pred) (map f) ...)  ; data pipeline
```

---

## Appendix: Babashka vs Bash side-by-side

| Bash | Babashka |
|---|---|
| `cat file.txt` | `(slurp "file.txt")` |
| `echo "text" > file` | `(spit "file" "text")` |
| `cp src dst` | `(fs/copy "src" "dst")` |
| `mv old new` | `(fs/move "old" "new")` |
| `rm -f file` | `(fs/delete-if-exists "file")` |
| `mkdir -p a/b/c` | `(fs/create-dirs "a/b/c")` |
| `ls *.clj` | `(fs/glob "." "*.clj")` |
| `grep "pat" f` | `(filter #(str/includes? % "pat") (str/split-lines (slurp "f")))` |
| `curl -s url` | `(-> (http/get url) :body)` |
| `curl -s url \| jq .name` | `(-> (http/get url) :body (json/parse-string true) :name)` |
| `$1, $2, ...` | `(first *command-line-args*)` |
| `export VAR=val` | `(System/setProperty "VAR" "val")` |
| `cmd1 \| cmd2` | `(-> (p/process "cmd1") (p/process "cmd2") deref)` |
| `set -e` (exit on error) | `(when-not (zero? (:exit (p/shell ...))) (System/exit 1))` |
