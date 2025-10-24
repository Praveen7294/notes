# JSON

- [Introduction](#introduction)
- [Jackson](#jackson)
    1. [Custom Serializers and Deserializers](#custom-serializers-and-deserializers)
    2. [Mixins](#mixins)
- [GSON](#gson)
- [JSON-B](#json-b)

# Introduction

**JSON (JavaScript Object Notation)** is a **lightweight data-interchange format** used to store and exchange data between systems, especially between a server and a web application.

It is **text-based**, **language-independent**, and **easy for humans to read and write** while also being **easy for machines to parse and generate**.

### Structure of JSON

JSON represents data as **key-value pairs**, similar to how a Java `Map` or Python `dictionary` works.

A JSON object is enclosed in curly braces `{ }`, and arrays are enclosed in square brackets `[ ]`.

**Example:**

```json
{
  "name": "John",
  "age": 25,
  "email": "john@example.com",
  "skills": ["Java", "Spring Boot", "SQL"]
}
```

---

### JSON Data Types

JSON supports only a few basic data types:

| Type | Example |
| --- | --- |
| String | `"Hello"` |
| Number | `25`, `3.14` |
| Boolean | `true`, `false` |
| Object | `{ "key": "value" }` |
| Array | `[1, 2, 3]` |
| Null | `null` |

---

### Key Rules of JSON

- Data is written as **key-value pairs** (`"key": "value"`).
- Keys must be **strings** enclosed in **double quotes**.
- Each key-value pair is separated by a comma.
- Objects are enclosed in `{ }` and arrays in `[ ]`.

---

### Why JSON is Used

- **Data exchange format** in RESTful APIs.
- **Lightweight and faster** than XML.
- **Easily parsed** by most programming languages.
- **Human-readable** structure.

---

### Example of Server-Client Use

When a client requests user data from a server, the server might respond with JSON like:

**Response:**

```json
{
  "userId": 101,
  "name": "Alice",
  "role": "Admin"
}

```

The client (like a web browser or mobile app) can then use this JSON data to display information or perform actions.

Spring Boot provides integration with three JSON mapping libraries:

- Gson
- Jackson
- JSON-B

Jackson is the preferred and default library.

## **Jackson**

### **What is Jackson?**

**Jackson** is a popular Java library used to **convert Java objects to JSON** (serialization) and **JSON to Java objects** (deserialization).

Spring Boot uses it internally to handle all JSON data by default.

---

### **What is Auto-Configuration in Spring Boot?**

**Auto-configuration** is one of Spring Boot’s core features.

It automatically sets up and configures beans in your application context based on what **libraries (dependencies)** are present on the **classpath** and your application’s environment.

In other words, Spring Boot detects the libraries you have added and automatically configures them for you — without requiring manual setup.

---

### **spring-boot-starter-json**

The dependency **`spring-boot-starter-json`** contains everything needed to handle JSON in Spring Boot.

It includes:

- **Jackson Databind** (for JSON serialization and deserialization)
- **Jackson Core**
- **Jackson Annotations**

When you use `spring-boot-starter-web` (which most web apps use), it automatically pulls in `spring-boot-starter-json`, so you don’t need to add it manually.

---

### **Automatic Creation of ObjectMapper Bean**

The **`ObjectMapper`** class is the core component of Jackson.

It is responsible for converting:

- Java objects → JSON
- JSON → Java objects

When Spring Boot detects that Jackson is on the classpath (i.e., you added it through the starter), it **automatically creates and configures an `ObjectMapper` bean** for you in the Spring Application Context.

This means:

- You don’t need to write `new ObjectMapper()` manually.
- You can inject it anywhere using:
    
    ```java
    @Autowired
    private ObjectMapper objectMapper;
    ```
    

Spring Boot will provide a **pre-configured ObjectMapper** with sensible defaults.

---

### **Customizing ObjectMapper via Configuration Properties**

Spring Boot also allows you to **customize Jackson’s behavior** through properties in the `application.properties` or `application.yml` file — without writing extra Java code.

Some commonly used properties:

```
# Enable pretty printing (indented JSON)
spring.jackson.serialization.indent_output=true

# Include only non-null fields
spring.jackson.default-property-inclusion=non_null

# Change the date format
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss

# Change the property naming strategy (camelCase → snake_case)
spring.jackson.property-naming-strategy=SNAKE_CASE
```

Spring Boot applies these settings automatically to the same `ObjectMapper` bean that it created.

---

### **Summary**

| Concept | Explanation |
| --- | --- |
| **Jackson** | Library for handling JSON in Java |
| **Auto-configuration** | Spring Boot automatically sets up required beans when dependencies are detected |
| **`spring-boot-starter-json`** | Starter dependency that brings Jackson support |
| **`ObjectMapper` bean** | Automatically created by Spring Boot for JSON conversion |
| **Configuration properties** | Allow easy customization of JSON behavior (pretty print, date format, naming, etc.) |

---

### Example

**application.properties**

```
spring.jackson.property-naming-strategy=SNAKE_CASE
spring.jackson.serialization.indent_output=true
```

**Controller**

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User("John", "Doe", 25);
    }
}
```

**Output JSON:**

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "age": 25
}
```

Here, the snake_case naming and pretty printing are applied automatically through configuration—no manual `ObjectMapper` setup needed.

## **Custom Serializers and Deserializers**

### **Why Do We Need Custom JSON Serialization/Deserialization?**

Normally, Jackson (used by Spring Boot) automatically converts Java objects to JSON and vice versa.

However, sometimes you need to **customize this process** — for example:

- Change how certain fields are displayed.
- Combine or split fields.
- Format date/time or numbers differently.
- Handle nested objects or complex data formats.

That’s where **custom serializers and deserializers** come in.

---

### **What Is `@JsonComponent` in Spring Boot?**

Spring Boot provides the `@JsonComponent` annotation as a **simple way to register custom Jackson serializers and deserializers automatically**.

Normally, in plain Jackson, you’d have to manually register your serializer/deserializer with a `SimpleModule` and add it to the `ObjectMapper`.

But with Spring Boot, if you use `@JsonComponent`, Spring **auto-detects and registers** them for you — no manual configuration required.

Because `@JsonComponent` is itself annotated with `@Component`, it works with **Spring’s component scanning**.

---

### **How It Works**

When you create a class annotated with `@JsonComponent`, and inside it define:

- A `JsonSerializer<T>` for **custom serialization**
- A `JsonDeserializer<T>` for **custom deserialization**

Spring Boot automatically registers them in the Jackson `ObjectMapper` that it manages.

---

### **Example Step by Step**

Let’s take a simple example where we have a `MyObject` class.

```java
public class MyObject {
    private String name;
    private int age;

    // constructor
    public MyObject(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getters and setters
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

---

**Creating a Custom Serializer and Deserializer**

We can create a `@JsonComponent` that includes both serializer and deserializer.

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import org.springframework.boot.jackson.JsonComponent;

import java.io.IOException;

@JsonComponent
public class MyJsonComponent {

    // Custom Serializer
    public static class Serializer extends JsonSerializer<MyObject> {
        @Override
        public void serialize(MyObject value, JsonGenerator jgen, SerializerProvider serializers) throws IOException {
            jgen.writeStartObject();
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
            jgen.writeEndObject();
        }
    }

    // Custom Deserializer
    public static class Deserializer extends JsonDeserializer<MyObject> {
        @Override
        public MyObject deserialize(JsonParser jsonParser, DeserializationContext ctxt) throws IOException {
            ObjectCodec codec = jsonParser.getCodec();
            JsonNode tree = codec.readTree(jsonParser);
            String name = tree.get("name").asText();
            int age = tree.get("age").asInt();
            return new MyObject(name, age);
        }
    }
}
```

---

**How This Works**

- The **Serializer** tells Jackson how to convert `MyObject` into JSON.
    
    Instead of using default field mapping, it writes the JSON manually using `JsonGenerator`.
    
- The **Deserializer** tells Jackson how to convert JSON back into a `MyObject`.
    
    It reads the JSON tree, extracts the fields, and creates a new `MyObject`.
    
- Because of `@JsonComponent`, Spring Boot automatically registers these with the global `ObjectMapper`.

---

**Controller Example**

```java
@RestController
@RequestMapping("/api")
public class MyController {

    @GetMapping("/myobject")
    public MyObject getObject() {
        return new MyObject("Alice", 30);
    }

    @PostMapping("/myobject")
    public String postObject(@RequestBody MyObject myObject) {
        return "Received: " + myObject.getName() + " (" + myObject.getAge() + ")";
    }
}
```

**GET `/api/myobject` → JSON Response**

```json
{
  "name": "Alice",
  "age": 30
}
```

**POST `/api/myobject` → Request JSON**

```json
{
  "name": "Alice",
  "age": 30
}
```

The custom serializer and deserializer will automatically handle these conversions.

---

**Using Spring’s `JsonObjectSerializer` and `JsonObjectDeserializer`**

Spring Boot provides helper base classes:

- `JsonObjectSerializer<T>` – simplifies serialization.
- `JsonObjectDeserializer<T>` – simplifies deserialization.

You can extend them instead of writing full Jackson boilerplate.

**Example using them**

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonNode;
import org.springframework.boot.jackson.JsonComponent;
import org.springframework.boot.jackson.JsonObjectSerializer;
import org.springframework.boot.jackson.JsonObjectDeserializer;
import java.io.IOException;

@JsonComponent
public class MyJsonComponent {

    // Custom Serializer
    public static class Serializer extends JsonObjectSerializer<MyObject> {
        @Override
        protected void serializeObject(MyObject value, JsonGenerator jgen, SerializerProvider provider)
                throws IOException {
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
        }
    }

    // Custom Deserializer
    public static class Deserializer extends JsonObjectDeserializer<MyObject> {
        @Override
        protected MyObject deserializeObject(JsonParser jsonParser, DeserializationContext context,
                                             ObjectCodec codec, JsonNode tree) throws IOException {
            String name = nullSafeValue(tree.get("name"), String.class);
            int age = nullSafeValue(tree.get("age"), Integer.class);
            return new MyObject(name, age);
        }
    }
}
```

Here:

- `JsonObjectSerializer` automatically handles the start and end of the JSON object.
- `JsonObjectDeserializer` provides a convenient `nullSafeValue()` method to safely read JSON fields.

This reduces boilerplate and is the **recommended** approach in Spring Boot.

## **Mixins**

### **What are Mixins in Jackson?**

In Jackson, **Mixins** are a powerful feature that allow you to **add or override annotations** (like `@JsonProperty`, `@JsonIgnore`, etc.) **to a class without modifying its source code**.

This is especially useful when:

- You want to customize how JSON is serialized/deserialized for a **third-party class** (i.e., a class you cannot edit).
- You want different JSON behaviors in different parts of your application without changing the original class.

---

### **How Mixins Work**

Normally, Jackson uses annotations directly on classes to know how to handle JSON conversion:

```java
public class User {
    private String name;
    private String password;

    @JsonIgnore
    public String getPassword() {
        return password;
    }
}
```

But suppose you **cannot modify** the `User` class (for example, it’s from a library).

You can use a **Mixin class** to tell Jackson how to treat it:

```java
public abstract class UserMixin {
    @JsonIgnore
    abstract String getPassword();
}
```

Now, you tell Jackson to **apply this mixin** to the `User` class:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.addMixIn(User.class, UserMixin.class);
```

Jackson will now treat the `User` class as if the `@JsonIgnore` annotation were actually present on its `getPassword()` method.

---

### **Spring Boot’s Auto-Configuration for Mixins**

Spring Boot makes this easier with **auto-configuration**.

When you use Spring Boot:

- Jackson is automatically configured through `spring-boot-starter-json`.
- An `ObjectMapper` bean is auto-created by Spring Boot.
- Spring Boot can automatically **scan for mixins** in your project.

So if you annotate a class with `@JsonMixin`, like this:

```java
@JsonMixin(target = User.class)
public abstract class UserMixin {
    @JsonIgnore
    abstract String getPassword();
}
```

Spring Boot will automatically detect this mixin and register it with the `ObjectMapper` for you.

You don’t have to manually call `mapper.addMixIn(...)` — Spring Boot does that internally using something called **JsonMixinModule**.

---

### **What is `JsonMixinModule`?**

`JsonMixinModule` is an internal Spring Boot component that:

- Scans your application for any classes annotated with `@JsonMixin`.
- Automatically registers them with the main `ObjectMapper`.

This means your custom JSON behavior is active **immediately** and **globally**, without extra setup code.

# **Gson**

## **What is Gson?**

- Gson is a **Java library for JSON serialization and deserialization**.
- It is **lightweight**, **fast**, and **easy to use**.
- It supports **Java objects**, including **collections** (like List, Map) and **nested objects**.
- It can also **format JSON**, **ignore fields**, and handle **custom types** using custom serializers/deserializers.

Gson is commonly used when:

- You want to **convert Java objects to JSON** for APIs or storage.
- You want to **read JSON and map it to Java objects**.
- You prefer a **Google-maintained library** instead of Jackson.

---

## **How Gson Works**

Gson provides two main functionalities:

1. **Serialization** – Converting Java objects into JSON.
2. **Deserialization** – Converting JSON strings into Java objects.

---

### **Serialization Example**

```java
import com.google.gson.Gson;

class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

public class GsonExample {
    public static void main(String[] args) {
        User user = new User("Alice", 25);

        Gson gson = new Gson();
        String json = gson.toJson(user);  // Convert Java object to JSON

        System.out.println(json);
    }
}
```

**Output:**

```json
{"name":"Alice","age":25}
```

---

### **Deserialization Example**

```java
String json = "{\"name\":\"Bob\",\"age\":30}";

Gson gson = new Gson();
User user = gson.fromJson(json, User.class);  // Convert JSON to Java object

System.out.println(user.getName());  // Bob
System.out.println(user.getAge());   // 30
```

---

## **Handling Collections**

Gson can also handle lists, maps, and nested objects.

```java
import java.util.List;
import com.google.gson.reflect.TypeToken;

String jsonList = "[{\"name\":\"Alice\",\"age\":25},{\"name\":\"Bob\",\"age\":30}]";

Gson gson = new Gson();
List<User> users = gson.fromJson(jsonList, new TypeToken<List<User>>(){}.getType());

System.out.println(users.size());  // 2
```

---

## **Custom Serialization/Deserialization**

Sometimes, default behavior is not enough. You can write **custom serializers and deserializers**:

```java
import com.google.gson.*;

class UserSerializer implements JsonSerializer<User> {
    @Override
    public JsonElement serialize(User user, java.lang.reflect.Type typeOfSrc, JsonSerializationContext context) {
        JsonObject obj = new JsonObject();
        obj.addProperty("fullName", user.getName());  // Rename field
        obj.addProperty("age", user.getAge());
        return obj;
    }
}

Gson gson = new GsonBuilder().registerTypeAdapter(User.class, new UserSerializer()).create();
String json = gson.toJson(new User("Alice", 25));
System.out.println(json);  // {"fullName":"Alice","age":25}
```

---

## **Features of Gson**

- **Automatic Mapping:** Automatically maps Java objects to JSON and vice versa.
- **Null Handling:** Can include or exclude null fields.
- **Pretty Printing:** Can output formatted JSON for readability.
- **Custom Adapters:** Allows you to define custom serializers and deserializers.
- **Type Support:** Works with generic types, nested objects, and collections.
- **Annotations:** Supports `@Expose` and `@SerializedName` to control field serialization.

---

### **Annotations in Gson**

- **`@SerializedName("json_name")`** – Maps a Java field to a different JSON field name.
- **`@Expose`** – Include/exclude fields selectively during serialization/deserialization.

Example:

```java
class User {
    @SerializedName("full_name")
    private String name;

    @Expose(serialize = false, deserialize = false)
    private String password;
}
```

---

## **Gson vs Jackson**

| Feature | Gson | Jackson |
| --- | --- | --- |
| Performance | Slightly slower for large datasets | Faster for large datasets |
| Annotations | Limited (`@SerializedName`, `@Expose`) | Rich set (`@JsonIgnore`, `@JsonProperty`, etc.) |
| Streaming API | Yes (JsonReader/JsonWriter) | Yes (faster and more feature-rich) |
| Spring Boot Integration | Not default; needs manual configuration | Default library for Spring Boot |
| Customization | Custom serializers/deserializers via `GsonBuilder` | Extensive with `@JsonComponent` and modules |

## **What the statement means**

Spring Boot has **auto-configuration support** for Gson — just like it does for Jackson.

This means:

- When **Gson** is **present on your classpath** (i.e., you added Gson dependency in your project),
- Spring Boot **automatically creates and configures a Gson bean** for you,
- You can **customize its behavior** through application properties or custom configuration beans.

In short, **you don’t need to manually create or configure Gson yourself** — Spring Boot does it automatically.

---

## **Automatic Gson Bean Creation**

When your project includes the Gson library (for example, by adding this dependency):

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
</dependency>
```

Spring Boot’s auto-configuration detects it and automatically creates a **`Gson`** bean.

That bean is used internally by Spring Boot’s HTTP message converters (e.g., to convert Java objects to JSON in REST APIs).

So, if you return a Java object from a controller method, it will automatically be converted to JSON using Gson.

Example:

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User("Alice", 25);
    }
}
```

If Gson is on your classpath, this endpoint will automatically return:

```json
{
  "name": "Alice",
  "age": 25
}
```

---

## **Configuration using `spring.gson.*` properties**

Spring Boot provides several **configuration properties** that you can define in your `application.properties` (or `application.yml`) to control Gson’s behavior.

For example:

```
spring.gson.pretty-printing=true
spring.gson.serialize-nulls=true
spring.gson.date-format=yyyy-MM-dd
spring.gson.disable-html-escaping=true
```

**Explanation:**

- `spring.gson.pretty-printing=true` → Enables formatted (indented) JSON output.
- `spring.gson.serialize-nulls=true` → Includes null values in JSON.
- `spring.gson.date-format=yyyy-MM-dd` → Sets a custom date format.
- `spring.gson.disable-html-escaping=true` → Prevents escaping of HTML characters (like `<`, `>`).

---

## **Using `GsonBuilderCustomizer` for more control**

If you want **more customization** than what `spring.gson.*` offers, you can define one or more **`GsonBuilderCustomizer` beans**.

Spring Boot automatically applies them to the default `GsonBuilder` before the `Gson` bean is created.

Example:

```java
import org.springframework.boot.autoconfigure.gson.GsonBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.google.gson.FieldNamingPolicy;

@Configuration
public class GsonConfig {

    @Bean
    public GsonBuilderCustomizer customGsonBuilder() {
        return builder -> builder
                .setPrettyPrinting()
                .serializeNulls()
                .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE);
    }
}
```

**What happens:**

- Spring Boot automatically finds this customizer bean.
- It applies the settings (`setPrettyPrinting`, `serializeNulls`, etc.) to the GsonBuilder.
- Then it creates the final Gson instance from that builder.

---

## **Order of customization**

Spring Boot applies customizations in this order:

1. Default configuration (basic setup)
2. `spring.gson.*` properties
3. Your custom `GsonBuilderCustomizer` beans

This means **your customizations override default ones** if there’s a conflict.

# JSON-B

**JSON-B (JSON Binding)** is a **standard specification in Java** for converting Java objects to JSON and vice versa. It is part of the **Java EE / Jakarta EE ecosystem** (JSR 367) and provides a standardized way to perform JSON serialization and deserialization across Java applications. Let’s go step by step.

---

## **What is JSON-B?**

- JSON-B stands for **JSON Binding**.
- It is a **standard Java API** (similar to JAXB for XML) that defines:
    - How Java objects are converted to JSON (**serialization**)
    - How JSON is converted back to Java objects (**deserialization**)
- Unlike Jackson or Gson, which are libraries, JSON-B is a **specification**, and there are **multiple implementations**, such as:
    - **Eclipse Yasson** (the reference implementation)
    - **Apache Johnzon**
- JSON-B is part of the **Jakarta EE** platform, making it a standard choice for enterprise applications.

---

## **Core Concepts of JSON-B**

JSON-B provides a **simple API** to work with JSON in Java. The main components are:

1. **`Jsonb` interface** – The primary API for JSON binding operations.
2. **Annotations** – Define how Java objects are mapped to JSON and vice versa.
3. **Configuration** – Allows customization of serialization/deserialization behavior.

---

### **The `Jsonb` Interface**

The `Jsonb` interface is the entry point for JSON-B operations. You typically create it using `JsonbBuilder`.

**Example:**

```java
import jakarta.json.bind.Jsonb;
import jakarta.json.bind.JsonbBuilder;

public class JsonBExample {
    public static void main(String[] args) throws Exception {
        User user = new User("Alice", 25);

        Jsonb jsonb = JsonbBuilder.create();

        // Serialization: Java object -> JSON
        String json = jsonb.toJson(user);
        System.out.println(json);

        // Deserialization: JSON -> Java object
        User user2 = jsonb.fromJson(json, User.class);
        System.out.println(user2.getName() + " - " + user2.getAge());
    }
}

class User {
    private String name;
    private int age;

    public User() {}  // Default constructor required

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getters and setters
    public String getName() { return name; }
    public int getAge() { return age; }
    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
}
```

**Output:**

```json
{"name":"Alice","age":25}
```

---

### **JSON-B Annotations**

JSON-B provides a set of annotations similar to Jackson or JAXB:

| Annotation | Purpose |
| --- | --- |
| `@JsonbProperty("name")` | Map a Java field to a specific JSON property name |
| `@JsonbTransient` | Ignore a field during serialization/deserialization |
| `@JsonbDateFormat("yyyy-MM-dd")` | Format dates in a specific pattern |
| `@JsonbNumberFormat("#.##")` | Format numbers during serialization |
| `@JsonbCreator` | Use for custom object creation during deserialization |

**Example:**

```java
import jakarta.json.bind.annotation.JsonbProperty;
import jakarta.json.bind.annotation.JsonbTransient;
import jakarta.json.bind.annotation.JsonbDateFormat;
import java.util.Date;

public class User {
    @JsonbProperty("full_name")
    private String name;

    private int age;

    @JsonbTransient
    private String password;  // will be ignored in JSON

    @JsonbDateFormat("yyyy-MM-dd")
    private Date dob;

    // getters, setters, constructors
}
```

---

### **Custom Serialization/Deserialization**

JSON-B allows you to write **custom adapters** for complex types.

```java
import jakarta.json.bind.adapter.JsonbAdapter;

public class DateAdapter implements JsonbAdapter<Date, String> {
    @Override
    public String adaptToJson(Date obj) {
        return new SimpleDateFormat("yyyy-MM-dd").format(obj);
    }

    @Override
    public Date adaptFromJson(String obj) throws Exception {
        return new SimpleDateFormat("yyyy-MM-dd").parse(obj);
    }
}
```

You can then annotate the field:

```java
@JsonbTypeAdapter(DateAdapter.class)
private Date dob
```

---

### **Configuration**

JSON-B can be configured using a **`JsonbConfig` object**:

```java
import jakarta.json.bind.JsonbConfig;
import jakarta.json.bind.JsonbBuilder;

JsonbConfig config = new JsonbConfig()
        .withFormatting(true)   // pretty print
        .withNullValues(false); // exclude nulls

Jsonb jsonb = JsonbBuilder.create(config);
```

### **Comparison with Gson and Jackson**

| Feature | JSON-B | Gson | Jackson |
| --- | --- | --- | --- |
| Standard | Yes (JSR 367) | No | No |
| Default in Spring Boot | No (Jackson is default) | No | Yes |
| Annotations | `@JsonbProperty`, `@JsonbTransient` | `@SerializedName`, `@Expose` | `@JsonProperty`, `@JsonIgnore` |
| Customization | JsonbAdapter, JsonbConfig | TypeAdapter, GsonBuilder | JsonSerializer, @JsonComponent, ObjectMapper |
| Integration with Jakarta EE | Native | Manual | Manual |

## JSON-B in spring boot

Spring Boot supports **JSON-B** in a similar way to Jackson or Gson:

1. **Auto-configuration**:
    
    Spring Boot can automatically create a `Jsonb` bean if it detects that **both the JSON-B API and an implementation** are on your classpath.
    
2. **Bean availability**:
    
    Once the auto-configuration runs, a **`Jsonb` instance** is available as a Spring bean. You can inject it anywhere in your application:
    
    ```java
    @Autowired
    private Jsonb jsonb;
    ```
    
3. **Preferred implementation**:
    - JSON-B is just a **specification**.
    - Spring Boot prefers **Eclipse Yasson** as the default implementation.
    - Spring Boot manages the dependency version for Yasson automatically if you include JSON-B support in your project.

---

### **Dependencies required**

To enable JSON-B auto-configuration in Spring Boot:

**Maven**

```xml
<!-- JSON-B API -->
<dependency>
    <groupId>jakarta.json.bind</groupId>
    <artifactId>jakarta.json.bind-api</artifactId>
</dependency>

<!-- Preferred implementation: Yasson -->
<dependency>
    <groupId>org.eclipse</groupId>
    <artifactId>yasson</artifactId>
</dependency>
```

**Gradle**

```
implementation 'jakarta.json.bind:jakarta.json.bind-api'
implementation 'org.eclipse:yasson'
```

Once these dependencies are present, Spring Boot automatically:

- Detects the API + implementation.
- Creates a **`Jsonb` bean**.
- Makes it available for dependency injection.

---

### **How auto-configuration works**

1. **Classpath detection**:
    
    Spring Boot checks if `jakarta.json.bind.Jsonb` is available on the classpath.
    
2. **Bean creation**:
    
    If yes, it automatically calls `JsonbBuilder.create()` to produce a `Jsonb` instance and registers it in the Spring context.
    
3. **Dependency management**:
    
    If you use Spring Boot’s dependency management, it ensures that **Eclipse Yasson** is included with a compatible version, so you don’t need to manually manage versions.
    

---

### **Using the auto-configured `Jsonb` bean**

Once auto-configured, you can inject it anywhere:

```java
import jakarta.json.bind.Jsonb;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Autowired
    private Jsonb jsonb;

    public String convertUserToJson(User user) {
        return jsonb.toJson(user);
    }

    public User convertJsonToUser(String json) {
        return jsonb.fromJson(json, User.class);
    }
}
```