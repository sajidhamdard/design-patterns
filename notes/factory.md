## Factory Pattern

Creates objects without exposing the instantiation logic.

```java
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() { System.out.println("Drawing Circle"); }
}

class Rectangle implements Shape {
    public void draw() { System.out.println("Drawing Rectangle"); }
}

class ShapeFactory {
    public Shape getShape(String type) {
        if ("CIRCLE".equals(type)) return new Circle();
        else if ("RECTANGLE".equals(type)) return new Rectangle();
        return null;
    }
}
```

Great! The code you've provided is a classic example of the **Factory Design Pattern**. In interviews, once you present this code, interviewers will often ask **cross-questions** to assess your understanding of:

* Factory Pattern principles
* Design flexibility
* Real-world usage
* Comparison with other patterns

---

## ✅ **Factory Pattern: Interview Questions & Answers**

---

### 🔁 **1. What is the Factory Design Pattern? Why Do We Use It?**

> **Answer:**
> The Factory pattern provides an interface for **creating objects** but **lets subclasses or methods decide** which class to instantiate.
> It helps in **decoupling object creation** from the client code and promotes loose coupling and better maintainability.

---

### 🧱 **2. How Is Your Code an Example of Factory Pattern?**

> **Answer:**
> In the `ShapeFactory` class, the `getShape()` method acts as a factory. Based on the input string (`"CIRCLE"` or `"RECTANGLE"`), it decides which subclass (`Circle` or `Rectangle`) of the `Shape` interface to instantiate and return.

---

### 🔄 **3. How Is Factory Pattern Different from Abstract Factory?**

> **Answer:**

| Feature    | Factory Pattern                | Abstract Factory Pattern                   |
| ---------- | ------------------------------ | ------------------------------------------ |
| Returns    | One product                    | A family of related products               |
| Complexity | Simple                         | More complex (uses multiple factories)     |
| Example    | `ShapeFactory` returns `Shape` | `GUIFactory` returns `Button`, `Menu` etc. |

---

### 🧩 **4. What Problem Does Factory Pattern Solve?**

> **Answer:**
> It solves the problem of **tight coupling**. Instead of instantiating classes directly (`new Circle()`), the client uses a factory. If a new shape is introduced, the factory can be extended with minimal impact on existing code.

---

### 🧪 **5. How Would You Test This Factory?**

> **Answer:**
> Create test cases for each shape:

```java
ShapeFactory factory = new ShapeFactory();
Shape shape = factory.getShape("CIRCLE");
shape.draw(); // Should print "Drawing Circle"
```

Also test:

* Case-insensitive inputs
* Invalid input (should return `null` or throw an exception)

---

### 🧠 **6. What Are the Advantages of Using Factory Pattern?**

> **Answer:**

* Promotes **loose coupling** between client and implementation
* Makes code **extensible** (easy to add new classes)
* Hides the **instantiation logic** from the client
* Promotes **Single Responsibility Principle** (creation logic is separate)

---

### ⚠️ **7. What Are the Limitations or Drawbacks?**

> **Answer:**

* Adding a new product may require **modifying the factory**, which might violate **Open/Closed Principle**
* Code can become harder to read if factories are too complex
* Can lead to **lots of small classes** if overused

---

### 🔄 **8. Can You Improve or Extend This Factory Implementation?**

> **Answer:**
> Yes:

* Make input case-insensitive
* Use an enum instead of strings to avoid typo issues
* Throw a custom exception instead of returning `null`
* Move creation logic to separate factories (Abstract Factory pattern)

Example improvement:

```java
public Shape getShape(String type) {
    if (type == null) throw new IllegalArgumentException("Shape type is null");
    switch (type.toUpperCase()) {
        case "CIRCLE": return new Circle();
        case "RECTANGLE": return new Rectangle();
        default: throw new IllegalArgumentException("Unknown shape: " + type);
    }
}
```

---

### 🔌 **9. Where Have You Used Factory Pattern in Real Projects?**

> **Answer:**
> I’ve used the Factory pattern in:

* **Parsing JSON/XML** where different parsers are created based on file type
* **UI components**: Creating buttons or inputs based on configuration
* **Database access**: Creating DAO implementations based on the data source

---

### 📦 **10. Where Is Factory Pattern Used in Java?**

> **Answer:**

* `java.util.Calendar.getInstance()`
* `java.text.NumberFormat.getInstance()`
* `java.sql.DriverManager.getConnection()`
* Spring Framework: `ApplicationContext.getBean()`

---

## ✅ **Real-World Factory Pattern Example**

### 📦 Use Case:

Suppose shapes are configured in a config file or database, and you want to create shapes dynamically using a factory without changing code every time a new shape is added.

---

### 🧱 **Step 1: Shape Interface & Implementations**

```java
public interface Shape {
    void draw();
}
```

```java
public class Circle implements Shape {
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

public class Rectangle implements Shape {
    public void draw() {
        System.out.println("Drawing Rectangle");
    }
}
```

---

### 🏭 **Step 2: ShapeFactory Using Reflection**

```java
import java.util.HashMap;
import java.util.Map;

public class ShapeFactory {
    private static final Map<String, Class<? extends Shape>> shapeRegistry = new HashMap<>();

    static {
        shapeRegistry.put("CIRCLE", Circle.class);
        shapeRegistry.put("RECTANGLE", Rectangle.class);
    }

    public Shape getShape(String type) {
        Class<? extends Shape> shapeClass = shapeRegistry.get(type.toUpperCase());
        if (shapeClass == null) {
            throw new IllegalArgumentException("Unknown shape type: " + type);
        }
        try {
            return shapeClass.getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            throw new RuntimeException("Error creating shape instance", e);
        }
    }

    // Optional: Allow external registration of new shapes
    public void registerShape(String type, Class<? extends Shape> shapeClass) {
        shapeRegistry.put(type.toUpperCase(), shapeClass);
    }
}
```

---

### 🔁 **Step 3: Usage Example**

```java
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();

        Shape circle = factory.getShape("circle");
        circle.draw();

        Shape rectangle = factory.getShape("rectangle");
        rectangle.draw();
    }
}
```

---

### ✅ **Advantages of This Real-World Version**

| Feature                       | Benefit                                      |
| ----------------------------- | -------------------------------------------- |
| Uses reflection               | No need to modify factory when adding shapes |
| Registry pattern              | Extensible via `registerShape()`             |
| Follows Open/Closed Principle | Add new shapes without touching factory code |
| Case-insensitive input        | Better usability                             |
| Proper error handling         | More production-like                         |

---

### 📚 **Bonus Interview Tip:**

If asked how you'd **scale this** further (for Spring-based apps), mention:

> “In a real-world Spring application, I might use dependency injection and leverage Spring’s `@Component` and `@Autowired` mechanisms to automatically register shapes in a factory using the `ApplicationContext`.”
