# Externalized Configuration

- [How Configuration Values are used](#how-configuration-values-are-used)
- [PropertySource Order](#propertysource-order)
- [Accessing Command Line Properties](#accessing-command-line-properties)
- [JSON Application Properties](#json-application-properties)
- [External Application Properties](#external-application-properties)
    1. [Optional Location](#optional-location)
    2. [Wildcard Location](#wildcard-locations)
    3. [Importing Additional Data](#importing-additional-data)
    4. [Importing ExtensionLess Files](#importing-extensionless-files)
    5. [Using Environment Variables](#using-environment-variables)
    6. [Using Configuration Trees](#using-configuration-trees)
    7. [Property Placeholders](#property-placeholders)
    8. [Working With Multi-Document Files](#working-with-multi-document-files)
    9. [Activation Properties](#activation-properties)
- [Encrypting Properties](#encrypting-properties)
- [Working With YAML](#working-with-yaml)
    1. [Mapping YAML to Properties](#mapping-yaml-to-properties)
    2. [Direct Loading YAML](#directly-loading-yaml)
- [Configuring Random Values](#configuring-random-values)
- [Configuring System Environment Properties](#configuring-system-environment-properties)
- [Type-Safe Configuration Properties](#type-safe-configuration-properties)
    1. [JavaBean Property Binding](#javabean-properties-binding)
    2. [Constructor Binding](#constructor-binding)
    3. [Enabling @ConfigurationProperties-annotated Types](#enabling-configurationproperties-annotated-types)
    4. [Using @ConfigurationProperties-annotated Types](#using-configurationproperties-annotated-types)
    5. [Third-Party Configuration](#third-party-configuration)
    6. [Relax Binding](#relaxed-binding)
    7. [Merging Complex Types](#merging-complex-types)
    8. [Property Conversion](#properties-conversion)
    9. [@ConfigurationProperties Validation](#configurationproperties-validation)

Spring Boot lets you externalize your configuration so that you can work with the same application code in different environments.

configuration values (like URLs, usernames, passwords, ports, etc.) are often hardcoded or written in static config files.

This makes the application environment-dependent — meaning you’d have to modify code or files every time you move from development to production.

Spring Boot solves this problem using **Externalized Configuration**, allowing you to **externalize and override configuration values** without changing the code.

This means the **same JAR file** can run in multiple environments (dev, test, prod) just by changing configuration sources like:

- Properties or YAML files
- Environment variables
- Command-line arguments
- JSON or system properties

---

# How Configuration Values Are Used

Spring Boot provides **three ways** to access configuration values:

## a. Using `@Value`

Used for injecting single property values.

```java
@Value("${server.port}")
private int port;
```

## b. Using `Environment`

You can inject Spring’s `Environment` object and get values dynamically.

```java
@Autowired
private Environment env;

public void printPort() {
    System.out.println(env.getProperty("server.port"));
}
```

## c. Using `@ConfigurationProperties`

Used for binding multiple related properties into a structured object.

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppConfig {
    private String name;
    private String version;

    // getters and setters
}
```

And in `application.properties`:

```
app.name=MyApp
app.version=1.0.0
```

---

# PropertySource Order

Spring Boot has a **well-defined priority order** for loading configuration sources.

If the same property key (like `server.port`) is defined in multiple sources, **the one loaded later in the order overrides the previous ones**.

Here’s the full order explained in **detail** (from lowest to highest priority):

---

## **1. Default properties**

- You can set default values programmatically using:
    
    ```java
    SpringApplication app = new SpringApplication(MyApplication.class);
    Map<String, Object> defaults = new HashMap<>();
    defaults.put("server.port", "8080");
    app.setDefaultProperties(defaults);
    ```
    
- These act as fallback values if no other configuration source defines the property.

---

## **2. @PropertySource annotations**

- Used to load properties manually from a custom file.

Example:

```java
@Configuration
@PropertySource("classpath:custom.properties")
public class MyConfig {
}
```

**Note:** These properties are added **after** the application context is being refreshed,

so they cannot configure early-stage properties like `logging.level.*` or `spring.main.*` because those are read before refresh.

---

## **3. Config data (application.properties / application.yml)**

- These are the standard configuration files located in:
    - `src/main/resources/application.properties`
    - `src/main/resources/application.yml`
- You can also have profile-specific files:
    - `application-dev.properties`
    - `application-prod.yml`

Spring Boot loads these automatically based on the **active profile**:

```
spring.profiles.active=dev
```

---

## **4. RandomValuePropertySource**

- Spring Boot automatically provides random values that you can use, such as:
    
    ```
    my.secret=${random.value}
    my.number=${random.int}
    ```
    
- Useful for generating test data, tokens, etc.

---

## **5. OS environment variables**

- You can set configuration values directly from the system’s environment:
    
    ```bash
    export SERVER_PORT=9090
    java -jar app.jar
    ```
    
- Environment variables are very common in cloud or container deployments.

---

## **6. Java System properties**

- These come from the JVM, and can be passed using the `D` flag:
    
    ```bash
    java -Dserver.port=9090 -jar app.jar
    ```
    

---

## **7. JNDI attributes**

- Used mostly in traditional servlet containers like Tomcat or WebSphere.
- Properties from `java:comp/env` namespace can be read if available.

---

## **8. ServletContext init parameters**

- Used in traditional WAR-based applications, defined in `web.xml`.

---

## **9. ServletConfig init parameters**

- Also used in servlet-based configurations for specific servlets.

---

## **10. SPRING_APPLICATION_JSON**

- Spring Boot can read configuration in **JSON format** passed as:
    - Environment variable
        
        ```bash
        export SPRING_APPLICATION_JSON='{"server.port":9090, "app.name":"MyApp"}'
        ```
        
    - Or system property
        
        ```bash
        java -DSPRING_APPLICATION_JSON='{"server.port":9090}' -jar app.jar
        ```
        

---

## **11. Command-line arguments**

- Passed directly when running the app:
    
    ```bash
    java -jar app.jar --server.port=9090 --app.name=TestApp
    ```
    
- These have very **high priority** and can override almost any other property source.

---

## **12. Properties on your tests**

When running tests with `@SpringBootTest`, you can specify inline properties:

```java
@SpringBootTest(properties = {"app.name=TestApp", "server.port=9090"})
```

These override normal configurations during testing.

---

## **13. @DynamicPropertySource**

- Used in integration tests to programmatically set properties.
    
    Example:
    

```java
@DynamicPropertySource
static void dynamicProperties(DynamicPropertyRegistry registry) {
    registry.add("server.port", () -> 9090);
}
```

---

## **14. @TestPropertySource**

- Loads properties from specific files during test execution.

```java
@TestPropertySource("classpath:test-config.properties")
```

---

## **15. DevTools global settings**

- When using Spring Boot DevTools (for hot reloading), you can place properties in:
    
    ```
    $HOME/.config/spring-boot/devtools.properties
    ```
    
- These override local properties when DevTools is active.

---

## Property Override Example

Let’s assume `server.port` is defined in multiple places:

| Source | Value |
| --- | --- |
| `application.properties` | 8080 |
| OS environment variable | 9090 |
| Command-line argument | 7070 |

When you run:

```bash
java -jar myapp.jar --server.port=7070
```

Spring Boot will pick **7070**, because command-line arguments have the **highest priority**.

---

## Table (Lowest → Highest Priority)

| Priority | Source | Example |
| --- | --- | --- |
| 1 | Default properties | `setDefaultProperties()` |
| 2 | `@PropertySource` | `@PropertySource("custom.properties")` |
| 3 | Config data (`application.properties`, `.yml`) | in resources folder |
| 4 | RandomValuePropertySource | `${random.int}` |
| 5 | OS Environment Variables | `SERVER_PORT=9090` |
| 6 | Java System Properties | `-Dserver.port=9090` |
| 7 | JNDI Attributes | `java:comp/env` |
| 8 | ServletContext init params | web.xml |
| 9 | ServletConfig init params | servlet-level |
| 10 | `SPRING_APPLICATION_JSON` | JSON string |
| 11 | Command-line arguments | `--server.port=7070` |
| 12 | Test properties (`@SpringBootTest`) | `properties={...}` |
| 13 | `@DynamicPropertySource` | in test class |
| 14 | `@TestPropertySource` | load from file |
| 15 | DevTools global settings | `$HOME/.config/spring-boot` |

## What are Config Data Files?

In Spring Boot, the **config data files** are typically:

- `application.properties` or `application.yml`
- `application-{profile}.properties` or `application-{profile}.yml`

They define most of your application’s configuration (e.g., server port, database URL, logging, etc.).

Spring Boot loads these automatically without you writing any extra code.

---

### Where These Files Can Exist

There are **two possible locations** for these configuration files:

1. **Inside your packaged JAR/WAR**
    
    (Usually placed in `src/main/resources/`)
    
2. **Outside your JAR/WAR**
    
    (Placed in the same directory as the JAR, or in a custom config location)
    

---

### Loading Order (from lowest to highest priority)

Spring Boot loads configuration files in the **following order**.

If the same property (e.g., `server.port`) exists in multiple files, **the later one overrides** the earlier ones.

Let’s go through each in order:

---

### **Application properties inside your JAR**

**Files:**

- `application.properties`
- `application.yml`

**Location:**

Inside your project’s `src/main/resources/`, so when you build your JAR, they get packaged inside it.

**Example:**

```
src/
 └── main/
      └── resources/
           └── application.properties
```

**Purpose:**

These are the default configurations bundled with your app.

They apply when no profile is active or no external configurations are provided.

**Example content:**

```
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/defaultdb
```

---

### **Profile-specific properties inside your JAR**

**Files:**

- `application-dev.properties`
- `application-prod.properties`
- `application-test.yml`

**Location:**

Inside `src/main/resources/` as well.

**Example:**

```
src/
 └── main/
      └── resources/
           ├── application.properties
           ├── application-dev.properties
           └── application-prod.properties
```

**Purpose:**

Used for environment-specific configuration.

You activate them using the profile setting:

```
spring.profiles.active=dev
```

When you run with:

```bash
java -jar myapp.jar --spring.profiles.active=dev
```

Spring Boot will load:

1. `application.properties`
2. Then `application-dev.properties`

and properties in `application-dev.properties` **override** those in `application.properties`.

---

### **Application properties outside your JAR**

**Files:**

- `application.properties`
- `application.yml`

**Location:**

Same directory (or above) where you run your JAR.

**Example structure:**

```
/opt/myapp/
 ├── myapp.jar
 └── application.properties
```

**Purpose:**

To override internal (packaged) configurations **without modifying or rebuilding** your application.

This is extremely useful in production.

**Example:**

If you have inside JAR:

```
server.port=8080
```

and outside the JAR (in the same folder):

```
server.port=9090
```

then Spring Boot will use **9090**, because **external config files have higher priority**.

---

### **Profile-specific properties outside your JAR**

**Files:**

- `application-dev.properties`
- `application-prod.yml`

**Location:**

Outside your JAR in the same directory or in a configured `config/` folder.

**Example structure:**

```
/opt/myapp/
 ├── myapp.jar
 ├── application-prod.properties
 └── application.properties
```

**Purpose:**

These are the **highest priority config files**.

They override all other config data files — both inside and outside your JAR.

So if you start your app like:

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

Spring Boot loads in this order:

1. `application.properties` (inside JAR)
2. `application-prod.properties` (inside JAR)
3. `application.properties` (outside JAR)
4. `application-prod.properties` (outside JAR)

If the same property appears in all four, **the one from (4)** wins.

---

### Example of Complete Behavior

Let’s say you have the following files and values:

| Location | File Name | Property | Value |
| --- | --- | --- | --- |
| Inside JAR | `application.properties` | server.port | 8080 |
| Inside JAR | `application-dev.properties` | server.port | 8081 |
| Outside JAR | `application.properties` | server.port | 9090 |
| Outside JAR | `application-dev.properties` | server.port | 9091 |

If you run:

```bash
java -jar myapp.jar --spring.profiles.active=dev
```

Then Spring Boot loads files in this order and uses the **latest override**:

1. Internal `application.properties` → 8080
2. Internal `application-dev.properties` → 8081
3. External `application.properties` → 9090
4. External `application-dev.properties` → **9091 (final value)**

**Result:**

```
Server running on port 9091
```

---

### Why This Order Exists

The order reflects a **logical priority system**:

- Files **inside the JAR** are meant to be **default settings** (for developers).
- Files **outside the JAR** are meant to be **environment overrides** (for admins or deployers).

This way, you can deploy the same application JAR in multiple environments — dev, test, staging, prod — and simply use different external property files.

> Note:
> 
> - It is recommended to stick with one format for your entire application. If you have configuration files with both `.properties` and YAML format in the same location, `.properties` takes precedence.

# **Accessing Command Line Properties**

## What Are Command-Line Arguments in Spring Boot?

When you run a Spring Boot application using the command line, you can **pass configuration options directly** using the `--key=value` format.

These arguments are called **command-line properties**.

**Example:**

```bash
java -jar myapp.jar --server.port=9000 --spring.datasource.username=admin
```

Here:

- `-server.port=9000`
- `-spring.datasource.username=admin`

are **command-line arguments**, and Spring Boot automatically treats them as **property key-value pairs** and adds them to the **Spring Environment**.

---

## How Spring Boot Handles These Arguments

Spring Boot’s `SpringApplication` class automatically:

1. Parses all command-line arguments that start with `-`.
2. Converts them into property entries (key-value pairs).
3. Adds them to the **Spring Environment**.

Once added, these properties **behave like any other property** defined in `application.properties`, `application.yml`, or environment variables.

---

## Why Command-Line Properties Have the Highest Priority

Spring Boot has a **property source order**, and command-line properties appear **at the top of that list**.

This means if a property is defined both in your configuration file and as a command-line argument, the **command-line version will override** the file version.

Example

application.properties:

```
server.port=8080
```

**Run command:**

```bash
java -jar myapp.jar --server.port=9000
```

**Result:**

The application will start on port **9000**, because the command-line argument takes **precedence** over file-based properties.

---

## Disabling Command-Line Properties

Sometimes you **don’t want command-line arguments** to override your configuration files — for example:

- In **production**, where configurations must be fixed and not overridden accidentally.
- In **testing environments**, where you want reproducible setups.

In that case, you can **disable** this feature.

**How?**

Use the method:

```java
SpringApplication.setAddCommandLineProperties(false)
```

This tells Spring Boot **not** to add command-line properties to the environment.

---

### Example: With and Without `setAddCommandLineProperties(false)`

**(a) Default behavior (enabled)**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

Run with:

```bash
java -jar myapp.jar --server.port=9000
```

**Output:**

```
Tomcat started on port(s): 9000
```

---

**(b) Disabled command-line properties**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApp.class);
        app.setAddCommandLineProperties(false); // Disable command-line properties
        app.run(args);
    }
}
```

Run with the same command:

```bash
java -jar myapp.jar --server.port=9000
```

**Output:**

```
Tomcat started on port(s): 8080
```

Why 8080?

Because now `--server.port=9000` is **ignored**, and Spring Boot falls back to:

- The value from `application.properties`, or
- The default (8080) if no value is defined.

# **JSON Application Properties**

one of the more **advanced configuration features** in Spring Boot — using **`spring.application.json`** (or `SPRING_APPLICATION_JSON`) to provide configuration data as **JSON**.

## Why Do We Need `spring.application.json?`

Sometimes, **environment variables** or **system properties** have restrictions on:

- Which characters you can use (for example, `.` and `_` may not be allowed)
- How many variables you can set
- How deeply nested configurations can go

For example, you cannot easily define this as an environment variable in some systems:

```
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
```

because the `.` (dot) in `spring.datasource.url` may not be supported in some OS environments.

To solve this, **Spring Boot provides a single JSON-based property**, where you can encode **multiple properties in a single variable**.

This JSON will be **parsed automatically** and merged into the Spring `Environment`.

---

## How It Works

When your Spring Boot app starts:

- Spring looks for a property named `spring.application.json`
    
    (either as an environment variable, system property, or command-line argument)
    
- The value of this property is expected to be a **valid JSON object**
- Spring Boot parses it and converts it into **individual property key-value pairs**

For example:

```json
{"my":{"name":"test"}}
```

is interpreted as:

```
my.name = test
```

---

## Different Ways to Use `spring.application.json`

You can supply the JSON in **four ways**.

---

**(a) As an Environment Variable**

In Linux / macOS:

```bash
$ SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

In Windows (PowerShell):

```powershell
setx SPRING_APPLICATION_JSON "{ \"my\": { \"name\": \"test\" } }"
java -jar myapp.jar
```

**Result:**

When Spring Boot starts, it reads this environment variable and adds it to the Spring Environment as:

```
my.name = test
```

---

**(b) As a System Property**

You can also pass it as a **JVM system property** using `-D`:

```bash
$ java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

Spring Boot automatically detects it and sets:

```
my.name = test
```

---

**(c) As a Command-Line Argument**

You can also use it directly on the command line:

```bash
$ java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

Spring Boot treats this just like any other command-line property.

---

If you deploy to a classic application server (Tomcat, WebSphere, etc.),

you can define a JNDI variable:

```
java:comp/env/spring.application.json
```

with the same JSON value, and Spring Boot will read it from there.

---

## Accessing These Properties

Once Spring Boot loads the JSON, you can access the properties like any normal configuration:

Using `@Value`

```java
@Value("${my.name}")
private String name;
```

Using `Environment`

```java
@Autowired
private Environment env;

public void printValue() {
    System.out.println(env.getProperty("my.name"));
}
```

If you run with the example above:

```bash
SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

**Output:**

```
test
```

---

## More Complex Example

```bash
SPRING_APPLICATION_JSON='{"server":{"port":9090},"spring":{"datasource":{"url":"jdbc:mysql://localhost:3306/mydb","username":"admin"}}}' \
java -jar myapp.jar
```

This is equivalent to defining these properties:

```
server.port=9090
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=admin
```

**Result:**

- Server runs on port `9090`
- DataSource URL and username are configured from the JSON

---

## Handling of `null` Values in JSON

You can also specify **null values** in your JSON, for example:

```bash
SPRING_APPLICATION_JSON='{"my":{"name":null}}' java -jar myapp.jar
```

In this case:

- Spring Boot adds `my.name = null` to the environment.
- However, Spring treats **null properties as missing values**, meaning they **cannot override** properties defined in lower-priority sources (like files).

**Example:**

application.properties:

```
my.name=John
```

Command:

```bash
SPRING_APPLICATION_JSON='{"my":{"name":null}}' java -jar myapp.jar
```

**Result:**

`my.name` remains `"John"`, because `null` does not override an existing value.

# **External Application Properties**

Spring Boot looks for config files, **how** it orders them (precedence), and **how** you can change that behavior with `spring.config.name`, `spring.config.location`, and `spring.config.additional-location`.

### Default search locations and ordering (precedence)

When Spring Boot starts it builds a list of `PropertySource` objects from configuration files. By default it checks the following locations **(higher precedence first)**:

1. `file:./config/` (config/ subdir of current directory)
2. `file:./` (current directory)
3. `classpath:/config/` (config/ package on classpath)
4. `classpath:/` (root of classpath)

Within each directory it looks for files named with the “base name” (default `application`) and default extensions: `.properties`, `.yml` / `.yaml`. Profile-specific variants are also considered: `application-{profile}.properties` or `application-{profile}.yml`.

> Precedence note: later items in the above list override earlier ones (so file:./config/ overrides classpath:/), and a property defined in a higher-precedence file wins.
> 

---

### Example directory layout (local project)

```
myapp/
├─ config/
│  ├─ application.properties         <-- (A) file:./config/
│  └─ application-dev.properties     <-- (A-dev) file:./config/
├─ application.properties            <-- (B) file:./
├─ src/
│  └─ main/
│     └─ resources/
│        ├─ application.properties   <-- (C) classpath:/ (packaged)
│        └─ application-prod.yml     <-- (C-prod) classpath:/
```

Suppose the contents are:

- `src/main/resources/application.properties` (C)

```
app.name=MyApp
app.port=8080
app.mode=classpath
```

- `application.properties` in project root (B)

```
app.port=9000
app.mode=current-dir
```

- `config/application.properties` (A)

```
app.port=10000
app.mode=config-dir
```

**Which values are active at runtime (no profile)?**

- `app.name` → from classpath C (`MyApp`) — only defined there.
- `app.port` → from `file:./config/application.properties` (A): `10000` (because file:./config has highest precedence).
- `app.mode` → `config-dir`.

So final effective values are:

```
app.name=MyApp
app.port=10000
app.mode=config-dir
```

---

### Profile-specific config files

If you start the app with a profile active, Spring Boot also loads profile-specific files, e.g. `application-dev.properties`.

Command to activate `dev` profile (example):

```
java -jar myapp.jar --spring.profiles.active=dev
```

Spring Boot will (in addition to the default files) try to load `application-dev.properties` / `application-dev.yml` from the same set of locations. Values in profile files are merged and follow the same location precedence rules.

Using example layout above, if `config/application-dev.properties` defines `app.port=11000`, then when `dev` profile is active, `app.port` would be `11000` (overriding the non-profile `application.properties`).

---

### Changing the basename: `spring.config.name`

By default the basename is `application`. You can change it with `spring.config.name` so Spring looks for `myproject.properties` / `myproject.yml` etc.

Example — run:

```
java -jar myproject.jar --spring.config.name=myproject
```

Spring Boot now searches the same default locations but using `myproject` as the base name: `myproject.properties`, `myproject.yml`, `myproject-{profile}.yml`, etc.

---

### Explicit locations: `spring.config.location`

`spring.config.location` **replaces** the default search locations. It accepts comma-separated locations (files or directories). If you provide directories, they should end with `/` so Spring will append filenames based on base name.

Example — explicit files:

```
java -jar myapp.jar --spring.config.location=classpath:/default.properties,optional:file:./override.properties
```

In this case Spring will load exactly those files (and also expanded profile variants), not the usual default locations.

Example — explicit directories:

```
java -jar myapp.jar --spring.config.location=optional:classpath:/custom-config/,optional:file:./custom-config/
```

Spring will look for `custom-config/<basename>.properties` and `<basename>-<profile>.properties` in those directories.

Use `optional:` prefix when a location might not exist and you do not want startup to fail.

> Note:
> 
> 1. **Order of config locations**
>     
>     In `--spring.config.location=classpath:/default.properties,optional:file:./override.properties`,
>     
>     Spring Boot loads `classpath:/default.properties` **first**, then `./override.properties`.
>     
>     Later files **override** earlier ones.
>     
> 2. **If optional file is missing**
>     
>     Spring Boot **skips** it silently and continues startup — no error is thrown.
>     
> 3. **If non-optional file is missing**
>     
>     Spring Boot **fails to start** with a `FileNotFoundException`.
>     
> 4. **If both (default and optional) files are missing**
>     - If the first (non-optional) is missing → startup **fails**.
>     - If both are marked optional → startup **succeeds**, but no file-based configuration is loaded.
> 5. **If mandatory configs are in an optional file that’s missing**
>     
>     Spring Boot **starts**, skips the file, but later **fails at runtime** when those missing properties are required (e.g., DB URL).
>     
> 6. **Rule of thumb**
>     - Use `optional:` only for **non-critical** or **environment-specific override** files.
>     - Keep **mandatory** configurations in **non-optional** locations to ensure startup safety.

---

### Adding additional locations: `spring.config.additional-location`

If you want to **add** locations while keeping defaults, use `spring.config.additional-location`. Files in additional locations are processed *after* the default locations and therefore can **override** defaults.

Example:

```
java -jar myapp.jar --spring.config.additional-location=optional:file:./custom-config/
```

Search order becomes default locations first, then the paths in additional-location (so properties in `./custom-config/` can override defaults).

---

### Grouping locations (to control how profile-specific files are grouped)

If you have many locations and profile-specific files you want grouped together, use `;` to group locations (items separated by `;` are at the same level). This is rarely needed, but useful in complex setups.

Example (conceptual):

```
--spring.config.location="classpath:/base/;file:./base/,classpath:/override/,file:./override/"
```

This tells Spring which files are part of the same “group” when matching profile-specific files.

---

### Concrete override example with commands

Project has:

- `src/main/resources/application.properties` (packaged)
    
    `app.msg=from-classpath`
    
- `./application.properties`
    
    `app.msg=from-root`
    
- `./config/application.properties`
    
    `app.msg=from-config-dir`
    

Run without extra args:

```
java -jar myapp.jar
```

Result: `app.msg=from-config-dir` (because `./config/` wins).

Now run with an additional-location:

```
java -jar myapp.jar --spring.config.additional-location=file:./custom/
```

If `./custom/application.properties` contains `app.msg=from-custom`, that value wins (as it’s processed after default locations).

If you instead run:

```
java -jar myapp.jar --spring.config.location=file:./custom/
```

Then Spring will only look in `./custom/` (defaults are not used). If `./custom/application.properties` exists, that's used; otherwise if you put `optional:` prefix it won’t fail.

---

### How to read properties in code (examples)

Using `@Value`:

```java
@RestController
public class HelloController {
    @Value("${app.msg:default-message}")
    private String msg;

    @GetMapping("/msg")
    public String msg() {
        return msg;
    }
}
```

Using `@ConfigurationProperties`:

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String msg;
    // getters & setters
}
```

Both will pick up the final resolved value according to the precedence rules described.

---

### Profile-specific and environment overrides order (summary, highest → lowest precedence)

Spring Boot merges lots of sources. A simplified order (higher overrides lower) is:

1. `Devtools` global settings (if devtools active)
2. Command line arguments (`-key=value`)
3. `SPRING_APPLICATION_JSON` (env var)
4. OS environment variables
5. `application-{profile}.properties` and `application-{profile}.yml` from locations in the order we discussed (file:./config/ → file:./ → classpath:/config/ → classpath:/)
6. `application.properties` / `application.yml` from those locations
7. `spring.config.*` programmatic defaults and other lower precedence sources
8. **Embedded** defaults (packaged `application.properties` in `src/main/resources`)

(There are more subtle sources — e.g., `@PropertySource`, `System.getProperties()` / system properties, servlet config params, etc. — but the list above covers typical runtime overrides.)

Important practical rule: **command-line args > external files > packaged classpath files**.

## Optional Location

By default, when a specified config data location does not exist, Spring Boot will throw a `ConfigDataLocationNotFoundException` and your application will not start.

If you want to specify a location, but you do not mind if it does not always exist, you can use the `optional:` prefix. You can use this prefix with the `spring.config.location` and `spring.config.additional-location` properties, as well as with `spring.config.import` declarations.

For example, a `spring.config.import` value of `optional:file:./myconfig.properties` allows your application to start, even if the `myconfig.properties` file is missing.

To change that behavior globally, Spring Boot provides a property:

```
spring.config.on-not-found
```

`spring.config.on-not-found` tells Spring Boot **what to do when a config file is not found**.

Possible values:

- `fail` (default): throw `ConfigDataLocationNotFoundException` and stop startup.
- `ignore`: skip the missing file and continue startup normally.

So if you set `spring.config.on-not-found=ignore`, Spring Boot will **not fail** even if a required config file is missing.

### Example scenario

Suppose you run:

```
java -jar myapp.jar --spring.config.location=file:./important.properties
```

and the file `important.properties` **does not exist**.

By default, Spring Boot will stop startup with this error:

```
ConfigDataLocationNotFoundException: Config data location 'file:./important.properties' cannot be found
```

---

**How to fix it using** `spring.config.on-not-found`

You can tell Spring Boot to ignore missing config files:

Option 1 — as a command-line argument

```
java -jar myapp.jar \
  --spring.config.location=file:./important.properties \
  --spring.config.on-not-found=ignore
```

Result:

Spring Boot will check `./important.properties`, not find it, log a warning, and **continue startup**.

---

Option 2 — as a system property

```
java -Dspring.config.on-not-found=ignore -jar myapp.jar \
  --spring.config.location=file:./important.properties
```

---

Option 3 — programmatically (in code)

In your `main()` method:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApp.class);
        app.setDefaultProperties(Collections.singletonMap("spring.config.on-not-found", "ignore"));
        app.run(args);
    }
}
```

This sets the default behavior for your app — if any config file is missing, startup continues.

## **Wildcard Locations**

A **wildcard location** is a way to tell Spring Boot to load configuration files from **multiple subdirectories** automatically — without listing them one by one.

You use it by adding a `*` in the **last part** of your config path, like:

```
config/*/
```

The `*` means “all immediate subdirectories here.”

---

### **Why is this useful?**

In real-world environments (like **Kubernetes**, **Docker**, or **microservices** setups), configurations are often split across multiple folders or mounted volumes — one per component or service.

Example:

```
/config/redis/application.properties
/config/mysql/application.properties
```

If you set:

```
--spring.config.location=config/*/
```

Then Spring Boot will:

- Look inside **all subdirectories** under `/config`
- Load **application.properties** (or `.yaml`) files from each of them
- Merge them together into the Environment

This allows you to separate configurations (like Redis, MySQL, etc.) neatly but still have them all loaded automatically.

---

### **Default behavior in Spring Boot**

By default, Spring Boot already includes:

```
config/*/
```

as part of its **default search locations**.

That means even without specifying anything, Spring Boot automatically looks for config files inside all subdirectories of `/config` (outside your JAR).

---

### **You can define your own wildcard locations**

You can customize search paths using either:

- `spring.config.location`
- `spring.config.additional-location`

Example:

```
--spring.config.additional-location=file:/etc/myapp/configs/*/
```

This will include all subdirectories under `/etc/myapp/configs`.

---

### **Important rules about wildcards**

- A wildcard () can appear **only once** in the path.
- It must be in the **last segment** of the path.
    - Example (directory-based): `config/*/`
    - Example (file-based): `config/*/application.properties`
- Wildcard locations are **expanded alphabetically** by path.
- Wildcards **work only with external directories**, not with classpath resources.
    
    That means this is **not allowed**:
    
    ```
    classpath:config/*/
    ```
    
    because classpath resources inside JARs cannot be scanned dynamically at runtime.
    

## **Importing Additional Data**

The property `spring.config.import` lets you **import other configuration files** into your main `application.properties` or `application.yaml`.

Think of it like **“include”** in C or **“import”** in Python — it allows you to pull additional config files into your main config dynamically.

When Spring Boot reads `application.properties`, if it finds a `spring.config.import` key, it will **load** the referenced file(s) immediately and **merge** their properties into the environment.

### **Basic Example**

Suppose you have this file:

`application.properties` (in classpath)

```
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

And in your project directory, you have:

`dev.properties` (in current directory)

```
spring.application.name=myapp-dev
server.port=8082
```

Result:

- Spring Boot first loads `application.properties`
- Sees the `spring.config.import` entry
- Loads `dev.properties`
- Merges both configurations

Because **imported configs override the importing one**, the effective result will be:

```
spring.application.name=myapp-dev
server.port=8082
```

If `dev.properties` does **not** exist, no error occurs (because of `optional:`).

---

### **“Fixed” vs “Import Relative” Locations**

### **Fixed Location**

A **fixed** location is a full, absolute path — it always points to the same place, regardless of where the importing file is.

It starts with:

- `/`
- `file:`
- `classpath:`
- or another URL-like prefix

Example:

```
spring.config.import=file:/etc/myapp/defaults.properties
```

This will always refer to `/etc/myapp/defaults.properties` — no matter where `application.properties` is.

---

### **Import Relative Location**

A **relative** location depends on **where the importing file lives**.

If the path **does not start** with `/` or a URL prefix, it’s relative.

Example setup:

```
/demo/
  ├── application.jar
  ├── application.properties
  └── core/
      └── core.properties
```

`application.properties`

```
spring.config.import=optional:core/core.properties
```

Here, `core/core.properties` is **relative** to `/demo/`

Spring Boot will try to load `/demo/core/core.properties`.

Now in `core/core.properties`

```
spring.config.import=optional:extra/extra.properties
```

This is **relative** to the folder containing `core.properties` (`/demo/core/`),

so it looks for `/demo/core/extra/extra.properties`.

---

### **Import Order**

The **order of imports** inside a single file does not matter — they are all processed, and the imported files **override** the properties of the importer.

Example 1:

```
spring.config.import=my.properties
my.property=value
```

Example 2:

```
my.property=value
spring.config.import=my.properties
```

Both behave the same — values from `my.properties` **override** the value of `my.property` in the importing file.

If you import multiple locations:

```
spring.config.import=file:defaults.properties,file:dev.properties
```

They are processed **in order**, so `dev.properties` can override values from `defaults.properties`.

---

**Profile-Specific Imports**

Spring Boot will automatically check for **profile-specific variants** of imported files.

Example:

If you import `my.properties`, and your active profile is `dev`,

Spring Boot will also look for:

```
my-dev.properties
```

and merge it automatically (if it exists).

---

### **Pluggable Import Sources**

`spring.config.import` is not limited to local files.

By default, Spring Boot supports importing:

- `.properties`
- `.yaml`
- **configuration trees** (like mounted directories with one file per property)

But third-party libraries can also extend this behavior using:

- `ConfigDataLocationResolver`
- `ConfigDataLoader`

This allows imports from **external configuration sources**, for example:

- **Consul**
- **Zookeeper**
- **Netflix Archaius**
- Or even your own custom configuration system.

Example (if supported):

```
spring.config.import=consul:
```

---

### **Real-World Example**

Imagine your folder structure:

```
project/
├── application.properties
├── database/
│   ├── mysql.properties
│   └── postgres.properties
└── environments/
    ├── dev.properties
    └── prod.properties
```

`application.properties`

```
spring.application.name=myapp
spring.config.import=optional:database/mysql.properties,optional:environments/dev.properties
```

`database/mysql.properties`

```
db.url=jdbc:mysql://localhost:3306/mydb
db.username=root
```

`environments/dev.properties`

```
server.port=8081
logging.level.root=DEBUG
```

**Effective configuration:**

- Loaded: `application.properties`
- Imports `database/mysql.properties` and `environments/dev.properties`
- Resulting properties merged as:
    
    ```
    spring.application.name=myapp
    db.url=jdbc:mysql://localhost:3306/mydb
    db.username=root
    server.port=8081
    logging.level.root=DEBUG
    ```
    

## **Importing Extensionless Files**

Some cloud platforms cannot add a file extension to volume mounted files. To import these extensionless files, you need to give Spring Boot a hint so that it knows how to load them. You can do this by putting an extension hint in square brackets.

For example, suppose you have a `/etc/config/myconfig` file that you wish to import as yaml. You can import it from your `application.properties` using the following:

```
spring.config.import=file:/etc/config/myconfig[.yaml]
```

## **Using Environment Variables**

### What are environment variables in this context?

An **environment variable** is a key-value pair that exists in the operating system environment where your application runs. Cloud platforms (like Kubernetes, Docker, or even Linux/Windows servers) can set environment variables for your application so that you don’t need to hardcode configuration values.

Example of environment variables in Linux:

```bash
export MY_CONFIGURATION="my.name=Service1
my.cluster=Cluster1"
```

Here, `MY_CONFIGURATION` is the environment variable containing multiple configuration properties in `.properties` format.

In Kubernetes, you might define it like this in a Pod manifest:

```yaml
env:
  - name: MY_CONFIGURATION
    value: |
      my.name=Service1
      my.cluster=Cluster1
```

### Using the `env:` prefix with `spring.config.import`

Spring Boot 2.4+ introduced **config import**, which allows you to load configuration from multiple sources, including environment variables.

If you have a **single environment variable containing multiple `.properties` entries**, you can tell Spring Boot to import it as a configuration file:

```
spring.config.import=env:MY_CONFIGURATION
```

Here’s what happens:

- `env:MY_CONFIGURATION` tells Spring Boot: "Look for an environment variable called `MY_CONFIGURATION`."
- Spring Boot reads the **contents** of that variable as a configuration file.
- Each line like `my.name=Service1` is treated as a Spring property.

You can also specify the file type if it’s YAML instead of `.properties`:

```
spring.config.import=env:MY_CONFIGURATION.yaml
```

The default extension is `.properties`, so if you don’t specify, Spring Boot assumes `.properties`.

---

### Why is this useful in cloud platforms?

Cloud environments often provide dynamic configuration or secrets through environment variables. Instead of creating separate property files inside your container or image, you can:

- Set configurations dynamically using environment variables.
- Use `spring.config.import=env:VAR_NAME` to load them.
- Avoid hardcoding values in your application.

For example, in Kubernetes:

```yaml
env:
  - name: MY_CONFIGURATION
    value: |
      my.name=Service1
      my.cluster=Cluster1
```

Then in your Spring Boot application:

```
spring.config.import=env:MY_CONFIGURATION
```

Your application will automatically pick up `my.name` and `my.cluster` as Spring properties.

## **Using Configuration Trees**

**Configuration trees in Spring Boot as an alternative to environment variables**, which is very relevant for cloud-native applications like those running on Kubernetes or Docker Swarm.

### The problem with environment variables

While environment variables are convenient, they have some **drawbacks**:

- They are visible in process listings or container metadata, which can be a security concern.
- For complex configurations (like multi-line YAML), storing everything as an environment variable can be messy.
- Secrets like passwords, API keys, or certificates should be stored securely rather than in environment variables.

Because of this, cloud platforms often provide **mounted volumes** as a more secure and flexible alternative.

---

### Volume mount patterns

When using mounted volumes for configuration, there are two main patterns:

**Pattern 1: Single file with all properties**

- One file contains the complete configuration.
- The file can be in `.properties` or `.yaml` format.
- You can import it directly in Spring Boot using:

```
spring.config.import=classpath:/myconfig.yaml
# or
spring.config.import=file:/etc/config/myapp.yaml
```

This works just like importing a normal property or YAML file.

**Pattern 2: Directory tree of files (config tree)**

- Each **file** represents one configuration property.
- The **folder structure** determines the property namespace.
- The **filename** becomes the property name.
- The **file content** becomes the property value.

Example Kubernetes volume:

```
/etc/config/myapp/
    username
    password
```

- `username` file contents → property `myapp.username`
- `password` file contents → property `myapp.password`

To import this directory as a **configuration tree**, you use the `configtree:` prefix:

```
spring.config.import=optional:configtree:/etc/config/myapp
```

Here:

- `configtree:` tells Spring Boot “load all files in this directory as properties”.
- `optional:` means that Spring Boot won’t fail if the directory doesn’t exist.

---

### Handling nested properties

- Folder names are part of the property key.
- Filenames with dots are mapped correctly.

Example:

```
/etc/config/
    myapp.username
```

- Becomes Spring property: `myapp.username`

This is helpful if you have more hierarchical configurations.

---

### Multiple configuration trees

If your volume has **multiple config trees**, you can use a wildcard `*`:

Example directory structure:

```
/etc/config/
    dbconfig/db/username
    dbconfig/db/password
    mqconfig/mq/username
    mqconfig/mq/password
```

You can import all of them with:

```
spring.config.import=optional:configtree:/etc/config/*/
```

Resulting Spring properties:

```
db.username
db.password
mq.username
mq.password
```

Notes:

- The directories are sorted alphabetically if you use a wildcard.
- If order matters, import each directory separately:

```
spring.config.import=optional:configtree:/etc/config/dbconfig/db,optional:configtree:/etc/config/mqconfig/mq
```

---

### Using configuration trees for secrets

- Docker Swarm and Kubernetes can **mount secrets** as files inside a container.
- For example, if a Docker secret `db.password` is mounted at `/run/secrets/db.password`, you can expose it to Spring Boot with:

```
spring.config.import=optional:configtree:/run/secrets/
```

- This makes the secret available as `db.password` in Spring’s `Environment`.

## **Property Placeholders**

### Property placeholders: the `${...}` syntax

In Spring Boot, you can **refer to other properties** using the `${property-name}` syntax inside `application.properties` or `application.yaml`.

Example:

```
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

Here:

- `${app.name}` is a **placeholder**.
- Spring Boot will replace `${app.name}` with the value of the `app.name` property, which is `MyApp`.
- So `app.description` becomes:

```
MyApp is a Spring Boot application
```

---

### Using default values

You can provide a **default value** for a placeholder using the `:` syntax:

```
app.description=${app.name} is written by ${username:Unknown}
```

Explanation:

- `${username:Unknown}` → Spring Boot will look for a property named `username`.
- If `username` exists (for example, set as an environment variable or system property), it will be used.
- If `username` does not exist, Spring Boot uses the default value `Unknown`.

So in this example, if `username` is not set, `app.description` becomes:

```
MyApp is written by Unknown
```

---

### Filtering through the Environment

Spring Boot doesn’t only look at `application.properties` or `application.yaml`. When it resolves `${...}` placeholders, it also searches **all property sources in the Environment**, including:

- System properties (`D` arguments)
- Environment variables (from OS or container)
- Other config files imported via `spring.config.import`
- Command-line arguments

This allows placeholders to **dynamically pick up values** from multiple sources.

---

### Canonical property names and relaxed binding

Spring Boot has a **relaxed binding mechanism**, which allows you to use different naming conventions for the same property:

- Kebab-case (`demo.item-price`)
- CamelCase (`demo.itemPrice`)
- Environment variable style (`DEMO_ITEMPRICE`)

When you use placeholders:

```
${demo.item-price}
```

Spring Boot will consider:

- `demo.item-price` (kebab-case in `application.properties`)
- `demo.itemPrice` (camelCase variant)
- `DEMO_ITEMPRICE` (environment variable or system property)

**Important:** If you used `${demo.itemPrice}` (camelCase in the placeholder), Spring Boot **won’t pick up kebab-case or environment variables**, so it’s recommended to always use **canonical kebab-case** in placeholders.

---

### Creating “short” variants of properties

You can use placeholders to create **shorter aliases** or **derived values**:

Example:

```
server.port=${custom.port:8080}
```

- Here, `server.port` will use the value of `custom.port` if set.
- If `custom.port` is not set, it will default to `8080`.
- This can be handy for **command-line arguments** or optional environment variables.

## **Working With Multi-Document Files**

### Concept of multi-document files

- In Spring Boot, a single `application.yaml` or `application.properties` file can contain **multiple logical documents**.
- Documents are **processed in order**, from top to bottom.
- Later documents can **override** properties defined in earlier documents.
- This is often used to **activate specific configurations conditionally**, e.g., based on profiles or cloud platforms.

---

### Multi-document in YAML

YAML has a built-in multi-document syntax:

- `---` marks the **end of one document** and the **start of the next**.

Example: `application.yaml`

```yaml
# Document 1
spring:
  application:
    name: "MyApp"
---
# Document 2
spring:
  application:
    name: "MyCloudApp"
  config:
    activate:
      on-cloud-platform: "kubernetes"
```

Explanation:

- **Document 1** sets `spring.application.name` to `"MyApp"`.
- **Document 2** overrides `spring.application.name` with `"MyCloudApp"` and adds a new property `spring.config.activate.on-cloud-platform=kubernetes`.
- When Spring Boot reads this file, the **second document takes precedence** for any overlapping properties.

---

### Multi-document in properties files

Properties files don’t natively support multiple documents, so Spring Boot uses a **special comment separator**:

- `#---` or `!---` marks the **split between logical documents**.
- Rules:
    - Must have **exactly three hyphens**.
    - No leading whitespace.
    - Lines before and after separator cannot use the same comment style.

Example: `application.properties`

```
# Document 1
spring.application.name=MyApp
#---
# Document 2
spring.application.name=MyCloudApp
spring.config.activate.on-cloud-platform=kubernetes
```

Explanation:

- **Document 1**: `spring.application.name=MyApp`.
- **Document 2**: overrides `spring.application.name` and adds `spring.config.activate.on-cloud-platform=kubernetes`.
- Spring Boot merges them in order, giving **later document precedence**.

---

### Practical use cases

1. **Profile-specific configuration**: You can have different documents activated for different profiles.

```yaml
# Document 1 (default)
spring:
  datasource:
    url: jdbc:h2:mem:testdb
---
# Document 2 (for prod profile)
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod-db:3306/mydb
```

- By default, the first document is used.
- When the `prod` profile is active, the second document is applied, overriding the datasource URL.
1. **Cloud platform-specific configuration**:

```yaml
# Document 1 (default)
spring:
  application:
    name: MyApp
---
# Document 2 (for Kubernetes)
spring:
  config:
    activate:
      on-cloud-platform: kubernetes
  logging:
    level: DEBUG
```

- When running on Kubernetes, Spring Boot will activate the second document.

---

### Important notes

- Multi-document files **cannot** be loaded via `@PropertySource` or `@TestPropertySource`.
- They are usually processed by Spring Boot automatically from `application.yaml` or `application.properties`.
- Activation properties (`spring.config.activate.on-profile`, `spring.config.activate.on-cloud-platform`) allow you to **conditionally apply a document**.

## **Activation Properties**

It is sometimes useful to only activate a given set of properties when certain conditions are met. For example, you might have properties that are only relevant when a specific profile is active.

You can conditionally activate a properties document using `spring.config.activate.*`.

The following activation properties are available:

| **Property** | **Note** |
| --- | --- |
| `on-profile` | A profile expression that must match for the document to be active, or a list of profile expressions of which at least one must match for the document to be active. |
| `on-cloud-platform` | The [`CloudPlatform`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/cloud/CloudPlatform.html) that must be detected for the document to be active. |

For example, the following specifies that the second document is only active when running on Kubernetes, and only when either the “prod” or “staging” profiles are active:

```
myprop=always-set
#---
spring.config.activate.on-cloud-platform=kubernetes
spring.config.activate.on-profile=prod | staging
myotherprop=sometimes-set
```

# **Encrypting Properties**

Spring Boot does not provide any built-in support for encrypting property values, however, it does provide the hook points necessary to modify values contained in the Spring [`Environment`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/Environment.html). The [`EnvironmentPostProcessor`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/env/EnvironmentPostProcessor.html) interface allows you to manipulate the [`Environment`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/Environment.html) before the application starts. See [Customize the Environment or ApplicationContext Before It Starts](https://docs.spring.io/spring-boot/how-to/application.html#howto.application.customize-the-environment-or-application-context) for details.

If you need a secure way to store credentials and passwords, the [Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/) project provides support for storing externalized configuration in [HashiCorp Vault](https://www.vaultproject.io/).

# **Working With YAML**

[YAML](https://yaml.org/) is a superset of JSON and, as such, is a convenient format for specifying hierarchical configuration data. The [`SpringApplication`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/SpringApplication.html) class automatically supports YAML as an alternative to properties whenever you have the [SnakeYAML](https://github.com/snakeyaml/snakeyaml) library on your classpath. **SnakeYAML library** to parse YAML files.

## **Mapping YAML to Properties**

YAML documents need to be converted from their hierarchical format to a flat structure that can be used with the Spring [`Environment`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/Environment.html). For example, consider the following YAML document:

```yaml
environments:
  dev:
    url: "https://dev.example.com"
    name: "Developer Setup"
  prod:
    url: "https://another.example.com"
    name: "My Cool App"
```

In order to access these properties from the [`Environment`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/Environment.html), they would be flattened as follows:

```
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

Likewise, YAML lists also need to be flattened. They are represented as property keys with `[index]` dereferencers. For example, consider the following YAML:

```yaml
 my:
  servers:
  - "dev.example.com"
  - "another.example.com"
```

The preceding example would be transformed into these properties:

```
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

> Properties that use the `[index]` notation can be bound to Java [`List`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html) or [`Set`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Set.html) objects using Spring Boot’s [`Binder`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/context/properties/bind/Binder.html) class. For more details see the Type-safe Configuration Properties section below.
> 

> Note:
> 
> 
> YAML files cannot be loaded by using the `@PropertySource` or `@TestPropertySource` annotations. So, in the case that you need to load values that way, you need to use a properties file.
> 

## **Directly Loading YAML**

Spring Framework provides two convenient classes that can be used to load YAML documents. The [`YamlPropertiesFactoryBean`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/beans/factory/config/YamlPropertiesFactoryBean.html) loads YAML as [`Properties`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Properties.html) and the [`YamlMapFactoryBean`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/beans/factory/config/YamlMapFactoryBean.html) loads YAML as a [`Map`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.html).

You can also use the [`YamlPropertySourceLoader`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/env/YamlPropertySourceLoader.html) class if you want to load YAML as a Spring [`PropertySource`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/core/env/PropertySource.html).

### **`YamlPropertiesFactoryBean`**

- Purpose: Converts a YAML file into a **`Properties` object**.
- Useful if you want to treat the YAML like a `.properties` file.

**Example:**

Suppose we have a `config.yml`:

```yaml
app:
  name: MyApp
  version: 1.0
database:
  url: jdbc:mysql://localhost:3306/mydb
  username: root
  password: pass

```

We can load it as `Properties`:

```java
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.core.io.ClassPathResource;

import java.util.Properties;

public class YamlAsPropertiesExample {
    public static void main(String[] args) {
        YamlPropertiesFactoryBean yamlFactory = new YamlPropertiesFactoryBean();
        yamlFactory.setResources(new ClassPathResource("config.yml"));
        Properties properties = yamlFactory.getObject();

        System.out.println(properties.getProperty("app.name"));       // Output: MyApp
        System.out.println(properties.getProperty("database.url"));   // Output: jdbc:mysql://localhost:3306/mydb
    }
}
```

- The YAML structure is flattened into keys like `app.name` and `database.url`.

---

### **`YamlMapFactoryBean`**

- Purpose: Converts a YAML file into a **`Map`**.
- Preserves the hierarchical structure more naturally.

**Example:**

```java
import org.springframework.beans.factory.config.YamlMapFactoryBean;
import org.springframework.core.io.ClassPathResource;

import java.util.Map;

public class YamlAsMapExample {
    public static void main(String[] args) {
        YamlMapFactoryBean yamlMapFactory = new YamlMapFactoryBean();
        yamlMapFactory.setResources(new ClassPathResource("config.yml"));
        Map<String, Object> map = yamlMapFactory.getObject();

        Map<String, Object> databaseMap = (Map<String, Object>) map.get("database");
        System.out.println(databaseMap.get("url")); // Output: jdbc:mysql://localhost:3306/mydb
        System.out.println(databaseMap.get("username")); // Output: root
    }
}
```

- The YAML hierarchy is preserved: the `database` key contains a nested map.

---

### **`YamlPropertySourceLoader`**

- Purpose: Load YAML as a **Spring `PropertySource`**, which integrates with Spring’s `Environment`.
- Useful if you want Spring to treat your YAML like a configuration source.

**Example:**

```java
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.boot.env.YamlPropertySourceLoader;

import java.io.IOException;
import java.util.List;

public class YamlPropertySourceExample {
    public static void main(String[] args) throws IOException {
        YamlPropertySourceLoader loader = new YamlPropertySourceLoader();
        List<PropertySource<?>> propertySources =
                loader.load("customYaml", new ClassPathResource("config.yml"));

        PropertySource<?> propertySource = propertySources.get(0);

        System.out.println(propertySource.getProperty("app.name"));       // Output: MyApp
        System.out.println(propertySource.getProperty("database.username")); // Output: root
    }
}
```

- The `PropertySource` can now be added to Spring’s `Environment` to use with `@Value` or `Environment.getProperty()`.

# **Configuring Random Values**

### **What is `RandomValuePropertySource`?**

- Spring Boot provides a special property source called `RandomValuePropertySource`.
- It allows you to **inject random values** into your configuration at runtime.
- This is useful for **secrets, test data, or temporary IDs**.

---

### **Supported Random Values**

You can inject random values using placeholders in your properties or YAML files:

| Placeholder | Output |
| --- | --- |
| `${random.value}` | Random alphanumeric string |
| `${random.int}` | Random integer |
| `${random.long}` | Random long number |
| `${random.uuid}` | Random UUID |
| `${random.int(n)}` | Random integer between 0 (inclusive) and n (exclusive) |
| `${random.int[a,b]}` | Random integer between a (inclusive) and b (exclusive) |
- **Syntax:** `random.int*` uses **OPEN(value,max)CLOSE** or **OPEN[value,max]CLOSE**.
- For example, `random.int(10)` → integer 0–9, `random.int[1024,65536]` → integer 1024–65535.

---

### **Example in YAML**

`application.yml`:

```yaml
my:
  secret: ${random.value}
  number: ${random.int}
  bignumber: ${random.long}
  uuid: ${random.uuid}
  number-less-than-ten: ${random.int(10)}
  number-in-range: ${random.int[1024,65536]}
```

# **Configuring System Environment Properties**

### **What is environment property prefix in Spring Boot?**

- Spring Boot can read configuration from **system environment variables**.
- Sometimes multiple applications share the same system environment, and you want to **avoid conflicts**.
- Spring Boot allows you to **set a prefix** so that it only reads environment variables with that prefix.
- This is done via:

```java
springApplication.setEnvironmentPrefix("PREFIX");
```

- **Effect:** The property `some.property` in Spring Boot will look for `PREFIX_SOME_PROPERTY` in the system environment.

---

### **How it works**

- **Without prefix:**
    - Spring property: `remote.timeout`
    - Environment variable: `REMOTE_TIMEOUT`
- **With prefix:**
    - If prefix is `INPUT`
    - Spring property: `remote.timeout`
    - Environment variable: `INPUT_REMOTE_TIMEOUT`
- Only **system environment variables** are affected. Other property sources like `application.properties` or `application.yml` remain the same.

---

### **Example**

Suppose you have an environment variable in your OS:

```bash
export INPUT_REMOTE_TIMEOUT=5000
```

**Spring Boot Application**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ConfigurableBootstrapContext;

@SpringBootApplication
public class EnvPrefixExampleApplication {

    @Value("${remote.timeout}")
    private int remoteTimeout;

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(EnvPrefixExampleApplication.class);
        // Set environment prefix for system properties
        app.setEnvironmentPrefix("INPUT");
        app.run(args);
    }

    @PostConstruct
    public void printValue() {
        System.out.println("Remote timeout: " + remoteTimeout);
    }
}
```

**What happens**

- Spring Boot looks for `INPUT_REMOTE_TIMEOUT` in the system environment.
- Reads the value `5000` and injects it into `remoteTimeout`.
- If there were no prefix, it would look for `REMOTE_TIMEOUT` instead.

**Notes:**

- Prefix **only affects environment variables**, not properties in `application.yml` or `.properties`.
- Helps prevent **naming conflicts** when multiple apps use the same system environment.
- The mapping automatically converts Spring property names to **uppercase with underscores**.

# **Type-safe Configuration Properties**

Using the `@Value("${property}")` annotation to inject configuration properties can sometimes be cumbersome, especially if you are working with multiple properties or your data is hierarchical in nature. Spring Boot provides an alternative method of working with properties that lets strongly typed beans govern and validate the configuration of your application.

### What happens with `@Value("${property}")`

When you use the `@Value` annotation, you inject individual configuration values like this:

```java
@Value("${server.port}")
private int port;

@Value("${app.datasource.url}")
private String datasourceUrl;

```

This approach works fine for **simple or few properties**.

However, when you have **many related properties** or **nested configurations**, this becomes **cumbersome** and **error-prone**.

---

### The drawbacks being pointed out

a) **Scattered configuration**

- Each property must be injected separately across different classes or fields.
- This leads to scattered property management — no single place represents the entire configuration.

Example:

```java
@Value("${app.datasource.url}")
private String url;

@Value("${app.datasource.username}")
private String username;

@Value("${app.datasource.password}")
private String password;
```

All these belong to the same logical group, but there’s no single object holding them together.

b) **No type safety**

- Properties injected with `@Value` are plain strings at first, and type conversion is done implicitly.
- If there’s a mismatch (e.g., you expect an integer but the property is not numeric), you won’t know until runtime.

Example:

```java
@Value("${app.max-users}")
private int maxUsers; // runtime error if "max-users" is not a number
```

So, compile-time validation is **not possible**.

**c) Harder to maintain hierarchical data**

- If your properties are structured hierarchically (e.g., `app.datasource.url`, `app.datasource.username`, etc.), it becomes repetitive and verbose to inject each key manually.

**d) No built-in validation**

- With `@Value`, you cannot easily validate the properties (for example, to check if a port number is in a valid range or a required field is non-null).

You would have to write manual validation code.

---

### The alternative: `@ConfigurationProperties`

To solve all the above, Spring Boot provides **type-safe configuration properties** using the `@ConfigurationProperties` annotation.

Example:

```java
@ConfigurationProperties(prefix = "app.datasource")
public class DataSourceProperties {

    private String url;
    private String username;
    private String password;

    // getters and setters
}
```

Then register it:

```java
@Component
@EnableConfigurationProperties(DataSourceProperties.class)
public class AppConfig {
    private final DataSourceProperties dataSourceProperties;

    public AppConfig(DataSourceProperties dataSourceProperties) {
        this.dataSourceProperties = dataSourceProperties;
    }
}
```

Now:

- All properties under `app.datasource.*` are **mapped automatically**.
- Type safety is ensured.
- You can use **JSR-303 validation annotations** like `@NotNull`, `@Min`, `@Max`.

## **JavaBean Properties Binding**

```java
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my.service")
public class MyProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	// getters and setters

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		// getters and setters

	}

}
```

The idea is that properties defined in external configuration files (like `application.properties` or `application.yml`) with the prefix `my.service` will be **automatically bound** to the corresponding fields in this class.

### How Property Binding Works

Spring Boot’s **Binder** mechanism maps configuration properties to Java objects.

For example, suppose your `application.yml` has:

```yaml
my:
  service:
    enabled: true
    remote-address: 192.168.1.10
    security:
      username: admin
      password: secret
      roles:
        - ADMIN
        - USER
```

Spring Boot automatically fills the `MyProperties` bean with these values.

### Key Points Explained

a) **The prefix `"my.service"`**

- All configuration keys beginning with `my.service` map to fields in this class.
- For example, `my.service.enabled` → `enabled`.

b) **Nested classes = Nested properties**

- The `Security` inner class creates a nested structure (`my.service.security.*`).
- Spring Boot recognizes nested POJOs and binds their properties accordingly.

c) **Type conversion**

- Spring Boot automatically converts string property values to appropriate Java types (e.g., `InetAddress`, `int`, `boolean`, `Duration`, etc.).
- So, `"192.168.1.10"` becomes an `InetAddress` object.

d) **Default values**

- If a field is initialized in the class (like `roles = new ArrayList<>(Collections.singleton("USER"))`), that becomes the default value if no configuration is provided.

### Handling Reserved Keywords

If a property name uses a **reserved keyword** (like `import`), you can use the `@Name` annotation to override how it maps.

Example:

```java
@Name("import")
private String importFile;
```

Then, this field maps to the property `my.service.import`.

## **Constructor Binding**

By default, Spring Boot binds configuration properties using **setters** (standard JavaBeans style).

However, in **immutable configuration classes** (where fields are `final` and no setters exist), Spring Boot can bind through a **constructor**. This is called **constructor binding**.

Example:

```java
@ConfigurationProperties("my.service")
public class MyProperties {

    private final boolean enabled;
    private final InetAddress remoteAddress;
    private final Security security;

    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    // getters...
}
```

Since there is **only one constructor**, Spring Boot automatically assumes **constructor binding**.

---

### What Happens When You Have Multiple Constructors

If your class has **multiple constructors**, Spring Boot **cannot automatically guess** which one should be used for binding.

That’s when you need to explicitly specify it using **`@ConstructorBinding`**.

Example:

```java
@ConfigurationProperties("my.service")
public class MyProperties {

    private final boolean enabled;
    private final InetAddress remoteAddress;
    private final Security security;

    // 1st constructor
    public MyProperties() {
        this.enabled = false;
        this.remoteAddress = null;
        this.security = new Security("default", "default", List.of("USER"));
    }

    // 2nd constructor
    @ConstructorBinding
    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    // getters...
}
```

Here:

- The presence of **`@ConstructorBinding`** tells Spring Boot **which constructor** to use for configuration property binding.
- Other constructors are simply ignored by the binder.

---

### Should You Put `@ConstructorBinding` on All Constructors?

**No.**

You should put `@ConstructorBinding` **only on the constructor that should be used for binding**.

Reason:

- Spring Boot allows **only one constructor** to be used for property binding.
- Annotating multiple constructors with `@ConstructorBinding` would cause ambiguity and will **fail at startup** with an error like:
    
    ```
    @ConstructorBinding annotation may only be used on a single constructor
    ```
    

---

### How Spring Boot Decides What to Use

Spring Boot follows this logic internally:

| Case | Behavior |
| --- | --- |
| Class has **one constructor** | That constructor is automatically used for constructor binding (no `@ConstructorBinding` needed). |
| Class has **multiple constructors** | You must annotate one of them with `@ConstructorBinding` to tell Spring Boot which one to use. |
| You want to **opt out** of constructor binding | Annotate the constructor with `@Autowired` or make it `private`. Then Spring will use setter binding instead. |

So Spring Boot will:

1. Look for a single public constructor.
2. If multiple exist, it searches for one annotated with `@ConstructorBinding`.
3. If none are annotated, and there are multiple public constructors, an **error** occurs because it’s ambiguous.

---

### Nested Classes and Constructor Binding

Constructor binding can also apply to **nested classes** (like your `Security` inner class).

You can also annotate its constructor with `@ConstructorBinding` if it has more than one.

However, if the outer class uses constructor binding, the binder will automatically propagate it to nested classes — so you usually **don’t need to annotate nested constructors** unless ambiguity exists there too.

Example:

```java
public static class Security {

    @ConstructorBinding
    public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
        this.username = username;
        this.password = password;
        this.roles = roles;
    }
}
```

---

### Opting Out of Constructor Binding

If you want to tell Spring **not** to use constructor binding for a specific constructor:

- Annotate it with `@Autowired`, or
- Make it `private`.

This signals Spring that this constructor is for dependency injection (or internal use), not for property binding.

Example:

```java
@ConfigurationProperties("my.service")
public class MyProperties {

    private final boolean enabled;
    private final InetAddress remoteAddress;
    private final Security security;

    @Autowired
    public MyProperties(SomeDependency dependency) {
        this.enabled = false;
        this.remoteAddress = null;
        this.security = new Security("admin", "pass", List.of("USER"));
    }

    @ConstructorBinding
    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }
}
```

Spring will use the **`@ConstructorBinding` constructor** for configuration binding and the `@Autowired` one for dependency injection.

### “Constructor binding can be used with records. Unless your record has multiple constructors, there is no need to use @ConstructorBinding.”

Explanation

- In Java, a **record** is a special type of class that is **immutable** and **automatically provides** a constructor, getters, and `equals()/hashCode()/toString()`.

Example:

```java
@ConfigurationProperties("my.service")
public record MyProperties(boolean enabled, InetAddress remoteAddress, Security security) { }
```

- Records **always have one canonical constructor** (one that takes all fields as parameters).
- Therefore, Spring Boot can **automatically bind properties** to a record’s constructor **without needing `@ConstructorBinding`**.

So, this:

```java
@ConfigurationProperties("my.service")
public record MyProperties(boolean enabled, InetAddress remoteAddress, Security security) { }
```

is equivalent to:

```java
@ConfigurationProperties("my.service")
@ConstructorBinding
public record MyProperties(boolean enabled, InetAddress remoteAddress, Security security) { }
```

If the record **defines additional constructors**, then you’ll need to explicitly specify **which constructor** Spring should use for binding by annotating one of them with `@ConstructorBinding`.

---

### “Nested members of a constructor bound class (such as Security in the example above) will also be bound through their constructor.”

Explanation

Let’s take your previous example again:

```java
@ConfigurationProperties("my.service")
public class MyProperties {
    private final boolean enabled;
    private final InetAddress remoteAddress;
    private final Security security;

    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public static class Security {
        private final String username;
        private final String password;
        private final List<String> roles;

        public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }
    }
}
```

Here, since `MyProperties` is **constructor-bound**, Spring Boot automatically assumes that its **nested class** (`Security`) should also be **constructor-bound**.

That means:

- Spring will call the `Security` constructor and pass the properties (like `username`, `password`, `roles`) directly to it.
- You don’t have to annotate `Security` with `@ConstructorBinding` explicitly.
- You don’t have to add setters — constructor binding will handle it automatically.

So, **constructor binding propagates to nested objects**.

---

### “Default values can be specified using @DefaultValue on constructor parameters and record components. The conversion service will be applied to coerce the annotation’s String value to the target type of a missing property.”

Explanation

When you use **constructor binding**, there are no setters, so you cannot assign default values after construction.

Instead, you can specify **default values** directly on the constructor parameters using the `@DefaultValue` annotation.

Example:

```java
public Security(
    String username,
    String password,
    @DefaultValue("USER") List<String> roles
) {
    this.username = username;
    this.password = password;
    this.roles = roles;
}
```

If no `my.service.security.roles` property is defined in the configuration file, the binder will use `"USER"` as the default.

**About Conversion**

The default value provided in `@DefaultValue` is a **String**, but Spring’s **ConversionService** automatically converts it to the target type.

For example:

```java
@DefaultValue("192.168.1.1") InetAddress remoteAddress
```

The string `"192.168.1.1"` will be converted to an `InetAddress` object.

---

### “Referring to the previous example, if no properties are bound to Security, the MyProperties instance will contain a null value for security.”

Explanation

Let’s suppose your configuration file has:

```
my.service.enabled=true
my.service.remote-address=192.168.1.10
```

But **no `my.service.security.*` properties** are provided.

Then:

- Spring Boot will bind the `enabled` and `remoteAddress` values.
- Since there’s **no configuration** for the `security` section, it will not create a `Security` object.
- So, the `security` field in `MyProperties` will be **null**.

This is because Spring Boot does not call constructors for missing nested objects — it only constructs them when corresponding properties exist.

---

### “To make it contain a non-null instance of Security even when no properties are bound to it ... use an empty @DefaultValue annotation.”

Explanation

If you want Spring Boot to always create a `Security` object, **even when the user provides no configuration for it**, you can use `@DefaultValue` **without specifying a value** — that is, an *empty annotation*.

Example:

```java
public MyProperties(boolean enabled, InetAddress remoteAddress, @DefaultValue Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
}
```

In this case:

- If no `my.service.security.*` properties are provided,
- Spring Boot will still **instantiate `Security`** using its constructor,
- Filling its parameters with `null` or their own defaults (if `@DefaultValue` is used inside it).

This ensures that `MyProperties.getSecurity()` never returns `null`.

## **Enabling @ConfigurationProperties-annotated Types**

### Purpose of `@EnableConfigurationProperties`

If your `@ConfigurationProperties` classes **cannot be discovered through scanning**, you can register them manually using `@EnableConfigurationProperties`.

**Example:**

```java
@Configuration
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {
}
```

- `@EnableConfigurationProperties(SomeProperties.class)` tells Spring Boot to create a bean of `SomeProperties`.
- Useful when you’re writing **auto-configuration** or **conditional configurations**, where component scanning isn’t ideal.

---

### Defining a `@ConfigurationProperties` class

```java
@ConfigurationProperties("some.properties")
public class SomeProperties {
    private String host;
    private int port;

    // getters and setters
}
```

**Explanation:**

- The prefix `some.properties` means properties in `application.properties` like:
    
    ```
    some.properties.host=localhost
    some.properties.port=8080
    ```
    
    will bind to `host` and `port`.
    

---

### Using `@ConfigurationPropertiesScan`

Instead of manually listing every configuration class, you can **enable package scanning**.

**Example:**

```java
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {
}
```

**Explanation:**

- Spring Boot will automatically find all classes annotated with `@ConfigurationProperties` inside those packages and register them as beans.
- No need to use `@EnableConfigurationProperties` manually for each one.

---

### Bean Naming Convention

When Spring Boot registers a `@ConfigurationProperties` bean, it names it using this pattern:

```
<prefix>-<fully-qualified-class-name>
```

**Example:**

For:

```java
@ConfigurationProperties("some.properties")
public class SomeProperties { }
```

If it’s in package `com.example.app`, the bean name will be:

```
some.properties-com.example.app.SomeProperties
```

If **no prefix** is provided, the name will be:

```
com.example.app.SomeProperties
```

### Recommended Practice

It’s recommended that `@ConfigurationProperties` classes **only bind environment values** (like from `.properties` or `.yaml` files).

They **should not inject other beans** directly.

**Example (Avoid this):**

```java
@ConfigurationProperties("my")
public class MyProperties {
    private final SomeService service; // Not recommended
}
```

**Why?**

- It mixes configuration data with application logic, breaking separation of concerns.

**If you must inject beans**, use:

- **Setter injection**, or
- **`Aware` interfaces**, such as:
    
    ```java
    public class MyProperties implements EnvironmentAware {
        private Environment environment;
    
        @Override
        public void setEnvironment(Environment environment) {
            this.environment = environment;
        }
    }
    ```
    

**If still needed via constructor:**

You can make it a normal Spring bean by adding `@Component` and using standard binding:

```java
@Component
@ConfigurationProperties("my")
public class MyProperties {
    private String name;
    // normal JavaBean binding applies here
}
```

## **Using @ConfigurationProperties-annotated Types**

### External YAML Configuration

The YAML snippet:

```yaml
my:
  service:
    remote-address: 192.168.1.1
    security:
      username: "admin"
      roles:
      - "USER"
      - "ADMIN"
```

**Explanation:**

- `my.service` corresponds to a `@ConfigurationProperties("my.service")` class.
- `remote-address` → binds to a field like `InetAddress remoteAddress`.
- `security.username` → binds to a nested `Security` object.
- `security.roles` → binds to a `List<String>` in the nested object.

Spring Boot automatically reads this YAML file at startup and **binds the values** to the corresponding fields of your configuration class.

---

### Using `@ConfigurationProperties` Bean in Another Class

Your service class:

```java
@Service
public class MyService {

    private final MyProperties properties;

    public MyService(MyProperties properties) {
        this.properties = properties;
    }

    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        server.start();
        // ...
    }
}
```

**Explanation:**

1. `MyProperties` is a **Spring-managed bean** because of either:
    - `@EnableConfigurationProperties(MyProperties.class)`
        
        **or**
        
    - `@ConfigurationPropertiesScan` (if it’s in a scanned package).
2. You can inject it **like any other Spring bean** via constructor injection.
3. `properties.getRemoteAddress()` gives the value from the YAML (`192.168.1.1`) as an `InetAddress`.
4. This allows your service (`MyService`) to **use configuration values directly** without reading `.yaml` or `.properties` manually.

---

### Benefits of This Approach

- **Type-safe configuration**: The fields in `MyProperties` have proper Java types (`InetAddress`, `List<String>`, etc.), so no manual conversion is needed.
- **Centralized configuration**: All `my.service` related properties are grouped in `MyProperties`.
- **Automatic wiring**: Spring Boot injects the configuration bean wherever needed.

## **Third-party Configuration**

### `@ConfigurationProperties` on a **@Bean method**

Normally, `@ConfigurationProperties` is used on a class:

```java
@ConfigurationProperties("my.service")
public class MyProperties { ... }

```

But you can also use it **directly on a `@Bean` method** inside a `@Configuration` class:

```java
@Configuration
public class ThirdPartyConfiguration {

    @Bean
    @ConfigurationProperties("another")
    public AnotherComponent anotherComponent() {
        return new AnotherComponent();
    }
}
```

**Explanation:**

- Spring Boot will create the bean by calling `anotherComponent()`.
- It will then **bind all properties with prefix `another`** from the environment (YAML, `.properties`, etc.) directly onto the `AnotherComponent` bean.
- This is useful for **third-party classes** you cannot annotate with `@ConfigurationProperties` directly.

## **Relaxed Binding**

### **@ConfigurationProperties and Prefix**

The `@ConfigurationProperties` annotation allows you to bind external configuration to a Java/Kotlin class. The prefix you specify determines which properties are mapped.

Example:

```java
@ConfigurationProperties("my.main-project.person")
public class MyPersonProperties {
    private String firstName;
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
}
```

- Prefix: `"my.main-project.person"`
- Bean property: `firstName`

---

### **Relaxed Binding Concept**

Normally, property names in Java follow **camelCase**, but configuration files or environment variables might use different naming conventions. **Relaxed binding** allows Spring Boot to match these different formats automatically.

This means you don’t have to write the property names exactly as the Java field. Spring Boot will map them correctly if the name is *similar enough*.

---

### **Supported Formats**

For the property `firstName` in `MyPersonProperties`, Spring Boot allows multiple formats:

| Format | Example | Notes |
| --- | --- | --- |
| Kebab-case | `my.main-project.person.first-name` | Recommended for `.properties` and YAML files |
| Camel-case | `my.main-project.person.firstName` | Matches standard Java naming |
| Snake-case | `my.main-project.person.first_name` | Alternative format in `.properties` and YAML |
| Upper-case | `MY_MAINPROJECT_PERSON_FIRSTNAME` | Used for environment variables, system-level settings |

**Key point:** The prefix in `@ConfigurationProperties` must be in kebab-case (`my.main-project.person`). The property names themselves can use any of the above formats depending on the source.

---

### **Binding Rules per Source**

Different sources have slightly different rules:

| Source | Property Naming | List Support |
| --- | --- | --- |
| `.properties` | camelCase, kebab-case, underscore | `[ ]` or comma-separated |
| YAML | camelCase, kebab-case, underscore | YAML lists or comma-separated |
| Environment Variables | Upper-case with underscores (`MY_MAINPROJECT_PERSON_FIRSTNAME`) | Numeric values can use underscores (e.g., `MY_NUMBER=_123_`) |
| System Properties | camelCase, kebab-case, underscore | `[ ]` or comma-separated |

---

Note:

- Use **lower-case kebab-case** (`my.person.first-name`) whenever possible for consistency across `.properties` and YAML.
- Environment variables should use **upper-case with underscores** (`MY_PERSON_FIRSTNAME`) to match OS conventions.

### **Binding Maps**

**The Problem**

When you bind properties to a `Map<String, ...>` in Spring Boot, keys in the configuration can have special characters like `/` or `.`. By default:

- Characters that are not **alphanumeric**, , or `.` are **removed** unless you use square brackets `[]`.
- This can cause your key names to be modified unintentionally.

Example:

```
my.map[/key1]=value1
my.map[/key2]=value2
my.map./key3=value3
```

- `/key1` → preserved because of `[ ]`
- `/key2` → preserved because of `[ ]`
- `/key3` → **slash removed** because no brackets are used → becomes `key3`

So if you don’t use brackets, Spring Boot may alter your key names.

---

**YAML Notes**

For YAML files:

```yaml
my:
  map:
    "/key1": value1
    "/key2": value2
    "/key3": value3
```

- The square brackets must be **surrounded by quotes** to parse correctly.
- Without quotes or brackets, special characters may be stripped or misinterpreted.

---

**Scalar Values vs Non-Scalar Values**

- **Scalar values**: simple values like `String`, `Integer`, `Boolean`, enums, etc.
- **Non-scalar values**: complex objects, like `Map<String, Object>`.

Rules:

1. **Scalar values (`Map<String, String>`):**
    - You can use keys with `.` directly.
    - Example:
        
        ```
        a.b=c
        ```
        
        binds to:
        
        ```java
        Map<String,String> map = {"a.b"="c"}
        ```
        
    - The `.` is preserved because it’s a scalar map.
2. **Non-scalar values (`Map<String, Object>`):**
    - Keys with `.` are treated as **nested maps** unless you use `[ ]`.
    - Example:
        
        ```
        a.b=c
        ```
        
        binds to:
        
        ```java
        Map<String,Object> map = {"a"={"b"="c"}}
        ```
        
    - If you want the key to be **literally `a.b`**, you must use brackets:
        
        ```
        [a.b]=c
        ```
        
        binds to:
        
        ```java
        Map<String,Object> map = {"a.b"="c"}
        ```
        

## **Binding From Environment Variables**

**OS Restrictions on Environment Variables**

Most operating systems (especially Unix/Linux) have strict rules for environment variable names:

- Only **letters (A-Z, a-z)**, **numbers (0-9)**, or **underscore (_)** are allowed.
- By convention, variable names are **UPPERCASE**.
- Special characters like `.` or  are **not allowed** in environment variable names.

Spring Boot’s relaxed binding handles this automatically, so that your normal configuration property names can still be mapped to environment variables.

---

**Rules for Mapping Property Names to Environment Variables**

To convert a **property name** (canonical form) into an environment variable, Spring Boot applies these rules:

1. **Replace dots (`.`) with underscores (`_`)**
    
    Example: `spring.main.log-startup-info` → `spring_main.log-startup-info` (intermediate)
    
2. **Remove dashes ()**
    
    Example: `spring_main.log-startup-info` → `spring_main_logstartupinfo`
    
3. **Convert everything to uppercase**
    
    Example: `spring_main_logstartupinfo` → `SPRING_MAIN_LOGSTARTUPINFO`
    

---

**Binding Lists from Environment Variables**

When binding to **lists or arrays**, the **index** of each element is included using underscores:

- Property: `my.service[0].other`
- Environment variable: `MY_SERVICE_0_OTHER`

Similarly, `my.service[1].other` → `MY_SERVICE_1_OTHER`

This allows Spring Boot to bind environment variables to Java `List` or array properties.

---

**Property Sources Supporting Environment Variables**

- Spring Boot applies this mapping to the **system environment** (`System.getenv()`).
- It also applies to any custom property source ending with `systemEnvironment`.

### **Binding Maps From Environment Variables**

**Lowercasing Environment Variable Names**

When Spring Boot reads an environment variable and binds it to a `@ConfigurationProperties` class:

- The **environment variable name** is automatically **converted to lowercase** before mapping.
- This lowercasing only affects the **keys** in a `Map` property, not the values.

---

**Example with a Map**

Consider this configuration class:

```java
@ConfigurationProperties("my.props")
public class MyMapsProperties {
    private final Map<String, String> values = new HashMap<>();
    public Map<String, String> getValues() { return this.values; }
}
```

- Property: `values` is a `Map<String, String>`.

If you set an environment variable like:

```
MY_PROPS_VALUES_KEY=value
```

Spring Boot does the following:

1. Converts the environment variable name to lowercase: `MY_PROPS_VALUES_KEY` → `my_props_values_key`.
2. Maps it to the `values` map, with the **key in lowercase**: `"key"`.
3. The value is kept **as-is**: `"value"`.

Resulting Map:

```java
{"key"="value"}
```

If the environment variable were:

```
MY_PROPS_VALUES_KEY=VALUE
```

Resulting Map:

```java
{"key"="VALUE"}
```

- Note that **only the key `"key"` is lowercased**.
- The **value `"VALUE"` retains its original case**.

### **Caching**

Relaxed binding uses a cache to improve performance. By default, this caching is only applied to immutable property sources. To customize this behavior, for example to enable caching for mutable property sources, use [`ConfigurationPropertyCaching`](https://docs.spring.io/spring-boot/3.5.6/api/java/org/springframework/boot/context/properties/source/ConfigurationPropertyCaching.html).

## **Merging Complex Types**

### **Lists in Spring Boot Configuration**

When you have a `List` property in a `@ConfigurationProperties` class:

```java
@ConfigurationProperties("my")
public class MyProperties {
    private final List<MyPojo> list = new ArrayList<>();
    public List<MyPojo> getList() { return list; }
}
```

- Each element of the list can be configured via indexed properties (`my.list[0]`, `my.list[1]`, etc.).
- **Important rule:** **If a list is defined in multiple property sources or profiles, Spring Boot does not merge the lists.**
- Instead, the **entire list is replaced** by the one from the highest-priority source.

---

Example 1: Single Entry with Profile Override

```
my.list[0].name=my name
my.list[0].description=my description
```

```
# dev profile
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

- **Without dev profile active:**
    - List contains one entry: `{name="my name", description="my description"}`
- **With dev profile active:**
    - List contains **only one entry**, not two.
    - Entry: `{name="my another name", description=null}`
- **Explanation:** The dev profile list **replaces** the previous list; it does not merge.

---

Example 2: Multiple List Items Across Profiles

```
my.list[0].name=my name
my.list[0].description=my description
my.list[1].name=another name
my.list[1].description=another description
```

```
# dev profile
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

- With dev profile active:
    - The list contains **only one entry** (from dev profile).
    - Previous items from the default profile are **ignored**.

**Key takeaway:** **Lists are fully replaced** when overridden in higher-priority sources (profiles, imported files, or command-line properties). Spring Boot does not merge lists.

---

### **Maps in Spring Boot Configuration**

For `Map` properties:

```java
@ConfigurationProperties("my")
public class MyProperties {
    private final Map<String, MyPojo> map = new LinkedHashMap<>();
    public Map<String, MyPojo> getMap() { return map; }
}
```

- Unlike lists, **Map properties are merged** from multiple sources.
- **Rule:** If a key exists in multiple sources, the value from the **highest-priority source wins**.
- New keys in higher-priority sources are **added**, not replaced.

---

Example 1: Map with Profile Override

```
my.map.key1.name=my name 1
my.map.key1.description=my description 1
```

```
# dev profile
spring.config.activate.on-profile=dev
my.map.key1.name=dev name 1
my.map.key2.name=dev name 2
my.map.key2.description=dev description 2
```

- **Without dev profile:**
    - Map contains one entry:
        - `key1` → `{name="my name 1", description="my description 1"}`
- **With dev profile active:**
    - Map contains **two entries**:
        - `key1` → `{name="dev name 1", description="my description 1"}` (name overridden, description preserved)
        - `key2` → `{name="dev name 2", description="dev description 2"}` (new entry added)

**Key takeaway:**

- **Maps are merged** across property sources.
- For conflicting keys, **higher-priority value wins**.
- For new keys, they are simply added to the existing map.

## **Properties Conversion**

### What is type coercion in Spring Boot?

When you define a configuration class, for example:

```java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private int port;
    private boolean enabled;
    private List<String> servers;

    // getters and setters
}
```

Spring Boot can automatically read values like:

```
app.port=8080
app.enabled=true
app.servers=server1,server2,server3
```

and **convert them from Strings (all properties are Strings in properties files) to the correct type**:

- `"8080"` → `int`
- `"true"` → `boolean`
- `"server1,server2,server3"` → `List<String>`

This automatic conversion is called **type coercion**.

---

### When default conversion is not enough

Sometimes, you have a property type that Spring Boot **doesn’t know how to convert automatically**. Examples:

- A custom class like `DurationRange`
- An enum with non-standard names
- Complex objects like `Map<String, MyCustomType>`

In such cases, you need to **provide custom logic** to tell Spring Boot how to convert the String from the properties file into your type.

---

### How to provide custom conversion

Spring Boot gives three main ways:

a) **ConversionService**

You can define a bean of type `ConversionService` and name it `conversionService`. Spring will use it to perform conversions during property binding.

Example:

```java
@Bean
public ConversionService conversionService() {
    DefaultConversionService service = new DefaultConversionService();
    service.addConverter(new StringToDurationRangeConverter());
    return service;
}
```

- This allows you to register **custom converters globally**.
- Any property binding that needs this type will automatically use your converter.

---

b) **Custom property editors**

You can use a `CustomEditorConfigurer` bean (older approach) to register **PropertyEditors**, which convert Strings to specific types.

Example:

```java
@Bean
public CustomEditorConfigurer customEditorConfigurer() {
    CustomEditorConfigurer configurer = new CustomEditorConfigurer();
    configurer.registerCustomEditor(DurationRange.class, new DurationRangeEditor());
    return configurer;
}
```

- Mostly used in legacy Spring projects.
- Converts Strings to your custom types during property binding.

---

c) **Custom converters with `@ConfigurationPropertiesBinding`**

Spring Boot also allows you to define **custom converters specifically for `@ConfigurationProperties` binding**:

```java
@WritingConverter
@ConfigurationPropertiesBinding
public class StringToDurationRangeConverter implements Converter<String, DurationRange> {
    @Override
    public DurationRange convert(String source) {
        return new DurationRange(source);
    }
}
```

- Annotate with `@ConfigurationPropertiesBinding` so Spring knows to use it **only for `@ConfigurationProperties` beans**.
- This is the modern and recommended approach for Spring Boot.

### **Converting Durations**

**`Duration` properties in Spring Boot**

If you have a configuration property that represents a **time duration**, you can use Java’s `java.time.Duration` type. For example:

```java
private Duration sessionTimeout;
private Duration readTimeout;
```

Spring Boot automatically **binds string values from properties files** (or YAML) into `Duration` objects.

---

**Supported formats**

Spring Boot supports multiple ways to express durations in your `application.properties` or `application.yml`:

1. **Long representation**
    - Treated as **milliseconds by default**, unless you specify another unit with `@DurationUnit`.
        
        Example:
        
    
    ```
    my.readTimeout=1000
    ```
    
    → Interpreted as 1000 milliseconds (1 second)
    
2. **ISO-8601 format** (used by `Duration`)
    - Standard `Duration` format: `PTnHnMnS`
        
        Example:
        
    
    ```
    my.sessionTimeout=PT30S
    ```
    
    → Interpreted as 30 seconds
    
3. **Readable format with value + unit**
    - You can append a unit like `ms`, `s`, `m`, `h`, `d` for milliseconds, seconds, minutes, hours, days.
        
        Example:
        
    
    ```
    my.sessionTimeout=30s
    my.readTimeout=500ms
    ```
    

---

**Using `@DurationUnit`**

By default, **plain numbers are treated as milliseconds**. If you want to change the default unit for a field, you can use `@DurationUnit`.

Example:

```java
@DurationUnit(ChronoUnit.SECONDS)
private Duration sessionTimeout = Duration.ofSeconds(30);
```

- Here, if you write `my.sessionTimeout=30`, it will interpret it as **30 seconds**, not milliseconds.
- Without `@DurationUnit`, `my.sessionTimeout=30` would mean 30 milliseconds.

---

**Constructor binding**

Spring Boot also supports **immutable configuration classes** with constructor injection. Example:

```java
public MyProperties(
    @DurationUnit(ChronoUnit.SECONDS) @DefaultValue("30s") Duration sessionTimeout,
    @DefaultValue("1000ms") Duration readTimeout) {
    this.sessionTimeout = sessionTimeout;
    this.readTimeout = readTimeout;
}
```

- `@DefaultValue` specifies a default duration if no value is provided in the properties.
- `@DurationUnit` sets the default unit for **plain numbers**.
- The binding works exactly the same way as with setters.

### **Converting Periods**

**`Period` type in Spring Boot**

`Period` represents a **date-based amount**, e.g., “2 years, 3 months, 5 days.” You can use it in a `@ConfigurationProperties` class:

```java
@ConfigurationProperties("my")
public class MyProperties {
    private Period vacationPeriod;
    private Period trialPeriod;

    // getters/setters
}
```

Spring Boot will **bind string values from your properties into Period objects** automatically.

---

**Supported formats**

There are three ways to specify a `Period` in properties:

a) Regular integer representation

- Default unit: **days**
- Plain numbers are interpreted as days unless you specify a different unit using `@PeriodUnit`.
    
    Example:
    

```
my.vacationPeriod=10
```

- Interpreted as 10 days.

---

b) ISO-8601 format

- Standard `Period` format: `PnYnMnD`
- Example:

```
my.vacationPeriod=P1Y3D
```

- Means **1 year and 3 days**.
- `P` stands for “period,” `Y` for years, `M` for months, `D` for days.
- This is exactly the same format used by `java.time.Period.parse()`.

---

c) Simple value+unit format

- You can write a **compact, readable string** with multiple unit-value pairs.
- Supported units:
    - `y` → years
    - `m` → months
    - `w` → weeks (converted internally to 7 days)
    - `d` → days

Example:

```
my.vacationPeriod=1y3d
my.trialPeriod=2w
```

- `1y3d` → 1 year and 3 days
- `2w` → 14 days (weeks are converted to days internally)

**Note:** `Period` never stores “weeks” as a separate field. It just converts them into days.

---

**Using `@PeriodUnit`**

If you want to interpret plain numbers in a unit other than days, you can use `@PeriodUnit`:

```java
@PeriodUnit(ChronoUnit.MONTHS)
private Period trialPeriod;
```

- Now, `my.trialPeriod=3` will be interpreted as **3 months** instead of 3 days.

### **Converting Data Sizes**

**`DataSize` properties in Spring Boot**

You can declare a configuration class like this:

```java
@ConfigurationProperties("my")
public class MyProperties {
    private DataSize bufferSize;
    private DataSize sizeThreshold;

    // getters and setters
}

```

Spring Boot will **automatically convert string values from properties into `DataSize` objects**.

---

**Supported formats**

There are two main ways to express a `DataSize` in properties:

a) Regular long representation

- Plain numbers are interpreted as **bytes by default**.
- You can override the default unit with `@DataSizeUnit`.

Example:

```
my.bufferSize=10
```

- Interpreted as 10 bytes unless `@DataSizeUnit` specifies otherwise.

---

b) Readable value+unit format

- You can append a unit to make it human-readable.
- Supported units:
    - `B` → bytes
    - `KB` → kilobytes
    - `MB` → megabytes
    - `GB` → gigabytes
    - `TB` → terabytes

Example:

```
my.bufferSize=10MB
my.sizeThreshold=256B
```

- `10MB` → 10 megabytes
- `256B` → 256 bytes

These are equivalent to the numeric representations if the default unit matches.

---

**Using `@DataSizeUnit`**

You can define a **default unit** for plain numbers using `@DataSizeUnit`:

```java
@DataSizeUnit(DataUnit.MEGABYTES)
private DataSize bufferSize = DataSize.ofMegabytes(2);
```

- Now `my.bufferSize=10` is interpreted as 10 megabytes instead of 10 bytes.
- Without this annotation, plain numbers default to bytes.

---

**Constructor binding**

You can also use **immutable configuration classes** with constructor injection:

```java
public MyProperties(
    @DataSizeUnit(DataUnit.MEGABYTES) @DefaultValue("2MB") DataSize bufferSize,
    @DefaultValue("512B") DataSize sizeThreshold) {
    this.bufferSize = bufferSize;
    this.sizeThreshold = sizeThreshold;
}
```

- `@DefaultValue` specifies a default if the property is not provided.
- `@DataSizeUnit` sets the unit for plain numbers.
- Works the same way as setter-based binding.

### **Converting Base64 Data**

**`Resource` properties**

In a configuration class, you can declare a property like this:

```java
@ConfigurationProperties("my")
public class MyProperties {
    private Resource property;

    // getters and setters
    public Resource getProperty() {
        return property;
    }
    public void setProperty(Resource property) {
        this.property = property;
    }
}
```

- The `Resource` type allows Spring to **abstract different ways of specifying a resource** (file, URL, classpath resource, etc.).

---

**Resolving from a file path**

- Normally, you can provide a file path or URL in the properties:

```
my.property=classpath:data.txt
my.property=file:/tmp/data.txt
my.property=https://example.com/data.txt
```

- Spring Boot automatically **loads the resource** at runtime.

---

**Resolving Base64-encoded data**

Spring Boot also supports **embedding the resource data directly in Base64**.

- Format: `base64:<base64-encoded-content>`
- Example:

```
my.property=base64:SGVsbG8gV29ybGQ=
```

- Here, `SGVsbG8gV29ybGQ=` is the Base64 encoding of `"Hello World"`.
- Spring Boot decodes it and makes it available as a `Resource`.
- This is useful when you want to **provide small binary resources directly in configuration** without using an external file.

---

**Versatility of `Resource`**

- The `Resource` property is flexible:
    1. File system path
    2. Classpath path
    3. URL
    4. Base64-encoded data
- At runtime, you can **read the contents** using standard Spring methods:

```java
InputStream inputStream = myProperties.getProperty().getInputStream();
String content = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
```

- This works the same whether the resource came from a file, URL, or Base64 string.

## **@ConfigurationProperties Validation**

### Enabling validation

Spring Boot can **automatically validate** configuration properties when you annotate your configuration class with `@Validated`:

```java
@ConfigurationProperties("my.service")
@Validated
public class MyProperties {
    // fields...
}
```

- `@Validated` tells Spring to **apply JSR-303 / Jakarta Bean Validation rules** to the fields of this class.
- Validation occurs **when the properties are bound**, i.e., when Spring reads values from `application.properties` or `application.yml` and sets them in the bean.

---

### Using JSR-303 / Jakarta Validation annotations

You can use standard validation annotations on fields, such as:

- `@NotNull` – field must not be null
- `@NotEmpty` – string or collection must not be empty
- `@Min` / `@Max` – numeric limits
- `@Email` – string must be a valid email
- Many others

Example:

```java
@NotNull
private InetAddress remoteAddress;
```

- If the property `my.service.remoteAddress` is missing or invalid, Spring will throw a **`BindException` or `ConstraintViolationException`** on startup.

**Requirements:**

- A compliant JSR-303 / Jakarta Bean Validation implementation must be on the classpath. Common choices:
    - Hibernate Validator (default in Spring Boot)
    - Apache BVal

---

### Cascading validation to nested properties

If your configuration class has nested properties, you can **cascade validation** using `@Valid`:

```java
@Valid
private final Security security = new Security();
```

- The nested class (`Security`) can have its own validation annotations:

```java
public static class Security {
    @NotEmpty
    private String username;
}
```

- Spring will validate `security.username` when binding.

---

### Validating beans created by `@Bean` methods

You can also validate configuration properties when they are created via `@Bean` methods:

```java
@Bean
@Validated
public MyProperties myProperties() {
    return new MyProperties();
}
```

- The `@Validated` annotation on the `@Bean` method ensures that Spring applies the validation rules.

---

### Custom Spring Validators

Spring Boot allows you to define a **custom validator** for configuration properties:

```java
@Bean
public static Validator configurationPropertiesValidator() {
    return new MyCustomValidator();
}
```

- The bean must be **named `configurationPropertiesValidator`**.
- The method should be **static** so that it can be instantiated early in the application lifecycle.
- This validator is applied **before standard binding occurs**, which is useful for custom rules not covered by JSR-303 annotations.

---

### Viewing configuration properties

If you include **Spring Boot Actuator**, you can **inspect all `@ConfigurationProperties` beans**:

- HTTP endpoint: `/actuator/configprops`
- JMX endpoint: `org.springframework.boot:type=ConfigProps,name=*`

This is useful for debugging or monitoring which properties were bound and their values.