## ✅ What is the **Builder Pattern**?

* The Builder Pattern is used to **construct complex objects step-by-step**.
* It helps when an object has **many optional parameters** or when the object construction involves several steps.
* It avoids constructor telescoping (multiple constructors with increasing parameters).
* It provides **readable, flexible object creation**.

---

### 🧠 Your Code Explanation

```java
User user = new User.Builder()
                .setFirstName("John")
                .setLastName("Doe")
                .setAge(30)
                .build();
```

* Here, the inner static class `Builder` builds the `User` object.
* You chain methods (`setFirstName`, `setLastName`, etc.) for clarity and readability.
* Once you're done setting values, calling `build()` creates the actual object.
---

### ❓ Common Interview Questions & Sample Answers

---

#### 1. **What is the Builder Pattern?**

**Answer:**

> The Builder Pattern is a creational design pattern used to construct complex objects step-by-step. It allows optional parameters to be set without requiring multiple constructors and improves code readability and maintainability.

---

#### 2. **Why use Builder over constructors?**

**Answer:**

> Constructors can become hard to read and use when you have too many parameters (called "telescoping constructors"). Builder solves this by letting you set only required fields and build the object when ready.

---

#### 3. **Is Builder pattern immutable?**

**Answer:**

> Yes, usually the main class (like `User`) is made immutable. The object is only built once using the builder, and no setters exist in the main class.

---

#### 4. **Can you explain how method chaining works in the Builder Pattern?**

**Answer:**

> Each setter in the builder returns `this`, which allows calls to be chained fluently:

```java
new Builder().setA(...).setB(...).setC(...);
```

---

#### 5. **What’s the difference between Builder and Factory patterns?**

**Answer:**

| Builder Pattern                             | Factory Pattern                          |
| ------------------------------------------- | ---------------------------------------- |
| Step-by-step construction of complex object | Chooses and returns one of many objects  |
| Used when many optional fields              | Used when object creation is complex     |
| Typically returns one final object          | Can return different subclasses or types |

---

#### 6. **Where is the Builder Pattern used in real-life Java?**

* `StringBuilder`
* `java.lang.StringBuffer`
* `Lombok`’s `@Builder` annotation
* Many APIs like `HttpRequest.newBuilder()` in Java 11+

---

#### 7. **Which SOLID principle does the Builder Pattern support?**

**Answer:**

> It supports the **Single Responsibility Principle** — the builder handles object construction while the main class focuses on business logic.

---

### ✅ Summary

| Feature           | Description                                    |
| ----------------- | ---------------------------------------------- |
| Pattern Type      | Creational                                     |
| Use Case          | Building complex objects with many parameters  |
| Benefits          | Readable, flexible, avoids constructor overuse |
| Java Examples     | `StringBuilder`, `HttpRequest.Builder`, Lombok |
| Related Principle | Single Responsibility Principle (SRP)          |

---

Here's a **complete and proper Java example** of the **Builder Pattern**, including:

* Required and optional fields
* Validation
* Method chaining
* `toString()` to print the output

---

### ✅ **Full Example of Builder Pattern in Java**

```java
public class User {
    // Required fields
    private final String firstName;
    private final String lastName;

    // Optional fields
    private final int age;
    private final String email;

    // Private constructor
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.email = builder.email;
    }

    // Static nested Builder class
    public static class Builder {
        // Required
        private final String firstName;
        private final String lastName;

        // Optional
        private int age;
        private String email;

        // Constructor for required fields
        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public Builder setAge(int age) {
            if (age < 0) throw new IllegalArgumentException("Age can't be negative");
            this.age = age;
            return this;
        }

        public Builder setEmail(String email) {
            this.email = email;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }

    @Override
    public String toString() {
        return "User [firstName=" + firstName + 
               ", lastName=" + lastName + 
               ", age=" + age + 
               ", email=" + email + "]";
    }

    // Main method for testing
    public static void main(String[] args) {
        User user = new User.Builder("John", "Doe")
                        .setAge(30)
                        .setEmail("john.doe@example.com")
                        .build();

        System.out.println(user);
    }
}
```

---

### 🧾 **Output**

```
User [firstName=John, lastName=Doe, age=30, email=john.doe@example.com]
```

---

### 🔍 Highlights

* **Required fields**: Enforced through the builder constructor.
* **Optional fields**: Set using method chaining.
* **Validation**: Example shown for age.
* **Immutability**: No setters in the `User` class.
* **Clean syntax**:

  ```java
  new Builder("John", "Doe")
      .setAge(30)
      .setEmail("john@example.com")
      .build();
  ```

---

### ✅ **Lombok-Based Builder Pattern Example**

#### 🔧 Add Lombok dependency (if you're using Maven):

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version> <!-- or latest -->
    <scope>provided</scope>
</dependency>
```

> 💡 For IDE support, install the Lombok plugin and enable annotation processing.

---

### 🧑‍💻 **Java Code Using Lombok**

```java
import lombok.Builder;
import lombok.Getter;
import lombok.ToString;

@Getter
@ToString
@Builder
public class User {
    private String firstName;
    private String lastName;
    private int age;
    private String email;

    public static void main(String[] args) {
        User user = User.builder()
                .firstName("John")
                .lastName("Doe")
                .age(30)
                .email("john.doe@example.com")
                .build();

        System.out.println(user);
    }
}
```

---

### 🧾 **Output**

```
User(firstName=John, lastName=Doe, age=30, email=john.doe@example.com)
```

---

### 🎯 Benefits of Using Lombok's `@Builder`

| Feature               | Benefit                                                              |
| --------------------- | -------------------------------------------------------------------- |
| Less boilerplate code | No need to write constructor, setters, or builder manually           |
| Readable syntax       | Easily chain values using `builder()`                                |
| Integration friendly  | Works well with `@Getter`, `@ToString`, and other Lombok annotations |
| Common in production  | Often used in DTOs, request objects, service inputs, etc.            |

---


## **Why the Builder pattern (especially with a static inner class)** is often preferred over regular object creation with setters.

---

## ✅ Your Suggestion (Using Setters)

```java
User user = new User();
user.setAge(30);
user.setEmail("john.doe@example.com");
```

### ✅ Pros:

* Simple and readable
* Familiar to most Java developers

### ❌ Cons:

1. **Object can be in an inconsistent state**:

   * Between constructor call and the last setter, the object might be incomplete.
   * Someone might forget to set required fields.

2. **No immutability**:

   * Fields can be modified later via setters, which makes the object mutable.

3. **Too many constructor overloads if required fields vary**:

   * `User(String firstName)`, `User(String firstName, String lastName)`, etc.

---

## ✅ Builder Pattern

```java
User user = new User.Builder("John", "Doe")
                   .setAge(30)
                   .setEmail("john.doe@example.com")
                   .build();
```

### ✅ Pros:

1. **Immutable objects**:

   * The `User` object can be made `final` and fields can be `private final`.

2. **Fluent API**:

   * Chaining methods makes it easy to read and use.

3. **Validation before object creation**:

   * The builder can validate input in `build()` before creating the object.

4. **Handles optional fields cleanly**:

   * Required fields go into the builder's constructor.
   * Optional fields are added via setters.

5. **Scalable and readable**:

   * Works well when there are many fields (esp. optional ones).
   * You avoid telescoping constructors (many overloaded constructors).

---

## 📦 Example Implementation

```java
public class User {
    private final String firstName; // required
    private final String lastName;  // required
    private final int age;          // optional
    private final String email;     // optional

    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.email = builder.email;
    }

    public static class Builder {
        private final String firstName;
        private final String lastName;
        private int age = 0;
        private String email;

        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public Builder setAge(int age) {
            this.age = age;
            return this;
        }

        public Builder setEmail(String email) {
            this.email = email;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }

    // Getters only, no setters – makes object immutable
}
```

---

## 🧠 When to Prefer Builder

Use Builder Pattern when:

* You have **many optional fields**.
* You want **immutability**.
* You want to **ensure required fields are set**.
* You need a **clean and readable** object creation process.

---

## TL;DR

| Criteria                | Setter Approach | Builder Pattern             |
| ----------------------- | --------------- | --------------------------- |
| Immutability            | ❌               | ✅                           |
| Handling many fields    | 😵 (Messy)      | ✅ (Clean and scalable)      |
| Fluent API              | ❌               | ✅                           |
| Validation before build | ❌               | ✅                           |
| Easy to forget required | ✅               | ❌ (enforced in constructor) |
