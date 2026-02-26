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


# Private Maven Repositories with JBang: Comprehensive Guide

---

## Why This Matters

Most JBang tutorials only show `mavencentral` dependencies. But in the real world, your company almost certainly has:

- An **internal Nexus or Artifactory** instance hosting proprietary libraries
- **Snapshot repositories** for pre-release artifacts
- **GitHub Packages** for org-level artifact sharing
- A **mirror** that proxies Maven Central for security/compliance reasons

Getting credentials wrong means your script silently falls back to Maven Central (and fails with "artifact not found"), or worse — leaks credentials into logs or source control.

This guide covers every approach, when to use each one, and the exact pitfalls to avoid.

---

## Understanding How JBang Resolves Dependencies

Before diving into configuration, it helps to know what JBang is doing under the hood.

JBang uses **Grape/Ivy** (not Maven) for dependency resolution, but it respects Maven conventions. When you write:

```java
//DEPS com.mycompany:internal-sdk:2.1.0
//REPOS myrepo=https://nexus.mycompany.com/repository/maven-public
```

JBang will:
1. Check its local cache (`~/.jbang/cache/`) first
2. Try each listed repository in order
3. Download and cache the artifact if found
4. Fail with a resolution error if nothing is found anywhere

The `//REPOS` line controls **where** JBang looks. The credentials control **whether** it can authenticate to get there.

```
Script Header         JBang             Nexus/Artifactory
─────────────         ─────             ─────────────────
//REPOS myrepo=... ──► resolves ──────► authenticates with
//DEPS com.x:y:1.0     coordinates      env vars / settings.xml
                       checks cache     downloads artifact
                       ◄─────────────── returns jar
```

---

## Approach 1: Environment Variables in Script Header

This is the **JBang-native approach**. Credentials never touch the script itself — they live in environment variables that JBang reads at runtime using the `@env.VAR` syntax.

### Basic Syntax

```java
//REPOS id=https://host/repo@env.USERNAME_VAR:env.PASSWORD_VAR
```

Breaking that down:
- `id` — arbitrary name, used internally for repo identity
- `https://host/repo` — the repository URL
- `@` — separator between URL and credentials
- `env.USERNAME_VAR` — reads from environment variable `USERNAME_VAR`
- `env.PASSWORD_VAR` — reads from environment variable `PASSWORD_VAR`

### Full Working Example

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// ─── REPOSITORIES ───────────────────────────────────────────────────────────
// Always include mavencentral explicitly when adding custom repos,
// otherwise JBang may not fall back to it for public dependencies.
//REPOS mavencentral,mycompany=https://nexus.mycompany.com/repository/maven-public@env.NEXUS_USER:env.NEXUS_PASS

// ─── DEPENDENCIES ───────────────────────────────────────────────────────────
// Public deps (resolved from mavencentral)
//DEPS com.google.code.gson:gson:2.10.1
//DEPS org.slf4j:slf4j-simple:2.0.9

// Private deps (resolved from mycompany repo)
//DEPS com.mycompany:internal-sdk:2.1.0
//DEPS com.mycompany:data-models:1.5.3

import com.mycompany.sdk.ApiClient;
import com.google.gson.Gson;

void main() {
    var client = new ApiClient("https://api.internal.mycompany.com");
    System.out.println("Connected: " + client.ping());
}
```

### Setting Up Environment Variables Locally

**Linux / macOS — temporary (current session only):**

```bash
export NEXUS_USER=john.smith
export NEXUS_PASS=supersecret123

# Verify they're set
echo $NEXUS_USER

# Now run your script
jbang myapp.java
```

**Linux / macOS — permanent (all sessions):**

```bash
# Add to your shell profile
# For bash:
echo 'export NEXUS_USER=john.smith'  >> ~/.bashrc
echo 'export NEXUS_PASS=supersecret123' >> ~/.bashrc
source ~/.bashrc

# For zsh (default on modern macOS):
echo 'export NEXUS_USER=john.smith'  >> ~/.zshrc
echo 'export NEXUS_PASS=supersecret123' >> ~/.zshrc
source ~/.zshrc
```

**Windows — PowerShell permanent:**

```powershell
# Set for current user (persists across sessions)
[Environment]::SetEnvironmentVariable("NEXUS_USER", "john.smith", "User")
[Environment]::SetEnvironmentVariable("NEXUS_PASS", "supersecret123", "User")

# Verify
[Environment]::GetEnvironmentVariable("NEXUS_USER", "User")

# Restart terminal or reload
$env:NEXUS_USER = [Environment]::GetEnvironmentVariable("NEXUS_USER", "User")
```

**Windows — Command Prompt:**

```cmd
:: Temporary
set NEXUS_USER=john.smith
set NEXUS_PASS=supersecret123

:: Permanent (requires admin for system-wide)
setx NEXUS_USER "john.smith"
setx NEXUS_PASS "supersecret123"
```

### What Happens If Variables Are Missing?

If `NEXUS_USER` or `NEXUS_PASS` aren't set, JBang will attempt an **unauthenticated request** to the repository. Depending on your Nexus/Artifactory configuration this will either:
- Return 401 Unauthorized → JBang fails with resolution error
- Return public artifacts only → private artifacts "not found"

You won't get a helpful "credentials missing" error by default. To catch this early, add a guard at the top of your script:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//REPOS mavencentral,mycompany=https://nexus.mycompany.com/repository/maven-public@env.NEXUS_USER:env.NEXUS_PASS
//DEPS com.mycompany:internal-sdk:2.1.0

void main() {
    // Guard: fail fast with a clear message if credentials missing
    var user = System.getenv("NEXUS_USER");
    var pass = System.getenv("NEXUS_PASS");

    if (user == null || user.isBlank()) {
        System.err.println("ERROR: NEXUS_USER environment variable is not set.");
        System.err.println("Run: export NEXUS_USER=your-username");
        System.exit(1);
    }
    if (pass == null || pass.isBlank()) {
        System.err.println("ERROR: NEXUS_PASS environment variable is not set.");
        System.err.println("Run: export NEXUS_PASS=your-password");
        System.exit(1);
    }

    // ... actual script logic
    System.out.println("Credentials present, proceeding...");
}
```

> **Note:** The guard runs *after* dependency resolution, so it won't prevent the resolution failure — but it will give your team a clear error message the next time they run the script without the right environment setup. For a pre-resolution check you need a wrapper shell script (see [Shell Wrapper Pattern](#shell-wrapper-pattern)).

---

## Approach 2: Maven settings.xml

Maven's `settings.xml` is the **most portable approach** — it works with Maven, Gradle, and JBang all at once, so if your team already uses Maven you may already have this set up.

JBang automatically reads `~/.m2/settings.xml` when resolving dependencies.

### File Location

```
~/.m2/settings.xml          ← user-level (recommended)
/etc/maven/settings.xml     ← system-level (all users on machine)
```

### Full settings.xml Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <!-- ─── SERVER CREDENTIALS ─────────────────────────────────────────────── -->
  <!-- The <id> here must match the <id> in your repositories section below   -->
  <servers>

    <server>
      <id>mycompany-releases</id>
      <username>john.smith</username>
      <password>supersecret123</password>
    </server>

    <server>
      <id>mycompany-snapshots</id>
      <username>john.smith</username>
      <password>supersecret123</password>
    </server>

    <!-- GitHub Packages uses your GitHub username + a Personal Access Token -->
    <server>
      <id>github-packages</id>
      <username>johngithub</username>
      <password>ghp_xxxxxxxxxxxxxxxxxxxx</password>
    </server>

  </servers>

  <!-- ─── PROFILES ────────────────────────────────────────────────────────── -->
  <profiles>
    <profile>
      <id>company-repos</id>
      <repositories>

        <repository>
          <id>mycompany-releases</id>
          <name>MyCompany Releases</name>
          <url>https://nexus.mycompany.com/repository/maven-releases</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>

        <repository>
          <id>mycompany-snapshots</id>
          <name>MyCompany Snapshots</name>
          <url>https://nexus.mycompany.com/repository/maven-snapshots</url>
          <releases>
            <enabled>false</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <!-- Check for new snapshots daily -->
            <updatePolicy>daily</updatePolicy>
          </snapshots>
        </repository>

        <repository>
          <id>github-packages</id>
          <name>GitHub Packages - MyOrg</name>
          <url>https://maven.pkg.github.com/myorg/myrepo</url>
        </repository>

      </repositories>
    </profile>
  </profiles>

  <!-- ─── ACTIVATE PROFILE ────────────────────────────────────────────────── -->
  <activeProfiles>
    <activeProfile>company-repos</activeProfile>
  </activeProfiles>

</settings>
```

### Using It in Scripts

When you have `settings.xml` configured, your script headers become **much cleaner** — no credentials, no repo URLs:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// No //REPOS needed at all — JBang picks up everything from settings.xml
//DEPS com.google.code.gson:gson:2.10.1
//DEPS com.mycompany:internal-sdk:2.1.0
//DEPS com.mycompany:data-models:1.5.3-SNAPSHOT

void main() {
    System.out.println("Resolved from private repo via settings.xml");
}
```

This is ideal for teams: everyone configures their own `settings.xml` locally with their own credentials, and the scripts stay credential-free and shareable.

### Encrypting Passwords in settings.xml

Storing plaintext passwords in `settings.xml` is uncomfortable. Maven has a built-in password encryption mechanism:

```bash
# Step 1: Create a master password
mvn --encrypt-master-password
# Enter your master password when prompted
# Outputs something like: {jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}

# Step 2: Store it in ~/.m2/settings-security.xml
cat > ~/.m2/settings-security.xml << 'EOF'
<settingsSecurity>
  <master>{jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}</master>
</settingsSecurity>
EOF

# Step 3: Encrypt your actual password
mvn --encrypt-password
# Enter your Nexus/Artifactory password when prompted
# Outputs: {COQLCE6DU6GtcS5P=}

# Step 4: Use the encrypted form in settings.xml
```

```xml
<server>
  <id>mycompany-releases</id>
  <username>john.smith</username>
  <password>{COQLCE6DU6GtcS5P=}</password>  <!-- encrypted -->
</server>
```

---

## Approach 3: JBang Global Config

JBang has its own config system that lives at `~/.jbang/jbang.properties`. This is the lightest-weight option for simple cases.

```bash
# Add a repo to global JBang config
jbang config set repo.mycompany https://nexus.mycompany.com/repository/maven-public

# This adds to ~/.jbang/jbang.properties:
# repo.mycompany=https://nexus.mycompany.com/repository/maven-public

# View all config
jbang config list

# Remove a repo
jbang config unset repo.mycompany
```

This approach **does not support credentials** — it's only useful for unauthenticated repositories (internal mirrors on a VPN, for example). For authenticated repos, use the `@env.` syntax or `settings.xml`.

```java
// Script using globally configured repo — no //REPOS header needed
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.mycompany:internal-lib:1.0.0

void main() {
    System.out.println("Repo comes from jbang global config");
}
```

---

## GitHub Actions: Complete Workflows

### Approach A: Environment Variables from Secrets

The simplest approach. Store credentials as GitHub secrets, pass them as env vars, use `@env.` syntax in the script.

**1. Add secrets to your repo:**

```
GitHub repo → Settings → Secrets and variables → Actions → New repository secret

NEXUS_URL  = https://nexus.mycompany.com/repository/maven-public
NEXUS_USER = ci-service-account
NEXUS_PASS = ci-token-or-password
```

**2. Workflow file:**

```yaml
# .github/workflows/run-script.yml
name: Run JBang Script

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:  # allow manual trigger

jobs:
  run:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup JBang
        uses: jbangdev/setup-jbang@main

      # Cache JBang dependency downloads between runs
      - name: Cache JBang dependencies
        uses: actions/cache@v4
        with:
          path: ~/.jbang/cache
          key: jbang-${{ runner.os }}-${{ hashFiles('**/*.java') }}
          restore-keys: |
            jbang-${{ runner.os }}-

      - name: Run script
        env:
          NEXUS_USER: ${{ secrets.NEXUS_USER }}
          NEXUS_PASS: ${{ secrets.NEXUS_PASS }}
        run: jbang myapp.java
```

**3. Script header:**

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//REPOS mavencentral,mycompany=https://nexus.mycompany.com/repository/maven-public@env.NEXUS_USER:env.NEXUS_PASS
//DEPS com.mycompany:internal-sdk:2.1.0

void main() {
    System.out.println("Running in CI with private deps!");
}
```

### Approach B: Generate settings.xml in Workflow

Better when you have multiple scripts or when you want to keep scripts completely credential-free.

```yaml
# .github/workflows/run-script.yml
name: Run JBang with Private Repo

on: [push, workflow_dispatch]

jobs:
  run:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup JBang
        uses: jbangdev/setup-jbang@main

      - name: Cache JBang dependencies
        uses: actions/cache@v4
        with:
          path: ~/.jbang/cache
          key: jbang-${{ runner.os }}-${{ hashFiles('**/*.java') }}
          restore-keys: |
            jbang-${{ runner.os }}-

      # Generate settings.xml from secrets
      # Using a dedicated step makes it easy to reuse across jobs
      - name: Configure Maven settings
        run: |
          mkdir -p ~/.m2
          cat > ~/.m2/settings.xml << 'SETTINGS_EOF'
          <?xml version="1.0" encoding="UTF-8"?>
          <settings>
            <servers>
              <server>
                <id>mycompany-releases</id>
                <username>NEXUS_USER_PLACEHOLDER</username>
                <password>NEXUS_PASS_PLACEHOLDER</password>
              </server>
              <server>
                <id>mycompany-snapshots</id>
                <username>NEXUS_USER_PLACEHOLDER</username>
                <password>NEXUS_PASS_PLACEHOLDER</password>
              </server>
            </servers>
            <profiles>
              <profile>
                <id>ci</id>
                <repositories>
                  <repository>
                    <id>mycompany-releases</id>
                    <url>NEXUS_URL_PLACEHOLDER/maven-releases</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>false</enabled></snapshots>
                  </repository>
                  <repository>
                    <id>mycompany-snapshots</id>
                    <url>NEXUS_URL_PLACEHOLDER/maven-snapshots</url>
                    <releases><enabled>false</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                  </repository>
                </repositories>
              </profile>
            </profiles>
            <activeProfiles>
              <activeProfile>ci</activeProfile>
            </activeProfiles>
          </settings>
          SETTINGS_EOF

          # Replace placeholders with actual secret values
          # This avoids secret values appearing in the heredoc above
          sed -i "s|NEXUS_USER_PLACEHOLDER|${{ secrets.NEXUS_USER }}|g" ~/.m2/settings.xml
          sed -i "s|NEXUS_PASS_PLACEHOLDER|${{ secrets.NEXUS_PASS }}|g" ~/.m2/settings.xml
          sed -i "s|NEXUS_URL_PLACEHOLDER|${{ secrets.NEXUS_URL }}|g"   ~/.m2/settings.xml

      - name: Run script
        run: jbang myapp.java

      - name: Run another script (same repo config applies)
        run: jbang process_data.java --input data.csv
```

> **Why the placeholder + sed approach?** GitHub Actions will automatically redact secret values from logs when they appear as `${{ secrets.X }}`. Using a heredoc directly with secrets embedded can sometimes cause issues with special characters (particularly `&`, `<`, `>`, `/` in passwords). The placeholder approach safely handles this.

### Approach C: GitHub Packages

When your private artifacts are published to GitHub Packages, you can use the built-in `GITHUB_TOKEN` — no extra secrets needed.

```yaml
# .github/workflows/run-script.yml
name: Run with GitHub Packages

on: [push]

jobs:
  run:
    runs-on: ubuntu-latest

    # Grant token permission to read packages
    permissions:
      contents: read
      packages: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup JBang
        uses: jbangdev/setup-jbang@main

      - name: Configure GitHub Packages access
        run: |
          mkdir -p ~/.m2
          cat > ~/.m2/settings.xml << EOF
          <settings>
            <servers>
              <server>
                <id>github</id>
                <username>${{ github.actor }}</username>
                <password>${{ secrets.GITHUB_TOKEN }}</password>
              </server>
            </servers>
          </settings>
          EOF

      - name: Run script
        run: jbang myapp.java
```

```java
// Script pointing to GitHub Packages
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// The repo id 'github' must match the server id in settings.xml
//REPOS mavencentral,github=https://maven.pkg.github.com/myorg/myrepo

//DEPS com.myorg:shared-library:1.0.0

void main() {
    System.out.println("Using artifact from GitHub Packages!");
}
```

### Approach D: Reusable Workflow for Multiple Repos

If you have many repos that all need the same private artifact access, extract it into a reusable workflow:

```yaml
# .github/workflows/setup-maven-auth.yml (reusable workflow)
name: Setup Maven Auth

on:
  workflow_call:
    secrets:
      NEXUS_URL:
        required: true
      NEXUS_USER:
        required: true
      NEXUS_PASS:
        required: true

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - name: Configure Maven settings
        run: |
          mkdir -p ~/.m2
          cat > ~/.m2/settings.xml << 'EOF'
          <settings>
            <servers>
              <server>
                <id>private</id>
                <username>USER_PLACEHOLDER</username>
                <password>PASS_PLACEHOLDER</password>
              </server>
            </servers>
            <profiles>
              <profile>
                <id>private</id>
                <repositories>
                  <repository>
                    <id>private</id>
                    <url>URL_PLACEHOLDER</url>
                  </repository>
                </repositories>
              </profile>
            </profiles>
            <activeProfiles><activeProfile>private</activeProfile></activeProfiles>
          </settings>
          EOF
          sed -i "s|USER_PLACEHOLDER|${{ secrets.NEXUS_USER }}|g" ~/.m2/settings.xml
          sed -i "s|PASS_PLACEHOLDER|${{ secrets.NEXUS_PASS }}|g" ~/.m2/settings.xml
          sed -i "s|URL_PLACEHOLDER|${{ secrets.NEXUS_URL }}|g"   ~/.m2/settings.xml
```

```yaml
# Any other workflow can now call it
jobs:
  run:
    uses: ./.github/workflows/setup-maven-auth.yml
    secrets:
      NEXUS_URL:  ${{ secrets.NEXUS_URL }}
      NEXUS_USER: ${{ secrets.NEXUS_USER }}
      NEXUS_PASS: ${{ secrets.NEXUS_PASS }}
```

---

## Snapshot Dependencies

Snapshots need special handling — they can change between builds, so you need to tell JBang where to find them and how often to check for updates.

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// Snapshot repo must be explicitly listed — not included in 'mavencentral'
//REPOS mavencentral,snapshots=https://nexus.mycompany.com/repository/maven-snapshots@env.NEXUS_USER:env.NEXUS_PASS

//DEPS com.mycompany:bleeding-edge-lib:2.0.0-SNAPSHOT

void main() {
    System.out.println("Using snapshot build!");
}
```

```bash
# Force JBang to re-check for newer snapshot version
# (otherwise it uses cached version even if a newer snapshot exists)
jbang --fresh myapp.java
```

In GitHub Actions, add `--fresh` to always pull the latest snapshot in CI:

```yaml
- name: Run with latest snapshot
  env:
    NEXUS_USER: ${{ secrets.NEXUS_USER }}
    NEXUS_PASS: ${{ secrets.NEXUS_PASS }}
  run: jbang --fresh myapp.java
```

---

## Shell Wrapper Pattern

For scripts that must validate credentials **before** attempting resolution, use a shell wrapper. This gives users a clear error message without waiting for a failed download:

```bash
#!/usr/bin/env bash
# run-myapp.sh

set -euo pipefail

# ─── CREDENTIAL CHECK ───────────────────────────────────────────────────────
missing=()

[[ -z "${NEXUS_USER:-}" ]] && missing+=("NEXUS_USER")
[[ -z "${NEXUS_PASS:-}" ]] && missing+=("NEXUS_PASS")

if [[ ${#missing[@]} -gt 0 ]]; then
    echo "❌ Missing required environment variables:"
    for var in "${missing[@]}"; do
        echo "   export $var=<value>"
    done
    echo ""
    echo "These are needed to access the private Maven repository."
    echo "Ask your team lead for the correct values."
    exit 1
fi

echo "✓ Credentials present"

# ─── RUN ────────────────────────────────────────────────────────────────────
exec jbang myapp.java "$@"
```

```bash
chmod +x run-myapp.sh
./run-myapp.sh --input data.csv
```

---

## Multiple Repositories

Real-world projects often need artifacts from several sources simultaneously:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21

// ─── REPOS ──────────────────────────────────────────────────────────────────
// JBang tries repos in the order listed.
// Put the most frequently used first for faster resolution.
//REPOS mavencentral,\
//  releases=https://nexus.mycompany.com/repository/maven-releases@env.NEXUS_USER:env.NEXUS_PASS,\
//  snapshots=https://nexus.mycompany.com/repository/maven-snapshots@env.NEXUS_USER:env.NEXUS_PASS,\
//  partner=https://repo.partner-company.com/maven@env.PARTNER_USER:env.PARTNER_PASS

// ─── DEPS ───────────────────────────────────────────────────────────────────
// Public
//DEPS com.google.code.gson:gson:2.10.1
//DEPS info.picocli:picocli:4.7.5

// Internal releases
//DEPS com.mycompany:internal-sdk:2.1.0
//DEPS com.mycompany:data-models:1.5.3

// Internal snapshot
//DEPS com.mycompany:new-feature-lib:3.0.0-SNAPSHOT

// Partner library
//DEPS com.partner:integration-kit:4.2.1

void main() {
    System.out.println("All deps resolved from mixed sources!");
}
```

---

## Troubleshooting

### Dependency Not Found

```bash
# Step 1: Run with verbose to see what repos JBang is actually trying
jbang --verbose myapp.java 2>&1 | grep -i "trying\|resolv\|error\|404\|401"

# Step 2: Check your resolved classpath
jbang info classpath myapp.java

# Step 3: Force fresh resolution (clear any bad cache)
jbang --fresh myapp.java

# Step 4: Clear the entire dep cache and retry
jbang cache clear --deps
jbang myapp.java
```

### 401 Unauthorized

This means JBang reached the repo but credentials were rejected.

```bash
# Verify env vars are actually set in current shell
echo "User: '$NEXUS_USER'"
echo "Pass length: ${#NEXUS_PASS}"

# Test credentials directly with curl
curl -u "$NEXUS_USER:$NEXUS_PASS" \
  "https://nexus.mycompany.com/repository/maven-public/com/mycompany/internal-sdk/2.1.0/internal-sdk-2.1.0.pom"

# Expected: XML content
# Got 401: wrong credentials
# Got 404: artifact doesn't exist at this path
```

### 403 Forbidden

The credentials are valid but the account doesn't have permission to access that repo. Contact your Nexus/Artifactory administrator to check repository permissions for your service account.

### SSL / Certificate Errors

Common in corporate environments with internal CAs:

```bash
# Option 1: Point JBang to a custom truststore
jbang --java-options "-Djavax.net.ssl.trustStore=/path/to/truststore.jks -Djavax.net.ssl.trustStorePassword=changeit" myapp.java

# Option 2: Add to script header
```

```java
//JAVA_OPTIONS -Djavax.net.ssl.trustStore=/etc/ssl/certs/company-truststore.jks
//JAVA_OPTIONS -Djavax.net.ssl.trustStorePassword=changeit
```

### Artifact Resolves Locally But Not in CI

Nine times out of ten, this is one of:
1. Secret not added to the repository (check Settings → Secrets)
2. Secret name typo (`NEXUS_PASS` vs `NEXUS_PASSWORD`)
3. The CI service account has different permissions than your personal account
4. `settings.xml` generated incorrectly (add a debug step to print it, masking the password)

```yaml
# Debug: print settings.xml with password masked
- name: Debug Maven settings
  run: |
    sed 's/<password>.*<\/password>/<password>***<\/password>/g' ~/.m2/settings.xml
```

---

## Security Checklist

```
✓  Credentials are in environment variables or settings.xml, never in script headers
✓  settings.xml is not committed to version control (.gitignore it)
✓  CI uses service accounts with minimal permissions (read-only to artifact repos)
✓  GitHub secrets are scoped to the environments that need them
✓  Rotate credentials regularly; update secrets when you do
✓  Don't use --verbose in CI logs when credentials are involved (it can expose URLs with embedded tokens)
✓  Use encrypted passwords in settings.xml on shared developer machines
```

---

## Summary: Which Approach to Use

| Situation | Recommended Approach | Why |
|---|---|---|
| Personal machine, one or two repos | `@env.` syntax in script header | Simple, explicit, no extra files |
| Team-shared scripts, company Nexus | `~/.m2/settings.xml` | Scripts stay credential-free and shareable |
| GitHub Actions, Nexus/Artifactory | Generate `settings.xml` in workflow step | Secrets stay in GitHub, scripts stay clean |
| GitHub Actions, GitHub Packages | `settings.xml` with `GITHUB_TOKEN` | No extra secrets needed |
| Unauthenticated internal mirror | `jbang config set repo.x ...` | Simplest option for auth-free repos |
| Multiple scripts in same repo | Reusable workflow + `settings.xml` | Configure once, use everywhere |
