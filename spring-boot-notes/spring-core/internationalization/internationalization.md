# Internationalization

# Key concepts (short)

- **Message bundles**: property files like `messages.properties`, `messages_fr.properties` etc. Spring Boot looks for a default `messages.properties` on the classpath to auto-configure a `MessageSource`. If you only have language-specific files, you must also provide the default base file. Home
- *spring.messages. properties*: configure basename(s), extra common message files, fallback behaviour, encoding, cache seconds, etc. The `basename` supports a comma-separated list of package names or classpath-root resources. [Home](https://docs.spring.io/spring-boot/reference/features/internationalization.html?utm_source=chatgpt.com)
- **MessageSource implementations**: `ResourceBundleMessageSource` and `ReloadableResourceBundleMessageSource` are provided by Spring; Boot auto-configures an appropriate `MessageSource` for you when it detects message files. [Home](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/MessageSource.html?utm_source=chatgpt.com)
- **Locale resolution**: Spring MVC uses a `LocaleResolver` (or `LocaleContextResolver`) to determine the current user’s `Locale`. Common resolvers: `AcceptHeaderLocaleResolver` (reads `Accept-Language`), `SessionLocaleResolver`, `CookieLocaleResolver`. You can also use `LocaleChangeInterceptor` to switch locale via a request parameter. [Baeldung on Kotlin+1](https://www.baeldung.com/spring-boot-internationalization?utm_source=chatgpt.com)

---

# Example application — goal

Create a Spring Boot web app that:

- Serves localized messages from `messages.properties` and `messages_fr.properties`.
- Supports switching languages via a `lang` request parameter (e.g., `?lang=fr`) **and** via `Accept-Language` header.
- Demonstrates fallback to default messages when a key is missing in the chosen locale.
- Shows `spring.messages.basename` using multiple base names.

---

## Project structure (Gradle — Java)

```
spring-i18n-example/
├─ src/
│  ├─ main/
│  │  ├─ java/com/example/i18n/
│  │  │  ├─ I18nApplication.java
│  │  │  ├─ WebConfig.java
│  │  │  └─ GreetingController.java
│  │  └─ resources/
│  │     ├─ application.yml
│  │     ├─ messages.properties
│  │     ├─ messages_fr.properties
│  │     └─ config/i18n/common.properties
├─ build.gradle.kts
```

### Gradle dependencies (build.gradle.kts)

```kotlin
plugins {
    id("org.springframework.boot") version "3.2.0" // adapt to your Spring Boot version
    id("io.spring.dependency-management") version "1.1.0"
    kotlin("jvm") version "1.9.0" apply false // or use plain java
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    // If you use templates (Thymeleaf) add:
    // implementation("org.springframework.boot:spring-boot-starter-thymeleaf")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

---

## application.yml

This shows multiple basenames and a `common-messages` resource example. `fallback-to-system-locale: false` avoids unexpected system-locale fallback. (These properties are honored by Spring Boot auto-config.) [Home](https://docs.spring.io/spring-boot/reference/features/internationalization.html?utm_source=chatgpt.com)

```yaml
spring:
  messages:
    basename: messages, config.i18n.common   # comma-separated list (package or path)
    common-messages: classpath:config/i18n/common.properties
    fallback-to-system-locale: false
    encoding: UTF-8
    cache-seconds: 3600
```

---

## message files (examples)

### src/main/resources/messages.properties (default/base)

```
greeting=Hello, {0}!
farewell=Goodbye!
welcome=Welcome to our app.
```

### src/main/resources/messages_fr.properties (French)

```
greeting=Bonjour, {0}!
farewell=Au revoir!
# note: intentionally omit 'welcome' to demonstrate fallback to default messages.properties
```

### src/main/resources/config/i18n/common.properties

```
app.name=My I18n App
```

---

## Java configuration — `WebConfig.java`

This config does three things:

1. Registers a `LocaleResolver` (session-based or accept-header — see comments).
2. Adds a `LocaleChangeInterceptor` so clients can switch language with a `lang` parameter (e.g. `?lang=fr`).
3. Demonstrates how to keep `Accept-Language` fallback if you prefer.

```java
package com.example.i18n;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.*;
import org.springframework.web.servlet.i18n.*;

import java.util.Locale;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    // Option A: use SessionLocaleResolver (allows changing via LocaleChangeInterceptor)
    // Uncomment to use session-based resolver:
    // @Bean
    // public LocaleResolver localeResolver() {
    //     SessionLocaleResolver resolver = new SessionLocaleResolver();
    //     resolver.setDefaultLocale(Locale.ENGLISH);
    //     return resolver;
    // }

    // Option B (recommended when you want Accept-Language header by default):
    // Use AcceptHeaderLocaleResolver to respect client's Accept-Language header.
    @Bean
    public LocaleResolver localeResolver() {
        AcceptHeaderLocaleResolver resolver = new AcceptHeaderLocaleResolver();
        resolver.setDefaultLocale(Locale.ENGLISH);
        return resolver;
    }

    // Interceptor that switches locale when 'lang' parameter is present (e.g., ?lang=fr).
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang"); // default is "locale" but using "lang" is common
        return interceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // Register the interceptor so that it sees each request
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```

Notes:

- If you use `AcceptHeaderLocaleResolver`, `LocaleChangeInterceptor` will still change the `LocaleContext` for the processing of that request but it will not persist across sessions (since AcceptHeaderResolver uses the header). If you want persisted changes, use `SessionLocaleResolver` or `CookieLocaleResolver`. Guides on these behaviors are widely used. [Baeldung on Kotlin+1](https://www.baeldung.com/spring-boot-internationalization?utm_source=chatgpt.com)

---

## Controller — `GreetingController.java`

Demonstrates looking up messages programmatically via `MessageSource` and using `Locale` from the request.

```java
package com.example.i18n;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.web.bind.annotation.*;

import java.util.Locale;

@RestController
@RequestMapping("/api")
public class GreetingController {

    private final MessageSource messageSource;

    @Autowired
    public GreetingController(MessageSource messageSource) {
        this.messageSource = messageSource;
    }

    // Example: GET /api/greet?name=Praveen
    @GetMapping("/greet")
    public String greet(@RequestParam(defaultValue = "User") String name, Locale locale) {
        // The Locale parameter is resolved by Spring using the configured LocaleResolver
        String greeting = messageSource.getMessage("greeting", new Object[]{name}, locale);
        String appName = messageSource.getMessage("app.name", null, locale);
        return greeting + " — " + appName;
    }

    // Example: GET /api/welcome (demonstrates fallback when locale file is missing the key)
    @GetMapping("/welcome")
    public String welcome(Locale locale) {
        String welcome = messageSource.getMessage("welcome", null, locale);
        return welcome;
    }
}
```

Notes:

- `messageSource.getMessage(code, args, locale)` uses the resolved locale. If the key is missing in the locale-specific file, it falls back to the default `messages.properties` base file (if present). If not found anywhere, a `NoSuchMessageException` may be thrown (or you can pass a default message overload). [Home](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/MessageSource.html?utm_source=chatgpt.com)

---

## Boot application (main)

```java
package com.example.i18n;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class I18nApplication {
    public static void main(String[] args) {
        SpringApplication.run(I18nApplication.class, args);
    }
}
```

---

# How to run and test (examples)

1. Start the app: `./gradlew bootRun` (or run from your IDE).
2. Default (English):

```
curl -s 'http://localhost:8080/api/greet?name=Praveen'
# Response: Hello, Praveen! — My I18n App
```

1. Switch to French using query param (works because of LocaleChangeInterceptor):

```
curl -s 'http://localhost:8080/api/greet?name=Praveen&lang=fr'
# Response: Bonjour, Praveen! — My I18n App
```

1. Switch using Accept-Language header (useful for API clients / browsers):

```
curl -s -H "Accept-Language: fr" 'http://localhost:8080/api/greet?name=Praveen'
# Response: Bonjour, Praveen! — My I18n App
```

1. Demonstrate fallback: `welcome` key is missing in `messages_fr.properties`, so French requests fall back to `messages.properties`.

```
curl -s -H "Accept-Language: fr" 'http://localhost:8080/api/welcome'
# Response: Welcome to our app.
```

## What is the main code of internationalization in Spring Boot?

At the core of i18n in Spring Boot (and Spring Framework) lies one key interface:

### **`MessageSource`**

It is responsible for **resolving messages** (text) for a given key and locale.

In other words, when you call:

```java
messageSource.getMessage("greeting", new Object[]{"Praveen"}, locale);
```

Spring looks for:

- A file (like `messages.properties`, `messages_fr.properties`, etc.)
- A key (`greeting`)
- A locale (e.g., `Locale.FRENCH`)

Then it returns the message text defined in the correct language file.

---

### **The main class behind it**

`org.springframework.context.support.ResourceBundleMessageSource`

(or `ReloadableResourceBundleMessageSource`)

This class:

- Loads `.properties` files from the classpath.
- Uses the provided locale to pick the right file (e.g., `messages_fr.properties` for French).
- Falls back to the default file (`messages.properties`) if no match is found.

Spring Boot **auto-configures** this `MessageSource` for you if it finds `messages.properties` in the classpath.

---

### Example: The main code in action

```java
@Autowired
private MessageSource messageSource;

public void showMessage() {
    Locale locale = Locale.FRENCH;
    String msg = messageSource.getMessage("greeting", new Object[]{"Praveen"}, locale);
    System.out.println(msg); // Output: Bonjour, Praveen!
}

```

This **is the main code** that makes i18n work at runtime.

Spring uses the locale to fetch the corresponding language string.

---

## How does Internationalization work internally?

Let’s look at the **step-by-step process** behind the scenes:

### Step 1 — Spring Boot autoconfiguration

- When your project starts, Spring Boot looks for a `messages.properties` file in the classpath.
- If it finds one, it automatically creates a `MessageSource` bean for you.
- You can customize its location using:
    
    ```
    spring.messages.basename=messages,config.i18n.common
    ```
    

---

### Step 2 — Locale determination

Before resolving a message, Spring must know **which locale** to use.

Spring can determine the locale in several ways (see section 3 below).

Examples:

- From browser’s `Accept-Language` header
- From a URL parameter (e.g., `?lang=fr`)
- From a session or cookie

The resolved locale is stored in a `LocaleContext`.

---

### Step 3 — Message resolution

When you call `messageSource.getMessage("key", args, locale)`:

1. Spring checks if there’s a property file for the given locale (e.g., `messages_fr.properties`).
2. If found, it retrieves the value for the key.
3. If not found, it falls back to the default file `messages.properties`.
4. If still not found, it throws `NoSuchMessageException`.

---

### Step 4 — Message interpolation

If your message includes placeholders like `{0}` or `{1}`, Spring automatically replaces them with the arguments provided.

Example:

```
greeting=Hello, {0}!
```

When called:

```java
messageSource.getMessage("greeting", new Object[]{"Praveen"}, Locale.ENGLISH);
```

Output → `Hello, Praveen!`

---

## How many ways can Internationalization be implemented in Spring Boot?

There are **three main ways** to implement i18n, depending on how you want to resolve the `Locale`.

---

### **Way 1: AcceptHeaderLocaleResolver (default in Spring Boot 2.6+)**

- Locale is determined automatically from the HTTP `Accept-Language` header sent by the browser or client.
- No manual configuration is needed.
- Stateless — suitable for REST APIs.

**Example:**

```java
@Bean
public LocaleResolver localeResolver() {
    AcceptHeaderLocaleResolver resolver = new AcceptHeaderLocaleResolver();
    resolver.setDefaultLocale(Locale.ENGLISH);
    return resolver;
}
```

Now, if a user’s browser sends `Accept-Language: fr`, the app uses French messages.

**Use case:** APIs or stateless applications.

---

### **Way 2: SessionLocaleResolver (locale stored in session)**

- Locale is stored in the user’s HTTP session.
- You can switch locale dynamically using a `LocaleChangeInterceptor`.

**Example:**

```java
@Bean
public LocaleResolver localeResolver() {
    SessionLocaleResolver resolver = new SessionLocaleResolver();
    resolver.setDefaultLocale(Locale.ENGLISH);
    return resolver;
}

@Bean
public LocaleChangeInterceptor localeChangeInterceptor() {
    LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
    interceptor.setParamName("lang");
    return interceptor;
}

@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(localeChangeInterceptor());
}
```

Now you can switch languages with:

```
GET /api/greet?lang=fr
```

**Use case:** Web apps where the user manually switches language.

---

### **Way 3: CookieLocaleResolver**

- Stores the user’s selected locale in a browser cookie.
- The language preference persists across sessions.

**Example:**

```java
@Bean
public LocaleResolver localeResolver() {
    CookieLocaleResolver resolver = new CookieLocaleResolver();
    resolver.setDefaultLocale(Locale.ENGLISH);
    resolver.setCookieName("localeInfo");
    resolver.setCookieMaxAge(3600);
    return resolver;
}
```

**Use case:** Web apps that should remember the language preference between sessions.

---

## 4. Summary of the three approaches

| Locale Resolver Type | How Locale is Determined | Persistent | Suitable For |
| --- | --- | --- | --- |
| **AcceptHeaderLocaleResolver** | From browser’s `Accept-Language` header | No | REST APIs / Stateless |
| **SessionLocaleResolver** | Stored in HTTP Session | Yes | Web apps |
| **CookieLocaleResolver** | Stored in Browser Cookie | Yes | Web apps |

---

## 5. Summary of i18n components

| Component | Purpose | Example |
| --- | --- | --- |
| **MessageSource** | Core interface to fetch localized messages | `messageSource.getMessage("key", args, locale)` |
| **ResourceBundleMessageSource** | Loads `.properties` files from classpath | Default implementation |
| **LocaleResolver** | Determines which locale to use | AcceptHeader, Session, Cookie |
| **LocaleChangeInterceptor** | Allows changing locale dynamically | URL param like `?lang=fr` |
| *spring.messages. properties** | Configure message file base names and settings | `spring.messages.basename=messages` |