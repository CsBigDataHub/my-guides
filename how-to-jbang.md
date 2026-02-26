# JBang: Complete Guide

---

## Installation

```bash
# Mac
brew install jbangdev/tap/jbang

# Linux / Mac (curl)
curl -Ls https://sh.jbang.dev | bash -s - app setup

# Windows (PowerShell)
iex "& { $(iex (New-Object Net.WebClient).DownloadString('https://ps.jbang.dev')) } app setup"

# Windows (Scoop)
scoop bucket add jbangdev https://github.com/jbangdev/scoop-bucket
scoop install jbang

# SDKMAN (cross platform)
sdk install jbang

# Verify
jbang --version
```

---

## Project Structure

```
my-scripts/
├── hello.java              # single file script
├── fetch_api.java          # single file with deps
├── complex/
│   ├── main.java           # multi-file script
│   └── helper.java
└── .jbang/
    └── jbang.properties    # project config (optional)
```

---

## The JBang Header - All Options

```java
///usr/bin/env jbang "$0" "$@" ; exit $?

// ─── JAVA VERSION ───────────────────────────────────────────
//JAVA 21
//JAVA 21+          // minimum version, use whatever is higher
//JAVA 17+

// ─── DEPENDENCIES (Maven coordinates) ───────────────────────
//DEPS com.google.code.gson:gson:2.10.1
//DEPS org.slf4j:slf4j-simple:2.0.9
//DEPS io.quarkus:quarkus-rest-client:3.6.0

// Multiple on one line
//DEPS com.google.guava:guava:32.1.3-jre, commons-io:commons-io:2.15.0

// ─── COMPILE OPTIONS ────────────────────────────────────────
//COMPILE_OPTIONS --enable-preview
//COMPILE_OPTIONS -parameters

// ─── RUNTIME OPTIONS (JVM flags) ────────────────────────────
//JAVA_OPTIONS -Xmx512m
//JAVA_OPTIONS -Xms256m -Xmx512m
//JAVA_OPTIONS --enable-preview
//JAVA_OPTIONS -Dfile.encoding=UTF-8

// ─── SOURCES (multi-file) ───────────────────────────────────
//SOURCES helper.java
//SOURCES utils/*.java
//SOURCES **/*.java

// ─── RESOURCE FILES ─────────────────────────────────────────
//FILES resource.properties
//FILES META-INF/MANIFEST.MF=manifest.mf

// ─── GAV (if publishing this script as artifact) ────────────
//GAV com.mycompany:my-tool:1.0.0

// ─── DESCRIPTION ────────────────────────────────────────────
//DESCRIPTION Fetches data from API and formats as table

void main() {
    System.out.println("Hello JBang!");
}
```

---

## Running Scripts

```bash
# Run a local file
jbang hello.java

# Run with arguments
jbang hello.java Alice 42

# Run directly from URL
jbang https://raw.githubusercontent.com/user/repo/main/hello.java

# Run from GitHub (shorthand)
jbang user/repo/hello.java

# Run from GitHub Gist
jbang https://gist.github.com/user/abc123def456

# Run specific version from Maven Central
jbang com.mycompany:my-cli-tool:1.0.0

# Run with JVM flags
jbang -Dmy.prop=value hello.java

# Run with extra classpath
jbang --cp extra.jar hello.java

# Verbose output (see what's happening)
jbang --verbose hello.java

# Force re-download dependencies
jbang --fresh hello.java
```

---

## Script Examples

### Hello World (Minimal)

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

void main() {
    System.out.println("Hello from JBang!");
}
```

```bash
chmod +x hello.java
./hello.java         # run directly like a shell script
# OR
jbang hello.java
```

---

### HTTP Client + JSON

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.google.code.gson:gson:2.10.1

import com.google.gson.*;
import java.net.http.*;
import java.net.URI;
import java.util.*;

record Todo(int id, String title, boolean completed) {}

void main(String[] args) throws Exception {
    var url = args.length > 0
        ? args[0]
        : "https://jsonplaceholder.typicode.com/todos";

    var client  = HttpClient.newHttpClient();
    var request = HttpRequest.newBuilder()
        .uri(URI.create(url))
        .header("Accept", "application/json")
        .build();

    var response = client.send(request, HttpResponse.BodyHandlers.ofString());

    if (response.statusCode() != 200) {
        System.err.println("Error: " + response.statusCode());
        System.exit(1);
    }

    var gson = new Gson();
    var todos = gson.fromJson(response.body(), Todo[].class);

    // Print as table
    System.out.printf("%-5s %-8s %s%n", "ID", "DONE", "TITLE");
    System.out.println("-".repeat(60));

    Arrays.stream(todos)
        .limit(10)
        .forEach(t -> System.out.printf(
            "%-5d %-8s %s%n",
            t.id(),
            t.completed() ? "✓" : "✗",
            t.title()
        ));
}
```

```bash
jbang todos.java
jbang todos.java https://jsonplaceholder.typicode.com/todos/1
```

---

### CLI Tool with PicoCLI

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS info.picocli:picocli:4.7.5
//DEPS com.google.code.gson:gson:2.10.1

import picocli.CommandLine;
import picocli.CommandLine.*;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.Callable;

@Command(
    name        = "fileinfo",
    description = "Display information about files",
    mixinStandardHelpOptions = true,  // adds --help and --version
    version     = "fileinfo 1.0"
)
class FileInfo implements Callable<Integer> {

    @Parameters(
        index       = "0",
        description = "File or directory to inspect"
    )
    Path path;

    @Option(
        names       = {"-r", "--recursive"},
        description = "Recurse into directories"
    )
    boolean recursive;

    @Option(
        names       = {"-e", "--extension"},
        description = "Filter by extension (e.g. .java)"
    )
    String extension;

    @Option(
        names       = {"-s", "--sort"},
        description = "Sort by: name, size, date",
        defaultValue = "name"
    )
    String sortBy;

    public static void main(String[] args) {
        int code = new CommandLine(new FileInfo()).execute(args);
        System.exit(code);
    }

    @Override
    public Integer call() throws Exception {
        if (!Files.exists(path)) {
            System.err.println("Path not found: " + path);
            return 1;
        }

        var stream = recursive
            ? Files.walk(path)
            : Files.list(path);

        var files = stream
            .filter(Files::isRegularFile)
            .filter(p -> extension == null || p.toString().endsWith(extension))
            .toList();

        // Sort
        var sorted = switch (sortBy) {
            case "size" -> files.stream()
                .sorted(Comparator.comparingLong(p -> p.toFile().length()))
                .toList();
            case "date" -> files.stream()
                .sorted(Comparator.comparingLong(p -> p.toFile().lastModified()))
                .toList();
            default -> files.stream()
                .sorted(Comparator.comparing(Path::toString))
                .toList();
        };

        // Display
        System.out.printf("%-50s %10s %s%n", "FILE", "SIZE", "MODIFIED");
        System.out.println("-".repeat(80));

        sorted.forEach(p -> {
            var file = p.toFile();
            System.out.printf("%-50s %10s %s%n",
                p.getFileName(),
                formatSize(file.length()),
                new java.util.Date(file.lastModified())
            );
        });

        System.out.printf("%nTotal: %d files%n", sorted.size());
        return 0;
    }

    String formatSize(long bytes) {
        if (bytes < 1024)        return bytes + "B";
        if (bytes < 1024 * 1024) return (bytes / 1024) + "KB";
        return (bytes / 1024 / 1024) + "MB";
    }
}
```

```bash
jbang fileinfo.java .
jbang fileinfo.java . --recursive --extension .java --sort size
jbang fileinfo.java --help
```

---

### Database Script (JDBC)

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.h2database:h2:2.2.224
//DEPS org.slf4j:slf4j-nop:2.0.9

import java.sql.*;
import java.util.*;

record Employee(int id, String name, String dept, double salary) {}

void main() throws Exception {
    // H2 in-memory database - great for scripts and testing
    var url = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";

    try (var conn = DriverManager.getConnection(url, "sa", "")) {
        setup(conn);
        seedData(conn);
        runQueries(conn);
    }
}

void setup(Connection conn) throws Exception {
    conn.createStatement().execute("""
        CREATE TABLE employees (
            id     INT PRIMARY KEY AUTO_INCREMENT,
            name   VARCHAR(100),
            dept   VARCHAR(50),
            salary DOUBLE
        )
        """);
}

void seedData(Connection conn) throws Exception {
    var sql = "INSERT INTO employees (name, dept, salary) VALUES (?, ?, ?)";
    try (var stmt = conn.prepareStatement(sql)) {
        var data = List.of(
            new Object[]{"Alice",   "Engineering", 95000},
            new Object[]{"Bob",     "Engineering", 88000},
            new Object[]{"Charlie", "Marketing",   72000},
            new Object[]{"Diana",   "Engineering", 102000},
            new Object[]{"Eve",     "Marketing",   68000}
        );

        for (var row : data) {
            stmt.setString(1, (String) row[0]);
            stmt.setString(2, (String) row[1]);
            stmt.setInt(3,    (Integer) row[2]);
            stmt.executeUpdate();
        }
    }
}

void runQueries(Connection conn) throws Exception {
    // All employees
    System.out.println("=== ALL EMPLOYEES ===");
    query(conn, "SELECT * FROM employees ORDER BY salary DESC")
        .forEach(e -> System.out.printf(
            "%-10s %-15s $%.0f%n",
            e.name(), e.dept(), e.salary()
        ));

    // Average salary by dept
    System.out.println("\n=== AVG SALARY BY DEPT ===");
    try (var rs = conn.createStatement().executeQuery("""
            SELECT dept, AVG(salary) as avg_sal, COUNT(*) as headcount
            FROM employees
            GROUP BY dept
            ORDER BY avg_sal DESC
            """)) {
        while (rs.next()) {
            System.out.printf("%-15s $%.0f  (%d people)%n",
                rs.getString("dept"),
                rs.getDouble("avg_sal"),
                rs.getInt("headcount")
            );
        }
    }
}

List<Employee> query(Connection conn, String sql) throws Exception {
    var result = new ArrayList<Employee>();
    try (var rs = conn.createStatement().executeQuery(sql)) {
        while (rs.next()) {
            result.add(new Employee(
                rs.getInt("id"),
                rs.getString("name"),
                rs.getString("dept"),
                rs.getDouble("salary")
            ));
        }
    }
    return result;
}
```

---

### Multi-File Script

```
project/
├── main.java
├── ApiClient.java
└── Formatter.java
```

```java
// main.java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.google.code.gson:gson:2.10.1
//SOURCES ApiClient.java
//SOURCES Formatter.java

void main(String[] args) throws Exception {
    var client    = new ApiClient("https://jsonplaceholder.typicode.com");
    var formatter = new Formatter();

    var data = client.get("/todos?_limit=5");
    System.out.println(formatter.toTable(data));
}
```

```java
// ApiClient.java
import java.net.http.*;
import java.net.URI;

public class ApiClient {
    private final String baseUrl;
    private final HttpClient client = HttpClient.newHttpClient();

    public ApiClient(String baseUrl) {
        this.baseUrl = baseUrl;
    }

    public String get(String path) throws Exception {
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .build();
        return client.send(request, HttpResponse.BodyHandlers.ofString()).body();
    }
}
```

```java
// Formatter.java
import com.google.gson.*;

public class Formatter {
    private final Gson gson = new GsonBuilder().setPrettyPrinting().create();

    public String toTable(String json) {
        var array = JsonParser.parseString(json).getAsJsonArray();
        var sb    = new StringBuilder();

        for (var element : array) {
            var obj = element.getAsJsonObject();
            obj.entrySet().forEach(e ->
                sb.append("%-15s: %s%n".formatted(e.getKey(), e.getValue()))
            );
            sb.append("-".repeat(40)).append("\n");
        }
        return sb.toString();
    }
}
```

```bash
jbang main.java
```

---

## JBang App - Install Scripts as Commands

```bash
# Install a script as a named command
jbang app install fileinfo.java

# Now run it from anywhere
fileinfo . --recursive

# Install from URL
jbang app install https://raw.githubusercontent.com/user/repo/main/tool.java

# Install with a specific name
jbang app install --name greet hello.java

# List installed apps
jbang app list

# Uninstall
jbang app uninstall fileinfo

# Update
jbang app update fileinfo
```

---

## JBang Init - Templates

```bash
# List available templates
jbang template list

# Create from template
jbang init hello.java                    # basic script
jbang init --template=cli mytool.java    # picocli CLI
jbang init --template=qcli mytool.java  # quarkus CLI
jbang init --template=hello hello.java  # hello world

# See all templates
jbang template list
```

---

## Dependency Management

```bash
# Search Maven Central from command line
jbang search gson
jbang search --limit 5 jackson

# Show dependency tree
jbang info classpath hello.java

# Export dependencies to local folder
jbang export local hello.java --output ./libs

# Export as fatjar (single executable jar)
jbang export fatjar hello.java --output hello.jar
java -jar hello.jar  # run the fatjar

# Export as native binary (requires GraalVM)
jbang export native hello.java
```

---

## Catalog - Share Collections of Scripts

```bash
# Create a catalog
# jbang-catalog.json in your git repo root

# Use scripts from a catalog
jbang alias@user/repo           # GitHub
jbang mytool@myorg/tools        # org repo

# Add alias to local catalog
jbang catalog add mytools https://github.com/me/my-jbang-tools/blob/main/jbang-catalog.json

# List catalogs
jbang catalog list

# Run from catalog
jbang tool@mytools
```

```json
// jbang-catalog.json - put this in your GitHub repo
{
  "aliases": {
    "hello": {
      "script-ref": "hello.java",
      "description": "Says hello"
    },
    "fileinfo": {
      "script-ref": "fileinfo.java",
      "description": "File information tool",
      "java-options": "-Xmx256m"
    }
  }
}
```

---

## Environment & Configuration

```bash
# Set default Java version
jbang config set java 21

# See all config
jbang config list

# Set custom Maven repo
jbang config set repo.myrepo https://my.nexus.com/repository/maven-public

# Proxy settings
jbang config set proxy.http  http://proxy:8080
jbang config set proxy.https https://proxy:8080

# Cache location
# Linux/Mac: ~/.jbang/cache
# Windows:   %USERPROFILE%\.jbang\cache

# Clear cache
jbang cache clear

# Clear specific dep cache
jbang cache clear --deps
```

---

## Using Snapshot & Custom Deps

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// Snapshot versions
//DEPS com.example:my-lib:1.0.0-SNAPSHOT
//REPOS mavencentral,snapshots=https://oss.sonatype.org/content/repositories/snapshots

// Local Maven repo (your own built jars)
//DEPS com.mycompany:my-local-lib:1.0.0
//REPOS mavencentral,local

// Private Nexus / Artifactory
//REPOS mycompany=https://nexus.mycompany.com/repository/maven-public

// With auth (use env vars - never hardcode!)
//REPOS myrepo=https://nexus.company.com/repo@env.NEXUS_USER:env.NEXUS_PASS

void main() {
    System.out.println("Using custom deps!");
}
```

---

## JBang in CI/CD

```yaml
# GitHub Actions
name: Run JBang Script
on: [push]

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup JBang
        uses: jbangdev/setup-jbang@main

      - name: Run script
        run: jbang hello.java

      - name: Run with args
        run: jbang process_data.java --input data.csv --output result.json
```

```dockerfile
# Dockerfile
FROM eclipse-temurin:21-jdk-alpine

# Install JBang
RUN curl -Ls https://sh.jbang.dev | bash -s - app setup
ENV PATH="/root/.jbang/bin:$PATH"

WORKDIR /app
COPY *.java .

# Pre-download dependencies (cache in layer)
RUN jbang --fresh fetch myapp.java

ENTRYPOINT ["jbang", "myapp.java"]
```

---

## Debugging & Troubleshooting

```bash
# See verbose output - what JBang is doing
jbang --verbose hello.java

# See generated code / classpath
jbang info classpath hello.java
jbang info tools hello.java

# Dry run - don't execute, just prepare
jbang --dry-run hello.java

# Debug mode - attach debugger on port 4004
jbang --debug hello.java
# Then in IntelliJ/VS Code: attach remote debugger to localhost:4004

# Custom debug port
jbang --debug=5005 hello.java

# Run with specific Java installation
jbang --java /usr/lib/jvm/java-21 hello.java

# Force fresh download (if dep seems corrupt)
jbang --fresh hello.java

# Show what java is being used
jbang jdk list
jbang jdk default 21
```

---

## JBang manages JDKs too

```bash
# List available JDKs to install
jbang jdk list --available

# Install a specific JDK (downloads automatically!)
jbang jdk install 21
jbang jdk install 17

# List installed JDKs
jbang jdk list

# Set default
jbang jdk default 21

# Use specific JDK for one run
jbang --java 17 hello.java

# Use Temurin, GraalVM etc
jbang jdk install 21-graalvm
jbang jdk install 21-temurin
```

---

## Quick Reference

```
COMMAND                              WHAT IT DOES
────────────────────────────────────────────────────────────────
jbang init hello.java                Create new script
jbang hello.java                     Run script
./hello.java                         Run directly (after chmod +x)
jbang hello.java arg1 arg2           Run with args
jbang --debug hello.java             Debug on port 4004
jbang --fresh hello.java             Force re-download deps
jbang --verbose hello.java           Verbose output
jbang export fatjar hello.java       Build executable jar
jbang export native hello.java       Build native binary
jbang app install hello.java         Install as system command
jbang app list                       List installed apps
jbang jdk list                       List JDKs
jbang jdk install 21                 Download JDK 21
jbang search gson                    Search Maven Central
jbang cache clear                    Clear all caches
jbang info classpath hello.java      Show resolved classpath

HEADER DIRECTIVES
────────────────────────────────────────────────────────────────
//JAVA 21                            Require Java 21
//DEPS group:artifact:version        Add Maven dependency
//SOURCES other.java                 Include other source files
//FILES config.properties            Include resource files
//JAVA_OPTIONS -Xmx512m             JVM flags at runtime
//COMPILE_OPTIONS --enable-preview   Compiler flags
//REPOS myrepo=https://...           Custom Maven repo
//DESCRIPTION My script              Script description
//GAV com.example:tool:1.0          Maven coordinates for publishing
```
