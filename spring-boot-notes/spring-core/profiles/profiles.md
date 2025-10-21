# Profiles

- [What are Spring Profiles?](#what-are-spring-profiles)
- [Adding Active Profiles](#adding-active-profiles)
- [Profile Groups](#profile-groups)
- [Programmatically Setting Profiles](#programmatically-setting-profiles)

# What are Spring Profiles?

Spring Profiles allow you to **group configurations** and **activate them conditionally** based on the environment — for example, **development**, **testing**, **staging**, or **production**.

You can mark configurations, beans, or even properties to **only load when a specific profile is active**.

This avoids clutter and keeps configurations **clean and environment-specific**.

---

**Example Scenario**

Imagine you are developing a Spring Boot web application with three environments:

- **dev** → for development (using H2 in-memory database)
- **test** → for testing (using PostgreSQL test DB)
- **prod** → for production (using MySQL production DB)

You want Spring Boot to automatically pick the right configuration depending on which environment you run.

---

## Declaring Profiles with `@Profile`

You can use the `@Profile` annotation on:

- `@Configuration` classes
- `@Component` classes
- `@ConfigurationProperties` classes (depending on how they are registered)

**Example**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("dev")
public class DevDatabaseConfig {

    @Bean
    public String dataSource() {
        System.out.println("Loaded Dev configuration: Using H2 Database");
        return "H2 Database";
    }
}
```

```java
@Configuration
@Profile("prod")
public class ProdDatabaseConfig {

    @Bean
    public String dataSource() {
        System.out.println("Loaded Prod configuration: Using MySQL Database");
        return "MySQL Database";
    }
}
```

---

## Activating a Profile

You can activate profiles in multiple ways.

**Option 1: application.properties**

```
spring.profiles.active=dev
```

When you run your application, the **DevDatabaseConfig** class will load.

Output:

```
Loaded Dev configuration: Using H2 Database
```

---

**Option 2: Command-line Argument**

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

This will activate the **prod** profile instead of **dev**.

Output:

```
Loaded Prod configuration: Using MySQL Database
```

---

**Option 3: Environment Variable**

```bash
export SPRING_PROFILES_ACTIVE=dev
```

or on Windows PowerShell:

```powershell
set SPRING_PROFILES_ACTIVE=dev
```

---

## Using Profile-Specific Property Files

Instead of changing one `application.properties` file, you can create **separate files for each profile**:

- `application-dev.properties`
- `application-test.properties`
- `application-prod.properties`

Example:

**application-dev.properties**

```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
```

**application-prod.properties**

```
spring.datasource.url=jdbc:mysql://prod-server/db
spring.datasource.username=prod_user
```

Then, when you activate the profile:

```
spring.profiles.active=prod
```

Spring Boot will automatically load:

- `application.properties`
- `application-prod.properties`

---

## Default Profile

If **no profile is active**, Spring Boot automatically activates a profile called **`default`**.

You can change it using:

```
spring.profiles.default=dev
```

This means if you **don’t specify** any active profile, Spring will treat **`dev`** as the default.

---

## Restriction on Profile Names

By default, profile names can include:

- **Letters** (a-z, A-Z)
- **Numbers**
- **Allowed special characters:** , `_`, `.`, `+`, `@`

They **must start and end with a letter or number**.

For example, valid names:

```
dev
test-db
prod_v2
qa.1
```

Invalid names:

```
_dev    (starts with underscore)
prod!   (contains invalid character)
```

If you want to allow more flexible profile names (e.g., containing `!` or `#`), you can disable this check:

```
spring.profiles.validate=false
```

---

## Profiles with `@ConfigurationProperties`

If you use `@ConfigurationProperties` to bind configuration values from files, and you enable it using `@EnableConfigurationProperties`, then the `@Profile` annotation should be applied on the **@Configuration** class that enables it.

**Example**

```java
@Configuration
@Profile("dev")
@EnableConfigurationProperties(MyAppProperties.class)
public class DevAppConfig {
}
```

But if you’re using **component scanning** to automatically detect your `@ConfigurationProperties` class, you can put the `@Profile` directly on it.

```java
@Profile("prod")
@ConfigurationProperties(prefix = "app")
public class MyAppProperties {
    private String url;
    private String username;
}
```

---

## Invalid Profile Configuration Example

You cannot define `spring.profiles.active` inside a profile-specific file.

For example, this is invalid:

```
spring.profiles.active=prod
#---
spring.config.activate.on-profile=prod
spring.profiles.active=metrics
```

Why?

Because `spring.profiles.active` can **only be used in non-profile-specific** configuration files.

It defines *which profiles are active*, not what gets activated *inside* a profile.

---

## Property Source Precedence (Ordering)

Spring Boot loads configuration in a specific order:

1. Command-line arguments (highest priority)
2. Environment variables
3. `application.properties` or `application.yml`
4. Profile-specific files like `application-dev.properties`
5. Default properties (lowest priority)

So if you define:

```
spring.profiles.active=dev
```

in `application.properties`,

and run:

```bash
java -jar app.jar --spring.profiles.active=prod
```

then **prod** wins because the **command line** has higher precedence.

---

## Complete Example Setup

Let’s combine everything.

**`application.properties`**

```
spring.profiles.active=dev
```

**`application-dev.properties`**

```
app.message=Running in Development Environment
```

**`application-prod.properties`**

```
app.message=Running in Production Environment
```

**Java Code**

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProfileExampleApp implements CommandLineRunner {

    @Value("${app.message}")
    private String message;

    public static void main(String[] args) {
        SpringApplication.run(ProfileExampleApp.class, args);
    }

    @Override
    public void run(String... args) {
        System.out.println(message);
    }
}
```

**Output**

If you run normally:

```
Running in Development Environment
```

If you run with:

```bash
java -jar app.jar --spring.profiles.active=prod
```

Output:

```
Running in Production Environment
```

# **Adding Active Profiles**

## Recap: `spring.profiles.active`

The property **`spring.profiles.active`** tells Spring Boot which **main environment profiles** should be active for the current run.

**Example:**

```
spring.profiles.active=dev
```

→ Activates the **`dev`** profile.

When you run the app:

- Spring loads **application.properties** and **application-dev.properties**.

---

## Problem with `spring.profiles.active`

Sometimes, you want to **add more profiles automatically**, not just replace them.

For example:

- You might want **common** configuration for all environments.
- But you still want to run with a **specific profile** like **dev**, **prod**, etc.

If you only use `spring.profiles.active`, you would have to specify **all profiles** manually each time:

```bash
--spring.profiles.active=common,dev
```

That’s repetitive and error-prone.

---

## Solution: `spring.profiles.include`

The property **`spring.profiles.include`** allows you to **add additional profiles** that should always be active — no matter what’s set in `spring.profiles.active`.

It **includes extra profiles** instead of **replacing** the active ones.

---

## Example: Using `spring.profiles.include`

**`application.properties`**

```
spring.profiles.include[0]=common
spring.profiles.include[1]=local
```

Now, if you run your application with:

```bash
--spring.profiles.active=dev
```

Spring will activate **all three profiles**:

```
common, local, dev
```

---

### How Spring Processes This

- `spring.profiles.include` adds (`common`, `local`)
- `spring.profiles.active` adds (`dev`)

The final list of active profiles (in order) becomes:

```
common → local → dev
```

> Included profiles are added before any active profiles.
> 

This ordering matters — for example, when resolving property conflicts.

If the same property is defined in multiple profiles, **later profiles override earlier ones**.

---

## Real Example with Property Files

Suppose you have these files:

**`application.properties`**

```
spring.profiles.include[0]=common
spring.profiles.include[1]=local
```

**`application-common.properties`**

```
app.title=My Spring App
logging.level.root=INFO
```

**`application-local.properties`**

```
app.url=http://localhost:8080
```

**`application-dev.properties`**

```
app.database=H2
```

Now run:

```bash
java -jar app.jar --spring.profiles.active=dev
```

Spring will load configurations from:

1. `application-common.properties`
2. `application-local.properties`
3. `application-dev.properties`

You effectively get a **merged environment** with properties from all three.

---

### Important Rule

Just like `spring.profiles.active`,

**`spring.profiles.include` cannot be used inside profile-specific files.**

That means you **cannot** write this inside `application-dev.properties`:

```
spring.profiles.include=common
```

Why?

Because Spring only allows these profile activation properties (`spring.profiles.active`, `spring.profiles.include`) in **non-profile-specific** files (like `application.properties`).

Otherwise, Spring would face a circular activation issue (trying to load a profile to activate another one).

---

## When is this useful?

This is very handy when you have **shared configurations** that apply to multiple environments.

For example:

| Profile | Purpose |
| --- | --- |
| `common` | Shared config (logging, metrics, CORS, etc.) |
| `local` | Developer machine config |
| `dev` | Development server config |
| `prod` | Production config |

You can make sure every environment includes **common** automatically, without repeating it.

---

## Example of Combined Usage

Let’s say your `application.properties` looks like this:

```
spring.profiles.include=common
spring.profiles.active=prod
```

And you run without any CLI switches.

Active profiles will be:

```
common, prod
```

But if you run:

```bash
java -jar app.jar --spring.profiles.active=dev
```

Then active profiles will be:

```
common, dev
```

The **included profile (`common`) always stays**,

while the **active one (`dev` or `prod`) changes**.

# **Profile Groups**

## Why Profile Groups?

Sometimes, you define **too many small profiles** in your application to control different components independently — for example:

- `proddb` → enables production database configuration
- `prodmq` → enables production message queue configuration
- `prodsecurity` → enables production security configuration

This approach is flexible, but it becomes **cumbersome** when you have to activate multiple profiles together every time.

For example, to run your production setup, you might have to do:

```bash
java -jar app.jar --spring.profiles.active=proddb,prodmq,prodsecurity
```

That’s long, repetitive, and easy to mistype.

---

## What Are Profile Groups?

**Profile Groups** solve this problem.

They let you **define a single logical profile name** (like `production`) that automatically activates **multiple profiles** under it (like `proddb`, `prodmq`, etc.).

So instead of listing three profiles manually, you can just activate **one group**.

---

## Example of Defining a Profile Group

In `application.properties`

```
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

In YAML (equivalent)

```yaml
spring:
  profiles:
    group:
      production:
        - proddb
        - prodmq
```

---

## How It Works

When you start your application with:

```bash
--spring.profiles.active=production
```

Spring Boot will automatically activate these three profiles:

```
production, proddb, prodmq
```

That means:

- The `production` group name itself becomes active.
- All its **member profiles** (`proddb`, `prodmq`) are also activated.

---

## Example Setup

Let’s build a practical example.

**application.properties**

```
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

**application-proddb.properties**

```
app.datasource=Connected to Production Database
```

**application-prodmq.properties**

```
app.messaging=Connected to Production Message Queue
```

**application-production.properties**

```
app.mode=Production Mode
```

**Main Application Code**

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProfileGroupExample implements CommandLineRunner {

    @Value("${app.datasource:Not Configured}")
    private String datasource;

    @Value("${app.messaging:Not Configured}")
    private String messaging;

    @Value("${app.mode:Not Configured}")
    private String mode;

    public static void main(String[] args) {
        SpringApplication.run(ProfileGroupExample.class, args);
    }

    @Override
    public void run(String... args) {
        System.out.println("Mode: " + mode);
        System.out.println("Datasource: " + datasource);
        System.out.println("Messaging: " + messaging);
    }
}
```

---

**Run the Application**

Run with:

```bash
java -jar app.jar --spring.profiles.active=production
```

**Output**

```
Mode: Production Mode
Datasource: Connected to Production Database
Messaging: Connected to Production Message Queue
```

All three configurations (`production`, `proddb`, `prodmq`) were automatically activated through the **profile group**.

---

## Why This Is Useful

Profile groups help when:

- You want to **combine related profiles** for convenience.
- You have **environment-based profiles** (like `dev`, `test`, `staging`, `prod`) that internally depend on several component-specific profiles.

For example:

| Group Name | Contains Profiles |
| --- | --- |
| `production` | `proddb`, `prodmq`, `prodsecurity` |
| `staging` | `stagingdb`, `stagingmq` |
| `dev` | `devdb`, `devmq`, `devtools` |

Then you can easily switch environments by setting:

```bash
--spring.profiles.active=production
```

without worrying about listing all the internal profiles.

---

## Restrictions (Same as Active and Include)

Like `spring.profiles.active` and `spring.profiles.include`,

**`spring.profiles.group`** can only be defined in **non-profile-specific files**.

That means you cannot define it inside:

- `application-dev.properties`
- `application-prod.properties`
- or any file with `spring.config.activate.on-profile`.

Reason:

Profile group definitions must be available **before profile activation**.

If they were inside a profile-specific file, Spring wouldn’t know how to interpret them until after activation, which causes a circular dependency.

# **Programmatically Setting Profiles**

You can also include profiles **from Java code** using `SpringApplication`.

**Example**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProfileExampleApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(ProfileExampleApplication.class);
        app.setAdditionalProfiles("common", "local");
        app.run(args);
    }
}
```

This does exactly the same as:

```
spring.profiles.include=common,local
```