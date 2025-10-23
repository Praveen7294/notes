# Logging

- [Introduction](#introduction)
- [Log Format](#log-format)
- [Console Output](#console-output)
    1. [Color-coded Output](#color-coded-output)
- [File Output](#file-output)
- [File Rotation](#file-rotation)
- [Log Levels](#log-levels)
- [Log Groups](#log-groups)
- [Using a Log Shutdown Hook](#using-a-log-shutdown-hook)
- [Custom Log Configuration](#custom-log-configuration)
- [Structured Logging](#structured-logging)
    1. [Elastic Common Schema (ECS)](#elastic-common-schema)
    2. [Graylog Extended Log Format (GELF)](#graylog-extended-log-format-gelf)
    3. [ECS vs GELF](#ecs-vs-gelf)
    4. [Logstash JSON Format](#logstash-json-format)
    5. [Customizing Structured Logging JSON](#customizing-structured-logging-json)
    6. [Customizing Structured Logging Stack Traces](#customizing-structured-logging-stack-traces)
    7. [Supporting Other Structured Logging Formats](#supporting-other-structured-logging-formats)
- [Logback Extensions](#logback-extensions)
    1. [Profile-specific Configuration](#profile-specific-configuration)
    2. [Environment Properties](#environment-properties)
- [Log4j2 Extensions](#log4j2-extensions)
    1. [Profile-specific Configuration](#profile-specific-configuration-1)
    2. [Environment Properties Lookup](#environment-properties-lookup)
    3. [Log4j2 System Properties](#log4j2-system-properties)

# Introduction

### **Commons Logging as a Facade**

Spring Boot uses **Apache Commons Logging (JCL)** internally as a *logging facade*.

A **facade** means it’s not doing the logging itself—it just provides a *uniform interface* so Spring Boot can log messages without caring about the actual logging framework underneath.

So inside Spring Boot’s code, when it logs something (like startup info), it uses Commons Logging APIs.

But those logs are actually handled by whatever **real logging implementation** is present in your project.

---

### **Underlying Logging Implementations**

There are multiple logging implementations available in Java:

- **Java Util Logging (JUL)** — built into Java (`java.util.logging`)
- **Log4j2**
- **Logback**

Spring Boot supports all three and provides default configurations for them.

---

### **Default Choice: Logback**

If you are using **Spring Boot Starter** dependencies (like `spring-boot-starter-web`),

then by default, **Logback** is included as the underlying logging framework.

That means all logs (including Spring’s internal ones) will be written using Logback.

---

### **Routing Between Different Frameworks**

You might have other libraries in your project that use different logging frameworks, for example:

- One library might use `java.util.logging`
- Another might use `Log4j`
- Another might use `Commons Logging`

Spring Boot automatically **routes all of them** to **Logback**, so everything ends up in one place (your console or log file).

This routing is handled by special adapters on the classpath (like `jul-to-slf4j`, `log4j-to-slf4j`, etc.).

So even though different parts of your app might use different logging APIs, all logs are unified.

---

### **Logging Output Configuration**

By default, Spring Boot’s logging configuration:

- Prints logs to the **console**
- Optionally can write to a **file** (if you configure it)

You don’t need to manually configure this—it just works out of the box.

---

### **When Deployed to an Application Server**

If you deploy your Spring Boot app as a **WAR** file inside a servlet container (like Tomcat or JBoss),

logging can get tricky.

That’s because the **container itself** uses `java.util.logging` internally.

Spring Boot’s routing only applies inside your application context — not outside it.

So logs created by the **container** or **other deployed applications** using `java.util.logging` won’t be redirected into your Spring Boot app’s logs.

This separation is intentional — it prevents logs from other apps or the container itself from mixing with your app’s logs.

# **Log Format**

The default log output from Spring Boot resembles the following example:

```bash
2025-09-18T11:18:46.890Z  INFO 139141 --- [myapp] [           main] o.s.b.d.f.logexample.MyApplication       : Starting MyApplication using Java 17.0.16 with PID 139141 (/opt/apps/myapp.jar started by myuser in /opt/apps/)
2025-09-18T11:18:46.901Z  INFO 139141 --- [myapp] [           main] o.s.b.d.f.logexample.MyApplication       : No active profile set, falling back to 1 default profile: "default"
2025-09-18T11:18:50.607Z  INFO 139141 --- [myapp] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2025-09-18T11:18:50.674Z  INFO 139141 --- [myapp] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2025-09-18T11:18:50.677Z  INFO 139141 --- [myapp] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.46]
2025-09-18T11:18:50.863Z  INFO 139141 --- [myapp] [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-09-18T11:18:50.873Z  INFO 139141 --- [myapp] [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 3724 ms
2025-09-18T11:18:52.159Z  INFO 139141 --- [myapp] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2025-09-18T11:18:52.208Z  INFO 139141 --- [myapp] [           main] o.s.b.d.f.logexample.MyApplication       : Started MyApplication in 7.341 seconds (process running for 8.401)
2025-09-18T11:18:52.230Z  INFO 139141 --- [myapp] [ionShutdownHook] o.s.b.w.e.tomcat.GracefulShutdown        : Commencing graceful shutdown. Waiting for active requests to complete
2025-09-18T11:18:52.249Z  INFO 139141 --- [myapp] [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : Graceful shutdown complete
```

The following items are output:

- Date and Time: Millisecond precision and easily sortable.
- Log Level: `ERROR`, `WARN`, `INFO`, `DEBUG`, or `TRACE`.
- Process ID.
- A `--` separator to distinguish the start of actual log messages.
- Application name: Enclosed in square brackets (logged by default only if `spring.application.name` is set)
- Application group: Enclosed in square brackets (logged by default only if `spring.application.group` is set)
- Thread name: Enclosed in square brackets (may be truncated for console output).
- Correlation ID: If tracing is enabled (not shown in the sample above)
- Logger name: This is usually the source class name (often abbreviated).
- The log message.

### **Logback does not have a FATAL level — it’s mapped to ERROR**

In logging systems, **log levels** represent the severity of a message.

Common levels (from most to least severe) are:

`FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`.

However, **Logback** (the default logging framework in Spring Boot) only defines these five:

```
ERROR, WARN, INFO, DEBUG, TRACE
```

It does **not** have a separate **FATAL** level.

So, if some library or framework logs a message with level **FATAL**, Logback automatically treats it as an **ERROR** level message.

In other words:

```
FATAL → ERROR
```

This mapping ensures compatibility with other logging frameworks (like Log4j) that do have a FATAL level.

---

### **`spring.application.name` and logging.include-application-name**

If you set your application name in `application.properties` like this:

```
spring.application.name=myapp
```

Spring Boot automatically includes the application name in each log line:

```
2025-09-18T11:18:46.890Z  INFO 139141 --- [myapp] [main] ...
```

But if you **don’t** want the application name to appear in your logs, you can disable it by setting:

```
logging.include-application-name=false
```

Then your logs would look like:

```
2025-09-18T11:18:46.890Z  INFO 139141 --- [main] ...
```

---

### **`spring.application.group` and logging.include-application-group**

Similarly, Spring Boot 3.3 introduced the `spring.application.group` property to group related applications under a common label (useful for multi-module systems or environments with many microservices).

Example:

```
spring.application.group=payment-services
```

By default, if this property is set, the group name also appears in logs:

```
2025-09-18T11:18:46.890Z  INFO 139141 --- [myapp] [payment-services] [main] ...
```

If you don’t want this group name in your logs, you can disable it using:

```
logging.include-application-group=false
```

Then it will be omitted from your log output.

> Note:
> 
> - If we remove the application name from the application.properties then application name will not display on the console.

# **Console Output**

### **Default Logging Behavior**

When you run a Spring Boot application, it automatically shows logs in the **console** (standard output).

By default, it logs only the following levels:

- **ERROR** – serious issues that cause failures
- **WARN** – potential problems or warnings
- **INFO** – general informational messages (like startup events)

So you’ll see important operational messages, but not detailed debugging information.

Example:

```
2025-09-18T11:18:50.607Z  INFO 139141 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
```

---

### **Enabling Debug Mode**

Sometimes you need **more detailed logs** to understand what’s happening internally in Spring Boot.

You can turn on **debug mode** in two ways:

**Option 1: Command-line flag**

```bash
java -jar myapp.jar --debug
```

**Option 2: Configuration file**

```
debug=true
```

---

### **What Debug Mode Actually Does**

When `--debug` (or `debug=true`) is enabled:

- It **does not** make all classes or libraries log everything at the DEBUG level.
- Instead, it turns on **DEBUG logging only for a specific set of “core loggers”** — mainly those related to Spring Boot internals and common frameworks.

Specifically, debug mode enables DEBUG-level logs for:

- The **embedded servlet container** (like Tomcat, Jetty, or Undertow)
- **Spring Boot’s own startup and auto-configuration logs**
- **Hibernate ORM** (but only limited debug info)

This helps you see:

- Which auto-configurations were applied or excluded
- Internal initialization steps of the embedded server
- Basic Hibernate setup details

Example:

```
2025-09-18T11:18:50.123 DEBUG 139141 --- [main] o.s.b.a.AutoConfigurationReportLoggingInitializer :
    Negative matches:
    -----------------
    DataSourceAutoConfiguration:
       Did not match: no supported DataSource type
```

So debug mode is designed for **developers** who want to understand *why* Spring Boot configured things in a certain way.

---

### **Enabling Trace Mode**

If you want even more detailed logs, you can enable **trace mode**.

**Option 1: Command-line flag**

```bash
java -jar myapp.jar --trace
```

**Option 2: Configuration file**

```
trace=true
```

---

### **What Trace Mode Does**

Trace mode provides **the most detailed** logging for certain parts of Spring Boot.

It enables TRACE-level logs for:

- The **embedded web container** (Tomcat/Jetty)
- **Hibernate schema generation**
- The entire **Spring Framework and Spring Boot** internals

TRACE mode logs nearly everything that happens inside the core framework, which is very verbose.

It’s mainly useful for **troubleshooting very specific or complex issues**, not for everyday use.

Example:

```
2025-09-18T11:18:50.456 TRACE 139141 --- [main] o.s.c.a.ConfigurationClassParser : Parsing configuration class [com.ex
```

## **Color-coded Output**

### **Color Output in the Terminal**

When you run a Spring Boot application in a terminal, it can display **colored log output** to make logs easier to read.

For example:

- Errors may appear in **red**
- Warnings in **yellow**
- Info messages in **green**

This helps visually separate different log levels.

Spring Boot checks automatically whether your terminal supports **ANSI color codes** (which most modern terminals do).

If it does, colors are enabled by default.

---

### **Controlling Color Output**

You can manually control whether color output is used by setting this property in `application.properties`:

```
spring.output.ansi.enabled=ALWAYS
```

Possible values:

- `ALWAYS` → Force colors on (even if the terminal might not support it)
- `NEVER` → Disable colors entirely
- `DETECT` → (default) Spring Boot automatically detects terminal support

So, for example:

```
spring.output.ansi.enabled=NEVER
```

would make all logs appear in plain text without color.

---

### **The `%clr` Conversion Word**

Spring Boot uses **Logback** under the hood for logging.

Logback defines how each line is formatted using a **pattern layout**, such as:

```
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} %5p --- [%t] %c : %m%n
```

The `%clr()` converter adds **color** to parts of that pattern.

Example:

```
%clr(%5p)
```

Here:

- `%5p` → displays the log level (like `INFO`, `WARN`, `ERROR`)
- `%clr(...)` → wraps that output in color, depending on the log level

So if the log level is:

- `ERROR` or `FATAL` → colored **red**
- `WARN` → colored **yellow**
- `INFO`, `DEBUG`, or `TRACE` → colored **green**

---

### **Mapping of Log Levels to Colors**

| Log Level | Default Color |
| --- | --- |
| FATAL | Red |
| ERROR | Red |
| WARN | Yellow |
| INFO | Green |
| DEBUG | Green |
| TRACE | Green |

This default mapping helps you identify severity instantly when reading logs.

---

### **Customizing Colors**

You’re not limited to automatic coloring.

You can explicitly choose a color or style by specifying it after the `%clr()` converter.

Example:

```
%clr(%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}){yellow}
```

Here:

- `%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}` → prints the timestamp in ISO format
- `{yellow}` → forces the timestamp text to appear in **yellow**

You can also combine other color names or styles.

---

### **Supported Colors and Styles**

Spring Boot supports these color/style names in Logback patterns:

```
blue
cyan
faint
green
magenta
red
yellow
```

You can apply them to **any part** of your log format, not just the log level.

Example – colorize different parts:

```
logging.pattern.console=%clr(%d{HH:mm:ss.SSS}){cyan} %clr(%5p){red} %clr(%-40.40logger{39}){blue} : %m%n
```

This will:

- Print timestamp in **cyan**
- Log level in **red**
- Logger name in **blue**
- Message in default color

# **File Output**

By default, Spring Boot logs only to the console and does not write log files. If you want to write log files in addition to the console output, you need to set a `logging.file.name` or `logging.file.path` property (for example, in your `application.properties`). If both properties are set, `logging.file.path` is ignored and only `logging.file.name` is used.

The following table shows how the `logging.*` properties can be used together:

| **`logging.file.name`** | **`logging.file.path`** | **Description** |
| --- | --- | --- |
| *(none)* | *(none)* | Console only logging. |
| Specific file (for example, `my.log`) | *(none)* | Writes to the location specified by `logging.file.name`. The location can be absolute or relative to the current directory. |
| *(none)* | Specific directory (for example, `/var/log`) | Writes `spring.log` to the directory specified by `logging.file.path`. The directory can be absolute or relative to the current directory. |
| Specific file | Specific directory | Writes to the location specified by `logging.file.name` and ignores `logging.file.path`. The location can be absolute or relative to the current directory. |

Log files rotate when they reach 10 MB and, as with console output, `ERROR`-level, `WARN`-level, and `INFO`-level messages are logged by default.

### **Example Configurations**

**Write logs to a specific file:**

```
logging.file.name=app.log
```

**Write logs to a directory (Spring Boot will create spring.log):**

```
logging.file.path=logs/
```

**Write to both console and file:**

(no extra setup needed — file logging doesn’t disable console output)

If you want to **set both a custom location and a custom name** for your log file in Spring Boot, you should use the property:

```
logging.file.name=<full path to your log file>
```

# **File Rotation**

Log rotation is the process of automatically creating new log files after certain conditions are met (like size or time) and optionally deleting old logs to save disk space. Spring Boot provides some properties that let you control this behavior directly in your `application.properties` or `application.yaml` **if you are using Logback**.

**`logging.logback.rollingpolicy.file-name-pattern`**

- **Purpose:** Specifies the **pattern for the filenames of archived log files**.
- Log rotation works by creating a new log file after certain conditions (like size or time) are met. This property controls the naming of those rotated files.
- **Syntax Example:**
    
    ```
    logging.logback.rollingpolicy.file-name-pattern=logs/app-%d{yyyy-MM-dd}.%i.log
    ```
    
    - `%d{yyyy-MM-dd}` → the date the log was created (e.g., `2025-10-22`)
    - `%i` → index number if multiple logs are created on the same day
- **Effect:** If your app generates multiple log files per day, the files will be named something like:
    
    ```
    app-2025-10-22.1.log
    app-2025-10-22.2.log
    ```
    

---

**`logging.logback.rollingpolicy.clean-history-on-start`**

- **Purpose:** Determines whether **old log files are automatically deleted when the application starts**.
- **Values:** `true` or `false`
    - `true` → the system will check the rotation history and delete old log files that exceed limits (`max-history` or `total-size-cap`) when the app starts.
    - `false` → no cleanup is done on startup; rotation will only delete logs based on limits when new logs are created.
- **Example:**
    
    ```
    logging.logback.rollingpolicy.clean-history-on-start=true
    ```
    
- **Effect:** If your `max-history` is set to 7, only the latest 7 rotated files will remain; older files will be deleted when the app restarts.

---

**`logging.logback.rollingpolicy.max-file-size`**

- **Purpose:** Sets the **maximum size of a single log file** before it is archived/rotated.
- **Format:** Supports units like `KB`, `MB`, `GB`.
- **Example:**
    
    ```
    logging.logback.rollingpolicy.max-file-size=10MB
    ```
    
- **Effect:** Once the current log file exceeds 10 MB, Logback will create a new file following the `file-name-pattern`.

---

 **`logging.logback.rollingpolicy.total-size-cap`**

- **Purpose:** Limits the **total size of all archived log files**.
- **Format:** Same as `max-file-size`.
- **Example:**
    
    ```
    logging.logback.rollingpolicy.total-size-cap=1GB
    ```
    
- **Effect:** If the combined size of all rotated logs exceeds 1 GB, the **oldest logs are deleted first** until the total size is under 1 GB.

---

**`logging.logback.rollingpolicy.max-history`**

- **Purpose:** Controls **how many rotated log files are kept**.
- **Default:** 7
- **Example:**
    
    ```
    logging.logback.rollingpolicy.max-history=10
    ```
    
- **Effect:** Logback will keep only the last 10 rotated files. Older files are deleted automatically.

### Important

**log rotation settings only work with Logback**. If your project uses **another logging framework**, like **Log4j2**, those Spring Boot properties won’t have any effect.

Instead, you need to configure Log4j2 **directly using its own configuration files**, which are typically named:

1. **`log4j2.xml`** – the standard XML configuration file for Log4j2.
2. **`log4j2-spring.xml`** – a special variant recognized by Spring Boot that allows **Spring-specific features**, like Spring placeholders (`${property.name}`) in log configuration.

These files let you define:

- Log file location and name
- Log rotation policy (by size, by date, or both)
- Number of backups or maximum history
- Logging levels per package or class

**Example Concept:**

If you want to rotate logs daily and keep 7 days of logs in Log4j2, you can’t just set `logging.logback.rollingpolicy.max-history=7`. Instead, you need something like this in `log4j2.xml`:

```xml
<Appenders>
    <RollingFile name="FileAppender" fileName="logs/app.log"
                 filePattern="logs/app-%d{yyyy-MM-dd}.log">
        <Policies>
            <TimeBasedTriggeringPolicy />
        </Policies>
        <DefaultRolloverStrategy max="7"/>
    </RollingFile>
</Appenders>
```

Here:

- `filePattern` defines the rotated log filenames.
- `TimeBasedTriggeringPolicy` rotates logs daily.
- `max="7"` keeps only the last 7 files.

So, the key idea is: **for logging frameworks other than Logback, you must use their native configuration files** to set up rotation and other advanced logging behaviors.

# **Log Levels**

### **Logging levels in Spring Boot**

Spring Boot supports the standard logging levels:

- **TRACE** – most detailed, usually only for troubleshooting.
- **DEBUG** – detailed information useful for debugging.
- **INFO** – general runtime information (default level).
- **WARN** – warnings that are not errors but may indicate problems.
- **ERROR** – errors that have occurred.
- **FATAL** – very severe errors that may terminate the application (some frameworks support it).
- **OFF** – disables logging.

---

### **Setting logging levels using `application.properties` or `application.yaml`**

You can control logging **per package or per class** by specifying the logger name:

```
logging.level.<logger-name>=<level>
```

- `<logger-name>` → usually a package or class name
- `<level>` → one of TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

**Examples in `application.properties`:**

```
# Set default/root logging level
logging.level.root=warn

# Set logging level for Spring MVC web package
logging.level.org.springframework.web=debug

# Set logging level for Hibernate
logging.level.org.hibernate=error
```

**Equivalent in YAML (`application.yaml`):**

```yaml
logging:
  level:
    root: warn
    org.springframework.web: debug
    org.hibernate: error
```

**Explanation of the example:**

- `logging.level.root=warn` → By default, only warnings and errors from all packages are logged.
- `logging.level.org.springframework.web=debug` → Detailed logs for Spring MVC web package.
- `logging.level.org.hibernate=error` → Only errors from Hibernate are logged; INFO/DEBUG are ignored.

This allows **fine-grained control**: you can keep your logs clean but still get detailed info for specific packages you are troubleshooting.

---

### **Setting logging levels using environment variables**

Spring Boot also allows you to set logging levels via environment variables. The environment variable format is:

```
LOGGING_LEVEL_<LOGGER_NAME>=<LEVEL>
```

- Replace dots (`.`) in the package name with underscores (`_`)
- Uppercase the letters

**Example:**

```bash
LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG
```

- This is equivalent to setting `logging.level.org.springframework.web=debug` in properties.
- Useful for **Docker containers, cloud deployments, or CI/CD pipelines** where you don’t want to edit property files.

---

### **Limitations of environment variables**

- Environment variables in Spring Boot are **lowercased during relaxed binding**, so **per-class logging is not supported directly** via environment variables.
- For example, setting `LOGGING_LEVEL_COM_EXAMPLE_MYCLASS=DEBUG` may not work reliably.

**Workaround:** Use the `SPRING_APPLICATION_JSON` environment variable, which allows **JSON-formatted configuration** including class-level loggers:

```bash
SPRING_APPLICATION_JSON='{"logging.level.com.example.MyClass":"DEBUG"}'
```

- This lets you configure **individual class logging levels** via environment variables.
- Spring Boot will read this JSON at startup and apply the logging levels accordingly.

# **Log Groups**

### **The Problem — Managing Multiple Related Loggers**

In a large Spring Boot application, different parts of the framework or libraries have their own logging packages.

For example, if you use **Tomcat** (the embedded web server), it uses several internal packages such as:

- `org.apache.catalina`
- `org.apache.coyote`
- `org.apache.tomcat`

If you want to adjust logging for Tomcat, you’d normally have to set each one like this:

```
logging.level.org.apache.catalina=debug
logging.level.org.apache.coyote=debug
logging.level.org.apache.tomcat=debug
```

This becomes repetitive and hard to maintain.

To solve that, **Spring Boot allows you to create a “logging group”** — a custom name that refers to multiple loggers.

---

### **Defining a Custom Logging Group**

You can define your own logging group in your configuration file.

**Example (in `application.properties`):**

```
logging.group.tomcat=org.apache.catalina,org.apache.coyote,org.apache.tomcat
```

**Example (in `application.yaml`):**

```yaml
logging:
  group:
    tomcat: org.apache.catalina,org.apache.coyote,org.apache.tomcat
```

This defines a **logging group named `tomcat`**, which includes all three Tomcat-related packages.

---

### **Using the Logging Group**

Once you define a group, you can set the logging level for the entire group using **one line**:

```
logging.level.tomcat=trace
```

or

```yaml
logging:
  level:
    tomcat: trace
```

**Effect:**

Spring Boot automatically applies the `TRACE` level to all the loggers inside the group:

- `org.apache.catalina`
- `org.apache.coyote`
- `org.apache.tomcat`

So you don’t need to repeat three separate `logging.level` lines.

---

### **Predefined Logging Groups in Spring Boot**

Spring Boot already includes some **built-in (predefined)** groups that you can use right away — you don’t need to define them manually.

| Group Name | Included Loggers |
| --- | --- |
| **web** | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web`, `org.springframework.boot.actuate.endpoint.web`, `org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| **sql** | `org.springframework.jdbc.core`, `org.hibernate.SQL`, `LoggerListener` |

These predefined groups help with common use cases:

- **`web` group**: Controls logging for all web-related components like Spring MVC, HTTP, and actuator endpoints.
- **`sql` group**: Controls logging for database access and SQL queries (via Spring JDBC and Hibernate).

So, for example, to see all SQL queries executed by Hibernate and JDBC, you can simply add:

```
logging.level.sql=debug
```

or to trace all HTTP requests and responses from the web layer:

```
logging.level.web=trace
```

# **Using a Log Shutdown Hook**

### **What is a Shutdown Hook**

A **shutdown hook** is a special mechanism in Java that allows you to run specific code **when the JVM (Java Virtual Machine) is shutting down** — for example, when:

- You stop the Spring Boot application manually (e.g., pressing Ctrl+C).
- The application exits normally.
- The operating system sends a termination signal.

Spring Boot automatically uses a shutdown hook for **logging cleanup**.

---

### **Purpose of the Logging Shutdown Hook**

When a Spring Boot application stops, the logging system (such as Logback, Log4j2, or Java Util Logging) may still hold **open resources** such as:

- File handles for log files
- Background logging threads
- Buffered log data not yet written to disk

The **shutdown hook ensures that all these resources are properly released and flushed** before the JVM terminates.

In simple terms, it makes sure:

- All logs are completely written to disk.
- No file remains locked.
- The logging framework shuts down gracefully.

Without this cleanup, you might encounter:

- **Incomplete log entries** (last messages missing).
- **File locking issues** if you restart the app quickly.
- **Memory leaks** in complex logging setups.

---

### **Automatic Registration**

Spring Boot **automatically registers this logging shutdown hook** for you when your application starts.

That means you normally don’t need to do anything — it’s handled automatically.

However, there’s one exception:

- **If your Spring Boot application is packaged as a WAR file and deployed to an external servlet container** (like Tomcat, Jetty, or WildFly), the **shutdown hook is not registered automatically**.

This is because the **application lifecycle is controlled by the servlet container**, not by Spring Boot directly, so the container itself handles cleanup.

---

### **When the Default Hook May Not Work Well**

The automatic shutdown hook may not be ideal in some advanced setups — especially when you have:

- **Multiple application contexts** (e.g., parent and child contexts in a complex Spring setup).
- **Custom logging environments** where each component has its own logging configuration or context.

In such cases, Spring Boot’s single global shutdown hook might:

- Close logging contexts too early.
- Interfere with other contexts that are still running.
- Cause “logger not found” errors if one context shuts down before another.

If you face such issues, you can **disable the Spring Boot-provided shutdown hook** and manage logging cleanup manually using the features provided by your logging system.

---

### **How to Disable the Shutdown Hook**

To disable it, you use the property:

**In `application.properties`:**

```
logging.register-shutdown-hook=false
```

**Or in `application.yaml`:**

```yaml
logging:
  register-shutdown-hook: false
```

When set to `false`, Spring Boot will **not register its default shutdown hook**, and you must handle logging cleanup yourself.

---

### **Example: Using Logback Context Selectors**

If you disable Spring Boot’s shutdown hook, you can use advanced features provided by Logback (or your chosen logging framework).

For example, Logback provides **context selectors**, which allow:

- Each application or component to have its own **separate logging context**.
- Independent initialization and shutdown of each logger.

This is useful if you have multiple applications or contexts running within the same JVM.

Example concept (not Spring Boot config):

```xml
<configuration>
  <contextName>MyAppLoggerContext</contextName>
</configuration>
```

Then, you can shut down specific logging contexts when needed using Logback’s API rather than relying on a global Spring Boot hook.

# **Custom Log Configuration**

## Logging in Spring Boot – Overview

Spring Boot comes with a **powerful and flexible logging framework** that works automatically once your application starts.

It sets up a **default logging system** (Logback by default) before the Spring `ApplicationContext` is created, so that even startup events are logged.

However, you can switch to other logging systems like **Log4j2** or **Java Util Logging (JUL)** by changing what’s on your **classpath** or by using a **system property**.

---

## How Spring Boot Detects and Activates Logging Systems

Spring Boot automatically decides which logging system to use based on the **libraries present on the classpath**:

| Logging System | Library Dependency Example |
| --- | --- |
| **Logback** (default) | `ch.qos.logback:logback-classic` |
| **Log4j2** | `org.apache.logging.log4j:log4j-core` |
| **Java Util Logging (JUL)** | Built into JDK |

So:

- If **Logback** is present → Spring Boot uses Logback.
- If **Log4j2** is present → Spring Boot uses Log4j2.
- If neither is found → It falls back to **JUL (Java Util Logging)**.

You can **force** Spring Boot to use a specific logging system even if others are available.

---

## Forcing a Logging System

To explicitly choose a logging system, use the **system property**:

```
-Dorg.springframework.boot.logging.LoggingSystem=<fully-qualified-class-name>
```

For example:

```bash
-Dorg.springframework.boot.logging.LoggingSystem=org.springframework.boot.logging.log4j2.Log4J2LoggingSystem
```

If you want to **disable Spring Boot’s logging configuration** entirely:

```bash
-Dorg.springframework.boot.logging.LoggingSystem=none
```

This means no automatic configuration — you must configure logging manually.

---

## Logging Configuration Files

Each logging system has its **own configuration file format**.

Spring Boot automatically looks for specific files in the classpath root or in a path defined by `logging.config`.

| Logging System | Supported Configuration Files |
| --- | --- |
| **Logback** | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, `logback.groovy` |
| **Log4j2** | `log4j2-spring.xml`, `log4j2.xml` |
| **Java Util Logging (JUL)** | `logging.properties` |

### Recommendation:

Use the **`-spring` variant** (e.g., `logback-spring.xml`) because:

- Spring Boot can process it before the logging system initializes fully.
- It allows using **Spring environment properties** (like `${logging.file.name}`) inside the file.

If you use plain `logback.xml`, Spring Boot can’t inject its configuration values, since logging starts **before** the ApplicationContext is ready.

---

## Setting Custom Logging Configuration Location

You can specify a custom config file path using:

```
logging.config=classpath:my-custom-logback-spring.xml
```

or

```
logging.config=file:/path/to/log4j2-spring.xml
```

---

## Environment Properties That Affect Logging

Spring Boot automatically copies certain **Spring Environment properties** (from `application.properties` or environment variables) into **System properties**.

This allows the logging system (like Logback) to use these variables inside its configuration files.

Here’s the mapping:

| Spring Property | System Property | Description |
| --- | --- | --- |
| `logging.exception-conversion-word` | `LOG_EXCEPTION_CONVERSION_WORD` | Defines how exceptions are logged. |
| `logging.file.name` | `LOG_FILE` | Custom log file name. |
| `logging.file.path` | `LOG_PATH` | Directory for log files. |
| `logging.pattern.console` | `CONSOLE_LOG_PATTERN` | Pattern for console logs. |
| `logging.pattern.file` | `FILE_LOG_PATTERN` | Pattern for log files. |
| `logging.pattern.dateformat` | `LOG_DATEFORMAT_PATTERN` | Date format in logs. |
| `logging.pattern.level` | `LOG_LEVEL_PATTERN` | Log level display format. |
| `logging.charset.console` | `CONSOLE_LOG_CHARSET` | Charset for console logs. |
| `logging.charset.file` | `FILE_LOG_CHARSET` | Charset for log files. |
| `logging.threshold.console` | `CONSOLE_LOG_THRESHOLD` | Minimum log level for console logs. |
| `logging.threshold.file` | `FILE_LOG_THRESHOLD` | Minimum log level for file logs. |
| `logging.structured.format.console` | `CONSOLE_LOG_STRUCTURED_FORMAT` | Structured logging format (e.g., JSON). |
| `logging.structured.format.file` | `FILE_LOG_STRUCTURED_FORMAT` | Structured logging format for files. |
| `PID` | `PID` | Process ID (auto-discovered if possible). |

These System properties are available to your logging configuration file automatically.

---

## Logback-Specific Properties

If you are using **Logback**, Spring Boot supports additional settings for **rolling policy**, which controls how log files rotate and archive.

| Spring Property | System Property | Description |
| --- | --- | --- |
| `logging.logback.rollingpolicy.file-name-pattern` | `LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN` | Pattern for archived logs (default `${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`). |
| `logging.logback.rollingpolicy.clean-history-on-start` | `LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START` | Whether to delete old logs on startup. |
| `logging.logback.rollingpolicy.max-file-size` | `LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE` | Maximum size before rolling occurs. |
| `logging.logback.rollingpolicy.total-size-cap` | `LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP` | Total disk space allowed for all logs. |
| `logging.logback.rollingpolicy.max-history` | `LOGBACK_ROLLINGPOLICY_MAX_HISTORY` | How many archived log files to retain. |

You can use these directly in `application.properties`, and Spring Boot will inject them into Logback.

---

## Placeholder Syntax

When you use placeholders in logging configuration files (like `${LOG_FILE}`), **use Spring Boot’s syntax**.

- Correct syntax:
    
    `${propertyName:defaultValue}`
    
- Do **not** use Logback’s `:-` syntax.

Example:

```xml
<fileName>${LOG_FILE:application.log}</fileName>
```

---

## MDC (Mapped Diagnostic Context) and Custom Log Content

You can enhance log output using **MDC (Mapped Diagnostic Context)** — this allows adding contextual information (like user ID, request ID) to every log line.

To include MDC values in logs, modify the `logging.pattern.level` or `LOG_LEVEL_PATTERN`.

Example:

```
logging.pattern.level=user:%X{user} %5p
```

If the `user` key exists in MDC, the log will include it:

```
2019-08-30 12:30:04.031 user:someone INFO 22174 --- [nio-8080-exec-0] demo.Controller : Handling authenticated request
```

This is useful for tracking which user or request produced certain log entries.

---

## Important Notes

1. **@PropertySource Limitation**:
    
    Logging starts **before** the ApplicationContext loads `@Configuration` classes, so you cannot configure logging from `@PropertySource`.
    
    Only **system properties** or **environment variables** work during early logging initialization.
    
2. **Executable JAR Caution (Java Util Logging)**:
    
    `java.util.logging` (JUL) has **classloader issues** when used in executable jars, so avoid using it with Spring Boot if possible.
    
3. **Default Configurations**:
    
    Spring Boot’s default configurations for Logback, Log4j2, and JUL are packaged inside `spring-boot.jar` under
    
    `/org/springframework/boot/logging/`.
    

---

## Example Summary

### application.properties

```
logging.file.name=logs/app.log
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
logging.level.root=INFO
logging.logback.rollingpolicy.max-history=10
```

### logback-spring.xml

```xml
<configuration>
    <property name="LOG_PATH" value="logs" />
    <property name="LOG_FILE" value="${LOG_PATH}/app.log" />

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN}</fileNamePattern>
            <maxHistory>${LOGBACK_ROLLINGPOLICY_MAX_HISTORY}</maxHistory>
        </rollingPolicy>
    </appender>

    <root level="INFO">
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

# **Structured Logging**

## What is Structured Logging?

Normally, application logs are plain text lines like this:

```
2025-10-23 12:30:04 INFO  com.example.App - User login successful
```

While readable for humans, such logs are **hard for machines** to parse, search, or analyze.

**Structured logging** solves this by writing log output in a **machine-readable format**, such as **JSON**, with key–value pairs.

For example:

```json
{
  "timestamp": "2025-10-23T12:30:04.123Z",
  "level": "INFO",
  "logger": "com.example.App",
  "thread": "main",
  "message": "User login successful"
}
```

This makes logs easy to:

- Query in log management tools (e.g., Elasticsearch, Splunk, Graylog)
- Filter by fields (like log level, user ID, etc.)
- Correlate events across services in distributed systems

---

## Spring Boot’s Built-in Structured Logging Support

Spring Boot supports **structured logging natively**, without needing third-party appenders, for **three popular formats**:

| Format | Description | Commonly Used With |
| --- | --- | --- |
| **ECS (Elastic Common Schema)** | A standard JSON format for Elastic Stack (Elasticsearch, Kibana) | Elastic Stack |
| **GELF (Graylog Extended Log Format)** | JSON format for Graylog and compatible tools | Graylog |
| **Logstash** | JSON format optimized for Logstash ingestion | Logstash, ELK stack |

---

## Enabling Structured Logging

You can enable structured logging directly in your `application.properties` (or YAML):

### For Console Output

```
logging.structured.format.console=ecs
```

### For File Output

```
logging.structured.format.file=logstash
```

Spring Boot will automatically:

- Detect the specified format (`ecs`, `gelf`, or `logstash`)
- Configure internal encoders to output logs in that structured format
- Replace standard text-based patterns with JSON-based structured entries

---

## How It Works Internally

Spring Boot transfers those settings into **System properties**:

| Spring Property | System Property | Purpose |
| --- | --- | --- |
| `logging.structured.format.console` | `CONSOLE_LOG_STRUCTURED_FORMAT` | Structured format for console |
| `logging.structured.format.file` | `FILE_LOG_STRUCTURED_FORMAT` | Structured format for file |

These system properties can then be **read by your logging configuration**, especially if you use a **custom Logback or Log4j2 configuration**.

---

## Using Structured Logging with Custom Configurations

If you use your own custom configuration file (like `logback-spring.xml`), you must explicitly configure your **encoder** to use structured output.

For **Logback**, you replace your encoder definition with a `StructuredLogEncoder`.

**Example: Logback Console Appender (structured)**

```xml
<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <!-- Structured JSON output -->
    <encoder class="org.springframework.boot.logging.logback.StructuredLogEncoder">
        <format>${CONSOLE_LOG_STRUCTURED_FORMAT}</format>
        <charset>${CONSOLE_LOG_CHARSET}</charset>
    </encoder>
</appender>
```

**Example: Logback File Appender (structured)**

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_FILE}</file>
    <encoder class="org.springframework.boot.logging.logback.StructuredLogEncoder">
        <format>${FILE_LOG_STRUCTURED_FORMAT}</format>
        <charset>${FILE_LOG_CHARSET}</charset>
    </encoder>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN}</fileNamePattern>
    </rollingPolicy>
</appender>
```

---

**What Happens When You Enable It**

Let’s say you set:

```
logging.structured.format.console=ecs
```

Spring Boot will:

1. Set the `CONSOLE_LOG_STRUCTURED_FORMAT` system property to `"ecs"`.
2. During startup, detect this value.
3. Replace normal text-based console logging with **Elastic Common Schema JSON** output.

Resulting log output (simplified example):

```json
{
  "@timestamp": "2025-10-23T12:45:01.501Z",
  "log.level": "INFO",
  "message": "Application started",
  "service.name": "demo-service",
  "process.thread.name": "main",
  "log.logger": "org.springframework.boot.StartupInfoLogger"
}
```

---

## Default Structured Logback Configurations

Spring Boot includes **default templates** for both **structured console** and **structured file appenders** inside `spring-boot.jar`:

- `org/springframework/boot/logging/logback/defaults.xml`
- `org/springframework/boot/logging/logback/structured-console-appender.xml`
- `org/springframework/boot/logging/logback/structured-file-appender.xml`

You can refer to them if you want to customize or extend the configuration.

They show how the `StructuredLogEncoder` is wired internally.

---

## Advantages of Structured Logging

1. **Machine-readable** – Easy to parse, filter, and analyze.
2. **Better integration** – Works seamlessly with ELK, Graylog, and other log analysis platforms.
3. **Consistency** – Same structure across console and file logs.
4. **Searchable fields** – You can search logs by timestamp, level, user ID, service name, etc.
5. **JSON format** – Each log entry is a JSON object, suitable for ingestion by log pipelines.

---

## When to Use Structured Logging

Use structured logging if:

- Your logs are sent to tools like **Elasticsearch**, **Graylog**, or **Logstash**.
- You need to **analyze logs programmatically**.
- You are building **microservices** and want consistent log formatting across services.

Avoid it (or use plain text) when:

- You only view logs directly in the console for local debugging.
- You don’t plan to use a log aggregation system.

---

## Example Complete Configuration

**application.properties**

```
logging.file.name=logs/app.json
logging.structured.format.console=ecs
logging.structured.format.file=logstash
logging.level.root=INFO
```

**logback-spring.xml**

```xml
<configuration>
    <include resource="org/springframework/boot/logging/logback/structured-console-appender.xml"/>
    <include resource="org/springframework/boot/logging/logback/structured-file-appender.xml"/>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

This setup outputs:

- **Console logs** → in ECS (Elastic Common Schema) JSON format.
- **File logs** → in Logstash JSON format (for ingestion by ELK stack).

## **Elastic Common Schema**

### What is Elastic Common Schema (ECS)?

**Elastic Common Schema (ECS)** is a **standardized JSON-based logging format** defined by **Elastic (the company behind Elasticsearch, Logstash, and Kibana)**.

It defines a consistent structure for logs, metrics, traces, and other events — making it easy to search, correlate, and analyze data in the **Elastic Stack (ELK: Elasticsearch, Logstash, Kibana)**.

ECS ensures that all logs, no matter which service they come from, have **the same field names** and **consistent structure**.

---

### Why ECS Format is Useful

When logs follow ECS, tools like **Kibana** can:

- Automatically detect common fields (`@timestamp`, `log.level`, `service.name`, etc.)
- Display dashboards and filters consistently across all services
- Easily correlate application logs with metrics or traces

Without ECS, you would have to manually parse or map fields to visualize data properly.

---

### Enabling ECS Logging in Spring Boot

Spring Boot provides **first-class support** for ECS structured logging.

You can enable it using the following properties in your configuration file.

**Example using `application.properties`**

```
logging.structured.format.console=ecs
logging.structured.format.file=ecs
```

**Example using YAML**

```yaml
logging:
  structured:
    format:
      console: ecs
      file: ecs
```

This tells Spring Boot to use the **Elastic Common Schema JSON format** for:

- Console output (`stdout`)
- File output (if file logging is enabled)

---

### Example ECS Log Output

When ECS logging is enabled, a typical log line looks like this:

```json
{
  "@timestamp": "2024-01-01T10:15:00.067462556Z",
  "log": {
    "level": "INFO",
    "logger": "org.example.Application"
  },
  "process": {
    "pid": 39599,
    "thread": {
      "name": "main"
    }
  },
  "service": {
    "name": "simple"
  },
  "message": "No active profile set, falling back to 1 default profile: \"default\"",
  "ecs": {
    "version": "8.11"
  }
}
```

**Explanation of Each Field**

| Field | Description |
| --- | --- |
| `@timestamp` | Exact time when the log was written (ISO 8601 format, UTC). |
| `log.level` | Log level (e.g., INFO, WARN, ERROR). |
| `log.logger` | Fully qualified logger name (e.g., `org.example.Application`). |
| `process.pid` | Operating system process ID. |
| `process.thread.name` | Thread name that produced the log. |
| `service.name` | Application/service name (defaults to `spring.application.name`). |
| `message` | The actual log message text. |
| `ecs.version` | The ECS schema version (e.g., `8.11`). |

---

### MDC and Key-Value Logging in ECS

ECS format automatically includes **MDC (Mapped Diagnostic Context)** entries in the log JSON.

### What this means:

If you use MDC in your code, such as:

```java
import org.slf4j.MDC;

MDC.put("userId", "12345");
logger.info("User logged in");
```

The resulting ECS log will include that MDC value:

```json
{
  "@timestamp": "...",
  "log": { "level": "INFO" },
  "message": "User logged in",
  "userId": "12345"
}
```

This is extremely powerful for tracing and debugging, since every log line can automatically carry context like:

- User ID
- Request ID
- Session ID
- Transaction ID

**Using SLF4J Fluent Logging API**

SLF4J 2.x provides a **fluent logging API** that lets you add structured key-value pairs directly:

```java
logger.atInfo()
      .addKeyValue("userId", 12345)
      .addKeyValue("action", "login")
      .log("User login successful");
```

Resulting ECS log:

```json
{
  "@timestamp": "...",
  "log": { "level": "INFO" },
  "message": "User login successful",
  "userId": 12345,
  "action": "login"
}
```

---

### Customizing ECS `service` Fields

ECS defines a `service` object to describe the running service.

Spring Boot lets you customize these values using the following properties.

**In `application.properties`**

```
logging.structured.ecs.service.name=MyService
logging.structured.ecs.service.version=1
logging.structured.ecs.service.environment=Production
logging.structured.ecs.service.node-name=Primary
```

**In YAML**

```yaml
logging:
  structured:
    ecs:
      service:
        name: MyService
        version: 1
        environment: Production
        node-name: Primary
```

**Explanation of Each Property**

| Property | ECS Field | Description |
| --- | --- | --- |
| `logging.structured.ecs.service.name` | `service.name` | Logical name of your service (e.g., "OrderService"). Defaults to `spring.application.name`. |
| `logging.structured.ecs.service.version` | `service.version` | Application version (e.g., `1.0.0`). Defaults to `spring.application.version`. |
| `logging.structured.ecs.service.environment` | `service.environment` | Environment name like `Production`, `Staging`, or `Dev`. |
| `logging.structured.ecs.service.node-name` | `service.node.name` | Identifier for the node (e.g., "Primary", "Node-1"). Useful in clustered deployments. |

**Example Customized ECS Log Output**

```json
{
  "@timestamp": "2024-01-01T10:15:00.067Z",
  "log": {
    "level": "INFO",
    "logger": "org.example.App"
  },
  "service": {
    "name": "MyService",
    "version": "1",
    "environment": "Production",
    "node": {
      "name": "Primary"
    }
  },
  "message": "Application started successfully",
  "ecs": {
    "version": "8.11"
  }
}
```

---

### Default Values and Behavior

- **`service.name`** defaults to `spring.application.name` if not explicitly set.
    
    Example:
    
    ```
    spring.application.name=OrderService
    ```
    
    This automatically sets:
    
    ```json
    "service": { "name": "OrderService" }
    ```
    
- **`service.version`** defaults to `spring.application.version` if available (for example, if defined in your build or manifest).

---

### When to Use ECS Logging

Use ECS format when:

- You are sending logs to **Elasticsearch** or **Logstash**.
- You want **structured, machine-readable logs**.
- You are managing **multiple Spring Boot services** and want consistent field naming and formats across all logs.

Avoid ECS logging (use plain text instead) if:

- You only view logs in local development or simple text files.
- You do not use ELK/Elastic tools.

## **Graylog Extended Log Format (GELF)**

### What is GELF?

**GELF (Graylog Extended Log Format)** is a **JSON-based structured logging format** used mainly by the **Graylog log analytics platform**.

It is designed to make logs more structured, machine-readable, and easy to analyze.

Instead of plain text logs, GELF writes log entries as **JSON objects** — each containing fields like timestamp, log level, message, and other metadata.

---

### Enabling GELF in Spring Boot

Spring Boot provides built-in support for GELF structured logging.

You can enable it using application properties:

**For console logs:**

```
logging.structured.format.console=gelf
```

**For file logs:**

```
logging.structured.format.file=gelf
```

This tells Spring Boot to output logs in GELF JSON format either to the console or to a log file.

---

### Example of a GELF log line

```json
{
  "version": "1.1",
  "short_message": "No active profile set, falling back to 1 default profile: \"default\"",
  "timestamp": 1725958035.857,
  "level": 6,
  "_level_name": "INFO",
  "_process_pid": 47649,
  "_process_thread_name": "main",
  "_log_logger": "org.example.Application"
}
```

Let’s explain each field:

| Field | Description |
| --- | --- |
| **version** | GELF format version (always `"1.1"`). |
| **short_message** | The actual log message. |
| **timestamp** | The UNIX timestamp (in seconds with milliseconds) when the log was created. |
| **level** | Syslog severity level number (`6` means INFO, `3` would mean ERROR, etc.). |
| **_level_name** | Readable log level name (`INFO`, `WARN`, `ERROR`, etc.). |
| **_process_pid** | The process ID of the running application. |
| **_process_thread_name** | The thread name where the log originated. |
| **_log_logger** | The Java class or logger name that produced the log. |

Fields starting with an underscore (`_`) are **additional fields** beyond the standard GELF specification.

---

### MDC and Key-Value Pairs

Spring Boot’s GELF integration also includes **MDC (Mapped Diagnostic Context)** values in the JSON automatically.

That means, if your code adds context-specific information (like user IDs or transaction IDs) to the MDC, it will automatically appear in the JSON logs.

Example (in code):

```java
import org.slf4j.MDC;

MDC.put("userId", "12345");
logger.info("User logged in");
MDC.clear();
```

Resulting JSON:

```json
{
  "version": "1.1",
  "short_message": "User logged in",
  "timestamp": 1725958035.857,
  "level": 6,
  "_level_name": "INFO",
  "_userId": "12345"
}
```

You can also add structured data dynamically using the **SLF4J fluent logging API**:

```java
logger.atInfo().addKeyValue("orderId", 101).log("Order processed");
```

---

### Customizing GELF Fields

You can customize specific GELF fields using the following properties:

**Example:**

```
logging.structured.gelf.host=MyService
logging.structured.gelf.service.version=1
```

**Meaning:**

- **logging.structured.gelf.host** → Sets the `host` field in GELF output.
    
    If not specified, it defaults to the value of `spring.application.name`.
    
- **logging.structured.gelf.service.version** → Sets the `version` field of your service.
    
    If not specified, it defaults to `spring.application.version`.
    

## ECS vs GELF

### Purpose and Ecosystem

| Aspect | **ECS (Elastic Common Schema)** | **GELF (Graylog Extended Log Format)** |
| --- | --- | --- |
| **Used by** | Elasticsearch / Elastic Stack (Elastic, Kibana, Beats, Logstash) | Graylog (and tools that support GELF) |
| **Purpose** | Standard schema for storing and analyzing logs in the **Elastic Stack**. | Simplified structured logging for sending logs to **Graylog** via GELF input. |
| **Design Goal** | Rich, hierarchical structure for advanced searching, filtering, and correlation. | Lightweight, flat JSON structure for efficient log ingestion. |

---

### Structure and Field Naming

| Aspect | **ECS** | **GELF** |
| --- | --- | --- |
| **Field Style** | Hierarchical (nested fields like `log.level`, `process.pid`, `service.name`) | Flat structure (simple key-value pairs like `_process_pid`, `_level_name`) |
| **Example Log** | `json { "@timestamp": "2024-01-01T10:15:00Z", "log": { "level": "INFO", "logger": "org.example.App" }, "process": { "pid": 39599, "thread": { "name": "main" } }, "service": { "name": "simple" }, "message": "No active profile set", "ecs": { "version": "8.11" } }` | `json { "version": "1.1", "short_message": "No active profile set", "timestamp": 1725958035.857, "level": 6, "_level_name": "INFO", "_process_pid": 47649, "_process_thread_name": "main", "_log_logger": "org.example.App" }` |
| **Timestamp Field** | `@timestamp` (ISO 8601 format) | `timestamp` (UNIX time in seconds) |
| **Message Field** | `message` | `short_message` |
| **Version Field** | `ecs.version` | `version` (always `"1.1"`) |
| **Log Level** | Nested as `log.level` | Stored as numeric `level` and readable `_level_name` |

---

### Customization

| Aspect | **ECS** | **GELF** |
| --- | --- | --- |
| **Service Metadata** | Configurable via `logging.structured.ecs.service.*` (name, version, environment, node-name) | Configurable via `logging.structured.gelf.*` (host, service.version) |
| **Defaults** | Uses `spring.application.name` and `spring.application.version` | Same defaults for `host` and `service.version` |

---

### Compatibility and Use Cases

| Aspect | **ECS** | **GELF** |
| --- | --- | --- |
| **Best suited for** | Applications sending logs to the **Elastic Stack** (Elasticsearch, Kibana, Logstash). | Applications sending logs to a **Graylog server**. |
| **Parsing** | Elasticsearch has built-in ECS parsing and mapping templates. | Graylog natively supports GELF over TCP/UDP/HTTP. |
| **Integration Complexity** | Slightly more complex (more nested JSON). | Simpler, smaller JSON structure. |

## **Logstash JSON format**

### What is Logstash JSON Format?

The **Logstash JSON format** is a **flat, JSON-based log structure** designed to be easily parsed by **Logstash**, which collects, processes, and forwards logs to **Elasticsearch** for visualization in **Kibana**.

It is simpler than ECS (Elastic Common Schema) but still standardized enough for efficient ingestion and querying inside the Elastic Stack.

---

### Enabling Logstash JSON Format in Spring Boot

You can enable it either for **console logs** or **file logs** by setting:

**In `application.properties`:**

```
logging.structured.format.console=logstash
logging.structured.format.file=logstash
```

**In `application.yaml`:**

```yaml
logging:
  structured:
    format:
      console: logstash
      file: logstash
```

Once enabled, all log entries will be written in **Logstash-compatible JSON format**.

---

### Example Log Entry

```json
{
  "@timestamp": "2024-01-01T10:15:00.111037681+02:00",
  "@version": "1",
  "message": "No active profile set, falling back to 1 default profile: \"default\"",
  "logger_name": "org.example.Application",
  "thread_name": "main",
  "level": "INFO",
  "level_value": 20000
}
```

**Explanation of Fields:**

| Field | Description |
| --- | --- |
| **@timestamp** | The time when the log was created. This is in ISO 8601 format and includes timezone information. It allows Logstash and Elasticsearch to order and analyze logs accurately. |
| **@version** | Indicates the Logstash event format version (commonly `"1"`). It helps with schema versioning. |
| **message** | The main log message — what you pass in the logger (e.g., `logger.info("Server started")`). |
| **logger_name** | The fully qualified class name (FQCN) of the logger, e.g., `org.example.Application`. |
| **thread_name** | The name of the thread where the log was produced. Useful for debugging concurrent applications. |
| **level** | The log level as a string (`INFO`, `ERROR`, `DEBUG`, etc.). |
| **level_value** | The numeric equivalent of the log level (e.g., `INFO=20000`, `DEBUG=10000`, etc.). Used by some tools for filtering or sorting. |

---

### MDC Integration (Mapped Diagnostic Context)

Just like ECS and GELF, this format also includes **all key-value pairs** from the **MDC (Mapped Diagnostic Context)**.

For example, if your application adds contextual data like:

```java
MDC.put("userId", "42");
MDC.put("sessionId", "abc123");
logger.info("User logged in");
```

Then your log output will automatically include these keys:

```json
{
  "@timestamp": "2024-01-01T10:15:00.111037681+02:00",
  "@version": "1",
  "message": "User logged in",
  "logger_name": "org.example.Application",
  "thread_name": "main",
  "level": "INFO",
  "level_value": 20000,
  "userId": "42",
  "sessionId": "abc123"
}
```

This makes it easy to filter logs by user, session, or other contextual attributes in Kibana.

---

### Support for Markers (Tags)

If you use **SLF4J markers** in your logging calls (for example, to tag specific log entries), these markers are automatically included as **tags** in the JSON output.

Example:

```java
Marker audit = MarkerFactory.getMarker("AUDIT");
logger.info(audit, "User performed a sensitive action");
```

Log output:

```json
{
  "@timestamp": "2024-01-01T10:15:00.111037681+02:00",
  "@version": "1",
  "message": "User performed a sensitive action",
  "logger_name": "org.example.Application",
  "thread_name": "main",
  "level": "INFO",
  "level_value": 20000,
  "tags": ["AUDIT"]
}
```

This helps categorize or filter specific log types (like security, audit, performance, etc.) when analyzing logs in Kibana or other log dashboards.

### Difference from ECS and GELF

| Aspect | **ECS** | **GELF** | **Logstash JSON** |
| --- | --- | --- | --- |
| **Used By** | Elastic Stack (ECS Schema) | Graylog | Logstash + Elastic Stack |
| **Structure** | Deeply nested (hierarchical) | Flat (with `_` prefixes) | Flat and simple |
| **Version Field** | `ecs.version` | `version: "1.1"` | `@version: "1"` |
| **Timestamp** | `@timestamp` (ISO 8601) | `timestamp` (UNIX seconds) | `@timestamp` (ISO 8601) |
| **Ideal For** | Advanced Elastic analysis and correlation | Graylog ingestion | Quick Elastic ingestion through Logstash |

### When to Use Logstash Format

Use the **Logstash JSON format** when:

- You are **sending logs to Logstash**, either directly or via filebeat.
- You want **simple JSON logs** without the full ECS schema.
- You want **fast ingestion and minimal setup** in an Elastic Stack environment.

If you are using **Elasticsearch and Kibana** but not defining a strict ECS mapping, the Logstash format is often the easiest to start with.

## **Customizing Structured Logging JSON**

### Why Customization is Needed

By default, Spring Boot provides a **standard structure** for JSON logs with common fields such as:

```json
{
  "@timestamp": "2024-01-01T10:15:00Z",
  "log": { "level": "INFO", "logger": "org.example.App" },
  "process": { "pid": 12345, "thread": { "name": "main" } },
  "service": { "name": "demo" },
  "message": "Application started"
}
```

While this works well in most cases, your **log ingestion or monitoring system** (e.g., Elastic Stack, Graylog, Datadog, Splunk, etc.) might expect:

- Different **field names** (e.g., `procid` instead of `process.id`)
- Only specific **fields** (e.g., omit log level)
- Additional **custom fields** (e.g., company name or environment tag)

Spring Boot allows you to **adjust the JSON log output** without modifying your Java code.

---

### The Main Customization Properties

There are three core configuration properties for structured JSON modification:

**a. `logging.structured.json.include` and `logging.structured.json.exclude`**

- **Purpose:** Control which fields should appear or be hidden in the final JSON output.
- **Behavior:**
    - `include` → only include these paths.
    - `exclude` → exclude these paths.

**Example:**

```
logging.structured.json.exclude=log.level
```

**Effect:** Removes the `log.level` field from every JSON log line.

So, instead of this:

```json
{
  "log": { "level": "INFO", "logger": "org.example.App" },
  "message": "App started"
}
```

You’d get:

```json
{
  "log": { "logger": "org.example.App" },
  "message": "App started"
}
```

---

**b. `logging.structured.json.rename`**

- **Purpose:** Rename specific JSON fields (or paths) to new names that better suit your system.

**Example:**

```
logging.structured.json.rename.process.id=procid
```

**Effect:**

Changes the field `process.id` → `procid`.

So instead of this:

```json
{
  "process": { "id": 39599 }
}
```

You’d get:

```json
{
  "procid": 39599
}
```

---

**c. `logging.structured.json.add`**

- **Purpose:** Add **new fields** with fixed values to every log entry.

**Example:**

```
logging.structured.json.add.corpname=mycorp
```

**Effect:**

Adds a `"corpname": "mycorp"` field to all log lines.

So you’d get:

```json
{
  "@timestamp": "2024-01-01T10:15:00Z",
  "message": "Application started",
  "corpname": "mycorp"
}
```

This is useful for:

- Identifying logs from a specific company or team.
- Adding environment or deployment identifiers (e.g., `"environment": "production"`).

---

### Combined Example

If you apply all three together:

```
logging.structured.json.exclude=log.level
logging.structured.json.rename.process.id=procid
logging.structured.json.add.corpname=mycorp
```

Then a typical JSON log would transform as follows:

**Before:**

```json
{
  "@timestamp": "2024-01-01T10:15:00Z",
  "log": { "level": "INFO", "logger": "org.example.App" },
  "process": { "id": 39599 },
  "message": "Server started"
}
```

**After:**

```json
{
  "@timestamp": "2024-01-01T10:15:00Z",
  "procid": 39599,
  "message": "Server started",
  "corpname": "mycorp"
}
```

---

### For Advanced Customization: `StructuredLoggingJsonMembersCustomizer`

If property-based customization is not enough (for example, if you need **conditional logic** or **dynamic values**), you can use a **Java-based customization interface**:

**Interface: `StructuredLoggingJsonMembersCustomizer`**

You can create your own implementation like:

```java
import org.springframework.boot.logging.structured.StructuredLoggingJsonMembersCustomizer;
import java.util.Map;

public class MyCustomJsonCustomizer implements StructuredLoggingJsonMembersCustomizer {
    @Override
    public void customize(Map<String, Object> members) {
        // Remove a field dynamically
        members.remove("process");

        // Add a dynamic field (e.g., current hostname)
        members.put("hostname", System.getenv("HOSTNAME"));

        // Modify an existing field
        members.put("environment", "production");
    }
}
```

**Registering the customizer**

There are **two ways** to make Spring Boot recognize your customizer:

1. **Via `application.properties`:**
    
    ```
    logging.structured.json.customizer=com.example.MyCustomJsonCustomizer
    ```
    
2. **Via `META-INF/spring.factories`:**
    
    ```
    org.springframework.boot.logging.structured.StructuredLoggingJsonMembersCustomizer=\
    com.example.MyCustomJsonCustomizer
    ```
    

This allows you to apply programmatic logic to modify or enrich log entries before they’re written.

## **Customizing Structured Logging Stack Traces**

By default, when a log entry includes an exception, the **complete stack trace** (including all nested exceptions, causes, and suppressed exceptions) is written to the JSON log output.

However, since these full stack traces can be large and costly to store or process (especially in cloud logging systems like ELK, Datadog, or Graylog), Spring Boot provides configuration options to **control or limit the stack trace information**.

---

### **Purpose**

To customize how exception stack traces appear in structured JSON logs — including:

- Order of exception frames
- Maximum output size
- Inclusion of common frames
- Hashes for deduplication

---

### **Key Properties**

| Property | Description |
| --- | --- |
| **`logging.structured.json.stacktrace.root`** | Defines the order of the root cause in the stack trace.• `last` (default): Root cause printed last (same as standard Java behavior).• `first`: Root cause printed first. |
| **`logging.structured.json.stacktrace.max-length`** | Limits the total length (in characters) of the printed stack trace. Useful to prevent very large logs. |
| **`logging.structured.json.stacktrace.max-throwable-depth`** | Limits how many stack frames are printed per throwable (includes suppressed and common frames). |
| **`logging.structured.json.stacktrace.include-common-frames`** | Controls whether "common frames" (repeated frames in nested exceptions) are printed or omitted. |
| **`logging.structured.json.stacktrace.include-hashes`** | Includes a hash (a unique fingerprint) of the stack trace. Helps with deduplication in log analysis tools. |

---

### **Example**

```
logging.structured.json.stacktrace.root=first
logging.structured.json.stacktrace.max-length=1024
logging.structured.json.stacktrace.include-common-frames=true
logging.structured.json.stacktrace.include-hashes=true
```

**Explanation of the above example:**

- The **root cause** of the exception will appear **first** in the JSON stack trace.
- The **maximum total size** of the stack trace will be **1024 characters**.
- **Common frames** (shared between exceptions in the chain) will be included.
- A **hash** of the stack trace will also be included for easy identification and grouping of similar errors.

---

### **Advanced Customization**

If you need more control than what these properties provide, you can define your own **custom stack trace printer**.

### **Property:**

```
logging.structured.json.stacktrace.printer
```

You can set it to:

- A **custom implementation class** of `StackTracePrinter`.
- Or to `logging-system` — this tells Spring Boot to use the regular logging system’s stack trace format instead of structured JSON.

**Custom Implementation Option**

If you implement your own `StackTracePrinter`, it can optionally take a constructor argument of type `StandardStackTracePrinter`, letting you extend or modify its behavior rather than replacing it completely.

## **Supporting Other Structured Logging Formats**

By default, Spring Boot provides built-in structured logging formats that output logs as JSON suitable for log aggregation systems. However, you might want to output logs in a completely different way — for example, in a simplified key-value format, XML, or even a custom JSON layout specific to your organization's requirements.

To support this flexibility, Spring Boot allows you to define your own **custom structured log formatter** by implementing the `StructuredLogFormatter` interface.

---

### **What is StructuredLogFormatter?**

`StructuredLogFormatter<T>` is an interface provided by Spring Boot for defining **how each log event is formatted** into a string before being written to the console or a file.

- The **generic type parameter** `<T>` represents the type of log event.
- This depends on the underlying logging system:
    - If you are using **Logback** (default in Spring Boot): use `ILoggingEvent`.
    - If you are using **Log4j2**: use `LogEvent`.

In short:

| Logging System | Type to Use |
| --- | --- |
| Logback | `ILoggingEvent` |
| Log4j2 | `LogEvent` |

---

### **Example Implementation**

```java
import ch.qos.logback.classic.spi.ILoggingEvent;
import org.springframework.boot.logging.structured.StructuredLogFormatter;

class MyCustomFormat implements StructuredLogFormatter<ILoggingEvent> {

    @Override
    public String format(ILoggingEvent event) {
        return "time=" + event.getInstant() +
               " level=" + event.getLevel() +
               " message=" + event.getMessage() + "\n";
    }
}
```

**Explanation:**

- The `format()` method is called for each log event.
- You have full control over how the event is converted into a string.
- In this example, the log line is formatted as simple key-value pairs:
    
    ```
    time=2025-10-23T05:30:00 level=INFO message=Application started
    ```
    
- The format doesn’t have to be JSON — it can be *any* structure you prefer.

---

### **Enabling Your Custom Format**

Once you create the class, you need to tell Spring Boot to use it.

You can do this by setting the following property:

### **For console output:**

```
logging.structured.format.console=com.example.MyCustomFormat
```

### **For file output:**

```
logging.structured.format.file=com.example.MyCustomFormat
```

Here, `com.example.MyCustomFormat` should be the **fully qualified class name** of your implementation.

---

### **Dependency Injection Support**

Your custom formatter can also have a **constructor with parameters**, and Spring Boot will automatically **inject dependencies** into it.

For example, you might want to inject configuration properties or environment variables to influence formatting.

For details on which parameters can be injected, you can refer to the **JavaDoc of `StructuredLogFormatter`**, which explains all supported constructor argument types.

# **Logback Extensions**

### **Logback in Spring Boot**

**Logback** is the **default logging framework** used by Spring Boot.

Normally, you can configure Logback using a file named `logback.xml`.

However, Spring Boot adds **extra Logback features** (called *extensions*) that make configuration more dynamic and environment-aware — for example:

- Using Spring properties (`springProperty`) in your logging configuration.
- Activating specific log configurations only for certain profiles (`springProfile`).
- Accessing Spring environment variables or configuration files directly inside the Logback config.

These extensions make Logback “Spring-aware.”

---

### **Why You Need `logback-spring.xml` Instead of `logback.xml`**

The **standard `logback.xml`** is loaded **before** the Spring context starts.

At that early stage:

- Spring Boot’s environment and property sources (like `application.properties` or `application.yml`) are **not yet available**.
- Therefore, Logback cannot access any Spring-specific features such as `springProperty` or `springProfile`.

To solve this, Spring Boot provides **`logback-spring.xml`**, a special configuration file that is loaded **after the Spring environment is ready**.

This means:

- You can safely use Spring properties (like `${spring.application.name}`).
- You can define profile-specific logging configurations.
- You can use the advanced Spring extensions that are unavailable in standard Logback.

---

### **Using the Extensions**

Some of the commonly used **Spring Boot Logback extensions** include:

| Extension | Purpose |
| --- | --- |
| `<springProperty>` | Allows you to use a Spring environment property in your Logback configuration. |
| `<springProfile>` | Enables profile-specific log configurations (similar to `@Profile` in Spring). |

Example:

```xml
<configuration>
    <!-- Use a property from Spring Boot's environment -->
    <springProperty scope="context" name="logFilePath" source="logging.file.path"/>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>${logFilePath}/app.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Profile-specific logging -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

This configuration:

- Reads a property (`logging.file.path`) from Spring’s environment.
- Changes the log level based on the active Spring profile.

---

### **Why You Can’t Use Logback Configuration Scanning**

Logback supports *configuration scanning*, which automatically reloads the configuration file if it changes at runtime.

However, **Spring Boot disables this feature** when you use `logback-spring.xml` because:

- The configuration includes Spring extensions that require the Spring context.
- Reloading the configuration at runtime would happen *outside* of Spring’s control.
- This can cause Logback to throw errors like:
    
    ```
    ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty]
    ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile]
    ```
    

These errors mean Logback tried to reload the file directly, but **did not understand Spring’s custom tags** (`springProperty`, `springProfile`) because they are not part of standard Logback.

---

### **If You Need to Use Extensions**

To use Spring Boot’s extensions correctly, you must:

- Use **`logback-spring.xml`** instead of `logback.xml`.
- Or set the property in your configuration:
    
    ```
    logging.config=classpath:logback-spring.xml
    ```
    
- Avoid using Logback’s *auto-scan* feature.

## **Profile-specific Configuration**

### **Purpose of `<springProfile>`**

- Spring Boot supports **different profiles** (e.g., `dev`, `staging`, `prod`) to manage environment-specific configurations.
- With `<springProfile>`, you can **enable or disable certain parts of the Logback configuration** depending on the currently active profile.
- This is useful when you want:
    - Different log levels per environment (`DEBUG` in `dev`, `INFO` in `prod`).
    - Different appenders or log files for different environments.
    - Conditional structured logging or external log integrations.

---

### **Where It Can Be Used**

- `<springProfile>` can appear **anywhere inside the `<configuration>` element** of your `logback-spring.xml`.
- You can nest appenders, loggers, or root logging configurations inside it.

---

### **Syntax and Examples**

**a. Single Profile**

```xml
<springProfile name="staging">
    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>
</springProfile>
```

- The configuration inside this block is **applied only when the `staging` profile is active**.
- If the `staging` profile is inactive, this configuration is ignored.

---

**b. Multiple Profiles with OR (`|`)**

```xml
<springProfile name="dev | staging">
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>
```

- The configuration applies if **either `dev` or `staging`** is active.
- Useful when multiple profiles share common logging requirements.

---

**c. NOT Profile (`!`)**

```xml
<springProfile name="!production">
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>
```

- This block is applied when **`production` is NOT active**.
- You can use it to **enable verbose logging in all non-production environments**.

---

**d. Complex Profile Expressions**

- You can combine **AND (`&`)** and **OR (`|`)** operators with parentheses.
- Example:

```xml
<springProfile name="production & (eu-central | eu-west)">
    <!-- configuration for production servers in either EU Central or EU West -->
</springProfile>
```

- Meaning: This configuration is applied **only if the `production` profile is active AND the active region is `eu-central` or `eu-west`**.

---

### **Benefits**

- **Environment-specific logging:** Allows you to tailor logging per profile without duplicating configuration.
- **Dynamic configuration:** Profiles are resolved at runtime based on Spring Boot’s `spring.profiles.active` property.
- **Centralized control:** All logging configurations remain in one `logback-spring.xml` file, making it easy to maintain.

---

### **How Spring Resolves Profiles**

1. Spring Boot reads the `spring.profiles.active` property from:
    - `application.properties` or `application.yml`
    - Environment variables (`SPRING_PROFILES_ACTIVE`)
    - Command-line arguments (`-spring.profiles.active=dev`)
2. `<springProfile>` blocks are evaluated **at logging initialization**, and only the blocks matching the active profiles are applied.

## **Environment Properties**

### **Purpose of `<springProperty>`**

- Standard Logback has a `<property>` tag that defines fixed values in your logging configuration.
- `<springProperty>` extends this functionality by allowing the value to come **directly from Spring’s Environment**, i.e., from:
    - `application.properties` or `application.yml`
    - Environment variables
    - Command-line arguments
- This makes your logging configuration **flexible and environment-specific** without hardcoding values.

---

### **Key Attributes**

| Attribute | Purpose |
| --- | --- |
| **`name`** | The name of the property as it will be used in the Logback file. Example: `${fluentHost}`. |
| **`source`** | The name of the Spring property in the environment that will supply the value. Example: `myapp.fluentd.host`. Must be in **kebab case** (`my.property-name`). |
| **`defaultValue`** | A fallback value if the property is **not defined** in the Spring Environment. Example: `"localhost"`. |
| **`scope`** | Where the property is available: usually `"context"` for global scope. |

---

### **Example Usage**

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host" defaultValue="localhost"/>

<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

**Explanation:**

1. `<springProperty>` reads the `myapp.fluentd.host` property from Spring’s environment.
2. The value is stored in a property called `fluentHost`.
3. The `${fluentHost}` placeholder is then used in the Logback appender configuration.
4. If the property is not defined in the environment, it defaults to `"localhost"`.

---

### **Notes on Property Naming**

- **Source names must be in kebab-case**, for example: `my.property-name`.
- Spring Boot supports **relaxed binding**, meaning it can map:
    - `my.property-name`
    - `my.propertyName`
    - `MY_PROPERTY_NAME`
        
        to the same property in the Spring Environment.
        
- This makes it flexible to reference properties regardless of naming style in `application.properties`, `application.yml`, or environment variables.

# **Log4j2 Extensions**

### **Log4j2 in Spring Boot**

- **Log4j2** is an alternative logging framework to Logback.
- Spring Boot provides **built-in integration** for Log4j2, including **advanced extensions** to make logging configuration environment-aware and dynamic, similar to the extensions in Logback.

These extensions allow you to:

- Access **Spring properties** inside the Log4j2 configuration.
- Enable **profile-specific logging** (e.g., different logging for `dev` vs `prod` profiles).
- Make logging configuration **dynamic** based on the Spring environment.

---

### **Why Use `log4j2-spring.xml`**

- **Standard `log4j2.xml`** is loaded **before the Spring context is initialized**.
- At that point, Spring properties, profiles, and environment variables are **not yet available**.
- Therefore, if you try to use Spring-specific extensions in `log4j2.xml`, they **won’t work**.

**Solution:**

- Use `log4j2-spring.xml` instead of `log4j2.xml`.
- Or define the property in your configuration:
    
    ```
    logging.config=classpath:log4j2-spring.xml
    ```
    
- `log4j2-spring.xml` is loaded **after the Spring context is available**, so it can use Spring properties and profiles.

---

### **Extensions Supersede Other Spring Boot Log4j Support**

- Spring Boot previously had support via the `log4j-spring-boot` module.
- **If you are using `log4j2-spring.xml`, you should not include the `org.apache.logging.log4j:log4j-spring-boot` module** in your build.
- The Spring Boot extensions in `log4j2-spring.xml` **replace that support** and provide a more integrated solution.

## **Profile-specific Configuration**

### **Purpose of `<SpringProfile>`**

- Spring Boot supports **profiles** (`dev`, `staging`, `prod`, etc.) to manage environment-specific behavior.
- `<SpringProfile>` allows you to **apply specific logging configuration only when certain profiles are active**.
- Useful scenarios:
    - Using `DEBUG` logging in development but `INFO` in production.
    - Writing logs to different appenders or files depending on the environment.
    - Integrating with external log systems only in specific profiles.

---

### **Where it Can Be Used**

- `<SpringProfile>` can appear **anywhere inside the `<Configuration>` element** of `log4j2-spring.xml`.
- You can include appenders, loggers, or root log configuration inside a profile-specific block.

---

### **Syntax and Examples**

**a. Single Profile**

```xml
<SpringProfile name="staging">
    <Root level="INFO">
        <AppenderRef ref="FILE"/>
    </Root>
</SpringProfile>
```

- Applied **only when the `staging` profile is active**.
- Ignored in all other profiles.

---

**b. Multiple Profiles with OR (`|`)**

```xml
<SpringProfile name="dev | staging">
    <Root level="DEBUG">
        <AppenderRef ref="CONSOLE"/>
    </Root>
</SpringProfile>
```

- Applied if **either `dev` or `staging`** profile is active.
- Useful when multiple environments share a logging configuration.

---

**c. NOT Profile (`!`)**

```xml
<SpringProfile name="!production">
    <Root level="DEBUG">
        <AppenderRef ref="CONSOLE"/>
    </Root>
</SpringProfile>
```

- Applied when **`production` is NOT active**.
- Useful for enabling verbose logging in non-production environments.

---

**d. Complex Profile Expressions**

```xml
<SpringProfile name="production & (eu-central | eu-west)">
    <!-- applied only if 'production' AND ('eu-central' OR 'eu-west') are active -->
</SpringProfile>
```

- Supports **AND (`&`)**, **OR (`|`)**, and parentheses for grouping.
- Allows fine-grained control over which profiles apply certain logging rules.

---

### **How Spring Resolves Profiles**

1. Spring Boot reads the active profiles from:
    - `spring.profiles.active` in `application.properties` or `application.yml`
    - Environment variables (`SPRING_PROFILES_ACTIVE`)
    - Command-line arguments (`-spring.profiles.active=dev`)
2. `<SpringProfile>` blocks are evaluated at logging initialization.
3. Only blocks matching the **active profile expression** are applied.

## **Environment Properties Lookup**

### **Purpose**

- Often, you want your logging configuration to depend on **application-specific properties**, such as:
    - Application name
    - Host, port, or environment
    - Any custom property from `application.properties` or `application.yml`
- Spring Boot provides a **`spring:` lookup prefix** that allows Log4j2 to reference properties from the **Spring Environment**.

---

### **Syntax**

```xml
<Properties>
    <Property name="applicationName">${spring:spring.application.name}</Property>
    <Property name="applicationGroup">${spring:spring.application.group}</Property>
</Properties>
```

**Explanation:**

- `<Properties>`: Defines properties that can be used elsewhere in the Log4j2 configuration (e.g., in appenders or loggers).
- `<Property>`: Declares a single property.
    - `name="applicationName"` → The property name you will reference as `${applicationName}` elsewhere in the XML.
    - `${spring:spring.application.name}` → Uses the **spring lookup** to get the value of `spring.application.name` from the Spring Environment.
    - `${spring:spring.application.group}` → Reads `spring.application.group` in the same way.
- The **lookup key** (`spring.application.name`) must be in **kebab-case** when referring to the Spring property.

---

### **How It Works**

1. When `log4j2-spring.xml` is loaded, Spring Boot makes the **Spring Environment available** to Log4j2.
2. The `spring:` lookup accesses the specified property from the Spring Environment.
3. If the property exists in `application.properties`, `application.yml`, environment variables, or command-line arguments, its value is used.
4. These properties can then be referenced in appenders, loggers, or anywhere else in the configuration:

```xml
<Appenders>
    <Console name="Console" target="SYSTEM_OUT">
        <PatternLayout pattern="%d [%t] %level %logger{36} - ${applicationName} - ${applicationGroup} - %msg%n"/>
    </Console>
</Appenders>
```

- This outputs the Spring application name and group as part of the log message.

---

### **Notes on Property Names**

- Use **kebab-case** for Spring property keys in the lookup, e.g.:
    - `my.property-name` → `${spring:my.property-name}`
- Spring Boot supports **relaxed binding**, so equivalent keys like `my.propertyName` or `MY_PROPERTY_NAME` will also work.

## **Log4j2 System Properties**

### **What Are Log4j2 System Properties?**

- Log4j2 uses **system properties** to configure various aspects of its behavior.
- Example:
    
    ```
    log4j2.skipJansi=true
    ```
    
    This property controls whether the **ConsoleAppender** uses **Jansi** (a library for colored console output on Windows).
    
- System properties can be set:
    - Directly in the JVM (`Dlog4j2.skipJansi=true`)
    - Or in `application.properties`/`application.yml` in Spring Boot

---

### **How Spring Boot Integrates System Properties**

- Spring Boot provides a **Spring Environment**, which contains:
    - Application properties (`application.properties` or `application.yml`)
    - Environment variables
    - Command-line arguments
- If a **system property is not already set** (via JVM or OS environment variable), Spring Boot can **inject its value from the Spring Environment**.

**Example:**

```
log4j2.skipJansi=false
```

- This will configure Log4j2 to **use Jansi on Windows** if the system property wasn’t already defined.

---

### **Order of Property Resolution**

1. **JVM System Properties** (highest priority)
2. **OS Environment Variables**
3. **Spring Environment properties** (`application.properties`, `application.yml`)
    - Only used if the property is **not already defined** in system properties or environment variables.

---

### **Limitations During Early Initialization**

- Some Log4j2 properties are needed **very early**, before Spring Boot has initialized the Spring Environment.
- For these properties:
    - Spring Environment values **cannot be used**
    - Only JVM system properties or environment variables take effect
- Example:
    - Choosing the **default Log4j2 implementation** (`Log4j2 implementation class`) happens **before Spring Environment is available**, so you cannot configure it via `application.properties`.